# 月井之春子导航

本文件负责第二层分流。先按任务或错误表现命中一个主题，再读取对应文件；不要默认加载全部 references。

## 按任务

- 官方 MoonBit 能力、工具链、语言或 FFI：`official-skills.md`
- Mooncakes 包搜索、个人包、版本或 target 评估：`mooncakes.md`
- 编译失败、warning 或行为异常的表现检索：`failure-index.md`
- Rabbita + moonback 全栈网站：`../playbooks/rabbita-fullstack.md`
- Mooncakes 发布：`../playbooks/mooncakes-publish.md`
- 更新本 skill：`../maintenance/update-skill.md`
- 历史复盘：`../history/README.md`

## 按错误表现

- `no version satisfies requirement`：可能是 registry 索引或上游依赖未传播；读 `../playbooks/mooncakes-publish.md`
- `moon publish` 当前检查通过、解包检查失败：开发树与发布包不一致，或依赖不可解析；读 `../playbooks/mooncakes-publish.md`
- `moon search` 找不到刚发布的版本：registry 索引尚未刷新；读 `../playbooks/mooncakes-publish.md`
- `moon ide doc` 空结果或 `unimplemented`：项目上下文、依赖、target 或本机符号索引问题；旧 `@async/fs` 默认 target 陷阱已于 `0.1.20260904` 消失。读 `failure-index.md`、`moon-ide-doc-gotcha.md`，并加载官方 `moonbit-orientation`
- native link / C compiler 错误：native 工具链或 FFI 配置；加载官方 C binding skill
- SSR 首屏无数据、页面资源 404 或热更新开发体验异常：Rabbita / moonback 流程误用；读 `../playbooks/rabbita-fullstack.md`
- sync trait 方法调用 async 函数：`0.1.20260904` 已从静默失效改为 `E4149` 诊断；读 `failure-index.md`，再查官方文档

## 资料优先级

```text
当前项目文件 / 当前 moon 与 Mooncakes 查询
> 官方 skill 与官方文档
> 月井之春当前规则
> 月井之春历史记录
> 模型记忆
```
