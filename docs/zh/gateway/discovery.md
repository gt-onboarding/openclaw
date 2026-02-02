---
title: 发现
summary: "节点发现和传输方式（Bonjour、Tailscale、SSH），用于定位 Gateway"
read_when:
  - 实现或更改 Bonjour 发现/广播
  - 调整远程连接模式（直连 vs SSH）
  - 为远程节点设计发现与配对机制
---

<div id="discovery-transports">
  # 发现与传输
</div>

OpenClaw 有两个在表面上相似、但本质不同的问题：

1. **操作员远程控制**：通过 macOS 菜单栏应用控制在其他地方运行的 Gateway。
2. **节点配对**：iOS/Android（以及未来的节点）发现 Gateway 并进行安全配对。

设计目标是将所有网络发现/广播都集中在 **Node Gateway**（`openclaw gateway`）中，并让客户端（macOS 应用、iOS）仅作为消费者。

<div id="terms">
  ## 术语
</div>

* **Gateway**：单个长时间运行的 Gateway 进程，负责持有状态（会话、配对、节点注册表）并运行各类通道。大多数情况下每台主机上运行一个；也支持多个相互隔离的多 Gateway 部署。
* **Gateway WS（控制平面）**：默认监听在 `127.0.0.1:18789` 的 WebSocket 端点；可以通过 `gateway.bind` 绑定到局域网/LAN 或 tailnet。
* **Direct WS 传输**：面向局域网/LAN 或 tailnet 的 Gateway WS 端点（不经 SSH）。
* **SSH 传输（回退方案）**：通过 SSH 转发 `127.0.0.1:18789` 实现远程控制。
* **旧版 TCP bridge（已弃用/已移除）**：旧的节点传输方式（参见 [Bridge protocol](/zh/gateway/bridge-protocol)）；不再用于服务发现。

协议详情：

* [Gateway 协议](/zh/gateway/protocol)
* [Bridge 协议（旧版）](/zh/gateway/bridge-protocol)

<div id="why-we-keep-both-direct-and-ssh">
  ## 为什么同时保留 “direct” 和 SSH
</div>

* **Direct WS** 是在同一网络和 tailnet 内最好的使用体验 (UX)：
  * 通过 Bonjour 在局域网 (LAN) 上自动发现
  * 配对令牌和 ACL 由 Gateway 统一管理
  * 不需要 shell 访问；协议暴露面可以保持精简且便于审计
* **SSH** 仍然是通用的兜底方案：
  * 只要你有 SSH 访问权限就能使用（即使跨互不相关的网络）
  * 可在 multicast/mDNS 出现问题时继续工作
  * 除了 SSH 之外不需要新的入站端口

<div id="discovery-inputs-how-clients-learn-where-the-gateway-is">
  ## 发现输入（客户端如何获知 Gateway 地址）
</div>

<div id="1-bonjour-mdns-lan-only">
  ### 1) Bonjour / mDNS（仅限 LAN）
</div>

Bonjour 采用 best-effort 机制，且不会跨网络工作。它只用于在“同一 LAN”内提供便捷发现。

目标方向：

* **Gateway** 通过 Bonjour 广播其 WS 端点。
* 客户端进行浏览并显示“选择一个 Gateway”的列表，然后存储所选端点。

故障排查和服务信标详情： [Bonjour](/zh/gateway/bonjour)。

<div id="service-beacon-details">
  #### 服务信标详细信息
</div>

* 服务类型：
  * `_openclaw-gw._tcp`（Gateway 传输信标）
* TXT 键（非敏感）：
  * `role=gateway`
  * `lanHost=<hostname>.local`
  * `sshPort=22`（或其他被广播的端口）
  * `gatewayPort=18789`（Gateway WS + HTTP）
  * `gatewayTls=1`（仅在启用 TLS 时）
  * `gatewayTlsSha256=<sha256>`（仅在启用 TLS 且可用指纹时）
  * `canvasPort=18793`（默认 canvas 主机端口；提供 `/__openclaw__/canvas/` 服务）
  * `cliPath=<path>`（可选；指向可运行的 `openclaw` 入口点或二进制文件的绝对路径）
  * `tailnetDns=<magicdns>`（可选提示；在 Tailscale 可用时自动检测）

禁用/覆盖：

* `OPENCLAW_DISABLE_BONJOUR=1` 会禁用服务广播。
* `~/.openclaw/openclaw.json` 中的 `gateway.bind` 控制 Gateway 绑定模式。
* `OPENCLAW_SSH_PORT` 覆盖在 TXT 中广播的 SSH 端口（默认为 22）。
* `OPENCLAW_TAILNET_DNS` 发布 `tailnetDns` 提示（MagicDNS）。
* `OPENCLAW_CLI_PATH` 覆盖广播的 CLI 路径。

<div id="2-tailnet-cross-network">
  ### 2) Tailnet（跨网络）
</div>

对于像伦敦 / 维也纳这种异地多站点部署，Bonjour 就帮不上忙了。推荐的“直接”目标是：

* Tailscale MagicDNS 名称（优先）或一个稳定的 Tailnet IP。

如果 Gateway 能检测到自己运行在 Tailscale 网络中，它会发布 `tailnetDns` 作为给客户端的可选提示信息（包括广域 beacon 信标）。

<div id="3-manual-ssh-target">
  ### 3) 手动 / SSH 目标
</div>

当没有直接路由（或已禁用直连）时，客户端始终可以通过 SSH 转发回环接口上的 Gateway 端口来进行连接。

参见 [远程访问](/zh/gateway/remote)。

<div id="transport-selection-client-policy">
  ## 传输方式选择（客户端策略）
</div>

推荐的客户端行为：

1. 如果已配置且已配对的直连端点可访问，则使用该端点。
2. 否则，如果 Bonjour 在局域网中发现了一个 Gateway，向用户提供“一键使用此 Gateway”的选项，并将其保存为直连端点。
3. 否则，如果已配置 tailnet DNS/IP，则尝试直连。
4. 否则，回退到 SSH。

<div id="pairing-auth-direct-transport">
  ## 配对与认证（直接传输）
</div>

Gateway 是节点 / 客户端接入的最终权威来源。

* 配对请求在 Gateway 中创建 / 批准 / 拒绝（参见 [Gateway 配对](/zh/gateway/pairing)）。
* Gateway 负责强制执行：
  * 认证（token / 密钥对）
  * scope/ACL 规则（Gateway 并不是对所有方法的简单透明代理）
  * 速率限制

<div id="responsibilities-by-component">
  ## 各组件的职责
</div>

* **Gateway**：广播发现用的 beacon 信号，负责配对决策，并托管 WS 端点。
* **macOS app**：帮助你选择 Gateway，展示配对提示，并仅将 SSH 作为回退方案使用。
* **iOS/Android 节点**：通过 Bonjour 方便地浏览服务，并连接到已配对 Gateway 的 WS 端点。