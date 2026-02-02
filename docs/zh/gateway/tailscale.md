---
title: Tailscale
summary: "为 Gateway 控制面板集成的 Tailscale Serve/Funnel 支持"
read_when:
  - 将 Gateway Control UI 暴露到 localhost 之外
  - 自动化 tailnet 或公共控制面板的访问
---

<div id="tailscale-gateway-dashboard">
  # Tailscale（Gateway 仪表盘）
</div>

OpenClaw 可以为 Gateway 仪表盘和 WebSocket 端口自动配置 Tailscale **Serve**（tailnet）或 **Funnel**（公共访问）。这可以让 Gateway 仅绑定到回环地址运行，同时由 Tailscale 提供 HTTPS、路由，以及（对于 Serve）身份头信息。

<div id="modes">
  ## 模式
</div>

- `serve`: 仅在 Tailnet 内通过 `tailscale serve` 提供服务。Gateway 仅监听在 `127.0.0.1`。
- `funnel`: 通过 `tailscale funnel` 提供公网 HTTPS。OpenClaw 需要一个共享密码。
- `off`: 默认（不启用 Tailscale 自动化）。

<div id="auth">
  ## 认证
</div>

设置 `gateway.auth.mode` 来控制握手流程：

- `token`（在设置了 `OPENCLAW_GATEWAY_TOKEN` 时为默认值）
- `password`（通过 `OPENCLAW_GATEWAY_PASSWORD` 或配置中的共享密钥）

当 `tailscale.mode = "serve"` 且 `gateway.auth.allowTailscale` 为 `true` 时，
有效的 Serve 代理请求可以通过 Tailscale 身份请求头
（`tailscale-user-login`）完成认证，而无需提供 `token`/`password`。OpenClaw
会通过本地 Tailscale 守护进程（`tailscale whois`）解析
`x-forwarded-for` 地址，并在接受该请求前将解析结果与该请求头进行匹配，
以验证身份。只有当请求从回环地址到达，并携带 Tailscale 设置的
`x-forwarded-for`、`x-forwarded-proto` 和 `x-forwarded-host`
请求头时，OpenClaw 才会将其视为 Serve 请求。
如需强制要求显式凭据认证，将 `gateway.auth.allowTailscale` 设为 `false`，
或强制使用 `gateway.auth.mode: "password"`。

<div id="config-examples">
  ## 配置示例
</div>

<div id="tailnet-only-serve">
  ### 仅限 Tailnet（Serve）
</div>

```json5
{
  gateway: {
    bind: "loopback",
    tailscale: { mode: "serve" }
  }
}
```

访问：`https://<magicdns>/`（或你配置的 `gateway.controlUi.basePath`）


<div id="tailnet-only-bind-to-tailnet-ip">
  ### 仅 Tailnet（绑定到 Tailnet IP）
</div>

当你希望让 Gateway 直接监听 Tailnet IP（不通过 Serve/Funnel）时，使用此配置。

```json5
{
  gateway: {
    bind: "tailnet",
    auth: { mode: "token", token: "your-token" }
  }
}
```

从另一台属于该 Tailnet 的设备进行连接：

* Control UI: `http://<tailscale-ip>:18789/`
* WebSocket: `ws://<tailscale-ip>:18789`

注意：回环地址（`http://127.0.0.1:18789`）在此模式下**无法**使用。


<div id="public-internet-funnel-shared-password">
  ### 公网（Funnel + 共享密码）
</div>

```json5
{
  gateway: {
    bind: "loopback",
    tailscale: { mode: "funnel" },
    auth: { mode: "password", password: "replace-me" }
  }
}
```

优先使用 `OPENCLAW_GATEWAY_PASSWORD`，不要将密码写入磁盘。


<div id="cli-examples">
  ## CLI 示例
</div>

```bash
openclaw gateway --tailscale serve
openclaw gateway --tailscale funnel --auth password
```


<div id="notes">
  ## 注意事项
</div>

- Tailscale Serve/Funnel 需要预先安装并登录 `tailscale` CLI。
- 若认证模式不是 `password`，`tailscale.mode: "funnel"` 将拒绝启动，以避免公开暴露。
- 如果你希望在 Gateway 关闭时让 OpenClaw 撤销 `tailscale serve`
  或 `tailscale funnel` 的配置，请设置 `gateway.tailscale.resetOnExit`。
- `gateway.bind: "tailnet"` 表示直接绑定到 Tailnet（无 HTTPS，也不经过 Serve/Funnel）。
- `gateway.bind: "auto"` 优先使用回环地址；如果你只想通过 Tailnet 访问，请使用 `tailnet`。
- Serve/Funnel 只暴露 **Gateway Control UI + WS**。节点通过同一个 Gateway WS 端点连接，因此 Serve 也可以用于节点访问。

<div id="browser-control-remote-gateway-local-browser">
  ## 浏览器控制（远程 Gateway + 本地浏览器）
</div>

如果你在一台机器上运行 Gateway，但希望在另一台机器上控制浏览器，
请在浏览器所在的机器上运行一个 **节点进程**，并确保两台机器处于同一个 tailnet 中。
Gateway 会将浏览器操作代理到该节点；不需要单独的控制服务器或 Serve URL。

不要使用 Funnel 进行浏览器控制；将节点配对按运维人员访问同等等级对待。

<div id="tailscale-prerequisites-limits">
  ## Tailscale 先决条件与限制
</div>

- 使用 Serve 需要在你的 tailnet 上启用 HTTPS；如果尚未启用，CLI 会提示你。
- Serve 会注入 Tailscale 身份请求头；Funnel 不会。
- 使用 Funnel 需要 Tailscale v1.38.3+、启用 MagicDNS、启用 HTTPS，以及一个 funnel 节点属性。
- Funnel 仅支持通过 TLS 的 `443`、`8443` 和 `10000` 端口。
- 在 macOS 上使用 Funnel 需要开源版 Tailscale 应用。

<div id="learn-more">
  ## 了解更多
</div>

- Tailscale Serve 概览：https://tailscale.com/kb/1312/serve
- `tailscale serve` 命令：https://tailscale.com/kb/1242/tailscale-serve
- Tailscale Funnel 概览：https://tailscale.com/kb/1223/tailscale-funnel
- `tailscale funnel` 命令：https://tailscale.com/kb/1311/tailscale-funnel