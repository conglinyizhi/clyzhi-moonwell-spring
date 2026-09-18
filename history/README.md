# 月井之春历史记录

> 本目录只记录已经发生的版本、调查和失败复盘。历史内容不自动等于当前规则；当前规则请回到 `SKILL.md`、`references/` 或 `playbooks/`。

## 记录格式

```markdown
## YYYY-MM-DD：主题

- 现象：agent 实际看到的错误或行为
- 结论：当时确认的原因和处理
- 影响文件：当前知识 / 专项流程 / 索引
- 验证：命令、页面或包版本
```

## 2026-09-18：补录 async 运行时的信号失效坑

- 现象：`async fn main` 挂起一次后进不挂起的紧循环，`SIGTERM` / `SIGINT` 完全失效，宽限期过后仍存活，只能 `SIGKILL`。
  这类问题既没有编译器报错也没有 warning，按“SIGTERM 无效”在 MoonBit 侧搜不到任何说法
- 结论：`../references/failure-index.md` 的「工具链与运行时行为」补一条，写清机制（信号被 `sigwait` 线程接管后投给事件循环、进程默认处置同时消失）、上游定性（`moonbitlang/async#612` 认定为 expected behavior）、绕法（`@signal.set_global_cancellation_signals([])`）、上游评论里的 API 单复数笔误，以及“编译器没有相关 warning”这一现状；`../references/index.md` 和 `SKILL.md` 各加一条入口
- 边界：写成“协作式单线程运行时的后果”，不写成“async 有 bug”；上游倾向的 hard timeout 标为倾向，不写成已实现
- 影响文件：`../references/failure-index.md`、`../references/index.md`、`SKILL.md`、`moonwell.toml`
- 验证：moon `0.1.20260916` + `moonbitlang/async@0.22.1` 下复现三 lane（紧循环 `term_ignored` exit=137、每轮挂起 exit 143、非 async 忙等 exit 143）；`@signal` 导出签名核对 v0.22.1 tag 的 `pkg.generated.mbti`；`moon explain --diagnostic` 全集确认无相关 warning

## 2026-09-13：新增 JS / wasm 产物接入 Node 专项

- 现象：把 MoonBit 产物接进 Node 时一连串报错都搜不到根因——`Cannot find package '_'`、
  `type incompatibility when transforming from/to JS`、`WebAssembly.instantiate(): Import #0 "wasm:js-string"`。
  官方 `moonbit-agent-guide` 的 link 配置示例把 wasm-gc 字符串命名空间写成 `"_"`，照抄到 Node 下正好是第一条报错
- 结论：新建 `../playbooks/js-wasm-interop.md`，给出 js / wasm-gc / native 三个 target 的选择依据与导出配方；
  `failure-index.md` 补 6 条编译器原文条目 + 7 条运行时行为条目；索引和 SKILL 粗分流各加入口
- 关键事实：wasm-gc 字符串互操作必须走 ESM `import`（`WebAssembly.compile` 路径 builtins 是关的）；
  命名空间要写 `wasm:js/string-constants`；Node 自 v24.5.0 / v22.19.0 起自动启用，无需 flag；
  当前 CLI 文档已无 `--experimental-wasm-modules`。以上以 Node 官方文档为准，并已在本机复现
- 影响文件：`../playbooks/js-wasm-interop.md`（新增）、`../references/failure-index.md`、
  `../references/index.md`、`../SKILL.md`
- 验证：moon `0.1.20260907` / Node `v26.8.1` 下逐条最小复现；wasm-gc 用例 5/5 与手写 JS 参考实现结果一致；
  字符串零拷贝直通、`Int` 直通、数组返回为不透明引用均已实测确认

## 2026-09-17：新增「静默错解」一节（5 条）

- 现象：写一个约 3k 行的 MoonBit 解析器时踩到一批**不报 error** 的坑：
  `if v is k`（右侧是变量）永远匹配、只给一条 `unused_value` warning；`unused_mut` 是 Error（0015）直接 fail build；
  按码元下标切片 `s[a:b]` 在代理对边界静默改边界（与官方文档写的「可能 raise」不一致）；
  弃用 warning 一簇攒到近 70 条、把真信号淹掉；`@strconv` 是空包、core 没有 String→Int 入口
- 结论：在 `../references/failure-index.md` 新增「静默错解：没有 error，只有容易淹掉的 warning」一节（5 条）；
  `../references/index.md` 按错误表现补 4 条入口；`SKILL.md` 的错误表现提醒加一条（恒真/恒假逻辑）
- 边界：`is` 与切片两条都把「官方文档说法 vs 本机 nightly 实测」写清楚了，以实测为准；
  `@strconv` 一条明确写成「当前 core 无入口」而不是「MoonBit 不支持」，避免泛化
- 影响文件：`../references/failure-index.md`、`../references/index.md`、`SKILL.md`
- 验证：moon `0.1.20260916` 下逐条最小复现（`enum Kind { A; B } derive(Eq)`、`let mut xs : Array[Int] = []`、
  `let s = "a\u{1F923}b"`、`moon ide doc "@strconv"`、`@bigint.BigInt::from_string("1234").to_int()`）

## 2026-09-10：补录工具链 / 运行时行为类坑位

- 现象：`.mbtx` 默认 target、`println` 缓冲、`@fs.write_file` 的 create 语义、`moon add` 不升级、`@http.Request.path` 带 query 这几类问题都不产出编译器报错原文，按报错搜搜不到；`App::mount` 的裸 id 要求只在 `rabbita-fullstack-notes.md` 历史里，用 playbook 时看不到
- 结论：在 `../references/failure-index.md` 新增「工具链与运行时行为（没有编译器报错可搜）」一节（8 条）；`mount` 的那条提升到 `../playbooks/rabbita-fullstack.md` 的常见失败表现；`Bytes::to_unchecked_string()` 作为已有 `to_bytes()` 条目的反方向补充，不另建条目
- 补充：错误码全集不另建索引——`moon explain --diagnostic` 不带参数即列出全部 warning 与非 warning 诊断，且来自本机编译器；在 `failure-index.md` 顶部补了这一条与可用的文档 URL
- 影响文件：`../references/failure-index.md`、`../references/index.md`、`../playbooks/rabbita-fullstack.md`
- 验证：moon `0.1.20260904` 下逐条最小复现；`moon explain --diagnostic [<id>]` 与 `--attribute` 实跑；错误码文档页 URL 实测 200/404

## 2026-09-07：导航层重构与 Mooncakes 发布经验

- 现象：总入口、补丁汇总、专项流程和更新日志混在一起，默认加载成本高；Mooncakes 发布经验需要可复用的独立流程
- 结论：`SKILL.md` 降为导航层；以任务和错误表现分流；Rabbita、Mooncakes 发布和维护流程分别进入 playbook / maintenance；旧补丁保留为兼容历史
- 影响文件：`../references/index.md`、`../references/failure-index.md`、`../references/mooncakes.md`、`../playbooks/`、`../maintenance/`
- 验证：`conglinyizhi/moondbus@0.1.1` 和 `conglinyizhi/moonsni@0.1.1` 已可由 `moon search` 查询

## 2026-09-07：nightly 坑位复测

- 现象：历史补丁包含默认 target、async trait 与 native 编译器的旧 nightly 结论，不能直接当作当前规则
- 结论：`@async/fs` 默认 target 的 `unimplemented` 陷阱、sync trait 调 async 后 impl 静默丢弃，均于 moon `0.1.20260904` 首次确认失效；core 文件 I/O 缺失仍成立；native C compiler 仅保留为环境前置条件
- 影响文件：`nightly-retest-20260904.md`、`../references/failure-index.md`、`../references/moon-ide-doc-gotcha.md`
- 验证：最小模块 `moon check`、当前 async 包 target 映射、当前编译器 `E4149`

## 既有历史入口

完整的旧补丁详情已归档为 `legacy-patches.md`；`../references/patches.md` 只保留兼容编号和新入口。新内容优先写入分层文件，不再继续扩大旧补丁汇总。
