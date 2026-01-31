---
title: Discord
summary: "Discord ボットのサポート状況、機能、設定"
read_when:
  - Discord チャンネル機能を実装しているとき
---

<div id="discord-bot-api">
  # Discord (Bot API)
</div>

ステータス: 公式の Discord Bot Gateway 経由で、DM およびギルドテキストチャンネルで利用可能です。

<div id="quick-setup-beginner">
  ## かんたんセットアップ（初心者向け）
</div>

1. Discord Bot を作成し、Bot Token をコピーします。
2. Discord アプリの設定で **Message Content Intent** を有効化します（許可リストや名前検索を使う予定がある場合は **Server Members Intent** も有効化してください）。
3. OpenClaw 用にトークンを設定します:
   * 環境変数: `DISCORD_BOT_TOKEN=...`
   * または設定ファイル: `channels.discord.token: "..."`。
   * 両方設定されている場合、設定ファイルが優先されます（環境変数はデフォルトアカウント用のフォールバックとしてのみ使われます）。
4. メッセージ権限を付与したうえで Bot をサーバーに招待します（DM だけを使いたい場合はプライベートサーバーを作成してください）。
5. Gateway を起動します。
6. DM アクセスはデフォルトでペアリングが必要です。最初のやり取り時にペアリングコードを承認してください。

最小構成:

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
  ## 目標
</div>

* Discord の DM またはギルドチャンネル経由で OpenClaw と対話できるようにする。
* ユーザーとのダイレクトメッセージでのチャットはエージェントのメインセッション（デフォルト `agent:main:main`）に集約し、ギルドチャンネルは `agent:<agentId>:discord:channel:<channelId>` として分離して保持する（表示名には `discord:<guildSlug>#<channelSlug>` を使用）。
* グループ DM はデフォルトでは無視する。`channels.discord.dm.groupEnabled` で有効にし、必要に応じて `channels.discord.dm.groupChannels` で制限する。
* ルーティングを決定論的に保つ：返信は常に受信元のチャンネルに戻す。

<div id="how-it-works">
  ## 仕組み
</div>

1. Discord application を作成し → Bot を追加して、必要な intent（DM + ギルドメッセージ + メッセージ内容）を有効化し、Bot トークンを取得します。
2. Bot をサーバーに招待し、利用したい場所でメッセージの読み取り/送信ができるように必要な権限を付与します。
3. OpenClaw を `channels.discord.token`（もしくはフォールバックとして `DISCORD_BOT_TOKEN`）で設定します。
4. Gateway を起動すると、トークンが利用可能で（まず config、なければ env）、かつ `channels.discord.enabled` が `false` でない場合、自動的に Discord チャンネルを開始します。
   * 環境変数を優先したい場合は、`DISCORD_BOT_TOKEN` を設定します（config ブロックは任意）。
5. ダイレクトチャット: 送信先指定に `user:<id>`（または `<@id>` メンション）を使用します。すべてのターンは共有の `main` セッションに格納されます。数値 ID のみの指定はあいまいなため拒否されます。
6. ギルドチャンネル: 送信先指定には `channel:<channelId>` を使用します。メンションはデフォルトで必須であり、ギルド単位またはチャンネル単位で設定できます。
7. ダイレクトチャット: `channels.discord.dm.policy`（デフォルト: `"pairing"`）により、デフォルトでセキュアになります。未知の送信者にはペアリングコードが発行されます（1 時間で失効）。`openclaw pairing approve discord <code>` で承認します。
   * 以前の「誰からでも受け付ける open 挙動」を維持するには: `channels.discord.dm.policy="open"` と `channels.discord.dm.allowFrom=["*"]` を設定します（`open` は、任意のユーザーからのメッセージを無制限に受け付ける設定です）。
   * 厳格な許可リスト方式にするには: `channels.discord.dm.policy="allowlist"` を設定し、`channels.discord.dm.allowFrom` に送信者を列挙します。
   * すべての DM を無視するには: `channels.discord.dm.enabled=false` または `channels.discord.dm.policy="disabled"` を設定します。
8. グループ DM はデフォルトで無視されます。`channels.discord.dm.groupEnabled` で有効化し、必要に応じて `channels.discord.dm.groupChannels` で制限します。
9. オプションのギルドルール: `channels.discord.guilds` をギルド ID（推奨）またはスラッグをキーに設定し、チャンネル単位のルールを定義します。
10. オプションのネイティブコマンド: `commands.native` はデフォルトで `"auto"`（Discord/Telegram ではオン、Slack ではオフ）です。`channels.discord.commands.native: true|false|"auto"` で上書きできます。`false` を指定すると、以前に登録されたコマンドは削除されます。テキストコマンドは `commands.text` によって制御され、単独の `/...` メッセージとして送信する必要があります。コマンドに対するアクセスグループチェックをスキップするには、`commands.useAccessGroups: false` を使用します。
    * コマンドの完全な一覧と設定: [Slash commands](/ja/tools/slash-commands)
11. オプションのギルドコンテキスト履歴: メンションに返信する際、直近 N 件のギルドメッセージをコンテキストとして含めるために `channels.discord.historyLimit`（デフォルト 20、フォールバックは `messages.groupChat.historyLimit`）を設定します。無効化するには `0` を設定します。
12. リアクション: エージェントは `discord` ツールを介してリアクションをトリガーできます（`channels.discord.actions.*` によって制御）。
    * リアクション削除のセマンティクス: [/tools/reactions](/ja/tools/reactions) を参照。
    * `discord` ツールは、現在のチャンネルが Discord の場合にのみ公開されます。
13. ネイティブコマンドは、共有の `main` セッションではなく、分離されたセッションキー（`agent:<agentId>:discord:slash:<userId>`）を使用します。

Note: Name → id 解決はギルドメンバー検索を使用し、Server Members Intent が必要です。Bot がメンバー検索を行えない場合は、id もしくは `<@id>` メンションを使用してください。
Note: スラッグは小文字で、スペースは `-` に置換されます。チャンネル名は先頭の `#` を除いた形でスラッグ化されます。
Note: ギルドコンテキストの `[from:]` 行には、ping しやすい返信を行えるように `author.tag` と `id` が含まれます。

<div id="config-writes">
  ## Config 書き込み
</div>

デフォルトでは、Discord は `/config set|unset` によってトリガーされる設定更新を書き込むことが許可されています（`commands.config: true` が必要です）。

無効化するには次のようにします:

```json5
{
  channels: { discord: { configWrites: false } }
}
```

<div id="how-to-create-your-own-bot">
  ## 独自のボットを作成する方法
</div>

これは、`#help` のようなサーバー（ギルド）チャンネルで OpenClaw を動かすための「Discord Developer Portal」の設定方法です。

<div id="1-create-the-discord-app-bot-user">
  ### 1) Discordアプリとボットユーザーを作成する
</div>

1. Discord Developer Portal → **Applications** → **New Application**
2. 作成したアプリを開き:
   * **Bot** タブ → **Add Bot**
   * **Bot Token** をコピーする（これを `DISCORD_BOT_TOKEN` に設定する）

<div id="2-enable-the-gateway-intents-openclaw-needs">
  ### 2) OpenClaw に必要な Gateway Intent を有効にする
</div>

Discord では、「特権 Intent」は明示的に有効化しない限りブロックされます。

**Bot** → **Privileged Gateway Intents** で、次を有効にします:

* **Message Content Intent**（ほとんどのギルドでメッセージ本文を読み取るのに必須です。これを有効にしないと「Used disallowed intents」と表示されたり、ボットが接続はしてもメッセージに反応しません）
* **Server Members Intent**（推奨。特定のメンバー／ユーザー検索や、ギルドにおける許可リスト照合に必要です）

通常は **Presence Intent** を有効にする必要は **ありません**。

<div id="3-generate-an-invite-url-oauth2-url-generator">
  ### 3) 招待URLを生成する（OAuth2 URL Generator）
</div>

アプリ内で：**OAuth2** → **URL Generator**

**スコープ（Scopes）**

* ✅ `bot`
* ✅ `applications.commands`（ネイティブコマンドに必須）

**Bot Permissions**（最小限のベースライン）

* ✅ View Channels（チャンネルを表示）
* ✅ Send Messages（メッセージを送信）
* ✅ Read Message History（メッセージ履歴を閲覧）
* ✅ Embed Links（リンクを埋め込む）
* ✅ Attach Files（ファイルを添付）
* ✅ Add Reactions（リアクションを追加）（任意だが推奨）
* ✅ Use External Emojis / Stickers（外部絵文字 / スタンプを使用）（任意・必要な場合のみ）

**Administrator** は、デバッグ中であり、かつそのボットを完全に信頼している場合を除き、付与しないでください。

生成されたURLをコピーして開き、サーバーを選択してボットをインストールします。

<div id="4-get-the-ids-guilduserchannel">
  ### 4) ID を取得する (guild/user/channel)
</div>

Discord はあらゆる箇所で数値の ID を使用します。OpenClaw の設定では ID 指定を推奨します。

1. Discord (デスクトップ/ウェブ) → **ユーザー設定** → **詳細設定** → **開発者モード** を有効化
2. 右クリック:
   * サーバー名 → **サーバーIDをコピー** (guild ID)
   * チャンネル (例: `#help`) → **チャンネルIDをコピー**
   * 自分のユーザー → **ユーザーIDをコピー**

<div id="5-configure-openclaw">
  ### 5) OpenClaw を構成する
</div>

<div id="token">
  #### トークン
</div>

サーバー環境では、環境変数でボットトークンを設定することを推奨します:

* `DISCORD_BOT_TOKEN=...`

または、設定ファイルで指定します:

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

マルチアカウントサポート: `channels.discord.accounts` を使用して、アカウントごとのトークンおよび任意指定の `name` を設定します。共通のパターンについては [`gateway/configuration`](/ja/gateway/configuration#telegramaccounts--discordaccounts--slackaccounts--signalaccounts--imessageaccounts) を参照してください。

<div id="allowlist-channel-routing">
  #### 許可リスト + チャンネルルーティング
</div>

例「単一サーバー、自分のみ許可、#help のみ許可」の場合:

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

注意事項:

* `requireMention: true` は、ボットがメンションされたときにのみ返信することを意味します（共有チャンネルでは推奨）。
* `agents.list[].groupChat.mentionPatterns`（または `messages.groupChat.mentionPatterns`）も、ギルドメッセージに対するメンションとして扱われます。
* マルチエージェントでの上書き設定: エージェントごとのパターンを `agents.list[].groupChat.mentionPatterns` に設定します。
* `channels` が存在する場合、列挙されていないチャンネルはデフォルトで拒否されます。
* `"*"` チャンネルエントリを使用して、すべてのチャンネルに対してデフォルト設定を適用できます。明示的なチャンネルエントリはワイルドカードより優先されます。
* スレッドは、スレッドのチャンネル ID を明示的に追加しない限り、親チャンネルの設定（許可リスト、`requireMention`、スキル、プロンプトなど）を継承します。
* ボットが作成したメッセージはデフォルトで無視されます。これらを許可するには `channels.discord.allowBots=true` を設定します（自身のメッセージは引き続きフィルタされます）。
* 警告: 他のボットへの返信を許可する場合（`channels.discord.allowBots=true`）、`requireMention`、`channels.discord.guilds.*.channels.<id>.users` の許可リスト、そして `AGENTS.md` と `SOUL.md` における明確なガードレールにより、ボット同士の返信ループを防止してください。

<div id="6-verify-it-works">
  ### 6) 動作確認をする
</div>

1. Gateway を起動します。
2. サーバーのチャンネルで、次のメッセージを送信します: `@Krill hello` （または自分の Bot 名）。
3. 何も起こらない場合は、以下の **Troubleshooting** を確認してください。

<div id="troubleshooting">
  ### トラブルシューティング
</div>

* まずは `openclaw doctor` と `openclaw channels status --probe` を実行してください（対応可能な警告 + 簡易診断）。
* **“Used disallowed intents”**: Developer Portal で **Message Content Intent**（おそらく **Server Members Intent** も）を有効化し、その後 Gateway を再起動してください。
* **Bot がギルドチャンネルに接続しても返信しない場合**:
  * **Message Content Intent** が有効になっていない、または
  * Bot にチャンネル権限（View/Send/Read History）がない、または
  * 設定でメンションが必須になっているのにメンションしていない、または
  * ギルド/チャンネルの許可リストがそのチャンネル/ユーザーを拒否している可能性があります。
* **`requireMention: false` なのに返信がない場合**:
* `channels.discord.groupPolicy` はデフォルトで **allowlist** です。`"open"` に設定するか、`channels.discord.guilds` 配下にギルドエントリを追加します（必要に応じて `channels.discord.guilds.<id>.channels` にチャンネルを列挙して制限できます）。
  * `DISCORD_BOT_TOKEN` だけを設定し、`channels.discord` セクションを作成していない場合、ランタイムは
    `groupPolicy` を `open` にデフォルト設定します。ロックダウンするには `channels.discord.groupPolicy`、
    `channels.defaults.groupPolicy`、またはギルド/チャンネルの許可リストを追加してください。
* `requireMention` は `channels.discord.guilds`（または特定チャンネル）配下に配置する必要があります。トップレベルの `channels.discord.requireMention` は無視されます。
* **権限監査**（`channels status --probe`）は数値チャンネル ID のみをチェックします。`channels.discord.guilds.*.channels` のキーとしてスラッグ名/チャンネル名を使用している場合、監査では権限を検証できません。
* **DM が動作しない場合**: `channels.discord.dm.enabled=false`、`channels.discord.dm.policy="disabled"`、またはまだ承認されていません（`channels.discord.dm.policy="pairing"` の場合）。

<div id="capabilities-limits">
  ## 機能と制限
</div>

* DM とギルドのテキストチャンネル（スレッドは別チャンネルとして扱われる。ボイスは未対応）。
* 入力中インジケーターはベストエフォートで送信される。メッセージ分割には `channels.discord.textChunkLimit`（デフォルト 2000）を使用し、長い返信は行数（`channels.discord.maxLinesPerMessage`、デフォルト 17）で分割する。
* オプションの改行チャンク処理: 長さによるチャンク処理の前に、空行（段落境界）で分割するには `channels.discord.chunkMode="newline"` を設定する。
* ファイルアップロードは、設定された `channels.discord.mediaMaxMb`（デフォルト 8 MB）までサポート。
* うるさいボットを避けるため、デフォルトではメンション付きのギルド返信のみ送信される。
* メッセージが別のメッセージを参照している場合、返信コンテキストが付与される（引用コンテンツ + id）。
* ネイティブの返信スレッド機能は**デフォルトでオフ**。`channels.discord.replyToMode` と返信タグを使用して有効化する。

<div id="retry-policy">
  ## リトライポリシー
</div>

送信側の Discord API 呼び出しは、Discord の `retry_after` が利用可能な場合にはそれを使用しつつ、レート制限 (429) に達した場合に指数バックオフとジッターを伴ってリトライします。`channels.discord.retry` で設定できます。詳細は [リトライポリシー](/ja/concepts/retry) を参照してください。

<div id="config">
  ## 設定
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
        policy: "pairing", // ペアリング | 許可リスト | オープン | 無効
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

Ackリアクションは、グローバルに `messages.ackReaction` と
`messages.ackReactionScope` で制御されます。`messages.removeAckAfterReply` を使用すると、ボットが返信した後に
Ackリアクションをクリアできます。

* `dm.enabled`: すべてのDMを無視する場合は `false` を設定します（デフォルトは `true`）。
* `dm.policy`: DMアクセス制御（`pairing` を推奨）。`"open"` を使う場合は `dm.allowFrom=["*"]` が必要です。
* `dm.allowFrom`: DMの許可リスト（ユーザーIDまたは名前）。`dm.policy="allowlist"` のとき、または `dm.policy="open"` の検証時に使用されます。ウィザードはユーザー名を受け付け、Botがメンバー検索可能な場合はIDに解決します。
* `dm.groupEnabled`: グループDMを有効にします（デフォルトは `false`）。
* `dm.groupChannels`: グループDM用のチャンネルIDまたはスラッグに対する任意の許可リスト。
* `groupPolicy`: ギルドチャンネルの扱いを制御します（`open|disabled|allowlist`）。`allowlist` の場合はチャンネルの許可リストが必要です。
* `guilds`: ギルドID（推奨）またはスラッグをキーにしたギルド単位のルール。
* `guilds."*"`: 明示的なエントリが存在しない場合に適用される、デフォルトのギルド単位設定。
* `guilds.<id>.slug`: 表示名として使われる任意のフレンドリーなスラッグ。
* `guilds.<id>.users`: 任意のギルド単位ユーザー許可リスト（IDまたは名前）。
* `guilds.<id>.tools`: 任意のギルド単位ツールポリシーの上書き（`allow`/`deny`/`alsoAllow`）。チャンネル側の上書きが存在しない場合に使用されます。
* `guilds.<id>.toolsBySender`: 任意の送信者単位ツールポリシー上書き（ギルドレベル）。チャンネル側の上書きが存在しない場合に適用されます（`"*"` ワイルドカード対応）。
* `guilds.<id>.channels.<channel>.allow`: `groupPolicy="allowlist"` のとき、このチャンネルを許可/拒否します。
* `guilds.<id>.channels.<channel>.requireMention`: チャンネルに対するメンション必須ゲート。
* `guilds.<id>.channels.<channel>.tools`: 任意のチャンネル単位ツールポリシー上書き（`allow`/`deny`/`alsoAllow`）。
* `guilds.<id>.channels.<channel>.toolsBySender`: チャンネル内の送信者単位ツールポリシー上書き（`"*"` ワイルドカード対応）。
* `guilds.<id>.channels.<channel>.users`: 任意のチャンネル単位ユーザー許可リスト。
* `guilds.<id>.channels.<channel>.skills`: スキルフィルタ（省略 = すべてのスキル、空 = なし）。
* `guilds.<id>.channels.<channel>.systemPrompt`: チャンネル用の追加システムプロンプト（チャンネルトピックと結合されます）。
* `guilds.<id>.channels.<channel>.enabled`: チャンネルを無効化する場合は `false` を設定します。
* `guilds.<id>.channels`: チャンネルルール（キーはチャンネルスラッグまたはID）。
* `guilds.<id>.requireMention`: ギルド単位のメンション必須設定（チャンネル単位で上書き可能）。
* `guilds.<id>.reactionNotifications`: リアクション関連システムイベントのモード（`off`, `own`, `all`, `allowlist`）。
* `textChunkLimit`: 送信テキストチャンクサイズ（文字数）。デフォルト: 2000。
* `chunkMode`: `length`（デフォルト）は `textChunkLimit` を超えた場合のみ分割します。`newline` は長さによるチャンク分割の前に、空行（段落境界）で分割します。
* `maxLinesPerMessage`: メッセージごとの行数のソフト上限。デフォルト: 17。
* `mediaMaxMb`: ディスクに保存する受信メディアのサイズ上限を設定します。
* `historyLimit`: メンションに返信する際にコンテキストとして含める直近ギルドメッセージ数（デフォルト20。`messages.groupChat.historyLimit` へフォールバックします。`0` で無効）。
* `dmHistoryLimit`: DMの履歴上限（ユーザーターン数）。ユーザー単位の上書き: `dms["<user_id>"].historyLimit`。
* `retry`: Discord APIへの送信リトライポリシー（attempts, minDelayMs, maxDelayMs, jitter）。
* `actions`: アクションごとのツールゲート。省略すると全許可（`false` を設定すると無効化）。
  * `reactions`（リアクション + リアクションの read を含む）
  * `stickers`, `emojiUploads`, `stickerUploads`, `polls`, `permissions`, `messages`, `threads`, `pins`, `search`
  * `memberInfo`, `roleInfo`, `channelInfo`, `voiceStatus`, `events`
  * `channels`（チャンネル + カテゴリ + 権限の作成/編集/削除）
  * `roles`（ロールの追加/削除。デフォルト `false`）
  * `moderation`（timeout/kick/ban。デフォルト `false`）

リアクション通知は `guilds.<id>.reactionNotifications` を使用します:

* `off`: リアクションイベントなし。
* `own`: Bot自身のメッセージへのリアクションのみ（デフォルト）。
* `all`: すべてのメッセージ上のすべてのリアクション。
* `allowlist`: すべてのメッセージ上で、`guilds.<id>.users` からのリアクション（リストが空の場合は無効）。

<div id="tool-action-defaults">
  ### ツールアクションのデフォルト
</div>

| アクショングループ | デフォルト | メモ |
| --- | --- | --- |
| reactions | enabled | リアクション＋リアクション一覧＋emojiList |
| stickers | enabled | スタンプを送信 |
| emojiUploads | enabled | 絵文字をアップロード |
| stickerUploads | enabled | スタンプをアップロード |
| polls | enabled | 投票を作成 |
| permissions | enabled | チャンネル権限のスナップショット |
| messages | enabled | 読み取り/送信/編集/削除 |
| threads | enabled | 作成/一覧/返信 |
| pins | enabled | ピン留め/ピン解除/一覧 |
| search | enabled | メッセージ検索（プレビュー機能） |
| memberInfo | enabled | メンバー情報 |
| roleInfo | enabled | ロール一覧 |
| channelInfo | enabled | チャンネル情報＋一覧 |
| channels | enabled | チャンネル/カテゴリ管理 |
| voiceStatus | enabled | ボイス状態の参照 |
| events | enabled | スケジュールイベントの一覧/作成 |
| roles | disabled | ロールの追加/削除 |
| moderation | disabled | タイムアウト/キック/BAN |

* `replyToMode`: `off`（デフォルト）、`first`、`all`。モデルがリプライタグを含む場合にのみ適用されます。

<div id="reply-tags">
  ## 返信タグ
</div>

スレッド形式の返信を要求するには、モデルは出力に次のいずれかのタグを含めることができます:

* `[[reply_to_current]]` — トリガーとなった Discord メッセージへの返信。
* `[[reply_to:<id>]]` — コンテキスト/履歴内の特定のメッセージ ID への返信。

現在のメッセージ ID はプロンプトに `[message_id: …]` として付加されます。履歴エントリにはすでに ID が含まれています。

動作は `channels.discord.replyToMode` で制御されます:

* `off`: タグを無視する。
* `first`: 最初の送信チャンク／添付ファイルのみ返信にする。
* `all`: すべての送信チャンク／添付ファイルを返信にする。

許可リストのマッチングに関する注意事項:

* `allowFrom`/`users`/`groupChannels` は ID、名前、タグ、`<@id>` のようなメンションを受け付けます。
* `discord:`/`user:`（ユーザー）や `channel:`（グループ DM）といったプレフィックスがサポートされています。
* 任意の送信者／チャンネルを許可するには `*` を使用します。
* `guilds.<id>.channels` が存在する場合、列挙されていないチャンネルはデフォルトで拒否されます。
* `guilds.<id>.channels` が省略されている場合、許可リストに登録された guild 内のすべてのチャンネルが許可されます。
* **一切のチャンネルを許可しない** 場合は、`channels.discord.groupPolicy: "disabled"` を設定します（または空の許可リストのままにします）。
* 設定ウィザードは（公開／非公開を含む）`Guild/Channel` 名を受け付け、可能な場合はそれらを ID に解決します。
* 起動時、OpenClaw は許可リスト内のチャンネル／ユーザー名を ID に解決し（ボットがメンバー検索可能な場合）、そのマッピングをログに出力します。解決できなかったエントリは入力された文字列のまま保持されます。

ネイティブコマンドに関する注意事項:

* 登録されるコマンドは OpenClaw のチャットコマンドを反映します。
* ネイティブコマンドは DM/guild メッセージ（`channels.discord.dm.allowFrom`、`channels.discord.guilds`、チャンネル単位のルール）と同じ許可リストに従います。
* Slash コマンドは、許可リストに含まれていないユーザーにも Discord の UI 上で表示される場合があります。OpenClaw は実行時に許可リストを適用し、`not authorized` と返信します。

<div id="tool-actions">
  ## ツールアクション
</div>

エージェントは、次のようなアクションで `discord` を呼び出せます:

* `react` / `reactions` (リアクションを追加または一覧表示)
* `sticker`, `poll`, `permissions`
* `readMessages`, `sendMessage`, `editMessage`, `deleteMessage`
* 読み取り／検索／ピン系ツールのペイロードには、正規化された `timestampMs`（UTC エポック ms）と `timestampUtc` が、生の Discord `timestamp` とあわせて含まれます。
* `threadCreate`, `threadList`, `threadReply`
* `pinMessage`, `unpinMessage`, `listPins`
* `searchMessages`, `memberInfo`, `roleInfo`, `roleAdd`, `roleRemove`, `emojiList`
* `channelInfo`, `channelList`, `voiceStatus`, `eventList`, `eventCreate`
* `timeout`, `kick`, `ban`

Discord メッセージ ID は、挿入されるコンテキスト内（`[discord message id: …]` や履歴行）に含まれるため、エージェントはそれらを対象として操作できます。
絵文字は、Unicode（例: `✅`）でも、`<:party_blob:1234567890>` のようなカスタム絵文字構文でもかまいません。

<div id="safety-ops">
  ## セキュリティと運用
</div>

* ボットトークンはパスワードと同様に扱い、管理下のホストでは `DISCORD_BOT_TOKEN` 環境変数の利用を優先するか、設定ファイルのパーミッションを厳しく制限してください。
* ボットには必要な権限だけを付与してください（通常は「Read Messages」「Send Messages」）。
* ボットがハングしたりレート制限を受けた場合、ほかのプロセスが Discord セッションを保持していないことを確認したうえで、Gateway を再起動してください（`openclaw gateway --force`）。