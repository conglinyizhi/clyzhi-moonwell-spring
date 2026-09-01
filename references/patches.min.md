# 补丁索引（精简版）

> 完整补丁详情见 `patches.md`（24 条目）。本文件供 Agent 日常快速索引，避免每次加载完整 patches.md。

## 补丁索引

| # | 条目 | 类型 |
|:--|:--|:--|
| 1 | 配置文件格式迁移（moon.mod / moon.pkg） | 格式变更 |
| 2 | moon.work 工作空间 | 新特性 |
| 3 | moon runwasm | 新命令 |
| 4 | declare 关键字（替代 #declaration_only） | 写法规范 |
| 5 | json_inspect() 写法 | 写法规范 |
| 6 | 属性完整列表（18 个） | 补充 |
| 7 | moon coverage 完整子命令 | 补充 |
| 8 | moon prove（Why3 验证） | 新命令 |
| 9 | moon explain | 补充 |
| 10 | moon fetch | 新命令 |
| 11 | moon run --profile | 补充 |
| 12 | .mbtx 脚本 | 新特性 |
| 13 | WASM Component Model | 新特性 |
| 14 | moon package --list | 补充 |
| 15 | moon check --output-json | 补充 |
| 16 | --unstable-feature / -Z | 补充 |
| 17 | .mbtx + @async/process 子进程调用 | 补充 |
| 18 | moon check --fmt / --explain / --patch-file | 补充 |
| 19 | moon run/test --build-only + stdin .mbtx + doc 废弃 | 补充 |
| 20 | Async / HTTP 服务默认栈（无独立后端框架） | 补充 |
| 21 | 数组模式至多一个 `..` + Show→Debug + catch/`<|` 优先级 | 写法规范 |
| 22 | Rabbita 全栈 SSR（SSR + moonback 后端；未入官方 skills） | 新特性 |
| 23 | MoonBit 语言/工具层实战坑（async trait impl 静默丢弃 / core 无文件 IO / MOON_CC / `--noproxy` / JWT base64） | 已知限制 |
| 24 | 生态包速查：LLM 底座（moonai / openai / pi-moonbit / mcp）+ D-Bus 桌面集成（moondbus / moonsni） | 生态包 |

---

## 官方技能对照

| 官方技能 | 主要问题 | 对应补丁 |
|:--|:--|:--|
| `moonbit-agent-guide` | 缺 moon.work / runwasm / .mbtx / Component Model / @async/process；缺 HTTP 服务脚手架与默认 target 陷阱 | 2、3、12、13、17、20、21 |
| `moonbit-c-binding` | Phase 1 用 `moon.mod.json`（已弃用）；`supported_targets` 数组写法 | 1 |
| `moonbit-orientation` | references 无 moon.work / runwasm；后端/HTTP 路由未点明 | 2、3、20 |
| `moonbit-spec-test-development` | `#declaration_only` → `declare`；`@json.inspect()` → `json_inspect()`；引用旧格式 | 4、5、1 |
| （全部官方技能） | 只讲语言与工具链，**不提第三方生态包**（LLM 底座 / 桌面集成 / 全栈） | 22、24 |

---

## 生态包速查（补丁24 详情）

> 均已 `moon add` 实际拉取 + 查阅 `.mbti` 验证。避免重复造轮子。

| 需求 | 包 | 一句话 |
|:--|:--|:--|
| 只调 OpenAI | `tonyfettes/openai` | 轻量（2734 行），30+ 参数全覆盖，`trait HttpClient` 可插拔 |
| OpenAI + Anthropic + 多 provider | `QuietlyChan/moonai` | 对标 Vercel AI SDK（12 万行）19 子包，含 MCP 与 pi harness（alpha）|
| AI coding agent / CLI | `eanzhao/pi-moonbit` | pi-mono 的 MoonBit 重写，带 `pimbt` CLI，7 个 provider |
| MCP server/client | `colmugx/mcp` | 类型安全 MCP SDK，STDIO/HTTP 双传输 |
| D-Bus 协议（纯 MoonBit） | `conglinyizhi/moondbus` | 无 GLib/GIO/libdbus，含可复用的服务端 `Server::serve` 循环 |
| Linux 托盘（KDE/SNI） | `conglinyizhi/moonsni` | 渐进式三子包；menu 子包**零 D-Bus 依赖**可单独用 |
| 全栈 SSR | `moonbit-community/rabbita` + `hackwaly/moonback` | 见补丁22 与 `rabbita-fullstack.md` |

---

## 官方信息来源

| 来源 | URL | 用途 |
|:--|:--|:--|
| moon 工具链 `--help` | 本地命令 | 命令/参数权威来源 |
| GitHub Releases | https://github.com/moonbitlang/moon/releases | 版本变更日志 |
| 官方文档 | https://docs.moonbitlang.com/ | 工具链/语言文档 |
| 官网 Blog | https://www.moonbitlang.com/blog/ | 特性预告 |
| 官网 Updates | https://www.moonbitlang.com/updates/ | 更新动态 |
| 官方 Skills | https://github.com/moonbitlang/skills | 技能上游变更 |
| moon Issues | https://github.com/moonbitlang/moon/issues | 已知问题/进行中功能 |
| Mooncakes 包文档 | https://mooncakes.io/docs/ | 包 API 文档（如 @async/process） |
| async 源码 | https://github.com/moonbitlang/async | async 库源码 |
