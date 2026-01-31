---
title: Whatsapp
summary: "WhatsApp（Web 渠道）集成：登录、收件箱、回复、媒体与运维操作"
read_when:
  - 处理 WhatsApp Web 渠道行为或收件箱路由
---

<div id="whatsapp-web-channel">
  # WhatsApp（Web 渠道）
</div>

状态：仅支持通过 Baileys 使用 WhatsApp Web。会话由 Gateway 管理。

<div id="quick-setup-beginner">
  ## 快速设置（入门）
</div>

1. 尽量使用**单独的电话号码**（推荐）。
2. 在 `~/.openclaw/openclaw.json` 中配置 WhatsApp。
3. 运行 `openclaw channels login` 命令以扫描二维码（“已链接的设备”）。
4. 启动 Gateway。

最小配置：

```json5
{
  channels: {
    whatsapp: {
      dmPolicy: "allowlist",  // 私信策略：允许列表
      allowFrom: ["+15551234567"]
    }
  }
}
```

<div id="goals">
  ## 目标
</div>

* 在单个 Gateway 进程中支持多个 WhatsApp 账号（多账号）。
* 确定性路由：回复直接返回到 WhatsApp，不经过模型路由。
* 模型能够看到足够的上下文，从而理解引用回复。

<div id="config-writes">
  ## 配置写入
</div>

默认情况下，允许 WhatsApp 通过 `/config set|unset` 写入配置更新（需要 `commands.config: true`）。

禁用方式：

```json5
{
  channels: { whatsapp: { configWrites: false } }
}
```

<div id="architecture-who-owns-what">
  ## 架构（所有权归属）
</div>

* **Gateway** 持有并管理 Baileys socket 和收件箱循环。
* **CLI / macOS app** 与 Gateway 通信；不会直接使用 Baileys。
* 执行出站发送时必须有一个**活动监听器**；否则发送会立即失败。

<div id="getting-a-phone-number-two-modes">
  ## 获取电话号码（两种模式）
</div>

WhatsApp 需要使用真实的手机号码进行验证。VoIP 和虚拟号码通常会被屏蔽。你可以通过两种受支持的方式在 WhatsApp 上运行 OpenClaw：

<div id="dedicated-number-recommended">
  ### 专用号码（推荐）
</div>

为 OpenClaw 使用一个**单独的手机号**。这样可以获得更好的用户体验、更清晰的路由，并避免各种自聊的怪异情况。理想方案：**备用/旧的 Android 手机 + eSIM**。让它保持连接 Wi‑Fi 和电源，然后通过二维码扫码进行连接。

**WhatsApp Business：**你可以在同一台设备上安装并使用 WhatsApp Business，搭配另一个号码使用。非常适合将你的个人 WhatsApp 与其隔离开来——安装 WhatsApp Business，并在其中注册 OpenClaw 使用的号码。

**示例配置（专用号码，单用户允许列表）：**

```json5
{
  channels: {
    whatsapp: {
      dmPolicy: "allowlist",
      allowFrom: ["+15551234567"]
    }
  }
}
```

**配对模式（可选）：**
如果你想使用配对而不是允许列表，将 `channels.whatsapp.dmPolicy` 设为 `pairing`。未知发送方会收到一个配对码；使用以下命令予以批准：
`openclaw pairing approve whatsapp <code>`

<div id="personal-number-fallback">
  ### 个人号码（备用方案）
</div>

快速备用方案：在**你自己的号码**上运行 OpenClaw。先给自己发消息（使用 WhatsApp 的“Message yourself”功能）进行测试，这样就不会打扰联系人。在设置和试验过程中，你需要在主手机上查看验证码。**必须启用自聊模式（self-chat mode）。**
当向导询问你的个人 WhatsApp 号码时，请输入你将用来发送消息的手机号码（所有者/发送方），而不是助手的号码。

**示例配置（个人号码，自聊）：**

```json
{
  "whatsapp": {
    "selfChatMode": true,
    "dmPolicy": "allowlist",
    "allowFrom": ["+15551234567"]
  }
}
```

当 `messages.responsePrefix` 未设置时，自聊回复默认使用前缀 `[{identity.name}]`（否则为 `[openclaw]`）。显式设置该值即可自定义或禁用前缀（将其设为 `""` 可移除前缀）。

<div id="number-sourcing-tips">
  ### 号码来源建议
</div>

* 来自你所在国家移动运营商的**本地 eSIM**（最可靠）
  * 奥地利：[hot.at](https://www.hot.at)
  * 英国：[giffgaff](https://www.giffgaff.com) — 免费 SIM，无合约
* **预付费 SIM 卡** — 价格便宜，只需要接收一条用于验证的短信即可

**避免使用：**TextNow、Google Voice 以及大多数“免费短信”服务 — WhatsApp 会严厉封禁这些号码。

**提示：**该号码只需要接收一次验证短信。之后，WhatsApp Web 会话会通过 `creds.json` 持久保持。

<div id="why-not-twilio">
  ## 为什么不用 Twilio？
</div>

* 早期版本的 OpenClaw 曾支持 Twilio 的 WhatsApp Business 集成。
* WhatsApp Business 号码并不适合作为个人助手的号码。
* Meta 强制执行 24 小时回复窗口；如果你在过去 24 小时内没有回复，该企业号码就不能发起新的消息。
* 高频或“聊天很多”的使用会触发非常激进的封锁，因为企业账号并不是用来发送大量个人助手消息的。
* 结果：消息投递不可靠且经常被封锁，因此我们移除了相关支持。

<div id="login-credentials">
  ## 登录与凭据
</div>

* 登录命令：`openclaw channels login`（通过“已链接设备”扫描二维码）。
* 多账户登录：`openclaw channels login --account <id>`（`<id>` = `accountId`）。
* 默认账户（省略 `--account` 时）：如果存在 `default`，则使用它，否则使用按排序结果的首个已配置账户 ID。
* 凭据存储在 `~/.openclaw/credentials/whatsapp/<accountId>/creds.json`。
* 备份副本位于 `creds.json.bak`（在损坏时用于恢复）。
* 旧版兼容性：更早的安装会将 Baileys 文件直接存放在 `~/.openclaw/credentials/` 中。
* 登出：`openclaw channels logout`（或 `--account <id>`）会删除 WhatsApp 认证状态（但保留共享的 `oauth.json`）。
* 如果 socket 已登出，将返回错误并提示重新链接。

<div id="inbound-flow-dm-group">
  ## 入站流程（私信 DM + 群组）
</div>

* WhatsApp 事件来自 `messages.upsert`（Baileys）。
* 关闭时会注销收件箱监听器，以避免在测试或重启过程中累积事件处理器。
* 会忽略状态/广播类会话。
* 私聊使用 E.164；群聊使用群组 JID。
* **私信策略（DM policy）**：`channels.whatsapp.dmPolicy` 控制私聊访问（默认：`pairing`）。
  * 配对（pairing）：未知发送者会收到一个配对码（通过 `openclaw pairing approve whatsapp <code>` 审批；配对码在 1 小时后过期）。
  * Open（`open`）：`open` 设置表示可以不受限制地接受任意用户的消息；需要在 `channels.whatsapp.allowFrom` 中包含 `"*"`。
  * 你绑定的 WhatsApp 号码被隐式信任，因此你给自己发送的消息会跳过 `channels.whatsapp.dmPolicy` 和 `channels.whatsapp.allowFrom` 的检查。

<div id="personal-number-mode-fallback">
  ### 个人号码模式（兜底）
</div>

如果你在**个人 WhatsApp 号码**上运行 OpenClaw，请启用 `channels.whatsapp.selfChatMode`（见上面的示例）。

行为：

* 发出的私信永远不会触发配对回复（以避免骚扰联系人）。
* 来自未知发件人的入站消息仍然遵循 `channels.whatsapp.dmPolicy`。
* 自聊模式（`allowFrom` 包含你的号码）会避免自动发送已读回执，并忽略提及的 JID。
* 会为非自聊私信发送已读回执。

<div id="read-receipts">
  ## 已读回执
</div>

默认情况下，一旦接收到并接受 WhatsApp 入站消息，Gateway 就会将其标记为已读（蓝色双勾）。

全局禁用：

```json5
{
  channels: { whatsapp: { sendReadReceipts: false } }
}
```

按账号单独禁用：

```json5
{
  channels: {
    whatsapp: {
      accounts: {
        personal: { sendReadReceipts: false }
      }
    }
  }
}
```

注意：

* 自聊模式始终跳过已读回执。

<div id="whatsapp-faq-sending-messages-pairing">
  ## WhatsApp 常见问题：发送消息与配对
</div>

**当我连接 WhatsApp 时，OpenClaw 会给随机联系人发消息吗？**\
不会。默认的私信（DM）策略是 **配对**，因此未知发送者只会收到一个配对码，他们的消息**不会被处理**。OpenClaw 只会回复它收到的聊天，或者你通过（智能体/CLI）显式触发的发送操作。

**配对在 WhatsApp 上是如何工作的？**\
配对是针对未知发送者的私信（DM）入口门控机制：

* 新发送者的第一条私信会收到一个短验证码（消息不会被处理）。
* 使用以下命令批准：`openclaw pairing approve whatsapp <code>`（使用 `openclaw pairing list whatsapp` 查看列表）。
* 验证码在 1 小时后过期；每个渠道的待处理请求数量上限为 3 个。

**多个用户能在同一个 WhatsApp 号码上使用不同的 OpenClaw 实例吗？**\
可以，可以通过 `bindings` 将每个发送者路由到不同的智能体（对端 `kind: "dm"`，发送者 E.164 号码如 `+15551234567`）。回复仍然来自**同一个 WhatsApp 账号**，并且一对一私信会折叠到各自智能体的主会话中，因此建议**每人一个智能体**。私信访问控制（`dmPolicy`/`allowFrom`）在每个 WhatsApp 账号上是全局配置。参见 [多智能体路由](/zh/concepts/multi-agent)。

**为什么向导要询问我的电话号码？**\
向导会用它来设置你的 **允许列表/owner**，以便允许你自己的私信通过。该号码不会被用于自动发送。如果你在个人 WhatsApp 号码上运行，请使用同一个号码，并启用 `channels.whatsapp.selfChatMode`。

<div id="message-normalization-what-the-model-sees">
  ## 消息规范化（模型所看到的内容）
</div>

* `Body` 是当前消息正文（包含 envelope）。
* 引用回复上下文**始终会被追加**：
  ```
  [Replying to +1555 id:ABC123]
  <quoted text or <media:...>>
  [/Replying]
  ```
* 同时会设置回复元数据：
  * `ReplyToId` = stanzaId
  * `ReplyToBody` = 引用的正文或媒体占位符
  * `ReplyToSender` = 已知时为 E.164 格式
* 仅包含媒体的入站消息会使用占位符：
  * `<media:image|video|audio|document|sticker>`

<div id="groups">
  ## 群组
</div>

* 群组映射到 `agent:<agentId>:whatsapp:group:<jid>` 会话。
* 群组策略：`channels.whatsapp.groupPolicy = open|disabled|allowlist`（默认值为 `allowlist`）。
* 激活模式：
  * `mention`（默认）：需要 @mention 或正则表达式匹配。
  * `always`：始终触发。
* `/activation mention|always` 仅限所有者使用，且必须作为一条独立消息发送。
* 所有者 = `channels.whatsapp.allowFrom`（如果未设置，则为自身的 E.164 号码）。
* **历史记录注入**（仅对挂起消息适用）：
  * 最近的 *未处理* 消息（默认 50 条）会插入在：
    `[Chat messages since your last reply - for context]`（已在会话中的消息不会被重新注入）
  * 当前消息会插入在：
    `[Current message - respond to this]`
  * 追加发送者后缀：`[from: Name (+E164)]`
* 群组元数据缓存 5 分钟（群标题 + 成员）。

<div id="reply-delivery-threading">
  ## 回复投递（线程）
</div>

* WhatsApp Web 只会发送普通消息（当前 Gateway 不支持带引用的回复线程）。
* 本通道上的回复标记会被忽略。

<div id="acknowledgment-reactions-auto-react-on-receipt">
  ## 确认类表情反应（接收时自动表情）
</div>

WhatsApp 可以在收到传入消息后立即自动发送表情反应，在机器人生成回复之前完成。这样可以即时向用户反馈他们的消息已被接收。

**配置：**

```json
{
  "whatsapp": {
    "ackReaction": {
      "emoji": "👀",
      "direct": true,
      "group": "mentions"
    }
  }
}
```

**选项：**

* `emoji` (string)：用于确认的表情符号（例如 “👀”、“✅”、“📨”）。为空或省略 = 禁用该功能。
* `direct` (boolean，默认：`true`)：在私聊/DM 会话中发送表情回应。
* `group` (string，默认：`"mentions"`)：群聊行为：
  * `"always"`：对所有群消息发送表情回应（即使没有 @ 提及）
  * `"mentions"`：仅在机器人被 @ 提及时发送表情回应
  * `"never"`：从不在群聊中发送表情回应

**按账号覆盖：**

```json
{
  "whatsapp": {
    "accounts": {
      "work": {
        "ackReaction": {
          "emoji": "✅",
          "direct": false,
          "group": "always"
        }
      }
    }
  }
}
```

**行为说明：**

* 在收到消息后，Reaction 会**立即**发送，早于正在输入指示器或机器人回复。
* 在 `requireMention: false`（激活：始终）的群组中，`group: "mentions"` 会对所有消息添加 Reaction（而不仅仅是 @ 提及）。
* 即发即忘：Reaction 失败会被记录到日志中，但不会阻止机器人回复。
* 群组 Reaction 会自动附带参与者 JID。
* WhatsApp 会忽略 `messages.ackReaction`；请改用 `channels.whatsapp.ackReaction`。

<div id="agent-tool-reactions">
  ## Agent 工具（表情回应）
</div>

* 工具：`whatsapp`，使用 `react` 动作（`chatJid`、`messageId`、`emoji`，可选 `remove`）。
* 可选字段：`participant`（群聊消息的发送者）、`fromMe`（对你自己的消息做出表情回应）、`accountId`（多账号）。
* 移除表情的语义：参见 [/tools/reactions](/zh/tools/reactions)。
* 工具权限控制：`channels.whatsapp.actions.reactions`（默认：启用）。

<div id="limits">
  ## 限制
</div>

* 出站文本会按 `channels.whatsapp.textChunkLimit`（默认 4000）进行分块。
* 可选按换行分块：将 `channels.whatsapp.chunkMode="newline"` 设置为按空行（段落边界）拆分，然后再进行长度分块。
* 入站媒体保存大小上限由 `channels.whatsapp.mediaMaxMb` 限制（默认 50 MB）。
* 出站媒体内容大小上限由 `agents.defaults.mediaMaxMb` 限制（默认 5 MB）。

<div id="outbound-send-text-media">
  ## 出站发送（文本 + 媒体）
</div>

* 使用已激活的 Web 监听器；如果 Gateway 未运行则报错。
* 文本分块：每条消息最多 4k（可通过 `channels.whatsapp.textChunkLimit` 配置，可选 `channels.whatsapp.chunkMode`）。
* 媒体：
  * 支持图片/视频/音频/文档。
  * 音频以 PTT 形式发送；`audio/ogg` =&gt; `audio/ogg; codecs=opus`。
  * 仅第一条媒体可以携带说明文字（caption）。
  * 媒体获取支持 HTTP(S) 和本地路径。
  * 动画 GIF：WhatsApp 期望使用带有 `gifPlayback: true` 的 MP4 以实现内联循环播放。
    * CLI：`openclaw message send --media <mp4> --gif-playback`
    * Gateway：`send` 参数包括 `gifPlayback: true`

<div id="voice-notes-ptt-audio">
  ## 语音消息（PTT 音频）
</div>

WhatsApp 会将音频作为 **语音消息**（PTT 气泡）发送。

* 为获得最佳效果，使用 OGG/Opus。OpenClaw 会将 `audio/ogg` 重写为 `audio/ogg; codecs=opus`。
* 在 WhatsApp 中，`[[audio_as_voice]]` 会被忽略（音频已经以语音消息形式发送）。

<div id="media-limits-optimization">
  ## 媒体限制与优化
</div>

* 默认出站上限：5 MB（每个媒体内容）。
* 可通过 `agents.defaults.mediaMaxMb` 覆盖默认值。
* 图片会在不超过上限的前提下自动优化为 JPEG（调整尺寸 + 质量调整）。
* 超出大小限制的媒体 =&gt; 报错；媒体回复会退回为文本警告。

<div id="heartbeats">
  ## 心跳
</div>

* **Gateway 心跳** 会记录连接健康状态（`web.heartbeatSeconds`，默认 60 秒）。
* **Agent 代理心跳** 可以按每个智能体配置（`agents.list[].heartbeat`），也可以通过
  全局的 `agents.defaults.heartbeat` 配置（在未设置单个智能体项时作为回退）。
  * 使用已配置的心跳提示词（默认：`Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.`），并支持 `HEARTBEAT_OK` 跳过行为。
  * 默认发送到上一次使用的渠道（或已配置的目标渠道）。

<div id="reconnect-behavior">
  ## 重连行为
</div>

* 退避策略：`web.reconnect`：
  * `initialMs`、`maxMs`、`factor`、`jitter`、`maxAttempts`。
* 如果已达到 `maxAttempts`，则停止 web 监控（降级）。
* 登出后 =&gt; 停止并要求重新关联。

<div id="config-quick-map">
  ## 配置速查表
</div>

* `channels.whatsapp.dmPolicy`（私信策略：pairing/allowlist/open/disabled）。
* `channels.whatsapp.selfChatMode`（同号部署；bot 使用你的个人 WhatsApp 号码）。
* `channels.whatsapp.allowFrom`（私信允许列表）。WhatsApp 使用 E.164 电话号码（不支持用户名）。
* `channels.whatsapp.mediaMaxMb`（入站媒体保存大小上限）。
* `channels.whatsapp.ackReaction`（收到消息时的自动表情回应：`{emoji, direct, group}`）。
* `channels.whatsapp.accounts.<accountId>.*`（每个账号的设置 + 可选的 `authDir`）。
* `channels.whatsapp.accounts.<accountId>.mediaMaxMb`（每个账号的入站媒体大小上限）。
* `channels.whatsapp.accounts.<accountId>.ackReaction`（每个账号的确认表情回应覆盖配置）。
* `channels.whatsapp.groupAllowFrom`（群聊发送者允许列表）。
* `channels.whatsapp.groupPolicy`（群聊策略）。
* `channels.whatsapp.historyLimit` / `channels.whatsapp.accounts.<accountId>.historyLimit`（群聊历史上下文；`0` 表示禁用）。
* `channels.whatsapp.dmHistoryLimit`（按用户轮次计的私信历史上限）。每个用户的覆盖配置：`channels.whatsapp.dms["<phone>"].historyLimit`。
* `channels.whatsapp.groups`（群聊允许列表 + 提及门控默认值；使用 `"*"` 允许所有）。
* `channels.whatsapp.actions.reactions`（控制 WhatsApp 工具的表情回应）。
* `agents.list[].groupChat.mentionPatterns`（或 `messages.groupChat.mentionPatterns`）
* `messages.groupChat.historyLimit`
* `channels.whatsapp.messagePrefix`（入站前缀；每个账号：`channels.whatsapp.accounts.<accountId>.messagePrefix`；已废弃：`messages.messagePrefix`）
* `messages.responsePrefix`（出站前缀）
* `agents.defaults.mediaMaxMb`
* `agents.defaults.heartbeat.every`
* `agents.defaults.heartbeat.model`（可选覆盖配置）
* `agents.defaults.heartbeat.target`
* `agents.defaults.heartbeat.to`
* `agents.defaults.heartbeat.session`
* `agents.list[].heartbeat.*`（每个智能体的覆盖配置）
* `session.*`（scope、空闲、存储、mainKey）
* `web.enabled`（为 false 时不启动该渠道）
* `web.heartbeatSeconds`
* `web.reconnect.*`

<div id="logs-troubleshooting">
  ## 日志和故障排查
</div>

* 子系统：`whatsapp/inbound`、`whatsapp/outbound`、`web-heartbeat`、`web-reconnect`。
* 日志文件：`/tmp/openclaw/openclaw-YYYY-MM-DD.log`（可配置）。
* 故障排查指南：[Gateway 故障排查](/zh/gateway/troubleshooting)。

<div id="troubleshooting-quick">
  ## 故障排查（快速）
</div>

**未链接 / 需要二维码登录**

* 症状：`channels status` 显示 `linked: false` 或警告 “Not linked”。
* 解决方法：在 Gateway 主机上运行 `openclaw channels login`，然后扫码登录（WhatsApp → Settings → Linked Devices）。

**已链接但断开 / 循环重连**

* 症状：`channels status` 显示 `running, disconnected` 或警告 “Linked but disconnected”。
* 解决方法：运行 `openclaw doctor`（或重启 Gateway）。如果问题仍然存在，使用 `channels login` 重新链接，并查看 `openclaw logs --follow` 日志。

**Bun 运行时**

* **不推荐** 使用 Bun。WhatsApp（Baileys）和 Telegram 在 Bun 上表现不稳定。
  请使用 **Node** 运行 Gateway。（参见“快速上手”中的运行时说明。）