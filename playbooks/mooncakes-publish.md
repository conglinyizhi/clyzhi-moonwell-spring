# Mooncakes 发布流程

## 适用场景

发布或迭代 MoonBit 模块，尤其是一个包发布后另一个包要依赖新版本的场景。

## 发布前

1. 按 SemVer 判断版本：不破坏公开 API 的修复通常升 patch；新增兼容 API 或不兼容变更另行判断
2. 检查 `moon.mod` 的模块版本、依赖版本和 target
3. 若有上下游依赖，先准备并发布上游模块
4. 运行项目自己的格式、检查和测试
5. 查看发布文件列表
6. 做发布复检

推荐命令：

```bash
moon fmt --check
moon check --target native
moon test --target native
moon package --list
moon publish --dry-run
```

native 包按项目实际情况设置 `MOON_CC`，不要把示例中的编译器名字当成通用事实。

## `moon publish` 的复检边界

`moon publish` 不只是上传当前源码。实际流程包含：

```text
当前模块检查
→ 生成发布 zip
→ 解包到临时目录
→ 对解包包重新 moon check
→ 上传 Mooncakes
```

所以“开发目录检查通过”不等于“发布完成”。只有发布命令完成并返回成功状态，且之后能在 Mooncakes 查询到版本，才算落地。

## 上游 / 下游顺序

```text
上游版本发布成功
→ registry 可解析到上游版本
→ 更新下游 moon.mod 依赖
→ 下游本地检查与发布复检
```

例如 `moonsni` 依赖 `moondbus` 新版本时，不要先发布 `moonsni`。

## 依赖传播异常

如果下游解包复检出现：

```text
no version satisfies requirement <owner>/<module> <version>
```

先判断上游是否已经确实发布，再刷新 registry：

```bash
moon update
moon search <owner>/<module>
moon publish
```

不要为了绕过复检而降级依赖或修改成不存在的版本。`moon search` 已看到版本但解包复检仍看不到时，优先按 registry 索引传播延迟处理，稍后重试。

## 发布后验收

```bash
moon search <owner>/<module>
```

核对：

- 新版本可见；
- `moon add <owner>/<module>@<version>` 能解析；
- 下游包的解包复检能下载该版本；
- 公开接口与预期一致。

## 当前个人包例子

本次实测已发布并查询确认：

```text
conglinyizhi/moondbus@0.1.1
conglinyizhi/moonsni@0.1.1
```

这两个版本是历史事实，不代替发布前的当前查询。
