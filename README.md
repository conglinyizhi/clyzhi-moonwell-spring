# clyzhi-moonwell-spring

> 月井之春 — MoonBit 官方技能热修复层。补丁如泉水回春。

追踪 moon 工具链最新特性，补充 [moonbitlang/skills](https://github.com/moonbitlang/skills) 尚未覆盖的内容。由 AI Agent 自动加载，为 MoonBit 开发提供准确的上下文。

**追踪版本**：moon 0.1.20260713（`moon run scripts/verify.mbtx --target native` 可核验）

---

## 使用方式

本技能**推荐与官方 moonbit-\* 技能同时加载**。Agent 加载后在 MoonBit 开发场景自动生效。用户说「更新 moonwell-spring」时沿 `references/update-workflow.md` 对月井之春进行滚动更新。

## 补丁覆盖

17 条补丁，涵盖：

- 配置文件格式迁移（moon.mod / moon.pkg）
- moon.work 工作空间、moon runwasm、moon prove、moon fetch
- declare 关键字、json_inspect() 写法规范
- 属性完整列表、coverage 子命令、moon explain
- moon run --profile、moon package --list、moon check --output-json
- .mbtx + @async/fs + @async/process 跨平台能力
- --unstable-feature / -Z、WASM Component Model

## 官方技能对照

| 官方技能 | 需关注补丁 |
|:--|:--|
| `moonbit-agent-guide` | 二、三、十二、十三、十七 |
| `moonbit-c-binding` | 一 |
| `moonbit-orientation` | 二、三 |
| `moonbit-spec-test-development` | 四、五、一 |

## 许可

Apache-2.0
