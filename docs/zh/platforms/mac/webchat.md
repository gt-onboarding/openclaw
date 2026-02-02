---
title: WebChat
summary: "macOS 应用如何嵌入 Gateway 的 WebChat，以及如何调试它"
read_when:
  - 调试 macOS WebChat 视图或回环端口
---

<div id="webchat-macos-app">
  # WebChat（macOS 应用）
</div>

macOS 菜单栏应用将 WebChat UI 以原生 SwiftUI 视图的形式嵌入。它会连接到 Gateway，并默认使用所选智能体的**主会话**（同时提供会话切换器用于切换到其他会话）。

- **本地模式**：直接连接本地 Gateway 的 WebSocket。
- **远程模式**：通过 SSH 转发 Gateway 控制端口，并使用该隧道作为数据平面。

<div id="launch-debugging">
  ## 启动与调试
</div>

- 手动：Lobster 菜单 → “Open Chat”。
- 自动启动用于测试：
  ```bash
  dist/OpenClaw.app/Contents/MacOS/OpenClaw --webchat
  ```
- 日志：`./scripts/clawlog.sh`（子系统 `bot.molt`，类别 `WebChatSwiftUI`）。

<div id="how-its-wired">
  ## 内部连接结构
</div>

- 数据平面：Gateway 的 WS 方法 `chat.history`、`chat.send`、`chat.abort`、
  `chat.inject`，以及事件 `chat`、`agent`、`presence`、`tick`、`health`。
- 会话：默认使用主会话（`main`，当 scope 为 global 时则为 `global`）。UI 可以在不同会话之间切换。
- Onboarding 流程使用一个专用会话，以将首次运行设置与其他内容隔离开来。

<div id="security-surface">
  ## 攻击面
</div>

- 远程模式仅通过 SSH 转发 Gateway 的 WebSocket 控制端口。

<div id="known-limitations">
  ## 已知限制
</div>

- UI 针对聊天会话进行了优化（而非完整的浏览器沙箱）。