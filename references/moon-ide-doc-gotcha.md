# `moon ide doc` 与 target：历史坑状态

## 已消失：`@async/fs` 在默认项目中只显示 `unimplemented`

**状态**：已于 **moon `0.1.20260904`** 首次确认消失。

旧 nightly 的经验是：`moon new` 默认 `wasm-gc`，`moon ide doc "@async/fs"` 只显示 `unimplemented`，因此容易误判文件 API 不存在。当前版本的复测结果不同：

```text
moon new probe
→ moon.mod: preferred_target = "wasm"
```

在分别设置 `preferred_target = "wasm"` 与 `preferred_target = "native"` 的最小模块中，以下代码均可通过 `moon check`：

```mbt
async fn main {
  ignore(@fs.exists("moon.mod"))
}
```

对应包 `moonbitlang/async@0.20.2/fs` 的当前 target 映射也将主要文件列为 `native` 与 `wasm`；`@fs.exists`、`@fs.read_file` 等公开 API 出现在生成的 native `.mbti` 中。

## 当前规则

- 不要再把“默认 `moon new` 项目下 `@async/fs` 只有 `unimplemented`”当成当前坑
- `moon ide doc` 仍会受**当前模块、已安装依赖、target、符号索引状态**影响；精确 API 以可编译的最小 import 和当前 `.mbti` / 包文档为准
- 如果 `moon ide doc` 本身报：

  ```text
  Fail to load core: .../core/_build/packages.json: No such file or directory
  ```

  这是本机 core 符号索引缺失，不能据此判定 API 或 target 行为

## 复测证据

```bash
moon version --all
# moon 0.1.20260904 (94521db 2026-09-04)

moon new probe
# 生成 preferred_target = "wasm"

# 在最小模块中加入 moonbitlang/async@0.20.2，
# 并在可执行包的 moon.pkg 导入：
# "moonbitlang/async" 和 "moonbitlang/async/fs" @fs
moon check
moon check --target native
```

`moon ide doc` 的具体输出仍应在目标项目中复验；不要把旧 issue 或历史输出当作当前事实。
