---
title: Android
summary: "Android 应用（节点）：连接运行指南 + Canvas/Chat/Camera"
read_when:
  - 配对或重新连接 Android 节点
  - 调试 Android 节点的 Gateway 发现或认证问题
  - 验证各客户端之间聊天记录的一致性
---

<div id="android-app-node">
  # Android 应用（节点）
</div>

<div id="support-snapshot">
  ## 支持概览
</div>

* 角色：辅助节点应用（Android 不托管 Gateway）。
* 是否需要 Gateway：需要（在 macOS、Linux 或通过 WSL2 的 Windows 上运行）。
* 安装：[入门指南](/zh/start/getting-started) + [配对](/zh/gateway/pairing)。
* Gateway：[运维手册](/zh/gateway) + [配置](/zh/gateway/configuration)。
  * 协议：[Gateway 协议](/zh/gateway/protocol)（节点 + 控制平面）。

<div id="system-control">
  ## 系统控制
</div>

系统控制（launchd/systemd）位于 Gateway 所在主机上。请参阅 [Gateway](/zh/gateway)。

<div id="connection-runbook">
  ## 连接运行手册
</div>

Android 节点应用 ⇄ (mDNS/NSD + WebSocket) ⇄ **Gateway**

Android 会直接通过 WebSocket 连接到 Gateway（默认 `ws://<host>:18789`），并使用由 Gateway 管理的配对流程。

<div id="prerequisites">
  ### 先决条件
</div>

* 你可以在“主控”机器上运行 Gateway。
* Android 设备/模拟器可以访问 Gateway 的 WebSocket：
  * 与 Gateway 处于同一局域网并支持 mDNS/NSD，**或者**
  * 处于同一 Tailscale tailnet，并使用 Wide-Area Bonjour / 单播 DNS-SD（见下文），**或者**
  * 手动配置 Gateway 主机/端口（回退方案）
* 你可以在运行 Gateway 的机器上（或通过 SSH 连接到该机器）运行 CLI（`openclaw`）。

<div id="1-start-the-gateway">
  ### 1) 启动 Gateway
</div>

```bash
openclaw gateway --port 18789 --verbose
```

在日志中确认出现类似如下信息：

* `listening on ws://0.0.0.0:18789`

对于仅限 tailnet 的部署（推荐用于 Vienna ⇄ London），将 Gateway 绑定到 tailnet IP：

* 在 Gateway 主机上的 `~/.openclaw/openclaw.json` 中设置 `gateway.bind: "tailnet"`。
* 重启 Gateway / macOS 菜单栏应用。

<div id="2-verify-discovery-optional">
  ### 2) 验证发现机制（可选）
</div>

在 Gateway 机器上：

```bash
dns-sd -B _openclaw-gw._tcp local.
```

更多调试说明，请参见 [Bonjour](/zh/gateway/bonjour)。

<div id="tailnet-vienna-london-discovery-via-unicast-dns-sd">
  #### 通过单播 DNS-SD 进行 Tailnet（Vienna ⇄ London）发现
</div>

Android 的 NSD/mDNS 发现不会跨网络工作。如果你的 Android 节点和 Gateway 位于不同网络，但通过 Tailscale 相连，请改用广域网 Bonjour（Wide-Area Bonjour）/ 单播 DNS-SD：

1. 在 Gateway 主机上设置一个 DNS-SD 区域（例如 `openclaw.internal.`），并发布 `_openclaw-gw._tcp` 记录。
2. 为你选定的域配置 Tailscale 拆分 DNS（split DNS），指向该 DNS 服务器。

更多细节和示例 CoreDNS 配置：参见 [Bonjour](/zh/gateway/bonjour)。

<div id="3-connect-from-android">
  ### 3) 从 Android 连接
</div>

在 Android 应用中：

* 应用通过 **前台服务**（常驻通知）保持与 Gateway 的连接。
* 打开 **Settings**。
* 在 **Discovered Gateways** 下，选择你的 Gateway 并点击 **Connect**。
* 如果 mDNS 被阻止，使用 **Advanced → Manual Gateway**（主机 + 端口），然后选择 **Connect (Manual)**。

首次配对成功后，Android 在启动应用时会自动重连：

* 手动配置的端点（如果已启用），否则
* 上一次发现的 Gateway（在可行的情况下）。

<div id="4-approve-pairing-cli">
  ### 4) 批准配对（CLI）
</div>

在运行 Gateway 的机器上：

```bash
openclaw nodes pending
openclaw nodes approve <requestId>
```

配对详情：[Gateway 配对](/zh/gateway/pairing)。

<div id="5-verify-the-node-is-connected">
  ### 5) 验证节点已连接
</div>

* 通过节点状态命令：
  ```bash
  openclaw nodes status
  ```
* 通过 Gateway：
  ```bash
  openclaw gateway call node.list --params "{}"
  ```

<div id="6-chat-history">
  ### 6) 聊天 + 历史记录
</div>

Android 节点的聊天面板使用 Gateway 的 **主会话 key**（`main`），因此历史记录和回复会在 WebChat 和其他客户端之间共享：

* 历史记录：`chat.history`
* 发送：`chat.send`
* 推送更新（尽力）：`chat.subscribe` → `event:"chat"`

<div id="7-canvas-camera">
  ### 7) Canvas + 摄像头
</div>

<div id="gateway-canvas-host-recommended-for-web-content">
  #### Gateway Canvas 主机（推荐用于网页内容）
</div>

如果你想让节点显示智能体可以在磁盘上编辑的真实 HTML/CSS/JS，就把节点指向 Gateway 的 canvas 主机。

注意：节点通过 `canvasHost.port`（默认 `18793`）使用独立的 canvas 主机。

1. 在 Gateway 主机上创建 `~/.openclaw/workspace/canvas/index.html`。

2. 在节点设备上通过局域网（LAN）打开该地址：

```bash
openclaw nodes invoke --node "<Android Node>" --command canvas.navigate --params '{"url":"http://<gateway-hostname>.local:18793/__openclaw__/canvas/"}'
```

Tailnet（可选）：如果两台设备都在 Tailscale 上，使用 MagicDNS 名称或 tailnet IP 替代 `.local`，例如：`http://<gateway-magicdns>:18793/__openclaw__/canvas/`。

此服务器会向 HTML 注入一个实时重载（live-reload）客户端，并在文件变更时自动重新加载。
A2UI 主机地址为 `http://<gateway-host>:18793/__openclaw__/a2ui/`。

Canvas 命令（仅前台）：

* `canvas.eval`、`canvas.snapshot`、`canvas.navigate`（使用 `{"url":""}` 或 `{"url":"/"}` 返回默认脚手架）。`canvas.snapshot` 返回 `{ format, base64 }`（默认 `format="jpeg"`）。
* A2UI：`canvas.a2ui.push`、`canvas.a2ui.reset`（`canvas.a2ui.pushJSONL` 为旧版别名）

Camera 命令（仅前台；受权限管控）：

* `camera.snap`（jpg）
* `camera.clip`（mp4）

参数和 CLI 辅助工具参见 [Camera 节点](/zh/nodes/camera)。
