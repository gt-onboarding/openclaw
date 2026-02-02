---
title: メッセージ
summary: "CLI リファレンス: `openclaw message`（送信 + チャネル操作）"
read_when:
  - メッセージ CLI アクションを追加または変更するとき
  - 送信チャネルの動作を変更するとき
---

<div id="openclaw-message">
  # `openclaw message`
</div>

メッセージおよびチャンネルアクションを送信するための単一の送信コマンド
（Discord/Google Chat/Slack/Mattermost（プラグイン）/Telegram/WhatsApp/Signal/iMessage/MS Teams）。

<div id="usage">
  ## 使用方法
</div>

```
openclaw message <subcommand> [flags]
```

チャンネル選択:

* 複数のチャンネルが設定されている場合は、`--channel` が必須です。
* チャンネルがちょうど 1 つだけ設定されている場合、それがデフォルトになります。
* 値: `whatsapp|telegram|discord|googlechat|slack|mattermost|signal|imessage|msteams`（Mattermost はプラグインが必要）

ターゲット形式（`--target`）:

* WhatsApp: E.164 またはグループ JID
* Telegram: チャット ID または `@username`
* Discord: `channel:<id>` または `user:<id>`（もしくは `<@id>` メンション。数値のみの ID はチャンネルとして扱われます）
* Google Chat: `spaces/<spaceId>` または `users/<userId>`
* Slack: `channel:<id>` または `user:<id>`（チャンネル ID をそのまま指定することもできます）
* Mattermost（プラグイン）: `channel:<id>`、`user:<id>`、または `@username`（素の ID はチャンネルとして扱われます）
* Signal: `+E.164`、`group:<id>`、`signal:+E.164`、`signal:group:<id>`、または `username:<name>` / `u:<name>`
* iMessage: ハンドル、`chat_id:<id>`、`chat_guid:<guid>`、または `chat_identifier:<id>`
* MS Teams: 会話 ID（`19:...@thread.tacv2`）または `conversation:<id>` または `user:<aad-object-id>`

名前解決:

* 対応しているプロバイダー（Discord/Slack など）の場合、`Help` や `#help` のようなチャンネル名はディレクトリキャッシュ経由で解決されます。
* キャッシュに存在しない場合、プロバイダーがサポートしていれば OpenClaw はリアルタイムのディレクトリ検索を試みます。


<div id="common-flags">
  ## 共通フラグ
</div>

- `--channel <name>`
- `--account <id>`
- `--target <dest>` (送信/poll/read などの対象となるチャネルまたはユーザー)
- `--targets <name>` (繰り返し指定可。ブロードキャスト時のみ使用)
- `--json`
- `--dry-run`
- `--verbose`

<div id="actions">
  ## アクション
</div>

<div id="core">
  ### コア
</div>

- `send`
  - チャンネル: WhatsApp/Telegram/Discord/Google Chat/Slack/Mattermost (プラグイン)/Signal/iMessage/MS Teams
  - 必須: `--target` に加えて `--message` または `--media`
  - 任意: `--media`, `--reply-to`, `--thread-id`, `--gif-playback`
  - Telegram のみ: `--buttons` (`channels.telegram.capabilities.inlineButtons` を有効にしておく必要あり)
  - Telegram のみ: `--thread-id` (フォーラムトピック ID)
  - Slack のみ: `--thread-id` (スレッドのタイムスタンプ。`--reply-to` も同じフィールドを使用)
  - WhatsApp のみ: `--gif-playback`

- `poll`
  - チャンネル: WhatsApp/Discord/MS Teams
  - 必須: `--target`, `--poll-question`, `--poll-option` (繰り返し指定)
  - 任意: `--poll-multi`
  - Discord のみ: `--poll-duration-hours`, `--message`

- `react`
  - チャンネル: Discord/Google Chat/Slack/Telegram/WhatsApp/Signal
  - 必須: `--message-id`, `--target`
  - 任意: `--emoji`, `--remove`, `--participant`, `--from-me`, `--target-author`, `--target-author-uuid`
  - 注意: `--remove` には `--emoji` が必須 (`--emoji` を省略すると、サポートされている場合は自分のリアクションをクリアします。/tools/reactions を参照)
  - WhatsApp のみ: `--participant`, `--from-me`
  - Signal のグループリアクション: `--target-author` または `--target-author-uuid` が必須

- `reactions`
  - チャンネル: Discord/Google Chat/Slack
  - 必須: `--message-id`, `--target`
  - 任意: `--limit`

- `read`
  - チャンネル: Discord/Slack
  - 必須: `--target`
  - 任意: `--limit`, `--before`, `--after`
  - Discord のみ: `--around`

- `edit`
  - チャンネル: Discord/Slack
  - 必須: `--message-id`, `--message`, `--target`

- `delete`
  - チャンネル: Discord/Slack/Telegram
  - 必須: `--message-id`, `--target`

- `pin` / `unpin`
  - チャンネル: Discord/Slack
  - 必須: `--message-id`, `--target`

- `pins` (一覧)
  - チャンネル: Discord/Slack
  - 必須: `--target`

- `permissions`
  - チャンネル: Discord
  - 必須: `--target`

- `search`
  - チャンネル: Discord
  - 必須: `--guild-id`, `--query`
  - 任意: `--channel-id`, `--channel-ids` (繰り返し指定), `--author-id`, `--author-ids` (繰り返し指定), `--limit`

<div id="threads">
  ### スレッド
</div>

- `thread create`
  - 対応チャネル: Discord
  - 必須: `--thread-name`, `--target`（チャンネル ID）
  - 任意: `--message-id`, `--auto-archive-min`

- `thread list`
  - 対応チャネル: Discord
  - 必須: `--guild-id`
  - 任意: `--channel-id`, `--include-archived`, `--before`, `--limit`

- `thread reply`
  - 対応チャネル: Discord
  - 必須: `--target`（スレッド ID）, `--message`
  - 任意: `--media`, `--reply-to`

<div id="emojis">
  ### 絵文字
</div>

- `emoji list`
  - Discord: `--guild-id`
  - Slack: 追加フラグなし

- `emoji upload`
  - チャンネル: Discord
  - 必須: `--guild-id`, `--emoji-name`, `--media`
  - 任意: `--role-ids` (繰り返し指定可)

<div id="stickers">
  ### スタンプ
</div>

- `sticker send`
  - チャンネル: Discord
  - 必須: `--target`, `--sticker-id` (複数指定可)
  - 任意: `--message`

- `sticker upload`
  - チャンネル: Discord
  - 必須: `--guild-id`, `--sticker-name`, `--sticker-desc`, `--sticker-tags`, `--media`

<div id="roles-channels-members-voice">
  ### ロール / チャンネル / メンバー / ボイス
</div>

- `role info` (Discord): `--guild-id`
- `role add` / `role remove` (Discord): `--guild-id`, `--user-id`, `--role-id`
- `channel info` (Discord): `--target`
- `channel list` (Discord): `--guild-id`
- `member info` (Discord/Slack): `--user-id` (Discord の場合は `--guild-id` も必要)
- `voice status` (Discord): `--guild-id`, `--user-id`

<div id="events">
  ### イベント
</div>

- `event list` (Discord): `--guild-id`
- `event create` (Discord): `--guild-id`, `--event-name`, `--start-time`
  - オプション: `--end-time`, `--desc`, `--channel-id`, `--location`, `--event-type`

<div id="moderation-discord">
  ### モデレーション (Discord)
</div>

- `timeout`: `--guild-id`, `--user-id` (省略可能: `--duration-min` または `--until`。両方省略するとタイムアウトを解除)
- `kick`: `--guild-id`, `--user-id` (+ `--reason`)
- `ban`: `--guild-id`, `--user-id` (+ `--delete-days`, `--reason`)
  - `timeout` でも `--reason` を指定可能

<div id="broadcast">
  ### ブロードキャスト
</div>

- `broadcast`
  - チャンネル: 任意の構成済みチャンネル。すべてのプロバイダーを対象にするには `--channel all` を使用
  - 必須: `--targets`（繰り返し指定可）
  - 任意: `--message`, `--media`, `--dry-run`

<div id="examples">
  ## 例
</div>

Discord への返信を送信:

```
openclaw message send --channel discord \
  --target channel:123 --message "hi" --reply-to 456
```

Discord で投票を作成する：

```
openclaw message poll --channel discord \
  --target channel:123 \
  --poll-question "Snack?" \
  --poll-option Pizza --poll-option Sushi \
  --poll-multi --poll-duration-hours 48
```

Teams にプロアクティブ メッセージを送信する:

```
openclaw message send --channel msteams \
  --target conversation:19:abc@thread.tacv2 --message "hi"
```

Teams の投票を作成する:

```
openclaw message poll --channel msteams \
  --target conversation:19:abc@thread.tacv2 \
  --poll-question "Lunch?" \
  --poll-option Pizza --poll-option Sushi
```

Slack でリアクションを付ける：

```
openclaw message react --channel slack \
  --target C123 --message-id 456 --emoji "✅"
```

Signal グループでリアクションを付ける:

```
openclaw message react --channel signal \
  --target signal:group:abc123 --message-id 1737630212345 \
  --emoji "✅" --target-author-uuid 123e4567-e89b-12d3-a456-426614174000
```

Telegram インラインボタンを送信:

```
openclaw message send --channel telegram --target @mychat --message "選択してください:" \
  --buttons '[ [{"text":"Yes","callback_data":"cmd:yes"}], [{"text":"No","callback_data":"cmd:no"}] ]'
```
