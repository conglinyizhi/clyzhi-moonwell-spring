# Rabbita / moonback 全栈网站专项流程

## 适用场景

从零搭建 Rabbita 前端、SSR 页面和 moonback native 后端。这里是专项 playbook，不在总导航展开 API 细节。

## 先问提督

如果需求是“从零搭网站”或“调试 Rabbita 开发体验”，先问：

> 是否要跑完整开发流程？包括 `moonbit-community/warren` 的 `warren dev` 热更新；如果只要一次性检查/构建，则不启动开发服务器。

## 工具入口

Mooncakes 当前可查询到：

```text
moonbit-community/rabbita
moonbit-community/warren
hackwaly/moonback
```

Warren 是 Rabbita 应用的开发工具，常见入口为：

```bash
moon search rabbita
moon install moonbit-community/warren@<version>
warren new <project>
warren dev
```

本机已核验 Warren 提供 `new`、`dev`、`build`；版本和确切 CLI 参数使用前仍以 Mooncakes、包 README 和本地命令为准。

## 推荐阶段

1. 先决定 SSR only、SSR + hydration 或 MPA
2. 用 Warren 建立开发骨架；确认是否需要热更新
3. 把共享 `app/` 保持为跨 target 的纯 UI 包
4. 后端 `cmd/server` 使用 native + moonback，先跑最小 endpoint
5. SSR 首屏数据由服务端预取，再通过闭包 input 注入组件
6. 再决定是否维护浏览器端 hydration；复杂度不值时回退为 SSR/MPA
7. 检查静态资源、字符集、404 路由和生产构建

## 常见失败表现

- **SSR 页面渲染成功但首屏数据为空**：`new(component)` 需要无参组件，服务端数据要通过闭包捕获 input；不能指望 SSR 内部自动 `on_mount` 发 Cmd
- **页面资源 404 或开发/生产路径不一致**：static 中间件、`dist`、`public` 和构建产物路径没有统一
- **中文或特殊字符乱码**：静态 HTML 的 `<head>` 缺少 charset
- **一开始就做 hydration，开发长期卡在 transcript / state 注入**：先做 SSR/MPA vertical slice，再评估 hydration
- **把 native 库包直接给 JS 前端 import**：共享 `app/` 与 native 后端库边界混了，数据应通过 API 传递
- **`App::mount` 抛 `$PanicError`、页面全白**：栈里是 `Nullable::unwrap` → `VDom::initialize`。参数是**裸 id**，内部走 `document.getElementById`，写 `"#app"` 查不到元素就直接 abort。正确写法 `app.mount("app")`

## 验收

```text
warren dev（若选择完整开发流程）
→ native check
→ 最小 API endpoint
→ SSR HTML 含预期数据
→ 静态资源可访问
→ 浏览器端构建（若启用）
→ 生产构建与 404 路由
```

具体 API、依赖版本和历史细节应从当前包 `.mbti`、README 和 Mooncakes 页面重新核验。

## 深入排障与历史细节

关于 `App::render` / `hydrate`、SSR input、`inner_html`、静态中间件、SSG 和字符集的完整历史笔记见：

```text
../history/rabbita-fullstack-notes.md
```

先按本文件完成最小流程；只有命中对应错误表现时再读历史笔记。
