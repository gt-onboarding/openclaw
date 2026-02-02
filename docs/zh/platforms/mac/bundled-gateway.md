---
title: 捆绑的 Gateway
summary: "macOS 上的 Gateway 运行时（外部 launchd 服务）"
read_when:
  - 打包 OpenClaw.app
  - 调试 macOS 上的 Gateway launchd 服务
  - 在 macOS 上安装 Gateway CLI
---

<div id="gateway-on-macos-external-launchd">
  # macOS 上的 Gateway（外部 launchd）
</div>

OpenClaw.app 不再内置 Node/Bun 或 Gateway 运行时。macOS 应用要求你**单独**安装 `openclaw` CLI，不会将 Gateway 作为子进程启动，而是为每个用户管理一个 launchd 服务以保持 Gateway 持续运行（或者在本地已有 Gateway 正在运行时连接到该实例）。

<div id="install-the-cli-required-for-local-mode">
  ## 安装 CLI（本地模式所必需）
</div>

你需要在 Mac 上安装 Node 22 或更高版本，然后全局安装 `openclaw`：

```bash
npm install -g openclaw@<version>
```

macOS 应用中的 **Install CLI** 按钮会通过 npm/pnpm 执行相同的流程（不推荐将 bun 用作 Gateway 的运行时环境）。


<div id="launchd-gateway-as-launchagent">
  ## Launchd（作为 LaunchAgent 运行的 Gateway）
</div>

标签：

- `bot.molt.gateway`（或 `bot.molt.<profile>`；历史遗留的 `com.openclaw.*` 可能仍然存在）

Plist 位置（按用户）：

- `~/Library/LaunchAgents/bot.molt.gateway.plist`
  （或 `~/Library/LaunchAgents/bot.molt.<profile>.plist`）

管理方式：

- 在本地模式下，macOS 应用负责安装/更新 LaunchAgent。
- 也可以通过 CLI 安装：`openclaw gateway install`。

行为：

- “OpenClaw Active” 用于启用/禁用 LaunchAgent。
- 退出应用**不会**停止 Gateway（launchd 会保持其运行）。
- 如果在已配置端口上已有 Gateway 运行，应用会连接到该进程，
  而不是启动一个新的实例。

日志记录：

- launchd stdout/err：`/tmp/openclaw/openclaw-gateway.log`

<div id="version-compatibility">
  ## 版本兼容性
</div>

macOS 应用会检查 Gateway 的版本是否与自身版本一致。若二者不兼容，请更新全局安装的 CLI，使其与应用版本保持一致。

<div id="smoke-check">
  ## 冒烟检查
</div>

```bash
openclaw --version

OPENCLAW_SKIP_CHANNELS=1 \
OPENCLAW_SKIP_CANVAS_HOST=1 \
openclaw gateway --port 18999 --bind loopback
```

然后：

```bash
openclaw gateway call health --url ws://127.0.0.1:18999 --timeout 3000
```
