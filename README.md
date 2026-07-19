# clyzhi-moonwell-spring

> 月井之春 — MoonBit 官方技能热修复层。补丁如泉水回春。

追踪 moon 工具链最新特性，补充 [moonbitlang/skills](https://github.com/moonbitlang/skills) 尚未覆盖的内容。由 AI Agent 自动加载，为 MoonBit 开发提供准确的上下文。

**追踪版本**：moon 0.1.20260713 | **最后更新**：2026-07-19

---

## 快速开始

```bash
git clone git@github.com:conglinyizhi/clyzhi-moonwell-spring.git

# 将技能目录链接到你的 pi/agent 技能路径
ln -sf $(pwd)/clyzhi-moonwell-spring ~/.pi/agent/skills/clyzhi-moonwell-spring

# 验证当前环境是否匹配
cd clyzhi-moonwell-spring && moon run scripts/verify.mbtx --target native
```

## 目录结构

```
├── SKILL.md                         # 骨架：触发条件、执行、注意事项、验证
├── moonwell.toml                    # 单一权威配置（版本号、阈值等）
├── scripts/
│   └── verify.mbtx                  # 跨平台验证脚本（mbtx + @async/process）
└── references/
    ├── patches.md                   # 完整补丁详情（17 条）
    ├── patches.min.md               # 精简索引 + 官方技能对照 + 信息来源
    ├── update-workflow.md           # 自动更新流程（5 Phase）
    └── moon-ide-doc-gotcha.md       # moon ide doc 踩坑记录
```

## 使用方式

本技能**必须与官方 moonbit-\* 技能同时加载**。Agent 加载后自动生效：

- 日常加载 `patches.min.md`（精简索引）获取补丁覆盖范围
- 关键词命中时按条目编号查阅 `patches.md` 获取详情
- 用户说「更新 moonwell-spring」时执行 `update-workflow.md` 流程

### 版本更新

```
5 Phase：版本检测 → 拉取 GitHub Release / Blog / 文档 / 官方 skills
       → 差异分析 → 增量检查 → 更新本文档
```

详见 `references/update-workflow.md`。

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
