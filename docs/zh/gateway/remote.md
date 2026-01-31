---
title: 远程访问
summary: "通过 SSH 隧道（Gateway WS）和 tailnet 网络实现远程访问"
read_when:
  - 运行或排查远程 Gateway 部署时
---

<div id="remote-access-ssh-tunnels-and-tailnets">
  # 远程访问（SSH、隧道和 tailnet）
</div>

该仓库通过在一台专用主机（桌面/服务器）上运行单个 Gateway（主 Gateway），并让客户端连接到它的方式，支持“通过 SSH 的远程访问”。

* 对于**运维人员（你 / macOS 应用）**：SSH 隧道是通用的后备方案。
* 对于**节点（iOS/Android 和未来设备）**：通过 **WebSocket** 连接到 Gateway（根据需要使用局域网/tailnet 或 SSH 隧道）。

<div id="the-core-idea">
  ## 核心概念
</div>

* Gateway 的 WebSocket 会在你配置的端口（默认为 18789）上绑定到 **loopback** 回环接口。
* 若要远程使用，你可以通过 SSH 转发该 loopback 端口（或使用 tailnet/VPN，减少额外的隧道转发）。

<div id="common-vpntailnet-setups-where-the-agent-lives">
  ## 常见的 VPN／tailnet 部署方式（智能体所在位置）
</div>

可以把 **Gateway 主机** 看作是“智能体所在的地方”。它管理会话、认证配置、渠道和状态。
你的笔记本／台式机（以及节点）会连接到这台主机。

<div id="1-always-on-gateway-in-your-tailnet-vps-or-home-server">
  ### 1) 你的 tailnet 中的常开 Gateway（VPS 或家用服务器）
</div>

在一台常驻运行的主机上运行 Gateway，通过 **Tailscale** 或 SSH 访问它。

* **最佳体验：** 保持 `gateway.bind: "loopback"`，并使用 **Tailscale Serve** 提供 Control UI 访问。
* **备用方案：** 保持 loopback 配置 + 从任何需要访问的机器建立 SSH 隧道。
* **示例：** [exe.dev](/zh/platforms/exe-dev)（简易虚拟机）或 [Hetzner](/zh/platforms/hetzner)（生产用 VPS）。

当你的笔记本电脑经常休眠，但你希望智能体始终在线时，这是理想方案。

<div id="2-home-desktop-runs-the-gateway-laptop-is-remote-control">
  ### 2）家里的台式机运行 Gateway，笔记本电脑作为远程控制端
</div>

笔记本电脑**不**运行智能体。它通过远程连接：

* 使用 macOS 应用的 **Remote over SSH** 模式（Settings → General → “OpenClaw runs”）。
* 应用会打开并管理隧道，因此 WebChat 和健康检查即可正常工作。

运行手册：[macOS 远程访问](/zh/platforms/mac/remote)。

<div id="3-laptop-runs-the-gateway-remote-access-from-other-machines">
  ### 3) 在笔记本电脑上运行 Gateway，从其他机器远程访问
</div>

让 Gateway 保持本地运行，但安全地提供远程访问：

* 从其他机器通过 SSH 隧道连接到该笔记本电脑，或
* 使用 Tailscale Serve 暴露 Control UI，并让 Gateway 仅绑定到回环地址。

指南：[Tailscale](/zh/gateway/tailscale) 和 [Web 概览](/zh/web)。

<div id="command-flow-what-runs-where">
  ## 命令流（在哪里运行什么）
</div>

一个 Gateway 服务负责管理状态和通道。节点是外围设备。

流程示例（Telegram → 节点）：

* Telegram 消息到达 **Gateway**。
* Gateway 运行 **智能体（agent）**，并决定是否需要调用节点工具。
* Gateway 通过 Gateway WebSocket（`node.*` RPC）调用 **节点（node）**。
* 节点返回结果；Gateway 再把回复发回 Telegram。

注意：

* **节点不运行 Gateway 服务。** 每台主机上应只运行一个 Gateway 实例，除非你有意运行彼此隔离的配置文件（参见 [Multiple gateways](/zh/gateway/multiple-gateways)）。
* macOS 应用的“node 模式”其实只是一个通过 Gateway WebSocket 连接的 node 客户端。

<div id="ssh-tunnel-cli-tools">
  ## SSH 隧道（CLI 和工具）
</div>

创建指向远程 Gateway WS 的本地隧道：

```bash
ssh -N -L 18789:127.0.0.1:18789 user@host
```

在隧道建立后：

* `openclaw health` 和 `openclaw status --deep` 现在会通过 `ws://127.0.0.1:18789` 访问远程 Gateway。
* `openclaw gateway {status,health,send,agent,call}` 在需要时也可以通过 `--url` 指向该转发后的 URL。

注意：将 `18789` 替换为你配置的 `gateway.port`（或 `--port`/`OPENCLAW_GATEWAY_PORT`）。

<div id="cli-remote-defaults">
  ## CLI 远程默认配置
</div>

你可以保存一个远程目标，这样 CLI 命令就会默认使用它：

```json5
{
  gateway: {
    mode: "remote", // 模式:"remote"
    remote: {
      url: "ws://127.0.0.1:18789",
      token: "your-token"
    }
  }
}
```

当 Gateway 仅绑定到回环地址时，将 URL 保持为 `ws://127.0.0.1:18789`，并先打开 SSH 隧道。

<div id="chat-ui-over-ssh">
  ## 通过 SSH 使用聊天 UI
</div>

WebChat 不再使用单独的 HTTP 端口。SwiftUI 聊天 UI 直接连接到 Gateway 的 WebSocket。

* 通过 SSH 转发端口 `18789`（见上文），然后让客户端连接到 `ws://127.0.0.1:18789`。
* 在 macOS 上，优先使用应用的 “Remote over SSH” 模式，该模式会自动管理隧道。

<div id="macos-app-remote-over-ssh">
  ## macOS 应用 “Remote over SSH”
</div>

macOS 菜单栏应用可以以相同方式完成端到端操作（远程状态检查、WebChat，以及语音唤醒请求转发）。

操作手册：[macOS 远程访问](/zh/platforms/mac/remote)。

<div id="security-rules-remotevpn">
  ## 安全规则（远程/VPN）
</div>

简要说明：除非你非常确定需要对外绑定，否则**让 Gateway 只监听回环地址（loopback-only）**。

* **回环 + SSH/Tailscale Serve** 是最安全的默认方案（无公开暴露）。
* **非回环绑定**（`lan`/`tailnet`/`custom`，或在回环不可用时使用的 `auto`）必须使用令牌/密码进行身份验证。
* `gateway.remote.token` **仅**用于远程 CLI 调用——它**不会**启用本地认证。
* 使用 `wss://` 时，`gateway.remote.tlsFingerprint` 用于固定远程 TLS 证书（证书指纹钉扎）。
* 当 `gateway.auth.allowTailscale: true` 时，**Tailscale Serve** 可以通过身份标头（identity headers）进行认证。
  如果你想改用令牌/密码，将其设置为 `false`。
* 把浏览器控制界面视为操作员访问：仅允许 tailnet 范围 + 明确的节点配对。

深入阅读：[Security](/zh/gateway/security)。