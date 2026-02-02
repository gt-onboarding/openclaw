---
title: 远程
summary: "使用 macOS 应用通过 SSH 控制远程 OpenClaw Gateway 的流程"
read_when:
  - 在配置或调试通过 Mac 进行远程控制时
---

<div id="remote-openclaw-macos-remote-host">
  # 远程 OpenClaw（macOS ⇄ 远程主机）
</div>

此模式允许 macOS 应用作为完整的远程控制端，控制运行在另一台主机（桌面/服务器）上的 OpenClaw Gateway。这就是应用的 **Remote over SSH**（通过 SSH 远程运行）功能。所有功能——健康检查、语音唤醒转发和 Web Chat——都会复用 *Settings → General*（“设置 → 通用”）中配置的同一套远程 SSH 设置。

<div id="modes">
  ## 模式
</div>

* **本地（此 Mac）**：所有内容都在本机运行，无需 SSH。
* **通过 SSH 的远程（默认）**：OpenClaw 命令在远程主机上执行。Mac 应用会通过 SSH 建立连接，使用 `-o BatchMode` 选项、你选择的身份/密钥，以及本地端口转发。
* **直接远程（ws/wss）**：无需 SSH 隧道。Mac 应用直接连接到 Gateway 的 URL（例如，通过 Tailscale Serve 或公共 HTTPS 反向代理）。

<div id="remote-transports">
  ## 远程传输方式
</div>

远程模式支持两种传输方式：

* **SSH 隧道**（默认）：使用 `ssh -N -L ...` 将 Gateway 端口转发到本地 localhost。由于隧道是本地回环连接，Gateway 看到的节点 IP 地址为 `127.0.0.1`。
* **Direct (ws/wss)**：直接连接到 Gateway 的 URL。Gateway 会看到真实的客户端 IP 地址。

<div id="prereqs-on-the-remote-host">
  ## 远程主机上的前提条件
</div>

1. 安装 Node.js 和 pnpm，并构建/安装 OpenClaw CLI（`pnpm install && pnpm build && pnpm link --global`）。
2. 确保在非交互式 shell 中 `openclaw` 位于 PATH 中可被找到（如有需要，将其符号链接到 `/usr/local/bin` 或 `/opt/homebrew/bin`）。
3. 启用基于密钥认证的 SSH 访问。我们推荐使用 **Tailscale** 的 IP，以便在离开局域网时也能保持稳定连通性。

<div id="macos-app-setup">
  ## macOS 应用设置
</div>

1. 打开 *Settings → General*。
2. 在 **OpenClaw runs** 下，选择 **Remote over SSH** 并进行如下设置：
   * **Transport**：**SSH tunnel** 或 **Direct (ws/wss)**。
   * **SSH target**：`user@host`（可选 `:port`）。
     * 如果 Gateway 与本机在同一局域网并通过 Bonjour 广播，你可以在已发现列表中选择它以自动填充该字段。
   * **Gateway URL**（仅 Direct）：`wss://gateway.example.ts.net`（本地/局域网可用 `ws://...`）。
   * **Identity file**（高级）：你的密钥文件路径。
   * **Project root**（高级）：用于执行命令的远程检出路径。
   * **CLI path**（高级）：可选的可执行 `openclaw` 入口/二进制路径（通过广播发现时会自动填充）。
3. 点击 **Test remote**。成功表示远程 `openclaw status --json` 运行正常。失败通常意味着 PATH/CLI 问题；返回码 127 表示在远程找不到 CLI。
4. 健康检查和 Web Chat 现在会自动通过此 SSH 隧道运行。

<div id="web-chat">
  ## Web Chat
</div>

* **SSH 隧道**：Web Chat 通过转发的 WebSocket 控制端口（默认 18789）连接到 Gateway。
* **直接连接（ws/wss）**：Web Chat 直接连接到配置的 Gateway URL。
* 现在不再有单独的 Web Chat HTTP 服务器。

<div id="permissions">
  ## 权限
</div>

* 远程主机需要与本机相同的 TCC 授权（Automation、Accessibility、Screen Recording、Microphone、Speech Recognition、Notifications）。在那台机器上运行引导流程以一次性授予这些权限。
* 节点会通过 `node.list` / `node.describe` 上报其权限状态，以便智能体了解当前可用的权限/能力。

<div id="security-notes">
  ## 安全注意事项
</div>

* 在远程主机上优先只绑定到回环地址，并通过 SSH 或 Tailscale 进行连接。
* 如果你将 Gateway 绑定到非回环接口，务必启用令牌/密码身份验证。
* 参见 [Security](/zh/gateway/security) 和 [Tailscale](/zh/gateway/tailscale)。

<div id="whatsapp-login-flow-remote">
  ## WhatsApp 登录流程（远程）
</div>

* **在远程主机上**运行 `openclaw channels login --verbose`。然后用你手机上的 WhatsApp 扫描二维码。
* 如果授权过期，在该主机上重新执行登录命令。健康检查会显示连接问题。

<div id="troubleshooting">
  ## 故障排查
</div>

* **exit 127 / not found**：`openclaw` 不在非登录 shell 的 PATH 中。将其添加到 `/etc/paths`、你的 shell 启动脚本（rc），或为其在 `/usr/local/bin` / `/opt/homebrew/bin` 下创建符号链接。
* **Health probe failed**：检查 SSH 连通性、PATH，以及 Baileys 是否已登录（`openclaw status --json`）。
* **Web Chat 卡住**：确认 Gateway 在远程主机上运行，并且转发的端口与 Gateway 的 WS 端口一致；UI 需要正常的 WS 连接。
* **Node IP 显示 127.0.0.1**：在使用 SSH 隧道时，这是预期行为。如果你希望 Gateway 看到真实的客户端 IP，请将 **Transport** 切换为 **Direct (ws/wss)**。
* **Voice Wake**：在远程模式下，唤醒词会自动转发；不需要单独的转发器。

<div id="notification-sounds">
  ## 通知声音
</div>

通过使用包含 `openclaw` 和 `node.invoke` 的脚本为每条通知设置声音，例如：

```bash
openclaw nodes notify --node <id> --title "Ping" --body "Remote gateway ready" --sound Glass
```

应用中不再提供全局的“默认提示音”开关；调用方需要为每个请求单独选择一个提示音（或不使用提示音）。
