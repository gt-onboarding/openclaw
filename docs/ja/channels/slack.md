---
title: Slack
summary: "ソケットまたは HTTP webhook モード向けの Slack セットアップ"
read_when: "Slack をセットアップするとき、または Slack のソケット/HTTP モードをデバッグするとき"
---

<div id="slack">
  # Slack
</div>

<div id="socket-mode-default">
  ## ソケットモード（デフォルト）
</div>

<div id="quick-setup-beginner">
  ### クイックセットアップ（初心者向け）
</div>

1. Slack アプリを作成し、**Socket Mode** を有効にします。
2. **App Token**（`xapp-...`）と **Bot Token**（`xoxb-...`）を作成します。
3. OpenClaw にトークンを設定して、Gateway を起動します。

最小構成例:

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
  ### セットアップ
</div>

1. https://api.slack.com/apps で Slack アプリを「From scratch」で作成します。
2. **Socket Mode** → 有効化します。その後 **Basic Information** → **App-Level Tokens** → **Generate Token and Scopes** で `connections:write` スコープ付きトークンを作成します。**App Token**（`xapp-...`）をコピーします。
3. **OAuth &amp; Permissions** → Bot トークンのスコープを追加します（下記のマニフェストを使用）。**Install to Workspace** をクリックします。**Bot User OAuth Token**（`xoxb-...`）をコピーします。
4. 任意: **OAuth &amp; Permissions** → **User Token Scopes** を追加します（下の read-only リストを参照）。アプリを再インストールし、**User OAuth Token**（`xoxp-...`）をコピーします。
5. **Event Subscriptions** → イベントを有効化し、以下のイベントを購読します:
   * `message.*`（編集/削除/スレッドのブロードキャストを含む）
   * `app_mention`
   * `reaction_added`, `reaction_removed`
   * `member_joined_channel`, `member_left_channel`
   * `channel_rename`
   * `pin_added`, `pin_removed`
6. ボットを、メッセージを読み取らせたいチャンネルに招待します。
7. Slash Commands → `channels.slack.slashCommand` を使う場合は `/openclaw` を作成します。ネイティブコマンドを有効にする場合は、組み込みコマンドごとに 1 つスラッシュコマンドを追加します（`/help` と同じ名前）。Slack ではネイティブコマンドはデフォルトで無効です。Slack で有効にするには `channels.slack.commands.native: true` を設定します（グローバルな `commands.native` は `"auto"` で、Slack は無効のままになります）。
8. App Home → **Messages Tab** を有効にして、ユーザーがボットに DM できるようにします。

下のマニフェストを使用すると、スコープとイベントの設定を同期した状態に保てます。

マルチアカウント対応: `channels.slack.accounts` を使用し、アカウントごとにトークンと任意の `name` を設定します。共通パターンについては [`gateway/configuration`](/ja/gateway/configuration#telegramaccounts--discordaccounts--slackaccounts--signalaccounts--imessageaccounts) を参照してください。

<div id="openclaw-config-minimal">
  ### OpenClaw 設定（最小構成）
</div>

トークンは環境変数で指定します（推奨）:

* `SLACK_APP_TOKEN=xapp-...`
* `SLACK_BOT_TOKEN=xoxb-...`

または設定ファイルで指定します:

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
  ### User token (optional)
</div>

OpenClaw は Slack のユーザートークン（`xoxp-...`）を read 操作（履歴、
ピン留め、リアクション、絵文字、メンバー情報）に使用できます。デフォルトでは
これは読み取り専用のままです。user token が存在する場合は read には
user token が優先して使われ、write については、明示的にオプトインしない限り
引き続き bot token が使われます。`userTokenReadOnly: false` を指定しても、
bot token が利用可能な場合は write には引き続き bot token が優先されます。

User token は設定ファイルで指定します（環境変数には非対応）。複数アカウント
構成では、`channels.slack.accounts.<id>.userToken` を設定します。

bot + app + user token を併用する例:

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

userTokenReadOnly を明示的に設定した例（ユーザートークンでの書き込みを許可する）:

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
  #### トークンの使用
</div>

* 読み取り系の操作（履歴、リアクションリスト、ピンリスト、絵文字リスト、メンバー情報、
  検索）は、設定されていればユーザートークンを優先し、それ以外の場合はボットトークンを使用します。
* 書き込み系の操作（メッセージの送信・編集・削除、リアクションの追加・削除、ピン留め・ピン解除、
  ファイルアップロード）は、デフォルトでボットトークンを使用します。`userTokenReadOnly: false` かつ
  ボットトークンが利用できない場合、OpenClaw はユーザートークンにフォールバックします。

<div id="history-context">
  ### 履歴コンテキスト
</div>

* `channels.slack.historyLimit`（または `channels.slack.accounts.*.historyLimit`）は、プロンプトに含める直近のチャンネル／グループメッセージ数を制御します。
* 設定されていない場合は `messages.groupChat.historyLimit` が使用されます。`0` を設定すると無効になり（デフォルトは 50）、履歴は送信されません。

<div id="http-mode-events-api">
  ## HTTP モード (Events API)
</div>

Gateway が HTTPS 経由で Slack から到達可能な場合（一般的にはサーバーにデプロイしている場合）、HTTP Webhook モードを使用します。
HTTP モードでは、Events API + Interactivity + Slash Commands を共有のリクエスト URL 1 つで利用します。

<div id="setup">
  ### セットアップ
</div>

1. Slack アプリを作成し、**Socket Mode を無効化**します（HTTP のみを使用する場合は任意）。
2. **Basic Information** → **Signing Secret** をコピーします。
3. **OAuth &amp; Permissions** → アプリをインストールし、**Bot User OAuth Token**（`xoxb-...`）をコピーします。
4. **Event Subscriptions** → イベントを有効化し、**Request URL** に Gateway の webhook パス（デフォルト `/slack/events`）を設定します。
5. **Interactivity &amp; Shortcuts** → 有効化して、同じ **Request URL** を設定します。
6. **Slash Commands** → コマンド用に同じ **Request URL** を設定します。

Request URL の例:
`https://gateway-host/slack/events`

### OpenClaw 設定（最小限）

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

マルチアカウント HTTP モード: `channels.slack.accounts.<id>.mode = "http"` を設定し、各アカウントごとに一意の
`webhookPath` を指定して、各 Slack アプリがそれぞれ自分専用の URL を指すようにします。

<div id="manifest-optional">
  ### マニフェスト（任意）
</div>

この Slack アプリのマニフェストを使うと、アプリを素早く作成できます（必要に応じて名前やコマンドを調整してください）。ユーザートークンを設定する予定がある場合は、ユーザースコープも含めてください。

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

ネイティブコマンドを有効にする場合は、公開したい（`/help` の一覧と一致させたい）各コマンドごとに 1 つの `slash_commands` エントリを追加します。`channels.slack.commands.native` でオーバーライドできます。

<div id="scopes-current-vs-optional">
  ## スコープ（現在のもの vs 任意のもの）
</div>

Slack の Conversations API は会話タイプごとのスコープ指定になっています。実際に扱う会話タイプ（channels, groups, im, mpim）に対するスコープだけを付与すれば十分です。概要については https://docs.slack.dev/apis/web-api/using-the-conversations-api/ を参照してください。

<div id="bot-token-scopes-required">
  ### Bot token スコープ（必須）
</div>

* `chat:write`（`chat.postMessage` を介してメッセージを送信／更新／削除）
  https://docs.slack.dev/reference/methods/chat.postMessage
* `im:write`（ユーザー DM に対して `conversations.open` を使用して DM を開始）
  https://docs.slack.dev/reference/methods/conversations.open
* `channels:history`, `groups:history`, `im:history`, `mpim:history`
  https://docs.slack.dev/reference/methods/conversations.history
* `channels:read`, `groups:read`, `im:read`, `mpim:read`
  https://docs.slack.dev/reference/methods/conversations.info
* `users:read`（ユーザー情報の参照）
  https://docs.slack.dev/reference/methods/users.info
* `reactions:read`, `reactions:write`（`reactions.get` / `reactions.add`）
  https://docs.slack.dev/reference/methods/reactions.get
  https://docs.slack.dev/reference/methods/reactions.add
* `pins:read`, `pins:write`（`pins.list` / `pins.add` / `pins.remove`）
  https://docs.slack.dev/reference/scopes/pins.read
  https://docs.slack.dev/reference/scopes/pins.write
* `emoji:read`（`emoji.list`）
  https://docs.slack.dev/reference/scopes/emoji.read
* `files:write`（`files.uploadV2` を介したアップロード）
  https://docs.slack.dev/messaging/working-with-files/#upload

<div id="user-token-scopes-optional-read-only-by-default">
  ### ユーザートークンのスコープ（任意・デフォルトは読み取り専用）
</div>

`channels.slack.userToken` を設定する場合は、**User Token Scopes** に以下を追加します。

* `channels:history`, `groups:history`, `im:history`, `mpim:history`
* `channels:read`, `groups:read`, `im:read`, `mpim:read`
* `users:read`
* `reactions:read`
* `pins:read`
* `emoji:read`
* `search:read`

<div id="not-needed-today-but-likely-future">
  ### 現時点では不要（ただし将来的には必要になる可能性が高い）
</div>

* `mpim:write`（`conversations.open` を使ってグループ DM の open/DM 開始機能を追加する場合のみ必要）
* `groups:write`（プライベートチャンネルの管理機能: 作成/名前変更/招待/アーカイブ を追加する場合のみ必要）
* `chat:write.public`（Bot が参加していないチャンネルに投稿したい場合のみ必要）
  https://docs.slack.dev/reference/scopes/chat.write.public
* `users:read.email`（`users.info` から email フィールドが必要な場合のみ必要）
  https://docs.slack.dev/changelog/2017-04-narrowing-email-access
* `files:read`（ファイルのメタデータの一覧取得/読み取りを行う場合のみ必要）

<div id="config">
  ## 設定
</div>

Slack は Socket モードのみを使用します（HTTP の Webhook サーバーは利用しません）。2 つのトークンを両方とも指定してください：

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

トークンは環境変数から指定することもできます：

* `SLACK_BOT_TOKEN`
* `SLACK_APP_TOKEN`

Ack リアクションは `messages.ackReaction` と
`messages.ackReactionScope` によってグローバルに制御されます。`messages.removeAckAfterReply` を使用して、ボットが返信した後に
Ack リアクションを削除します。

<div id="limits">
  ## 制限事項
</div>

* 送信テキストは `channels.slack.textChunkLimit`（デフォルト 4000）でチャンク化されます。
* 改行ごとのチャンク化を有効にするには、長さによるチャンク化の前に空行（段落境界）で分割するよう `channels.slack.chunkMode="newline"` を設定します。
* メディアのアップロードサイズは `channels.slack.mediaMaxMb`（デフォルト 20）で上限が設定されています。

<div id="reply-threading">
  ## 返信のスレッド化
</div>

デフォルトでは、OpenClaw はメインチャンネルに返信します。`channels.slack.replyToMode` を使って自動スレッド化のモードを設定できます。

| モード | 動作 |
| --- | --- |
| `off` | **デフォルト。** メインチャンネルに返信します。トリガーとなったメッセージがすでにスレッド内にある場合のみ、そのスレッドに返信します。 |
| `first` | 最初の返信はスレッド（トリガーとなったメッセージの下）に送信し、2 回目以降の返信はメインチャンネルに送信します。スレッドの散在を避けつつ、コンテキストを見える形で保ちたい場合に有用です。 |
| `all` | すべての返信をスレッドに送信します。会話をスレッド内に収められますが、可視性が下がる可能性があります。 |

このモードは、自動返信とエージェントのツール呼び出し（`slack sendMessage`）の両方に適用されます。

<div id="per-chat-type-threading">
  ### チャット種別ごとのスレッド化
</div>

`channels.slack.replyToModeByChatType` を設定することで、チャット種別ごとに異なるスレッド動作を設定できます。

```json5
{
  channels: {
    slack: {
      replyToMode: "off",        // default for channels
      replyToModeByChatType: {
        direct: "all",           // DMs always thread
        group: "first"           // グループDM/MPIMは最初の返信をスレッド化
      },
    }
  }
}
```

サポートされているチャット種別：

* `direct`: 1:1 の DM（Slack の `im`）
* `group`: グループ DM / MPIM（Slack の `mpim`）
* `channel`: 標準チャンネル（パブリック / プライベート）

優先順位：

1. `replyToModeByChatType.<chatType>`
2. `replyToMode`
3. プロバイダーのデフォルト（`off`）

レガシーな `channels.slack.dm.replyToMode` も、チャット種別ごとのオーバーライドが設定されていない場合の `direct` 用フォールバックとして引き続き有効です。

例：

DM のスレッド返信のみとする場合：

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

グループDMはスレッド化し、チャンネルはルートに残します:

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

チャンネルはスレッドで返信し、DM は常に通常メッセージとして投稿する:

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
  ### 手動スレッドタグ
</div>

より細かく制御したい場合は、エージェントの応答内で次のタグを使用します。

* `[[reply_to_current]]` — トリガーとなったメッセージに返信します（スレッドを開始／継続）。
* `[[reply_to:<id>]]` — 特定のメッセージIDに返信します。

<div id="sessions-routing">
  ## セッションとルーティング
</div>

* DM（ダイレクトメッセージ）は `main` セッションを共有します（WhatsApp/Telegram と同様）。
* チャンネルは `agent:<agentId>:slack:channel:<channelId>` セッションにマッピングされます。
* スラッシュコマンドは `agent:<agentId>:slack:slash:<userId>` セッションを使用します（プレフィックスは `channels.slack.slashCommand.sessionPrefix` で設定可能）。
* Slack が `channel_type` を提供しない場合、OpenClaw はチャンネル ID のプレフィックス（`D`、`C`、`G`）から推論し、セッションキーを安定させるためにデフォルトで `channel` を使用します。
* ネイティブコマンド登録には `commands.native`（グローバルデフォルトは `"auto"` → Slack 向けは無効）を使用し、`channels.slack.commands.native` によってワークスペース単位で上書きできます。テキストコマンドは単独の `/...` メッセージを必要とし、`commands.text: false` で無効化できます。Slack のスラッシュコマンドは Slack アプリ側で管理され、自動的には削除されません。コマンド用のアクセスグループチェックをバイパスするには `commands.useAccessGroups: false` を使用します。
* コマンド一覧と設定の詳細: [Slash commands](/ja/tools/slash-commands)

<div id="dm-security-pairing">
  ## DM セキュリティ（ペアリング）
</div>

* デフォルト: `channels.slack.dm.policy="pairing"` — 未知の DM 送信者にはペアリングコードが返されます（1 時間後に失効）。
* 承認方法: `openclaw pairing approve slack <code>` を実行します。
* 誰からでも許可するには: `channels.slack.dm.policy="open"`（任意のユーザーからのメッセージ受付を無制限に許可する設定）と `channels.slack.dm.allowFrom=["*"]` を設定します。
* `channels.slack.dm.allowFrom` にはユーザー ID、@ハンドル、またはメールアドレスを指定できます（トークンが許可している場合、起動時に解決されます）。ウィザードではユーザー名を受け付け、トークンが許可している場合、セットアップ中にそれらを ID に対応付けます。

<div id="group-policy">
  ## グループポリシー
</div>

* `channels.slack.groupPolicy` はチャネルの処理方法を制御します（`open|disabled|allowlist`）。
* `allowlist` を使用する場合、チャネルは `channels.slack.channels` に列挙されている必要があります。
* `SLACK_BOT_TOKEN`/`SLACK_APP_TOKEN` だけを設定して `channels.slack` セクションを作成しない場合、
  実行時には `groupPolicy` のデフォルトが `open`（任意のユーザーからのメッセージを無制限に受け入れる設定）になります。これを制限するには、`channels.slack.groupPolicy`、
  `channels.defaults.groupPolicy`、またはチャネルの許可リストを追加してください。
* 設定ウィザードは `#channel` 名を受け取り、可能な場合にはそれを ID に解決します
  （パブリック + プライベート）。複数一致がある場合は、アクティブなチャネルを優先します。
* 起動時に、OpenClaw は許可リスト内のチャネル/ユーザー名を（トークンが許可する範囲で）ID に解決し、
  その対応関係をログに記録します。解決できなかったエントリは、入力された文字列のまま保持されます。
* **どのチャネルも許可しない** 場合は、`channels.slack.groupPolicy: "disabled"` を設定する（または許可リストを空のままにする）ことで実現できます。

チャネルオプション（`channels.slack.channels.<id>` または `channels.slack.channels.<name>`）:

* `allow`: `groupPolicy="allowlist"` の場合に、このチャネルを許可/拒否します。
* `requireMention`: このチャネルでメンションが必要かどうかを制御します。
* `tools`: チャネル単位の任意のツールポリシー上書き設定（`allow`/`deny`/`alsoAllow`）。
* `toolsBySender`: チャネル内の送信者単位の任意のツールポリシー上書き設定（キーは送信者の ID/@ハンドル/メールアドレス；`"*"` ワイルドカード対応）。
* `allowBots`: このチャネルでボットが作成したメッセージを許可します（デフォルト: false）。
* `users`: チャネル単位の任意のユーザー許可リスト。
* `skills`: スキルフィルタ（省略 = すべてのスキル、空 = なし）。
* `systemPrompt`: このチャネル用の追加の system prompt（トピック/目的と統合されます）。
* `enabled`: `false` を設定すると、このチャネルを無効化します。

<div id="delivery-targets">
  ## 配信ターゲット
</div>

これらは cron や CLI の送信で使用します:

* `user:<id>` は DM 用
* `channel:<id>` はチャンネル用

<div id="tool-actions">
  ## ツールアクション
</div>

Slack のツールアクションは `channels.slack.actions.*` で制御できます:

| アクショングループ | デフォルト | 説明 |
| --- | --- | --- |
| reactions | 有効 | リアクションの追加 + 一覧表示 |
| messages | 有効 | 閲覧/送信/編集/削除 |
| pins | 有効 | ピン留め/ピン解除/一覧表示 |
| memberInfo | 有効 | メンバー情報 |
| emojiList | 有効 | カスタム絵文字の一覧 |

<div id="security-notes">
  ## セキュリティに関する注意事項
</div>

* 書き込み操作はデフォルトで bot トークンを使用するため、状態を変更する処理はアプリの bot 権限とアイデンティティの範囲内に制限されます。
* `userTokenReadOnly: false` を設定すると、bot トークンが利用できない場合に書き込み操作にユーザートークンを使用できるようになり、その結果、インストールしたユーザーのアクセス権限でアクションが実行されます。ユーザートークンは高権限トークンとして扱い、アクションのゲートや許可リストは厳格に維持してください。
* ユーザートークンでの書き込みを有効にする場合は、そのユーザートークンに想定している書き込み用のスコープ（`chat:write`、`reactions:write`、`pins:write`、`files:write`）が含まれていることを必ず確認してください。含まれていない場合、これらの操作は失敗します。

<div id="notes">
  ## 注意事項
</div>

* メンションによるゲート制御は `channels.slack.channels`（`requireMention` を `true` に設定）で行われます。`agents.list[].groupChat.mentionPatterns`（または `messages.groupChat.mentionPatterns`）もメンションとして扱われます。
* マルチエージェント向けのオーバーライド: `agents.list[].groupChat.mentionPatterns` にエージェントごとのパターンを設定します。
* リアクション通知は `channels.slack.reactionNotifications` に従います（モードを `allowlist` にして `reactionAllowlist` を使用します）。
* Bot が作成したメッセージはデフォルトで無視されます。有効化するには `channels.slack.allowBots` または `channels.slack.channels.<id>.allowBots` を使用します。
* 警告: 他の Bot への返信を許可する場合（`channels.slack.allowBots=true` または `channels.slack.channels.<id>.allowBots=true`）、`requireMention`、`channels.slack.channels.<id>.users` の許可リスト、および `AGENTS.md` と `SOUL.md` で明確なガードレールを設定することで、Bot 同士の返信ループを防いでください。
* Slack ツールにおけるリアクション削除の挙動については [/tools/reactions](/ja/tools/reactions) を参照してください。
* 添付ファイルは、許可されていてサイズ上限以内であればメディアストアにダウンロードされます。