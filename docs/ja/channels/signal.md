---
title: Signal
summary: "signal-cli（JSON-RPC + SSE）を利用した Signal サポート、セットアップ、電話番号モデル"
read_when:
  - Signal サポートのセットアップ
  - Signal 送信/受信のデバッグ
---

<div id="signal-signal-cli">
  # Signal (signal-cli)
</div>

状態: 外部CLI連携。Gatewayは HTTP JSON-RPC と SSE を通じて `signal-cli` と通信します。

<div id="quick-setup-beginner">
  ## クイックセットアップ（初心者向け）
</div>

1. ボット用に **別の Signal 番号** を使用してください（推奨）。
2. `signal-cli` をインストールしてください（Java が必要です）。
3. ボットデバイスをリンクし、デーモンを起動します:
   * `signal-cli link -n "OpenClaw"`
4. OpenClaw を設定し、Gateway を起動します。

最小限の設定:

```json5
{
  channels: {
    signal: {
      enabled: true,
      account: "+15551234567",
      cliPath: "signal-cli",
      dmPolicy: "pairing",
      allowFrom: ["+15557654321"]
    }
  }
}
```

<div id="what-it-is">
  ## 概要
</div>

* `signal-cli` を使った Signal チャンネル（組み込みの libsignal ではなく外部の CLI を利用）。
* 決定的なルーティング：返信は必ず Signal に戻る。
* DM（ダイレクトメッセージ）はエージェントのメインセッションを共有し、グループは分離される（`agent:<agentId>:signal:group:<groupId>`）。

<div id="config-writes">
  ## 設定の書き込み
</div>

デフォルトでは、Signal には `/config set|unset` によってトリガーされる設定更新を書き込むことが許可されています（`commands.config: true` が必要）。

これを無効にするには、次のようにします：

```json5
{
  channels: { signal: { configWrites: false } }
}
```

<div id="the-number-model-important">
  ## 番号モデル（重要）
</div>

* Gateway は **Signal デバイス**（`signal-cli` アカウント）に接続します。
* **自分の個人用 Signal アカウント**でボットを動かす場合は、自分自身が送ったメッセージは無視されます（ループ防止のため）。
* 「自分がボットにメッセージを送ると、ボットが返信してくれる」形で使いたい場合は、**ボット専用の別番号**を用意してください。

<div id="setup-fast-path">
  ## セットアップ（簡易手順）
</div>

1. `signal-cli` をインストールします（Java が必要）。
2. ボットアカウントをリンクします:
   * `signal-cli link -n "OpenClaw"` を実行し、Signal で表示される QR コードをスキャンします。
3. Signal を設定し、Gateway を起動します。

例:

```json5
{
  channels: {
    signal: {
      enabled: true,
      account: "+15551234567",
      cliPath: "signal-cli",
      dmPolicy: "pairing",
      allowFrom: ["+15557654321"]
    }
  }
}
```

マルチアカウント対応: `channels.signal.accounts` を使用して、アカウントごとの設定と任意指定の `name` フィールドを定義します。共通パターンについては [`gateway/configuration`](/ja/gateway/configuration#telegramaccounts--discordaccounts--slackaccounts--signalaccounts--imessageaccounts) を参照してください。

<div id="external-daemon-mode-httpurl">
  ## 外部デーモンモード (httpUrl)
</div>

JVM のコールドスタートが遅い、コンテナ初期化が必要、CPU を共有しているといった理由から `signal-cli` を自前で管理したい場合は、デーモンを別途起動し、OpenClaw からそのデーモンを参照するように設定します。

```json5
{
  channels: {
    signal: {
      httpUrl: "http://127.0.0.1:8080",
      autoStart: false
    }
  }
}
```

これは OpenClaw 内での自動起動処理と起動待ち時間をスキップします。自動起動時の起動が遅い場合は、`channels.signal.startupTimeoutMs` を設定してください。

<div id="access-control-dms-groups">
  ## アクセス制御（DM とグループ）
</div>

DM:

* デフォルト: `channels.signal.dmPolicy = "pairing"`。
* 未知の送信者にはペアリングコードが送信され、承認されるまでメッセージは無視されます（コードの有効期限は 1 時間）。
* 承認手順:
  * `openclaw pairing list signal`
  * `openclaw pairing approve signal <CODE>`
* ペアリングは、Signal の DM におけるデフォルトのトークン交換メカニズムです。詳細: [Pairing](/ja/start/pairing)
* UUID しか分からない送信者（`sourceUuid` 由来）は、`channels.signal.allowFrom` に `uuid:<id>` として保存されます。

グループ:

* `channels.signal.groupPolicy = open | allowlist | disabled`（`open` は、任意のユーザーからのメッセージを制限なく受け付ける設定）。
* `channels.signal.groupAllowFrom` は、`allowlist` が設定されている場合に、グループ内で誰がトリガーできるかを制御します。

<div id="how-it-works-behavior">
  ## 動作の仕組み（挙動）
</div>

* `signal-cli` はデーモンとして動作し、Gateway は SSE 経由でイベントを取得します。
* 受信メッセージは共通のチャネルエンベロープ形式に正規化されます。
* 返信は常に同じ電話番号またはグループにルーティングされます。

<div id="media-limits">
  ## メディアと制限
</div>

* 送信テキストは `channels.signal.textChunkLimit`（デフォルト 4000）ごとに分割されます。
* 任意の改行チャンク分割: 長さによるチャンク分割の前に、空行（段落境界）で分割するには `channels.signal.chunkMode="newline"` を設定します。
* 添付ファイルをサポートします（`signal-cli` から base64 で取得）。
* デフォルトのメディア上限: `channels.signal.mediaMaxMb`（デフォルト 8）。
* メディアのダウンロードをスキップするには `channels.signal.ignoreAttachments` を使用します。
* グループ履歴コンテキストには `channels.signal.historyLimit`（または `channels.signal.accounts.*.historyLimit`）が使用され、未設定の場合は `messages.groupChat.historyLimit` が使われます。無効化するには `0` を設定します（デフォルト 50）。

<div id="typing-read-receipts">
  ## 入力中インジケーターと既読通知
</div>

* **入力中インジケーター**: OpenClaw は `signal-cli sendTyping` を介して入力中ステータスを送信し、返信処理が実行されている間は継続的に更新します。
* **既読通知**: `channels.signal.sendReadReceipts` が true の場合、OpenClaw は許可されている DM（ダイレクトメッセージ）に対して既読通知を転送します。
* signal-cli はグループ向けの既読通知を提供していません。

<div id="reactions-message-tool">
  ## リアクション（message ツール）
</div>

* `channel=signal` と併せて `message action=react` を使用します。
* 対象: 送信者の E.164 または UUID（ペアリング出力の `uuid:<id>` を使用、UUID 単体でも可）。
* `messageId` は、リアクション対象メッセージの Signal タイムスタンプです。
* グループへのリアクションには `targetAuthor` または `targetAuthorUuid` が必要です。

例:

```
message action=react channel=signal target=uuid:123e4567-e89b-12d3-a456-426614174000 messageId=1737630212345 emoji=🔥
message action=react channel=signal target=+15551234567 messageId=1737630212345 emoji=🔥 remove=true
message action=react channel=signal target=signal:group:<groupId> targetAuthor=uuid:<sender-uuid> messageId=1737630212345 emoji=✅
```

Config:

* `channels.signal.actions.reactions`: リアクション用アクションの有効化/無効化（デフォルトは true）。
* `channels.signal.reactionLevel`: `off | ack | minimal | extensive`。
  * `off`/`ack` はエージェントによるリアクションを無効化します（メッセージツール `react` はエラーになります）。
  * `minimal`/`extensive` はエージェントによるリアクションを有効化し、ガイダンスレベルを設定します。
* アカウント単位の上書き設定: `channels.signal.accounts.<id>.actions.reactions`, `channels.signal.accounts.<id>.reactionLevel`。

<div id="delivery-targets-clicron">
  ## 配信ターゲット（CLI/cron）
</div>

* DM: `signal:+15551234567`（またはプレーンな E.164 形式）
* UUID DM: `uuid:<id>`（または UUID のみ）
* グループ: `signal:group:<groupId>`
* ユーザー名: `username:<name>`（利用している Signal アカウントでサポートされている場合）

<div id="configuration-reference-signal">
  ## 設定リファレンス（Signal）
</div>

設定全体: [Configuration](/ja/gateway/configuration)

プロバイダーオプション:

* `channels.signal.enabled`: チャンネルの起動を有効/無効にする。
* `channels.signal.account`: ボットアカウント用の E.164。
* `channels.signal.cliPath`: `signal-cli` へのパス。
* `channels.signal.httpUrl`: デーモンの完全な URL（host/port を上書き）。
* `channels.signal.httpHost`, `channels.signal.httpPort`: デーモンのバインド（デフォルト 127.0.0.1:8080）。
* `channels.signal.autoStart`: デーモンを自動起動する（`httpUrl` が未設定の場合のデフォルトは true）。
* `channels.signal.startupTimeoutMs`: 起動待機タイムアウト時間（ミリ秒単位、最大 120000）。
* `channels.signal.receiveMode`: `on-start | manual`。
* `channels.signal.ignoreAttachments`: 添付ファイルのダウンロードをスキップする。
* `channels.signal.ignoreStories`: デーモンからのストーリーを無視する。
* `channels.signal.sendReadReceipts`: 既読レシートを転送する。
* `channels.signal.dmPolicy`: `pairing | allowlist | open | disabled`（デフォルト: pairing）。
* `channels.signal.allowFrom`: DM の許可リスト（E.164 または `uuid:<id>`）。`open` の場合は `"*"` が必要。Signal にはユーザー名が存在しないため、電話番号/UUID ID を使用する。
* `channels.signal.groupPolicy`: `open | allowlist | disabled`（デフォルト: allowlist）。
* `channels.signal.groupAllowFrom`: グループ送信者の許可リスト。
* `channels.signal.historyLimit`: コンテキストとして含めるグループメッセージ数の上限（0 で無効）。
* `channels.signal.dmHistoryLimit`: DM の履歴上限（ユーザーターン数）。ユーザーごとのオーバーライド: `channels.signal.dms["<phone_or_uuid>"].historyLimit`。
* `channels.signal.textChunkLimit`: 送信メッセージのチャンクサイズ（文字数）。
* `channels.signal.chunkMode`: `length`（デフォルト）または `newline`。`newline` の場合、長さによるチャンク分割の前に空行（段落境界）で分割する。
* `channels.signal.mediaMaxMb`: 受信/送信メディアの上限（MB）。

関連するグローバルオプション:

* `agents.list[].groupChat.mentionPatterns`（Signal はネイティブなメンションをサポートしない）。
* `messages.groupChat.mentionPatterns`（グローバルなフォールバック）。
* `messages.responsePrefix`。