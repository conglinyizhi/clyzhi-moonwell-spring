# 官方 MoonBit skill 委派导航

月井之春不复制官方 skill 的完整内容。使用时优先读取本地文件；本地没有时访问或临时拉取官方仓库。

## 本地位置

常见安装位置：

```text
~/.pi/agent/skills/external/moonbit-skills/skills/*/SKILL.md
~/.pi/agent/skills/clyzhi/moonbit-skills-guide/SKILL.md
```

用下面的命令确认当前机器实际位置：

```bash
find ~/.pi/agent/skills ~/.agents/skills -path '*moonbit*' -name SKILL.md 2>/dev/null
```

## 委派

- 总体能力、API 新鲜度和工具链诊断：`moonbit-orientation`
- 项目结构、moon 命令、测试和文档：`moonbit-agent-guide`
- C FFI 基础：`moonbit-c-binding`
- C/C++ 绑定完整工程：`make-moonbit-c-bindings`
- MoonBit 重构：`moonbit-refactoring`
- 证明：`moonbit-proof`
- 规格测试：`moonbit-spec-test-development` / `moonbit-extract-spec-test`

官方远程仓库：<https://github.com/moonbitlang/skills>

如果 exact API、命令参数、target 支持或版本行为无法由本地项目确认，按照 `moonbit-orientation` 的 freshness gate 查 `moon ide doc`、官方文档、Mooncakes 或工具链输出，不要从本文件猜签名。
