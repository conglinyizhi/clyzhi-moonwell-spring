# moon `0.1.20260904` nightly 复测

- 版本：`moon 0.1.20260904 (94521db 2026-09-04)`
- 复测时间：2026-09-07
- 范围：只测试仍作为当前规则出现的工具链、语言行为与 Mooncakes 专项结论；旧补丁全文仍是历史材料

## 已消失

### 默认项目中 `@async/fs` 只暴露 `unimplemented`

- 旧现象：默认 target 下容易只看到 `unimplemented`，误以为 `@async/fs` 不可用
- 当前证据：`moon new` 生成 `preferred_target = "wasm"`；分别以 `wasm` 和 `native` 编译引用 `moonbitlang/async@0.20.2/fs` 的最小 `@fs.exists` 均通过
- 结论：于 `0.1.20260904` 首次确认该旧结论失效
- 当前入口：`../references/moon-ide-doc-gotcha.md`

### sync trait 调用 async 函数后 impl 静默丢弃

- 旧现象：同步 trait 方法体调用 async 文件 API 时，曾可编译却令 impl 行为失效
- 当前证据：最小复现被当前编译器拒绝：`E4149 cannot call async function in non-async function`；同时还有错误类型传播诊断 `E4122`
- 结论：于 `0.1.20260904` 首次确认“静默丢弃”已消失。不要据此宣称 async trait API 已全面稳定；这里只确认非法同步调用不再静默通过

## 仍成立

### core 没有可用文件 I/O 包

- 复测：在最小模块可解析 core 的前提下，`moonbitlang/core/fs`、`moonbitlang/core/file`、`moonbitlang/core/io` 均无法作为 import 解析
- 对照：`moonbitlang/async@0.20.2/fs` 可安装并在 native 最小程序中通过 `moon check`
- 结论：`0.1.20260904` 下仍应从 `moonbitlang/async/fs` 查找文件 I/O

### native 构建依赖可用 C toolchain

- 复测：当前机器的 `/usr/bin/cc` 指向 GCC，干净项目的 `moon check --target native` 可通过；但 `moon run --target native cmd/main` 未设 `MOON_CC` 时仍寻找 `/usr/bin/lib.exe` 并失败，`MOON_CC=gcc` 后输出 `Hello`
- 结论：当前 nightly 的 native **检查**不一定触发问题，但 run/build 的 C toolchain 选择问题仍存在；保留 `MOON_CC` 兜底，不把它写成所有 native 命令的必需前置

## 条件性结论：未宣称修复

### Mooncakes 发布后依赖传播延迟

- 当前 `moon search` 可解析：`conglinyizhi/moondbus@0.1.1`、`conglinyizhi/moonsni@0.1.1`、Rabbita、Warren、moonback
- 这只证明当前 registry 可查询，不能反证“刚发布版本在下游解包复检暂不可见”的时序问题
- 未进行新发布，因此保留发布 playbook 的刷新 / 重试流程，不标记为已修复

### Rabbita SSR input、static 与 hydration 问题

- 当前可查询：`moonbit-community/rabbita@0.15.6`、`moonbit-community/warren@0.3.3`、`hackwaly/moonback@0.8.1`
- `warren --help` 仍列出 `new`、`dev`、`build`
- 未搭建完整 SSR / hydration 项目；不把包可见或 CLI 存在写成框架行为已修复
