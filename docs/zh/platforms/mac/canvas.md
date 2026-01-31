---
title: Canvas 画布
summary: "通过 WKWebView 和自定义 URL scheme 嵌入的 Agent 代理控制 Canvas 面板"
read_when:
  - 实现 macOS Canvas 面板
  - 为可视化工作区添加智能体控制
  - 调试 WKWebView 中的 Canvas 加载
---

<div id="canvas-macos-app">
  # Canvas（macOS 应用）
</div>

macOS 应用通过 `WKWebView` 嵌入一个由智能体控制的 **Canvas 面板**。它是一个轻量级可视化工作区，用于 HTML/CSS/JS、A2UI 以及小型交互式 UI 区域。

<div id="where-canvas-lives">
  ## Canvas 所在位置
</div>

Canvas 状态保存在 Application Support 目录中：

- `~/Library/Application Support/OpenClaw/canvas/<session>/...`

Canvas 面板通过一个**自定义 URL scheme**来访问这些文件：

- `openclaw-canvas://<session>/<path>`

示例：

- `openclaw-canvas://main/` → `<canvasRoot>/main/index.html`
- `openclaw-canvas://main/assets/app.css` → `<canvasRoot>/main/assets/app.css`
- `openclaw-canvas://main/widgets/todo/` → `<canvasRoot>/main/widgets/todo/index.html`

如果根目录下不存在 `index.html`，应用会显示一个**内置模板页面**。

<div id="panel-behavior">
  ## 面板行为
</div>

- 无边框且可调整大小的面板，固定在菜单栏附近（或鼠标光标附近）。
- 按会话分别记住面板的大小和位置。
- 本地 Canvas 文件发生变更时自动重新加载。
- 任意时刻只显示一个 Canvas 面板（按需切换会话）。

可以在 Settings → **Allow Canvas** 中禁用 Canvas。禁用后，Canvas
节点命令会返回 `CANVAS_DISABLED`。

<div id="agent-api-surface">
  ## Agent API 接口
</div>

Canvas 通过 **Gateway WebSocket** 暴露给智能体，因此智能体可以：

* 显示/隐藏面板
* 导航到某个路径或 URL
* 执行 JavaScript
* 捕获快照图像

CLI 示例：

```bash
openclaw nodes canvas present --node <id>
openclaw nodes canvas navigate --node <id> --url "/"
openclaw nodes canvas eval --node <id> --js "document.title"
openclaw nodes canvas snapshot --node <id>
```

Notes:

* `canvas.navigate` 接受 **本地 Canvas 路径**、`http(s)` URL 和 `file://` URL。
* 如果你传入 `"/"`，Canvas 会显示本地脚手架或 `index.html`。


<div id="a2ui-in-canvas">
  ## Canvas 中的 A2UI
</div>

A2UI 由 Gateway 的 Canvas 主机提供，并在 Canvas 面板内渲染。
当 Gateway 发布一个 Canvas 主机时，macOS 应用会在首次打开时自动跳转到
A2UI 主机页面。

默认的 A2UI 主机 URL：

```
http://<gateway-host>:18793/__openclaw__/a2ui/
```


<div id="a2ui-commands-v08">
  ### A2UI 命令（v0.8）
</div>

Canvas 当前支持接收 **A2UI v0.8** 服务器→客户端消息：

* `beginRendering`
* `surfaceUpdate`
* `dataModelUpdate`
* `deleteSurface`

不支持 `createSurface`（v0.9）。

CLI 示例：

```bash
cat > /tmp/a2ui-v0.8.jsonl <<'EOFA2'
{"surfaceUpdate":{"surfaceId":"main","components":[{"id":"root","component":{"Column":{"children":{"explicitList":["title","content"]}}}},{"id":"title","component":{"Text":{"text":{"literalString":"Canvas (A2UI v0.8)"},"usageHint":"h1"}}},{"id":"content","component":{"Text":{"text":{"literalString":"If you can read this, A2UI push works."},"usageHint":"body"}}}]}}
{"beginRendering":{"surfaceId":"main","root":"root"}}
EOFA2

openclaw nodes canvas a2ui push --jsonl /tmp/a2ui-v0.8.jsonl --node <id>
```

快速冒烟测试：

```bash
openclaw nodes canvas a2ui push --node <id> --text "Hello from A2UI"
```


<div id="triggering-agent-runs-from-canvas">
  ## 从 Canvas 触发智能体运行
</div>

Canvas 可以通过深度链接触发新的智能体运行：

* `openclaw://agent?...`

示例（在 JS 中）：

```js
window.location.href = "openclaw://agent?message=Review%20this%20design";
```

除非提供了有效密钥，否则应用会提示进行确认。


<div id="security-notes">
  ## 安全说明
</div>

- Canvas URL scheme 会阻止目录遍历；文件必须位于会话根目录之下。
- 本地 Canvas 内容使用自定义 URL scheme（不需要本地回环服务器）。
- 外部 `http(s)` URL 只有在被显式导航时才允许访问。