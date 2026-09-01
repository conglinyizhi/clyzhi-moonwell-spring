## 补丁内容

---

### 1、配置文件格式迁移

`moon.mod.json` 在 v0.10.4 已弃用。`moon new` 现在创建 `moon.mod`（TOML-like）。迁移：运行 `moon fmt`。

```toml
# moon.mod（新格式）
name = "username/hello"
version = "0.1.0"
preferred_target = "native"
import { "moonbitlang/x@0.4.6", }
```

> ⚠️ 新 `moon.mod` 不再支持本地路径依赖（`path` 字段）。跨模块本地依赖必须使用 `moon.work`。

`moon.pkg`（新）同理，替代 `moon.pkg.json`：

```toml
# moon.pkg（新格式）
import { "username/hello/liba", }
import { "username/hello/test_helpers", } for "test"
supported_targets = "native"
options("is-main": true,)
```

关于 `supported_targets`：✅ `supported_targets = "native"`（字符串，推荐），⚠️ 数组写法可能阻止下游包在其他 target 构建。

---

### 2、moon.work 工作空间

多个模块同仓库时统一管理。

```bash
moon work init mod_a mod_b    # 初始化 + 注册成员
moon work use mod_c           # 添加新成员
moon work sync                # 同步成员间依赖版本
```

生成的 `moon.work`：
```toml
members = ["./mod_a", "./mod_b"]
```

工作空间根目录下 `moon check/test/info/clean` 作用于所有成员。模块专有命令（如 `publish`）需 `moon -C mod_a publish`。

> ⚠️ 已知限制：无法排除成员 ([#1903](https://github.com/moonbitlang/moon/issues/1903))、preferred_target 行为待明确 ([#1788](https://github.com/moonbitlang/moon/issues/1788))、moonc syncheck 不支持 ([#1859](https://github.com/moonbitlang/moon/issues/1859))。

---

### 3、moon runwasm

```bash
moon runwasm main
moon runwasm moonbitlang/parser/cmd/moonfmt@0.3.3   # Mooncakes 坐标
```

---

### 4、declare 关键字

官方 `moonbit-spec-test-development` 使用 `#declaration_only`。两种写法都有效，但 **`declare` 是推荐写法**。

```mbt
///|
declare pub type Yaml
///|
declare pub fn parse_yaml(s : String) -> Yaml raise
```

---

### 5、json_inspect() 写法

官方技能使用 `@json.inspect(...)`。**正确写法是 `json_inspect()`**（prelude 导入，无需包前缀）：

```mbt
json_inspect(value, content={ "key": "value" })   // ✅
```

---

### 6、属性完整列表

`moon explain --attribute` 输出（moon 0.1.20260826）：

| 属性 | 用途 |
|:--|:--|
| `#alert` | 编译时告警 |
| `#alias` | 符号别名 |
| `#as_free_fn` | 方法暴露为独立函数 |
| `#borrow`, `#owned` | C FFI 所有权标注 |
| `#callsite` | 调用位置信息 |
| `#cfg` | 条件编译 |
| `#coverage.skip` | 跳过覆盖率 |
| `#deprecated` | 弃用标记 |
| `#doc` | 文档元数据 |
| `#external` | C 指针（C 端管理生命周期） |
| `#inline` | 内联提示 |
| `#internal` | 内部 API |
| `#label_migration` | 标签迁移辅助 |
| `#module` | 模块级标注 |
| `#must_implement_one` | trait 方法互斥实现约束 |
| `#proof_external` | 外部证明声明 |
| `#proof_import` | 证明导入 |
| `#proof_pure` | 纯证明函数 |
| `#skip` | 跳过检查 |
| `#unsafe_cycle_free` | 非安全循环自由约束 |
| `#visibility` | 可见性控制 |
| `#warnings` | 警告控制 |

用 `moon explain --attribute <NAME>` 查看详情。

---

### 7、moon coverage 完整子命令

```bash
moon coverage analyze   # 插桩 + 收集覆盖率
moon coverage report    # 生成报告
moon coverage clean     # 清理产物
```

官方技能只提到 `analyze`。

---

### 8、moon prove（Why3 验证）

```bash
moon prove [PATH]
moon prove --why3-config <PATH>
```

Why3 运行时已内置。配合 `moonbit-proof` 技能使用。

---

### 9、moon explain

```bash
moon explain --diagnostic                  # 列出所有诊断码
moon explain --diagnostic 31               # 按编号
moon explain --diagnostic unused_optional_argument  # 按助记名
moon explain --attribute                   # 列出所有属性
moon explain --attribute deprecated        # 解释特定属性
```

---

### 10、moon fetch

```bash
moon fetch moonbitlang/async@0.18.1   # 下载到 .repos/（不修改 moon.mod）
```

`.gitignore` 加 `.repos/`。添加依赖用 `moon add`。

---

### 11、moon run --profile

```bash
moon run --profile --target native --release cmd/<main>
```

生成 `profile.json` + `.trace`。self-time → 哪个函数烧 CPU，inclusive-time → 哪个调用子树是瓶颈。

---

### 12、.mbtx 脚本

文档有 "Running .mbtx Scripts" 章节。`.mbtx` 是可执行 MoonBit 脚本。官方技能未涉及。

---

### 13、WASM Component Model

MoonBit 支持 WASM Component Model，可编译为 WASM 组件互操作。官方技能未涉及。

---

### 14、moon package --list

```bash
moon package --list   # 验证发布文件列表
```

---

### 15、moon check --output-json

```bash
moon check --output-json 2>&1 | jq -R 'fromjson? | select(.message | contains("unused"))'
```

---

### 16、--unstable-feature / -Z

```bash
moon -Z <FEATURE_NAME> check   # 启用不稳定特性
```

当前启用的 feature flags：`rr_moon_mod`、`rr_moon_pkg`。

---

### 17、.mbtx + @async/fs + @async/process 子进程与文件 I/O

官方技能未提及 `.mbtx` 可通过 `@async` 系列包进行跨平台子进程调用和文件 I/O。两者可在同一个 `async fn main` 中共存。

mbtx 依赖声明：
```mbt
import {
  "moonbitlang/async@0.20.2",
  "moonbitlang/async@0.20.2/fs",
  "moonbitlang/async@0.20.2/process",
  "moonbitlang/core/string" @string,
}
```

子进程（`@async/process`）：
```mbt
@process.collect_stdout("moon", ["version"])     // → (exit_code, &@io.Data)
@process.collect_stderr("moon", ["check"])       // → (exit_code, &@io.Data)
@process.run("moon", ["build"])                  // → exit_code
```

文件 I/O（`@async/fs`）：
```mbt
@fs.read_file("moonwell.toml")                   // → &@io.Data
@fs.open("out.txt", mode=ReadOnly)               // → File (支持 read_all 等方法)
```

`&@io.Data` 统一接口：
```mbt
let data = @process.collect_stdout(...)  // 或 @fs.read_file(...)
let text = data.text()                    // → String (UTF-8 解码)
let json = data.json()                    // → Json (解析 JSON)
let bytes = data.binary()                 // → Bytes (原始二进制)
```

来源：[moonbitlang/async/src/process](https://github.com/moonbitlang/async/tree/main/src/process)、[moonbitlang/async/src/fs](https://github.com/moonbitlang/async/tree/main/src/fs)

---

### 18、moon check --fmt / --explain / --patch-file（moon ≥ 0.1.20260717）

```bash
moon check --fmt        # 只检查格式，不修改代码（CI 友好）
moon check --explain     # 检查时展开解释错误码详情
moon check --patch-file <FILE>  # 对单个包的补丁文件检查
```

`--fmt` 与 `moon fmt --check` 等价，但在 `moon check` 流程中统一执行。官方技能未提及这些选项。

---

### 19、moon run/test --build-only + stdin 脚本 + moon doc 废弃（moon ≥ 0.1.20260717）

```bash
moon run --build-only <PKG>   # 只构建不运行
moon test --build-only        # 只构建不运行测试
echo 'fn main { println("hi") }' | moon run -   # stdin 读取 .mbtx
```

`moon doc [SYMBOL]` 已废弃：
```bash
moon ide doc <SYMBOL>   # ✅ 替代方案
```

官方技能未提及 `--build-only` 和 stdin `.mbtx` 能力。

---

### 20、Async / HTTP 服务默认栈（官方无独立后端框架）

MoonBit **官方** async 库没有 Express / Gin / Axum 级独立 Web 框架。HTTP 服务端能力由官方异步库提供：

> ⚠️ **已修正（实战回流）**：官方 async 库确实无框架，但第三方 `hackwaly/moonback` 提供 Express 级路由/中间件/依赖注入框架，`moonbit-community/rabbita` 提供 SSR/前端。**全栈场景见 `references/rabbita-fullstack.md`（补丁22）**。本条的「无框架」结论不再代表 MoonBit 整体，仅限官方 async 库。

| 能力 | 包 | 入口 |
|:--|:--|:--|
| HTTP Server / Client | `moonbitlang/async/http` | `@http.Server`、`@http.get` |
| TCP / 地址 | `moonbitlang/async/socket` | `@socket.Addr`、`TcpServer` |
| 异步运行时 | `moonbitlang/async` | `async fn main`、`with_task_group` |

**默认陷阱（`moon new`）：**

- 新建模块默认 `preferred_target = "wasm-gc"`
- `async fn main` / `@http.Server` **需要 native**（且须依赖 `moonbitlang/async`）
- 未改目标或未加依赖时：`Cannot use async fn main: package moonbitlang/async is not imported`

**最小服务端脚手架：**

```bash
moon add moonbitlang/async@0.20.2   # 版本以 mooncakes 最新为准
```

```toml
# moon.mod
preferred_target = "native"
import {
  "moonbitlang/async@0.20.2",
}
```

```toml
# moon.pkg（可执行包，如 cmd/main）
import {
  "moonbitlang/async",
  "moonbitlang/async/http" @http,
  "moonbitlang/async/socket" @socket,
}
supported_targets = "+native"
options(
  "is-main": true,
)
```

```mbt
///|
async fn main {
  let server = @http.Server(@socket.Addr::parse("0.0.0.0:8080"))
  try server.run_forever(allow_failure=true, (request, _body, conn) => {
    // 1. send_response → 2. 写 body（作 Writer）→ 3. end_response（可省略，返回后自动 end）
    conn.send_response(200, "OK", extra_headers={
      "Content-Type": "application/json; charset=utf-8",
    })
    conn.write_string("{\"ok\":true}")
  }) catch {
    err => println("server stopped: \{err}")
  }
}
```

**路由：** 官方 API 无内置路由器；手写 `match (request.meth, path)` 即可。URL 路由库可选用社区 `ShellWen/sw_router`（仅路由，不是完整 Web 框架）。HTTP 头解析可参考 `f4ah6o/http11`（解析层，不是服务框架）。

**官方示例路径（`moon fetch moonbitlang/async` 后）：**

- `examples/http_file_server/`
- `examples/http_server_benchmark/`
- `src/http/README.mbt.md`（Writing HTTP servers 一节）

**JSON 响应注意：**

- 统一信封可用 `Json` 字面量 + `body.stringify()`
- `String.length()` 是 **UTF-16 码元** 数；`Content-Length` 需要 **字节** 数。纯 ASCII JSON 二者一致；含非 ASCII 的 `err` 时勿直接用 `text.length()` 当字节长度

**官方技能缺口：** `moonbit-agent-guide` 有 async 语法，但未点明「无后端框架 + 默认 wasm-gc 与 HTTP native 冲突 + Server 脚手架」。

---

### 21、数组模式至多一个 `..` + Show 弃用改 Debug

#### 数组 / 字符串模式：`..` 至多一个

编译器错误：`At most one \`..\` is allowed in array pattern.`

```mbt
// ❌ 非法：多个 rest
match path {
  [.. p, .. "?", ..] => p
  _ => path
}

// ✅ 合法：先 find / 切片，再处理
let s = path.to_owned()
let without_query = match s.find("?") {
  Some(idx) => s[:idx].to_owned()
  None => s
}
```

字符串 / 字节上「去掉 query、尾斜杠」等，优先 `find` / 切片，不要堆多个 `..`。

#### `Show` 弃用 → `@debug.to_string` / `repr`

对部分类型（含部分 enum、`Option` 相关路径）直接 `\{value}` 插值或依赖旧 `Show` 会触发 **deprecated** 警告。

```mbt
// ⚠️ 可能 deprecated
let detail = "method not allowed: \{meth}"

// ✅ 调试 / 错误详情
let detail = "method not allowed: \{@debug.to_string(meth)}"
// 或 prelude 的 repr（与 @debug.to_string 同一函数）
let detail = "method not allowed: \{repr(meth)}"
```

规则：

- **用户可见展示** → 手写 `impl Show` 或领域格式
- **日志 / 错误详情 / 快照** → `Debug` + `@debug.to_string` / `repr` / `debug_inspect`

#### `catch` 与 `<|` 优先级

警告 `[0051] ambiguous_precedence`：`server.run_forever() <| (...) catch { ... }` 歧义。

```mbt
// ❌ 易警告
server.run_forever() <| ((req, body, conn) => { ... }) catch { err => ... }

// ✅
try server.run_forever(allow_failure=true, (req, body, conn) => { ... }) catch {
  err => ...
}

---

### 22、Rabbita 全栈 SSR（官方技能未覆盖）

`moonbit-community/rabbita`（原名 Rabbit-TEA）是 MoonBit 的 Elm/Bonsai 风格声明式 Web UI 框架，`hackwaly/moonback` 是其配套的 Express 级后端框架。二者组成 MoonBit 全栈 SSR 的完整生态，**官方 moonbit-* 技能完全未提及**。

- 全栈模式：共享组件包 `app/`（纯 UI）+ 后端 `cmd/server`（moonback Module + Rabbita SSR）+ 前端 `cmd/browser` + 根库包（`+native`）
- **核心约束**：根库包 `+native`，前端 JS 水合不能直接 import → `app/` 只能纯 UI，数据走 REST API
- **关键决策**：SSR + 水合复杂度高到不值，实战回退 MPA（SSR 完整 HTML + 内联 JS）

完整 API 速查、依赖声明、失败经验与坑 → **`references/rabbita-fullstack.md`**。

---

### 23、MoonBit 语言/工具层实战坑（native 全栈开发）

#### async trait impl 未稳定（静默丢弃）

`impl` 里**同步 fn 方法体不能调 async fn**。一旦调用，整个 `impl` 会被**静默丢弃**——编译通过、符号存在，但方法体不执行。方法上标 `async` 会标注「useless」。MoonBit 的 async trait impl 语法仍未稳定（官方文档标题 experimental）。

```mbt
// ❌ 陷阱：同步 trait 方法体调 async fn → impl 被丢弃，且无编译错误
// trait Storage { fn read(self, k: String) -> Option[String] }
impl Storage for LocalStorage with fn read(self, k) {
  // @fs.* 是 async → 此 impl 整体失效
  Some(@fs.read_file(k).text())
}

// ✅ 务实方案：trait 走同步 + core 同步文件 API；或独立 async 函数
```

> 症状是「方法不生效 / 符号 undefined」，而非编译失败。排查优先做最小实验定位（见下）。

#### core 没有文件 IO 包

`moonbitlang/core` **不含文件 I/O**。读文件用 `moonbitlang/async/fs`（`@fs.read_file`、`@fs.exists`）。区分 `@fs`（async 包）与 `@io`（统一数据接口：`text()/json()/binary()`）。

#### MOON_CC 坑（native backend）

native backend 需要 C 编译器/链接器驱动。工具链可能在找 `/usr/bin/lib.exe`（Windows archiver），此时 `moon run/build` 报错。解决：

```bash
MOON_CC=gcc moon run --target native .
```

项目级固化：在 `moon.mod`/`moon.pkg` 配 `link.native.cc`，或在 Makefile/构建脚本统一导出 `MOON_CC`。moon 会提示「`MOON_CC overrides link.native.cc configured by package`」。

#### 代理坑

本地有代理时，`moon` / `curl` 对本地地址可能走代理导致超时/403。用 `--noproxy '*'` 绕开：

```bash
moon build --noproxy '*'
curl -x http://127.0.0.1:10738 --noproxy '*' -L <url>
```

#### JWT/base64 解码在 nightly 不稳定

nightly 的 base64 解码路径不稳定，跨进程 JWT 验签可能失败。务实做法：同一进程内用 **token registry**（会话权威）兜底，JWT 仍按规范签发；退出时撤销登记（契合「仅服务生命周期有效」）。此属已知限制，非 API 契约。

---

### 24、生态包速查：LLM 底座 / D-Bus 桌面集成（官方技能未覆盖）

官方 moonbit-\* 技能只讲语言与工具链，**不提第三方生态包**。以下条目均为**实际 `moon add` 拉取并查阅 `.mbti` 验证过**的包，避免开发者重复造轮子。

#### LLM / AI 底座（不必自己实现 OpenAI / Anthropic 规范）

| 包 | 版本 | 体量 | 定位 |
|:--|:--|:--|:--|
| `QuietlyChan/moonai` | 0.1.0 | 481 文件 / 12.1 万行 / 19 子包 | **最全面**，对标 Vercel AI SDK |
| `tonyfettes/openai` | 0.1.1 | 5 文件 / 2734 行 | 轻量，**只做 OpenAI**（社区组织 `moonbit-community` 维护）|
| `eanzhao/pi-moonbit` | 0.1.5 | — | pi-mono 的 MoonBit 重写，带 `pimbt` CLI |
| `colmugx/mcp` | 0.17.3 | — | 类型安全 MCP SDK（server/client，STDIO/HTTP）|

**`QuietlyChan/moonai`** —— 唯一同时把 OpenAI 与 Anthropic 做成一等公民的统一层：

- 统一 API：`generate_text` / `stream_text` / `generate_object` / `embed` / `embed_many` / `complete` / `transcribe`
- Provider 抽象：`LanguageModelV4` / `EmbeddingModelV4` / `ImageModelV4` / `TranscriptionModelV4` / `SpeechModelV4` / `RerankingModelV4` / `VideoModelV4`
- Provider 子包：`openai`、`anthropic`、`openai_compatible`、`deepseek`、`alibaba`、`bytedance`、`minimax`、`moonshotai`、`open_responses`
- 附带：`mcp`（MCP 客户端）、`harness_pi` / `harness_opencode` / `harness_deepagents`（**含 pi 的 harness 适配**）
- 机制：provider registry、middleware、`RetryPolicy`、`CancellationToken`、`HttpTransport` 可插拔
- Anthropic 入口：`anthropic(model, api_key?, base_url?, beta_features?, http_transport?, ...) -> &LanguageModelV4`
- 依赖 `moonbitlang/async@0.20.2` + `cc06b/mooncry@0.13.1`
- ⚠️ 0.1.0 早期 alpha，1.0 前 API 可能变动

**`tonyfettes/openai`** —— 只调 OpenAI 时的轻量选择：

- `Client::new(http_client~, base_url?, api_key?)` —— base_url 可指 OpenRouter / Azure / Ollama / vLLM
- `async ChatCompletionsService::create(...)` —— **30+ 具名参数**（tools / reasoning_effort / response_format / web_search_options / logit_bias / prediction / modalities…）
- `async ChatCompletionsService::stream(...) -> Reader[ChatCompletionChunk]`
- 多模态 content part：`text_content_part` / `image_content_part` / `audio_content_part` / `file_content_part`
- 消息构造：`system_message` / `user_message` / `assistant_message` / `tool_message`
- **`pub(open) trait HttpClient`** —— HTTP 层可插拔（自带 test/http、test/async 两个适配包）
- 依赖 `moonbitlang/x@0.4.32`

选型：只要 OpenAI → `tonyfettes/openai`；要 OpenAI + Anthropic + 多模态 + MCP → `QuietlyChan/moonai`；做 AI coding agent → `eanzhao/pi-moonbit`。

#### D-Bus / Linux 桌面集成（纯 MoonBit，无 GLib/GIO/libdbus）

| 包 | 版本 | 定位 |
|:--|:--|:--|
| `conglinyizhi/moondbus` | 0.1.0 | 纯 MoonBit D-Bus 协议实现 |
| `conglinyizhi/moonsni` | 0.1.0 | KDE/freedesktop 系统托盘（StatusNotifierItem）|

**背景**：mooncakes 上此前**没有纯 MoonBit 的 D-Bus 协议实现**。`justjavac/tray` 是绕开 D-Bus、动态加载 C 的 AppIndicator（GTK3 依赖，且 Linux 下托盘点击事件不可用）。这两个包填的是协议层空缺。

**`conglinyizhi/moondbus`** —— D-Bus 协议层，仅 ~64 行 C 处理 unix socket：

- 连接：`connect_bus()`（SASL `EXTERNAL` + `BEGIN` 握手）、`recv_message`（按消息边界读取）
- 客户端：`hello` / `call` / `call_simple` / `call_with_string` / `request_name` / `emit_signal`
- **服务端 `Server` 抽象**：`Server::connect/hello/request_name/serve/emit_signal`，`serve(dispatch)` 是可复用的常驻事件循环，`DispatchCtx`（iface/member/sender/body）+ `Reply::make(body, signature)`
- 编码器 `Encoder`：偏移感知（`new_at(base)`），`array_with(elem_align, f)` / `struct_with(f)` / `struct_with_align(align, f)` / `variant_with(sig, f)` / `signature` / `variant_object_path`

**`conglinyizhi/moonsni`** —— 托盘，渐进式三子包（可独立取用）：

- `src/menu`：纯菜单数据模型（`Menu` / `ItemHandle` / `SubMenu` / i18n），**零 D-Bus 依赖**
- `src/dbusmenu`：纯 `com.canonical.dbusmenu` 编解码（`encode_layout` / `parse_event`）
- `src/tray`：完整托盘（`tray(cfg)` / `on` / `checkbox` / `submenu` / `separator` / `apply_translations` / `on_click` / `on_middle_click` / `on_scroll` / `run`）

#### D-Bus 手写协议的四个硬坑（踩过并修复）

1. **`String` 是 UTF-16**，`+` 拼接或 `@utf8.encode` 会引入 `\x00` → D-Bus 消息必须用 `Encoder` 字节级构建
2. **`signature`（`g` 类型）是单字节长度** + 内容 + NUL，与 `s`（u32 长度）不同
3. **回复必须设 `destination = 请求方 sender`**，否则总线无法路由回去
4. **dbusmenu 两个致命细节**：① `toggle-type` / `toggle-state` 必须出现在**每个**菜单项上（普通项给空串和 `0`），缺失会让 KDE 把子菜单渲染成整个父菜单；② `GetLayout` 必须尊重 `parentID` 参数（0=整树，非 0=该子菜单子树），否则点子菜单会重复弹出父菜单

**未解坑（供后来者参考）**：SNI 的 `IconPixmap`（`a(iiibay)`）在 variant 嵌套下的对齐编码，KDE 会断开连接。`encode_pixmap` 单独输出的字节结构已逐字节验证正确（`len | w | h | stride | ay_len | ARGB`），问题出在 `variant_with` 嵌套。moonsni 中该 API 标注为实验性。
