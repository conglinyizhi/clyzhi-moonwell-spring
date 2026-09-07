# 旧补丁汇总（历史兼容）

> 本文件不再是月井之春的默认知识入口。它保留历史补丁编号和迁移线索，避免旧链接失效。
> 日常从 `index.md` 开始；当前规则在 `references/`、`playbooks/` 和 `maintenance/`；长篇历史在 `history/`。

## 旧补丁编号

- 1：配置文件格式迁移（moon.mod / moon.pkg）
- 2：moon.work 工作空间
- 3：moon runwasm
- 4：declare 关键字
- 5：json_inspect() 写法
- 6：属性完整列表
- 7：moon coverage 子命令
- 8：moon prove
- 9：moon explain
- 10：moon fetch
- 11：moon run --profile
- 12：.mbtx 脚本
- 13：WASM Component Model
- 14：moon package --list
- 15：moon check --output-json
- 16：--unstable-feature / -Z
- 17：.mbtx + @async/process 子进程调用
- 18：moon check --fmt / --explain / --patch-file
- 19：moon run/test --build-only 与 stdin .mbtx
- 20：Async / HTTP 服务默认栈
- 21：数组模式、Show / Debug、catch 优先级
- 22：Rabbita 全栈 SSR
- 23：MoonBit 语言 / 工具层实战坑
- 24：生态包速查：LLM 与 D-Bus 桌面集成
- 25：Mooncakes 发布、打包复检、版本顺序与依赖索引传播延迟

## 当前替代入口

- 官方工具链、语言、FFI、证明和规格：`official-skills.md`
- 包搜索、版本、target 和个人包调查：`mooncakes.md`
- 错误表现排查：`failure-index.md`
- Rabbita / moonback 从零开发：`../playbooks/rabbita-fullstack.md`
- Mooncakes 发布：`../playbooks/mooncakes-publish.md`
- 更新本 skill：`../maintenance/update-skill.md`

## 历史材料

- Rabbita 全栈长篇实战笔记：`../history/rabbita-fullstack-notes.md`
- 更新记录：`../history/README.md`

保留旧编号是为了支持已有引用；新增内容不要继续追加到本文件。
