---
title: 运行状况
summary: "用于验证渠道连通性的健康检查步骤"
read_when:
  - 诊断 WhatsApp 渠道的运行状况
---

<div id="health-checks-cli">
  # 健康检查（CLI）
</div>

用于在无需猜测的情况下验证各通道连接情况的简明指南。

<div id="quick-checks">
  ## 快速检查
</div>

- `openclaw status` — 本地摘要：Gateway 可达性/模式、更新提示、已连接渠道的认证时长、会话与最近活动。
- `openclaw status --all` — 完整本地诊断（只读、带颜色输出，可以放心粘贴用于调试）。
- `openclaw status --deep` — 还会探测正在运行的 Gateway（在支持的情况下按渠道逐一探测）。
- `openclaw health --json` — 向正在运行的 Gateway 请求完整健康快照（仅支持 WS；不直接访问 Baileys socket）。
- 在 WhatsApp/WebChat 中单独发送 `/status` 消息，可在不调用智能体的情况下获取状态回复。
- 日志：tail `/tmp/openclaw/openclaw-*.log` 并筛选 `web-heartbeat`、`web-reconnect`、`web-auto-reply`、`web-inbound`。

<div id="deep-diagnostics">
  ## 深度诊断
</div>

- 磁盘上的凭证：`ls -l ~/.openclaw/credentials/whatsapp/<accountId>/creds.json`（mtime 应为最近的更新时间）。
- 会话存储：`ls -l ~/.openclaw/agents/<agentId>/sessions/sessions.json`（路径可在配置中覆盖）。通过 `status` 可以查看会话数量和最近的收件人。
- 重新链接流程：当日志中出现状态码 409–515 或 `loggedOut` 时，运行 `openclaw channels logout && openclaw channels login --verbose`。（注意：在配对后，对于状态码 515，二维码登录流程会自动重启一次。）

<div id="when-something-fails">
  ## 出现故障时
</div>

- 显示 `logged out` 或状态码 409–515 → 先运行 `openclaw channels logout`，再运行 `openclaw channels login` 重新关联。
- Gateway 无法访问 → 启动它：`openclaw gateway --port 18789`（如果端口被占用，使用 `--force`）。
- 收不到入站消息 → 确认已关联的手机在线且发送方被允许（`channels.whatsapp.allowFrom`）；对于群聊，确保允许列表和 @ 提及规则配置一致（`channels.whatsapp.groups`、`agents.list[].groupChat.mentionPatterns`）。

<div id="dedicated-health-command">
  ## 专用的 "health" 命令
</div>

`openclaw health --json` 会向正在运行的 Gateway 请求其健康状态快照（CLI 不会直接建立通道套接字连接）。该命令会在可用时报告已关联凭据/认证的存续时间、按通道的探针摘要、会话存储摘要以及探针持续时间。如果 Gateway 不可达，或者探针失败/超时，该命令将以非零状态码退出。使用 `--timeout <ms>` 可以覆盖默认的 10 秒超时设置。