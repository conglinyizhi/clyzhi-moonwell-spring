# Rabbita 全栈 SSR 开发实战（官方技能未覆盖）

> 来源：rootwarren（原 mbt-mdwiki）全栈 SSR 重写实战回流（moon 0.1.20260717 期间）。
> 官方 `moonbit-*` 技能未提及 Rabbita / moonback / warren，本文件补齐这一块生态。

## Rabbita 是什么

**Rabbita**（`moonbit-community/rabbita`，原名 `Rabbit-TEA`）是 MoonBit 的声明式、函数式 Web UI 框架，受 Elm / Bonsai 启发：

- 组件 = 纯函数，状态经 typed message 更新（The Elm Architecture）
- 无 `Any`、无 stringly-typed API
- ~15KB min+gzip（含 streaming VDOM diff + moonc DCE）
- 既可 SSR（`App::render`），也可前端挂载/水合（`mount` / `hydrate`）

**配套脚手架**：`moon install moonbit-community/warren` + `warren new my-project` + `warren dev`（开发模式热更新）。

## full-stack 架构模式

```
├─ app/         共享组件包（纯 UI，js+native+wasm 均构建）
│   └─ home_page.mbt   组件 = (input) -> @rabbita.Val[@html.Html]
├─ cmd/server   后端入口（native；moonback Module + Rabbita SSR）
├─ cmd/browser  前端入口（js；水合/挂载）
└─ 根库包        storage / meta_store / auth（+native，被 cmd 复用）
```

**核心约束**：根库包是 `+native`，前端 JS 水合**不能直接 import**。所以共享组件包 `app/` **只能做纯 UI**，数据必须走后端 REST API。

**最终形态（关键决策）**：SSR + 水合的复杂度高到不值，项目回退 **MPA**——各页面服务端渲染完整 HTML + 内联 JS，`cmd/browser` 水合入口留空（`fn main { () }`，仅保留产出构建产物）。官方技能未提示这一抉择，是实战里最贵的教训之一。

## 依赖声明

```toml
# moon.mod（根）
import {
  "moonbitlang/async@0.21.0",
  "moonbit-community/cmark@0.4.6",
  "moonbitlang/x@0.5.1",
  "moonbit-community/rabbita@0.15.4",
  "hackwaly/moonback@0.8.1",
}
```

```toml
# app/moon.pkg（共享组件包，纯 UI）
import {
  "moonbit-community/rabbita",
  "moonbit-community/rabbita/html",
  "moonbit-community/rabbita/http",
  "moonbit-community/rabbita/sub",
  "moonbitlang/core/json",
}
supported_targets = "js+native+wasm"
```

```toml
# cmd/server/moon.pkg（后端入口）
import {
  "moonbit-community/rabbita",
  "hackwaly/moonback",
  "hackwaly/moonback/middlewares/unstable_static" @static,
  "moonbitlang/async",
  "moonbitlang/async/fs",
  "moonbitlang/async/http",
  "moonbitlang/core/argparse",
  "moonbitlang/core/env",
  "moonbitlang/core/encoding/utf8",
  "moonbitlang/core/json",
  "moonbitlang/core/string",
  "conglinyizhi/rootwarren" @lib,
  "conglinyizhi/rootwarren/app",
}
supported_targets = "native+wasm"
pkgtype(kind: "executable")
```

> static 中间件路径为 `hackwaly/moonback/middlewares/unstable_static`（习惯上别名 `@static`）。

## 核心 API 速查

### 组件

```mbt
///| 组件 = 返回 @rabbita.Val[@html.Html] 的函数（input 闭包捕获）
pub fn home_page(input~ : HomeInput) -> @rabbita.Val[@html.Html] {
  let (state, _set) = @rabbita.create_pure_state(
    input,
    update=fn(model, _msg) { model },   // 纯展示组件
  )
  state.view(model => { ... })          // 渲染完整 Html
}
```

- 组件签名：`(input) -> @rabbita.Val[@html.Html]`
- `@rabbita.create_pure_state` 包纯状态（无 Cmd）；交互组件用 `create_state` / `elmish`
- `@rabbita.elmish(model~, view~, update~, subscriptions?)` Elm 风格完整组件

### 注入 Markdown 渲染的 HTML（关键）

```mbt
@html.div(
  attrs=@html.Attrs::build().inner_html(model.doc_html),  // ✅ RawHtml
  @html.nothing,                                          // ✅ 空 children
)
```

- `@html.Attrs::build().inner_html(<html>)` 把后端 cmark 渲染的 Markdown HTML 作为 `@vdom.RawHtml` 注入
- `innerHTML` key 在 `Html::node` / `from_vnode` 里被识别为 `RawHtml`
- `@html.nothing` 作空 children；`@html.ul([])` 传空数组亦可

### 顶层 re-export

```mbt
// @rabbita 顶层 re-export @cmd 与 @html
#cfg(target="js") pub using @cmd {perform, attempt, effect}
pub using @cmd {none, batch, delay, type Cmd}   // @rabbita.none = 空命令
pub using @html {type Html}
```

`@rabbita.none` 是 `@cmd.none`（空命令，在 update 里返回），**不是**空 HTML children。

### SSR + 挂载

```mbt
// 后端（native）：@rabbita.new(component).render(...) 产 HTML 字符串
#warnings("-alert_experimental")
let page = @rabbita.new(fn() { @app.home_page(input = input) })  // 闭包捕获输入
let html = page.render(url = "http://127.0.0.1/", timeout = 5000)
res.send_html(@moonback.Html::raw(html))

// 前端（js）：挂载 / 水合
@rabbita.new(counter).mount("app")   // mount 到 DOM
page.hydrate()                       // 水合（如走 SSR+水合模式）
```

## 失败经验 / 坑（⚠️）

1. **SSR 无法直接注入 input**：`@rabbita.new(component)` 的 `component : () -> Val[Html]` **无参**。要传服务端采集的数据，用**闭包捕获** `fn() { @app.home_page(input = input) }`，或 `create_state_with_input`。
2. **SSR 无 on_mount / 不能自动发 Cmd**：SSR 首屏无法靠组件内部自动加载数据。首屏带数据**必须在服务端预取**后经 input 注入。
3. **`App::render` / `hydrate` 是 `#internal(experimental)`**：不处理会告警 `alert_experimental`。在入口加 `#warnings("-alert_experimental")` 压制。
4. **static 中间件冲突**：`@static.new(root="public")` 挂中间件；但 rabbit server 的 `page_service` 也会按 `dist` 挂 static，**prod 模式 `dist=None` 时并不挂**。注入的静态资源路径（`/styles.css`、`/index.js`）与自定义路径（`/site.css`）要统一；`public/index.js` 是构建产物，`.gitignore` 忽略、由构建生到 `public/`。
5. **水合弃用（MPA 回退）**：SSR + 水合要同时维护水合 transcript、state 注入、static、prefetch，收益不匹配 → 只保留 SSR 完整 HTML + 内联 JS。
6. **`#internal(experimental)` 前缀**：Rabbita 大量 API 带该前缀，属「不稳定但可用」，非语义版本号建议，别因它是 experimental 就绕开——它已是当前 full-stack SSR 的必要上游。

## moonback 后端框架（Express 级，官方技能未提及）

moonback（`hackwaly/moonback`）是 MoonBit 原生异步 Web 后端框架，**native target**。它**修正了补丁20「无独立后端框架」的结论**：`moonbitlang/async` 确实无框架，但 moonback 提供了 Express 级路由/中间件/依赖注入。

```mbt
#warnings("-alert_experimental")
async fn main {
  let api = @moonback.Module(ctx => {
    ctx.add_middleware(@static.new(root="public"))
    ctx.get("/", (req, res) => {
      let page = @rabbita.new(fn() { @app.home_page(input = input) })
      let html = page.render(url = "http://127.0.0.1/", timeout = 5000)
      res.send_html(@moonback.Html::raw(html))
    })
    ctx.get("/*path", (_req, res) => { res.send_html(..., status=404) })
    // ctx.post / ctx.add_route / ctx.use_(module) / ctx.mount(app) / ctx.add_middleware
  })
  let app = @moonback.App(api)
  let listener = @moonback.listen(reuse_addr=true, port=port)
  app.serve(listener)
}
```

- `@moonback.App(module)`、`@moonback.listen(port~, reuse_addr~)`、`app.serve(listener)`、`app.close()`
- `ModuleContext`：`get` / `post` / `add_route` / `add_middleware` / `use_` / `mount` / `config` / `on_close`
- `request`：`.body.text()/.binary()/.json()`；`responder`：`send_text` / `send_html`（配 `@moonback.Html::raw`）等
- `@static.new(root="public")` 静态中间件（`hackwaly/moonback/middlewares/unstable_static`）
- 依赖注入 `TypedKey`；typed query / cookie 帮助；`ctx.on_close` 优雅退出钩子

## SSG / 静态渲染（MPA 落地）额外坑

用 Rabbita 做**静态站点生成（SSG / MPA）**（各页 SSR 完整 HTML，GitHub Pages 免后端）时：

- **`@rabbita.new(fn(){ page() }).render(url, timeout)` 产完整 HTML 字符串**（含 `<!DOCTYPE>`）。
  SSG 直接写盘成 `.html`，无需 moonback。组件的 input 用闭包捕获 `fn(){ page(input = input) }`（同坑1）。
- **`@fs.read_file` 返回 `&@io.Data`（不是 String）**：转 String 用 `data.text()`（raise），
  且 **read_file 是 async** —— helper 得是 `async fn`。用 `@io.Data::text`。
- **`@fs.write_file(path, String)` 直接传 String 值**（String 实现 `@io.Data`），不用 `&string`。
- **`@fs.mkdir(path, recursive = true)`** 确保父目录（write_file 不自动建目录）。
- **`catch { _ => "..." }`**：catch 分支类型须与 try 的 ok 类型一致；忽略错误用 `_ =>`。
- **moon.work 嵌套项目**：子项目（如 `site/`）挂父库用 **`..`**（指向库根），
  不要 `../moonbit-css-helper`（会多一层，报 No such file）。
- **`moon add` 一次一个模块**：`moon add <module>@<ver>`（单数），不能一次列多个。
- **rabbit SSR 的 `<head>` 不带 `<meta charset="utf-8">`**：`@html.node("head", ...)` 生成的
  head 只有你放的内容 + `__rabbita_transcript` script，**没有 charset 声明**。SSG 写盘前
  用字符串 `html.replace(old="<head>", new="<head><meta charset=\"utf-8\">")` 注入，
  否则中文/特殊字符在浏览器可能按默认编码乱码。rabbit `Attrs` 没有 `charset` 方法。
- 产出的 `.html` 里 `<link rel="stylesheet" href="tailwind.css">` 引用**同目录** CSS；
  `<script id="__rabbita_transcript">` 是 SSR 附加，对静态页无害。

## 相关工具链坑（详见 patches.md 补丁23）

- `MOON_CC` 环境变量（native backend 需 C 驱动，工具链可能找 `/usr/bin/lib.exe`）
- 本地代理导致 `moon` / SSR 请求超时 → `--noproxy '*'`
- 帧路由 SSR 端口默认 8080，dev 模式走 `WARREN_MODE` / `WARREN_PORT` 环境变量
