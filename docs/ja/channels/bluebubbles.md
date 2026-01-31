---
title: Bluebubbles
summary: "BlueBubbles macOS サーバー経由での iMessage（REST による送受信、入力中表示、リアクション、ペアリング、高度な操作）。"
read_when:
  - BlueBubbles チャンネルをセットアップするとき
  - webhook のペアリングをトラブルシューティングするとき
  - macOS 上で iMessage を設定するとき
---

<div id="bluebubbles-macos-rest">
  # BlueBubbles (macOS REST)
</div>

ステータス: HTTP 経由で BlueBubbles macOS サーバーと通信する同梱プラグイン。従来の imsg チャンネルと比較して、よりリッチな API と簡単なセットアップが可能なため、**iMessage 連携用途に推奨されます**。

<div id="overview">
  ## 概要
</div>

* BlueBubbles ヘルパーアプリ（[bluebubbles.app](https://bluebubbles.app)）経由で macOS 上で動作します。
* 推奨／テスト済み: macOS Sequoia (15)。macOS Tahoe (26) でも動作しますが、Tahoe では現在編集機能が動作せず、グループアイコンの更新は成功と表示されても同期されない場合があります。
* OpenClaw は REST API（`GET /api/v1/ping`、`POST /message/text`、`POST /chat/:id/*`）経由で通信します。
* 受信メッセージは webhook で届き、送信返信、入力中インジケーター、既読通知、Tapback リアクションは REST コールとして送信されます。
* 添付ファイルとステッカーは受信メディアとして取り込まれ、可能な場合はエージェントに提示されます。
* ペアリング／許可リストは他のチャネルと同様に（`/start/pairing` など）`channels.bluebubbles.allowFrom`＋ペアリングコードで動作します。
* リアクションは Slack／Telegram と同様にシステムイベントとして表面化されるため、エージェントは返信前にそれらへ「言及」できます。
* 高度な機能: 編集、送信取り消し、スレッド形式の返信、メッセージエフェクト、グループ管理。

<div id="quick-start">
  ## クイックスタート
</div>

1. Mac に BlueBubbles サーバーをインストールします（[bluebubbles.app/install](https://bluebubbles.app/install) の手順に従ってください）。
2. BlueBubbles の設定で Web API を有効化し、パスワードを設定します。
3. `openclaw onboard` を実行して BlueBubbles を選択するか、手動で次のように設定します：
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
4. BlueBubbles の webhook の送信先を Gateway に設定します（例：`https://your-gateway-host:3000/bluebubbles-webhook?password=<password>`）。
5. Gateway を起動すると、webhook ハンドラーが登録され、ペアリングが開始されます。

<div id="onboarding">
  ## オンボーディング
</div>

BlueBubbles は対話型セットアップウィザードから利用を開始できます。

```
openclaw onboard
```

ウィザードでは次の項目の入力が求められます:

* **Server URL** (必須): BlueBubbles サーバーのアドレス (例: `http://192.168.1.100:1234`)
* **Password** (必須): BlueBubbles Server の設定で指定した API パスワード
* **Webhook path** (任意): デフォルトは `/bluebubbles-webhook`
* **DM policy**: pairing, allowlist, open（すべてのユーザーからのメッセージを無制限に受け付ける）, または disabled
* **Allow list**: 許可リストとして登録する電話番号、メールアドレス、またはチャットの宛先

BlueBubbles は CLI 経由で追加することもできます:

```
openclaw channels add bluebubbles --http-url http://192.168.1.100:1234 --password <password>
```

<div id="access-control-dms-groups">
  ## アクセス制御（DM とグループ）
</div>

DM:

* 既定値: `channels.bluebubbles.dmPolicy = "pairing"`。
* 不明な送信者にはペアリングコードが送られ、承認されるまでメッセージは無視されます（コードの有効期限は 1 時間）。
* 承認方法:
  * `openclaw pairing list bluebubbles`
  * `openclaw pairing approve bluebubbles <CODE>`
* ペアリングはデフォルトのトークン交換方式です。詳細: [Pairing](/ja/start/pairing)

グループ:

* `channels.bluebubbles.groupPolicy = open | allowlist | disabled`（既定値: `allowlist`）。`open` は、任意のユーザーからのメッセージ受信を無制限に許可する設定です。
* `channels.bluebubbles.groupAllowFrom` は、`allowlist` が設定されている場合に、誰がグループ内でトリガーできるかを制御します。

<div id="mention-gating-groups">
  ### メンションゲーティング（グループ）
</div>

BlueBubbles はグループチャット向けのメンションゲーティングをサポートしており、iMessage／WhatsApp と同様に動作します：

* メンションを検出するために `agents.list[].groupChat.mentionPatterns`（または `messages.groupChat.mentionPatterns`）を使用します。
* グループで `requireMention` が有効な場合、エージェントはメンションされたときのみ応答します。
* 認可された送信者からの制御コマンドは、メンションゲーティングの対象外となります。

グループごとの設定：

```json5
{
  channels: {
    bluebubbles: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15555550123"],
      groups: {
        "*": { requireMention: true },  // default for all groups
        "iMessage;-;chat123": { requireMention: false }  // 特定グループの上書き
      }
    }
  }
}
```

<div id="command-gating">
  ### コマンドゲーティング
</div>

* 制御コマンド（例: `/config`、`/model`）には認可が必要です。
* `allowFrom` と `groupAllowFrom` を使用して、コマンドの認可を判定します。
* 認可された送信者は、グループでメンションされていなくても制御コマンドを実行できます。

<div id="typing-read-receipts">
  ## 入力中インジケーターと既読通知
</div>

* **入力中インジケーター**: レスポンスを生成する前および生成中に自動的に送信されます。
* **既読通知**: `channels.bluebubbles.sendReadReceipts` で制御されます（デフォルト: `true`）。
* **入力中インジケーター**: OpenClaw は入力開始イベントを送信し、BlueBubbles は送信またはタイムアウト時に自動的に入力中状態をクリアします（DELETE リクエストによる手動停止は信頼性に欠けます）。

```json5
{
  channels: {
    bluebubbles: {
      sendReadReceipts: false  // 既読通知を無効化
    }
  }
}
```

<div id="advanced-actions">
  ## 高度なアクション
</div>

BlueBubbles では、設定で有効にしている場合、メッセージの高度な操作をサポートします。

```json5
{
  channels: {
    bluebubbles: {
      actions: {
        reactions: true,       // tapbacks (default: true)
        edit: true,            // 送信済みメッセージの編集 (macOS 13+、macOS 26 Tahoe では不具合あり)
        unsend: true,          // unsend messages (macOS 13+)
        reply: true,           // reply threading by message GUID
        sendWithEffect: true,  // message effects (slam, loud, etc.)
        renameGroup: true,     // rename group chats
        setGroupIcon: true,    // set group chat icon/photo (flaky on macOS 26 Tahoe)
        addParticipant: true,  // add participants to groups
        removeParticipant: true, // remove participants from groups
        leaveGroup: true,      // leave group chats
        sendAttachment: true   // send attachments/media
      }
    }
  }
}
```

利用可能なアクション:

* **react**: Tapbackリアクションを追加/削除する (`messageId`, `emoji`, `remove`)
* **edit**: 送信済みメッセージを編集する (`messageId`, `text`)
* **unsend**: メッセージの送信を取り消す (`messageId`)
* **reply**: 特定のメッセージに返信する (`messageId`, `text`, `to`)
* **sendWithEffect**: iMessageエフェクト付きで送信する (`text`, `to`, `effectId`)
* **renameGroup**: グループチャット名を変更する (`chatGuid`, `displayName`)
* **setGroupIcon**: グループチャットのアイコン/写真を設定する (`chatGuid`, `media`) — macOS 26 Tahoe では動作が不安定で、APIは成功を返してもアイコンが同期されない場合があります。
* **addParticipant**: グループに参加者を追加する (`chatGuid`, `address`)
* **removeParticipant**: グループから参加者を削除する (`chatGuid`, `address`)
* **leaveGroup**: グループチャットから退出する (`chatGuid`)
* **sendAttachment**: メディア/ファイルを送信する (`to`, `buffer`, `filename`, `asVoice`)
  * ボイスメモ: **MP3** または **CAF** オーディオを iMessage の音声メッセージとして送信するには、`asVoice: true` を設定します。BlueBubbles はボイスメモ送信時に MP3 を CAF に変換します。

<div id="message-ids-short-vs-full">
  ### メッセージ ID（短い形式 vs フル形式）
</div>

OpenClaw はトークン節約のために *短い* メッセージ ID（例: `1`、`2`）を出力する場合があります。

* `MessageSid` / `ReplyToId` は短い ID になる場合があります。
* `MessageSidFull` / `ReplyToIdFull` にはプロバイダー側のフル ID が含まれます。
* 短い ID はメモリ内のみで管理され、再起動やキャッシュ削除によって失効する可能性があります。
* アクションは短い形式とフル形式のどちらの `messageId` も受け付けますが、短い ID がすでに利用できない場合はエラーになります。

永続的な自動化やストレージ用途ではフル ID を使用してください:

* テンプレート: `{{MessageSidFull}}`, `{{ReplyToIdFull}}`
* コンテキスト: インバウンドペイロード内の `MessageSidFull` / `ReplyToIdFull`

テンプレート変数については [Configuration](/ja/gateway/configuration) を参照してください。

<div id="block-streaming">
  ## ブロックストリーミング
</div>

レスポンスを1つのメッセージとして送信するか、ブロックごとにストリーミングするかを制御します。

```json5
{
  channels: {
    bluebubbles: {
      blockStreaming: true  // ブロックストリーミングを有効化（デフォルトの動作）
    }
  }
}
```

<div id="media-limits">
  ## メディアと制限
</div>

* 受信した添付ファイルはダウンロードされ、メディアキャッシュに保存されます。
* メディア上限は `channels.bluebubbles.mediaMaxMb` で設定します（デフォルト: 8 MB）。
* 送信テキストは `channels.bluebubbles.textChunkLimit`（デフォルト: 4000 文字）で指定された上限に合わせて分割されます。

<div id="configuration-reference">
  ## 設定リファレンス
</div>

設定全体: [Configuration](/ja/gateway/configuration)

プロバイダーオプション:

* `channels.bluebubbles.enabled`: チャンネルを有効化/無効化します。
* `channels.bluebubbles.serverUrl`: BlueBubbles REST API のベース URL。
* `channels.bluebubbles.password`: API パスワード。
* `channels.bluebubbles.webhookPath`: Webhook エンドポイントパス (デフォルト: `/bluebubbles-webhook`)。
* `channels.bluebubbles.dmPolicy`: `pairing | allowlist | open | disabled` (デフォルト: `pairing`)。
* `channels.bluebubbles.allowFrom`: DM 許可リスト (ハンドル、メールアドレス、E.164 番号、`chat_id:*`、`chat_guid:*`)。
* `channels.bluebubbles.groupPolicy`: `open | allowlist | disabled` (デフォルト: `allowlist`)。
* `channels.bluebubbles.groupAllowFrom`: グループ送信者の許可リスト。
* `channels.bluebubbles.groups`: グループ単位の設定 (`requireMention` など)。
* `channels.bluebubbles.sendReadReceipts`: 既読レシートを送信 (デフォルト: `true`)。
* `channels.bluebubbles.blockStreaming`: ブロックストリーミングを有効化 (デフォルト: `true`)。
* `channels.bluebubbles.textChunkLimit`: 送信テキストのチャンクサイズ (文字数) (デフォルト: 4000)。
* `channels.bluebubbles.chunkMode`: `length` (デフォルト) は `textChunkLimit` を超えた場合のみ分割します。`newline` は長さによる分割の前に、空行 (段落境界) で分割します。
* `channels.bluebubbles.mediaMaxMb`: 受信メディアの上限 (MB) (デフォルト: 8)。
* `channels.bluebubbles.historyLimit`: コンテキストとして保持するグループメッセージの最大数 (0 で無効化)。
* `channels.bluebubbles.dmHistoryLimit`: DM 履歴の最大数。
* `channels.bluebubbles.actions`: 特定のアクションを有効化/無効化します。
* `channels.bluebubbles.accounts`: マルチアカウント設定。

関連するグローバルオプション:

* `agents.list[].groupChat.mentionPatterns` (または `messages.groupChat.mentionPatterns`)。
* `messages.responsePrefix`。

<div id="addressing-delivery-targets">
  ## アドレス指定 / 配信ターゲット
</div>

安定したルーティングのためには `chat_guid` を優先して使用してください:

* `chat_guid:iMessage;-;+15555550123`（グループでは推奨）
* `chat_id:123`
* `chat_identifier:...`
* 直接のハンドル: `+15555550123`, `user@example.com`
  * 直接のハンドルに既存の DM チャットが存在しない場合、OpenClaw は `POST /api/v1/chat/new` を経由してチャットを新規作成します。これには BlueBubbles Private API が有効になっている必要があります。

<div id="security">
  ## セキュリティ
</div>

* Webhook リクエストは、クエリパラメータまたはヘッダーの `guid` / `password` を `channels.bluebubbles.password` と比較することで認証されます。`localhost` からのリクエストも受け付けられます。
* API パスワードと Webhook エンドポイントは秘密に保ってください（認証情報と同様に扱ってください）。
* `localhost` を信頼する設定では、同一ホスト上のリバースプロキシが意図せずパスワード認証を迂回してしまう可能性があります。Gateway をプロキシする場合は、プロキシ側で認証を必須にし、`gateway.trustedProxies` を設定してください。詳しくは [Gateway security](/ja/gateway/security#reverse-proxy-configuration) を参照してください。
* LAN の外部に BlueBubbles サーバーを公開する場合は、HTTPS とファイアウォールルールを有効にしてください。

<div id="troubleshooting">
  ## トラブルシューティング
</div>

* タイピング/read イベントが動作しなくなった場合は、BlueBubbles サーバーの webhook ログを確認し、Gateway のパスが `channels.bluebubbles.webhookPath` と一致していることを確認してください。
* ペアリングコードは 1 時間で期限切れになります。`openclaw pairing list bluebubbles` と `openclaw pairing approve bluebubbles <code>` を使用してください。
* リアクションには BlueBubbles のプライベート API（`POST /api/v1/message/react`）が必要です。利用中のサーバーバージョンでこの API が公開されていることを確認してください。
* 編集/送信取り消しには、macOS 13 以降かつ互換性のある BlueBubbles サーバーバージョンが必要です。macOS 26（Tahoe）では、プライベート API の変更により現在 edit が動作していません。
* グループアイコンの更新は macOS 26（Tahoe）では不安定になることがあります。API が成功を返しても、新しいアイコンが同期されない場合があります。
* OpenClaw は、BlueBubbles サーバーの macOS バージョンに基づいて、既知の不具合があるアクションを自動的に非表示にします。macOS 26（Tahoe）でまだ edit が表示される場合は、`channels.bluebubbles.actions.edit=false` を設定して手動で無効化してください。
* ステータス/ヘルス情報の確認には、`openclaw status --all` または `openclaw status --deep` を使用してください。

チャネル全般のワークフローについては、[チャネル](/ja/channels) と [プラグイン](/ja/plugins) ガイドを参照してください。