# 旧补丁索引（兼容入口）

> 日常不要加载本文件。先读 `index.md`；本文件保留给旧链接、补丁编号和历史迁移。
> 当前规则以 `references/`、`playbooks/` 和 `maintenance/` 下的分层文件为准。

## 旧编号映射

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

## 新入口

- 官方 MoonBit 内容：`official-skills.md`
- Mooncakes 生态与个人包探查：`mooncakes.md`
- 按错误表现排查：`failure-index.md`
- Rabbita / moonback 网站：`../playbooks/rabbita-fullstack.md`
- Mooncakes 发布：`../playbooks/mooncakes-publish.md`
- 更新月井之春：`../maintenance/update-skill.md`
- 历史补丁详情：`patches.md`

## 生态包快记

- OpenAI：`tonyfettes/openai`
- 多 provider / Anthropic：`QuietlyChan/moonai`
- MCP：`colmugx/mcp`
- D-Bus：`conglinyizhi/moondbus@0.1.1`
- Linux SNI 托盘：`conglinyizhi/moonsni@0.1.1`
- Rabbita 开发工具：`moonbit-community/warren`

包版本、公开 API 和 target 不是静态事实。使用前走 `mooncakes.md` 的当前查询流程。
