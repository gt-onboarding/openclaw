---
title: Web
summary: "Gateway Web 界面：Control UI、绑定模式和安全性"
read_when:
  - 你想通过 Tailscale 远程访问 Gateway
  - 你想在浏览器中使用 Control UI 并编辑配置
---

<div id="web-gateway">
  # Web（Gateway）
</div>

Gateway 会通过与 Gateway WS（WebSocket）相同的端口提供一个精简的 **浏览器 Control UI**（Vite + Lit）：

* 默认：`http://<host>:18789/`
* 可选前缀：设置 `gateway.controlUi.basePath`（例如 `/openclaw`）

相关功能位于 [Control UI](/zh/web/control-ui) 中。
本页重点介绍绑定模式、安全性以及面向 Web 的对外接口。

<div id="webhooks">
  ## Webhooks
</div>

当 `hooks.enabled=true` 时，Gateway 也会在同一个 HTTP 服务器上暴露一个简单的 webhook 端点。
有关认证和负载的说明，参见 [Gateway configuration](/zh/gateway/configuration) → `hooks`。

<div id="config-default-on">
  ## 配置（默认启用）
</div>

当存在资源文件（`dist/control-ui`）时，Control UI **默认启用**。
你可以在配置中对其进行控制：

```json5
{
  gateway: {
    controlUi: { enabled: true, basePath: "/openclaw" } // basePath 可选
  }
}
```

<div id="tailscale-access">
  ## 通过 Tailscale 访问
</div>

<div id="integrated-serve-recommended">
  ### 集成 Serve（推荐）
</div>

让 Gateway 继续只监听本机 loopback 接口，由 Tailscale Serve 为其提供代理转发：

```json5
{
  gateway: {
    bind: "loopback",
    tailscale: { mode: "serve" }
  }
}
```

接下来启动 Gateway：

```bash
openclaw gateway
```

在浏览器中打开：

* `https://<magicdns>/`（或你已配置的 `gateway.controlUi.basePath`）

<div id="tailnet-bind-token">
  ### Tailnet 绑定和令牌
</div>

```json5
{
  gateway: {
    bind: "tailnet",
    controlUi: { enabled: true },
    auth: { mode: "token", token: "your-token" }
  }
}
```

然后启动 Gateway（绑定到非回环地址时需要令牌）：

```bash
openclaw gateway
```

打开：

* `http://<tailscale-ip>:18789/`（或你配置的 `gateway.controlUi.basePath`）

<div id="public-internet-funnel">
  ### 公网（Funnel）
</div>

```json5
{
  gateway: {
    bind: "loopback",
    tailscale: { mode: "funnel" },
    auth: { mode: "password" } // 或使用 OPENCLAW_GATEWAY_PASSWORD
  }
}
```

<div id="security-notes">
  ## 安全注意事项
</div>

* 默认情况下需要 Gateway 认证（token/密码或 Tailscale 身份头信息）。
* 非 loopback 绑定仍然**需要**共享 token/密码（`gateway.auth` 或环境变量）。
* 向导默认会生成一个 gateway token（即使在 loopback 上也是如此）。
* UI 会发送 `connect.params.auth.token` 或 `connect.params.auth.password`。
* 使用 Serve 时，在 `gateway.auth.allowTailscale` 为 `true` 时，Tailscale 身份头信息即可满足认证要求（不需要 token/密码）。将 `gateway.auth.allowTailscale` 设为 `false` 可强制要求显式凭据。参见
  [Tailscale](/zh/gateway/tailscale) 和 [Security](/zh/gateway/security)。
* `gateway.tailscale.mode: "funnel"` 需要 `gateway.auth.mode: "password"`（共享密码）。

<div id="building-the-ui">
  ## 构建 UI
</div>

Gateway 会从 `dist/control-ui` 目录提供静态文件。使用以下命令进行构建：

```bash
pnpm ui:build # 首次运行时自动安装 UI 依赖项
```
