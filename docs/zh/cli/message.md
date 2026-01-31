---
title: 消息
summary: "`openclaw message` 的 CLI 参考（发送 + 通道操作）"
read_when:
  - 添加或修改消息 CLI 操作
  - 更改出站通道行为
---

<div id="openclaw-message">
  # `openclaw message`
</div>

用于发送消息和执行频道操作的统一出站命令
（Discord/Google Chat/Slack/Mattermost（插件）/Telegram/WhatsApp/Signal/iMessage/MS Teams）。

<div id="usage">
  ## 使用方法
</div>

```
openclaw message <subcommand> [flags]
```

频道选择：

* 如果配置了多个频道，则必须指定 `--channel`。
* 如果只配置了一个频道，则该频道会作为默认频道。
* 取值：`whatsapp|telegram|discord|googlechat|slack|mattermost|signal|imessage|msteams`（Mattermost 需要插件）

目标格式（`--target`）：

* WhatsApp：E.164 或群组 JID
* Telegram：chat id 或 `@username`
* Discord：`channel:<id>` 或 `user:<id>`（或 `<@id>` 提及；纯数字 id 会被视为频道）
* Google Chat：`spaces/<spaceId>` 或 `users/<userId>`
* Slack：`channel:<id>` 或 `user:<id>`（接受原始频道 id）
* Mattermost（插件）：`channel:<id>`、`user:<id>`，或 `@username`（未加前缀的 id 会被视为频道）
* Signal：`+E.164`、`group:<id>`、`signal:+E.164`、`signal:group:<id>`，或 `username:<name>`/`u:<name>`
* iMessage：handle、`chat_id:<id>`、`chat_guid:<guid>`，或 `chat_identifier:<id>`
* MS Teams：会话 id（`19:...@thread.tacv2`）或 `conversation:<id>` 或 `user:<aad-object-id>`

名称解析：

* 对于支持的提供方（Discord/Slack 等），像 `Help` 或 `#help` 这样的频道名称会通过目录缓存进行解析。
* 当缓存未命中时，如果提供方支持，OpenClaw 会尝试进行实时目录查询。


<div id="common-flags">
  ## 常用参数
</div>

- `--channel <name>`
- `--account <id>`
- `--target <dest>`（用于 send/poll/read 等操作的目标通道或用户）
- `--targets <name>`（可重复；仅用于广播）
- `--json`
- `--dry-run`
- `--verbose`

<div id="actions">
  ## 操作
</div>

<div id="core">
  ### 核心
</div>

- `send`
  - 频道：WhatsApp/Telegram/Discord/Google Chat/Slack/Mattermost（插件）/Signal/iMessage/MS Teams
  - 必需：`--target`，以及 `--message` 或 `--media`
  - 可选：`--media`、`--reply-to`、`--thread-id`、`--gif-playback`
  - 仅限 Telegram：`--buttons`（需要在 `channels.telegram.capabilities.inlineButtons` 中启用）
  - 仅限 Telegram：`--thread-id`（论坛主题 ID）
  - 仅限 Slack：`--thread-id`（线程时间戳；`--reply-to` 使用同一字段）
  - 仅限 WhatsApp：`--gif-playback`

- `poll`
  - 频道：WhatsApp/Discord/MS Teams
  - 必需：`--target`、`--poll-question`、`--poll-option`（可重复）
  - 可选：`--poll-multi`
  - 仅限 Discord：`--poll-duration-hours`、`--message`

- `react`
  - 频道：Discord/Google Chat/Slack/Telegram/WhatsApp/Signal
  - 必需：`--message-id`、`--target`
  - 可选：`--emoji`、`--remove`、`--participant`、`--from-me`、`--target-author`、`--target-author-uuid`
  - 注意：`--remove` 需要同时提供 `--emoji`（在支持的平台上，省略 `--emoji` 以清除自己的所有表情反应；参见 /tools/reactions）
  - 仅限 WhatsApp：`--participant`、`--from-me`
  - Signal 群组表情反应：必须提供 `--target-author` 或 `--target-author-uuid`

- `reactions`
  - 频道：Discord/Google Chat/Slack
  - 必需：`--message-id`、`--target`
  - 可选：`--limit`

- `read`
  - 频道：Discord/Slack
  - 必需：`--target`
  - 可选：`--limit`、`--before`、`--after`
  - 仅限 Discord：`--around`

- `edit`
  - 频道：Discord/Slack
  - 必需：`--message-id`、`--message`、`--target`

- `delete`
  - 频道：Discord/Slack/Telegram
  - 必需：`--message-id`、`--target`

- `pin` / `unpin`
  - 频道：Discord/Slack
  - 必需：`--message-id`、`--target`

- `pins`（列表）
  - 频道：Discord/Slack
  - 必需：`--target`

- `permissions`
  - 频道：Discord
  - 必需：`--target`

- `search`
  - 频道：Discord
  - 必需：`--guild-id`、`--query`
  - 可选：`--channel-id`、`--channel-ids`（可重复）、`--author-id`、`--author-ids`（可重复）、`--limit`

<div id="threads">
  ### 线程
</div>

- `thread create`
  - 渠道：Discord
  - 必需：`--thread-name`、`--target`（频道 ID）
  - 可选：`--message-id`、`--auto-archive-min`

- `thread list`
  - 渠道：Discord
  - 必需：`--guild-id`
  - 可选：`--channel-id`、`--include-archived`、`--before`、`--limit`

- `thread reply`
  - 渠道：Discord
  - 必需：`--target`（线程 ID）、`--message`
  - 可选：`--media`、`--reply-to`

<div id="emojis">
  ### 表情符号
</div>

- `emoji list`
  - Discord：`--guild-id`
  - Slack：无额外参数

- `emoji upload`
  - 频道：Discord
  - 必需：`--guild-id`、`--emoji-name`、`--media`
  - 可选：`--role-ids`（可重复）

<div id="stickers">
  ### 贴纸
</div>

- `sticker send`
  - 渠道：Discord
  - 必需参数：`--target`、`--sticker-id`（可重复）
  - 可选参数：`--message`

- `sticker upload`
  - 渠道：Discord
  - 必需参数：`--guild-id`、`--sticker-name`、`--sticker-desc`、`--sticker-tags`、`--media`

<div id="roles-channels-members-voice">
  ### 角色 / 频道 / 成员 / 语音
</div>

- `role info` (Discord)：`--guild-id`
- `role add` / `role remove` (Discord)：`--guild-id`、`--user-id`、`--role-id`
- `channel info` (Discord)：`--target`
- `channel list` (Discord)：`--guild-id`
- `member info` (Discord/Slack)：`--user-id`（Discord 需额外提供 `--guild-id`）
- `voice status` (Discord)：`--guild-id`、`--user-id`

<div id="events">
  ### 事件
</div>

- `event list`（Discord）：`--guild-id`
- `event create`（Discord）：`--guild-id`、`--event-name`、`--start-time`
  - 可选：`--end-time`、`--desc`、`--channel-id`、`--location`、`--event-type`

<div id="moderation-discord">
  ### 管理（Discord）
</div>

- `timeout`：`--guild-id`、`--user-id`（可选 `--duration-min` 或 `--until`；两者都省略则清除超时状态）
- `kick`：`--guild-id`、`--user-id`（并指定 `--reason`）
- `ban`：`--guild-id`、`--user-id`（并指定 `--delete-days`、`--reason`）
  - `timeout` 也支持 `--reason`

<div id="broadcast">
  ### 广播
</div>

- `broadcast`
  - Channels：任何已配置的渠道；使用 `--channel all` 以面向所有提供方
  - 必须：`--targets`（可重复）
  - 可选：`--message`、`--media`、`--dry-run`

<div id="examples">
  ## 示例
</div>

发送 Discord 回复：

```
openclaw message send --channel discord \
  --target channel:123 --message "hi" --reply-to 456
```

在 Discord 中创建投票：

```
openclaw message poll --channel discord \
  --target channel:123 \
  --poll-question "Snack?" \
  --poll-option Pizza --poll-option Sushi \
  --poll-multi --poll-duration-hours 48
```

向 Teams 发送一条主动消息：

```
openclaw message send --channel msteams \
  --target conversation:19:abc@thread.tacv2 --message "hi"
```

在 Teams 中创建投票：

```
openclaw message poll --channel msteams \
  --target conversation:19:abc@thread.tacv2 \
  --poll-question "Lunch?" \
  --poll-option Pizza --poll-option Sushi
```

在 Slack 中添加表情反应：

```
openclaw message react --channel slack \
  --target C123 --message-id 456 --emoji "✅"
```

在 Signal 群组中添加表情回应：

```
openclaw message react --channel signal \
  --target signal:group:abc123 --message-id 1737630212345 \
  --emoji "✅" --target-author-uuid 123e4567-e89b-12d3-a456-426614174000
```

发送包含 Telegram 内联按钮的消息：

```
openclaw message send --channel telegram --target @mychat --message "请选择：" \
  --buttons '[ [{"text":"Yes","callback_data":"cmd:yes"}], [{"text":"No","callback_data":"cmd:no"}] ]'
```
