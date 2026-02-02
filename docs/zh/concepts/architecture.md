---
title: 架构
summary: "WebSocket Gateway 架构、组件和客户端流程"
read_when:
  - 在开发 Gateway 协议、客户端或传输通道时
---

<div id="gateway-architecture">
  # Gateway 架构
</div>

最后更新于：2026-01-22

<div id="overview">
  ## 概览
</div>

* 单个长时间运行的 **Gateway** 管理所有消息通道（通过 Baileys 的 WhatsApp、通过 grammY 的 Telegram、Slack、Discord、Signal、iMessage、WebChat）。
* 控制平面客户端（macOS 应用、CLI、Web UI、自动化流程）通过 **WebSocket** 连接到 Gateway，使用配置的绑定主机地址（默认 `127.0.0.1:18789`）。
* **Nodes（节点）**（macOS/iOS/Android/无头模式）同样通过 **WebSocket** 连接，但会声明 `role: node`，并显式声明其能力/命令。
* 每台主机只运行一个 Gateway；只有它会建立 WhatsApp 会话。
* **canvas host**（默认 `18793`）提供可由智能体编辑的 HTML 和 A2UI。

<div id="components-and-flows">
  ## 组件与流程
</div>

<div id="gateway-daemon">
  ### Gateway（守护进程）
</div>

* 维护与提供方的连接。
* 暴露类型化的 WS API（请求、响应、服务器推送事件）。
* 基于 JSON Schema 校验入站数据帧。
* 触发诸如 `agent`、`chat`、`presence`、`health`、`heartbeat`、`cron` 等事件。

<div id="clients-mac-app-cli-web-admin">
  ### 客户端（macOS 应用 / CLI / Web 管理界面）
</div>

* 每个客户端一个 WS 连接。
* 发送请求（`health`、`status`、`send`、`agent`、`system-presence`）。
* 订阅事件（`tick`、`agent`、`presence`、`shutdown`）。

<div id="nodes-macos-ios-android-headless">
  ### 节点（macOS / iOS / Android / 无头环境）
</div>

* 使用 `role: node` 连接到**同一个 WS 服务器**。
* 在 `connect` 中提供设备标识；配对是**基于设备**的（角色为 `node`），
  审批信息保存在设备配对存储中。
* 提供诸如 `canvas.*`、`camera.*`、`screen.record`、`location.get` 等命令。

协议详情：

* [Gateway 协议](/zh/gateway/protocol)

<div id="webchat">
  ### WebChat
</div>

* 基于 Gateway WS API 的静态 UI，用于展示聊天记录并发送消息。
* 在远程部署中，通过与其他客户端相同的 SSH/Tailscale 隧道进行连接。

<div id="connection-lifecycle-single-client">
  ## 连接生命周期（单一客户端）
</div>

```
Client                    Gateway
  |                          |
  |---- req:connect -------->|
  |<------ res (ok) ---------|   (or res error + close)
  |   (payload=hello-ok carries snapshot: presence + health)
  |                          |
  |<------ event:presence ---|
  |<------ event:tick -------|
  |                          |
  |------- req:agent ------->|
  |<------ res:agent --------|   (ack: {runId,status:"accepted"})
  |<------ event:agent ------|   (streaming)
  |<------ res:agent --------|   (final: {runId,status,summary})
  |                          |
```

<div id="wire-protocol-summary">
  ## 线协议（摘要）
</div>

* 传输方式：WebSocket，文本帧承载 JSON 负载。
* 第一帧**必须**是 `connect`。
* 握手完成后：
  * 请求：`{type:"req", id, method, params}` → `{type:"res", id, ok, payload|error}`
  * 事件：`{type:"event", event, payload, seq?, stateVersion?}`
* 如果设置了 `OPENCLAW_GATEWAY_TOKEN`（或 `--token`），则 `connect.params.auth.token`
  必须匹配，否则连接会被关闭。
* 对有副作用的方法（`send`、`agent`）必须提供幂等性键，以便安全重试；服务器会维护一个短期的去重缓存。
* 节点必须在 `connect` 中包含 `role: "node"`，以及能力/命令/权限信息。

<div id="pairing-local-trust">
  ## 配对与本地信任
</div>

* 所有 WS 客户端（操作员和节点）在 `connect` 时都会携带一个**设备标识**。
* 新的设备 ID 需要配对并经批准；Gateway 会签发一个**设备令牌（device token）**
  用于后续连接。
* **本地**连接（回环地址或 Gateway 宿主机自身的 tailnet 地址）可以自动批准，
  以保证同一主机上的使用体验流畅。
* **非本地**连接必须对 `connect.challenge` 随机挑战值（nonce）进行签名，并且需要显式批准。
* Gateway 认证（`gateway.auth.*`）仍然适用于**所有**连接，无论本地还是远程。

详情见：[Gateway 协议](/zh/gateway/protocol)、[配对](/zh/start/pairing)、[安全性](/zh/gateway/security)。

<div id="protocol-typing-and-codegen">
  ## 协议类型与代码生成
</div>

* 使用 TypeBox 模式（schema）定义协议。
* 基于这些模式生成 JSON Schema。
* 基于 JSON Schema 生成 Swift 模型。

<div id="remote-access">
  ## 远程访问
</div>

* 首选：Tailscale 或 VPN。
* 备选：SSH 隧道
  ```bash
  ssh -N -L 18789:127.0.0.1:18789 user@host
  ```
* 通过隧道时，同样使用相同的握手流程和认证令牌。
* 在远程场景中，可以为 WS 启用 TLS 和可选的证书钉扎（pinning）。

<div id="operations-snapshot">
  ## 运行概览
</div>

* 启动：`openclaw gateway`（前台运行，日志输出到 stdout）。
* 健康检查：通过 WS 调用 `health`（也包含在 `hello-ok` 中）。
* 进程管理：使用 launchd/systemd 实现自动重启。

<div id="invariants">
  ## 不变量
</div>

* 在每台主机上，必须且只能有一个 Gateway 控制一个 Baileys 会话。
* 必须先完成握手；任何首帧如果不是 JSON 或不是 `connect`，都将被强制立即断开连接。
* 事件不会回放；一旦发现事件序列存在缺口，客户端必须执行刷新。