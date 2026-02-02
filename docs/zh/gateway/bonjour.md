---
title: Bonjour（mDNS）
summary: "Bonjour/mDNS 发现与调试（Gateway 广播、客户端与常见故障模式）"
read_when:
  - 在 macOS/iOS 上调试 Bonjour 发现相关问题
  - 更改 mDNS 服务类型、TXT 记录或发现体验（UX）
---

<div id="bonjour-mdns-discovery">
  # Bonjour / mDNS 发现
</div>

OpenClaw 使用 Bonjour（mDNS / DNS‑SD）作为一种**仅用于局域网（LAN）的便捷机制**来发现
处于活动状态的 Gateway（WebSocket 端点）。它是尽力而为的机制，并且**不会**取代 SSH 或
基于 Tailnet 的连接方式。

<div id="widearea-bonjour-unicast-dnssd-over-tailscale">
  ## 通过 Tailscale 使用广域 Bonjour（单播 DNS‑SD）
</div>

如果节点和 Gateway 位于不同网络，组播 mDNS 无法跨越网络边界。你可以通过在 Tailscale 上切换到 **单播 DNS‑SD**（“广域 Bonjour”）来保持相同的发现体验（discovery UX）。

大致步骤：

1. 在运行 Gateway 的主机上部署一个 DNS 服务器（可通过 Tailnet 访问）。
2. 在一个专用 DNS 区域（zone）下为 `_openclaw-gw._tcp` 发布 DNS‑SD 记录
   （例如：`openclaw.internal.`）。
3. 配置 Tailscale 的 **split DNS**，使你选定的域名在客户端（包括 iOS）上通过该 DNS 服务器解析。

OpenClaw 支持任意发现域名；`openclaw.internal.` 只是一个示例。
iOS/Android 节点会同时浏览 `local.` 和你配置的广域域名。

<div id="gateway-config-recommended">
  ### 推荐的 Gateway 配置
</div>

```json5
{
  gateway: { bind: "tailnet" }, // tailnet-only (recommended)
  discovery: { wideArea: { enabled: true } } // 启用广域 DNS-SD 发布
}
```

<div id="onetime-dns-server-setup-gateway-host">
  ### 一次性 DNS 服务器配置（Gateway 主机）
</div>

```bash
openclaw dns setup --apply
```

这会安装 CoreDNS 并将其配置为：

* 仅在 Gateway 的 Tailscale 接口上监听 53 端口
* 从 `~/.openclaw/dns/<domain>.db` 为你选定的域名（例如：`openclaw.internal.`）提供解析服务

在已连接到 Tailnet 的机器上进行验证：

```bash
dns-sd -B _openclaw-gw._tcp openclaw.internal.
dig @<TAILNET_IPV4> -p 53 _openclaw-gw._tcp.openclaw.internal PTR +short
```

<div id="tailscale-dns-settings">
  ### Tailscale DNS 设置
</div>

在 Tailscale 管理控制台中：

* 添加一个指向 Gateway 的 tailnet IP（UDP/TCP 53）的名称服务器（nameserver）。
* 配置 Split DNS，使你的发现域名使用该名称服务器。

一旦客户端开始使用 tailnet DNS，iOS 节点就可以在你的发现域名中发现并浏览
`_openclaw-gw._tcp`，而无需使用多播。

<div id="gateway-listener-security-recommended">
  ### Gateway 监听安全（推荐）
</div>

Gateway 的 WS 端口（默认 `18789`）默认只绑定到回环地址。要通过局域网 / tailnet 访问时，请显式绑定，并保持身份认证开启。

对于仅限 tailnet 的部署：

* 在 `~/.openclaw/openclaw.json` 中设置 `gateway.bind: "tailnet"`。
* 重启 Gateway（或重启 macOS 菜单栏应用）。

<div id="what-advertises">
  ## 哪些组件会广播
</div>

只有 Gateway 会发布 `_openclaw-gw._tcp` 服务广播。

<div id="service-types">
  ## 服务类型
</div>

* `_openclaw-gw._tcp` — Gateway 传输广播（供 macOS/iOS/Android 节点使用）。

<div id="txt-keys-nonsecret-hints">
  ## TXT 键（非敏感提示）
</div>

Gateway 会广播一些小的非敏感提示，以便让 UI 流程更方便：

* `role=gateway`
* `displayName=<friendly name>`
* `lanHost=<hostname>.local`
* `gatewayPort=<port>`（Gateway WS + HTTP）
* `gatewayTls=1`（仅在启用 TLS 时存在）
* `gatewayTlsSha256=<sha256>`（仅在启用 TLS 且指纹可用时存在）
* `canvasPort=<port>`（仅在启用 canvas 主机时存在；默认值为 `18793`）
* `sshPort=<port>`（未显式覆盖时默认为 22）
* `transport=gateway`
* `cliPath=<path>`（可选；指向可运行 `openclaw` 入口点的绝对路径）
* `tailnetDns=<magicdns>`（Tailnet 可用时的可选提示）

<div id="debugging-on-macos">
  ## 在 macOS 上调试
</div>

一些有用的内置工具：

* 浏览实例：
  ```bash
  dns-sd -B _openclaw-gw._tcp local.
  ```
* 解析某个实例（将 `<instance>` 替换为目标实例名）：
  ```bash
  dns-sd -L "<instance>" _openclaw-gw._tcp local.
  ```

如果浏览正常但解析失败，通常是遇到了局域网策略或 mDNS 解析器问题。

<div id="debugging-in-gateway-logs">
  ## 在 Gateway 日志中进行调试
</div>

Gateway 会写入一个循环日志文件（在启动时会打印为
`gateway log file: ...`）。在其中查找以 `bonjour:` 开头的行，特别是：

* `bonjour: advertise failed ...`
* `bonjour: ... name conflict resolved` / `hostname conflict resolved`
* `bonjour: watchdog detected non-announced service ...`

<div id="debugging-on-ios-node">
  ## 在 iOS 节点上调试
</div>

iOS 节点使用 `NWBrowser` 来发现 `_openclaw-gw._tcp`。

要获取日志：

* 设置 → Gateway → 高级 → **发现调试日志（Discovery Debug Logs）**
* 设置 → Gateway → 高级 → **发现日志（Discovery Logs）** → 复现问题 → **复制**

日志包括浏览器状态变化和结果集变化。

<div id="common-failure-modes">
  ## 常见故障模式
</div>

* **Bonjour 无法跨网络工作**：请使用 Tailnet 或 SSH。
* **多播被阻止**：某些 Wi‑Fi 网络会禁用 mDNS。
* **睡眠 / 网络接口频繁切换**：macOS 可能会暂时丢失 mDNS 结果；请重试。
* **浏览成功但解析失败**：请尽量保持计算机名称简单（避免使用表情符号或标点符号），然后重启 Gateway。服务实例名源自主机名，过于复杂的名称可能会让某些解析器产生问题。

<div id="escaped-instance-names-032">
  ## 转义的实例名称（`\032`）
</div>

Bonjour/DNS‑SD 经常会将服务实例名称中的字节转义为十进制形式的 `\DDD` 序列（例如空格会变成 `\032`）。

* 这在协议层面是正常行为。
* UI 在展示时应对其进行解码（iOS 使用 `BonjourEscapes.decode`）。

<div id="disabling-configuration">
  ## 禁用 / 配置
</div>

* `OPENCLAW_DISABLE_BONJOUR=1` 会禁用服务广播（旧名称：`OPENCLAW_DISABLE_BONJOUR`）。
* `~/.openclaw/openclaw.json` 中的 `gateway.bind` 控制 Gateway 绑定模式。
* `OPENCLAW_SSH_PORT` 会覆盖在 TXT 记录中广播的 SSH 端口（旧名称：`OPENCLAW_SSH_PORT`）。
* `OPENCLAW_TAILNET_DNS` 会在 TXT 记录中发布一个 MagicDNS 提示（旧名称：`OPENCLAW_TAILNET_DNS`）。
* `OPENCLAW_CLI_PATH` 会覆盖广播的 CLI 路径（旧名称：`OPENCLAW_CLI_PATH`）。

<div id="related-docs">
  ## 相关文档
</div>

* 发现策略与传输方式选择：[Discovery](/zh/gateway/discovery)
* 节点配对与审批：[Gateway pairing](/zh/gateway/pairing)