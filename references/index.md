# 月井之春子导航

本文件负责第二层分流。先按任务或错误表现命中一个主题，再读取对应文件；不要默认加载全部 references。

## 按任务

- 官方 MoonBit 能力、工具链、语言或 FFI：`official-skills.md`
- Mooncakes 包搜索、个人包、版本或 target 评估：`mooncakes.md`
- 编译失败、warning 或行为异常的表现检索：`failure-index.md`
- Rabbita + moonback 全栈网站：`../playbooks/rabbita-fullstack.md`
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

## 资料优先级

```text
当前项目文件 / 当前 moon 与 Mooncakes 查询
> 官方 skill 与官方文档
> 月井之春当前规则
> 月井之春历史记录
> 模型记忆
```
