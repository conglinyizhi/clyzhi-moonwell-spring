# 常见坑：按错误表现检索

这是索引，不是语言手册。命中后先读官方 skill 和当前工具输出；历史记录只解释“为什么曾经这样处理”。状态以 moon `0.1.20260904`（2026-09-04）复测为准。

- 编译器报具体诊断码，或某 API 签名不确定：`moon explain --diagnostic <code-or-name>`、`moon ide doc`、官方 `moonbit-orientation`
- **想查错误码 / warning 全集，不要另建索引**：`moon explain --diagnostic` 不带参数就会列出全部
  （前半是约 90 条 warning 的 mnemonic/description/id/state，后半是 `Available non-warning diagnostics`），
  且它来自**本机编译器**本身，比抄一份静态索引可靠。`moon explain --diagnostic 4014`（可省 `E`）
  给出完整解释与错误示例；`moon explain --attribute` 列全部属性
- 不在终端时查单个码：`https://docs.moonbitlang.com/en/latest/language/error_codes/E0001.html`
  （必须是 `.com` + `/en/latest/`；`.cn` 同名路径与 `error_codes.html` 索引页都是 404，
  `_sources/...md` 是 Sphinx 原始源文件）
- `moon ide doc` 返回 `unimplemented` / 没有预期 API：先确认模块、依赖、target 和本机符号索引；旧的 `@async/fs` 默认 target 陷阱已于 `0.1.20260904` 消失，详见 `moon-ide-doc-gotcha.md`
- native `moon run` / `moon build` 找不到 C compiler、linker 或 `/usr/bin/lib.exe`：官方 `moonbit-c-binding`、`make-moonbit-c-bindings`；`0.1.20260904` 当前机器未设 `MOON_CC` 仍可复现，设置 `MOON_CC=gcc` 后通过。它是环境 / toolchain 选择问题，不要泛化成所有项目都会失败
- async trait 的同步方法调用 async 函数：旧的“编译通过但 impl 静默丢弃”已于 `0.1.20260904` 消失；当前报 `E4149 cannot call async function in non-async function`，见 `../history/legacy-patches.md` 的补丁23与 `../history/nightly-retest-20260904.md`
- core 里找不到文件 I/O：仍未发现 `moonbitlang/core/fs`、`core/file` 或 `core/io`；使用 `moonbitlang/async/fs`。此结论在 `0.1.20260904` 复测仍成立
- `no version satisfies requirement ...`：读 `../playbooks/mooncakes-publish.md`；registry 传播延迟需要真实发布链路才能复现或排除，当前不能标为已修复
- 发布成功但下游包仍解析不到上游新版本：刷新 Mooncakes registry，重跑解包复检；见发布 playbook。当前 registry 查询正常，但这不反证时序问题
- Rabbita SSR 页面没有预期 input 数据：读 `../playbooks/rabbita-fullstack.md` 的 SSR input / prefetch 小节；当前未重新搭建完整 SSR 项目，不能声称已修复
- Rabbita 静态页资源 404 或中文显示异常：读 `../playbooks/rabbita-fullstack.md` 的 static / charset 小节；这是项目配置问题，当前未证实框架层有变化

## 最小证据链

```text
错误原文 / 复现命令
→ 当前项目 moon.mod、moon.pkg、moon.lock（如有）
→ moon ide doc / moon search / Mooncakes docs
→ 官方 skill
→ 月井之春专项 playbook
→ 历史记录（只在需要解释时）
```

## 语言 / FFI 报错原文 → 根因（2026-09-09 实测，moon 0.1.20260904）

以下每条都以**编译器实际输出的英文原文**为检索键。官方 skill 已有概念说明的，在条目里指回去；
官方没写「会看到什么报错」的，这里补上。来源：在 MoonBit native 项目上实现统一搜索网关时逐条踩到。

- **`Cannot create values of the read-only type: X`** 同时出现 **`There is no record definition with the fields: ...`**
  - 现象：在 A 包构造 B 包声明的 struct / enum 字面量，两条一起报
  - 根因：B 包只写了 `pub struct` / `pub enum`。`pub` 只允许外部**读**，不允许外部**构造**
  - 修法：给需要外部构造的类型写 `pub(all) struct` / `pub(all) enum`；只想暴露不透明类型才保留 `pub`
  - 概念见官方 `moonbit-agent-guide` 的 Visibility 小节（`pub(all)` allows external construction）

- **`Type X does not implement trait T, although an impl is defined`** / hint 列出 `method xxx is missing`
  - 现象：impl 明明写了，下游包仍说类型没实现 trait
  - 根因：impl 少了 `pub`
  - 修法：`pub impl Trait for Type with fn ...`。少了 `pub`，impl 只在本包内可见
  - 这条报错极具误导性——先查 `pub`，再去怀疑方法签名

- **`Parse error, unexpected token 'async', you may expect id (lowercase start)`**（位置在 `impl ... with fn` 之后）
  - 根因：`impl Trait for T with fn m` 语法**不能写 `async` 关键字**；是否异步由 trait 声明决定
  - 修法：`pub impl T for X with fn m(self, ...) { ... }`，方法体里正常调用 async API

- **`Expr Type Mismatch: has type UInt16, wanted Char`**
  - 根因：`s[i]` 返回 UTF-16 码元 `UInt16`，不是 `Char`
  - 修法：要 `Char` 用 `s.get_char(i) -> Char?`；遍历用 `for c in s`；需要「按下标 + 前瞻」时先 `for c in s { arr.push(c) }` 转 `Array[Char]`
  - 官方 `moonbit-agent-guide` 的 String 小节有说明
  - **连带坑**：`s[i].to_string()` 得到的是**码元的十进制**，不是字符。写 hex/随机串生成时，
    `"0123456789abcdef"[(v>>4)&15].to_string()` 会产出 `"48"` 而不是 `"0"`，结果长度翻几倍。
    必须用 `s.get_char(i)` 拿 `Char` 再 `.to_string()`
  - **另一个变体：URL 百分号编码**。`for c in s` 拿到的是 Char（UTF-16 码元），
    直接对码点做 `%XX` 转义，中文会编错（一个汉字是 3 个 UTF-8 字节）。
    正确做法：先 `@utf8.encode(s)` 拿字节，再逐字节编码

- **把 `String::to_bytes()` 的结果传给 C：乱码 / `strlen` 只有 1 / 字节之间夹 `\x00`**
  - 根因：`to_bytes()` 返回的是 **UTF-16 原始字节**（实现为 `FixedArray::make(len*2)` + `blit_from_string`），不是 UTF-8
  - 官方 skill 只说「String 是 UTF-16」并给了 `@utf8.encode`，**没提醒 `to_bytes()` 是跨 FFI 的错误选择**
  - 修法：MoonBit → C 一律 `@utf8.encode(s)`；C → MoonBit 返回 `moonbit_bytes_t` 后 `@utf8.decode_lossy(b)`
  - 附加：**不要假设 `Bytes` 有 NUL 结尾**。需要 `const char*` 的地方，C 侧自己 `malloc` + `memcpy` + 补 `\0`
  - 最小复现：C 探针同时返回 `strlen((const char*)b)` 与 `Moonbit_array_length(b)`，两者不一致即中招
  - **反方向同样中招：`Bytes::to_unchecked_string()` 不是 UTF-8 解码**。它把字节按 UTF-16 码元重新解释，
    读 HTTP 响应体 / 文件用它，非 ASCII 直接变乱码。正确写法是 `&@io.Data::text()` 或 `@utf8.decode(b)`
  - 实测：`@utf8.encode("请求频率过高")` 得 18 字节；`to_unchecked_string()` 得长度 9 的乱码，
    `@utf8.decode` 得长度 6 的正确文本。**只测 ASCII 负载时两边都对**，必须用非 ASCII 才暴露
  - 跨实现边界时尤其危险：客户端和服务端如果都只在 ASCII 上测过，这个错误会一直活着

- **`This expression has type () -> Json, wanted Json`**
  - 根因：`Json::null` 是构造函数（函数），不是值
  - 修法：写 `Json::null()`

- **C 风格注释 `/* ... */` 报 `Parse error, unexpected token infix`**
  - 修法：MoonBit 只认 `//` 与 `///|`（文档注释），不要用 `/* */`
- **用 MoonBit 字符串拼前端代码，浏览器报 `SyntaxError: string literal contains an unescaped line break`**
  - 根因：MoonBit 的 `\n` 在编译期就变成真换行，拼进 JS 单引号串里就是裸换行，整个 `<script>` 解析失败、页面全白
  - 修法：写 `\\n`（MoonBit 源里两个反斜杠 + n），JS 才收到转义序列
  - 通用规则：**宿主语言拼目标语言代码时，目标语言字符串里的换行/引号/反斜杠都要多一层转义**
  - 防线：编译期查不出来，必须在验收里加 `node --check`（提取 `<script>` 后检查）

- **FTS5 中文搜不到（英文 / 数字正常）**
  - 现象：索引了含「统一搜索网关设计说明」的文档，`MATCH '网关'` 返回 0 行
  - 根因：FTS5 的 `unicode61` 分词器按空白 / 标点切词，**整段连续 CJK 会被当成一个 token**
  - 修法：索引与查询都做 **bigram 预处理**——CJK 连续段切成重叠二元组（`统一搜索网关` → `统一 一搜 搜索 索网 网关`）、非 CJK 按空白切词、空格分隔；查询串同样处理后用**短语查询**保证顺序一致
  - 副作用：`snippet()` 拿到的是切分后的文本，不能直接用；改为在应用侧从原文截窗口
  - 单字 CJK 查询会退化成整段 token 而匹配不到，需要时用 `LIKE` 兜底
  - 相关：SQLite 官方 amalgamation 自带 fts5 源码，`#define SQLITE_ENABLE_FTS5` 后 `#include "sqlite3.c"` 即可启用；`moonbit-community/sqlite3@0.2.1` 未开该宏（`sqlite_compileoption_used("ENABLE_FTS5")` 返回 0）

- **进程无征兆 abort，`exit status 134`（SIGABRT），栈顶是 `IoHandle::write_via_worker`**
  - 原文：`PanicError at @moonbitlang/async/internal/event_loop.IoHandle::write_via_worker (io.mbt:182)`
  - 根因：`write_via_worker` 开头是 `guard! handle.write is Idle`。
    `@stdio.stdout` / `@stdio.stderr` 各是一个**全局唯一的 `IoHandle`**，
    两个协程同时写同一个句柄，第二个必然命中 guard → abort。
  - 触发条件：`TcpServer::run_forever` 给每个连接 `spawn_bg`，所以只要在请求处理里
    写一行 stderr（哪怕只是访问日志），并发压一下就必崩。
  - 修法：所有对同一句柄的写串行化。用信号量当锁即可：
    ```moonbit
    let log_lock : @async.Semaphore = @async.Semaphore(1)
    async fn log_line(msg : String) -> Unit {
      log_lock.acquire()
      defer log_lock.release()
      @stdio.stderr.write(msg)
      @stdio.stderr.write("\n")
    }
    ```
  - 注意：顶层 `let` 名必须小写（大写会被要求改 `const`）。
  - 排查提示：`exit 134` + 日志里最后一行正常输出 = panic，不是超时、不是 OOM；
    崩溃栈会直接打到 stderr，别只看 ingress 的 `proxy error`。

- **`Cannot implement trait 'X' because it is readonly.`**（error 4145）
  - 现象：A 包声明 `pub trait T`，B 包写 `pub impl @a.T for MyType`，报 trait 只读
  - 根因：`pub trait` 只允许外部**使用**，不允许外部**实现**
  - 修法：`pub(open) trait T { ... }`
  - 改完可能接着撞下面那条 orphan rule —— `pub(open)` 只解决可见性，不解决归属

- **`Cannot implement foreign trait @a.T for foreign type @a.S`**（error 4061）
  - 根因：孤儿规则。不能给「外部的 trait + 外部的类型」写 impl
  - 修法：impl 必须写在**定义 trait 的包**或**定义类型的包**里。
    下游想用就走 `T::method(x)`，或让上游包加 `pub extend`
  - 实测：把 impl 挪进 trait 所在包后，下游用 `f.method()` 可编译

- **`Warning (implicit_impl_as_method)`：The methods m from `impl T for X` are implicit promoted as regular method for `X`. This behavior is deprecated and will be removed in the future.**
  - 现象：`x.m()` 能编译，但有 deprecated 警告，将来会失效
  - 修法：在定义 impl 的包里加 `pub extend X with T::{m}`；不想暴露就给它加 `#deprecated`
  - 实测：加 `pub extend` 后从 1 warning 降到 0 warnings

- **`Using let statement in 'the action part of a matching case' directly is not allowed. Consider moving the let binding into a curly braces block.`**（error 3002）
  - 连带：`Parse error, unexpected token '+', you may expect '=>'`
  - 现象：match 分支里直接写多行语句
  - 根因：分支体是单个表达式，`let` 这类语句必须包在 `{}` 里
  - 修法：`1 => { let a = 10; a + 1 }` —— 多语句分支一律加花括号

- **`Expected lower case identifier for name of let, found upper case identifier. Did you mean const?`**（error 3002）
  - 根因：顶层 `let` 只能用小写名
  - 修法：改小写（`let log_lock : ... = ...`）；要常量语义就用 `const`（const 用大写）
  - 常见于「全局锁 / 全局计数器」

- **`Expr Type Mismatch: has type Int, wanted Double`**（`Json::number(n)` 里传 Int，error 4014）
  - 修法：`Json::number(n.to_double())`。`Json::number` 只收 Double

- **`This expression has type Int, its value cannot be implicitly ignored (hint: use ignore(...) or let _ = ...)`**（error 4139，外层常伴随 `Expr Type Mismatch`）
  - 现象：`let y = if x > 0 { 1 }` —— **无 `else` 的 `if` 是 Unit 类型**
  - 修法：补 `else`；确实只要副作用时让分支体是 Unit（如 `ignore(...)`）

- **`The type @moonbitlang/async/http.Request has no field method.`**
  - 根因：字段名是 `meth`，类型是 `RequestMethod`（enum），不是 `String`
  - 修法：显式 `match req.meth { Get => "GET"; Post => "POST"; ... }`。
    `RequestMethod` 的 `Show` impl 只在 `http/deprecated.mbt` 里，别依赖 `to_string()`

- **SQLite 在 WAL 下仍然每个写事务 fsync（native 应用性能坑，不是语言报错）**
  - 现象：本地 0-1ms 的接口，部署到容器 bind mount 后变 1-3 秒；多个请求在**同一毫秒**一起返回
  - 根因：`PRAGMA journal_mode=WAL` 只改了日志模式，**`synchronous` 默认仍是 `FULL`**，
    WAL 下每个提交都要 fsync。一个请求里 3 次自动提交写就是 3 次 fsync
  - 修法：`PRAGMA synchronous=NORMAL`（WAL 下只在 checkpoint 同步；断电最多丢最后几个事务，不损坏库）；
    同一请求内的多次写合并进一个 `BEGIN/COMMIT`；常查的列补索引
  - 排查手法：给 sqlite 绑定加累计耗时计数，访问日志打 `total=Xms db=Yms/Nops`。
    `db=` 接近总耗时 = 存储问题；`db=` 很小但总耗时高 = 应用问题

- **`@async.Semaphore` 当锁用时 release 必须 1:1，否则 abort**
  - 报错：`semaphore: too many release`
  - 修法：`acquire()` 后紧跟 `defer sem.release()`，不要在多个分支里各释放一次

- **`TcpServer::run_forever` 对每个连接 `spawn_bg`（`allow_failure` 默认 true）**
  - 含义：请求处理天然并发；处理函数里的一次 `abort()` 会带走**整个进程**，不是只断那一条连接
  - 推论：请求路径上任何全局可变资源（`@stdio` 句柄、单例连接、共享 Map）都要显式串行化

- **`The type Error is not a trait`**（error 4100）
  - 现象：写 `pub impl Error for MyError with message(self) { ... }`
  - 根因：新版 core 里 `Error` 是**内置类型**不是 trait（`pub fn Error::to_string(self : Error) -> String`）
  - 修法：直接删掉该 impl。`suberror X { ... }` 本身就能作为 `Error` 值进 `Result[T, Error]`，
    函数签名写 `-> Result[T, Error] noraise` 即可，构造 `Err(X(...))` 会自动协变

- **`Using constructors as higher order function directly is forbidden.`**（error 4203）
  - 现象：`on_click=emit(SetTab)`、`tag=SomeLoaded` 这类把带参构造器当函数传
  - 修法：包一层闭包 —— `fn(v) { emit(SetTab(v)) }` / `fn(r) { SomeLoaded(r) }`
  - 注意：`emit` 的签名是 `(Msg) -> Cmd`，而 `emit(Constr)` 得到的是 `(T) -> Cmd`，类型对不上

- **`Function with labelled arguments can only be applied directly.`**（error 4117，常连带 `Using constructors as higher order function directly is forbidden`）
  - 现象：`let m = Map[String, Json]([])`
  - 根因：`Map` 构造器带 labelled 参数，不能这样显式套类型参数调用
  - 修法：写 `let m = Map([])`，类型交给上下文推断（或 `let m : Map[String, Json] = Map([])`）

- **Rabbita：`@async/http` 的 JS 后端要求绝对 URL**
  - 现象：浏览器里 `@http.get("/v1/modes")` 抛 `InvalidFormat`，页面静默失败
  - 根因：`resolve_url` 只认 `http://` / `https://` 前缀（`async/src/http/request.mbt`），
    不像浏览器 `fetch` 那样接受相对路径
  - 修法：`extern "js" fn page_origin() -> String = "() => window.location.origin"`，
    拼 `page_origin() + path`
  - 同类：`@rabbita/http` 的 `op.mbt` 内部会 `resolve_request_url(url, origin)`，
    所以用它时相对路径反而没问题；只有直接调 `@async/http` 才要自己拼

- **Rabbita：`@rabbita/http` 不支持自定义请求头**
  - 根因：`op.mbt` 只写 `Content-Type` 与 `User-Agent`
  - 后果：Bearer 鉴权的接口没法用它，得 `@cmd.perform(fn(r) { GotX(r) }, async fn() { ...@async_http... })` 自己拿 header 控制权

- **Rabbita：初始加载要 `create_state_with_init`，`create_state` / `elmish` 没有初始 Cmd 口子**
  - 现象：`create_state` 的 init 固定是 `_ => (model, none)`，首屏想自动拉数据没地方挂
  - 修法：`@rabbita.create_state_with_init(init=fn(emit) { (model, load_cmd(emit)) }, update~)`
  - 约束：`Model : Eq`；原地修改后要返回**新值**（或像本项目那样用 `version` 计数器实现 `Eq`），
    否则 `Val` 变更检测认不出变化，界面不刷新

- **Rabbita：hash 路由用 `@sub.on_url_changed` + `url.fragment`**
  - 要点：该订阅是 **app-scoped**，只有根 cell 的 `subscriptions` 返回它才生效
  - 写法：`subscriptions=fn(_, emit) { @sub.on_url_changed(fn(u) { emit(UrlChanged(u)) }) }`，
    update 里 `match url.fragment { Some("modes") => Modes; ... }`；
    切标签时自己写 `window.location.hash`
  - 只写 hash 不订阅的话，浏览器前进/后退不会更新界面

- **`#export_name "x" can only be used in a foreign library`**（E4219，moon `0.1.20260907` 实测）
  - 修法：包声明加 `pkgtype(kind: "foreign_library")`
  - **是冒号，不是等号**：`pkgtype(kind = "...")` 报 `Parsing error: unexpected token`
  - 这条报错也是判断「当前包能不能导出符号」最快的手段

- **`data did not match any variant of untagged enum BoolOrLink`**（moon `0.1.20260907` 实测）
  - 场景：给 wasm-gc 的 `imported-string-constants` 传布尔或数组
  - 根因：该键只收字符串。传 `true` 或 `[...]` 都会落进这条；字符串值（含错误值）反而能通过解析
  - 后果：解析通过不等于配置正确——**错误字符串要构建 + 导入后才暴露**，见下面的 `Cannot find package '_'`

- **`Unexpected key 'link' found in moon.pkg`**（moon `0.1.20260907` 实测）
  - 场景：照官方示例把 `link` 当 `moon.pkg` 顶层键写
  - 修法：link 配置走 `options("link": {...})`；顶层只放 `supported_targets` / `pkgtype(...)` 这类
  - 顶层写 `link({...})` 块形式报 `Unexpected key 'link' found in moon.pkg.`；
    写 `"link": {...}` 对象形式报 `Parsing error: unexpected token "link"`；两种都不成立

- **`unknown token` 出现在 `supported_targets` 里**（moon `0.1.20260907` 实测）
  - 原文：`` invalid `supported_targets` expression `js,wasm-gc,native`: unknown token `js,wasm` ``
  - 根因：**不是逗号分隔**，`+` 才是连接符；逗号会被当成 token 的一部分切在中间
  - 修法：写 `supported_targets = "js+wasm-gc"`
  - 报错尾部自带提示 `Valid examples: js or all-js+wasm-gc`——`all-js+wasm-gc` 形式合法，拿不准时照它写

- **`Package 'x' does not support target backend 'wasm-gc'. Supported backends: [js]`**
  （moon `0.1.20260907` 实测）
  - 场景：`supported_targets` 只声明了 js，却直接 `moon build --target wasm-gc`
  - 修法：先扩 `supported_targets`，再构建。报错里会直接列出当前支持的后端，不用猜

- **`parse error` 指向 `extern "js"` 后的 `async`**（moon `0.1.20260907` 实测）
  - 表现：`extern "js" async fn f() -> Unit = "..."` 解析失败，光标停在 `async`
  - 根因：async 是 MoonBit 侧的效果，**不写在 extern 声明上**
  - 修法：extern 只声明返回 `@js_async.Promise[T]`，async 留给调用方的 MoonBit 函数
  - 同族：`async () => ...` 也是 parse error（`async` 只能在 `fn` 前）；箭头函数靠上下文推断 async-ness，直接写 `() => { ... }`

## 工具链与运行时行为（没有编译器报错可搜）

这些坑的共同点：命令能跑起来，或者报错文本本身极具误导性，按报错原文搜搜不到。
状态以 moon `0.1.20260904`（2026-09-10 逐条复现）为准。

- **`.mbtx` 脚本默认编译目标是 wasm，不是 native**
  - 表现：`extern "c" fn` 报 `Error: [4156] extern "C" is unsupported in wasm backend`；
    `@process.spawn_orphan` / `read_from_process` 报 `Value spawn_orphan not found in package process`
  - 根因：不是 API 不存在，是 `@process` 的进程 / 管道部分在 wasm 后端里就没有
  - 修法：一律 `moon run --target native foo.mbtx`

- **`.mbtx` 里 `async fn main` 必须 import `moonbitlang/async` 本体**
  - 表现：`Error: [4037] Cannot use 'async fn main': package moonbitlang/async is not imported.`
  - 根因：只 import 子包（`.../fs`、`.../stdio`、`.../http`）不算，本体要单独列出
  - 修法：import 列表里加不带子路径的那一行。普通包（带 moon.pkg 的）里是同样规则

- **`.mbtx` 的子包 import 必须带版本号**
  - 表现：`Failed to parse single file front matter configuration: multiple versions specified for module 'moonbitlang/async': '0.20.1' and '0.21.3'`
  - 根因：`"moonbitlang/async@0.20.1"` 带了版本，而 `"moonbitlang/async/http"` 没带、按 registry 最新版解析，同一模块出现两个版本
  - 修法：同一模块每一行都写成「模块@版本/子包」，如 `"moonbitlang/async@0.20.1/http"`

- **`moon build foo.mbtx` 的产物名是固定的 `single.exe`**
  - 路径：`<脚本所在目录>/_build/native/debug/build/single/single.exe`
  - 后果：同一目录下编译两个 `.mbtx` 会互相覆盖
  - 用法：要同时持有多个脚本产物，build 完立刻 `cp` 成独立文件名再运行

- **`println` 写重定向的 stdout 是块缓冲的；stderr 不是**
  - 现象：脚本 `println(...)` 后输出重定向到文件，**进程还在跑时文件一直是空的**；进程被 kill 时整块丢失
  - 修法：需要「立刻可见」的输出改用 `@stdio.stdout.write(...)`，或直接走 stderr
  - 实测：同一进程里 `@stdio.stderr.write` 的内容立刻可见，`println` 的内容要等进程退出
  - 典型受害场景：测试脚本从子进程 stdout 读端口号做握手——用 `println` 会永远等不到；
    后台服务把日志写 stdout 再被 kill，日志会整块消失，排查时看起来像「什么都没发生」

- **`@fs.write_file` 不传 `create` 不会创建新文件**
  - 表现：`OSError("@fs.open(): \"new.txt\": No such file or directory")`，**但目录明明存在**
  - 根因：`create`（是权限值，不是布尔开关）缺省时 `create_mode` 落到 `TruncateExisting`
  - 修法：要新建就写 `create_mode=@fs.CreateOrTruncate`
  - 实测：同一目录下，新文件 + 默认 → 报错；新文件 + `create_mode` → 成功；已存在文件 + 默认 → 成功
  - 这个报错极具误导性：按 `No such file or directory` 排查会去怀疑路径、权限、cwd，全都不对

- **`moon add pkg@版本` 对已存在的依赖是 no-op**
  - 表现：`Warning: dependency 'x/y' already exists, 'moon add' will not update it. To update ... run 'moon add --upgrade x/y@<version>'`
  - 后果：**在同一个模块里连续 add 两个版本做对比，会拿到同一份代码**——据此得出「两个版本行为一样」的结论是假的
  - 修法：升级用 `moon add --upgrade x/y@<version>`；做版本对比实验时每个版本用干净模块
  - 隐蔽点：warning 不阻断流程，`moon.mod` 静默不变，很容易以为已经切过去了

- **`@http.Request` 的 `path` 带 query string**
  - 表现：路由匹配 `/api/meta` 失败；`/?autorun=1` 被当成一个不存在的静态文件名，返回 404
  - 根因：`request.path` 是完整 request-target，值为 `"/x/y?a=1&b=2"` 这种
  - 修法：路由前先按 `?` 切一刀：`match raw.find("?") { Some(i) => raw[:i].to_owned(); None => raw }`
  - 官方 `moonbitlang/async` 的 `examples/http_file_server` 也是这样处理的

- **wasm-gc 产物导入 Node 报 `Cannot find package '_'`**（moon `0.1.20260907` / Node `v26.8.1` 实测）
  - 原文：`Error [ERR_MODULE_NOT_FOUND]: Cannot find package '_' imported from .../x.wasm`
  - 根因：开了 `use-js-builtin-string` 但没指定字符串常量的命名空间。
    MoonBit 把字符串字面量也做成导入，缺省命名空间是 `_`，Node 把它当裸包名去解析
  - 修法：`options("link": {"wasm-gc": {"use-js-builtin-string": true, "imported-string-constants": "wasm:js/string-constants"}})`
  - **官方 `moonbit-agent-guide` 的 `advanced-moonbit-build.md` 示例值是 `"_"`，那个值在 Node 下正好是这条报错**。
    写 `"_"` 会重现，只是不报「配置错」，而是报「找不到包」——很容易往包管理方向排错
  - 详细配方读 `../playbooks/js-wasm-interop.md`

- **`type incompatibility when transforming from/to JS`**（moon `0.1.20260907` / Node `v26.8.1` 实测）
  - 表现：`import` 报错文本很短，堆栈指向 `this.FunctionDescriptor`；把 `mb_scan("字符串")` 改成 `mb_scan(0)` 也一样报
  - 根因：wasm-gc 没开 `use-js-builtin-string`，MoonBit 用自己的字符串表示，
    JS 字符串不是合法的 `externref` 字符串
  - 修法：补 `use-js-builtin-string: true` + `imported-string-constants: "wasm:js/string-constants"`
  - 锚点：不要被「改传数字也报」带偏——它不是参数类型不匹配，是模块级 ABI 没配

- **`TypeError: WebAssembly.instantiate(): Import #0 "wasm:js-string": module is not an object or function`**
  （Node `v26.8.1` 实测）
  - 场景：用 `WebAssembly.compile` / `instantiate` 手动加载 wasm-gc 产物
  - 根因：**JS String Builtins 是编译期导入**，只在 ESM Integration 路径下链接。
    官方原文：`they cannot be inspected via WebAssembly.Module.imports(mod)`，
    走 `WebAssembly.compile` 相当于「with string builtins disabled」
  - 修法：改用 ESM import——`import { mb_scan } from "./x.wasm"`
  - 陷阱：`WebAssembly.Module.imports(mod)` 也看不到 builtin，**看不到不等于没有**；
    别用「imports 是空的」证明模块自包含

- **wasm-gc 的 JS 字符串互操作不需要任何 Node flag**
  - 官方文档：JavaScript String Builtins（Added in v24.5.0 / v22.19.0，Stability 1.2 Release candidate），
    `automatically enabled through the ESM Integration`
  - 当前 CLI 文档已无 `--experimental-wasm-modules`；`.wasm` 是原生 ESM 扩展名
  - 会打一条 `ExperimentalWarning: Importing WebAssembly module instances is an experimental feature`，不影响功能
  - `--experimental-wasm-js-string-builtins` 之类的 flag **不存在**，Node 会报 `bad option`

- **wasm-gc 跨边界只有标量和字符串直通**
  - 直通：`Int` / `UInt` / `Double` / `Bool` / `String`（参数与返回都行，字符串零拷贝）
  - 不直通：数组、元组、结构体返回的是不透明 wasm 引用，`console.log` 显示 `[Object: null prototype] {}`，
    `typeof` 是 `object` 但没有 `length`
  - wasm-gc **不生成 `.d.ts`**（js target 会生成），导出签名要自己维护
  - 修法：需要复合返回值时，导出配套的取长度 / 取下标的函数，在 JS 侧手工编组

- **js target 的 debug 产物比 release 慢得多**（moon `0.1.20260907` 实测）
  - 表现：同一个字符扫描函数，debug 产物里有 `new _M0TPB8MutLocalGiE(0)` 和 `.val` 字段访问
  - 根因：debug 下 `let mut` 编成堆上装箱对象，每次改动都是对象属性写；
    `UInt16` 比较也变成函数调用而非 `===`
  - 实测：debug 1.38x 于手写 JS，release 消除装箱后降到 1.24x
  - 结论：**拿 js 后端做性能判断必须用 `--release`**，debug 数字没有参考价值

- **导出的 `pub async fn` 是 CPS 形态，JS 不能直接 await**
  - 表现：`#export_name` 的 async 函数在 JS 侧签名是 `f(s, _cont, _err_cont)`，
    调 `await f(x)` 拿不到结果
  - 修法：另包一层返回 Promise：
    `pub fn f_js(s : String) -> @js_async.Promise[String] { @js_async.Promise::from_async(() => { f(s) }) }`
  - 代价：async 会把整个协程运行时内联进产物。实测 20 行源码的 async 导出生成 **1,599 行 JS**，
    其中自己的代码只占 14 行。纯逻辑模块不要为了省事引入 async
