# 补丁索引（精简版）

> 完整补丁详情见 `patches.md`。本文件供 Agent 日常快速索引，避免每次加载完整 patches.md。

## 补丁索引

| # | 条目 | 类型 |
|:--|:--|:--|
| 一 | 配置文件格式迁移（moon.mod / moon.pkg） | 格式变更 |
| 二 | moon.work 工作空间 | 新特性 |
| 三 | moon runwasm | 新命令 |
| 四 | declare 关键字（替代 #declaration_only） | 写法规范 |
| 五 | json_inspect() 写法 | 写法规范 |
| 六 | 属性完整列表（18 个） | 补充 |
| 七 | moon coverage 完整子命令 | 补充 |
| 八 | moon prove（Why3 验证） | 新命令 |
| 九 | moon explain | 补充 |
| 十 | moon fetch | 新命令 |
| 十一 | moon run --profile | 补充 |
| 十二 | .mbtx 脚本 | 新特性 |
| 十三 | WASM Component Model | 新特性 |
| 十四 | moon package --list | 补充 |
| 十五 | moon check --output-json | 补充 |
| 十六 | --unstable-feature / -Z | 补充 |
| 十七 | .mbtx + @async/process 子进程调用 | 补充 |

---

## 官方技能对照

| 官方技能 | 主要问题 | 对应补丁 |
|:--|:--|:--|
| `moonbit-agent-guide` | 缺 moon.work / runwasm / .mbtx / Component Model / @async/process | 二、三、十二、十三、十七 |
| `moonbit-c-binding` | Phase 1 用 `moon.mod.json`（已弃用）；`supported_targets` 数组写法 | 一 |
| `moonbit-orientation` | references 无 moon.work / runwasm | 二、三 |
| `moonbit-spec-test-development` | `#declaration_only` → `declare`；`@json.inspect()` → `json_inspect()`；引用旧格式 | 四、五、一 |

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
