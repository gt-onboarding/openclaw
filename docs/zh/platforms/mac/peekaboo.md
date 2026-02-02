---
title: Peekaboo
summary: "用于 macOS UI 自动化的 PeekabooBridge 集成"
read_when:
  - 在 OpenClaw.app 中托管 PeekabooBridge
  - 通过 Swift Package Manager 集成 Peekaboo
  - 修改 PeekabooBridge 协议/路径
---

<div id="peekaboo-bridge-macos-ui-automation">
  # Peekaboo Bridge（macOS UI 自动化）
</div>

OpenClaw 可以托管 **PeekabooBridge**，作为本地、具权限感知能力的 UI 自动化
代理服务。这样 `peekaboo` CLI 就可以在复用 macOS 应用的 TCC 权限的同时驱动 UI 自动化。

<div id="what-this-is-and-isnt">
  ## 这是什么（以及不是什么）
</div>

- **Host**：OpenClaw.app 可以作为 PeekabooBridge 的 host。
- **Client**：使用 `peekaboo` CLI（没有单独的 `openclaw ui ...` 界面层）。
- **UI**：UI 叠加层仍然在 Peekaboo.app 中；OpenClaw 只是一个轻量级的中介 host。

<div id="enable-the-bridge">
  ## 启用桥接
</div>

在 macOS 应用中：

- Settings → **Enable Peekaboo Bridge**

启用后，OpenClaw 会启动一个本地 UNIX 套接字服务器。若禁用，则该主机会停止运行，`peekaboo` 将回退使用其他可用主机。

<div id="client-discovery-order">
  ## 客户端发现顺序
</div>

Peekaboo 客户端通常按以下顺序尝试主机：

1. Peekaboo.app（完整用户体验 UX）
2. Claude.app（如果已安装）
3. OpenClaw.app（轻量级代理）

使用 `peekaboo bridge status --verbose` 查看当前使用的主机以及正在使用的 socket 路径。你可以通过以下方式手动覆盖：

```bash
export PEEKABOO_BRIDGE_SOCKET=/path/to/bridge.sock
```


<div id="security-permissions">
  ## 安全与权限
</div>

- 桥接服务会验证**调用方代码签名**；并强制执行 TeamID 允许列表（Peekaboo 宿主 TeamID + OpenClaw 应用 TeamID）。
- 请求会在约 10 秒后超时。
- 如果缺少所需权限，桥接服务会返回清晰的错误信息，而不是启动“系统设置”。

<div id="snapshot-behavior-automation">
  ## 快照行为（自动化）
</div>

快照存储在内存中，并会在短时间后自动失效。
如果你需要更长的保留期，请通过客户端重新采集快照。

<div id="troubleshooting">
  ## 故障排查
</div>

- 如果 `peekaboo` 报告“bridge client is not authorized”错误，请确保客户端已正确签名，或者仅在 **debug** 模式下设置 `PEEKABOO_ALLOW_UNSIGNED_SOCKET_CLIENTS=1` 来运行 host。
- 如果未找到任何 host，请打开任一 host 应用（Peekaboo.app 或 OpenClaw.app），并确认已经授予相关权限。