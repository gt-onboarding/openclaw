---
title: Telegram
summary: "Telegram 机器人支持状态、功能与配置"
read_when:
  - 正在开发或调试 Telegram 功能或 webhooks
---

<div id="telegram-bot-api">
  # Telegram（Bot API）
</div>

状态：已可用于通过 grammY 进行机器人私聊和群组对话的生产环境。默认使用长轮询；webhook 为可选项。

<div id="quick-setup-beginner">
  ## 快速设置（入门）
</div>

1. 使用 **@BotFather** 创建一个 bot，并复制 token。
2. 设置 token：
   * 环境变量：`TELEGRAM_BOT_TOKEN=...`
   * 或配置：`channels.telegram.botToken: "..."`。
   * 如果两者都设置了，则以配置为准（环境变量仅作为默认账户的兜底值）。
3. 启动 Gateway。
4. 私信（DM）访问默认采用配对模式；首次联系时需要批准配对码。

最小配置：

```json5
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "123:abc",
      dmPolicy: "pairing"
    }
  }
}
```

<div id="what-it-is">
  ## 这是什么
</div>

* 由 Gateway 管理的 Telegram Bot API 通道。
* 确定性路由：回复只会发回 Telegram；模型不会自行选择通道。
* 私聊共享该智能体的主会话；群组保持相互隔离（`agent:<agentId>:telegram:group:<chatId>`）。

<div id="setup-fast-path">
  ## 快速配置
</div>

<div id="1-create-a-bot-token-botfather">
  ### 1) 创建机器人令牌（BotFather）
</div>

1. 打开 Telegram，并与 **@BotFather** 开始聊天。
2. 执行 `/newbot`，然后按照提示操作（设置名称 + 以 `bot` 结尾的用户名）。
3. 复制令牌并妥善保存。

可选的 BotFather 设置：

* `/setjoingroups` — 允许/禁止将机器人添加到群组。
* `/setprivacy` — 控制机器人是否能看到所有群组消息。

<div id="2-configure-the-token-env-or-config">
  ### 2) 配置 Token（环境变量或配置）
</div>

示例：

```json5
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "123:abc",
      dmPolicy: "配对",
      groups: { "*": { requireMention: true } }
    }
  }
}
```

环境变量选项：`TELEGRAM_BOT_TOKEN=...`（适用于默认账号）。
如果同时设置了环境变量和配置项，则以配置项为准。

多账号支持：使用 `channels.telegram.accounts` 配置每个账号的 token 和可选的 `name`。通用模式参见 [`gateway/configuration`](/zh/gateway/configuration#telegramaccounts--discordaccounts--slackaccounts--signalaccounts--imessageaccounts)。

3. 启动 Gateway。Telegram 会在解析到 token 时启动（优先使用配置，其次回退为环境变量）。
4. 私聊（DM）访问默认为配对流程。首次联系 bot 时，批准其显示的配对码。
5. 群组场景：先把 bot 加入群组，按需决定隐私/管理员行为（见下文），然后设置 `channels.telegram.groups` 以控制 @ 提及访问控制和允许列表。

<div id="token-privacy-permissions-telegram-side">
  ## Token + 隐私 + 权限（Telegram 侧）
</div>

<div id="token-creation-botfather">
  ### 令牌创建（BotFather）
</div>

* 使用 `/newbot` 创建机器人账号，并会返回令牌（请妥善保密）。
* 如果令牌泄露，请通过 @BotFather 撤销或重新生成令牌，并更新你的配置。

<div id="group-message-visibility-privacy-mode">
  ### 群组消息可见性（隐私模式）
</div>

Telegram 机器人默认启用 **隐私模式（Privacy Mode）**，这会限制它们能接收的群组消息。
如果你的机器人需要看到 *所有* 群组消息，你有两种选择：

* 使用 `/setprivacy` 禁用隐私模式，**或**
* 将机器人设置为群组**管理员**（管理员机器人会接收所有消息）。

**注意：**当你切换隐私模式时，Telegram 要求你在每个群组中先移除再重新添加机器人，
更改才能生效。

<div id="group-permissions-admin-rights">
  ### 群组权限（管理员权限）
</div>

管理员状态在群组（Telegram UI）中进行设置。管理员机器人始终会接收所有
群组消息，因此如果你需要查看全部消息，请将其设为管理员。

<div id="how-it-works-behavior">
  ## 工作原理（行为）
</div>

* 入站消息会被规范化为共享的通道封装结构，并带有回复上下文和媒体占位符。
* 群组回复默认需要带有提及（原生 @mention 或 `agents.list[].groupChat.mentionPatterns` / `messages.groupChat.mentionPatterns`）。
* 多智能体覆写：在 `agents.list[].groupChat.mentionPatterns` 上为每个智能体设置匹配模式。
* 回复始终会路由回同一个 Telegram 聊天。
* 长轮询使用 grammY runner，并按聊天维度保证顺序；整体并发量受 `agents.defaults.maxConcurrent` 限制。
* Telegram Bot API 不支持已读回执；不存在 `sendReadReceipts` 选项。

<div id="formatting-telegram-html">
  ## 格式（Telegram HTML）
</div>

* 发往 Telegram 的文本使用 `parse_mode: "HTML"`（Telegram 支持的标签子集）。
* 类似 Markdown 的输入会被渲染为**对 Telegram 安全的 HTML**（粗体/斜体/删除线/代码/链接）；块级元素会被扁平化为带有换行/项目符号的文本。
* 来自模型的原始 HTML 会被转义，以避免出现 Telegram 解析错误。
* 如果 Telegram 拒绝该 HTML 负载，OpenClaw 会改为将同一条消息以纯文本形式重试发送。

<div id="commands-native-custom">
  ## 命令（内置 + 自定义）
</div>

OpenClaw 在启动时会将内置命令（例如 `/status`、`/reset`、`/model`）注册到 Telegram 的机器人菜单中。
你可以通过配置将自定义命令添加到该菜单中：

```json5
{
  channels: {
    telegram: {
      customCommands: [
        { command: "backup", description: "Git backup" },
        { command: "generate", description: "Create an image" }
      ]
    }
  }
}
```

<div id="troubleshooting">
  ## 故障排查
</div>

* 日志中的 `setMyCommands failed` 通常意味着到 `api.telegram.org` 的出站 HTTPS/DNS 访问被阻断。
* 如果你看到 `sendMessage` 或 `sendChatAction` 失败，请检查 IPv6 路由和 DNS。

更多帮助：[频道故障排查](/zh/channels/troubleshooting)。

注意事项：

* 自定义命令仅作为**菜单条目**；除非你在其他地方自行处理，OpenClaw 不会对这些命令执行任何逻辑。
* 命令名称会被规范化（去掉前导 `/`，转为小写），并且必须匹配 `a-z`、`0-9`、`_`（1–32 个字符）。
* 自定义命令**不能覆盖原生命令**。冲突会被忽略并记录到日志。
* 如果禁用了 `commands.native`，则只会注册自定义命令（如果没有自定义命令，则会清空已注册命令）。

<div id="limits">
  ## 限制
</div>

* 发送的文本会按 `channels.telegram.textChunkLimit` 进行分块（默认 4000）。
* 可选换行分块：将 `channels.telegram.chunkMode="newline"` 设为该值，可以在长度分块前先按空行（段落边界）拆分。
* 媒体下载/上传大小受 `channels.telegram.mediaMaxMb` 限制（默认 5）。
* Telegram Bot API 请求在 `channels.telegram.timeoutSeconds` 后超时（通过 grammY 默认为 500）。可设置为更低值以避免长时间阻塞。
* 群组历史上下文使用 `channels.telegram.historyLimit`（或 `channels.telegram.accounts.*.historyLimit`），否则回退为 `messages.groupChat.historyLimit`。设为 `0` 可禁用（默认 50）。
* 私聊历史可通过 `channels.telegram.dmHistoryLimit` 限制（按用户对话轮次计数）。按用户覆盖配置：`channels.telegram.dms["<user_id>"].historyLimit`。

<div id="group-activation-modes">
  ## 群组激活模式
</div>

默认情况下，bot 只会在群组中被提及时才会响应（`@botname` 或 `agents.list[].groupChat.mentionPatterns` 中的匹配模式）。若要更改此行为：

<div id="via-config-recommended">
  ### 通过配置（推荐）
</div>

```json5
{
  channels: {
    telegram: {
      groups: {
        "-1001234567890": { requireMention: false }  // 始终在该群组中响应
      }
    }
  }
}
```

**重要：** 设置 `channels.telegram.groups` 会创建一个**允许列表**——只有列表中的群组（或 `"*"`）才会被接受。
论坛主题会继承其父级群组的配置（allowFrom、requireMention、技能、prompts），除非你在 `channels.telegram.groups.<groupId>.topics.<topicId>` 下为某个主题添加了单独的覆盖配置。

要允许所有群组并启用始终响应（always-respond）：

```json5
{
  channels: {
    telegram: {
      groups: {
        "*": { requireMention: false }  // 所有群组,始终响应
      }
    }
  }
}
```

要让所有群组保持“仅在被提及时激活”模式（默认行为）：

```json5
{
  channels: {
    telegram: {
      groups: {
        "*": { requireMention: true }  // 或完全省略 groups
      }
    }
  }
}
```

<div id="via-command-session-level">
  ### 通过命令（会话级别）
</div>

在群组中发送：

* `/activation always` - 响应所有消息
* `/activation mention` - 仅在被提及时才响应（默认）

**注意：**命令只会更新会话状态。若要在重启后仍然保持该行为，请使用配置。

<div id="getting-the-group-chat-id">
  ### 获取群聊 ID
</div>

在 Telegram 中把群组里的任意消息转发给 `@userinfobot` 或 `@getidsbot`，即可看到群聊 ID（类似 `-1001234567890` 这样的负数）。

**提示：** 如果你想获取自己的用户 ID，可以直接私聊该 bot，它会回复你的用户 ID（配对消息）；或者在命令启用后使用 `/whoami`。

**隐私说明：** `@userinfobot` 是第三方 bot。如果你更在意隐私，可以选择把 bot 加入群组，在群里发送一条消息，然后使用 `openclaw logs --follow` 查看 `chat.id`，或者使用 Bot API 的 `getUpdates`。

<div id="config-writes">
  ## 配置写入
</div>

默认情况下，Telegram 被允许根据频道事件或 `/config set|unset` 写入配置更新。

在以下情况下会发生：

* 当群组升级为超级群组并且 Telegram 触发 `migrate_to_chat_id`（聊天 ID 发生变化）时，OpenClaw 可以自动迁移 `channels.telegram.groups`。
* 你在 Telegram 聊天中运行 `/config set` 或 `/config unset`（需要 `commands.config: true`）。

若要禁用：

```json5
{
  channels: { telegram: { configWrites: false } }
}
```

<div id="topics-forum-supergroups">
  ## 话题（论坛超级群组）
</div>

Telegram 论坛话题中的每条消息都包含一个 `message_thread_id`。OpenClaw 会：

* 在 Telegram 群组会话键后追加 `:topic:<threadId>`，使每个话题彼此隔离。
* 在发送“正在输入”指示和回复时携带 `message_thread_id`，以便响应始终停留在对应话题中。
* 通用话题（线程 ID 为 `1`）是特殊情况：发送消息时会省略 `message_thread_id`（否则会被 Telegram 拒绝），但“正在输入”指示仍然会包含它。
* 在模板上下文中提供 `MessageThreadId` 和 `IsForum`，用于路由和模板渲染。
* 话题级配置位于 `channels.telegram.groups.<chatId>.topics.<threadId>` 下（技能、允许列表、自动回复、系统提示词、禁用）。
* 话题配置会继承群组设置（requireMention、允许列表、技能、prompts、enabled），除非在话题级单独覆盖。

在某些边缘场景中，私聊也可能包含 `message_thread_id`。OpenClaw 保持私信会话键不变，但在存在线程 ID 时，仍会使用该线程 ID 进行回复和草稿流式传输。

<div id="inline-buttons">
  ## 内联按钮
</div>

Telegram 支持使用带有回调按钮的内联键盘。

```json5
{
  "channels": {
    "telegram": {
      "capabilities": {
        "inlineButtons": "allowlist"
      }
    }
  }
}
```

按账户配置：

```json5
{
  "channels": {
    "telegram": {
      "accounts": {
        "main": {
          "capabilities": {
            "inlineButtons": "allowlist"
          }
        }
      }
    }
  }
}
```

作用域（scope）：

* `off` — 禁用 inline 按钮
* `dm` — 仅 DM（阻止群组目标）
* `group` — 仅群组（阻止 DM 目标）
* `all` — DM + 群组
* `allowlist` — DM + 群组，但仅允许 `allowFrom`/`groupAllowFrom` 所允许的发送方（规则与控制命令相同）

默认值：`allowlist`。
旧版配置：`capabilities: ["inlineButtons"]` = `inlineButtons: "all"`。

<div id="sending-buttons">
  ### 发送按钮消息
</div>

使用 message 工具，搭配 `buttons` 参数：

```json5
{
  "action": "send",
  "channel": "telegram",
  "to": "123456789",
  "message": "Choose an option:",
  "buttons": [
    [
      {"text": "Yes", "callback_data": "yes"},
      {"text": "No", "callback_data": "no"}
    ],
    [
      {"text": "Cancel", "callback_data": "cancel"}
    ]
  ]
}
```

当用户点击按钮时，回调数据会以如下格式作为消息发送回智能体：
`callback_data: value`

<div id="configuration-options">
  ### 配置选项
</div>

Telegram 的能力可以配置在两个级别（上面展示的是对象形式；仍然兼容旧版的字符串数组形式）：

* `channels.telegram.capabilities`：应用于所有 Telegram 账户的全局默认能力配置，除非被覆盖。
* `channels.telegram.accounts.<account>.capabilities`：针对特定账户的能力配置，会覆盖该账户的全局默认设置。

当所有 Telegram 机器人/账户应表现一致时，使用全局设置。当不同机器人需要不同行为时，使用按账户的配置（例如，一个账户只处理私聊，而另一个账户允许加入群组）。

<div id="access-control-dms-groups">
  ## 访问控制（私聊和群组）
</div>

<div id="dm-access">
  ### 私聊（DM）访问
</div>

* 默认：`channels.telegram.dmPolicy = "pairing"`。未知发送者会收到一个配对码；在批准之前其消息会被忽略（配对码在 1 小时后过期）。
* 通过以下方式批准：
  * `openclaw pairing list telegram`
  * `openclaw pairing approve telegram <CODE>`
* 配对是 Telegram 私聊所使用的默认令牌交换方式。详情参见：[配对](/zh/start/pairing)
* `channels.telegram.allowFrom` 接受数值型用户 ID（推荐）或 `@username` 条目。它**不是** bot 的用户名；请使用实际发送者（人类用户）的 ID。向导接受 `@username`，并在可能时将其解析为数值 ID。

<div id="finding-your-telegram-user-id">
  #### 查找你的 Telegram 用户 ID
</div>

更安全的方式（不使用第三方 bot）：

1. 启动 Gateway，并给你的 bot 发送私信（DM）。
2. 运行 `openclaw logs --follow`，查找 `from.id`。

替代方式（官方 Bot API）：

1. 给你的 bot 发送私信（DM）。
2. 使用你的 bot token 获取更新，并查看 `message.from.id`：
   ```bash
   curl "https://api.telegram.org/bot<bot_token>/getUpdates"
   ```

第三方方式（隐私性较差）：

* 私信 `@userinfobot` 或 `@getidsbot`，并使用返回的用户 ID。

<div id="group-access">
  ### 群组访问控制
</div>

有两个彼此独立的控制项：

**1. 允许哪些群组**（通过 `channels.telegram.groups` 配置群组允许列表）：

* 没有 `groups` 配置 = 允许所有群组
* 配置了 `groups` = 只允许配置中列出的群组或 `"*"`
* 示例：`"groups": { "-1001234567890": {}, "*": {} }` 表示允许所有群组

**2. 允许哪些发送方**（通过 `channels.telegram.groupPolicy` 进行发送方过滤）：

* `"open"` = 所有在已允许群组中的发送方都可以发消息（即允许来自任意用户的不受限消息）
* `"allowlist"` = 只有在 `channels.telegram.groupAllowFrom` 中的发送方可以发消息
* `"disabled"` = 完全不接受任何群组消息

默认配置为 `groupPolicy: "allowlist"`（在你添加 `groupAllowFrom` 之前都处于阻止状态）。

大多数用户推荐使用的配置为：`groupPolicy: "allowlist"` + `groupAllowFrom` + 在 `channels.telegram.groups` 中列出特定群组

<div id="long-polling-vs-webhook">
  ## 长轮询与 webhook
</div>

* 默认：长轮询（不需要公网 URL）。
* Webhook 模式：设置 `channels.telegram.webhookUrl`（可选再设置 `channels.telegram.webhookSecret` + `channels.telegram.webhookPath`）。
  * 本地监听器绑定到 `0.0.0.0:8787`，默认处理 `POST /telegram-webhook` 请求。
  * 如果你的公网 URL 不同，请使用反向代理，并将 `channels.telegram.webhookUrl` 指向该公网端点。

<div id="reply-threading">
  ## 回复线程
</div>

Telegram 通过标签支持可选的线程式回复：

* `[[reply_to_current]]` -- 回复触发该操作的消息。
* `[[reply_to:<id>]]` -- 回复指定消息 ID 的消息。

由 `channels.telegram.replyToMode` 控制：

* `first`（默认）、`all`、`off`。

<div id="audio-messages-voice-vs-file">
  ## 音频消息（语音 vs 文件）
</div>

Telegram 会区分**语音消息**（圆形气泡）和**音频文件**（带元数据卡片）。
出于向后兼容性的考虑，OpenClaw 默认使用音频文件。

要在智能体回复中强制使用语音消息气泡，可在回复中的任意位置加入以下标签：

* `[[audio_as_voice]]` — 将音频作为语音消息而不是文件发送。

该标签会从最终发送的文本中移除。其他渠道会忽略此标签。

对于通过消息工具发送的消息，在请求中设置 `asVoice: true`，并提供与语音消息兼容的音频 `media` URL
（当提供 media 时，`message` 是可选的）：

```json5
{
  "action": "send",
  "channel": "telegram",
  "to": "123456789",
  "media": "https://example.com/voice.ogg",
  "asVoice": true
}
```

<div id="stickers">
  ## 贴纸
</div>

OpenClaw 支持接收和发送 Telegram 贴纸，并提供智能缓存机制。

<div id="receiving-stickers">
  ### 接收贴纸
</div>

当用户发送贴纸时，OpenClaw 会根据贴纸类型进行处理：

* **静态贴纸（WEBP）：** 下载后通过视觉分析进行处理。贴纸将在消息内容中显示为 `<media:sticker>` 占位符。
* **动态贴纸（TGS）：** 跳过（不支持处理 Lottie 格式）。
* **视频贴纸（WEBM）：** 跳过（不支持处理该视频格式）。

接收贴纸时可用的模板上下文字段：

* `Sticker` — 对象，包含：
  * `emoji` — 与贴纸关联的表情符号
  * `setName` — 贴纸包名称
  * `fileId` — Telegram 文件 ID（可用于回发同一张贴纸）
  * `fileUniqueId` — 用于缓存查找的稳定 ID
  * `cachedDescription` — 在可用时的缓存视觉描述

<div id="sticker-cache">
  ### 贴纸缓存
</div>

贴纸会通过 AI 的视觉能力进行处理，以生成描述。由于同一张贴纸经常被反复发送，OpenClaw 会缓存这些描述，以避免重复的 API 调用。

**工作原理：**

1. **首次收到：** 将贴纸图片发送给 AI 做视觉分析。AI 会生成一个描述（例如：&quot;A cartoon cat waving enthusiastically&quot;）。
2. **写入缓存：** 该描述会和贴纸的文件 ID、emoji 以及贴纸包名称一起保存。
3. **后续收到：** 当再次收到同一张贴纸时，直接使用缓存中的描述，不会再次把图片发送给 AI。

**缓存位置：** `~/.openclaw/telegram/sticker-cache.json`

**缓存条目格式：**

```json
{
  "fileId": "CAACAgIAAxkBAAI...",
  "fileUniqueId": "AgADBAADb6cxG2Y",
  "emoji": "👋",
  "setName": "CoolCats",
  "description": "一只卡通猫热情挥手",
  "cachedAt": "2026-01-15T10:30:00.000Z"
}
```

**优点：**

* 通过避免对同一贴纸重复进行视觉识别调用来降低 API 成本
* 对已缓存贴纸的响应时间更快（没有视觉处理延迟）
* 基于缓存的描述提供贴纸搜索功能

缓存会在接收贴纸时自动构建，无需进行任何手动缓存管理。

<div id="sending-stickers">
  ### 发送贴纸
</div>

智能体可以使用 `sticker` 和 `sticker-search` 动作来发送和搜索贴纸。这些动作默认禁用，需要在配置中启用：

```json5
{
  channels: {
    telegram: {
      actions: {
        sticker: true
      }
    }
  }
}
```

**发送贴纸：**

```json5
{
  "action": "sticker",
  "channel": "telegram",
  "to": "123456789",
  "fileId": "CAACAgIAAxkBAAI..."
}
```

参数：

* `fileId` (必需) — Telegram 中该贴纸的文件 ID。接收贴纸时可从 `Sticker.fileId` 获取，或从 `sticker-search` 的结果中获取。
* `replyTo` (可选) — 要回复的消息 ID。
* `threadId` (可选) — 论坛话题的消息线程 ID。

**搜索贴纸：**

智能体可以通过描述、表情或贴纸包名称搜索已缓存的贴纸：

```json5
{
  "action": "sticker-search",
  "channel": "telegram",
  "query": "cat waving",
  "limit": 5
}
```

返回缓存中匹配的贴纸：

```json5
{
  "ok": true,
  "count": 2,
  "stickers": [
    {
      "fileId": "CAACAgIAAxkBAAI...",
      "emoji": "👋",
      "description": "一只卡通猫热情挥手",
      "setName": "CoolCats"
    }
  ]
}
```

搜索会对描述文本、emoji 字符和集合名称进行模糊匹配。

**线程示例：**

```json5
{
  "action": "sticker",
  "channel": "telegram",
  "to": "-1001234567890",
  "fileId": "CAACAgIAAxkBAAI...",
  "replyTo": 42,
  "threadId": 123
}
```

<div id="streaming-drafts">
  ## 流式传输（草稿）
</div>

在智能体生成回复时，Telegram 可以以流式方式更新**草稿气泡**。
OpenClaw 使用 Bot API `sendMessageDraft`（而非真实消息），然后再将
最终回复作为普通消息发送。

要求（Telegram Bot API 9.3+）：

* **开启了主题的私聊**（为该 bot 启用论坛主题模式）。
* 传入消息必须包含 `message_thread_id`（私聊主题线程）。
* 在群组 / 超级群组 / 频道中会忽略流式传输。

配置：

* `channels.telegram.streamMode: "off" | "partial" | "block"`（默认：`partial`）
  * `partial`：使用最新的流式文本更新草稿气泡。
  * `block`：以较大的区块（分块）更新草稿气泡。
  * `off`：禁用草稿流式传输。
* 可选（仅用于 `streamMode: "block"`）：
  * `channels.telegram.draftChunk: { minChars?, maxChars?, breakPreference? }`
    * 默认值：`minChars: 200`、`maxChars: 800`、`breakPreference: "paragraph"`（会被限制在 `channels.telegram.textChunkLimit` 范围内）。

注意：草稿流式传输与**区块流式传输**（频道消息）是分开的。
区块流式传输默认关闭，如果你希望优先在 Telegram 中发送早期消息，而不只是更新草稿，
需要将 `channels.telegram.blockStreaming` 设置为 `true`。

推理流（仅限 Telegram）：

* `/reasoning stream` 会在生成回复时，将推理内容以流式方式写入草稿气泡，
  然后发送不带推理内容的最终答案。
* 如果 `channels.telegram.streamMode` 为 `off`，则会禁用推理流。
  更多上下文： [Streaming + chunking](/zh/concepts/streaming)。

<div id="retry-policy">
  ## 重试策略
</div>

对发出的 Telegram api 调用，在遇到瞬时网络错误或 429 错误时，会采用带抖动的指数退避方式进行重试。可通过 `channels.telegram.retry` 进行配置。参见[重试策略](/zh/concepts/retry)。

<div id="agent-tool-messages-reactions">
  ## Agent 工具（消息 + reaction）
</div>

* 工具：`telegram` 的 `sendMessage` 动作（`to`、`content`，可选 `mediaUrl`、`replyToMessageId`、`messageThreadId`）。
* 工具：`telegram` 的 `react` 动作（`chatId`、`messageId`、`emoji`）。
* 工具：`telegram` 的 `deleteMessage` 动作（`chatId`、`messageId`）。
* reaction 移除语义：参见 [/tools/reactions](/zh/tools/reactions)。
* 工具门控：`channels.telegram.actions.reactions`、`channels.telegram.actions.sendMessage`、`channels.telegram.actions.deleteMessage`（默认：启用），以及 `channels.telegram.actions.sticker`（默认：禁用）。

<div id="reaction-notifications">
  ## Reaction 通知
</div>

**Reaction 的工作方式：**
Telegram 的 reaction 会作为**单独的 `message_reaction` 事件**到达，而不是作为消息负载中的属性。当用户添加 reaction 时，OpenClaw 会：

1. 从 Telegram API 接收 `message_reaction` 更新
2. 将其转换为格式为 `"Telegram reaction added: {emoji} by {user} on msg {id}"` 的**系统事件**
3. 使用与普通消息**相同的 session key（会话键）**将系统事件入队
4. 当该会话中下一条消息到达时，将系统事件队列清空，并插入到智能体上下文的最前面

智能体在会话历史中将 reaction 视为**系统通知**，而不是消息元数据。

**配置：**

* `channels.telegram.reactionNotifications`：控制哪些 reaction 会触发通知
  * `"off"` — 忽略所有 reaction
  * `"own"` — 当用户对 bot 消息添加 reaction 时通知（尽力而为；仅存于内存）（默认）
  * `"all"` — 对所有 reaction 发送通知

* `channels.telegram.reactionLevel`：控制智能体对消息添加 reaction 的能力
  * `"off"` — 智能体不能对消息添加 reaction
  * `"ack"` — bot 发送确认类 reaction（处理时用 👀）（默认）
  * `"minimal"` — 智能体可以少量使用 reaction（指导原则：每 5–10 次交互使用不超过 1 次）
  * `"extensive"` — 智能体在合适场景下可以更频繁地使用 reaction

**论坛群组：** 论坛群组中的 reaction 包含 `message_thread_id`，并使用类似 `agent:main:telegram:group:{chatId}:topic:{threadId}` 的 session key。这样可以确保同一主题中的 reaction 与消息被归为同一会话。

**配置示例：**

```json5
{
  channels: {
    telegram: {
      reactionNotifications: "all",  // See all reactions
      reactionLevel: "minimal"        // 智能体可以适度做出反应
    }
  }
}
```

**要求：**

* Telegram 机器人必须在 `allowed_updates` 中显式请求 `message_reaction`（由 OpenClaw 自动配置）
* 对于 webhook 模式，表情反应会包含在 webhook 的 `allowed_updates` 中
* 对于轮询模式，表情反应会包含在 `getUpdates` 的 `allowed_updates` 中

<div id="delivery-targets-clicron">
  ## 投递目标（CLI/cron）
</div>

* 使用聊天 ID（`123456789`）或用户名（`@name`）作为目标。
* 示例：`openclaw message send --channel telegram --target 123456789 --message "hi"`。

<div id="troubleshooting">
  ## 故障排查
</div>

**机器人在群组中对未提及的消息没有响应：**

* 如果你设置了 `channels.telegram.groups.*.requireMention=false`，则必须在 Telegram 的 Bot API 中关闭**隐私模式（privacy mode）**。
  * 在 BotFather 中：`/setprivacy` → 选择 **Disable**（然后将机器人从群组中移除并重新添加）
* 当配置期望接收未被提及的群消息时，`openclaw channels status` 会显示警告。
* `openclaw channels status --probe` 还可以对显式的数字群组 ID 检查成员关系（它无法审计通配 `"*"` 规则）。
* 快速测试：`/activation always`（仅对当前会话生效；持久化请使用配置）

**机器人完全看不到群消息：**

* 如果设置了 `channels.telegram.groups`，则必须显式列出该群组或者使用 `"*"`
* 在 @BotFather 的隐私设置中检查 → &quot;Group Privacy&quot; 应为 **OFF**
* 确认机器人实际是群成员（而不是仅有管理员身份但没有 read 访问权限）
* 检查 Gateway 日志：`openclaw logs --follow`（查找 &quot;skipping group message&quot;）

**机器人能响应提及，但对 `/activation always` 没反应：**

* `/activation` 命令会更新会话状态，但不会持久化到配置文件
* 如需持久化该行为，在 `channels.telegram.groups` 中为该群添加配置，并设置 `requireMention: false`

**像 `/status` 这样的命令不起作用：**

* 确保你的 Telegram 用户 ID 已被授权（通过配对或 `channels.telegram.allowFrom`）
* 即使在 `groupPolicy: "open"` 的群组中，命令也需要授权

**在 Node 22+ 上长轮询立即中止（常见于使用代理/自定义 fetch 时）：**

* Node 22+ 对 `AbortSignal` 实例更严格；外部的 `AbortSignal` 实例可能会立即中止 `fetch` 调用。
* 升级到会标准化中止信号的 OpenClaw 构建，或者在你能升级之前先在 Node 20 上运行 Gateway。

**机器人启动后，随后静默停止响应（或日志中出现 `HttpError: Network request ... failed`）：**

* 某些主机解析 `api.telegram.org` 时会优先得到 IPv6。如果你的服务器没有可用的 IPv6 出站连接，grammY 可能会被卡在仅 IPv6 的请求上。
* 解决方法：启用 IPv6 出站连接，**或者**强制将 `api.telegram.org` 解析为 IPv4（例如在 `/etc/hosts` 中添加使用 IPv4 A 记录的条目，或在操作系统 DNS 栈中优先 IPv4），然后重启 Gateway。
* 快速检查：运行 `dig +short api.telegram.org A` 和 `dig +short api.telegram.org AAAA` 以确认 DNS 返回的记录。

<div id="configuration-reference-telegram">
  ## 配置参考（Telegram）
</div>

完整配置：[Configuration](/zh/gateway/configuration)

提供方选项：

* `channels.telegram.enabled`：启用/禁用频道启动。
* `channels.telegram.botToken`：bot token（由 BotFather 提供）。
* `channels.telegram.tokenFile`：从文件路径读取 token。
* `channels.telegram.dmPolicy`：`pairing | allowlist | open | disabled`（默认：pairing）。
* `channels.telegram.allowFrom`：私信 DM 允许列表（id/用户名）。`open` 模式需要 `"*"`。
* `channels.telegram.groupPolicy`：`open | allowlist | disabled`（默认：allowlist）。
* `channels.telegram.groupAllowFrom`：群消息发件人允许列表（id/用户名）。
* `channels.telegram.groups`：按群配置默认值和允许列表（使用 `"*"` 作为全局默认）。
  * `channels.telegram.groups.<id>.requireMention`：@ 提及门控默认值。
  * `channels.telegram.groups.<id>.skills`：技能过滤器（省略 = 所有技能，空 = 无）。
  * `channels.telegram.groups.<id>.allowFrom`：每个群的发件人允许列表覆盖。
  * `channels.telegram.groups.<id>.systemPrompt`：该群的额外 system prompt。
  * `channels.telegram.groups.<id>.enabled`：为 `false` 时禁用该群。
  * `channels.telegram.groups.<id>.topics.<threadId>.*`：按话题覆盖（字段与群相同）。
  * `channels.telegram.groups.<id>.topics.<threadId>.requireMention`：按话题的 @ 提及门控覆盖。
* `channels.telegram.capabilities.inlineButtons`：`off | dm | group | all | allowlist`（默认：allowlist）。
* `channels.telegram.accounts.<account>.capabilities.inlineButtons`：按账号覆盖。
* `channels.telegram.replyToMode`：`off | first | all`（默认：`first`）。
* `channels.telegram.textChunkLimit`：出站分块大小（字符数）。
* `channels.telegram.chunkMode`：`length`（默认）或 `newline`，在按长度分块前先按空行（段落边界）拆分。
* `channels.telegram.linkPreview`：切换是否为出站消息启用链接预览（默认：true）。
* `channels.telegram.streamMode`：`off | partial | block`（草稿流式模式）。
* `channels.telegram.mediaMaxMb`：入站/出站媒体大小上限（MB）。
* `channels.telegram.retry`：出站 Telegram API 调用的重试策略（attempts、minDelayMs、maxDelayMs、jitter）。
* `channels.telegram.network.autoSelectFamily`：覆盖 Node 的 autoSelectFamily（true = 启用，false = 禁用）。在 Node 22 上默认禁用，以避免 Happy Eyeballs 超时。
* `channels.telegram.proxy`：用于 Bot API 调用的代理 URL（SOCKS/HTTP）。
* `channels.telegram.webhookUrl`：启用 webhook 模式。
* `channels.telegram.webhookSecret`：webhook 密钥（可选）。
* `channels.telegram.webhookPath`：本地 webhook 路径（默认 `/telegram-webhook`）。
* `channels.telegram.actions.reactions`：控制 Telegram 工具 reaction 操作的权限。
* `channels.telegram.actions.sendMessage`：控制 Telegram 工具发送消息操作的权限。
* `channels.telegram.actions.deleteMessage`：控制 Telegram 工具删除消息操作的权限。
* `channels.telegram.actions.sticker`：控制 Telegram 贴纸操作——发送和搜索（默认：false）。
* `channels.telegram.reactionNotifications`：`off | own | all` —— 控制哪些 reaction 会触发系统事件（默认：未设置时为 `own`）。
* `channels.telegram.reactionLevel`：`off | ack | minimal | extensive` —— 控制智能体的 reaction 能力级别（默认：未设置时为 `minimal`）。

相关全局选项：

* `agents.list[].groupChat.mentionPatterns`（@ 提及门控匹配模式）。
* `messages.groupChat.mentionPatterns`（全局回退）。
* `commands.native`（默认 `"auto"` → 在 Telegram/Discord 上开启，在 Slack 上关闭）、`commands.text`、`commands.useAccessGroups`（命令行为）。可通过 `channels.telegram.commands.native` 覆盖。
* `messages.responsePrefix`、`messages.ackReaction`、`messages.ackReactionScope`、`messages.removeAckAfterReply`。