---
name: clyzhi-moonwell-spring
description: >
  MoonBit 官方技能热修复层。月井之春——补丁如泉水回春。必须与官方 moonbit-* 技能同时加载。
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

---

### 一、配置文件格式迁移

`moon.mod.json` 在 v0.10.4 已弃用。`moon new` 现在创建 `moon.mod`（TOML-like）。迁移：运行 `moon fmt`。

```toml
# moon.mod（新格式）
name = "username/hello"
version = "0.1.0"
preferred_target = "native"
import { "moonbitlang/x@0.4.6", }
```

> ⚠️ 新 `moon.mod` 不再支持本地路径依赖（`path` 字段）。跨模块本地依赖必须使用 `moon.work`。

`moon.pkg`（新）同理，替代 `moon.pkg.json`：

```toml
# moon.pkg（新格式）
import { "username/hello/liba", }
import { "username/hello/test_helpers", } for "test"
supported_targets = "native"
options("is-main": true,)
```

关于 `supported_targets`：✅ `supported_targets = "native"`（字符串，推荐），⚠️ 数组写法可能阻止下游包在其他 target 构建。

---

### 二、moon.work 工作空间

多个模块同仓库时统一管理。

```bash
moon work init mod_a mod_b    # 初始化 + 注册成员
moon work use mod_c           # 添加新成员
moon work sync                # 同步成员间依赖版本
```

生成的 `moon.work`：
```toml
members = ["./mod_a", "./mod_b"]
```

工作空间根目录下 `moon check/test/info/clean` 作用于所有成员。模块专有命令（如 `publish`）需 `moon -C mod_a publish`。

> ⚠️ 已知限制：无法排除成员 ([#1903](https://github.com/moonbitlang/moon/issues/1903))、preferred_target 行为待明确 ([#1788](https://github.com/moonbitlang/moon/issues/1788))、moonc syncheck 不支持 ([#1859](https://github.com/moonbitlang/moon/issues/1859))。

---

### 三、moon runwasm

```bash
moon runwasm main
moon runwasm moonbitlang/parser/cmd/moonfmt@0.3.3   # Mooncakes 坐标
```

---

### 四、declare 关键字

官方 `moonbit-spec-test-development` 使用 `#declaration_only`。两种写法都有效，但 **`declare` 是推荐写法**。

```mbt
///|
declare pub type Yaml
///|
declare pub fn parse_yaml(s : String) -> Yaml raise
```

---

### 五、json_inspect() 写法

官方技能使用 `@json.inspect(...)`。**正确写法是 `json_inspect()`**（prelude 导入，无需包前缀）：

```mbt
json_inspect(value, content={ "key": "value" })   // ✅
```

---

### 六、属性完整列表

`moon explain --attribute` 输出（moon 0.1.20260626）：

| 属性 | 用途 |
|:--|:--|
| `#alert` | 编译时告警 |
| `#alias` | 符号别名 |
| `#as_free_fn` | 方法暴露为独立函数 |
| `#borrow`, `#owned` | C FFI 所有权标注 |
| `#callsite` | 调用位置信息 |
| `#cfg` | 条件编译 |
| `#coverage.skip` | 跳过覆盖率 |
| `#deprecated` | 弃用标记 |
| `#doc` | 文档元数据 |
| `#external` | C 指针（C 端管理生命周期） |
| `#inline` | 内联提示 |
| `#internal` | 内部 API |
| `#label_migration` | 标签迁移辅助 |
| `#module` | 模块级标注 |
| `#must_implement_one` | trait 方法互斥实现约束 |
| `#skip` | 跳过检查 |
| `#visibility` | 可见性控制 |
| `#warnings` | 警告控制 |

用 `moon explain --attribute <NAME>` 查看详情。

---

### 七、moon coverage 完整子命令

```bash
moon coverage analyze   # 插桩 + 收集覆盖率
moon coverage report    # 生成报告
moon coverage clean     # 清理产物
```

官方技能只提到 `analyze`。

---

### 八、moon prove（Why3 验证）

```bash
moon prove [PATH]
moon prove --why3-config <PATH>
```

Why3 运行时已内置。配合 `moonbit-proof` 技能使用。

---

### 九、moon explain

```bash
moon explain --diagnostic                  # 列出所有诊断码
moon explain --diagnostic 31               # 按编号
moon explain --diagnostic unused_optional_argument  # 按助记名
moon explain --attribute                   # 列出所有属性
moon explain --attribute deprecated        # 解释特定属性
```

---

### 十、moon fetch

```bash
moon fetch moonbitlang/async@0.18.1   # 下载到 .repos/（不修改 moon.mod）
```

`.gitignore` 加 `.repos/`。添加依赖用 `moon add`。

---

### 十一、moon run --profile

```bash
moon run --profile --target native --release cmd/<main>
```

生成 `profile.json` + `.trace`。self-time → 哪个函数烧 CPU，inclusive-time → 哪个调用子树是瓶颈。

---

### 十二、.mbtx 脚本

文档有 "Running .mbtx Scripts" 章节。`.mbtx` 是可执行 MoonBit 脚本。官方技能未涉及。

---

### 十三、WASM Component Model

MoonBit 支持 WASM Component Model，可编译为 WASM 组件互操作。官方技能未涉及。

---

### 十四、moon package --list

```bash
moon package --list   # 验证发布文件列表
```

---

### 十五、moon check --output-json

```bash
moon check --output-json 2>&1 | jq -R 'fromjson? | select(.message | contains("unused"))'
```

---

### 十六、--unstable-feature / -Z

```bash
moon -Z <FEATURE_NAME> check   # 启用不稳定特性
```

当前启用的 feature flags：`rr_moon_mod`、`rr_moon_pkg`。

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
