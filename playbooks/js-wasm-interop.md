# MoonBit 产物接入 Node / JS 专项流程

## 适用场景

把 MoonBit 编译产物接进 Node 运行时：pi 扩展、CLI 辅助进程、脚本库。
触发词是 `#export_name`、`extern "js"`、`pkgtype(kind: "foreign_library")`，
或导入 js / wasm 产物时报 `Cannot find package`、`type incompatibility`。

## 先选 target

- **js**：产物是 ESM，Node 直接 `import`。与 JS SDK 的胶水层用它。
  速度不占优（实测 1.0–1.4x 于手写 JS），换来的是类型安全
- **wasm-gc**：进程内、无 IPC、字符串零拷贝直通。纯计算热点用它
  （实测字符扫描类负载比手写 JS 快约 1.6x，单次调用无编组税）
- **native**：只能出可执行文件，进程内接不进 Node。
  官方明确 native 后端不支持把 `foreign_library` 导出为库产物，需要导出库时只能用 Wasm 或 JavaScript

选 native 前先算跨进程账：每次 spawn 实测约 1.5ms，常驻子进程管道往返约 52µs，
进程内 JS 直调约 1.5µs。单次调用输入小于几十 KB 时不划算。

## js target 配方

`moon.pkg`：

```text
supported_targets = "js"
pkgtype(kind: "foreign_library")
```

- `#export_name("名")` 加在 `pub fn` 上即导出；产物是 ESM，并生成 `.d.ts`
- 导出的 `pub async fn` 是 CPS 形态（多收 `_cont` / `_err_cont`），JS 侧不能直接 `await`
- 要给 JS 一个可 await 的 Promise，另包一层：

```text
pub fn f_js(s : String) -> @js_async.Promise[String] {
  @js_async.Promise::from_async(() => { f(s) })
}
```

- 反向在 MoonBit 里 await JS Promise：`@js_async.Promise::wait()`
- async 箭头函数里不要写 `async`——`async` 只能放在 `fn` 前，async-ness 按上下文推断

## wasm-gc target 配方

`moon.pkg`，两个 link 选项必须同时给：

```text
supported_targets = "js+wasm-gc"
pkgtype(kind: "foreign_library")
options("link": {"wasm-gc": {"use-js-builtin-string": true, "imported-string-constants": "wasm:js/string-constants"}})
```

Node 侧不加任何 flag：

```js
import { mb_scan } from "./bench.wasm";
```

- `imported-string-constants` 必须写 `"wasm:js/string-constants"`。
  官方 skill 示例值 `"_"` 在 Node 下会抛 `Cannot find package '_'`
- 必须走 ESM `import`。`WebAssembly.compile` / `instantiate` 那条路 builtins 是关的，
  模块里的 `wasm:js-string` 导入会变成 `TypeError: ... module is not an object or function`
- Node 自 v24.5.0 / v22.19.0 起由 ESM Integration 自动启用 JS String Builtins；
  当前 CLI 文档已无 `--experimental-wasm-modules`，`.wasm` 是原生 ESM 扩展名
- 运行时会打一条 `ExperimentalWarning: Importing WebAssembly module instances is an experimental feature`，不影响功能

## 跨边界类型

- 直通（零开销）：`Int` / `UInt` / `Double` / `Bool` / `String` 作参数与返回
- js target 下 `FixedArray<T>` 就是 JS 数组
- wasm-gc 下数组、元组、结构体返回不透明 wasm 引用（`[Object: null prototype] {}`），
  要自己导出取长度 / 取下标的函数做编组
- wasm-gc 不生成 `.d.ts`，导出签名需自行维护

## JS 对象互操作

把 JS 对象（如 SDK 句柄）传进 MoonBit：

```text
#external
pub type JsObject

extern "js" fn js_call1(o : JsObject, m : String, a : String) -> String = "(o,m,a) => o[m](a)"
```

- `#external` 声明不透明 JS 类型；`extern "js"` 的函数体是 JS 源码字符串
- 回调和闭包直通，产物就是原生箭头函数
- extern 没有静态类型检查，边界签名要自己保证。只放一层薄胶水，业务逻辑仍写在 MoonBit 里

## 验收

- 必须用 `--release`。js target 的 debug 产物把 `let mut` 编成堆装箱（`MutLocal`），
  实测比手写 JS 慢 1.38x；release 消除装箱后为 1.24x
- wasm-gc 造一个 `import` 直通用例，与手写 JS 参考实现逐条比对返回值，不要只硬编码期望值

## 相关

- async 的协程运行时会整体内联进 js 产物：实测 20 行源码的 async 导出生成 1,599 行 JS。
  纯逻辑模块不要为了省事引入 async
- link 配置全貌看官方 `moonbit-agent-guide` 的 `references/advanced-moonbit-build.md`；
  C 方向看官方 `moonbit-c-binding`
