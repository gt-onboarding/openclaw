---
title: Control UI
summary: "Gateway 的浏览器版 Control UI（聊天、节点、配置）"
read_when:
  - 你希望通过浏览器操作 Gateway
  - 你希望在不使用 SSH 隧道的情况下访问 Tailnet
---

<div id="control-ui-browser">
  # Control UI（浏览器）
</div>

Control UI 是由 Gateway 提供的一个 **Vite + Lit** 单页应用：

* 默认：`http://<host>:18789/`
* 可选前缀：设置 `gateway.controlUi.basePath`（例如 `/openclaw`）

它通过同一端口**直接与 Gateway 的 WS（WebSocket）通信**。

<div id="quick-open-local">
  ## 快速打开（本地）
</div>

如果 Gateway 在同一台计算机上运行，打开：

* http://127.0.0.1:18789/（或 http://localhost:18789/）

如果页面无法加载，请先启动 Gateway：`openclaw gateway`。

认证信息会在 WebSocket 握手时通过以下参数传递：

* `connect.params.auth.token`
* `connect.params.auth.password`
  仪表盘设置面板允许你保存一个 token；密码不会被持久化。
  引导向导默认会生成一个 Gateway token，因此在首次连接时将它粘贴到这里。

<div id="what-it-can-do-today">
  ## 它目前可以做什么
</div>

* 通过 Gateway 的 WS 与模型聊天（`chat.history`、`chat.send`、`chat.abort`、`chat.inject`）
* 在聊天中流式显示工具调用 + 实时工具输出卡片（智能体事件）
* 渠道：WhatsApp/Telegram/Discord/Slack + 插件渠道（如 Mattermost 等）状态 + 二维码登录 + 按渠道单独配置（`channels.status`、`web.login.*`、`config.patch`）
* 实例：在线实例列表 + 刷新（`system-presence`）
* 会话：列出 + 按会话覆盖思考/详细输出设置（`sessions.list`、`sessions.patch`）
* 定时任务：列出/添加/运行/启用/禁用 + 运行历史（`cron.*`）
* 技能：状态、启用/禁用、安装、API 密钥更新（`skills.*`）
* 节点：列出 + 能力（`node.list`）
* 执行审批：编辑 Gateway 或节点允许列表 + 为 `exec host=gateway/node` 查询策略（`exec.approvals.*`）
* 配置：查看/编辑 `~/.openclaw/openclaw.json`（`config.get`、`config.set`）
* 配置：应用 + 校验后重启（`config.apply`），并唤醒上次活跃的会话
* 配置写入包含基础哈希保护，防止覆盖并发编辑
* 配置 schema + 表单渲染（`config.schema`，包括插件 + 渠道的 schema）；原始 JSON 编辑器仍可用
* 调试：状态/健康/模型快照 + 事件日志 + 手动 RPC 调用（`status`、`health`、`models.list`）
* 日志：实时跟踪 Gateway 文件日志，支持过滤/导出（`logs.tail`）
* 更新：运行包/git 更新 + 重启（`update.run`），并提供重启报告

<div id="chat-behavior">
  ## 对话行为
</div>

* `chat.send` 是**非阻塞**的：它会立即返回 `{ runId, status: "started" }`，并通过 `chat` 事件以流式方式传输响应。
* 使用相同的 `idempotencyKey` 重新发送时，在运行期间返回 `{ status: "in_flight" }`，完成后返回 `{ status: "ok" }`。
* `chat.inject` 会在会话记录中追加一条助手备注，并广播一个仅用于 UI 更新的 `chat` 事件（不触发智能体运行，不进行渠道投递）。
* 停止：
  * 点击 **Stop**（调用 `chat.abort`）
  * 输入 `/stop`（或 `stop|esc|abort|wait|exit|interrupt`）以带外中止
  * `chat.abort` 支持仅传入 `{ sessionKey }`（无 `runId`）以中止该会话的所有正在进行的运行

<div id="tailnet-access-recommended">
  ## 通过 Tailnet 访问（推荐）
</div>

<div id="integrated-tailscale-serve-preferred">
  ### 集成 Tailscale Serve（推荐）
</div>

让 Gateway 监听在本地回环地址上，由 Tailscale Serve 通过 HTTPS 进行代理：

```bash
openclaw gateway --tailscale serve
```

打开：

* `https://<magicdns>/`（或你配置的 `gateway.controlUi.basePath`）

默认情况下，当 `gateway.auth.allowTailscale` 为 `true` 时，Serve 请求可以通过 Tailscale 身份头（`tailscale-user-login`）进行认证。OpenClaw 会使用 `tailscale whois` 解析 `x-forwarded-for` 地址并将其与该头进行匹配以验证身份，并且只在请求通过回环地址且携带由 Tailscale 设置的 `x-forwarded-*` 头时才接受这些请求。如果你希望即使对 Serve 流量也强制要求令牌/密码，则将
`gateway.auth.allowTailscale: false`（或强制使用 `gateway.auth.mode: "password"`）进行设置。

<div id="bind-to-tailnet-token">
  ### 绑定到 tailnet 和令牌
</div>

```bash
openclaw gateway --bind tailnet --token "$(openssl rand -hex 32)"
```

然后打开：

* `http://<tailscale-ip>:18789/`（或你配置的 `gateway.controlUi.basePath`）

在 UI 设置中粘贴该 token（它会作为 `connect.params.auth.token` 发送）。

<div id="insecure-http">
  ## 不安全的 HTTP
</div>

如果你通过明文 HTTP（`http://<lan-ip>` 或 `http://<tailscale-ip>`）打开仪表盘，
浏览器会处于一个**不安全的上下文环境**中，并阻止 WebCrypto。默认情况下，
OpenClaw 会在没有设备身份的情况下**阻止** Control UI 连接。

**推荐解决方案：** 使用 HTTPS（Tailscale Serve）或在本地访问 UI：

* `https://<magicdns>/`（Serve）
* `http://127.0.0.1:18789/`（在 Gateway 主机上）

**降级示例（仅使用 token 通过 HTTP）：**

```json5
{
  gateway: {
    controlUi: { allowInsecureAuth: true },
    bind: "tailnet",
    auth: { mode: "token", token: "replace-me" }
  }
}
```

这会为 Control UI 禁用设备身份和配对功能（即使在 HTTPS 下也是如此）。仅在你完全信任当前网络时才使用。

有关 HTTPS 配置说明，请参见 [Tailscale](/zh/gateway/tailscale)。

<div id="building-the-ui">
  ## 构建 UI
</div>

Gateway 会从 `dist/control-ui` 目录提供静态资源。使用以下命令进行构建：

```bash
pnpm ui:build # 首次运行时自动安装 UI 依赖项
```

可选的绝对基路径（当你需要固定的静态资源 URL 时）：

```bash
OPENCLAW_CONTROL_UI_BASE_PATH=/openclaw/ pnpm ui:build
```

用于本地开发（使用独立的开发服务器）：

```bash
pnpm ui:dev # 首次运行时自动安装 UI 依赖项
```

然后将 UI 配置为使用你的 Gateway WS URL（例如 `ws://127.0.0.1:18789`）。

<div id="debuggingtesting-dev-server-remote-gateway">
  ## 调试/测试：开发服务器 + 远程 Gateway
</div>

Control UI 是静态文件；WebSocket 目标地址是可配置的，并且可以与 HTTP 源不同。
当你希望在本地使用 Vite 开发服务器，而让 Gateway 运行在其他位置时，这会非常方便。

1. 启动 UI 开发服务器：`pnpm ui:dev`
2. 打开类似如下的 URL：

```text
http://localhost:5173/?gatewayUrl=ws://<gateway-host>:18789
```

可选的一次性认证（如有需要）：

```text
http://localhost:5173/?gatewayUrl=wss://<gateway-host>:18789&token=<gateway-token>
```

注意：

* `gatewayUrl` 会在加载后写入 localStorage，并从 URL 中移除。
* `token` 存储在 localStorage 中；`password` 仅保留在内存中。
* 当 Gateway 通过 TLS 提供服务（Tailscale Serve、HTTPS 代理等）时，请使用 `wss://`。

远程访问的设置详情：参见 [Remote access](/zh/gateway/remote)。
