# moon ide doc 踩坑记录

## `@async/fs` 在无模块上下文中返回 `unimplemented`

**现象**：在无 `moon.mod` / `moon.pkg` 的目录中运行：

```bash
moon ide doc "@async/fs"
# → package "moonbitlang/async/fs" // moonbitlang/async@0.19.1
# → pub let unimplemented : Unit
```

**实际**：`@async/fs` 有完整的 File API——`read_file`、`open`、`File::read_all`、`File::size` 等。

**原因**：`moon ide doc` 在非模块上下文（无 moon.mod/moon.pkg 的项目中）查询某些 async 子包时，可能无法正确解析包成员，返回 `unimplemented` 占位符。

**教训**：`moon ide doc` 返回 `unimplemented` 不代表包真的空。以 GitHub 源码或 mooncakes.io 文档页面为准。在 `.mbtx` 脚本中实际 import + `moon run` 是最终的验证手段。

---

## 验证方式

```bash
# ❌ 可能误导
moon ide doc "@async/fs"

# ✅ 可靠验证：用 .mbtx 实际 import 并 run
moon run -c 'import { "moonbitlang/async@0.20.2/fs" }; async fn main { println(@fs.read_file("test.txt").text()) }'
```
