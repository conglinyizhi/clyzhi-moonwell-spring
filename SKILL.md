---
name: clyzhi-moonwell-spring
description: >
  MoonBit 生态与常见实战坑的导航层。必须与官方 moonbit-* 技能同时加载。
---

# 月井之春：MoonBit 导航层

> 本技能是导航和提醒，不是 MoonBit 全量手册。
> **必须先加载官方 `moonbit-*` 技能**；语言、工具链、FFI、证明和重构等通用知识交给官方技能维护。
> 月井之春只补充：常见失败表现、个人实战坑、Mooncakes 生态和专项开发流程。

## 加载顺序

1. 先读取本文件，判断任务类别和错误表现
2. 读取 `references/index.md` 做粗分流
3. 按命中的主题读取对应子导航
4. 只有需要具体历史细节时，才读取专项 playbook 或历史记录
5. exact API、包版本和 target 支持必须以本地工具链、项目文件或 Mooncakes 当前页面为准

## 粗分流

- MoonBit 语法、类型、包布局、`moon` 命令或编译目标：先读官方 skill，再看 `references/official-skills.md`
- 编译器诊断、warning 或 API 不确定：读 `references/failure-index.md`，再回到官方 skill / `moon ide doc`
- 查找、评估或组合 Mooncakes 包：读 `references/mooncakes.md`
- 发布包、升级版本、下游依赖找不到新版本：读 `playbooks/mooncakes-publish.md`
- 从零做 Rabbita / moonback 网站：读 `playbooks/rabbita-fullstack.md`；先问是否要跑完整开发流程
- C FFI、native 胶水或跨平台桌面集成：读官方 `moonbit-c-binding` / `make-moonbit-c-bindings`，再查 `references/failure-index.md`
- 把产物接进 Node / JS（js 或 wasm-gc target、`#export_name`、`extern "js"`、导入 .wasm 报错）：读 `playbooks/js-wasm-interop.md`
- 更新月井之春本身：读 `maintenance/update-skill.md`
- 追溯某次历史决定或失败：读 `history/README.md`；不要把历史记录当作当前规则

## 错误表现提醒

遇到问题时，先按“错误长什么样”检索，而不是凭模块名猜：

- **`no version satisfies requirement ...` / 发布包解包复检失败**：通常命中 Mooncakes registry 或依赖传播，读 `playbooks/mooncakes-publish.md`
- **`moon ide doc` 返回空、`unimplemented` 或 API 与记忆不符**：先确认项目上下文、依赖、target 与符号索引，再读官方 API；旧的 `@async/fs` 默认 target 陷阱已在 `0.1.20260904` 失效，见 `references/failure-index.md`
- **`native` run/build 找不到 C 编译器、链接器或 `/usr/bin/lib.exe`**：读官方 C binding skill，再查 failure index；当前 nightly 仍可因 compiler 选择失败，必要时显式设置有效的 `MOON_CC`
- **Rabbita 页面能 SSR 但首屏数据为空、hydration / static 路径异常**：读 `playbooks/rabbita-fullstack.md`
- **sync trait 方法里调用 async 函数**：当前 nightly 会报 `E4149`，不再静默忽略 impl；读 `references/failure-index.md`
- **跨包构造报 `Cannot create values of the read-only type`**：类型需要 `pub(all)`；impl 需要 `pub`，否则下游报 `does not implement trait ... although an impl is defined`；读 `references/failure-index.md`
- **FFI 传字符串乱码 / `strlen` 异常**：`String::to_bytes()` 是 UTF-16，跨 C FFI 必须 `@utf8.encode()`；读 `references/failure-index.md`
- **wasm-gc 导入 Node 报 `Cannot find package '_'`**：`imported-string-constants` 没指到 `wasm:js/string-constants`（官方 skill 的示例值 `"_"` 在 Node 下正好坏）；读 `playbooks/js-wasm-interop.md`
- **`type incompatibility when transforming from/to JS`**：wasm-gc 没开 `use-js-builtin-string`；读 `playbooks/js-wasm-interop.md`
- **看到一个熟悉的包名但不确定版本/API/target**：先 `moon search <query>`，再 `moon add` 到临时模块或读取 Mooncakes docs

## 官方 skill 委派

官方技能存在时优先让它们维护重复内容：

- `moonbit-orientation`：能力判断、信息源选择、API 新鲜度
- `moonbit-agent-guide`：项目结构、测试、`moon` 工作流
- `moonbit-c-binding`：MoonBit C FFI 基础
- `make-moonbit-c-bindings`：完整 C/C++ 绑定工程流程
- `moonbit-refactoring`：MoonBit 重构与 API 设计
- `moonbit-proof`：Why3 / 证明携带代码
- `moonbit-spec-test-development` / `moonbit-extract-spec-test`：规格与测试

若本地没有官方 skill：

1. 从官方仓库 https://github.com/moonbitlang/skills 查对应目录和 `SKILL.md`
2. 可以联网读取就直接读取远程内容
3. 仍无法确认时，再将仓库拉到临时本地目录后读取
4. 不要把未验证的官方内容复制进月井之春

## Mooncakes 与 Rabbita 提醒

- 需要做 plan、前导调查或包选型：先进入 `references/mooncakes.md`；个人包也必须按同一条证据链重新核验
- 要从零搭建 Rabbita / moonback 网站：进入 `playbooks/rabbita-fullstack.md`；该 playbook 会要求先确认是否启动 Warren 的完整热更新开发流程

## 维护入口

用户说“更新月井之春”时只读取并遵守：

```text
maintenance/update-skill.md
```

该文件规定：官方 skill 优先、Mooncakes 事实核验、常见坑的错误表现索引、历史记录边界、文件拆分和验证要求。不要直接把新资料堆进本文件。
