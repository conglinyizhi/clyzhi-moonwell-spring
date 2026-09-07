# Mooncakes 生态子导航

## 什么时候查

以下情况不要直接手写功能：

- 需要 TOML、JSON、HTTP、WebSocket、音频、托盘、剪贴板、TUI 等基础能力；
- 需要做迁移 plan，想知道生态是否已有可复用实现；
- 想评估自己发布的包是否能作为下游依赖；
- 不确定包的最新版本、公开 API 或 target。

## 最短检索流程

```bash
moon search <关键词>
moon add <owner>/<module>@<version>   # 在临时模块中验证下载和解析
moon tree
moon ide doc <包或符号>
```

不要仅凭搜索摘要判断可用性。至少核对：

- Mooncakes 当前版本；
- `moon.mod` 的 `preferred_target` / `supported_targets`；
- 依赖树；
- `.mbti` 公开接口；
- 测试、README 和真实目标后端。

## 个人包探查

`conglinyizhi` 下的包也走普通 Mooncakes 证据链，不视为隐式可信：

```bash
moon search conglinyizhi
moon add conglinyizhi/toml@<version>
moon add conglinyizhi/moondbus@<version>
moon add conglinyizhi/moonsni@<version>
```

当前已验证的桌面相关包见 `../playbooks/mooncakes-publish.md` 和补丁历史；版本会变化，使用前重新查询。

## 选择包时要问

```text
这个包是否支持目标 backend？
它提供的是完整运行路径，还是只有 helper / 设备枚举？
它是否有 native-only、外部程序或系统库前置条件？
公开 API 是否稳定，能否从 .mbti 和测试确认？
```
