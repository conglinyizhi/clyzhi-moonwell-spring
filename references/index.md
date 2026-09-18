# 月井之春子导航

本文件负责第二层分流。先按任务或错误表现命中一个主题，再读取对应文件；不要默认加载全部 references。

## 按任务

- 官方 MoonBit 能力、工具链、语言或 FFI：`official-skills.md`
- Mooncakes 包搜索、个人包、版本或 target 评估：`mooncakes.md`
- 编译失败、warning 或行为异常的表现检索：`failure-index.md`
- Rabbita + moonback 全栈网站：`../playbooks/rabbita-fullstack.md`
- 产物接入 Node / JS（js 与 wasm-gc target、`#export_name`、`extern "js"`）：`../playbooks/js-wasm-interop.md`
- Mooncakes 发布：`../playbooks/mooncakes-publish.md`
- 更新本 skill：`../maintenance/update-skill.md`
- 历史复盘：`../history/README.md`

## 按错误表现

- `no version satisfies requirement`：可能是 registry 索引或上游依赖未传播；读 `../playbooks/mooncakes-publish.md`
- `moon publish` 当前检查通过、解包检查失败：开发树与发布包不一致，或依赖不可解析；读 `../playbooks/mooncakes-publish.md`
- `moon search` 找不到刚发布的版本：registry 索引尚未刷新；读 `../playbooks/mooncakes-publish.md`
- `moon ide doc` 空结果或 `unimplemented`：项目上下文、依赖、target 或本机符号索引问题；旧 `@async/fs` 默认 target 陷阱已于 `0.1.20260904` 消失。读 `failure-index.md`、`moon-ide-doc-gotcha.md`，并加载官方 `moonbit-orientation`
- native link / C compiler 错误：native 工具链或 FFI 配置；加载官方 C binding skill
- SSR 首屏无数据、页面资源 404 或热更新开发体验异常：Rabbita / moonback 流程误用；读 `../playbooks/rabbita-fullstack.md`
- sync trait 方法调用 async 函数：`0.1.20260904` 已从静默失效改为 `E4149` 诊断；读 `failure-index.md`，再查官方文档
- `Cannot create values of the read-only type` / `There is no record definition with the fields`：跨包构造需要 `pub(all) struct` / `pub(all) enum`；读 `failure-index.md`
- `Type X does not implement trait T, although an impl is defined`：impl 少了 `pub`；读 `failure-index.md`
- 把 `String::to_bytes()` 传给 C 得到乱码 / `strlen` 异常：`to_bytes()` 返回 UTF-16，跨 FFI 改用 `@utf8.encode`；读 `failure-index.md`
- FTS5 中文搜不到（英文正常）：`unicode61` 不切 CJK，需 bigram 预处理；读 `failure-index.md`
- `Cannot implement trait 'X' because it is readonly.`：`pub trait` 只能被外部使用，不能实现；改 `pub(open) trait`；读 `failure-index.md`
- `Cannot implement foreign trait ... for foreign type ...`：孤儿规则，impl 必须与 trait 或类型同包；读 `failure-index.md`
- wasm-gc 产物导入 Node 报 `Cannot find package '_'`：`imported-string-constants` 命名空间没指到 `wasm:js/string-constants`；读 `../playbooks/js-wasm-interop.md`
- `type incompatibility when transforming from/to JS`：wasm-gc 缺 `use-js-builtin-string`；读 `../playbooks/js-wasm-interop.md`
- `WebAssembly.instantiate(): Import #0 "wasm:js-string": module is not an object or function`：用 `WebAssembly.compile` 手动加载了 wasm；改走 ESM `import`；读 `../playbooks/js-wasm-interop.md`
- `#export_name` 报 `can only be used in a foreign library`：包缺 `pkgtype(kind: "foreign_library")`（冒号不是等号）；读 `../playbooks/js-wasm-interop.md`
- 导出的 async 函数在 JS 侧不能 `await`：CPS 形态，要 `@js_async.Promise::from_async` 包一层；读 `../playbooks/js-wasm-interop.md`
- `Warning (implicit_impl_as_method)`：用 `x.m()` 调 trait 方法已废弃，需 `pub extend X with T::{m}`；读 `failure-index.md`
- `Using let statement in 'the action part of a matching case' directly is not allowed`：match 多语句分支要加 `{}`；读 `failure-index.md`
- `Expected lower case identifier for name of let`：顶层 `let` 名必须小写（大写用 `const`）；读 `failure-index.md`
- 本地毫秒级接口部署后变秒级、多个请求同一毫秒一起返回：SQLite WAL 下 `synchronous` 默认仍是 FULL，每写一次 fsync；读 `failure-index.md`
- `.mbtx` 脚本报错（`extern "C" is unsupported in wasm backend` / `Value spawn_orphan not found in package process` / `Cannot use 'async fn main': package moonbitlang/async is not imported`）：默认目标是 wasm，且 `async fn main` 要求 import `moonbitlang/async` 本体；读 `failure-index.md`
- `println` 到重定向的 stdout 长时间不出内容、进程被 kill 后日志整块丢失：`println` 是块缓冲，改用 `@stdio.stdout.write` 或 stderr；读 `failure-index.md`
- `@fs.open(): ... No such file or directory` 但目录确实存在：`write_file` 不传 `create` 默认走 `TruncateExisting`，新建文件要 `create_mode`；读 `failure-index.md`
- `moon add pkg@新版本` 后 `moon.mod` 没变：对已存在的依赖是 no-op，升级要 `--upgrade`；版本对比实验会拿到同一份代码；读 `failure-index.md`
- `@http.Request.path` 匹配不上路由（如 `/?a=1` 返回 404）：它带 query string，路由前先按 `?` 切；读 `failure-index.md`
- `App::mount` 抛 `$PanicError`、页面白屏，栈里有 `VDom::initialize`：参数是裸 id（内部用 `getElementById`），不能写 `"#app"`；读 `../playbooks/rabbita-fullstack.md`
- 判定/过滤逻辑静默恒真或恒假、只伴随 `Warning unused_value`：`if v is k` 里的裸标识符是绑定新变量，永远匹配；常量用字面构造子、比较变量用 `==`；读 `failure-index.md`
- `Error Warning (unused_mut)` 让 `moon check` 失败：`mut` 只用于改变量本身；Array/Map 的 push 与字段赋值不需要；读 `failure-index.md`
- 按下标切片 `s[a:b]` 结果长度不对（emoji/非 ASCII 边界）：当前 nightly 不 raise 而是静默改边界，扫描器改用 `unsafe_substring`；读 `failure-index.md`
- 想解析字符串里的整数：当前 core 无直接入口（`@strconv` 是空包），用 `@bigint.BigInt::from_string` 或自写循环；读 `failure-index.md`
- `SIGTERM` / `Ctrl-C` 收不到、进程只能 `kill -9`：async 程序里有长时间不挂起的同步循环，信号被运行时接管后投不进事件循环；读 `failure-index.md`

## 资料优先级

```text
当前项目文件 / 当前 moon 与 Mooncakes 查询
> 官方 skill 与官方文档
> 月井之春当前规则
> 月井之春历史记录
> 模型记忆
```
