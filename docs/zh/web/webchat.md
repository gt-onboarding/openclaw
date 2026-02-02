---
title: Webchat
summary: "Loopback WebChat 静态托管与在聊天 UI 中使用 Gateway WS"
read_when:
  - 在调试或配置 WebChat 访问时
---

<div id="webchat-gateway-websocket-ui">
  # WebChat（Gateway WebSocket UI）
</div>

当前状态：macOS/iOS SwiftUI 聊天 UI 直接与 Gateway WebSocket 通信。

<div id="what-it-is">
  ## 功能概览
</div>

* Gateway 的原生聊天 UI（无嵌入式浏览器，也无需本地静态服务器）。
* 使用与其他渠道相同的会话和路由规则。
* 确定性路由：回复始终返回 WebChat。

<div id="quick-start">
  ## 快速入门
</div>

1. 启动 Gateway。
2. 打开 WebChat UI（macOS/iOS 应用）或 Control UI 的聊天标签页。
3. 确保已配置 Gateway 认证（默认情况下是必需的，即使只在本机回环地址上使用也是如此）。

<div id="how-it-works-behavior">
  ## 工作原理（行为）
</div>

* UI 连接到 Gateway 的 WebSocket，并使用 `chat.history`、`chat.send` 和 `chat.inject`。
* `chat.inject` 会将一条助手注释直接追加到对话记录中，并将其广播到 UI（不会触发智能体运行）。
* 历史记录始终从 Gateway 获取（不会监视本地文件）。
* 如果 Gateway 无法访问，WebChat 将变为只读。

<div id="remote-use">
  ## 远程使用
</div>

* 远程模式会通过 SSH/Tailscale 为 Gateway 的 WebSocket 建立隧道。
* 你不需要单独运行 WebChat 服务器。

<div id="configuration-reference-webchat">
  ## 配置参考（WebChat）
</div>

完整配置：[Configuration](/zh/gateway/configuration)

通道选项：

* 没有专门的 `webchat.*` 配置块。WebChat 使用 Gateway 端点加下方的认证设置。

相关全局选项：

* `gateway.port`, `gateway.bind`: WebSocket 主机/端口。
* `gateway.auth.mode`, `gateway.auth.token`, `gateway.auth.password`: WebSocket 认证。
* `gateway.remote.url`, `gateway.remote.token`, `gateway.remote.password`: 远程 Gateway 目标。
* `session.*`: 会话存储与主键的默认值。