---
name: moonbit-hotfix-skills
description: >
  MoonBit 官方技能热修复层。**必须与官方 moonbit-* 技能同时加载，不可单独使用。**
  追踪最新 moon 工具链特性，补充官方技能尚未覆盖的内容。
  触发关键词：MoonBit、moonbit、.mbt、moon.mod、moon.pkg、moon check、moon build、
  moon test、moon run、moon work、moon.work、moon prove、moon fmt、moon new。
tracked_version: "moon 0.1.20260626 (305d2bc 2026-06-26)"
last_updated: "2026-07-19"
---

# MoonBit Hotfix Skills

## ⚠️ 加载要求（重要）

本技能是 [moonbitlang/skills](https://github.com/moonbitlang/skills) 官方技能的**热修复层**。

- **必须同时加载**：本技能 + 官方 moonbit-* 系列技能
- **优先级**：本技能内容与官方技能冲突时，以本技能为准
- **不替代**：本技能只记录差异和补充，不重复官方技能已有内容
- **版本锚点**：本技能内容基于下方 `tracked_version` 验证；如果工具实际输出与本文档不一致，以工具输出为准，并触发更新流程

---

## 📋 版本追踪

| 项目 | 值 |
|:--|:--|
| 追踪的 moon 版本 | `moon 0.1.20260626 (305d2bc 2026-06-26)` |
| 本技能最后更新 | `2026-07-19` |
| 官方技能仓库 | [moonbitlang/skills](https://github.com/moonbitlang/skills) |
| 本技能仓库 | [conglinyizhi/moonbit-hotfix-skills](https://github.com/conglinyizhi/moonbit-hotfix-skills) |

---

## 🔄 自动更新工作流

当用户要求「更新 moonbit-hotfix-skills」或检测到 moon 版本变更时，按以下步骤执行：

### Phase 1：版本检测

```bash
# 1. 获取当前安装的 moon 版本
moon version

# 2. 与本文档 tracked_version 对比。如果版本号一致，跳到 Phase 4（增量检查）；
#    如果版本号变化，进入 Phase 2。
```

### Phase 2：拉取变更信息

按以下顺序拉取，每条信息源拉取后立即记录关键变更点：

```bash
# A. moon --help 全量对比
#    遍历每个子命令的 --help，与本技能已记录的命令列表对比
moon --help
for cmd in $(moon --help | grep -oP '^\s{2}\w+'); do
  moon $cmd --help 2>/dev/null
done

# B. GitHub Release 信息
gh release list --repo moonbitlang/moon --limit 5
# 对每个比 tracked_version 新的 release：
gh release view <tag> --repo moonbitlang/moon

# C. 官方文档更新
#    检查以下页面是否有新增章节或变更：
curl -sL "https://docs.moonbitlang.com/en/latest/toolchain/moon/" 2>/dev/null

# D. 官网 Blog（发行说明、特性预告）
curl -sL "https://www.moonbitlang.com/blog/" 2>/dev/null
curl -sL "https://www.moonbitlang.com/updates/" 2>/dev/null

# E. 官方 skills 仓库更新
gh issue list --repo moonbitlang/skills --state all --limit 10
gh pr list --repo moonbitlang/skills --state all --limit 10

# F. moon 仓库近期 issue（了解已知问题/正在进行的功能）
gh issue list --repo moonbitlang/moon --limit 20
```

### Phase 3：差异分析与补丁生成

对每个变更来源，判断是否需要新增/修改补丁条目：

```
变更类型               →  处理方式
─────────────────────────────────────────────
新命令/子命令          →  新增补丁条目（如官方技能未覆盖）
命令参数变化           →  更新对应条目
配置文件格式变化        →  更新对应条目 + 标注迁移方式
属性/诊断码增减         →  更新属性列表/诊断列表
官方技能已覆盖的新特性   →  从本技能删除（不再需要补丁）
已知 bug/限制           →  新增 ⚠️ 警告条目
弃用/移除              →  新增条目 + 标注版本和替代方案
```

### Phase 4：增量检查（版本未变时）

即使版本号相同，也做轻量检查：

```bash
# 检查官方 skills 仓库是否有新 issue/PR
gh issue list --repo moonbitlang/skills --state all --limit 5
gh pr list --repo moonbitlang/skills --state all --limit 5

# 快速检查 moon --help 是否有新增子命令（diff 对比）
moon --help
```

### Phase 5：更新文档

1. 更新 frontmatter 中的 `tracked_version` 和 `last_updated`
2. 更新「版本追踪」表格
3. 新增/修改/删除对应补丁条目
4. 在「上次更新执行记录」中追加本次更新摘要
5. 提交 commit：`hotfix: sync to moon <version> (<date>)`

---

## 📡 信息源列表

| 信息源 | URL | 用途 |
|:--|:--|:--|
| moon 工具链 `--help` | 本地命令 | 命令/参数权威来源 |
| GitHub Releases | `https://github.com/moonbitlang/moon/releases` | 版本变更日志 |
| 官方文档 | `https://docs.moonbitlang.com/en/latest/toolchain/moon/` | 工具链文档 |
| 官网 Blog | `https://www.moonbitlang.com/blog/` | 特性预告、发行说明 |
| 官网 Updates | `https://www.moonbitlang.com/updates/` | 更新动态 |
| 官方 Skills 仓库 | `https://github.com/moonbitlang/skills` | 技能上游变更 |
| moon 仓库 Issues | `https://github.com/moonbitlang/moon/issues` | 已知问题/进行中功能 |
| MoonBit 文档站 | `https://docs.moonbitlang.com/` | 完整文档索引 |

---

## 📝 上次更新执行记录

**日期**：2026-07-19
**从版本**：无（初始创建）
**到版本**：moon 0.1.20260626

**执行步骤**：
1. `moon version` → 确认版本 0.1.20260626
2. `moon --help` → 遍历所有子命令，列出完整命令列表（27 个子命令）
3. 逐个读取 8 个官方 moonbit-* 技能文件
4. 逐项对比工具实际输出 vs 技能文档记录
5. 运行 `moon explain --attribute` → 获取 18 个属性完整列表
6. 运行 `moon explain --diagnostic` → 确认诊断码列表
7. 运行 `moon new` → 确认默认创建 `moon.mod`(新) 而非 `moon.mod.json`(旧)
8. 验证 `declare` 关键字和 `#declaration_only` 属性均有效
9. 搜索官方仓库 issue/PR → 确认无他人提出相同问题
10. 生成 16 个补丁条目

**对比维度**：
- 命令覆盖：moon --help 27 个命令 vs 技能文档提及 ~20 个 → 遗漏 `runwasm`、`work`、`prove`、`generate-build-matrix`(legacy)、`fetch` 等
- 配置格式：官方技能仍引用 `moon.mod.json`/`moon.pkg.json`，实际已迁移到 `moon.mod`/`moon.pkg`
- 属性覆盖：18 个属性 vs 技能涉及 ~10 个
- 子命令覆盖：`coverage` 有 3 个子命令，技能只提了 `analyze`
- 写法规范：`declare` vs `#declaration_only`，`json_inspect()` vs `@json.inspect()`

---

## 补丁内容

---

### 一、配置文件格式迁移（官方技能已过时）

#### moon.mod（新） vs moon.mod.json（旧，已弃用）

`moon.mod.json` 在 v0.10.4 已弃用，**下个版本移除**。`moon new` 现在直接创建 `moon.mod`。

```toml
# moon.mod（新格式，TOML-like）
name = "username/hello"
version = "0.1.0"
readme = "README.mbt.md"
repository = ""
license = "Apache-2.0"
keywords = []
preferred_target = "native"
description = ""

import {
  "moonbitlang/x@0.4.6",
}
```

迁移方式：运行 `moon fmt`（会自动改写 `moon.mod.json` → `moon.mod`）。

> ⚠️ **注意**：新 `moon.mod` 格式**不再支持本地路径依赖**（`path` 字段）。
> 跨模块本地依赖必须使用 `moon.work`（见下文）。

#### moon.pkg（新） vs moon.pkg.json（旧）

同样，新格式是 `moon.pkg`（TOML-like），旧格式 `moon.pkg.json` 已弃用。

```toml
# moon.pkg（新格式）
import {
  "username/hello/liba",
  "moonbitlang/x/encoding" @enc,
}
import {
  "username/hello/test_helpers",
} for "test"
import {
  "username/hello/internal_test_helpers",
} for "wbtest"

supported_targets = "native"
options(
  "is-main": true,
)
```

关于 `supported_targets` 的写法：
- ✅ `supported_targets = "native"`（字符串，推荐）
- ⚠️ `supported_targets: ["native"]`（数组，可能阻止下游包在其它 target 构建）
- ✅ 用 `targets` 字段 gate 单个文件：`"ffi.mbt": ["native"]`

---

### 二、moon.work 工作空间（官方技能完全未覆盖）

多个模块放在同一仓库时，用 `moon.work` 统一管理。

```bash
moon work init mod_a mod_b    # 初始化，注册成员
moon work use mod_c           # 添加新成员
moon work sync                # 同步成员间依赖版本
```

生成的 `moon.work`：
```toml
members = [
  "./mod_a",
  "./mod_b",
]
```

工作空间根目录下，以下命令作用于所有成员：
```bash
moon check --target all
moon test --target all
moon info
moon clean
```

模块专有命令（如 `publish`）需进入子目录：`moon -C mod_a publish`。

> ⚠️ **已知限制**（moon 仓库 open issue）：
> - [#1903](https://github.com/moonbitlang/moon/issues/1903) 无法排除工作空间成员
> - [#1788](https://github.com/moonbitlang/moon/issues/1788) `preferred_target` 行为待明确
> - [#1859](https://github.com/moonbitlang/moon/issues/1859) moonc syncheck 不支持 moon.work
>
> 单模块仓库不需要创建 `moon.work`。

---

### 三、moon runwasm（官方技能未覆盖）

```bash
# 本地包
moon runwasm main
moon runwasm ./main

# Mooncakes 坐标（缓存到 $MOON_HOME/registry/cache/assets）
moon runwasm moonbitlang/parser/cmd/moonfmt@0.3.3
```

---

### 四、声明规范：declare 关键字

官方 `moonbit-spec-test-development` 技能中使用了 `#declaration_only` 属性。两种写法目前都有效，但 **`declare` 关键字是推荐写法**。

```mbt
// ✅ 推荐
///|
declare pub type Yaml

///|
declare pub fn parse_yaml(s : String) -> Yaml raise
```

---

### 五、json_inspect() 写法

官方 `moonbit-spec-test-development` 技能中使用了 `@json.inspect(...)`。

**正确写法是 `json_inspect()`**（prelude 导入，无需包前缀）：

```mbt
// ✅ 正确
json_inspect(value, content={ "key": "value" })

// ❌ 避免
@json.inspect(value)
```

---

### 六、属性完整列表

`moon explain --attribute` 输出（moon 0.1.20260626）：

| 属性 | 用途 |
|:--|:--|
| `#alert` | 编译时告警 |
| `#alias` | 符号别名（向后兼容迁移） |
| `#as_free_fn` | 方法暴露为独立函数（迁移辅助） |
| `#borrow`, `#owned` | C FFI 所有权标注 |
| `#callsite` | 调用位置信息 |
| `#cfg` | 条件编译 |
| `#coverage.skip` | 跳过覆盖率统计 |
| `#deprecated` | 弃用标记 |
| `#doc` | 文档元数据 |
| `#external` | C 指针，C 端管理生命周期 |
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
moon coverage analyze   # 插桩运行测试并收集覆盖率
moon coverage report    # 生成覆盖率报告
moon coverage clean     # 清理覆盖率产物
```

官方技能只提到了 `analyze`。

---

### 八、moon prove（Why3 形式化验证）

```bash
moon prove [PATH]                    # 验证当前包
moon prove --why3-config <PATH>      # 使用自定义 Why3 配置
```

Why3 运行时已内置。与 `moonbit-proof` 技能配合使用。

---

### 九、moon explain

```bash
moon explain --diagnostic            # 列出所有诊断码
moon explain --diagnostic 31         # 按编号解释
moon explain --diagnostic unused_optional_argument  # 按助记名解释
moon explain --attribute             # 列出所有属性
moon explain --attribute deprecated  # 解释特定属性
```

---

### 十、moon fetch（下载第三方源码）

```bash
moon fetch moonbitlang/async@0.18.1   # 下载到 .repos/（不修改 moon.mod）
```

`.gitignore` 加 `.repos/`。要真正添加依赖用 `moon add`。

---

### 十一、moon run --profile（性能采样）

```bash
moon run --profile --target native --release cmd/<main>
```

生成 `profile.json` 和 `.trace`。自耗时（self-time）→ 哪个函数烧 CPU；总耗时（inclusive-time）→ 哪个调用子树是瓶颈。

---

### 十二、.mbtx 脚本

文档中有 "Running .mbtx Scripts" 章节。`.mbtx` 是可执行的 MoonBit 脚本文件。官方技能未涉及。

---

### 十三、WASM Component Model

MoonBit 已支持 WASM Component Model，可编译为 WASM 组件与其他语言互操作。官方技能未涉及。

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

可配合 `moon run -e` 做复杂后处理。

---

### 十六、--unstable-feature / -Z 标志

```bash
moon -Z <FEATURE_NAME> check    # 启用不稳定特性
```

当前启用的 feature flags：`rr_moon_mod`、`rr_moon_pkg`。

---

## 🔧 官方技能对照表

| 官方技能 | 已知问题 | 本技能补丁条目 |
|:--|:--|:--|
| `moonbit-agent-guide` | 已较全面，缺 moon.work/runwasm/.mbtx/Component Model | 二、三、八、九、十、十一、十二、十三、十四、十五、十六 |
| `moonbit-c-binding` | Phase 1 示例用 `moon.mod.json`（已弃用）；`supported_targets` 数组写法 | 一 |
| `moonbit-extract-spec-test` | 基本正确 | — |
| `moonbit-orientation` | references 中无 moon.work/runwasm 相关内容 | 二、三 |
| `moonbit-proof` | 覆盖完整 | — |
| `moonbit-refactoring` | 基本正确 | — |
| `moonbit-spec-test-development` | `#declaration_only` 应统一为 `declare`；`@json.inspect()` 应改为 `json_inspect()`；引用 `moon.mod.json` | 一、四、五 |
