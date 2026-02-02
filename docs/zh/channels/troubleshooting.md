---
title: 故障排查
summary: "各渠道快速故障排查（Discord/Telegram/WhatsApp）"
read_when:
  - 渠道已连接但消息无法收发
  - 排查渠道配置错误（intents、权限、隐私模式）
---

<div id="channel-troubleshooting">
  # 渠道故障排查
</div>

从以下内容开始：

```bash
openclaw doctor
openclaw channels status --probe
```

当 `channels status --probe` 检测到常见的渠道配置错误时，会输出警告，并执行一些轻量级的实时检查（凭证、部分权限/成员关系）。

<div id="channels">
  ## 消息渠道
</div>

* Discord: [/channels/discord#troubleshooting](/zh/channels/discord#troubleshooting)
* Telegram: [/channels/telegram#troubleshooting](/zh/channels/telegram#troubleshooting)
* WhatsApp: [/channels/whatsapp#troubleshooting-quick](/zh/channels/whatsapp#troubleshooting-quick)

<div id="telegram-quick-fixes">
  ## Telegram 快速修复
</div>

* 日志显示 `HttpError: Network request for 'sendMessage' failed` 或 `sendChatAction` → 检查 IPv6 DNS。如果 `api.telegram.org` 优先解析为 IPv6 且主机没有 IPv6 出站能力，则强制使用 IPv4 或启用 IPv6。参见 [/channels/telegram#troubleshooting](/zh/channels/telegram#troubleshooting)。
* 日志显示 `setMyCommands failed` → 检查到 `api.telegram.org` 的出站 HTTPS 和 DNS 连通性（在严格锁定的 VPS 或代理环境中很常见）。