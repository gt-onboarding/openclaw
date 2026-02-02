---
title: 聊天渠道
summary: "OpenClaw 可连接的消息平台"
read_when:
  - 你想为 OpenClaw 选择聊天渠道
  - 你需要快速了解已支持的消息平台
---

<div id="chat-channels">
  # 聊天通道
</div>

OpenClaw 可以在你已经使用的任意聊天应用中与你对话。每个通道都通过 Gateway 接入。
所有通道都支持文本消息；对媒体和表情回应的支持则因通道而异。

<div id="supported-channels">
  ## 支持的渠道
</div>

* [WhatsApp](/zh/channels/whatsapp) — 最常用；使用 Baileys，并需要二维码配对。
* [Telegram](/zh/channels/telegram) — 通过 grammY 使用 Bot API；支持群组。
* [Discord](/zh/channels/discord) — Discord Bot API + Gateway；支持服务器、频道和私信。
* [Slack](/zh/channels/slack) — Bolt SDK；工作区应用。
* [Google Chat](/zh/channels/googlechat) — 通过 HTTP webhook 的 Google Chat API 应用。
* [Mattermost](/zh/channels/mattermost) — Bot API + WebSocket；频道、群组、私信（插件，需单独安装）。
* [Signal](/zh/channels/signal) — 使用 signal-cli；注重隐私。
* [BlueBubbles](/zh/channels/bluebubbles) — **推荐用于 iMessage**；使用 BlueBubbles macOS 服务器 REST API，支持完整特性（编辑、撤回、特效、表情回应、群组管理——编辑在 macOS 26 Tahoe 上目前存在问题）。
* [iMessage](/zh/channels/imessage) — 仅限 macOS；通过 imsg 的原生集成（旧方案，新部署建议使用 BlueBubbles）。
* [Microsoft Teams](/zh/channels/msteams) — Bot Framework；面向企业的支持（插件，需单独安装）。
* [LINE](/zh/channels/line) — LINE Messaging API 机器人（插件，需单独安装）。
* [Nextcloud Talk](/zh/channels/nextcloud-talk) — 基于 Nextcloud Talk 的自托管聊天（插件，需单独安装）。
* [Matrix](/zh/channels/matrix) — Matrix 协议（插件，需单独安装）。
* [Nostr](/zh/channels/nostr) — 通过 NIP-04 的去中心化私信（插件，需单独安装）。
* [Tlon](/zh/channels/tlon) — 基于 Urbit 的消息应用（插件，需单独安装）。
* [Twitch](/zh/channels/twitch) — 通过 IRC 连接的 Twitch 聊天（插件，需单独安装）。
* [Zalo](/zh/channels/zalo) — Zalo Bot API；越南流行的即时通讯应用（插件，需单独安装）。
* [Zalo Personal](/zh/channels/zalouser) — 通过二维码登录的 Zalo 个人账号（插件，需单独安装）。
* [WebChat](/zh/web/webchat) — 通过 WebSocket 的 Gateway WebChat UI。

<div id="notes">
  ## 注意事项
</div>

* 渠道可以同时运行；你可以配置多个渠道，OpenClaw 会按会话分别路由消息。
* 通常配置最快的是 **Telegram**（只需要一个简单的 bot token）。WhatsApp 需要扫码配对，并且
  会在磁盘上存储更多状态数据。
* 群聊行为因渠道而异；参见 [群组](/zh/concepts/groups)。
* 为了安全性，会强制执行私信配对和允许列表；参见 [安全性](/zh/gateway/security)。
* Telegram 内部细节：参见 [grammY 说明](/zh/channels/grammy)。
* 故障排查：参见 [渠道故障排查](/zh/channels/troubleshooting)。
* 模型提供方单独文档说明；参见 [模型提供方](/zh/providers/models)。