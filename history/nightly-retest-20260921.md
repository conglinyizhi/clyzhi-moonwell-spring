# moon `0.1.20260921` 复测

- 版本：`moon 0.1.20260921 (e46d2ed 2026-09-21)`，`moonc v0.10.14+32f9dd1de-nightly`
- 复测时间：2026-09-24
- 范围：`../references/failure-index.md` 里 20 条可低成本复现的断言，逐条写最小片段实跑
- 方法：单文件 `.mbtx` 用 `moon check` / `moon run --target native` / `moon build --target native`；
  需要多包时 `moon new` 新建最小项目。全部为本机实测，未联网 `moon update`

## 已消失

### native 构建会去找 `/usr/bin/lib.exe` 并失败

- 旧现象：不设 `MOON_CC` 时 `moon run --target native` 失败，找不到 `/usr/bin/lib.exe`；设 `MOON_CC=gcc` 后通过
- 当前证据：新项目里不设 `MOON_CC`，`moon run --target native cmd/main` 直接 exit 0 输出 `Hello`；
  `moon clean` 后重建同样通过，产物是真的 ELF；本机 `/usr/bin/lib.exe` 不存在，`/usr/bin/cc -> gcc`（GCC 16.2.1）
- 结论：于 `0.1.20260921` 首次确认该旧结论失效
- 当前入口：`../references/failure-index.md` 的 native C toolchain 一条

### core 里没有 String → Int / Double 的直接入口

- 旧现象：只能靠 `@bigint.BigInt::from_string(...).to_int()` 或手写十进制循环
- 当前证据：`core/string/pkg.generated.mbti` 有 `parse_int(StringView, base? : Int) -> Int raise`、
  `parse_double`、`parse_int64`、`parse_uint`、`parse_uint64`、`parse_bool`、`parse_bigint`；
  `@string.parse_int("1234")` 实测输出 `1234`
- 结论：旧结论失效。注意边界：**顶层 `@strconv` 仍然是空包**，入口在 `@string`；
  `Int::from_string` / `String::to_int` 这两个名字仍不存在
- 未判定：这些 `parse_*` 是哪一版进 core 的，本轮没查
- 当前入口：`../references/failure-index.md` 的 `@strconv` 一条

### 同目录两个 `.mbtx` 的 `single.exe` 会互相覆盖

- 旧现象：产物落在 `<脚本所在目录>/_build/native/debug/build/single/single.exe`，后者覆盖前者
- 当前证据：路径变成 `_build/<脚本名>.mbtx/native/debug/build/single/single.exe`；
  同目录 `t15.mbtx` 与 `t15b.mbtx` 各自的 `single.exe` 并存，分别输出 `hi` / `hi2`
- 结论：覆盖问题随路径改成按脚本名分目录而消失
- 当前入口：`../references/failure-index.md` 的 `.mbtx` 产物一条

## 仍成立（含需改措辞的细节）

- `/* */` 报 parse error（原文多了优先级数字：`unexpected token infix 3`）
- `impl … with async fn` 报 `unexpected token \`async\``
- 顶层大写 `let` 报 `Did you mean \`const\`?`
- `unused_mut` 是 Error（0015），`moon check` 退出码非 0
- `if v is k`（右侧变量）静默恒真、`moon check` exit 0，但 warning 由旧记录的 1 条变成 **2 条**同名 `Unused variable 'k'`
- UTF-16 码元切片 `s[1:2]` 静默给 0，`unsafe_substring` 给 1
- `ArrayView::to_array()` 报 deprecated → `to_owned`
- 构造子当高阶函数报 4203
- match 分支直写 `let` 报 3002
- `.mbtx` 默认 target 是 wasm（`extern "c"` 报 4156）
- `.mbtx` 只 import 子包时 `async fn main` 报 4037
- `.mbtx` 子包 import 版本混写报 `multiple versions specified for module`
- `moon explain --diagnostic` 列出全集（含 0015 error / 0020 warn / 0025 warn / 0079 warn）
- `moon new` 默认 `preferred_target = "wasm"`
- `Json::number` 只收 Double：**仅对 Int 变量成立**，字面量 `Json::number(1)` 已能编译

## 条件性结论：未宣称失效

### `moon ide doc` 报 `Fail to load core … packages.json` 时的判读规则

- 该前置错误在本机复现不了：`~/.moon/lib/core/_build/packages.json` 存在，
  `moon ide doc "@async/fs"` 正常列出 `pub async fn exists` 等
- 因此「出现该错误时不要据此判定 API 不存在」这条判读规则**未复测**，保留原文
- 当前入口：`../references/moon-ide-doc-gotcha.md`

## 未覆盖

- Rabbita / Warren / moonback 的 SSR、hydration、static 路径类断言（未搭完整框架项目）
- wasm-gc / js target 一簇（`#export_name`、`BoolOrLink`、`supported_targets` 等，需 Node 侧验证）
- C FFI、SQLite、Semaphore、TcpServer 等运行时断言
- 任何涉及 registry 最新版的行为：本轮未联网 `moon update`，本地缓存与索引状态会掺水，不作为结论
