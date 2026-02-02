---
title: 仪表板
summary: "Gateway 仪表板（Control UI）的访问和身份验证"
read_when:
  - 更改仪表板身份验证或对外暴露模式
---

<div id="dashboard-control-ui">
  # Dashboard（Control UI）
</div>

Gateway 仪表盘是由 Gateway 在 `/`（默认路径）提供的浏览器 Control UI
（可通过 `gateway.controlUi.basePath` 覆盖）。

快速打开（本地 Gateway）：

* http://127.0.0.1:18789/（或 http://localhost:18789/）

关键参考文档：

* [Control UI](/zh/web/control-ui) — 使用方法和 UI 功能说明。
* [Tailscale](/zh/gateway/tailscale) — 用于 Serve/Funnel 自动化。
* [Web surfaces](/zh/web) — 绑定模式与安全说明。

在 WebSocket 握手阶段会通过 `connect.params.auth`（token 或密码）强制进行认证。
参见 [Gateway configuration](/zh/gateway/configuration) 中的 `gateway.auth`。

安全提示：Control UI 是一个**管理界面（admin surface）**（对话、配置、执行操作审批）。
不要将其暴露在公共网络上。UI 会在首次加载后将 token 存入 `localStorage`。
优先使用 localhost、Tailscale Serve 或 SSH 隧道访问。

<div id="fast-path-recommended">
  ## 快速路径（推荐）
</div>

* 完成初始引导后，CLI 会自动使用你的令牌打开仪表盘，并输出同一个带令牌的链接。
* 随时重新打开：`openclaw dashboard`（会复制链接，如果可能则打开浏览器，在无头环境下则显示 SSH 提示）。
* 令牌始终只保留在本地（仅作为查询参数）；UI 在首次加载后会将其从地址中移除，并保存在 localStorage 中。

<div id="token-basics-local-vs-remote">
  ## Token 基础知识（本地 vs 远程）
</div>

* **Localhost**：打开 `http://127.0.0.1:18789/`。如果看到“unauthorized”，运行 `openclaw dashboard` 并使用带有 token 的链接（`?token=...`）。
* **Token 来源**：`gateway.auth.token`（或 `OPENCLAW_GATEWAY_TOKEN`）；UI 在首次加载后会保存它。
* **非 Localhost**：使用 Tailscale Serve（如果 `gateway.auth.allowTailscale: true` 则可免 token 访问）、在 tailnet 中使用 token 进行绑定，或使用 SSH 隧道。参见 [Web 界面](/zh/web)。

<div id="if-you-see-unauthorized-1008">
  ## 如果你看到 “unauthorized” / 1008
</div>

* 运行 `openclaw dashboard` 以获取一条新的带令牌链接。
* 确保 Gateway 可访问（本地：`openclaw status`；远程：先建立 SSH 隧道 `ssh -N -L 18789:127.0.0.1:18789 user@host`，然后打开 `http://127.0.0.1:18789/?token=...`）。
* 在 Dashboard 的设置中，粘贴你在 `gateway.auth.token`（或 `OPENCLAW_GATEWAY_TOKEN`）中配置的同一个令牌。