---
title: GrammY
summary: "通过 grammY 集成 Telegram Bot API，并附带设置说明"
read_when:
  - 正在处理 Telegram 或 grammY 相关流程时
---

<div id="grammy-integration-telegram-bot-api">
  # grammY 集成（Telegram Bot API）
</div>

<div id="why-grammy">
  # 为什么选择 grammY
</div>

- TypeScript 优先的 Bot API 客户端，内置长轮询 + webhook 辅助工具、中间件、错误处理和限流器。
- 比手写 fetch + FormData 更简洁的媒体处理工具；支持所有 Bot API 方法。
- 可扩展：通过自定义 fetch 支持代理、可选的会话中间件，以及类型安全的上下文。

<div id="what-we-shipped">
  # 我们发布的内容
</div>

- **单一客户端路径：** 已移除基于 `fetch` 的实现；grammY 现在是唯一的 Telegram 客户端（发送 + Gateway），并默认启用 grammY 节流插件（throttler）。
- **Gateway：** `monitorTelegramProvider` 构建一个 grammY 的 `Bot`，接好提及/允许列表控制逻辑，通过 `getFile`/`download` 下载媒体，并使用 `sendMessage/sendPhoto/sendVideo/sendAudio/sendDocument` 发送回复。通过 `webhookCallback` 支持长轮询或 webhook。
- **代理：** 可选的 `channels.telegram.proxy` 通过 grammY 的 `client.baseFetch` 使用 `undici.ProxyAgent`。
- **Webhook 支持：** `webhook-set.ts` 封装 `setWebhook/deleteWebhook`；`webhook.ts` 承载回调并提供健康检查 + 优雅关闭。当设置了 `channels.telegram.webhookUrl` 时，Gateway 启用 webhook 模式（否则使用长轮询）。
- **会话：** 直接聊天汇总到智能体主会话（`agent:<agentId>:<mainKey>`）；群组使用 `agent:<agentId>:telegram:group:<chatId>`；回复会路由回同一通道。
- **配置项：** `channels.telegram.botToken`, `channels.telegram.dmPolicy`, `channels.telegram.groups`（允许列表 + 提及默认值）, `channels.telegram.allowFrom`, `channels.telegram.groupAllowFrom`, `channels.telegram.groupPolicy`, `channels.telegram.mediaMaxMb`, `channels.telegram.linkPreview`, `channels.telegram.proxy`, `channels.telegram.webhookSecret`, `channels.telegram.webhookUrl`。
- **草稿流式输出：** 可选的 `channels.telegram.streamMode` 在私有主题聊天中使用 `sendMessageDraft`（Bot API 9.3+）。这与通道块级流式输出是分开的。
- **测试：** 使用 grammY mock 覆盖私信 + 群组提及门控以及向外发送；欢迎补充更多媒体/webhook 测试用例（fixtures）。

未决问题

- 如果遇到 Bot API 429，是否使用可选的 grammY 插件（throttler）。
- 增加更多结构化的媒体测试（贴纸、语音消息）。
- 让 webhook 监听端口可配置（目前固定为 8787，除非通过 Gateway 转发）。