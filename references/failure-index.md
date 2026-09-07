# 常见坑：按错误表现检索

这是索引，不是语言手册。命中后先读官方 skill 和当前工具输出；历史记录只解释“为什么曾经这样处理”。状态以 moon `0.1.20260904`（2026-09-04）复测为准。

- 编译器报具体诊断码，或某 API 签名不确定：`moon explain --diagnostic <code-or-name>`、`moon ide doc`、官方 `moonbit-orientation`
- `moon ide doc` 返回 `unimplemented` / 没有预期 API：先确认模块、依赖、target 和本机符号索引；旧的 `@async/fs` 默认 target 陷阱已于 `0.1.20260904` 消失，详见 `moon-ide-doc-gotcha.md`
- native `moon run` / `moon build` 找不到 C compiler、linker 或 `/usr/bin/lib.exe`：官方 `moonbit-c-binding`、`make-moonbit-c-bindings`；`0.1.20260904` 当前机器未设 `MOON_CC` 仍可复现，设置 `MOON_CC=gcc` 后通过。它是环境 / toolchain 选择问题，不要泛化成所有项目都会失败
- async trait 的同步方法调用 async 函数：旧的“编译通过但 impl 静默丢弃”已于 `0.1.20260904` 消失；当前报 `E4149 cannot call async function in non-async function`，见 `../history/legacy-patches.md` 的补丁23与 `../history/nightly-retest-20260904.md`
- core 里找不到文件 I/O：仍未发现 `moonbitlang/core/fs`、`core/file` 或 `core/io`；使用 `moonbitlang/async/fs`。此结论在 `0.1.20260904` 复测仍成立
- `no version satisfies requirement ...`：读 `../playbooks/mooncakes-publish.md`；registry 传播延迟需要真实发布链路才能复现或排除，当前不能标为已修复
- 发布成功但下游包仍解析不到上游新版本：刷新 Mooncakes registry，重跑解包复检；见发布 playbook。当前 registry 查询正常，但这不反证时序问题
- Rabbita SSR 页面没有预期 input 数据：读 `../playbooks/rabbita-fullstack.md` 的 SSR input / prefetch 小节；当前未重新搭建完整 SSR 项目，不能声称已修复
- Rabbita 静态页资源 404 或中文显示异常：读 `../playbooks/rabbita-fullstack.md` 的 static / charset 小节；这是项目配置问题，当前未证实框架层有变化

## 最小证据链

```text
错误原文 / 复现命令
→ 当前项目 moon.mod、moon.pkg、moon.lock（如有）
→ moon ide doc / moon search / Mooncakes docs
→ 官方 skill
→ 月井之春专项 playbook
→ 历史记录（只在需要解释时）
```
