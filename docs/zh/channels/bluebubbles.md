---
title: Bluebubbles
summary: "通过 BlueBubbles macOS 服务器使用 iMessage（REST 发送/接收、输入状态、表情回复、配对、高级操作）。"
read_when:
  - 设置 BlueBubbles 通道
  - 排查 webhook 配对问题
  - 在 macOS 上配置 iMessage
---

<div id="bluebubbles-macos-rest">
  # BlueBubbles（macOS REST）
</div>

状态：内置插件，通过 HTTP 与 BlueBubbles macOS 服务器通信。**推荐用于 iMessage 集成**，因为其 API 更加丰富，且相比旧版 imsg 渠道更易于设置。

<div id="overview">
  ## 概述
</div>

* 通过 BlueBubbles 辅助应用在 macOS 上运行（[bluebubbles.app](https://bluebubbles.app)）。
* 推荐/已测试：macOS Sequoia (15)。macOS Tahoe (26) 也可运行；当前在 Tahoe 上编辑功能不可用，群组图标更新可能显示成功但不会同步。
* OpenClaw 通过其 REST API 与其通信（`GET /api/v1/ping`、`POST /message/text`、`POST /chat/:id/*`）。
* 传入消息通过 webhooks 接收；传出回复、正在输入指示、已读回执和 tapback（快速表情反馈）通过 REST 调用完成。
* 附件和贴纸会作为入站媒体被接收（并在可能的情况下暴露给智能体）。
* 配对/允许列表的工作方式与其他通道相同（`/start/pairing` 等），通过 `channels.bluebubbles.allowFrom` + 配对码实现。
* 表情反应会和 Slack/Telegram 一样作为系统事件上报，因此智能体可以在回复前“提及”它们。
* 高级功能：编辑、撤回（取消发送）、线程式回复、消息特效、群组管理。

<div id="quick-start">
  ## 快速开始
</div>

1. 在你的 Mac 上安装 BlueBubbles 服务器（按照 [bluebubbles.app/install](https://bluebubbles.app/install) 上的说明操作）。
2. 在 BlueBubbles 配置中启用 Web API 并设置密码。
3. 运行 `openclaw onboard` 并选择 BlueBubbles，或手动配置：
   ```json5
   {
     channels: {
       bluebubbles: {
         enabled: true,
         serverUrl: "http://192.168.1.100:1234",
         password: "example-password",
         webhookPath: "/bluebubbles-webhook"
       }
     }
   }
   ```
4. 将 BlueBubbles 的 webhook 指向你的 Gateway（示例：`https://your-gateway-host:3000/bluebubbles-webhook?password=<password>`）。
5. 启动 Gateway；它会注册 webhook 处理器并开始配对。

<div id="onboarding">
  ## 入门
</div>

你可以在交互式设置向导中完成 BlueBubbles 的配置：

```
openclaw onboard
```

向导会提示你输入：

* **Server URL**（必填）：BlueBubbles 服务器地址（例如 `http://192.168.1.100:1234`）
* **Password**（必填）：BlueBubbles Server 设置中的 API 密码
* **Webhook path**（可选）：默认为 `/bluebubbles-webhook`
* **DM policy**：配对、允许列表、open（允许任何用户无限制地发送消息）或禁用
* **Allow list**：电话号码、电子邮件地址或聊天对象

你也可以通过 CLI 添加 BlueBubbles：

```
openclaw channels add bluebubbles --http-url http://192.168.1.100:1234 --password <password>
```

<div id="access-control-dms-groups">
  ## 访问控制（私信 + 群组）
</div>

私信（DM）：

* 默认：`channels.bluebubbles.dmPolicy = "pairing"`。
* 未知发件人会收到一个配对码；在你批准之前，其消息都会被忽略（配对码 1 小时后过期）。
* 通过以下方式批准：
  * `openclaw pairing list bluebubbles`
  * `openclaw pairing approve bluebubbles <CODE>`
* 配对是默认的令牌交换方式。详情见：[配对](/zh/start/pairing)

群组：

* `channels.bluebubbles.groupPolicy = open | allowlist | disabled`（默认：`allowlist`）。其中 `open` 表示允许从任何用户不受限制地接收消息。
* 当 `allowlist` 已设置时，`channels.bluebubbles.groupAllowFrom` 用于控制哪些人可以在群组中触发。

<div id="mention-gating-groups">
  ### 提及门控（群组）
</div>

BlueBubbles 在群聊中支持提及门控，与 iMessage/WhatsApp 的行为一致：

* 使用 `agents.list[].groupChat.mentionPatterns`（或 `messages.groupChat.mentionPatterns`）来检测提及。
* 当为某个群组启用 `requireMention` 时，智能体只有在被提及时才会回复。
* 来自已授权发送者的控制命令会绕过提及门控。

按群组进行配置：

```json5
{
  channels: {
    bluebubbles: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15555550123"],
      groups: {
        "*": { requireMention: true },  // default for all groups
        "iMessage;-;chat123": { requireMention: false }  // 特定群组的覆盖设置
      }
    }
  }
}
```

<div id="command-gating">
  ### 命令访问控制
</div>

* 控制命令（例如 `/config`、`/model`）需要授权。
* 使用 `allowFrom` 和 `groupAllowFrom` 来确定命令授权。
* 已授权的发送方即使在群组中未被提及（未被 @），也可以运行控制命令。

<div id="typing-read-receipts">
  ## 正在输入指示 + 已读回执
</div>

* **正在输入指示**：在生成回复之前和生成过程中自动发送。
* **已读回执**：由 `channels.bluebubbles.sendReadReceipts` 控制（默认：`true`）。
* **正在输入指示**：OpenClaw 会发送“开始输入”事件；BlueBubbles 会在发送消息或超时后自动清除正在输入状态（通过 DELETE 手动停止并不可靠）。

```json5
{
  channels: {
    bluebubbles: {
      sendReadReceipts: false  // 禁用已读回执
    }
  }
}
```

<div id="advanced-actions">
  ## 高级操作
</div>

在配置中启用相关选项后，BlueBubbles 支持高级消息操作：

```json5
{
  channels: {
    bluebubbles: {
      actions: {
        reactions: true,       // 点按回应(默认: true)
        edit: true,            // 编辑已发送的消息(macOS 13+,在 macOS 26 Tahoe 上损坏)
        unsend: true,          // 撤回消息(macOS 13+)
        reply: true,           // 通过消息 GUID 进行回复串联
        sendWithEffect: true,  // 消息效果(重击、响亮等)
        renameGroup: true,     // 重命名群组聊天
        setGroupIcon: true,    // 设置群组聊天图标/照片(在 macOS 26 Tahoe 上不稳定)
        addParticipant: true,  // 向群组添加参与者
        removeParticipant: true, // 从群组移除参与者
        leaveGroup: true,      // 离开群组聊天
        sendAttachment: true   // 发送附件/媒体
      }
    }
  }
}
```

可用操作：

* **react**: 添加/移除 tapback 表情反应（`messageId`、`emoji`、`remove`）
* **edit**: 编辑已发送的消息（`messageId`、`text`）
* **unsend**: 撤回消息（`messageId`）
* **reply**: 回复指定消息（`messageId`、`text`、`to`）
* **sendWithEffect**: 使用 iMessage 特效发送（`text`、`to`、`effectId`）
* **renameGroup**: 重命名群聊（`chatGuid`、`displayName`）
* **setGroupIcon**: 设置群聊图标/照片（`chatGuid`、`media`）——在 macOS 26 Tahoe 上表现不稳定（API 可能返回成功，但图标不会同步）。
* **addParticipant**: 向群聊中添加成员（`chatGuid`、`address`）
* **removeParticipant**: 将成员从群聊中移除（`chatGuid`、`address`）
* **leaveGroup**: 退出群聊（`chatGuid`）
* **sendAttachment**: 发送媒体/文件（`to`、`buffer`、`filename`、`asVoice`）
  * 语音备忘录：将 `asVoice: true` 与 **MP3** 或 **CAF** 音频一起设置，以 iMessage 语音消息的形式发送。BlueBubbles 在发送语音备忘录时会将 MP3 转换为 CAF。

<div id="message-ids-short-vs-full">
  ### 消息 ID（短 ID 与完整 ID）
</div>

OpenClaw 可能会显示*短*消息 ID（例如 `1`、`2`）以节省 token。

* `MessageSid` / `ReplyToId` 可以是短 ID。
* `MessageSidFull` / `ReplyToIdFull` 包含提供方的完整 ID。
* 短 ID 只存在于内存中；在重启或缓存淘汰后可能会失效。
* 操作可以接受短或完整的 `messageId`，但如果短 ID 已不可用，将会报错。

在需要长期可靠的自动化与存储场景中，请使用完整 ID：

* 模板：`{{MessageSidFull}}`、`{{ReplyToIdFull}}`
* 上下文：入站负载中的 `MessageSidFull` / `ReplyToIdFull`

模板变量详见 [Configuration](/zh/gateway/configuration)。

<div id="block-streaming">
  ## 分块流式传输
</div>

控制响应是作为单条消息一次性发送，还是以分块流式传输的方式发送：

```json5
{
  channels: {
    bluebubbles: {
      blockStreaming: true  // 启用块流式传输（默认行为）
    }
  }
}
```

<div id="media-limits">
  ## 媒体与限制
</div>

* 入站附件会被下载并存储到媒体缓存中。
* 通过 `channels.bluebubbles.mediaMaxMb` 控制媒体大小上限（默认：8 MB）。
* 出站文本会根据 `channels.bluebubbles.textChunkLimit` 拆分为多段（默认：4000 个字符）。

<div id="configuration-reference">
  ## 配置参考
</div>

完整配置： [Configuration](/zh/gateway/configuration)

提供方配置项：

* `channels.bluebubbles.enabled`：启用/禁用该通道。
* `channels.bluebubbles.serverUrl`：BlueBubbles REST API 基础 URL。
* `channels.bluebubbles.password`：API 密码。
* `channels.bluebubbles.webhookPath`：Webhook 端点路径（默认：`/bluebubbles-webhook`）。
* `channels.bluebubbles.dmPolicy`：`pairing | allowlist | open | disabled`（默认：`pairing`）。
* `channels.bluebubbles.allowFrom`：DM 允许列表（handles、电子邮件、E.164 号码、`chat_id:*`、`chat_guid:*`）。
* `channels.bluebubbles.groupPolicy`：`open | allowlist | disabled`（默认：`allowlist`）。
* `channels.bluebubbles.groupAllowFrom`：群组发送者允许列表。
* `channels.bluebubbles.groups`：按群组配置（`requireMention` 等）。
* `channels.bluebubbles.sendReadReceipts`：发送已读回执（默认：`true`）。
* `channels.bluebubbles.blockStreaming`：启用块流式传输（默认：`true`）。
* `channels.bluebubbles.textChunkLimit`：出站文本块大小（字符数）（默认：4000）。
* `channels.bluebubbles.chunkMode`：`length`（默认）仅在超过 `textChunkLimit` 时分块；`newline` 会先按空行（段落边界）分块，再按长度分块。
* `channels.bluebubbles.mediaMaxMb`：入站媒体大小上限（MB）（默认：8）。
* `channels.bluebubbles.historyLimit`：用于上下文的群组消息最大数量（0 表示禁用）。
* `channels.bluebubbles.dmHistoryLimit`：DM 历史记录上限。
* `channels.bluebubbles.actions`：启用/禁用特定操作。
* `channels.bluebubbles.accounts`：多账户配置。

相关全局选项：

* `agents.list[].groupChat.mentionPatterns`（或 `messages.groupChat.mentionPatterns`）。
* `messages.responsePrefix`。

<div id="addressing-delivery-targets">
  ## 寻址 / 投递目标
</div>

进行稳定路由时，优先使用 `chat_guid`：

* `chat_guid:iMessage;-;+15555550123`（群聊时首选）
* `chat_id:123`
* `chat_identifier:...`
* 直接 handle：`+15555550123`、`user@example.com`
  * 如果某个直接 handle 尚不存在对应的私信（DM）会话，OpenClaw 会通过 `POST /api/v1/chat/new` 创建一个。此操作需要先启用 BlueBubbles Private API。

<div id="security">
  ## 安全性
</div>

* 通过将 `guid`/`password` 查询参数或请求头与 `channels.bluebubbles.password` 进行比较来对 Webhook 请求进行身份验证。来自 `localhost` 的请求也会被接受。
* 保持 API 密码和 Webhook 端点的机密性（将它们视为凭据）。
* 对 localhost 的信任意味着同一主机上的反向代理可能在无意间绕过密码验证。如果你为 Gateway 配置反向代理，请在代理层强制要求认证，并配置 `gateway.trustedProxies`。参见 [Gateway 安全性](/zh/gateway/security#reverse-proxy-configuration)。
* 如果将 BlueBubbles 服务器暴露到局域网外（例如公网），请启用 HTTPS 并配置防火墙规则。

<div id="troubleshooting">
  ## 疑难解答
</div>

* 如果输入状态/`read` 事件停止工作，请检查 BlueBubbles webhook 日志，并确认 Gateway 路径与 `channels.bluebubbles.webhookPath` 匹配。
* 配对码在一小时后过期；请使用 `openclaw pairing list bluebubbles` 和 `openclaw pairing approve bluebubbles <code>`。
* 表情回应需要 BlueBubbles 私有 API（`POST /api/v1/message/react`）；请确保所用服务器版本已暴露该接口。
* 编辑/撤回需要 macOS 13+ 和兼容的 BlueBubbles 服务器版本。在 macOS 26（Tahoe）上，由于私有 API 变更，编辑功能当前不可用。
* 在 macOS 26（Tahoe）上，群组图标更新可能不稳定：API 可能返回成功，但新图标并不会同步。
* OpenClaw 会根据 BlueBubbles 服务器所运行的 macOS 版本自动隐藏已知失效的操作。如果在 macOS 26（Tahoe）上仍然出现编辑操作，请通过 `channels.bluebubbles.actions.edit=false` 手动禁用。
* 查看状态/运行状况信息：`openclaw status --all` 或 `openclaw status --deep`。

有关通道整体工作流的参考，请参见 [Channels](/zh/channels) 和 [Plugins](/zh/plugins) 指南。