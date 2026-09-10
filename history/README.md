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
