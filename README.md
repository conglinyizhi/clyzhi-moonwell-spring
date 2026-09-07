# 月井之春

MoonBit 官方技能的导航与生态热修复层。它不重写官方 skill：语言、工具链、FFI、证明和重构等通用内容优先交给 [moonbitlang/skills](https://github.com/moonbitlang/skills)；月井之春只补充常见失败表现、Mooncakes 生态、个人包探查与专项开发流程。

**追踪版本**：moon 0.1.20260904。以 `MOON_CC=gcc moon run scripts/verify.mbtx --target native` 核验；当前 nightly 在本机未设 `MOON_CC` 的 native run/build 仍可能寻找 `/usr/bin/lib.exe`，应显式选择可用 C 编译器。

## 使用方式

先读 `SKILL.md`，再按 `references/index.md` 分流。专项任务读取 `playbooks/`，历史细节只在需要追溯时读取 `history/`。用户说「更新 moonwell-spring」时沿 `maintenance/update-skill.md` 更新。

## 目录职责

- `SKILL.md`：一级导航、粗分流和错误表现提醒
- `references/`：二级导航、官方 skill 委派、Mooncakes 生态和失败索引
- `playbooks/`：Mooncakes 发布、Rabbita / moonback 等专项完整流程
- `history/`：历史版本、失败复盘和长篇深入笔记
- `maintenance/`：更新本 skill 的规范
- `references/patches*.md`：旧补丁兼容入口，不作为日常默认加载资料

## 当前补充范围

现有 25 条历史补丁已逐步迁移到分层入口，主要包括：

- 配置、workspace、`moon` 子命令和诊断的历史兼容记录
- Async / HTTP、语言层常见行为坑和 native 开发注意事项（包含已于当前 nightly 消失的历史坑标记）
- Rabbita + moonback 全栈流程与 Warren 开发工具
- Mooncakes 生态包、个人包探查、发布打包复检和依赖索引传播

## 官方技能委派

- `moonbit-orientation`：总体能力、API 新鲜度和工具链诊断
- `moonbit-agent-guide`：项目结构、moon 命令、测试和文档
- `moonbit-c-binding`：C FFI 基础
- `make-moonbit-c-bindings`：完整 C/C++ 绑定工程
- `moonbit-refactoring`：MoonBit 重构
- `moonbit-proof`：证明
- `moonbit-spec-test-development` / `moonbit-extract-spec-test`：规格与测试

## 许可

MIT
