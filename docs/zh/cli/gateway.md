---
title: Gateway
summary: "OpenClaw Gateway CLI（`openclaw gateway`）— 运行、查询和发现 Gateway"
read_when:
  - 通过 CLI 运行 Gateway（开发环境或服务器）
  - 调试 Gateway 的身份验证、绑定模式和连接问题
  - 通过 Bonjour 发现 Gateway（局域网 + tailnet）
---

<div id="gateway-cli">
  # Gateway CLI
</div>

Gateway 是 OpenClaw 的 WebSocket 服务器（用于处理 channels、节点、会话和 hooks）。

本页中的子命令均在 `openclaw gateway …` 命令之下。

相关文档：

* [/gateway/bonjour](/zh/gateway/bonjour)
* [/gateway/discovery](/zh/gateway/discovery)
* [/gateway/configuration](/zh/gateway/configuration)

<div id="run-the-gateway">
  ## 运行 Gateway
</div>

在本地运行一个 Gateway 进程：

```bash
openclaw gateway
```

前台运行别名：

```bash
openclaw gateway run
```

Notes:

* 默认情况下，除非在 `~/.openclaw/openclaw.json` 中将 `gateway.mode=local` 设置为该值，否则 Gateway 会拒绝启动。临时/开发环境下运行可使用 `--allow-unconfigured`。
* 在没有认证的情况下，尝试将监听地址绑定到 loopback 回环地址之外会被阻止（安全防护措施）。
* 在已授权时，`SIGUSR1` 会触发进程内重启（启用 `commands.restart`，或使用 Gateway 工具/配置执行 apply/update）。
* `SIGINT`/`SIGTERM` 处理程序会停止 Gateway 进程，但不会恢复任何自定义的终端状态。如果你用 TUI 或原始模式输入对 CLI 进行了封装，请在退出前手动恢复终端。

<div id="options">
  ### 选项
</div>

* `--port <port>`: WebSocket 端口（默认从配置/环境中获取；通常为 `18789`）。
* `--bind <loopback|lan|tailnet|auto|custom>`: 监听绑定模式。
* `--auth <token|password>`: 覆盖默认认证模式。
* `--token <token>`: 覆盖 token（同时为该进程设置 `OPENCLAW_GATEWAY_TOKEN`）。
* `--password <password>`: 覆盖密码（同时为该进程设置 `OPENCLAW_GATEWAY_PASSWORD`）。
* `--tailscale <off|serve|funnel>`: 通过 Tailscale 暴露 Gateway。
* `--tailscale-reset-on-exit`: 在关停时重置 Tailscale serve/funnel 配置。
* `--allow-unconfigured`: 允许在配置中没有 `gateway.mode=local` 时启动 Gateway。
* `--dev`: 若缺失则创建开发配置和工作区（跳过 BOOTSTRAP.md）。
* `--reset`: 重置开发配置、凭据、会话和工作区（需要 `--dev`）。
* `--force`: 启动前终止所选端口上任意已存在的监听进程。
* `--verbose`: 输出详细日志。
* `--claude-cli-logs`: 仅在控制台中显示 claude-cli 日志（并启用其 stdout/stderr）。
* `--ws-log <auto|full|compact>`: WS 日志格式（默认 `auto`）。
* `--compact`: `--ws-log compact` 的别名。
* `--raw-stream`: 将原始模型流事件记录到 jsonl。
* `--raw-stream-path <path>`: 原始流 jsonl 路径。

<div id="query-a-running-gateway">
  ## 查询正在运行的 Gateway
</div>

所有查询类命令都使用 WebSocket RPC。

输出模式：

* 默认：便于阅读的人类可读输出（TTY 中带颜色）。
* `--json`：机器可读的 JSON（无样式/无旋转加载动画）。
* `--no-color`（或 `NO_COLOR=1`）：禁用 ANSI 颜色，同时保留人类友好的布局。

通用选项（在支持的情况下）：

* `--url <url>`：Gateway 的 WebSocket 地址。
* `--token <token>`：Gateway 令牌。
* `--password <password>`：Gateway 密码。
* `--timeout <ms>`：超时时间/时间预算（因命令而异）。
* `--expect-final`：等待“最终”响应（用于智能体调用）。

<div id="gateway-health">
  ### `gateway health`
</div>

```bash
openclaw gateway health --url ws://127.0.0.1:18789
```

<div id="gateway-status">
  ### `gateway status`
</div>

`gateway status` 会显示 Gateway 服务（launchd/systemd/schtasks），并且可以附带一次可选的 RPC 探测。

```bash
openclaw gateway status
openclaw gateway status --json
```

Options:

* `--url <url>`: 覆盖默认探测 URL。
* `--token <token>`: 使用 token 进行探测认证。
* `--password <password>`: 使用密码进行探测认证。
* `--timeout <ms>`: 探测超时时间（默认 `10000`）。
* `--no-probe`: 跳过 RPC 探测（仅服务视图）。
* `--deep`: 同时扫描系统级服务。

<div id="gateway-probe">
  ### `gateway probe`
</div>

`gateway probe` 是用于“调试一切”的命令。它始终会探测：

* 你配置的远程 Gateway（如果已设置），以及
* 本机（localhost 回环地址），**即使已经配置了远程 Gateway**。

如果可以访问多个 Gateway，它会将它们全部列出。使用隔离的配置文件/端口（例如用于救援的机器人）时，可以同时运行多个 Gateway，但大多数部署仍然只运行单个 Gateway。

```bash
openclaw gateway probe
openclaw gateway probe --json
```

<div id="remote-over-ssh-mac-app-parity">
  #### 通过 SSH 远程（与 Mac 应用行为一致）
</div>

macOS 应用的 “Remote over SSH” 模式使用本地端口转发，使远程 Gateway（可能只绑定在回环地址上）可以通过 `ws://127.0.0.1:<port>` 访问。

对应的 CLI 命令：

```bash
openclaw gateway probe --ssh user@gateway-host
```

Options:

* `--ssh <target>`：`user@host` 或 `user@host:port`（端口默认为 `22`）。
* `--ssh-identity <path>`：SSH 身份密钥文件。
* `--ssh-auto`：自动选择首个发现的 Gateway 主机作为 SSH 目标（仅限 LAN/WAB）。

Config（可选，用作默认值）：

* `gateway.remote.sshTarget`
* `gateway.remote.sshIdentity`

<div id="gateway-call-method">
  ### `gateway call <method>`
</div>

底层 RPC 辅助命令。

```bash
openclaw gateway call status
openclaw gateway call logs.tail --params '{"sinceMs": 60000}'
```

<div id="manage-the-gateway-service">
  ## 管理 Gateway 服务
</div>

```bash
openclaw gateway install
openclaw gateway start
openclaw gateway stop
openclaw gateway restart
openclaw gateway uninstall
```

Notes:

* `gateway install` 支持 `--port`、`--runtime`、`--token`、`--force`、`--json`。
* 生命周期命令支持 `--json` 选项，便于脚本化。

<div id="discover-gateways-bonjour">
  ## 发现 Gateway（Bonjour）
</div>

`gateway discover` 会扫描 Gateway 信标广播（`_openclaw-gw._tcp`）。

* 组播 DNS-SD：`local.`
* 单播 DNS-SD（广域 Bonjour）：选择一个域名（示例：`openclaw.internal.`），并配置 split DNS 和 DNS 服务器；参见 [/gateway/bonjour](/zh/gateway/bonjour)

只有启用了 Bonjour 发现功能（默认）的 Gateway 才会广播该信标。

广域发现记录（TXT）包括：

* `role`（Gateway 角色提示）
* `transport`（传输方式提示，例如 `gateway`）
* `gatewayPort`（WebSocket 端口，通常为 `18789`）
* `sshPort`（SSH 端口；如果未提供则默认为 `22`）
* `tailnetDns`（MagicDNS 主机名（如可用时））
* `gatewayTls` / `gatewayTlsSha256`（TLS 是否启用 + 证书指纹）
* `cliPath`（用于远程安装的可选提示）

<div id="gateway-discover">
  ### `gateway discover`
</div>

```bash
openclaw gateway discover
```

选项：

* `--timeout <ms>`：每条命令的超时时间（browse/resolve），默认值为 `2000`。
* `--json`：机器可读的输出（同时禁用样式/加载动画）。

示例：

```bash
openclaw gateway discover --timeout 4000
openclaw gateway discover --json | jq '.beacons[].wsUrl'
```
