---
name: clyzhi-moonwell-spring
description: >
  MoonBit 官方技能热修复层。月井之春:补丁如泉水回春。必须与官方 moonbit-* 技能同时加载。
tracked_version: "moon 0.1.20260626 (305d2bc 2026-06-26)"
last_updated: "2026-07-19"
---

# MoonBit Hotfix Skills · 月井之春

> 官方 [moonbitlang/skills](https://github.com/moonbitlang/skills) 的热修复层。
> **必须与官方 moonbit-* 系列技能同时加载。** 本技能只记录官方技能尚未覆盖或已过时的内容，
> 不重复官方技能已有信息。内容冲突时以本技能为准。

**追踪版本**：moon 0.1.20260626 | **最后更新**：2026-07-19 | **仓库**：[conglinyizhi/clyzhi-moonwell-spring](https://github.com/conglinyizhi/clyzhi-moonwell-spring)

---

## 触发条件

以下关键词/场景触发本技能（与官方 moonbit-* 技能同时加载）：

- 使用 MoonBit 开发、编写 `.mbt` 代码
- 配置 `moon.mod`、`moon.pkg`、`moon.work`
- 执行 `moon check`、`moon build`、`moon test`、`moon run`、`moon fmt`、`moon new`
- 使用 `moon work`、`moon prove`、`moon runwasm`、`moon fetch`
- 涉及 C FFI 绑定、形式化验证、WASM 编译目标
- 用户要求「更新 moonbit-hotfix」或 moon 版本升级后

---

## 执行

本技能有两种执行模式：

### 模式 A：知识覆盖（默认，每次加载）

Agent 加载本技能后，以下补丁内容自动覆盖官方技能的对应条目。无需用户额外操作。

### 模式 B：版本更新（用户触发）

当用户说「更新 moonwell-spring」或 moon 版本变化时，执行更新工作流。完整步骤见：

→ **`references/update-workflow.md`**（5 Phase：版本检测 → 拉取变更 → 差异分析 → 增量检查 → 更新文档）

摘要：
```bash
moon version                          # Phase 1: 检测版本
moon --help && moon <cmd> --help      # Phase 2: 拉取变更
gh release list --repo moonbitlang/moon
# Phase 3: 按差异类型生成/更新补丁条目
# Phase 4: 如版本未变，仅检查官方 skills 仓库
# Phase 5: 更新 frontmatter + 补丁条目 + commit
```

---

## 补丁内容

见 **`references/patches.md`**。共 16 个补丁条目：

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

---

## 注意事项

- 本技能内容基于 `tracked_version` 验证。如果 `moon --help` 实际输出与本文档不一致，**以工具输出为准**，并触发更新流程
- `moon ide doc` 是 API 发现的首选方式，比文本搜索更准确
- moon.work 仍在迭代中（3 个 open issue），单模块仓库暂不需要
- `moon.mod.json` → `moon.mod` 迁移不可逆，`moon fmt` 后本地路径依赖会丢失——提前用 `moon.work` 替代
- 官方技能对照：

| 官方技能 | 主要问题 |
|:--|:--|
| `moonbit-agent-guide` | 缺 moon.work / runwasm / .mbtx / Component Model |
| `moonbit-c-binding` | Phase 1 用 `moon.mod.json`；`supported_targets` 数组写法 |
| `moonbit-orientation` | references 无 moon.work / runwasm |
| `moonbit-spec-test-development` | `#declaration_only` → `declare`；`@json.inspect()` → `json_inspect()`；引用旧格式 |

---

## 验证

以下命令确认本技能追踪内容与当前 moon 版本匹配。全部通过 = 技能新鲜。

```bash
# 1. 版本匹配
moon version | grep -q "0.1.20260626" && echo "✅ 版本匹配" || echo "❌ 版本不一致，需要更新"

# 2. 属性数量（≥18）
[ $(moon explain --attribute 2>&1 | grep -c '^  #') -ge 18 ] && echo "✅ 属性 ≥18" || echo "❌ 属性减少"

# 3. 命令数量（≥27）
[ $(moon --help 2>&1 | grep -cP '^\s{2}\w') -ge 27 ] && echo "✅ 命令 ≥27" || echo "❌ 命令减少"

# 4. 新格式生效
moon new --help 2>&1 | grep -q 'moon.mod' && echo "✅ 新格式生效" || echo "❌ 格式可能已变"

# 5. moon.work 存在
moon work --help 2>&1 | grep -q 'init' && echo "✅ moon.work 可用" || echo "❌ moon.work 状态变化"
```
