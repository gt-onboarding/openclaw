---
title: Slack
summary: "Slack 的 socket 或 HTTP webhook 模式配置"
read_when: "在配置或调试 Slack 的 socket/HTTP 模式时阅读"
---

<div id="slack">
  # Slack
</div>

<div id="socket-mode-default">
  ## Socket 模式（默认）
</div>

<div id="quick-setup-beginner">
  ### 快速设置（入门）
</div>

1. 创建一个 Slack 应用并启用 **Socket Mode**。
2. 创建一个 **App Token**（`xapp-...`）和 **Bot Token**（`xoxb-...`）。
3. 在 OpenClaw 中配置这些 token，并启动 Gateway。

最小配置：

```json5
{
  channels: {
    slack: {
      enabled: true,
      appToken: "xapp-...",
      botToken: "xoxb-..."
    }
  }
}
```

<div id="setup">
  ### 设置
</div>

1. 在 https://api.slack.com/apps 中创建一个 Slack 应用（选择 **From scratch**）。
2. 打开 **Socket Mode** → 切换为开启。然后进入 **Basic Information** → **App-Level Tokens** → 使用 `connections:write` scope **Generate Token and Scopes**。复制 **App Token**（`xapp-...`）。
3. 在 **OAuth &amp; Permissions** 中添加 bot token scopes（使用下面的 manifest）。点击 **Install to Workspace**。复制 **Bot User OAuth Token**（`xoxb-...`）。
4. 可选：在 **OAuth &amp; Permissions** 中添加 **User Token Scopes**（见下面的只读列表）。重新安装应用并复制 **User OAuth Token**（`xoxp-...`）。
5. 在 **Event Subscriptions** 中启用事件订阅，并订阅：
   * `message.*`（包括编辑/删除/线程广播）
   * `app_mention`
   * `reaction_added`, `reaction_removed`
   * `member_joined_channel`, `member_left_channel`
   * `channel_rename`
   * `pin_added`, `pin_removed`
6. 将 bot 邀请到你希望它能够读取的频道中。
7. Slash Commands → 如果你使用 `channels.slack.slashCommand`，则创建 `/openclaw`。如果你启用原生命令，为每个内置命令添加一个 slash command（名称与 `/help` 中的命令相同）。对于 Slack，原生命令默认关闭，除非你设置 `channels.slack.commands.native: true`（全局 `commands.native` 为 `"auto"`，此时 Slack 仍保持关闭）。
8. App Home → 启用 **Messages Tab**，以便用户可以通过 DM 向 bot 发送消息。

使用下面的 manifest，以便 scopes 和 events 保持一致。

多账户支持：使用 `channels.slack.accounts`，为每个账户配置各自的 tokens，并可选设置 `name`。通用模式参考 [`gateway/configuration`](/zh/gateway/configuration#telegramaccounts--discordaccounts--slackaccounts--signalaccounts--imessageaccounts)。

<div id="openclaw-config-minimal">
  ### OpenClaw 配置（最简）
</div>

通过环境变量设置令牌（推荐）：

* `SLACK_APP_TOKEN=xapp-...`
* `SLACK_BOT_TOKEN=xoxb-...`

或者通过配置：

```json5
{
  channels: {
    slack: {
      enabled: true,
      appToken: "xapp-...",
      botToken: "xoxb-..."
    }
  }
}
```

<div id="user-token-optional">
  ### User token（可选）
</div>

OpenClaw 可以使用 Slack 用户 token（`xoxp-...`）执行 read 操作（历史记录、
置顶、reaction、emoji、成员信息）。默认情况下它保持只读：在存在用户 token
时，read 操作优先使用它，而写操作仍然使用 bot token，除非你显式关闭只读限制。
即使设置了 `userTokenReadOnly: false`，在可用时写操作仍然优先使用 bot token。

User token 通过配置文件进行配置（不支持环境变量）。对于多账号场景，请设置
`channels.slack.accounts.<id>.userToken`。

同时使用 bot + app + user token 的示例：

```json5
{
  channels: {
    slack: {
      enabled: true,
      appToken: "xapp-...",
      botToken: "xoxb-...",
      userToken: "xoxp-..."
    }
  }
}
```

显式将 userTokenReadOnly 设置的示例（允许用户 token 写入）：

```json5
{
  channels: {
    slack: {
      enabled: true,
      appToken: "xapp-...",
      botToken: "xoxb-...",
      userToken: "xoxp-...",
      userTokenReadOnly: false
    }
  }
}
```

<div id="token-usage">
  #### 令牌使用
</div>

* 读取操作（历史记录、表情反应列表、置顶列表、表情符号列表、成员信息、搜索）在配置了用户令牌时优先使用用户令牌，否则使用机器人令牌。
* 写入操作（发送/编辑/删除消息，添加/移除表情反应，置顶/取消置顶，文件上传）默认使用机器人令牌。如果 `userTokenReadOnly: false` 且没有可用的机器人令牌，OpenClaw 将回退到使用用户令牌。

<div id="history-context">
  ### 历史上下文
</div>

* `channels.slack.historyLimit`（或 `channels.slack.accounts.*.historyLimit`）控制会将多少条最近的频道/群组消息包含到提示中。
* 若未设置，则回退为 `messages.groupChat.historyLimit`。设为 `0` 可禁用（默认 50）。

<div id="http-mode-events-api">
  ## HTTP 模式（Events API）
</div>

当你的 Gateway 能通过 HTTPS 被 Slack 访问时（典型的服务器部署场景），使用 HTTP webhook 模式。
HTTP 模式使用一个共享的请求 URL，同步承载 Events API、交互（Interactivity）以及斜杠命令（Slash Commands）。

<div id="setup">
  ### 设置
</div>

1. 创建一个 Slack 应用，并**禁用 Socket Mode**（如果你只使用 HTTP，则可选）。
2. 进入 **Basic Information** → 复制 **Signing Secret**。
3. 进入 **OAuth &amp; Permissions** → 安装应用并复制 **Bot User OAuth Token**（`xoxb-...`）。
4. 进入 **Event Subscriptions** → 启用事件，并将 **Request URL** 设置为你的 Gateway webhook 路径（默认 `/slack/events`）。
5. 进入 **Interactivity &amp; Shortcuts** → 启用并设置相同的 **Request URL**。
6. 进入 **Slash Commands** → 为你的命令设置相同的 **Request URL**。

示例请求 URL：
`https://gateway-host/slack/events`

### OpenClaw 最小配置

```json5
{
  channels: {
    slack: {
      enabled: true,
      mode: "http",
      botToken: "xoxb-...",
      signingSecret: "your-signing-secret",
      webhookPath: "/slack/events"
    }
  }
}
```

多账号 HTTP 模式：设置 `channels.slack.accounts.<id>.mode = "http"`，并为每个账号提供唯一的 `webhookPath`，这样每个 Slack 应用就可以指向各自的 URL。

<div id="manifest-optional">
  ### Manifest（可选）
</div>

使用这个 Slack 应用 manifest 可以快速创建应用（如有需要，可以调整名称/命令）。如果你打算配置 user token，请在其中加入对应的 user scopes。

```json
{
  "display_information": {
    "name": "OpenClaw",
    "description": "Slack connector for OpenClaw"
  },
  "features": {
    "bot_user": {
      "display_name": "OpenClaw",
      "always_online": false
    },
    "app_home": {
      "messages_tab_enabled": true,
      "messages_tab_read_only_enabled": false
    },
    "slash_commands": [
      {
        "command": "/openclaw",
        "description": "Send a message to OpenClaw",
        "should_escape": false
      }
    ]
  },
  "oauth_config": {
    "scopes": {
      "bot": [
        "chat:write",
        "channels:history",
        "channels:read",
        "groups:history",
        "groups:read",
        "groups:write",
        "im:history",
        "im:read",
        "im:write",
        "mpim:history",
        "mpim:read",
        "mpim:write",
        "users:read",
        "app_mentions:read",
        "reactions:read",
        "reactions:write",
        "pins:read",
        "pins:write",
        "emoji:read",
        "commands",
        "files:read",
        "files:write"
      ],
      "user": [
        "channels:history",
        "channels:read",
        "groups:history",
        "groups:read",
        "im:history",
        "im:read",
        "mpim:history",
        "mpim:read",
        "users:read",
        "reactions:read",
        "pins:read",
        "emoji:read",
        "search:read"
      ]
    }
  },
  "settings": {
    "socket_mode_enabled": true,
    "event_subscriptions": {
      "bot_events": [
        "app_mention",
        "message.channels",
        "message.groups",
        "message.im",
        "message.mpim",
        "reaction_added",
        "reaction_removed",
        "member_joined_channel",
        "member_left_channel",
        "channel_rename",
        "pin_added",
        "pin_removed"
      ]
    }
  }
}
```

启用原生命令后，为每个你想公开的命令添加一个 `slash_commands` 条目（与 `/help` 列表对应）。可通过 `channels.slack.commands.native` 覆盖该设置。

<div id="scopes-current-vs-optional">
  ## Scopes（当前 vs 可选）
</div>

Slack 的 Conversations API 使用按类型划分的 scope：你只需要为实际会涉及到的会话类型（channels、groups、im、mpim）申请对应的 scopes。概览参见：https://docs.slack.dev/apis/web-api/using-the-conversations-api/。

<div id="bot-token-scopes-required">
  ### Bot token 的 scope（必需）
</div>

* `chat:write`（通过 `chat.postMessage` 发送/更新/删除消息）
  https://docs.slack.dev/reference/methods/chat.postMessage
* `im:write`（通过 `conversations.open` 为用户打开 DM 会话）
  https://docs.slack.dev/reference/methods/conversations.open
* `channels:history`、`groups:history`、`im:history`、`mpim:history`
  https://docs.slack.dev/reference/methods/conversations.history
* `channels:read`、`groups:read`、`im:read`、`mpim:read`
  https://docs.slack.dev/reference/methods/conversations.info
* `users:read`（查询用户信息）
  https://docs.slack.dev/reference/methods/users.info
* `reactions:read`、`reactions:write`（`reactions.get` / `reactions.add`）
  https://docs.slack.dev/reference/methods/reactions.get
  https://docs.slack.dev/reference/methods/reactions.add
* `pins:read`、`pins:write`（`pins.list` / `pins.add` / `pins.remove`）
  https://docs.slack.dev/reference/scopes/pins.read
  https://docs.slack.dev/reference/scopes/pins.write
* `emoji:read`（`emoji.list`）
  https://docs.slack.dev/reference/scopes/emoji.read
* `files:write`（通过 `files.uploadV2` 上传文件）
  https://docs.slack.dev/messaging/working-with-files/#upload

<div id="user-token-scopes-optional-read-only-by-default">
  ### 用户 token 的 scope（可选，默认仅为只读）
</div>

如果你配置了 `channels.slack.userToken`，请在 **User Token Scopes** 中添加以下 scope：

* `channels:history`, `groups:history`, `im:history`, `mpim:history`
* `channels:read`, `groups:read`, `im:read`, `mpim:read`
* `users:read`
* `reactions:read`
* `pins:read`
* `emoji:read`
* `search:read`

<div id="not-needed-today-but-likely-future">
  ### 今天还不需要（但很可能将来会用到）
</div>

* `mpim:write`（只有在我们通过 `conversations.open` 支持发起群组私信/DM 时才需要）
* `groups:write`（只有在我们添加私有频道管理功能：创建/重命名/邀请/归档时才需要）
* `chat:write.public`（只有在我们想向机器人未加入的频道发消息时才需要）
  https://docs.slack.dev/reference/scopes/chat.write.public
* `users:read.email`（只有在我们需要从 `users.info` 中获取 email 字段时才需要）
  https://docs.slack.dev/changelog/2017-04-narrowing-email-access
* `files:read`（只有在我们开始列出/读取文件元数据时才需要）

<div id="config">
  ## 配置
</div>

Slack 仅使用 Socket 模式（不使用 HTTP webhook 服务器）。请提供这两个令牌：

```json
{
  "slack": {
    "enabled": true,
    "botToken": "xoxb-...",
    "appToken": "xapp-...",
    "groupPolicy": "allowlist",
    "dm": {
      "enabled": true,
      "policy": "pairing",
      "allowFrom": ["U123", "U456", "*"],
      "groupEnabled": false,
      "groupChannels": ["G123"],
      "replyToMode": "all"
    },
    "channels": {
      "C123": { "allow": true, "requireMention": true },
      "#general": {
        "allow": true,
        "requireMention": true,
        "users": ["U123"],
        "skills": ["search", "docs"],
        "systemPrompt": "Keep answers short."
      }
    },
    "reactionNotifications": "own",
    "reactionAllowlist": ["U123"],
    "replyToMode": "off",
    "actions": {
      "reactions": true,
      "messages": true,
      "pins": true,
      "memberInfo": true,
      "emojiList": true
    },
    "slashCommand": {
      "enabled": true,
      "name": "openclaw",
      "sessionPrefix": "slack:slash",
      "ephemeral": true
    },
    "textChunkLimit": 4000,
    "mediaMaxMb": 20
  }
}
```

也可以通过环境变量传入 token：

* `SLACK_BOT_TOKEN`
* `SLACK_APP_TOKEN`

确认表情反应在全局通过 `messages.ackReaction` +
`messages.ackReactionScope` 控制。使用 `messages.removeAckAfterReply` 在机器人回复后清除该确认反应。

<div id="limits">
  ## 限制
</div>

* 发出的文本会根据 `channels.slack.textChunkLimit` 进行分块（默认 4000）。
* 可选的按换行分块模式：将 `channels.slack.chunkMode` 设置为 `"newline"`，会先按空行（段落边界）拆分，然后再按长度分块。
* 媒体上传大小受 `channels.slack.mediaMaxMb` 限制（默认 20）。

<div id="reply-threading">
  ## 回复线程
</div>

默认情况下，OpenClaw 会在主频道中回复。使用 `channels.slack.replyToMode` 来控制自动串线方式：

| 模式 | 行为 |
| --- | --- |
| `off` | **默认。** 在主频道中回复。只有当触发消息已经在某个线程中时，才在该线程中回复。 |
| `first` | 第一次回复发送到线程（位于触发消息下方），后续回复发送到主频道。适合在保持上下文可见的同时，避免线程过于杂乱。 |
| `all` | 所有回复都发送到线程。可以将会话集中在一起，但可能降低可见性。 |

该模式同时适用于自动回复和智能体工具调用（`slack sendMessage`）。

<div id="per-chat-type-threading">
  ### 按聊天类型配置线程
</div>

你可以通过设置 `channels.slack.replyToModeByChatType`，为不同的聊天类型配置线程回复行为：

```json5
{
  channels: {
    slack: {
      replyToMode: "off",        // default for channels
      replyToModeByChatType: {
        direct: "all",           // DMs always thread
        group: "first"           // 群组私信/MPIM 首次回复使用会话串
      },
    }
  }
}
```

支持的聊天类型：

* `direct`: 一对一私信（Slack `im`）
* `group`: 群组私信 / MPIM（Slack `mpim`）
* `channel`: 标准频道（公开/私有）

优先级顺序：

1. `replyToModeByChatType.<chatType>`
2. `replyToMode`
3. 提供方默认值（`off`）

旧配置项 `channels.slack.dm.replyToMode` 仍会被接受，在未设置按聊天类型的覆盖配置时，作为 `direct` 的回退配置。

示例：

仅在线程中的私信：

```json5
{
  channels: {
    slack: {
      replyToMode: "off",
      replyToModeByChatType: { direct: "all" }
    }
  }
}
```

群组私信使用线程回复，但频道消息保持在根级：

```json5
{
  channels: {
    slack: {
      replyToMode: "off",
      replyToModeByChatType: { group: "first" }
    }
  }
}
```

让频道里的消息使用线程，把私信回复保持在顶层对话中：

```json5
{
  channels: {
    slack: {
      replyToMode: "first",
      replyToModeByChatType: { direct: "off", group: "off" }
    }
  }
}
```

<div id="manual-threading-tags">
  ### 手动线程标签
</div>

若要进行更细粒度的控制，可在智能体回复中使用以下标签：

* `[[reply_to_current]]` — 回复触发的这条消息（开始或继续线程）。
* `[[reply_to:<id>]]` — 回复指定的消息 ID。

<div id="sessions-routing">
  ## 会话与路由
</div>

* 私信（DM）共享 `main` 会话（类似 WhatsApp/Telegram）。
* 频道映射到 `agent:<agentId>:slack:channel:<channelId>` 会话。
* 斜杠命令使用 `agent:<agentId>:slack:slash:<userId>` 会话（前缀可通过 `channels.slack.slashCommand.sessionPrefix` 配置）。
* 如果 Slack 不提供 `channel_type`，OpenClaw 会根据频道 ID 前缀（`D`、`C`、`G`）进行推断，并默认视为 `channel`，以保持会话 key 稳定。
* 原生命令注册使用 `commands.native`（全局默认值为 `"auto"` → 在 Slack 上为关闭状态），并可在每个工作区通过 `channels.slack.commands.native` 覆盖。文本命令要求以独立的 `/...` 消息发送，并可通过 `commands.text: false` 禁用。Slack 斜杠命令在 Slack 应用中管理，不会被自动移除。使用 `commands.useAccessGroups: false` 可跳过对命令的访问组检查。
* 完整命令列表与配置：[斜杠命令](/zh/tools/slash-commands)

<div id="dm-security-pairing">
  ## DM 安全（配对）
</div>

* 默认：`channels.slack.dm.policy="pairing"` — 来自未知 DM 发送者的消息会收到一个配对码（1 小时后过期）。
* 通过以下命令批准：`openclaw pairing approve slack <code>`。
* 若要允许任何人发送：设置 `channels.slack.dm.policy="open"`（允许从任何用户无条件接收消息），并将 `channels.slack.dm.allowFrom=["*"]`。
* `channels.slack.dm.allowFrom` 接受用户 ID、@handle 或电子邮件（在令牌权限允许的情况下，会在启动时进行解析）。向导在设置期间接受用户名，并在令牌权限允许时将其解析为 ID。

<div id="group-policy">
  ## 群组策略
</div>

* `channels.slack.groupPolicy` 控制频道处理策略（`open|disabled|allowlist`）。
* `allowlist` 要求频道必须列在 `channels.slack.channels` 中。
* 如果你只设置了 `SLACK_BOT_TOKEN`/`SLACK_APP_TOKEN` 而从未创建 `channels.slack` 配置段，
  运行时时会将 `groupPolicy` 默认为 `open`（表示允许从任意用户不受限地接收消息）。添加 `channels.slack.groupPolicy`、
  `channels.defaults.groupPolicy`，或者为频道配置允许列表以收紧访问控制。
* 配置向导接受 `#channel` 名称，并在可能时将其解析为 ID
  （公开 + 私有）；如果存在多个匹配项，则优先使用当前活动频道。
* 启动时，OpenClaw 会将允许列表中的频道/用户名解析为 ID（在 token 权限允许的情况下），
  并记录映射；未解析的条目会保持原样。
* 若要**不允许任何频道**，将 `channels.slack.groupPolicy: "disabled"`（或保持允许列表为空）。

频道选项（`channels.slack.channels.<id>` 或 `channels.slack.channels.<name>`）：

* `allow`：在 `groupPolicy="allowlist"` 时允许/拒绝该频道。
* `requireMention`：该频道的 @ 提及门控设置。
* `tools`：可选的按频道工具策略覆盖（`allow`/`deny`/`alsoAllow`）。
* `toolsBySender`：可选的频道内按发送方的工具策略覆盖（键为发送方 id/@ 用户名/邮箱；支持 `"*"` 通配符）。
* `allowBots`：允许此频道中的机器人消息（默认：false）。
* `users`：可选的按频道用户允许列表。
* `skills`：技能过滤器（省略 = 所有技能，空列表 = 无）。
* `systemPrompt`：该频道的额外 system prompt（与主题/用途组合使用）。
* `enabled`：设置为 `false` 以禁用该频道。

<div id="delivery-targets">
  ## 发送目标
</div>

在使用 cron/CLI 发送时使用：

* `user:<id>` 表示私信（DM）
* `channel:<id>` 表示频道

<div id="tool-actions">
  ## 工具操作
</div>

Slack 工具操作可以通过 `channels.slack.actions.*` 进行权限控制：

| 操作组 | 默认值 | 说明 |
| --- | --- | --- |
| reactions | 启用 | 添加/列出表情反应 |
| messages | 启用 | 读取/发送/编辑/删除 |
| pins | 启用 | 固定/取消固定/列出 |
| memberInfo | 启用 | 成员信息 |
| emojiList | 启用 | 自定义表情列表 |

<div id="security-notes">
  ## 安全注意事项
</div>

* 写操作默认使用 bot token，从而使所有会改变状态的操作都被限制在应用的 bot 权限和身份范围内。
* 将 `userTokenReadOnly` 设为 `false` 时，在没有 bot token 的情况下，user token 可以用于写操作，这意味着操作将以安装该应用的用户权限执行。请将 user token 视为高权限凭证，对操作访问控制和允许列表保持严格控制。
* 如果你启用了基于 user token 的写操作，请确保该 user token 包含你预期的写入 scope（`chat:write`、`reactions:write`、`pins:write`、`files:write`），否则这些操作将会失败。

<div id="notes">
  ## 备注
</div>

* 提及触发由 `channels.slack.channels` 控制（将 `requireMention` 设为 `true`）；`agents.list[].groupChat.mentionPatterns`（或 `messages.groupChat.mentionPatterns`）也会被视为提及。
* 多智能体级覆盖：在 `agents.list[].groupChat.mentionPatterns` 中为每个智能体设置单独的匹配模式。
* 表情回应通知遵循 `channels.slack.reactionNotifications`（使用 `reactionAllowlist` 且模式为 `allowlist`）。
* 由机器人生成的消息默认会被忽略；可通过 `channels.slack.allowBots` 或 `channels.slack.channels.<id>.allowBots` 启用。
* 警告：如果你允许回复其他机器人（`channels.slack.allowBots=true` 或 `channels.slack.channels.<id>.allowBots=true`），请使用 `requireMention`、`channels.slack.channels.<id>.users` 允许列表，以及在 `AGENTS.md` 和 `SOUL.md` 中设定清晰的防护规则，来防止机器人之间的循环回复。
* 对于 Slack 工具，删除 reaction 的语义见 [/tools/reactions](/zh/tools/reactions)。
* 在被允许且大小在限制范围内时，附件会被下载到媒体存储中。