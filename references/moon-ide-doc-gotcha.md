# moon ide doc 踩坑记录

## `@async/fs` 在 wasm-gc 目标下只显示 `unimplemented`

**现象**：`moon ide doc "@async/fs"` 只返回：

```
pub let unimplemented : Unit
```

**根因**：**不是 bug，是目标过滤**。

`moon ide doc` 默认使用模块的 `preferred_target`（`moon new` 创建的项目默认为 `wasm-gc`）。`@async/fs` 的 `unimplemented.mbt` 有 `#cfg(not(target="native"))`——即在 wasm/js 目标下只暴露这个占位符，真正的 API（`read_file`、`open` 等）在 native 文件中，被条件编译排除。async/fs 需要文件系统操作，wasm 目标下本来就不可用。

**正确用法**：

```bash
# ❌ wasm-gc 默认目标，只能看到 unimplemented
moon ide doc "@async/fs"

# ✅ 在 native 目标的项目中
# 在 moon.mod 中设置 preferred_target = "native" 后即可看到完整 API

# ✅ 用 .mbtx 实际 import + moon run --target native 验证
moon run -c 'import { "moonbitlang/async@0.20.2/fs" }; async fn main { println(@fs.read_file("test.txt").text()) }' --target native
```

**教训**：`moon ide doc` 按编译目标过滤，结果取决于 `preferred_target`。看到 `unimplemented` 时不代表包是空的——可能只是当前目标下不可用。以 `.mbtx` 实际运行验证为准。

---

## 相关 GitHub Issue

[moonbitlang/moon#1914](https://github.com/moonbitlang/moon/issues/1914) — 建议改进：在 wasm 目标下对 native-only 包给更友好的提示，而非仅显示 `unimplemented`。
