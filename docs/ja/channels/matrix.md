---
title: Matrix
summary: "Matrix のサポート状況、機能、および設定"
read_when:
  - Matrix チャネル機能の開発作業中
---

<div id="matrix-plugin">
  # Matrix（プラグイン）
</div>

Matrix はオープンで分散型のメッセージングプロトコルです。OpenClaw は任意のホームサーバー上の Matrix **ユーザー**
として接続するため、ボット用の Matrix アカウントが必要です。ログイン後は、ボットに直接 DM を送るか、
ボットをルーム（Matrix の「グループ」）に招待できます。Beeper も有効なクライアントオプションですが、
E2EE を有効化する必要があります。

ステータス: プラグイン（@vector-im/matrix-bot-sdk）経由でサポートされています。ダイレクトメッセージ、ルーム、スレッド、メディア、リアクション、
投票（テキストとしての send + poll-start）、位置情報、および E2EE（暗号化サポート付き）に対応しています。

<div id="plugin-required">
  ## 必要なプラグイン
</div>

Matrix プラグインはコアのインストールには含まれていません。

CLI（npm レジストリ経由）でインストールします:

```bash
openclaw plugins install @openclaw/matrix
```

ローカルチェックアウト（Git リポジトリから実行する場合）：

```bash
openclaw plugins install ./extensions/matrix
```

configure/onboarding の際に Matrix を選択し、git checkout が検出された場合、
OpenClaw はローカルのインストールパスを自動的に表示します。

詳細: [プラグイン](/ja/plugin)

<div id="setup">
  ## セットアップ
</div>

1. Matrix プラグインをインストールします:
   * npm から: `openclaw plugins install @openclaw/matrix`
   * ローカルのチェックアウトから: `openclaw plugins install ./extensions/matrix`
2. ホームサーバー上に Matrix アカウントを作成します:
   * ホスティングオプションについては [https://matrix.org/ecosystem/hosting/](https://matrix.org/ecosystem/hosting/) を参照してください
   * もしくは自前でホストします。
3. ボットアカウント用のアクセストークンを取得します:

   * ホームサーバーに対して `curl` を使って Matrix の login API を呼び出します:

   ```bash
   curl --request POST \
     --url https://matrix.example.org/_matrix/client/v3/login \
     --header 'Content-Type: application/json' \
     --data '{
     "type": "m.login.password",
     "identifier": {
       "type": "m.id.user",
       "user": "your-user-name"
     },
     "password": "your-password"
   }'
   ```

   * `matrix.example.org` を自分のホームサーバーの URL に置き換えます。
   * もしくは `channels.matrix.userId` と `channels.matrix.password` を設定します。
     OpenClaw は同じ login エンドポイントを呼び出し、アクセストークンを
     `~/.openclaw/credentials/matrix/credentials.json` に保存し、
     次回起動時に再利用します。
4. 資格情報を設定します:
   * 環境変数: `MATRIX_HOMESERVER`, `MATRIX_ACCESS_TOKEN`（または `MATRIX_USER_ID` + `MATRIX_PASSWORD`）
   * もしくは設定: `channels.matrix.*`
   * 両方設定されている場合は、設定ファイルが優先されます。
   * アクセストークンを使用する場合、ユーザー ID は `/whoami` で自動取得されます。
   * `channels.matrix.userId` を設定する場合は、完全な Matrix ID（例: `@bot:example.org`）にしてください。
5. Gateway を再起動するか、オンボーディングを完了します。
6. 任意の Matrix クライアント（Element, Beeper など。https://matrix.org/ecosystem/clients/ を参照）から
   ボットとの DM を開始するか、そのボットをルームに招待します。Beeper は E2EE が必須のため、
   `channels.matrix.encryption: true` を設定し、デバイスを検証してください。

最小構成例（アクセストークンのみを指定し、ユーザー ID は自動取得）:

```json5
{
  channels: {
    matrix: {
      enabled: true,
      homeserver: "https://matrix.example.org",
      accessToken: "syt_***",
      dm: { policy: "pairing" }
    }
  }
}
```

E2EE 設定（エンドツーエンド暗号化を有効にした場合）:

```json5
{
  channels: {
    matrix: {
      enabled: true,
      homeserver: "https://matrix.example.org",
      accessToken: "syt_***",
      encryption: true,
      dm: { policy: "pairing" }
    }
  }
}
```

<div id="encryption-e2ee">
  ## 暗号化 (E2EE)
</div>

エンドツーエンド暗号化は Rust crypto SDK を通じて**サポートされています**。

`channels.matrix.encryption: true` を指定して有効化します:

* crypto モジュールが読み込まれた場合、暗号化ルームは自動的に復号されます。
* 暗号化ルームへの送信時には、送信メディアが暗号化されます。
* 最初の接続時に、OpenClaw はあなたの他のセッションに対してデバイス検証を要求します。
* キー共有を有効にするには、別の Matrix クライアント (Element など) でデバイスを検証してください。
* crypto モジュールを読み込めない場合、E2EE は無効化され、暗号化ルームは復号されません。
  このとき OpenClaw は警告をログ出力します。
* crypto モジュールが見つからないというエラーが表示される場合 (例: `@matrix-org/matrix-sdk-crypto-nodejs-*`)、
  `@matrix-org/matrix-sdk-crypto-nodejs` のビルドスクリプトを許可し、
  `pnpm rebuild @matrix-org/matrix-sdk-crypto-nodejs` を実行するか、
  `node node_modules/@matrix-org/matrix-sdk-crypto-nodejs/download-lib.js` でバイナリを取得してください。

暗号状態はアカウント + アクセストークンごとに
`~/.openclaw/matrix/accounts/<account>/<homeserver>__<user>/<token-hash>/crypto/`
(SQLite データベース) に保存されます。同期状態はその隣の `bot-storage.json` に保存されます。
アクセストークン (デバイス) が変更されると、新しいストアが作成され、
ボットは暗号化ルーム用に再度検証を行う必要があります。

**デバイス検証:**
E2EE が有効な場合、起動時にボットは他のセッションに検証要求を送信します。
Element (または別のクライアント) を開き、検証要求を承認して信頼関係を確立してください。
検証が完了すると、ボットは暗号化ルーム内のメッセージを復号できるようになります。

<div id="routing-model">
  ## ルーティングモデル
</div>

* 返信は常に Matrix に返送されます。
* DM はエージェントのメインセッションを共有し、ルームはグループセッションにマッピングされます。

<div id="access-control-dms">
  ## アクセス制御（DM）
</div>

* デフォルト: `channels.matrix.dm.policy = "pairing"`。未知の送信者にはペアリングコードが付与されます。
* 承認方法:
  * `openclaw pairing list matrix`
  * `openclaw pairing approve matrix <CODE>`
* DM を公開モードにするには: `channels.matrix.dm.policy="open"` に加えて `channels.matrix.dm.allowFrom=["*"]` を設定します（`open` は、任意のユーザーからのDMを無制限に受け付ける設定を示すポリシートークンです）。
* `channels.matrix.dm.allowFrom` はユーザーIDまたは表示名を受け付けます。ディレクトリ検索が利用可能な場合、ウィザードが表示名をユーザーIDに解決します。

<div id="rooms-groups">
  ## ルーム（グループ）
</div>

* デフォルト: `channels.matrix.groupPolicy = "allowlist"`（メンション経由のみ許可）。未設定時のデフォルトを上書きするには `channels.defaults.groupPolicy` を使用します。
* `channels.matrix.groups`（ルーム ID、エイリアス、または名前）でルームを許可リストに登録します:

```json5
{
  channels: {
    matrix: {
      groupPolicy: "allowlist",
      groups: {
        "!roomId:example.org": { allow: true },
        "#alias:example.org": { allow: true }
      },
      groupAllowFrom: ["@owner:example.org"]
    }
  }
}
```

* `requireMention: false` にすると、そのルームで自動返信が有効になります。
* `groups."*"` で、全ルーム共通のメンション要否のデフォルト設定を行えます。
* `groupAllowFrom` は、そのルームでどの送信者がボットをトリガーできるかを制限します（省略可能）。
* ルーム単位の `users` 許可リストで、特定ルーム内の送信者をさらに絞り込めます。
* 設定ウィザードは、ルーム許可リスト（ルーム ID、エイリアス、または名前）の入力を促し、可能な場合は名前を解決します。
* 起動時に OpenClaw は、許可リスト内のルーム／ユーザー名を ID に解決してマッピングをログに記録し、解決できなかったエントリは入力どおりに保持します。
* 招待はデフォルトで自動参加します。`channels.matrix.autoJoin` と `channels.matrix.autoJoinAllowlist` で制御します。
* **どのルームも許可しない** 場合は、`channels.matrix.groupPolicy: "disabled"` を設定します（または許可リストを空のままにします）。
* レガシーキー: `channels.matrix.rooms`（`groups` と同じ構造）。

<div id="threads">
  ## スレッド
</div>

* 返信のスレッド表示に対応しています。
* `channels.matrix.threadReplies` は、返信をスレッド内に残すかどうかを制御します:
  * `off`、`inbound`（デフォルト）、`always`
* `channels.matrix.replyToMode` は、スレッド外で返信する場合の reply-to メタデータを制御します:
  * `off`（デフォルト）、`first`、`all`

<div id="capabilities">
  ## 機能
</div>

| 機能 | ステータス |
|---------|--------|
| ダイレクトメッセージ | ✅ サポート |
| ルーム | ✅ サポート |
| スレッド | ✅ サポート |
| メディア | ✅ サポート |
| E2EE | ✅ サポート（crypto モジュールが必要） |
| リアクション | ✅ サポート（ツール経由での送信/read） |
| 投票 | ✅ 送信をサポート。受信した投票の開始はテキストに変換（回答/終了は無視） |
| 位置情報 | ✅ サポート（geo URI。高度は無視） |
| ネイティブコマンド | ✅ サポート |

<div id="configuration-reference-matrix">
  ## 設定リファレンス（Matrix）
</div>

完全な設定: [Configuration](/ja/gateway/configuration)

プロバイダーオプション:

* `channels.matrix.enabled`: チャンネルの起動を有効/無効にする。
* `channels.matrix.homeserver`: ホームサーバーの URL。
* `channels.matrix.userId`: Matrix ユーザー ID（アクセストークンがあれば省略可）。
* `channels.matrix.accessToken`: アクセストークン。
* `channels.matrix.password`: ログイン用パスワード（トークンを保存）。
* `channels.matrix.deviceName`: デバイス表示名。
* `channels.matrix.encryption`: エンドツーエンド暗号化（E2EE）を有効化（デフォルト: false）。
* `channels.matrix.initialSyncLimit`: 初回同期の上限件数。
* `channels.matrix.threadReplies`: `off | inbound | always`（デフォルト: inbound）。
* `channels.matrix.textChunkLimit`: 送信テキストのチャンクサイズ（文字数）。
* `channels.matrix.chunkMode`: `length`（デフォルト）または `newline`。`newline` は長さで分割する前に空行（段落境界）で分割する。
* `channels.matrix.dm.policy`: `pairing | allowlist | open | disabled`（デフォルト: pairing）。
* `channels.matrix.dm.allowFrom`: DM の許可リスト（ユーザー ID または表示名）。`open` の場合は `"*"` が必須。ウィザードは可能な場合、名前を ID に解決する。
* `channels.matrix.groupPolicy`: `allowlist | open | disabled`（デフォルト: allowlist）。
* `channels.matrix.groupAllowFrom`: グループメッセージの許可送信者。
* `channels.matrix.allowlistOnly`: DM とルームに対して許可リストルールを強制。
* `channels.matrix.groups`: グループ許可リスト + ルーム単位の設定マップ。
* `channels.matrix.rooms`: レガシーなグループ許可リスト/設定。
* `channels.matrix.replyToMode`: スレッド/タグ用の返信モード。
* `channels.matrix.mediaMaxMb`: 受信/送信メディアの上限（MB）。
* `channels.matrix.autoJoin`: 招待処理（`always | allowlist | off`、デフォルト: always）。
* `channels.matrix.autoJoinAllowlist`: 自動参加を許可するルーム ID/エイリアス。
* `channels.matrix.actions`: アクションごとのツール制御（reactions/messages/pins/memberInfo/channelInfo）。