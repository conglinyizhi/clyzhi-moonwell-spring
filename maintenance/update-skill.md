# 月井之春维护规范

## 目标

本文件是“更新月井之春”时的唯一操作入口。更新内容优先服务导航和常见实战坑，不追求复制 MoonBit 官方手册。

## 资料优先级

```text
当前 moon / 当前项目 / 当前 Mooncakes 查询
> 官方 moonbit-* skill 与官方文档
> 月井之春当前规则
> 月井之春历史记录
> 模型记忆
```

## 官方 skill 读取规则

1. 先检查本地官方 skill：
   ```bash
   find ~/.pi/agent/skills ~/.agents/skills -path '*moonbit*' -name SKILL.md 2>/dev/null
   ```
2. 本地存在则读取对应 `SKILL.md`，不要重复维护其通用内容
3. 本地不存在则访问 <https://github.com/moonbitlang/skills>
4. 远程读取失败时，才把官方仓库拉到临时目录后读取
5. 仅把官方 skill 未覆盖、已过时或 MoonBit 生态特有的内容写入本技能

## Mooncakes 事实核验

涉及包名、版本、API、target、依赖或发布状态时：

```bash
moon search <query>
moon add <owner>/<module>@<version>
moon tree
moon ide doc <package-or-symbol>
```

必要时在临时模块中安装，读取 `.mbti`、`moon.mod`、README 和测试。个人包（包括 `conglinyizhi/*`）与其他包使用同样的证据标准。没有 Mooncakes 登录态时，不把“发布成功”写成事实；提示用户登录或让用户提供可验证结果即可。

## 先按错误表现写索引

新坑不要只写“某包有问题”，先记录 agent 实际会看到的表现：

```text
错误原文 / warning / 命令输出
→ 可能命中的坑类型
→ 先读哪个子导航或专项 playbook
→ 最小复现 / 验收命令
→ 当前结论
→ 历史背景（如需要）
```

更新 `../references/failure-index.md` 时，优先添加错误表现和下一站；详细解释放专项 playbook，长时间线放 `../history/`。

## 文件职责

- `../SKILL.md`：只做一级导航、粗分流、提醒和官方委派
- `../references/index.md`：二级主题 / 错误表现导航
- `../references/official-skills.md`：官方 skill 委派位置
- `../references/*`：当前知识、短索引和常见坑
- `../playbooks/*`：一个专项从准备到验收的完整流程
- `../history/*`：历史失败、版本和已完成工作；明确标注历史，不覆盖当前规则
- `../maintenance/*`：本 skill 自身的维护规范

不要把所有补丁拆成大量孤立文件；只有具有独立触发词、流程、验收或更新节奏的主题才建专项文件。

## 文档格式

默认使用无序列表和短段落，减少 token 与 Markdown 噪声：

- 导航、触发条件、错误表现、包速查、步骤：默认用无序列表
- 只有需要固定字段逐列横向比较，且列表会明显丢失关系时才使用表格
- `SKILL.md` 不使用表格；它必须保持短、可扫描、只负责导航
- code block 只保留可执行命令、错误原文或必须保真的输出
- 不为视觉整齐复制冗余列、分隔符或宽表

## 修改流程

1. 读取本文件和受影响的导航
2. 明确这次新增是当前规则、专项流程还是历史记录
3. 查本地官方 skill、当前 moon 和 Mooncakes 事实
4. 更新最小必要文件及交叉索引
5. 检查每个链接、文件路径、补丁数量和版本描述一致
6. 运行与内容相关的验证命令
7. 在 `../history/README.md` 追加简短更新记录
8. 用清晰的提交说明提交变更

## 不应合入

以下内容不属于 MoonBit skill 知识，应留在外部项目/agent 流程：

- agent 调度和 subagent 运行策略
- 沙箱权限、工作区权限和工具授权过程
- GitHub API / Git smart protocol 的操作事故
- 与 MoonBit / Mooncakes 无关的项目内部信息
