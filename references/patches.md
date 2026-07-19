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

### 十七、.mbtx + @async/fs + @async/process 子进程与文件 I/O

官方技能未提及 `.mbtx` 可通过 `@async` 系列包进行跨平台子进程调用和文件 I/O。两者可在同一个 `async fn main` 中共存。

mbtx 依赖声明：
```mbt
import {
  "moonbitlang/async@0.20.2",
  "moonbitlang/async@0.20.2/fs",
  "moonbitlang/async@0.20.2/process",
  "moonbitlang/core/string" @string,
}
```

子进程（`@async/process`）：
```mbt
@process.collect_stdout("moon", ["version"])     // → (exit_code, &@io.Data)
@process.collect_stderr("moon", ["check"])       // → (exit_code, &@io.Data)
@process.run("moon", ["build"])                  // → exit_code
```

文件 I/O（`@async/fs`）：
```mbt
@fs.read_file("moonwell.toml")                   // → &@io.Data
@fs.open("out.txt", mode=ReadOnly)               // → File (支持 read_all 等方法)
```

`&@io.Data` 统一接口：
```mbt
let data = @process.collect_stdout(...)  // 或 @fs.read_file(...)
let text = data.text()                    // → String (UTF-8 解码)
let json = data.json()                    // → Json (解析 JSON)
let bytes = data.binary()                 // → Bytes (原始二进制)
```

来源：[moonbitlang/async/src/process](https://github.com/moonbitlang/async/tree/main/src/process)、[moonbitlang/async/src/fs](https://github.com/moonbitlang/async/tree/main/src/fs)

