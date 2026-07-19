---
name: clyzhi-moonwell-spring
description: >
  MoonBit 官方技能热修复层。月井之春:补丁如泉水回春。必须与官方 moonbit-* 技能同时加载。
---

# MoonBit Hotfix Skills · 月井之春

> 官方 [moonbitlang/skills](https://github.com/moonbitlang/skills) 的热修复层。
> **必须与官方 moonbit-\* 系列技能同时加载。** 本技能只记录官方技能尚未覆盖或已过时的内容，
> 不重复官方技能已有信息。内容冲突时以本技能为准。

**配置**：[moonwell.toml](moonwell.toml) | **仓库**：[conglinyizhi/clyzhi-moonwell-spring](https://github.com/conglinyizhi/clyzhi-moonwell-spring)

---

## 触发条件

以下关键词/场景触发本技能（与官方 moonbit-\* 技能同时加载）：

- 使用 MoonBit 开发、编写 `.mbt` 代码
- 配置 `moon.mod`、`moon.pkg`、`moon.work`
- 执行 `moon check`、`moon build`、`moon test`、`moon run`、`moon fmt`、`moon new`
- 使用 `moon work`、`moon prove`、`moon runwasm`、`moon fetch`
- 涉及 C FFI 绑定、形式化验证、WASM 编译目标、`.mbtx` 脚本
- 用户要求「更新 moonwell-spring」或 moon 版本升级后

---

## 执行

本技能有两种执行模式：

### 模式 A：知识覆盖（默认，每次加载）

Agent 加载本技能后，补丁内容自动覆盖官方技能的对应条目。日常加载用精简索引 `references/patches.min.md`，需要详情时按条目编号查阅 `references/patches.md`。

### 模式 B：版本更新（用户触发）

当用户说「更新 moonwell-spring」或 moon 版本变化时执行。5 Phase：版本检测 → 拉取 GitHub Release / Blog / 文档 / 官方 skills → 差异分析 → 增量检查 → 更新本文档。完整步骤见 `references/update-workflow.md`。

---

## 补丁内容

→ 精简索引：**`references/patches.min.md`**（17 条目索引 + 官方信息来源 + 官方技能对照）
→ 完整详情：**`references/patches.md`**（17 条目详细说明）

---

## 注意事项

- 本技能内容基于 `tracked_version` 验证。如果 `moon --help` 实际输出与本文档不一致，**以工具输出为准**，并触发更新流程
- `moon ide doc` 是 API 发现的首选方式，比文本搜索更准确
- moon.work 仍在迭代中（3 个 open issue），单模块仓库暂不需要
- `moon.mod.json` → `moon.mod` 迁移不可逆，`moon fmt` 后本地路径依赖会丢失——提前用 `moon.work` 替代
- `.mbtx` 脚本可通过 `@async/process` + `@async/fs` 进行跨平台子进程调用和文件 I/O，官方技能未提此能力（详见 `references/patches.md` 补丁十七）
- ⚠️ `moon ide doc "@async/fs"` 在非模块上下文可能仅返回 `unimplemented`，不代表包空。详见 `references/moon-ide-doc-gotcha.md`。

---

## 验证

运行跨平台验证脚本（需要 `moon` + `moonbitlang/async`）：

```bash
moon run scripts/verify.mbtx --target native
```

输出格式：

```text
[PASS] moon version
expected: 0.1.20260626
actual:   0.1.20260626

[FAIL] moon --help command count
expected: ≥ 27
actual:   3
```

5 项检查全部 [PASS] = 技能新鲜。
