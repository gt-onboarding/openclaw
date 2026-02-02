---
title: Gateway
summary: "Gateway 服务生命周期与运维手册"
read_when:
  - 运行或调试 Gateway 进程时
---

<div id="gateway-service-runbook">
  # Gateway 服务运维手册
</div>

最后更新：2025-12-09

<div id="what-it-is">
  ## 功能概述
</div>

* 始终运行的常驻进程，维护唯一的 Baileys/Telegram 连接，并负责控制/事件平面。
* 替代旧版的 `gateway` 命令。CLI 入口点：`openclaw gateway`。
* 持续运行直到被停止；在发生致命错误时以非零退出码退出，以便由监督进程重启。

<div id="how-to-run-local">
  ## 如何在本地运行
</div>

```bash
openclaw gateway --port 18789
# for full debug/trace logs in stdio:
openclaw gateway --port 18789 --verbose
# 如果端口被占用，先终止监听器再启动：
openclaw gateway --force
# dev loop (auto-reload on TS changes):
pnpm gateway:watch
```

* 配置热重载监听 `~/.openclaw/openclaw.json`（或 `OPENCLAW_CONFIG_PATH`）。
  * 默认模式：`gateway.reload.mode="hybrid"`（对安全变更进行热应用，对关键变更执行重启）。
  * 热重载在需要时通过 **SIGUSR1** 执行进程内重启。
  * 使用 `gateway.reload.mode="off"` 可禁用。
* 将 WebSocket 控制平面绑定到 `127.0.0.1:<port>`（默认 18789）。
* 同一端口同时提供 HTTP 服务（Control UI、hooks、A2UI），单端口多路复用。
  * OpenAI Chat Completions（HTTP）：[`/v1/chat/completions`](/zh/gateway/openai-http-api)。
  * OpenResponses（HTTP）：[`/v1/responses`](/zh/gateway/openresponses-http-api)。
  * Tools Invoke（HTTP）：[`/tools/invoke`](/zh/gateway/tools-invoke-http-api)。
* 默认在 `canvasHost.port`（默认为 `18793`）上启动 Canvas 文件服务器，从 `~/.openclaw/workspace/canvas` 提供 `http://<gateway-host>:18793/__openclaw__/canvas/`。使用 `canvasHost.enabled=false` 或 `OPENCLAW_SKIP_CANVAS_HOST=1` 可禁用。
* 日志输出到 stdout；使用 launchd/systemd 维持进程存活并轮转日志。
* 添加 `--verbose` 参数，可在排障时将调试日志（握手、请求/响应、事件）从日志文件镜像到 stdio。
* `--force` 使用 `lsof` 查找所选端口上的监听进程，发送 SIGTERM，记录被终止的进程，然后启动 Gateway（如果缺少 `lsof` 会快速失败）。
* 如果在进程管理器下运行（launchd/systemd/mac 应用子进程模式），停止/重启通常会发送 **SIGTERM**；旧版本中可能会表现为 `pnpm` `ELIFECYCLE` 退出码 **143**（SIGTERM），这是正常关闭，而非崩溃。
* **SIGUSR1** 在获得授权时触发进程内重启（通过 Gateway 工具/配置的应用或更新，或开启 `commands.restart` 以进行手动重启）。
* 默认启用 Gateway 认证：设置 `gateway.auth.token`（或 `OPENCLAW_GATEWAY_TOKEN`）或 `gateway.auth.password`。除非使用 Tailscale Serve 身份，客户端必须发送 `connect.params.auth.token/password`。
* 向导现在默认会生成一个 token，即使在回环接口上也是如此。
* 端口优先级：`--port` &gt; `OPENCLAW_GATEWAY_PORT` &gt; `gateway.port` &gt; 默认 `18789`。

<div id="remote-access">
  ## 远程访问
</div>

* 优先使用 Tailscale/VPN；否则使用 SSH 隧道：
  ```bash
  ssh -N -L 18789:127.0.0.1:18789 user@host
  ```
* 然后客户端通过该隧道连接到 `ws://127.0.0.1:18789`。
* 如果配置了 token，客户端即使通过隧道连接，也必须在 `connect.params.auth.token` 中包含该 token。

<div id="multiple-gateways-same-host">
  ## 多个 Gateway（同一主机）
</div>

通常没有必要：一个 Gateway 就可以同时服务多个消息通道和智能体。只有在需要冗余或严格隔离时（例如救援机器人）才使用多个 Gateway。

前提是你要隔离状态和配置，并使用不重复的端口。完整指南：[多个 Gateway](/zh/gateway/multiple-gateways)。

服务名称是按配置档（profile）区分的：

* macOS：`bot.molt.<profile>`（旧版 `com.openclaw.*` 可能仍然存在）
* Linux：`openclaw-gateway-<profile>.service`
* Windows：`OpenClaw Gateway (<profile>)`

安装元数据被嵌入在服务配置中：

* `OPENCLAW_SERVICE_MARKER=openclaw`
* `OPENCLAW_SERVICE_KIND=gateway`
* `OPENCLAW_SERVICE_VERSION=<version>`

Rescue-Bot 模式：单独部署第二个 Gateway，使用独立的配置档（profile）、状态目录、工作区和基础端口区间，实现完全隔离。完整指南：[Rescue-bot 指南](/zh/gateway/multiple-gateways#rescue-bot-guide)。

<div id="dev-profile-dev">
  ### 开发配置（`--dev`）
</div>

快速方式：在不改动主环境设置的前提下，运行一个在配置/状态/工作区上完全隔离的开发实例。

```bash
openclaw --dev setup
openclaw --dev gateway --allow-unconfigured
# 然后针对开发实例进行操作：
openclaw --dev status
openclaw --dev health
```

默认值（可通过环境变量/标志/配置覆盖）：

* `OPENCLAW_STATE_DIR=~/.openclaw-dev`
* `OPENCLAW_CONFIG_PATH=~/.openclaw-dev/openclaw.json`
* `OPENCLAW_GATEWAY_PORT=19001`（Gateway 的 WS + HTTP）
* 浏览器控制服务端口 = `19003`（由 `gateway.port+2` 推导，仅本地回环）
* `canvasHost.port=19005`（由 `gateway.port+4` 推导）
* 当你在 `--dev` 下运行 `setup`/`onboard` 时，`agents.defaults.workspace` 默认变为 `~/.openclaw/workspace-dev`。

推导端口（经验法则）：

* 基础端口 = `gateway.port`（或 `OPENCLAW_GATEWAY_PORT` / `--port`）
* 浏览器控制服务端口 = 基础端口 + 2（仅本地回环）
* `canvasHost.port = 基础端口 + 4`（或 `OPENCLAW_CANVAS_HOST_PORT` / 配置覆盖）
* 浏览器 profile 的 CDP 端口会从 `browser.controlPort + 9 .. + 108` 自动分配（按 profile 持久化）。

每个实例的检查清单：

* 唯一的 `gateway.port`
* 唯一的 `OPENCLAW_CONFIG_PATH`
* 唯一的 `OPENCLAW_STATE_DIR`
* 唯一的 `agents.defaults.workspace`
* 独立的 WhatsApp 号码（如果使用 WA）

按 profile 的服务安装：

```bash
openclaw --profile main gateway install
openclaw --profile rescue gateway install
```

示例：

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json OPENCLAW_STATE_DIR=~/.openclaw-a openclaw gateway --port 19001
OPENCLAW_CONFIG_PATH=~/.openclaw/b.json OPENCLAW_STATE_DIR=~/.openclaw-b openclaw gateway --port 19002
```

<div id="protocol-operator-view">
  ## 协议（运维视角）
</div>

* 完整文档请参见：[Gateway 协议](/zh/gateway/protocol) 和 [Bridge 协议（旧版）](/zh/gateway/bridge-protocol)。
* 客户端必须发送的首个帧：`req {type:"req", id, method:"connect", params:{minProtocol,maxProtocol,client:{id,displayName?,version,platform,deviceFamily?,modelIdentifier?,mode,instanceId?}, caps, auth?, locale?, userAgent? } }`。
* Gateway 回复 `res {type:"res", id, ok:true, payload:hello-ok }`（或 `ok:false` 并附带错误信息，然后关闭连接）。
* 完成握手后：
  * 请求：`{type:"req", id, method, params}` → `{type:"res", id, ok, payload|error}`
  * 事件：`{type:"event", event, payload, seq?, stateVersion?}`
* 结构化在线状态（presence）条目：`{host, ip, version, platform?, deviceFamily?, modelIdentifier?, mode, lastInputSeconds?, ts, reason?, tags?[], instanceId? }`（对于 WS 客户端，`instanceId` 来自 `connect.client.instanceId`）。
* `agent` 响应是两阶段的：先返回确认 `res` `{runId,status:"accepted"}`，然后在运行结束后返回最终 `res` `{runId,status:"ok"|"error",summary}`；流式输出通过 `event:"agent"` 事件送达。

<div id="methods-initial-set">
  ## 方法（初始集）
</div>

* `health` — 完整健康快照（结构与 `openclaw health --json` 相同）。
* `status` — 简要概览。
* `system-presence` — 当前在线状态列表。
* `system-event` — 发布在线状态/系统备注（结构化）。
* `send` — 通过当前激活的渠道发送一条消息。
* `agent` — 运行一次智能体轮次（在同一连接上以事件流形式返回）。
* `node.list` — 列出已配对且当前已连接的节点（包括 `caps`、`deviceFamily`、`modelIdentifier`、`paired`、`connected` 以及已发布的 `commands`）。
* `node.describe` — 描述一个节点（功能/能力 + 支持的 `node.invoke` 命令；适用于已配对节点和当前已连接的未配对节点）。
* `node.invoke` — 在节点上调用一个命令（例如 `canvas.*`、`camera.*`）。
* `node.pair.*` — 配对生命周期（`request`、`list`、`approve`、`reject`、`verify`）。

另见：[Presence](/zh/concepts/presence)，了解在线状态是如何生成/去重的，以及为什么稳定的 `client.instanceId` 很重要。

<div id="events">
  ## 事件
</div>

* `agent` — 来自智能体运行时的流式工具/输出事件（带序号标签）。
* `presence` — 推送给所有已连接客户端的在线状态增量更新（包含 `stateVersion`）。
* `tick` — 周期性 keepalive/空操作，用于确认服务存活。
* `shutdown` — Gateway 正在退出；消息体包含 `reason` 和可选的 `restartExpectedMs`。客户端应重新连接。

<div id="webchat-integration">
  ## WebChat 集成
</div>

* WebChat 是一个原生的 SwiftUI UI，直接通过 Gateway 的 WebSocket 获取历史记录、发送消息、执行中止操作并处理事件。
* 远程使用通过同一个 SSH/Tailscale 隧道；如果配置了 Gateway token，客户端会在执行 `connect` 时附带该 token。
* macOS 应用通过单个 WS（共享连接）进行连接；它从初始快照中加载在线状态，并监听 `presence` 事件来更新 UI。

<div id="typing-and-validation">
  ## 类型与校验
</div>

* 服务器使用 AJV，根据协议定义生成的 JSON Schema 校验每一个入站帧。
* 客户端（TS/Swift）使用生成的类型（TS 直接使用；Swift 通过仓库内的生成器获取）。
* 协议定义是单一事实来源；使用以下命令重新生成 schema/模型：
  * `pnpm protocol:gen`
  * `pnpm protocol:gen:swift`

<div id="connection-snapshot">
  ## 连接快照
</div>

* `hello-ok` 会包含一个 `snapshot`，其中带有 `presence`、`health`、`stateVersion` 和 `uptimeMs`，以及 `policy {maxPayload,maxBufferedBytes,tickIntervalMs}`，以便客户端无需额外请求即可立即渲染。
* `health`/`system-presence` 仍然可用于手动刷新，但在建立连接时不是必需的。

<div id="error-codes-reserror-shape">
  ## 错误码（res.error 结构）
</div>

* 错误对象结构为 `{ code, message, details?, retryable?, retryAfterMs? }`。
* 标准错误码：
  * `NOT_LINKED` — WhatsApp 未通过认证。
  * `AGENT_TIMEOUT` — 智能体未在配置的超时时间内响应。
  * `INVALID_REQUEST` — schema/参数验证失败。
  * `UNAVAILABLE` — Gateway 正在关闭或某个依赖不可用。

<div id="keepalive-behavior">
  ## 保活机制
</div>

* 会定期发出 `tick` 事件（或 WS ping/pong），即使在没有流量时也能让客户端知道 Gateway 仍在运行。
* 发送/智能体确认应保持为单独的响应；不要将 tick 复用为发送确认。

<div id="replay-gaps">
  ## 重放 / 序列缺口
</div>

* 事件不会进行重放。客户端会检测序列缺口（seq gaps），并且在继续之前应该先刷新（`health` + `system-presence`）。WebChat 和 macOS 客户端现在会在出现缺口时自动刷新。

<div id="supervision-macos-example">
  ## 守护（macOS 示例）
</div>

* 使用 launchd 保持服务常驻运行：
  * Program：`openclaw` 的路径
  * Arguments：`gateway`
  * KeepAlive：true
  * StandardOut/Err：文件路径或 `syslog`
* 发生故障时，launchd 会重启进程；致命级配置错误应持续退出，以便运维人员注意到问题。
* LaunchAgents 是按用户级别生效，且需要有已登录会话；在无头部署中请使用自定义 LaunchDaemon（默认不随程序提供）。
  * `openclaw gateway install` 会写入 `~/Library/LaunchAgents/bot.molt.gateway.plist`
    （或 `bot.molt.<profile>.plist`；旧版的 `com.openclaw.*` 会被清理）。
  * `openclaw doctor` 会审计 LaunchAgent 配置，并可将其更新为当前默认值。

<div id="gateway-service-management-cli">
  ## Gateway 服务管理（CLI）
</div>

使用 Gateway CLI 来安装、启动、停止、重启并查看状态：

```bash
openclaw gateway status
openclaw gateway install
openclaw gateway stop
openclaw gateway restart
openclaw logs --follow
```

Notes:

* `gateway status` 默认使用服务解析得到的端口/配置探测 Gateway RPC（可以用 `--url` 覆盖）。
* `gateway status --deep` 会增加系统级扫描（LaunchDaemons/system units）。
* `gateway status --no-probe` 会跳过 RPC 探测（在网络中断时很有用）。
* `gateway status --json` 的输出对脚本来说是稳定的。
* `gateway status` 会将 **supervisor 运行时状态**（launchd/systemd 是否在运行）与 **RPC 可达性**（WS 连接 + status RPC）分开报告。
* `gateway status` 会打印配置路径和探测目标，以避免对“localhost 与 LAN 绑定”的混淆以及 profile 不匹配。
* 当服务看起来在运行但端口关闭时，`gateway status` 会包含最近一次 gateway 错误的最后一行。
* `logs` 通过 RPC 实时跟踪 Gateway 文件日志（不需要手动 `tail`/`grep`）。
* 如果检测到其他类似 gateway 的服务，CLI 会发出警告，除非它们是 OpenClaw 的 profile 服务。
  我们仍然推荐在大多数部署中 **每台机器只运行一个 gateway**；若需要冗余或救援 bot，请使用隔离的 profile/端口。参见 [Multiple gateways](/zh/gateway/multiple-gateways)。
  * 清理：`openclaw gateway uninstall`（当前服务）以及 `openclaw doctor`（处理遗留迁移）。
* `gateway install` 在已安装时是空操作；如需重新安装（profile/环境/路径变更），请使用 `openclaw gateway install --force`。

Bundled mac app:

* OpenClaw.app 可以捆绑一个基于 Node 的 gateway relay，并安装一个按用户划分的 LaunchAgent，标签为
  `bot.molt.gateway`（或 `bot.molt.<profile>`；旧版 `com.openclaw.*` 标签仍可正常卸载）。
* 要优雅地停止它，请使用 `openclaw gateway stop`（或 `launchctl bootout gui/$UID/bot.molt.gateway`）。
* 要重启，请使用 `openclaw gateway restart`（或 `launchctl kickstart -k gui/$UID/bot.molt.gateway`）。
  * 只有在 LaunchAgent 已安装的情况下，`launchctl` 才能工作；否则请先使用 `openclaw gateway install`。
  * 运行具名 profile 时，将标签替换为 `bot.molt.<profile>`。

<div id="supervision-systemd-user-unit">
  ## 监督（systemd 用户单元）
</div>

OpenClaw 在 Linux/WSL2 上默认安装 **systemd 用户服务**。我们建议在单用户机器上使用用户服务（环境更简单，按用户维度配置）。对于多用户或需要始终在线的服务器，请使用 **系统服务**（无需启用 lingering，集中统一管理）。

`openclaw gateway install` 会写入用户单元文件。`openclaw doctor` 会检查该单元，并可将其更新为当前推荐的默认配置。

创建 `~/.config/systemd/user/openclaw-gateway[-<profile>].service`：

```
[Unit]
Description=OpenClaw Gateway (profile: <profile>, v<version>)
After=network-online.target
Wants=network-online.target

[Service]
ExecStart=/usr/local/bin/openclaw gateway --port 18789
Restart=always
RestartSec=5
Environment=OPENCLAW_GATEWAY_TOKEN=
WorkingDirectory=/home/youruser

[Install]
WantedBy=default.target
```

启用 lingering（这是必须的，以便用户服务在注销/空闲后仍能继续运行）：

```
sudo loginctl enable-linger youruser
```

Onboarding 会在 Linux/WSL2 环境下运行此命令（可能会提示输入 sudo；并写入 `/var/lib/systemd/linger`）。
然后启用该服务：

```
systemctl --user enable --now openclaw-gateway[-<profile>].service
```

**替代方案（系统服务）** - 对于需要常驻运行或多用户的服务器，你可以
安装 systemd 的**系统级（system）**单元，而不是用户级单元（不需要 lingering）。
创建 `/etc/systemd/system/openclaw-gateway[-<profile>].service`（复制上面的单元，
将 `WantedBy` 切换为 `multi-user.target`，并设置 `User=` 和 `WorkingDirectory=`），然后执行：

```
sudo systemctl daemon-reload
sudo systemctl enable --now openclaw-gateway[-<profile>].service
```

<div id="windows-wsl2">
  ## Windows（WSL2）
</div>

Windows 安装应使用 **WSL2**，并按照上文的 Linux systemd 小节进行操作。

<div id="operational-checks">
  ## 运行状况检查
</div>

* 活跃性（Liveness）：建立 WS 连接并发送 `req:connect` → 应收到带有 `payload.type="hello-ok"`（包含快照）的 `res`。
* 就绪性（Readiness）：调用 `health` → 应看到 `ok: true`，并在 `linkChannel` 中看到已关联的通道（如适用）。
* 调试（Debug）：订阅 `tick` 和 `presence` 事件；确认 `status` 显示通道关联/认证以来的时长；presence 条目中应显示 Gateway 主机和已连接的客户端。

<div id="safety-guarantees">
  ## 安全保证
</div>

* 默认假设每台主机只运行一个 Gateway；如果运行多个配置文件（profile），请隔离端口和状态，并确保请求落到正确的实例上。
* 不会回退到直接的 Baileys 连接；如果 Gateway 宕机，发送将快速失败（fail fast）。
* 非连接类型的首帧或格式错误的 JSON 会被拒绝，并会立即关闭套接字。
* 优雅关闭：在关闭前触发 `shutdown` 事件；客户端必须处理连接关闭并执行重连。

<div id="cli-helpers">
  ## CLI 辅助命令
</div>

* `openclaw gateway health|status` — 通过 Gateway 的 WS 请求运行状况/状态。
* `openclaw message send --target <num> --message "hi" [--media ...]` — 通过 Gateway 发送（在 WhatsApp 上是幂等的）。
* `openclaw agent --message "hi" --to <num>` — 执行一次智能体对话轮次（默认等待最终结果）。
* `openclaw gateway call <method> --params '{"k":"v"}'` — 用于调试的原始方法调用工具。
* `openclaw gateway stop|restart` — 停止/重启受管的 Gateway 服务（launchd/systemd）。
* Gateway 辅助子命令假定在 `--url` 指定地址上已有正在运行的 Gateway 实例；它们不再自动拉起 Gateway。

<div id="migration-guidance">
  ## 迁移指南
</div>

* 停止使用 `openclaw gateway` 和旧版 TCP 控制端口。
* 将客户端更新为使用 WS 协议，并在连接时执行必需的 `connect` 握手以及发送结构化的 `presence` 状态。