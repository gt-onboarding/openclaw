---
title: Discord
summary: "Discord 机器人支持状态、功能与配置"
read_when:
  - 在开发 Discord 频道相关功能时
---

<div id="discord-bot-api">
  # Discord（Bot API）
</div>

状态：已准备就绪，可通过官方 Discord 机器人 Gateway 处理私信（DM）和服务器文字频道。

<div id="quick-setup-beginner">
  ## 快速设置（初学者）
</div>

1. 创建一个 Discord 机器人并复制机器人的 token。
2. 在 Discord 应用设置中启用 **Message Content Intent**（如果你打算使用允许列表或名称查找，也需要启用 **Server Members Intent**）。
3. 为 OpenClaw 设置 token：
   * 环境变量：`DISCORD_BOT_TOKEN=...`
   * 或配置：`channels.discord.token: "..."`。
   * 如果两者都设置了，则以配置为准（环境变量只作为默认账号的兜底）。
4. 将机器人邀请到你的服务器并授予消息权限（如果你只想用私信（DM），可以创建一个私有服务器）。
5. 启动 Gateway。
6. 私信访问默认采用配对机制；首次联系时需要批准配对码。

最小配置：

```json5
{
  channels: {
    discord: {
      enabled: true,
      token: "YOUR_BOT_TOKEN"
    }
  }
}
```

<div id="goals">
  ## 目标
</div>

* 通过 Discord 私信（DM）或服务器频道与 OpenClaw 对话。
* 直接私聊会汇总到该智能体的主会话（默认 `agent:main:main`）；服务器频道则保持隔离为 `agent:<agentId>:discord:channel:<channelId>`（显示名称使用 `discord:<guildSlug>#<channelSlug>`）。
* 群组私信（Group DM）默认会被忽略；可通过 `channels.discord.dm.groupEnabled` 启用，并可选地用 `channels.discord.dm.groupChannels` 进行限制。
* 保持路由的确定性：回复始终发送回消息到达的同一频道。

<div id="how-it-works">
  ## 工作原理
</div>

1. 创建一个 Discord application → Bot，启用你需要的 intents（DM、服务器消息、消息内容），并获取 bot token。
2. 将 bot 邀请到你的服务器，并授予它在你希望使用的地方读取/发送消息所需的权限。
3. 使用 `channels.discord.token`（或回退为 `DISCORD_BOT_TOKEN`）配置 OpenClaw。
4. 运行 Gateway；当有可用的 token（优先使用配置，其次是环境变量），且 `channels.discord.enabled` 不为 `false` 时，会自动启动 Discord 频道。
   * 如果你更喜欢使用环境变量，设置 `DISCORD_BOT_TOKEN` 即可（配置块是可选的）。
5. 私信：投递时使用 `user:<id>`（或 `<@id>` 提及）；所有轮次会进入共享的 `main` 会话。裸数字 ID 不明确，会被拒绝。
6. 服务器频道：投递时使用 `channel:<channelId>`。默认要求使用提及，并且可以按服务器或按频道进行配置。
7. 私信：通过 `channels.discord.dm.policy` 实现默认安全（默认值：`"pairing"`）。未知发送者会收到一个配对码（1 小时后过期）；通过 `openclaw pairing approve discord <code>` 进行审批。
   * 如需保留旧的「对任何人开放」行为：设置 `channels.discord.dm.policy="open"`（允许来自任意用户的不受限消息）并将 `channels.discord.dm.allowFrom=["*"]`。
   * 如需严格使用允许列表控制：设置 `channels.discord.dm.policy="allowlist"`，并在 `channels.discord.dm.allowFrom` 中列出发送者。
   * 如需忽略所有 DM：设置 `channels.discord.dm.enabled=false` 或 `channels.discord.dm.policy="disabled"`。
8. 群组 DM 默认被忽略；可通过 `channels.discord.dm.groupEnabled` 启用，并可选地用 `channels.discord.dm.groupChannels` 进行限制。
9. 可选的服务器规则：通过 `channels.discord.guilds`，以服务器 id（推荐）或 slug 为键，配置每频道规则。
10. 可选的原生命令：`commands.native` 默认为 `"auto"`（Discord/Telegram 开启，Slack 关闭）。可通过 `channels.discord.commands.native: true|false|"auto"` 覆盖；`false` 会清除之前已注册的命令。文本命令由 `commands.text` 控制，且必须作为独立的 `/...` 消息发送。使用 `commands.useAccessGroups: false` 可绕过命令的访问组检查。
    * 完整命令列表与配置： [Slash commands](/zh/tools/slash-commands)
11. 可选的服务器上下文历史：设置 `channels.discord.historyLimit`（默认 20，回退到 `messages.groupChat.historyLimit`），在回复提及时包含最近 N 条服务器消息作为上下文。设为 `0` 可禁用。
12. Reactions：智能体可以通过 `discord` 工具触发 reactions（受 `channels.discord.actions.*` 控制）。
    * reaction 删除语义：参见 [/tools/reactions](/zh/tools/reactions)。
    * 只有当前频道为 Discord 时才会暴露 `discord` 工具。
13. 原生命令使用隔离的会话键（`agent:<agentId>:discord:slash:<userId>`），而不是共享的 `main` 会话。

Note: Name → id 解析通过服务器成员搜索实现，需要启用 Server Members Intent；如果 bot 无法搜索成员，请使用 id 或 `<@id>` 提及。
Note: slug 为小写，空格替换为 `-`。频道名称会被转为 slug，但不包含前导的 `#`。
Note: 服务器上下文中的 `[from:]` 行包含 `author.tag` + `id`，以便于快速构造可直接 @ 的回复。

<div id="config-writes">
  ## 配置写入
</div>

默认情况下，允许 Discord 通过 `/config set|unset` 写入配置更新（需要 `commands.config: true`）。

可通过以下方式禁用：

```json5
{
  channels: { discord: { configWrites: false } }
}
```

<div id="how-to-create-your-own-bot">
  ## 如何创建你自己的机器人
</div>

下面是在 “Discord Developer Portal” 中的相关设置，用于在服务器（guild）频道（例如 `#help`）中运行 OpenClaw。

<div id="1-create-the-discord-app-bot-user">
  ### 1）创建 Discord 应用和机器人用户
</div>

1. 打开 Discord Developer Portal → **Applications** → **New Application**
2. 在你的应用中：
   * 进入 **Bot** → **Add Bot**
   * 复制 **Bot Token**（这就是你要填入 `DISCORD_BOT_TOKEN` 的值）

<div id="2-enable-the-gateway-intents-openclaw-needs">
  ### 2) 启用 OpenClaw 所需的 Gateway Intents
</div>

除非你明确启用，否则 Discord 会阻止“特权 Intents”。

在 **Bot** → **Privileged Gateway Intents** 中，启用：

* **Message Content Intent**（在大多数服务器中读取消息文本所必需；否则你会看到 “Used disallowed intents”，或者机器人能够连接但不会对消息做出响应）
* **Server Members Intent**（推荐；用于某些成员/用户查询以及在服务器内进行允许列表匹配，是必需的）

通常你**不**需要启用 **Presence Intent**。

<div id="3-generate-an-invite-url-oauth2-url-generator">
  ### 3) 生成邀请 URL（OAuth2 URL 生成器）
</div>

在你的应用中：**OAuth2** → **URL Generator**

**Scopes**

* ✅ `bot`
* ✅ `applications.commands`（原生命令所必需）

**Bot 权限**（最低基线）

* ✅ 查看频道
* ✅ 发送消息
* ✅ 读取消息历史
* ✅ 嵌入链接
* ✅ 上传文件
* ✅ 添加表情反应（可选但推荐）
* ✅ 使用外部表情 / 贴纸（可选；仅在你需要时）

除非在调试并且你完全信任该 bot，否则避免使用 **Administrator**。

复制生成的 URL，打开它，选择你的服务器，然后安装该 bot。

<div id="4-get-the-ids-guilduserchannel">
  ### 4) 获取 ID（guild/user/channel）
</div>

Discord 到处都使用数值 ID；OpenClaw 配置更偏好使用 ID。

1. Discord（桌面端/网页端）→ **User Settings** → **Advanced** → 启用 **Developer Mode**
2. 右键点击：
   * 服务器名称 → **Copy Server ID**（guild ID）
   * 频道（例如 `#help`）→ **Copy Channel ID**
   * 你的账号 → **Copy User ID**

<div id="5-configure-openclaw">
  ### 5）配置 OpenClaw
</div>

<div id="token">
  #### 令牌
</div>

通过环境变量设置 bot token（推荐在服务器上使用）：

* `DISCORD_BOT_TOKEN=...`

或者在配置中设置：

```json5
{
  channels: {
    discord: {
      enabled: true,
      token: "YOUR_BOT_TOKEN"
    }
  }
}
```

多账户支持：使用 `channels.discord.accounts`，为每个账户配置各自的 token，并可选地设置 `name`。通用配置模式参见 [`gateway/configuration`](/zh/gateway/configuration#telegramaccounts--discordaccounts--slackaccounts--signalaccounts--imessageaccounts)。

<div id="allowlist-channel-routing">
  #### 允许列表 + 频道路由
</div>

示例：“单台服务器、仅允许我、仅允许 #help”：

```json5
{
  channels: {
    discord: {
      enabled: true,
      dm: { enabled: false },
      guilds: {
        "YOUR_GUILD_ID": {
          users: ["YOUR_USER_ID"],
          requireMention: true,
          channels: {
            help: { allow: true, requireMention: true }
          }
        }
      },
      retry: {
        attempts: 3,
        minDelayMs: 500,
        maxDelayMs: 30000,
        jitter: 0.1
      }
    }
  }
}
```

注意:

* `requireMention: true` 表示机器人只在被提及时才会回复（推荐用于共享频道）。
* `agents.list[].groupChat.mentionPatterns`（或 `messages.groupChat.mentionPatterns`）在公会消息中同样视为提及。
* 多智能体重写配置：在 `agents.list[].groupChat.mentionPatterns` 上为每个智能体分别设置匹配模式。
* 如果存在 `channels`，则任何未列出的频道默认被拒绝。
* 使用 `"*"` 频道条目可以将默认设置应用于所有频道；显式的频道条目会覆盖通配符。
* 线程会继承父频道配置（允许列表、`requireMention`、技能、提示词等），除非你显式为该线程添加频道 ID。
* 机器人自己发送的消息默认会被忽略；设置 `channels.discord.allowBots=true` 可以允许它们（机器人自己的消息仍会被过滤）。
* 警告：如果你允许回复其他机器人（`channels.discord.allowBots=true`），请通过 `requireMention`、`channels.discord.guilds.*.channels.<id>.users` 允许列表，以及在 `AGENTS.md` 和 `SOUL.md` 中设置清晰的安全护栏，以防止机器人之间产生回复循环。

<div id="6-verify-it-works">
  ### 6) 验证是否正常工作
</div>

1. 启动 Gateway。
2. 在你的服务器频道中发送：`@Krill hello`（或你机器人的名称）。
3. 如果没有任何反应：查看下方的**故障排查**。

<div id="troubleshooting">
  ### 故障排查
</div>

* 首先：运行 `openclaw doctor` 和 `openclaw channels status --probe`（可操作的警告 + 快速审计）。
* **“Used disallowed intents”**：在 Developer Portal 中启用 **Message Content Intent**（通常还需要 **Server Members Intent**），然后重启 Gateway。
* **Bot 能连接但在某个 guild 频道里从不回复**：
  * 缺少 **Message Content Intent**，或
  * Bot 缺乏频道权限（View/Send/Read History），或
  * 你的配置要求被提及但你没有 @ 它，或
  * 你的 guild/频道允许列表拒绝了该频道/用户。
* **`requireMention: false` 但仍然没有回复**：
* `channels.discord.groupPolicy` 默认是 **allowlist**；将其设置为 `"open"`（允许从任何用户无条件接收消息），或在 `channels.discord.guilds` 下添加一个 guild 条目（可以在 `channels.discord.guilds.<id>.channels` 下列出频道以进行限制）。
  * 如果你只设置了 `DISCORD_BOT_TOKEN` 而从未创建 `channels.discord` 段，在运行时
    会将 `groupPolicy` 默认为 `open`。添加 `channels.discord.groupPolicy`、
    `channels.defaults.groupPolicy`，或某个 guild/频道允许列表以收紧权限。
* `requireMention` 必须放在 `channels.discord.guilds`（或某个具体频道）下面。顶层的 `channels.discord.requireMention` 会被忽略。
* **权限审计**（`channels status --probe`）只检查数值型的频道 ID。如果你在 `channels.discord.guilds.*.channels` 中使用 slug/名称作为键，审计将无法验证权限。
* **DM 不工作**：`channels.discord.dm.enabled=false`、`channels.discord.dm.policy="disabled"`，或者你尚未通过批准（`channels.discord.dm.policy="pairing"`）。

<div id="capabilities-limits">
  ## 功能与限制
</div>

* 支持私信（DM）和服务器文本频道（将线程视作独立频道；不支持语音）。
* 尽力发送“正在输入”状态指示；消息分块使用 `channels.discord.textChunkLimit`（默认 2000），并按行数拆分篇幅较长的回复（`channels.discord.maxLinesPerMessage`，默认 17）。
* 可选的按空行分块：将 `channels.discord.chunkMode="newline"` 设为该值，以在按长度分块前先按空行（段落边界）拆分。
* 支持文件上传，大小上限为配置的 `channels.discord.mediaMaxMb`（默认 8 MB）。
* 为避免机器人刷屏，服务器内回复默认需要被提及后才会发送。
* 当一条消息引用另一条消息时，会注入回复上下文（被引用的内容 + id）。
* 原生回复线程功能**默认关闭**；可通过 `channels.discord.replyToMode` 和回复标签启用。

<div id="retry-policy">
  ## 重试策略
</div>

发出的 Discord API 调用在遇到限流（429）时会进行重试，若 Discord 提供 `retry_after` 则会使用该值，并配合指数退避和抖动。你可以通过 `channels.discord.retry` 进行配置。参见 [重试策略](/zh/concepts/retry)。

<div id="config">
  ## 配置
</div>

```json5
{
  channels: {
    discord: {
      enabled: true,
      token: "abc.123",
      groupPolicy: "allowlist",
      guilds: {
        "*": {
          channels: {
            general: { allow: true }
          }
        }
      },
      mediaMaxMb: 8,
      actions: {
        reactions: true,
        stickers: true,
        emojiUploads: true,
        stickerUploads: true,
        polls: true,
        permissions: true,
        messages: true,
        threads: true,
        pins: true,
        search: true,
        memberInfo: true,
        roleInfo: true,
        roles: false,
        channelInfo: true,
        channels: true,
        voiceStatus: true,
        events: true,
        moderation: false
      },
      replyToMode: "off",
      dm: {
        enabled: true,
        policy: "pairing", // 配对 | 允许列表 | 开放 | 禁用
        allowFrom: ["123456789012345678", "steipete"],
        groupEnabled: false,
        groupChannels: ["openclaw-dm"]
      },
      guilds: {
        "*": { requireMention: true },
        "123456789012345678": {
          slug: "friends-of-openclaw",
          requireMention: false,
          reactionNotifications: "own",
          users: ["987654321098765432", "steipete"],
          channels: {
            general: { allow: true },
            help: {
              allow: true,
              requireMention: true,
              users: ["987654321098765432"],
              skills: ["search", "docs"],
              systemPrompt: "Keep answers short."
            }
          }
        }
      }
    }
  }
}
```

确认反应在全局通过 `messages.ackReaction` +
`messages.ackReactionScope` 控制。使用 `messages.removeAckAfterReply` 在机器人回复后清除该确认反应。

* `dm.enabled`：设置为 `false` 时忽略所有 DM（默认 `true`）。
* `dm.policy`：DM 访问控制（推荐使用 `pairing`）。`"open"` 需要同时设置 `dm.allowFrom=["*"]`。
* `dm.allowFrom`：DM 允许列表（用户 id 或名称）。用于 `dm.policy="allowlist"`，也用于 `dm.policy="open"` 的校验。当机器人可以搜索成员时，配置向导会接受用户名并将其解析为 id。
* `dm.groupEnabled`：启用群组 DM（默认 `false`）。
* `dm.groupChannels`：可选的群组 DM 频道 id 或 slug 允许列表。
* `groupPolicy`：控制公会频道处理方式（`open|disabled|allowlist`）；`allowlist` 需要配置频道允许列表。
* `guilds`：按公会 id（优先）或 slug 作为键的每公会规则。
* `guilds."*"`：当不存在显式条目时应用的默认每公会设置。
* `guilds.<id>.slug`：用于显示名称的可选友好 slug。
* `guilds.<id>.users`：可选的每公会用户允许列表（id 或名称）。
* `guilds.<id>.tools`：可选的每公会 tool 策略覆盖（`allow`/`deny`/`alsoAllow`），在缺少频道级覆盖时使用。
* `guilds.<id>.toolsBySender`：可选的公会级按发送者划分的 tool 策略覆盖（在频道级覆盖缺失时生效；支持 `"*"` 通配符）。
* `guilds.<id>.channels.<channel>.allow`：在 `groupPolicy="allowlist"` 时允许/拒绝该频道。
* `guilds.<id>.channels.<channel>.requireMention`：该频道的 @ 提及门控。
* `guilds.<id>.channels.<channel>.tools`：可选的每频道 tool 策略覆盖（`allow`/`deny`/`alsoAllow`）。
* `guilds.<id>.channels.<channel>.toolsBySender`：频道内按发送者划分的可选 tool 策略覆盖（支持 `"*"` 通配符）。
* `guilds.<id>.channels.<channel>.users`：可选的每频道用户允许列表。
* `guilds.<id>.channels.<channel>.skills`：技能过滤（省略 = 所有技能，空列表 = 无）。
* `guilds.<id>.channels.<channel>.systemPrompt`：该频道的额外 system prompt（会与频道主题合并）。
* `guilds.<id>.channels.<channel>.enabled`：设置为 `false` 以禁用该频道。
* `guilds.<id>.channels`：频道规则（键为频道 slug 或 id）。
* `guilds.<id>.requireMention`：每公会的提及要求（可按频道覆盖）。
* `guilds.<id>.reactionNotifications`：reaction 系统事件模式（`off`、`own`、`all`、`allowlist`）。
* `textChunkLimit`：出站文本分块大小（字符数）。默认：2000。
* `chunkMode`：`length`（默认）仅在超过 `textChunkLimit` 时分块；`newline` 会在空行（段落边界）处分块，然后再按长度分块。
* `maxLinesPerMessage`：每条消息的软性最大行数。默认：17。
* `mediaMaxMb`：限制保存到磁盘的入站媒体大小上限。
* `historyLimit`：在回复提及时，作为上下文包含的最近公会消息数量（默认 20；回退到 `messages.groupChat.historyLimit`；`0` 表示禁用）。
* `dmHistoryLimit`：DM 的历史记录上限（按用户轮次计）。每用户覆盖：`dms["<user_id>"].historyLimit`。
* `retry`：出站 Discord API 调用的重试策略（attempts、minDelayMs、maxDelayMs、jitter）。
* `actions`：按操作划分的 tool 权限门控；省略表示允许所有（设置为 `false` 表示禁用）。
  * `reactions`（涵盖表情反应 + read reactions）
  * `stickers`、`emojiUploads`、`stickerUploads`、`polls`、`permissions`、`messages`、`threads`、`pins`、`search`
  * `memberInfo`、`roleInfo`、`channelInfo`、`voiceStatus`、`events`
  * `channels`（创建/编辑/删除频道 + 分类 + 权限）
  * `roles`（角色添加/移除，默认 `false`）
  * `moderation`（超时/踢出/封禁，默认 `false`）

Reaction 通知使用 `guilds.<id>.reactionNotifications`：

* `off`：不处理任何 reaction 事件。
* `own`：仅处理机器人自己消息上的 reactions（默认）。
* `all`：处理所有消息上的所有 reactions。
* `allowlist`：处理来自 `guilds.<id>.users` 的 reactions，作用于所有消息（空列表则禁用）。

<div id="tool-action-defaults">
  ### 工具操作默认设置
</div>

| 操作组 | 默认值 | 说明 |
| --- | --- | --- |
| reactions | 启用 | 添加反应 + 列出反应 + emojiList |
| stickers | 启用 | 发送贴纸 |
| emojiUploads | 启用 | 上传表情 |
| stickerUploads | 启用 | 上传贴纸 |
| polls | 启用 | 创建投票 |
| permissions | 启用 | 频道权限快照 |
| messages | 启用 | 读取/发送/编辑/删除 |
| threads | 启用 | 创建/列出/回复 |
| pins | 启用 | 固定/取消固定/列出 |
| search | 启用 | 消息搜索（预览功能） |
| memberInfo | 启用 | 成员信息 |
| roleInfo | 启用 | 角色列表 |
| channelInfo | 启用 | 频道信息 + 列表 |
| channels | 启用 | 频道/分类管理 |
| voiceStatus | 启用 | 语音状态查询 |
| events | 启用 | 列出/创建预定事件 |
| roles | 禁用 | 添加/移除角色 |
| moderation | 禁用 | 禁言/踢出/封禁 |

* `replyToMode`：`off`（默认）、`first` 或 `all`。仅当模型输出包含回复标签时生效。

<div id="reply-tags">
  ## 回复标签
</div>

要请求线程式回复，模型可以在输出中包含以下标签之一：

* `[[reply_to_current]]` — 回复触发当前交互的 Discord 消息。
* `[[reply_to:<id>]]` — 回复上下文/历史中的特定消息 id。
  当前消息 id 会作为 `[message_id: …]` 追加到提示中；历史记录条目已经包含 id。

行为由 `channels.discord.replyToMode` 控制：

* `off`: 忽略标签。
* `first`: 只有第一个出站分片/附件作为回复。
* `all`: 每一个出站分片/附件都作为回复。

允许列表匹配说明：

* `allowFrom`/`users`/`groupChannels` 接受 id、名称、标签，或类似 `<@id>` 的提及。
* 支持 `discord:`/`user:`（用户）和 `channel:`（群组私信）等前缀。
* 使用 `*` 允许任意发送者/频道。
* 当存在 `guilds.<id>.channels` 时，未列出的频道默认被拒绝。
* 当省略 `guilds.<id>.channels` 时，允许列表中的该 guild 内的所有频道。
* 若要不允许任何频道，将 `channels.discord.groupPolicy` 设为 `"disabled"`（或保持允许列表为空）。
* 配置向导接受 `Guild/Channel` 名称（公开 + 私有），并在可能时将其解析为 ID。
* 启动时，OpenClaw 会将允许列表中的频道/用户名称解析为 ID（在机器人可以搜索成员的情况下），
  并将映射关系写入日志；未解析的条目将按原样保留。

原生命令说明：

* 已注册命令与 OpenClaw 的聊天命令一一对应。
* 原生命令遵守与私信/guild 消息相同的允许列表（`channels.discord.dm.allowFrom`、`channels.discord.guilds`、按频道规则）。
* 即使用户不在允许列表中，斜杠命令仍可能在 Discord UI 中可见；OpenClaw 会在执行时强制检查允许列表，并回复“未授权”。

<div id="tool-actions">
  ## 工具操作
</div>

智能体可以通过 `discord` 调用以下操作：

* `react` / `reactions`（添加或列出表情反应）
* `sticker`、`poll`、`permissions`
* `readMessages`、`sendMessage`、`editMessage`、`deleteMessage`
* 读取/搜索/置顶类工具的负载会包含标准化的 `timestampMs`（UTC 纪元毫秒）和 `timestampUtc`，以及原始的 Discord `timestamp`。
* `threadCreate`、`threadList`、`threadReply`
* `pinMessage`、`unpinMessage`、`listPins`
* `searchMessages`、`memberInfo`、`roleInfo`、`roleAdd`、`roleRemove`、`emojiList`
* `channelInfo`、`channelList`、`voiceStatus`、`eventList`、`eventCreate`
* `timeout`、`kick`、`ban`

Discord 消息 ID 会在注入的上下文中提供（`[discord message id: …]` 以及历史记录行），从而让智能体可以精准定位这些消息。
表情可以是 Unicode（例如 `✅`），也可以是类似 `<:party_blob:1234567890>` 的自定义表情语法。

<div id="safety-ops">
  ## 安全与运维
</div>

* 将机器人令牌视为密码；在受管主机上优先使用 `DISCORD_BOT_TOKEN` 环境变量，或严格限制配置文件权限。
* 仅授予机器人所需的权限（通常是“读取/发送消息”（Read/Send Messages）权限）。
* 如果机器人卡住或被限流，在确认没有其他进程占用该 Discord 会话后再重启 Gateway（`openclaw gateway --force`）。