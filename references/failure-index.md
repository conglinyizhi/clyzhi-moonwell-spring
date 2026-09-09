# 常见坑：按错误表现检索

这是索引，不是语言手册。命中后先读官方 skill 和当前工具输出；历史记录只解释“为什么曾经这样处理”。状态以 moon `0.1.20260904`（2026-09-04）复测为准。

- 编译器报具体诊断码，或某 API 签名不确定：`moon explain --diagnostic <code-or-name>`、`moon ide doc`、官方 `moonbit-orientation`
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
