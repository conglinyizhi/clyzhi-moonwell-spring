---
name: moonbit-hotfix-skills
description: >
  MoonBit 官方技能热修复层。当使用 MoonBit 开发、编写 MoonBit 代码、配置 moon 工具链时，
  请在加载官方 moonbit-* 技能的同时加载本技能。本技能追踪 moon 0.1.20260626+ 的最新特性和
  官方技能尚未覆盖的内容，包括 moon.work 工作空间、moon.mod/moon.pkg 新格式迁移、
  moon runwasm、declare 关键字规范、moon explain、moon prove、moon coverage 子命令、
  .mbtx 脚本、WASM Component Model 等。
  触发关键词：MoonBit、moonbit、.mbt、moon.mod、moon.pkg、moon check、moon build、
  moon test、moon run、moon work、moon.work、moon prove、moon fmt、moon new。
---

# MoonBit Hotfix Skills

> **定位**：官方 [moonbitlang/skills](https://github.com/moonbitlang/skills) 的热修复层。
> 本技能记录官方技能尚未覆盖或已过时的内容。加载策略：**本技能 + 官方 moonbit-* 技能同时加载**，本技能的内容优先。

**最后同步**：moon 0.1.20260626 (305d2bc 2026-06-26)

---

## 一、配置文件格式迁移（官方技能已过时）

### moon.mod（新） vs moon.mod.json（旧，已弃用）

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

### moon.pkg（新） vs moon.pkg.json（旧）

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

## 二、moon.work 工作空间（官方技能完全未覆盖）

多个模块放在同一仓库时，用 `moon.work` 统一管理。

```bash
# 初始化工作空间
moon work init mod_a mod_b

# 添加新成员
moon work use mod_c

# 同步成员间依赖版本
moon work sync
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

> ⚠️ `moon.work` 本身还在迭代中（已知限制：无法排除成员、`preferred_target` 行为待明确、moonc syncheck 不支持）。
> 如果暂时只有单模块仓库，不需要创建 `moon.work`。

---

## 三、moon runwasm（官方技能未覆盖）

运行本地 wasm 包或从 Mooncakes 获取预编译 wasm 二进制：

```bash
# 本地包
moon runwasm main
moon runwasm ./main

# Mooncakes 坐标（自动缓存到 $MOON_HOME/registry/cache/assets）
moon runwasm moonbitlang/parser/cmd/moonfmt@0.3.3
moon runwasm moonbitlang/parser@0.3.3/cmd/moonfmt
```

---

## 四、声明与规范：declare vs #declaration_only

`moonbit-spec-test-development` 技能中使用了 `#declaration_only` 属性。两种写法目前都有效（都产生 E0068 warning），但 **`declare` 关键字是推荐写法**，在 `moonbit-agent-guide` 和 `moonbit-extract-spec-test` 中一致使用。

```mbt
// ✅ 推荐：declare 关键字
///|
declare pub type Yaml

///|
declare pub fn parse_yaml(s : String) -> Yaml raise

///|
declare pub impl Eq for Yaml
```

---

## 五、json_inspect() vs @json.inspect()

`moonbit-spec-test-development` 技能中使用了 `@json.inspect(...)`。

**正确写法是 `json_inspect()`**（prelude 导入，无需包前缀）：

```mbt
// ✅ 正确
json_inspect(value, content={ "key": "value" })

// ❌ 避免
@json.inspect(value)
```

---

## 六、属性完整列表（moon explain --attribute）

官方技能中只涉及部分属性。当前 moon 0.1.20260626 支持的完整属性：

| 属性 | 用途 |
|:--|:--|
| `#alert` | 编译时告警 |
| `#alias` | 符号别名（向后兼容迁移） |
| `#as_free_fn` | 将方法暴露为独立函数（迁移辅助） |
| `#borrow`, `#owned` | C FFI 所有权标注 |
| `#callsite` | 调用位置信息 |
| `#cfg` | 条件编译 |
| `#coverage.skip` | 跳过覆盖率统计 |
| `#deprecated` | 弃用标记 |
| `#doc` | 文档元数据 |
| `#external` | C 指针，C 端管理生命周期 |
| `#inline` | 内联提示 |
| `#internal` | 内部 API（非 pub） |
| `#label_migration` | 标签迁移辅助 |
| `#module` | 模块级标注 |
| `#must_implement_one` | trait 方法互斥实现约束 |
| `#skip` | 跳过某检查 |
| `#visibility` | 可见性控制 |
| `#warnings` | 警告控制 |

用 `moon explain --attribute <NAME>` 查看具体属性的详细说明。

---

## 七、moon coverage 完整子命令

官方技能只提到 `moon coverage analyze`。实际有三个子命令：

```bash
moon coverage analyze   # 插桩运行测试并收集覆盖率
moon coverage report    # 生成覆盖率报告
moon coverage clean     # 清理覆盖率产物
```

---

## 八、moon prove（Why3 形式化验证）

```bash
moon prove [PATH]                    # 验证当前包
moon prove --why3-config <PATH>      # 使用自定义 Why3 配置
```

与 `moonbit-proof` 技能配合使用。Why3 运行时已内置捆绑，无需额外安装。

---

## 九、moon explain（官方技能已覆盖，此处补充细节）

```bash
moon explain --diagnostic            # 列出所有诊断码
moon explain --diagnostic 31         # 解释 warning 31
moon explain --diagnostic unused_optional_argument  # 按助记名解释
moon explain --attribute             # 列出所有属性
moon explain --attribute deprecated  # 解释特定属性
```

---

## 十、moon fetch（下载第三方源码，不添加依赖）

```bash
moon fetch moonbitlang/async@0.18.1   # 下载到 .repos/ 供离线阅读
```

不会修改 `moon.mod`。用 `moon add` 才能真正添加依赖。记得 `.gitignore` 加 `.repos/`。

---

## 十一、moon run --profile（性能采样）

```bash
moon run --profile --target native --release cmd/<main>
```

生成 `profile.json` 和 `.trace`（可用 Instruments 打开）。需要 macOS + 完整 Xcode。自耗时（self-time）看哪个函数烧 CPU，总耗时（inclusive-time）看哪个调用子树是瓶颈。

---

## 十二、.mbtx 脚本（官方技能未覆盖）

文档中有 "Running .mbtx Scripts" 章节。`.mbtx` 是可执行的 MoonBit 脚本文件。

---

## 十三、WASM Component Model（官方技能未覆盖）

MoonBit 已支持 [WASM Component Model](https://docs.moonbitlang.com/en/latest/toolchain/moon/component-model.html)，可将 MoonBit 编译为 WASM 组件，与其他语言编写的组件互操作。

---

## 十四、moon package --list（官方技能未覆盖）

验证打包结果：

```bash
moon package --list   # 列出将要发布的文件
```

---

## 十五、moon check --output-json（管道后处理）

```bash
moon check --output-json 2>&1 | jq -R 'fromjson? | select(.message | contains("unused"))'
```

可配合 `moon run -e` 做更复杂的后处理。

---

## 十六、--unstable-feature / -Z 标志

```bash
moon -Z <FEATURE_NAME> check    # 启用不稳定特性
```

当前启用的 feature flags：`rr_moon_mod`、`rr_moon_pkg`（来自 `moon version` 输出）。

---

## 使用约定

1. 涉及 MoonBit 开发时，**同时加载本技能和官方 moonbit-* 技能**
2. 本技能内容与官方技能冲突时，**以本技能为准**
3. 如果本技能内容与 `moon --help` / `moon explain` 实际输出不一致，**以工具实际输出为准**，并更新本技能
4. `moon ide doc` 是 API 发现的首选方式，比文本搜索更准确
