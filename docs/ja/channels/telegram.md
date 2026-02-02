---
title: Telegram
summary: "Telegram ボットのサポート状況、機能、および設定"
read_when:
  - Telegram 機能や Webhook の作業をしているとき
---

<div id="telegram-bot-api">
  # Telegram (Bot API)
</div>

ステータス: grammY 経由のボット DM およびグループ向けに本番運用可能な状態。デフォルトはロングポーリングで、Webhook はオプション。

<div id="quick-setup-beginner">
  ## クイックセットアップ（初心者向け）
</div>

1. **@BotFather** を使ってボットを作成し、トークンをコピーします。
2. トークンを設定します:
   * 環境変数: `TELEGRAM_BOT_TOKEN=...`
   * または設定ファイル: `channels.telegram.botToken: "..."`。
   * 両方が設定されている場合は、設定ファイルが優先されます（環境変数はデフォルトアカウントに対するフォールバックとしてのみ使用されます）。
3. Gateway を起動します。
4. DM アクセスはデフォルトでペアリング方式です。最初のやり取り時にペアリングコードを承認してください。

最小限の設定:

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
  ## 概要
</div>

* Gateway が所有する Telegram Bot API チャンネルです。
* 決定論的ルーティング：返信は常に Telegram に戻り、モデルがチャネルを選択することはありません。
* DM（ダイレクトメッセージ）はエージェントのメインセッションを共有し、グループは個別に分離されたままになります（`agent:<agentId>:telegram:group:<chatId>`）。

<div id="setup-fast-path">
  ## セットアップ（クイック手順）
</div>

<div id="1-create-a-bot-token-botfather">
  ### 1) ボットトークンを作成する（BotFather）
</div>

1. Telegram を開き、**@BotFather** とチャットを開始します。
2. `/newbot` を実行し、指示に従って（名前 + `bot` で終わるユーザー名）を設定します。
3. 発行されたトークンをコピーして、安全な場所に保管します。

任意で設定できる BotFather オプション:

* `/setjoingroups` — ボットをグループに追加できるかどうかを許可／拒否します。
* `/setprivacy` — ボットがグループ内のすべてのメッセージを取得できるかどうかを制御します。

<div id="2-configure-the-token-env-or-config">
  ### 2) トークンを設定する（環境変数または設定）
</div>

例:

```json5
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "123:abc",
      dmPolicy: "pairing",
      groups: { "*": { requireMention: true } }
    }
  }
}
```

Env オプション: `TELEGRAM_BOT_TOKEN=...`（デフォルトアカウント用として動作します）。
env と config の両方が設定されている場合は、config が優先されます。

マルチアカウント対応: `channels.telegram.accounts` を使用し、アカウントごとのトークンと任意の `name` を設定します。共通パターンについては [`gateway/configuration`](/ja/gateway/configuration#telegramaccounts--discordaccounts--slackaccounts--signalaccounts--imessageaccounts) を参照してください。

3. Gateway を起動します。Telegram はトークンが解決され次第起動します（config が優先され、env がフォールバックになります）。
4. DM アクセスはデフォルトでペアリングが有効です。ボットに最初に連絡したときにコードを承認します。
5. グループの場合: ボットを追加し、プライバシー / 管理者としての挙動（下記）を決めてから、`channels.telegram.groups` を設定してメンションゲートと許可リストを制御します。

<div id="token-privacy-permissions-telegram-side">
  ## トークン、プライバシー、権限（Telegram 側）
</div>

<div id="token-creation-botfather">
  ### トークンの作成 (BotFather)
</div>

* `/newbot` はボットを作成し、トークンを返します（トークンは外部に漏らさないよう厳重に管理してください）。
* トークンが漏えいした場合は、@BotFather で失効/再生成し、設定を更新してください。

<div id="group-message-visibility-privacy-mode">
  ### グループメッセージの可視性（プライバシーモード）
</div>

Telegram ボットはデフォルトで**プライバシーモード**になっており、受信できるグループメッセージが制限されます。
ボットにグループ内の*すべての*メッセージを受信させる必要がある場合は、次のいずれかを実行します:

* `/setprivacy` を使ってプライバシーモードを無効にする **または**
* ボットをグループの**管理者**として追加する（管理者ボットはすべてのメッセージを受信します）。

**注意:** プライバシーモードを切り替えた場合、その変更を反映させるには、
各グループから一度ボットを削除し、再度追加する必要があります。

<div id="group-permissions-admin-rights">
  ### グループ権限（管理者権限）
</div>

管理者ステータスはグループ内（Telegram の UI）で設定します。ボットを管理者に設定すると、そのボットは常にすべてのグループメッセージを受信するため、グループ内のやり取りをすべて把握する必要がある場合は管理者にしてください。

<div id="how-it-works-behavior">
  ## 仕組み（動作）
</div>

* 受信メッセージは、返信コンテキストとメディア用プレースホルダー付きで、共有チャネルエンベロープに正規化されます。
* グループへの返信には、デフォルトでメンションが必要です（ネイティブの @mention または `agents.list[].groupChat.mentionPatterns` / `messages.groupChat.mentionPatterns`）。
* マルチエージェント構成時の上書き設定: `agents.list[].groupChat.mentionPatterns` でエージェントごとのパターンを設定します。
* 返信は常に同じ Telegram チャットにルーティングされます。
* ロングポーリングは、チャットごとのシーケンス管理付きで grammY runner を使用します。全体の同時実行数は `agents.defaults.maxConcurrent` によって制限されます。
* Telegram Bot API は既読通知をサポートしていないため、`sendReadReceipts` オプションは存在しません。

<div id="formatting-telegram-html">
  ## 書式設定（Telegram HTML）
</div>

* 送信される Telegram テキストは `parse_mode: "HTML"`（Telegram がサポートするタグのサブセット）を使用します。
* Markdown 風の入力は **Telegram で安全に扱える HTML**（太字／斜体／取り消し線／コード／リンク）に変換され、ブロック要素は改行や箇条書きを伴うテキストへフラットに変換されます。
* モデルからの生の HTML は、Telegram のパースエラーを避けるためにエスケープされます。
* Telegram が HTML ペイロードを拒否した場合、OpenClaw は同じメッセージをプレーンテキストとして再送信します。

<div id="commands-native-custom">
  ## コマンド（ネイティブ + カスタム）
</div>

OpenClaw は起動時に、Telegram のボットメニューにネイティブコマンド（`/status`、`/reset`、`/model` など）を登録します。
設定でメニューにカスタムコマンドを追加できます。

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
  ## トラブルシューティング
</div>

* ログに `setMyCommands failed` と出る場合、ほとんどは `api.telegram.org` へのアウトバウンド HTTPS/DNS 通信がブロックされていることを意味します。
* `sendMessage` や `sendChatAction` が失敗する場合は、IPv6 ルーティングと DNS を確認してください。

詳しくは: [チャンネルのトラブルシューティング](/ja/channels/troubleshooting).

注意事項:

* カスタムコマンドは **メニュー項目としてのみ** 扱われます。OpenClaw 自体は、別の場所で処理しない限り、それらを解釈・処理しません。
* コマンド名は正規化されます（先頭の `/` は除去され、小文字に変換されます）。そのうえで、`a-z`、`0-9`、`_`（1〜32 文字）に一致している必要があります。
* カスタムコマンドは **ネイティブコマンドを上書きできません**。競合は無視され、ログに記録されます。
* `commands.native` が無効化されている場合、カスタムコマンドのみが登録されます（カスタムコマンドがなければ登録はすべてクリアされます）。

<div id="limits">
  ## 制限事項
</div>

* 送信テキストは `channels.telegram.textChunkLimit`（デフォルト 4000）ごとにチャンク分割されます。
* 改行単位での任意チャンク分割: 長さによるチャンク分割の前に空行（段落境界）で分割するには、`channels.telegram.chunkMode="newline"` を設定します。
* メディアのダウンロード／アップロードは `channels.telegram.mediaMaxMb`（デフォルト 5）で上限が決まります。
* Telegram Bot API リクエストは `channels.telegram.timeoutSeconds`（grammY 経由のデフォルト 500）でタイムアウトします。長時間ハングするのを避けるには、より小さい値に設定してください。
* グループ履歴コンテキストは `channels.telegram.historyLimit`（または `channels.telegram.accounts.*.historyLimit`）を使用し、指定がなければ `messages.groupChat.historyLimit` にフォールバックします。`0` を設定すると無効化されます（デフォルト 50）。
* DM 履歴は `channels.telegram.dmHistoryLimit`（ユーザーターン数）で制限できます。ユーザーごとの上書き設定: `channels.telegram.dms["<user_id>"].historyLimit`。

<div id="group-activation-modes">
  ## グループ有効化モード
</div>

デフォルトでは、ボットはグループ内でのメンション（`@botname` または `agents.list[].groupChat.mentionPatterns` 内のパターン）にのみ応答します。この挙動を変更するには、以下の設定を行います。

<div id="via-config-recommended">
  ### 設定から（推奨）
</div>

```json5
{
  channels: {
    telegram: {
      groups: {
        "-1001234567890": { requireMention: false }  // このグループでは常に応答
      }
    }
  }
}
```

**重要:** `channels.telegram.groups` を設定すると**許可リスト**として機能し、列挙されたグループ（または `"*"`）のみが受け付けられます。
フォーラムトピックは、`channels.telegram.groups.<groupId>.topics.<topicId>` 以下でトピックごとのオーバーライドを追加しない限り、親グループの設定（allowFrom, requireMention, スキル, プロンプト）を継承します。

常に応答する設定で全グループを許可するには:

```json5
{
  channels: {
    telegram: {
      groups: {
        "*": { requireMention: false }  // 全グループ、常に応答
      }
    }
  }
}
```

すべてのグループをメンション専用モードのままにしておく場合（デフォルトの動作）:

```json5
{
  channels: {
    telegram: {
      groups: {
        "*": { requireMention: true }  // または groups を完全に省略する
      }
    }
  }
}
```

<div id="via-command-session-level">
  ### コマンド経由（セッションレベル）
</div>

グループ内で次を送信します:

* `/activation always` - すべてのメッセージに応答する
* `/activation mention` - メンションされた場合のみ応答する（デフォルト）

**注意:** コマンドはセッションの状態のみを更新します。再起動後も同じ動作を維持したい場合は、設定を使用してください。

<div id="getting-the-group-chat-id">
  ### グループチャット ID の取得
</div>

グループ内の任意のメッセージを Telegram 上の `@userinfobot` または `@getidsbot` に転送すると、チャット ID（`-1001234567890` のような負の数）が表示されます。

**ヒント:** 自分のユーザー ID が必要な場合は、そのボットに DM を送ると、ユーザー ID（ペアリング用メッセージ）で返信されます。あるいは、コマンドが有効になってから `/whoami` を使用してください。

**プライバシーに関する注意:** `@userinfobot` はサードパーティ製ボットです。プライバシーが気になる場合は、そのボットをグループに追加してメッセージを送り、`openclaw logs --follow` を使って `chat.id` を `read` するか、Bot API の `getUpdates` を使用してください。

<div id="config-writes">
  ## Config の書き込み
</div>

デフォルトでは、チャネルイベントや `/config set|unset` によってトリガーされた config の更新を書き込むことが Telegram に許可されています。

この動作は次の場合に発生します:

* グループがスーパーグループにアップグレードされ、Telegram が `migrate_to_chat_id`（チャット ID が変更される）を送出したとき。OpenClaw は `channels.telegram.groups` を自動的に移行できます。
* Telegram チャット内で `/config set` または `/config unset` を実行したとき（`commands.config: true` が必要）。

無効化するには:

```json5
{
  channels: { telegram: { configWrites: false } }
}
```

<div id="topics-forum-supergroups">
  ## トピック（フォーラムスーパーグループ）
</div>

Telegram フォーラムのトピックでは、各メッセージに `message_thread_id` が含まれます。OpenClaw は次のように動作します:

* 各トピックを分離するために、Telegram グループのセッションキーに `:topic:<threadId>` を付加します。
* レスポンスがトピック内にとどまるように、`message_thread_id` を付けて入力中インジケーターと返信を送信します。
* 一般トピック（スレッド ID `1`）は特別扱いです: メッセージ送信では `message_thread_id` を省略します（Telegram が拒否するため）一方で、入力中インジケーターには引き続き含めます。
* ルーティングやテンプレート用に、テンプレートコンテキストで `MessageThreadId` と `IsForum` を提供します。
* トピック単位の設定は `channels.telegram.groups.<chatId>.topics.<threadId>` の下で利用可能です（スキル、許可リスト、自動返信、システムプロンプト、無効化）。
* トピック設定は、トピックごとに上書きされない限り、グループ設定（requireMention、許可リスト、スキル、プロンプト、enabled）を継承します。

一部の例外的なケースでは、プライベートチャットにも `message_thread_id` が含まれることがあります。OpenClaw は DM のセッションキーは変更しませんが、存在する場合は返信や下書きストリーミングにそのスレッド ID を使用します。

<div id="inline-buttons">
  ## インラインボタン
</div>

Telegram では、コールバックボタン付きのインラインキーボードを利用できます。

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

アカウントごとの設定の場合:

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

スコープ:

* `off` — インラインボタンは無効
* `dm` — DM のみ（グループ宛はブロック）
* `group` — グループのみ（DM 宛はブロック）
* `all` — DM + グループ
* `allowlist` — DM + グループだが、`allowFrom`/`groupAllowFrom` で許可された送信者のみ（制御コマンドと同じルール）

デフォルト: `allowlist`。
レガシー: `capabilities: ["inlineButtons"]` = `inlineButtons: "all"`。

<div id="sending-buttons">
  ### ボタンの送信
</div>

`buttons` パラメーターを指定して message ツールを使用します：

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

ユーザーがボタンをクリックすると、コールバックデータは次の形式のメッセージとしてエージェントに送り返されます：
`callback_data: value`

<div id="configuration-options">
  ### 設定オプション
</div>

Telegram の機能は 2 つのレベルで設定できます（上記のオブジェクト形式のほか、従来の文字列配列も引き続きサポートされています）:

* `channels.telegram.capabilities`: すべての Telegram アカウントに適用されるグローバルなデフォルト機能設定。アカウント側での上書きがない場合に適用されます。
* `channels.telegram.accounts.<account>.capabilities`: 特定のアカウントについて、グローバルデフォルトを上書きするアカウント単位の機能設定。

すべての Telegram ボット／アカウントに同じ動作をさせたい場合はグローバル設定を使用してください。ボットごとに異なる動作が必要な場合（例: あるアカウントは DM のみを処理し、別のアカウントはグループでの利用を許可するなど）は、アカウント単位の設定を使用してください。

<div id="access-control-dms-groups">
  ## アクセス制御（DM およびグループ）
</div>

<div id="dm-access">
  ### DM アクセス
</div>

* デフォルト: `channels.telegram.dmPolicy = "pairing"`。未知の送信者にはペアリングコードが送信され、承認されるまでメッセージは無視されます（コードは 1 時間後に失効します）。
* 承認方法:
  * `openclaw pairing list telegram`
  * `openclaw pairing approve telegram <CODE>`
* ペアリングは、Telegram の DM に対して使用されるデフォルトのトークン交換方式です。詳細は [Pairing](/ja/start/pairing) を参照してください。
* `channels.telegram.allowFrom` は数値のユーザー ID（推奨）または `@username` エントリーを受け付けます。これはボットのユーザー名では **ありません**。人間の送信者の ID を使用してください。ウィザードは `@username` を受け付け、可能な場合は数値 ID に解決して変換します。

<div id="finding-your-telegram-user-id">
  #### Telegram のユーザー ID を確認する
</div>

より安全な方法（サードパーティ製 bot 不要）:

1. Gateway を起動し、自分の bot に DM します。
2. `openclaw logs --follow` を実行し、`from.id` を確認します。

別案（公式 Bot API を使用）:

1. 自分の bot に DM します。
2. bot トークンを使って update を取得し、`message.from.id` を確認します:
   ```bash
   curl "https://api.telegram.org/bot<bot_token>/getUpdates"
   ```

サードパーティ利用（プライバシー保護の面で劣る）:

* `@userinfobot` または `@getidsbot` に DM し、返ってきたユーザー ID を使用します。

<div id="group-access">
  ### グループアクセス
</div>

2 つの独立した制御があります:

**1. どのグループを許可するか**（`channels.telegram.groups` によるグループ許可リスト）:

* `groups` 設定なし = すべてのグループを許可
* `groups` 設定あり = 設定内で列挙されたグループ、または `"*"` のみ許可
* 例: `"groups": { "-1001234567890": {}, "*": {} }` はすべてのグループを許可

**2. どの送信者を許可するか**（`channels.telegram.groupPolicy` による送信者フィルタリング）:

* `"open"` = 許可されたグループ内のすべての送信者からメッセージを受け付ける（許可されたグループ内であれば、誰からでも制限なくメッセージを受け付ける設定）
* `"allowlist"` = `channels.telegram.groupAllowFrom` に含まれる送信者のみメッセージを送信可能
* `"disabled"` = グループメッセージは一切受け付けない\
  デフォルトは `groupPolicy: "allowlist"`（`groupAllowFrom` を追加しない限りブロック）です。

ほとんどのユーザーに推奨される構成: `groupPolicy: "allowlist"` + `groupAllowFrom` + `channels.telegram.groups` に特定のグループを列挙

<div id="long-polling-vs-webhook">
  ## ロングポーリング vs Webhook
</div>

* デフォルト: ロングポーリング（パブリック URL は不要）。
* Webhook モード: `channels.telegram.webhookUrl` を設定（必要に応じて `channels.telegram.webhookSecret` と `channels.telegram.webhookPath` も設定）。
  * ローカルリスナーは `0.0.0.0:8787` で待ち受け、デフォルトで `POST /telegram-webhook` を受け付けます。
  * パブリック URL が異なる場合は、リバースプロキシを使用し、`channels.telegram.webhookUrl` をパブリックエンドポイントに指定してください。

<div id="reply-threading">
  ## 返信スレッド
</div>

Telegram はタグによるオプションのスレッド返信をサポートしています：

* `[[reply_to_current]]` -- トリガー元のメッセージに返信します。
* `[[reply_to:<id>]]` -- 特定のメッセージ ID に返信します。

`channels.telegram.replyToMode` で制御します：

* `first`（デフォルト）, `all`, `off`。

<div id="audio-messages-voice-vs-file">
  ## 音声メッセージ（ボイスメッセージ vs ファイル）
</div>

Telegram は **ボイスメッセージ（voice note）**（丸い吹き出し）と **音声ファイル**（メタデータカード）を区別します。
OpenClaw は後方互換性のため、デフォルトでは音声ファイルとして送信します。

エージェントの返信を強制的にボイスメッセージの吹き出しにしたい場合は、返信内のどこかに次のタグを含めてください:

* `[[audio_as_voice]]` — 音声をファイルではなくボイスメッセージとして送信します。

このタグは配信されるテキストからは削除されます。他のチャンネルはこのタグを無視します。

メッセージツールで送信する場合は、ボイスメッセージ対応の音声用 `media` URL に対して `asVoice: true` を設定します
（`media` がある場合、`message` は省略可能です）:

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
  ## ステッカー
</div>

OpenClaw は、インテリジェントなキャッシュ機構を用いて Telegram ステッカーの受信・送信をサポートします。

<div id="receiving-stickers">
  ### ステッカーの受信
</div>

ユーザーがステッカーを送信した場合、OpenClaw はステッカーの種類に応じて次のように処理します:

* **静的ステッカー (WEBP):** ダウンロードされ、vision 機能で処理されます。メッセージ本文内では `<media:sticker>` プレースホルダーとして表示されます。
* **アニメーションステッカー (TGS):** スキップされます（Lottie 形式は処理に対応していません）。
* **ビデオステッカー (WEBM):** スキップされます（このビデオ形式は処理に対応していません）。

ステッカー受信時に利用可能なテンプレートコンテキストフィールド:

* `Sticker` — 次のフィールドを持つオブジェクト:
  * `emoji` — ステッカーに紐づく絵文字
  * `setName` — ステッカーセットの名前
  * `fileId` — Telegram ファイル ID（同じステッカーを送り返す際に使用）
  * `fileUniqueId` — キャッシュ検索用の安定した ID
  * `cachedDescription` — 利用可能な場合の、vision による説明のキャッシュ

<div id="sticker-cache">
  ### スタンプキャッシュ
</div>

スタンプは AI のビジョン機能で処理され、説明テキストが生成されます。同じスタンプが繰り返し送信されることが多いため、OpenClaw はこれらの説明をキャッシュして、不要な API 呼び出しを避けます。

**動作概要:**

1. **初回の受信時:** スタンプ画像がビジョン解析のために AI に送られます。AI は説明文を生成します（例: 「A cartoon cat waving enthusiastically」）。
2. **キャッシュへの保存:** 説明文はスタンプのファイル ID、絵文字、セット名と一緒に保存されます。
3. **2 回目以降の受信時:** 同じスタンプが再び受信された場合、キャッシュ済みの説明がそのまま使用されます。画像は AI に送られません。

**キャッシュの場所:** `~/.openclaw/telegram/sticker-cache.json`

**キャッシュエントリ形式:**

```json
{
  "fileId": "CAACAgIAAxkBAAI...",
  "fileUniqueId": "AgADBAADb6cxG2Y",
  "emoji": "👋",
  "setName": "CoolCats",
  "description": "元気に手を振っている漫画の猫",
  "cachedAt": "2026-01-15T10:30:00.000Z"
}
```

**利点:**

* 同じステッカーに対してビジョン API 呼び出しを繰り返さないことで API コストを削減できる
* キャッシュ済みステッカーは応答が高速になる（ビジョン処理による遅延が発生しない）
* キャッシュされた説明に基づくステッカー検索機能を有効にできる

キャッシュはステッカーを受信したタイミングで自動的に保存されます。手動でキャッシュを管理する必要はありません。

<div id="sending-stickers">
  ### ステッカーの送信
</div>

エージェントは `sticker` および `sticker-search` アクションを使用してステッカーを送信したり検索したりできます。これらのアクションはデフォルトでは無効であり、config で明示的に有効化する必要があります。

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

**スタンプを送信する:**

```json5
{
  "action": "sticker",
  "channel": "telegram",
  "to": "123456789",
  "fileId": "CAACAgIAAxkBAAI..."
}
```

パラメータ:

* `fileId` (必須) — ステッカーの Telegram ファイル ID。受信したステッカーの `Sticker.fileId` から取得するか、`sticker-search` の結果から取得します。
* `replyTo` (任意) — 返信先のメッセージ ID。
* `threadId` (任意) — フォーラムトピックのメッセージスレッド ID。

**ステッカーを検索する:**

エージェントは説明、絵文字、またはセット名でキャッシュされたステッカーを検索できます：

```json5
{
  "action": "sticker-search",
  "channel": "telegram",
  "query": "cat waving",
  "limit": 5
}
```

キャッシュから条件に一致するステッカーを返します。

```json5
{
  "ok": true,
  "count": 2,
  "stickers": [
    {
      "fileId": "CAACAgIAAxkBAAI...",
      "emoji": "👋",
      "description": "元気に手を振っている漫画の猫",
      "setName": "CoolCats"
    }
  ]
}
```

検索では、説明テキスト、絵文字、セット名に対してファジーマッチングを行います。

**スレッドの例:**

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
  ## ストリーミング（ドラフト）
</div>

Telegram は、エージェントが応答を生成している間、**ドラフトバブル**をストリーミング表示できます。
OpenClaw は Bot API の `sendMessageDraft`（実際のメッセージではない）を使用し、
最終的な返信は通常のメッセージとして送信します。

要件（Telegram Bot API 9.3 以降）:

* **トピックが有効なプライベートチャット**（ボット用のフォーラムトピックモード）。
* 受信メッセージに `message_thread_id`（プライベートトピックスレッド）が含まれている必要があります。
* グループ / スーパーグループ / チャンネルではストリーミングは無視されます。

設定:

* `channels.telegram.streamMode: "off" | "partial" | "block"`（デフォルト: `partial`）
  * `partial`: 最新のストリーミングテキストでドラフトバブルを更新します。
  * `block`: ドラフトバブルをより大きなブロック（チャンク）単位で更新します。
  * `off`: ドラフトストリーミングを無効にします。
* オプション（`streamMode: "block"` の場合のみ）:
  * `channels.telegram.draftChunk: { minChars?, maxChars?, breakPreference? }`
    * デフォルト: `minChars: 200`, `maxChars: 800`, `breakPreference: "paragraph"`（`channels.telegram.textChunkLimit` の範囲に収まるように制限されます）。

注意: ドラフトストリーミングは、**ブロックストリーミング**（チャンネルメッセージ）とは別です。
ブロックストリーミングはデフォルトで無効で、ドラフト更新ではなく早めの Telegram メッセージ送信を行いたい場合は
`channels.telegram.blockStreaming: true` を有効にする必要があります。

推論ストリーム（Telegram のみ）:

* `/reasoning stream` は、返信を生成している間、推論内容をドラフトバブルにストリーミングし、
  最後に推論なしの最終回答を送信します。
* `channels.telegram.streamMode` が `off` の場合、推論ストリームは無効化されます。
  詳細: [ストリーミング + チャンク処理](/ja/concepts/streaming)。

<div id="retry-policy">
  ## リトライポリシー
</div>

外部への Telegram API 呼び出しは、一時的なネットワークエラーや 429 エラーが発生した場合、指数バックオフとジッター付きでリトライが行われます。`channels.telegram.retry` で設定できます。詳しくは [リトライポリシー](/ja/concepts/retry) を参照してください。

<div id="agent-tool-messages-reactions">
  ## エージェントツール（メッセージ + リアクション）
</div>

* ツール: `telegram` の `sendMessage` アクション（`to`, `content`, オプションの `mediaUrl`, `replyToMessageId`, `messageThreadId`）。
* ツール: `telegram` の `react` アクション（`chatId`, `messageId`, `emoji`）。
* ツール: `telegram` の `deleteMessage` アクション（`chatId`, `messageId`）。
* リアクション削除の仕様: [/tools/reactions](/ja/tools/reactions) を参照。
* ツールのゲート設定: `channels.telegram.actions.reactions`, `channels.telegram.actions.sendMessage`, `channels.telegram.actions.deleteMessage`（デフォルト: 有効）、および `channels.telegram.actions.sticker`（デフォルト: 無効）。

<div id="reaction-notifications">
  ## リアクション通知
</div>

**リアクションの動作:**
Telegram のリアクションは、メッセージペイロード内のプロパティではなく、**別個の `message_reaction` イベント**として届きます。ユーザーがリアクションを追加すると、OpenClaw は次のように処理します:

1. Telegram API から `message_reaction` のアップデートを受信する
2. それを次の形式の**システムイベント**に変換する: `"Telegram reaction added: {emoji} by {user} on msg {id}"`
3. 通常のメッセージと**同じセッションキー**でシステムイベントをキューイングする
4. その会話で次のメッセージが到着したとき、システムイベントを取り出してエージェントのコンテキストの先頭に追加する

エージェントは、リアクションをメッセージメタデータではなく、会話履歴内の**システム通知**として認識します。

**設定:**

* `channels.telegram.reactionNotifications`: どのリアクションで通知を発生させるかを制御します
  * `"off"` — すべてのリアクションを無視
  * `"own"` — ユーザーがボットのメッセージにリアクションしたときに通知（ベストエフォート／インメモリ）（デフォルト）
  * `"all"` — すべてのリアクションを通知

* `channels.telegram.reactionLevel`: エージェントのリアクション機能を制御します
  * `"off"` — エージェントはメッセージにリアクションできない
  * `"ack"` — ボットが確認用リアクションを送信（処理中は 👀）（デフォルト）
  * `"minimal"` — エージェントは控えめにリアクション可能（目安: 5〜10 往復につき 1 回）
  * `"extensive"` — 適切な場合にはエージェントが積極的にリアクション可能

**フォーラムグループ:** フォーラムグループのリアクションには `message_thread_id` が含まれ、`agent:main:telegram:group:{chatId}:topic:{threadId}` のようなセッションキーが使用されます。これにより、同じトピック内のリアクションとメッセージが同じセッションとしてまとめて扱われます。

**設定例:**

```json5
{
  channels: {
    telegram: {
      reactionNotifications: "all",  // See all reactions
      reactionLevel: "minimal"        // エージェントは控えめにリアクション可能
    }
  }
}
```

**要件:**

* Telegram ボットは、`allowed_updates` 内で `message_reaction` を明示的に要求する必要があります（OpenClaw によって自動的に設定されます）
* Webhook モードでは、リアクションは webhook の `allowed_updates` に含まれます
* ポーリング モードでは、リアクションは `getUpdates` の `allowed_updates` に含まれます

<div id="delivery-targets-clicron">
  ## 配信先（CLI/cron）
</div>

* chat ID（`123456789`）またはユーザー名（`@name`）を配信先として使用します。
* 例: `openclaw message send --channel telegram --target 123456789 --message "hi"`。

<div id="troubleshooting">
  ## トラブルシューティング
</div>

**Bot がグループでメンションなしのメッセージに反応しない:**

* `channels.telegram.groups.*.requireMention=false` を設定している場合、Telegram の Bot API の **プライバシーモード**を無効にする必要があります。
  * BotFather: `/setprivacy` → **Disable**（その後、bot をグループから削除して再追加）
* `openclaw channels status` は、設定がメンションなしのグループメッセージを想定している場合に警告を表示します。
* `openclaw channels status --probe` は、数値のグループ ID に対するメンバーシップを追加でチェックできます（ワイルドカード `"*"` のルールは検査できません）。
* 簡易テスト: `/activation always`（セッション単位のみの有効化。永続化には設定ファイルを使用）

**Bot がグループメッセージをまったく受信しない:**

* `channels.telegram.groups` が設定されている場合、そのグループがリストに含まれているか、または `"*"` を使用する必要があります。
* @BotFather の Privacy Settings を確認 → &quot;Group Privacy&quot; は **OFF** である必要があります。
* bot が実際にメンバーになっていることを確認する（read 権限のない管理者としてだけ追加されていないか確認）。
* Gateway のログを確認: `openclaw logs --follow`（&quot;skipping group message&quot; を探す）

**Bot がメンションには反応するが `/activation always` には反応しない:**

* `/activation` コマンドはセッション状態を更新しますが、設定には永続化されません。
* 永続的な挙動にするには、`channels.telegram.groups` にグループを追加し、`requireMention: false` を指定します。

**`/status` のようなコマンドが動作しない:**

* あなたの Telegram ユーザー ID が認可されていることを確認してください（ペアリング、または `channels.telegram.allowFrom` 経由）。
* コマンドは、`groupPolicy: "open"` のグループ内であっても認可が必要です。

**Node 22+ でロングポーリングがすぐに中断される（プロキシ/カスタム fetch 利用時に多い）:**

* Node 22+ は `AbortSignal` インスタンスに対してより厳格であり、外部由来のシグナルが `fetch` 呼び出しを即座に中断させることがあります。
* AbortSignal を正規化する OpenClaw ビルドにアップグレードするか、アップグレードできるまで Gateway を Node 20 上で実行してください。

**Bot が起動後しばらくして黙って応答しなくなる（あるいは `HttpError: Network request ... failed` がログに出る）:**

* 一部のホストは `api.telegram.org` を IPv6 を優先して解決します。サーバーに有効な IPv6 egress がない場合、grammY が IPv6 のみのリクエストでハングすることがあります。
* IPv6 egress を有効にする**か**、`api.telegram.org` の解決を IPv4 に強制してください（例: IPv4 の A レコードを使って `/etc/hosts` にエントリを追加するか、OS の DNS スタックで IPv4 を優先する）。その後 Gateway を再起動します。
* 簡易チェック: `dig +short api.telegram.org A` および `dig +short api.telegram.org AAAA` を実行し、DNS が何を返すか確認します。

<div id="configuration-reference-telegram">
  ## 設定リファレンス（Telegram）
</div>

完全な設定: [Configuration](/ja/gateway/configuration)

プロバイダーオプション:

* `channels.telegram.enabled`: チャンネルの起動を有効化/無効化します。
* `channels.telegram.botToken`: ボットトークン（BotFather）。
* `channels.telegram.tokenFile`: ファイルパスからトークンを読み込みます。
* `channels.telegram.dmPolicy`: `pairing | allowlist | open | disabled`（デフォルト: pairing）。
* `channels.telegram.allowFrom`: DM の許可リスト（id/ユーザー名）。`open` の場合は `"*"` が必要です。
* `channels.telegram.groupPolicy`: `open | allowlist | disabled`（デフォルト: allowlist）。
* `channels.telegram.groupAllowFrom`: グループ送信者の許可リスト（id/ユーザー名）。
* `channels.telegram.groups`: グループ単位のデフォルト設定 + 許可リスト（グローバルデフォルトには `"*"` を使用）。
  * `channels.telegram.groups.<id>.requireMention`: メンション必須設定のデフォルト。
  * `channels.telegram.groups.<id>.skills`: スキルフィルター（省略 = すべてのスキル、空文字列 = なし）。
  * `channels.telegram.groups.<id>.allowFrom`: グループ単位の送信者許可リストの上書き。
  * `channels.telegram.groups.<id>.systemPrompt`: グループ用の追加 system prompt。
  * `channels.telegram.groups.<id>.enabled`: `false` の場合、このグループを無効化します。
  * `channels.telegram.groups.<id>.topics.<threadId>.*`: トピック単位の上書き（グループと同じフィールド）。
  * `channels.telegram.groups.<id>.topics.<threadId>.requireMention`: トピック単位のメンション必須設定の上書き。
* `channels.telegram.capabilities.inlineButtons`: `off | dm | group | all | allowlist`（デフォルト: allowlist）。
* `channels.telegram.accounts.<account>.capabilities.inlineButtons`: アカウント単位の上書き。
* `channels.telegram.replyToMode`: `off | first | all`（デフォルト: `first`）。
* `channels.telegram.textChunkLimit`: 送信メッセージの分割チャンクサイズ（文字数）。
* `channels.telegram.chunkMode`: `length`（デフォルト）または `newline`。`newline` の場合、長さによる分割の前に空行（段落境界）で分割します。
* `channels.telegram.linkPreview`: 送信メッセージのリンクプレビューを切り替えます（デフォルト: true）。
* `channels.telegram.streamMode`: `off | partial | block`（ドラフトストリーミング）。
* `channels.telegram.mediaMaxMb`: 受信/送信メディアの上限（MB）。
* `channels.telegram.retry`: Telegram API への送信リクエストのリトライポリシー（attempts, minDelayMs, maxDelayMs, jitter）。
* `channels.telegram.network.autoSelectFamily`: Node の autoSelectFamily を上書きします（true = 有効化、false = 無効化）。Node 22 では Happy Eyeballs タイムアウトを避けるためデフォルトで無効です。
* `channels.telegram.proxy`: Bot API 呼び出し用のプロキシ URL（SOCKS/HTTP）。
* `channels.telegram.webhookUrl`: webhook モードを有効化します。
* `channels.telegram.webhookSecret`: webhook シークレット（任意）。
* `channels.telegram.webhookPath`: ローカルの webhook パス（デフォルト `/telegram-webhook`）。
* `channels.telegram.actions.reactions`: Telegram ツールのリアクション操作を制御します。
* `channels.telegram.actions.sendMessage`: Telegram ツールによるメッセージ送信を制御します。
* `channels.telegram.actions.deleteMessage`: Telegram ツールによるメッセージ削除を制御します。
* `channels.telegram.actions.sticker`: Telegram スタンプ（ステッカー）のアクション（送信および検索）を制御します（デフォルト: false）。
* `channels.telegram.reactionNotifications`: `off | own | all` — どのリアクションでシステムイベントをトリガーするかを制御します（デフォルト: 未設定時は `own`）。
* `channels.telegram.reactionLevel`: `off | ack | minimal | extensive` — エージェントのリアクション機能レベルを制御します（デフォルト: 未設定時は `minimal`）。

関連するグローバルオプション:

* `agents.list[].groupChat.mentionPatterns`（メンションによるゲート用パターン）。
* `messages.groupChat.mentionPatterns`（グローバルフォールバック）。
* `commands.native`（デフォルトは `"auto"` → Telegram/Discord ではオン、Slack ではオフ）、`commands.text`, `commands.useAccessGroups`（コマンド動作）。`channels.telegram.commands.native` で上書き可能。
* `messages.responsePrefix`, `messages.ackReaction`, `messages.ackReactionScope`, `messages.removeAckAfterReply`。