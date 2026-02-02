---
title: "macOS"
summary: "OpenClaw macOS 配套应用（菜单栏 + Gateway 代理）"
read_when:
  - 实现 macOS 应用功能
  - 在 macOS 上更改 Gateway 生命周期或节点桥接
---

<div id="openclaw-macos-companion-menu-bar-gateway-broker">
  # OpenClaw macOS Companion（菜单栏 + Gateway 中继）
</div>

这个 macOS 应用是 OpenClaw 的**菜单栏伴侣应用**。它负责权限管理，
在本地（通过 launchd 或手动）管理/接入 Gateway，并将 macOS
系统能力以节点的形式暴露给智能体使用。

<div id="what-it-does">
  ## 它的作用
</div>

* 在菜单栏中显示原生通知和状态。
* 接管 TCC 授权弹窗（通知、辅助功能、屏幕录制、麦克风、
  语音识别、自动化/AppleScript）。
* 运行或连接 Gateway（本地或远程）。
* 提供仅适用于 macOS 的工具（Canvas、Camera、屏幕录制、`system.run`）。
* 在**远程**模式下通过 launchd 启动本地节点宿主服务，在**本地**模式下将其停止。
* 可选地托管用于 UI 自动化的 **PeekabooBridge**。
* 按需通过 npm/pnpm 安装全局 CLI `openclaw`（不建议使用 bun 作为 Gateway 运行时）。

<div id="local-vs-remote-mode">
  ## 本地模式与远程模式
</div>

* **本地（Local）**（默认）：如果本地已有正在运行的 Gateway，应用会连接到该实例；
  否则会通过 `openclaw gateway install` 启用 launchd 服务。
* **远程（Remote）**：应用通过 SSH/Tailscale 连接到远程 Gateway，并且永远不会启动
  本地进程。
  应用会启动本地的 **节点主机服务（node host service）**，以便远程 Gateway 可以访问这台 Mac。
  应用不会将 Gateway 作为子进程启动。

<div id="launchd-control">
  ## Launchd 控制
</div>

该应用会为每个用户管理一个 LaunchAgent，标签为 `bot.molt.gateway`
（在使用 `--profile`/`OPENCLAW_PROFILE` 时则为 `bot.molt.<profile>`；旧版的 `com.openclaw.*` 仍会被卸载）。

```bash
launchctl kickstart -k gui/$UID/bot.molt.gateway
launchctl bootout gui/$UID/bot.molt.gateway
```

在运行具名配置文件时，将该标签替换为 `bot.molt.&lt;profile&gt;`。

如果尚未安装 LaunchAgent，请在应用中将其启用，或运行
`openclaw gateway install`。

<div id="node-capabilities-mac">
  ## 节点能力（mac）
</div>

macOS 应用会作为一个节点存在。常见命令：

* 画布：`canvas.present`, `canvas.navigate`, `canvas.eval`, `canvas.snapshot`, `canvas.a2ui.*`
* 相机：`camera.snap`, `camera.clip`
* 屏幕：`screen.record`
* 系统：`system.run`, `system.notify`

该节点会上报一个 `permissions` 映射表，以便智能体决定允许哪些操作。

节点服务 + 应用 IPC：

* 当无头节点宿主服务在运行时（远程模式），它会作为节点连接到 Gateway 的 WS。
* `system.run` 会在 macOS 应用中执行（UI/TCC 上下文），通过本地 Unix 套接字进行通信；交互提示与输出都保留在应用内。

示意图（SCI）：

```
Gateway -> Node Service (WS)
                 |  IPC (UDS + token + HMAC + TTL)
                 v
             Mac App (UI + TCC + system.run)
```

<div id="exec-approvals-systemrun">
  ## Exec approvals（system.run）
</div>

`system.run` 由 macOS 应用中的 **Exec approvals** 控制（Settings → Exec approvals）。
Security + ask + allowlist 会本地存储在这台 Mac 上，位置为：

```
~/.openclaw/exec-approvals.json
```

示例：

```json
{
  "version": 1,
  "defaults": {
    "security": "deny",
    "ask": "on-miss"
  },
  "agents": {
    "main": {
      "security": "allowlist",
      "ask": "on-miss",
      "allowlist": [
        { "pattern": "/opt/homebrew/bin/rg" }
      ]
    }
  }
}
```

注意：

* `allowlist` 条目是针对解析后的可执行文件路径的 glob 通配符模式。
* 在提示中选择“始终允许”会将该命令添加到 `allowlist` 中。
* `system.run` 的环境变量覆盖设置会先被过滤（丢弃 `PATH`、`DYLD_*`、`LD_*`、`NODE_OPTIONS`、`PYTHON*`、`PERL*`、`RUBYOPT`），然后与应用的环境变量合并。

<div id="deep-links">
  ## 深度链接
</div>

该应用会注册 `openclaw://` URL scheme，用于触发本地操作。

<div id="openclawagent">
  ### `openclaw://agent`
</div>

触发一个 Gateway 的 `agent`（智能体）请求。

```bash
open 'openclaw://agent?message=Hello%20from%20deep%20link'
```

查询参数：

* `message`（必需）
* `sessionKey`（可选）
* `thinking`（可选）
* `deliver` / `to` / `channel`（可选）
* `timeoutSeconds`（可选）
* `key`（可选的无人值守模式密钥）

安全性：

* 未提供 `key` 时，应用会提示进行确认。
* 提供有效 `key` 时，运行将为无人值守模式（用于个人自动化场景）。

<div id="onboarding-flow-typical">
  ## 入门流程（典型）
</div>

1. 安装并启动 **OpenClaw.app**。
2. 完成权限检查清单（TCC 提示）。
3. 确保 **本地** 模式已启用且 Gateway 正在运行。
4. 如果你需要通过终端进行操作，请安装 CLI。

<div id="build-dev-workflow-native">
  ## 构建与开发流程（原生）
</div>

* `cd apps/macos && swift build`
* `swift run OpenClaw`（或使用 Xcode）
* 打包应用程序：`scripts/package-mac-app.sh`

<div id="debug-gateway-connectivity-macos-cli">
  ## 调试 Gateway 连接性（macOS CLI）
</div>

使用调试 CLI 复现与 macOS 应用相同的 Gateway WebSocket 握手和发现逻辑，而无需启动应用程序。

```bash
cd apps/macos
swift run openclaw-mac connect --json
swift run openclaw-mac discover --timeout 3000 --json
```

连接选项：

* `--url <ws://host:port>`：覆盖配置中的设置
* `--mode <local|remote>`：从配置中解析（默认：配置值或 local）
* `--probe`：强制执行新的健康检查
* `--timeout <ms>`：请求超时时间（默认：`15000`）
* `--json`：用于差异比较（diff）的结构化输出

发现选项：

* `--include-local`：包含本来会被视为“本地”而被过滤掉的 Gateway
* `--timeout <ms>`：整体发现时间窗口（默认：`2000`）
* `--json`：用于差异比较（diff）的结构化输出

提示：将其与 `openclaw gateway discover --json` 的输出进行比较，以查看
macOS 应用的发现流程（NWBrowser + tailnet DNS‑SD 回退）是否与
Node CLI 基于 `dns-sd` 的发现流程是否不同。

<div id="remote-connection-plumbing-ssh-tunnels">
  ## 远程连接管道（SSH 隧道）
</div>

当 macOS 应用处于 **Remote** 模式运行时，它会建立一个 SSH 隧道，这样本地 UI 组件就可以像访问 localhost 一样与远程 Gateway 通信。

<div id="control-tunnel-gateway-websocket-port">
  ### 控制隧道（Gateway WebSocket 端口）
</div>

* **用途：** 健康检查、状态查询、Web Chat、配置以及其他控制平面调用。
* **本地端口：** Gateway 端口（默认 `18789`），始终固定不变。
* **远程端口：** 远程主机上的同一 Gateway 端口。
* **行为：** 不会使用随机本地端口；应用会复用已存在且健康的隧道，
  如有需要则重启该隧道。
* **SSH 形式：** 使用 BatchMode + ExitOnForwardFailure + keepalive 选项的 `ssh -N -L <local>:127.0.0.1:<remote>` 命令。
* **IP 报告：** 由于 SSH 隧道使用回环地址，Gateway 看到的节点 IP 将是 `127.0.0.1`。如果你希望看到真实客户端 IP，请使用 **Direct (ws/wss)** 传输（参见 [macOS remote access](/zh/platforms/mac/remote)）。

有关设置步骤，请参见 [macOS remote access](/zh/platforms/mac/remote)。有关协议细节，请参见 [Gateway protocol](/zh/gateway/protocol)。

<div id="related-docs">
  ## 相关文档
</div>

* [Gateway 运维手册](/zh/gateway)
* [Gateway（macOS）](/zh/platforms/mac/bundled-gateway)
* [macOS 权限](/zh/platforms/mac/permissions)
* [Canvas](/zh/platforms/mac/canvas)