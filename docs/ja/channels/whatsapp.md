---
title: Whatsapp
summary: "WhatsApp（Web チャネル）連携：ログイン、受信トレイ、返信、メディア、運用"
read_when:
  - WhatsApp/Web チャネルの動作または受信トレイのルーティングに関する作業をしている場合
---

<div id="whatsapp-web-channel">
  # WhatsApp（Web チャンネル）
</div>

ステータス：Baileys 経由の WhatsApp Web のみ対応。Gateway がセッションを管理します。

<div id="quick-setup-beginner">
  ## クイックセットアップ（初心者向け）
</div>

1. 可能であれば、**別の電話番号**を使用してください（推奨）。
2. `~/.openclaw/openclaw.json` で WhatsApp を設定してください。
3. `openclaw channels login` を実行して、QR コード（Linked Devices）をスキャンしてください。
4. Gateway を起動してください。

最小限の設定:

```json5
{
  channels: {
    whatsapp: {
      dmPolicy: "allowlist",
      allowFrom: ["+15551234567"]
    }
  }
}
```

<div id="goals">
  ## 目標
</div>

* 1つの Gateway プロセス内で複数の WhatsApp アカウント（マルチアカウント）を扱えること。
* ルーティングが一意に決まること：返信は必ず WhatsApp に戻り、モデル側でのルーティングは行われないこと。
* モデルが引用返信を理解できるだけのコンテキストを取得できること。

<div id="config-writes">
  ## Config writes
</div>

デフォルトでは、WhatsApp には `/config set|unset` によってトリガーされる設定更新を行うことが許可されています（`commands.config: true` が必要）。

これを無効化するには次を実行します：

```json5
{
  channels: { whatsapp: { configWrites: false } }
}
```

<div id="architecture-who-owns-what">
  ## アーキテクチャ（どれが何を担当するか）
</div>

* **Gateway** が Baileys のソケットと inbox ループを管理します。
* **CLI / macOS アプリ** は Gateway と通信し、Baileys を直接は利用しません。
* 送信には **アクティブなリスナー** が必須で、存在しない場合は送信は即座に失敗します。

<div id="getting-a-phone-number-two-modes">
  ## 電話番号の取得方法（2つのモード）
</div>

WhatsApp では、認証のために有効な携帯電話番号が必要です。VoIP や仮想番号は通常ブロックされます。OpenClaw を WhatsApp で動かすには、次の 2 通りの方法がサポートされています。

<div id="dedicated-number-recommended">
  ### 専用番号（推奨）
</div>

OpenClaw 用には**専用の電話番号**を用意してください。UX が最も良く、ルーティングも明確で、自分とのチャット特有の妙な挙動も避けられます。理想的な構成は **予備の／古い Android 端末 + eSIM** です。Wi‑Fi と電源につないだままにし、QR コードでリンクしてください。

**WhatsApp Business:** 同じ端末上で、異なる番号を使って WhatsApp Business を併用できます。個人用 WhatsApp と分けておくのに最適です — WhatsApp Business をインストールし、そこで OpenClaw 用の番号を登録してください。

**サンプル設定（専用番号・単一ユーザー許可リスト）：**

```json5
{
  channels: {
    whatsapp: {
      dmPolicy: "allowlist",
      allowFrom: ["+15551234567"]
    }
  }
}
```

**ペアリングモード（オプション）:**
許可リストではなくペアリングを使いたい場合は、`channels.whatsapp.dmPolicy` を `pairing` に設定します。未登録の送信者にはペアリングコードが表示されます。次のコマンドで承認します:
`openclaw pairing approve whatsapp <code>`

<div id="personal-number-fallback">
  ### 個人番号（フォールバック）
</div>

フォールバック用の簡単な方法として、**自分の番号**で OpenClaw を動かします。テスト時は連絡先にスパムを送らないよう、自分自身にメッセージを送信してください（WhatsApp の「自分宛てメッセージ」など）。セットアップや検証中は、メインのスマートフォンで認証コードを確認することになると想定してください。**セルフチャットモードを有効にしておく必要があります。**
ウィザードで個人の WhatsApp 番号を尋ねられたら、アシスタントの番号ではなく、メッセージを送信する側（所有者／送信者）の電話番号を入力してください。

**サンプル設定（個人番号・セルフチャット）:**

```json
{
  "whatsapp": {
    "selfChatMode": true,
    "dmPolicy": "allowlist",
    "allowFrom": ["+15551234567"]
  }
}
```

自己チャットの返信は、`messages.responsePrefix` が未設定の場合、設定されていればデフォルトで `[{identity.name}]`（設定されていなければ `[openclaw]`）になります。プレフィックスをカスタマイズまたは無効化するには、明示的に値を設定してください（削除するには `""` を使用します）。

<div id="number-sourcing-tips">
  ### 電話番号の調達に関するヒント
</div>

* 自国の携帯キャリアによる **ローカル eSIM**（最も信頼性が高い）
  * オーストリア: [hot.at](https://www.hot.at)
  * イギリス: [giffgaff](https://www.giffgaff.com) — 無料SIM、契約不要
* **プリペイドSIM** — 安価で、認証用のSMSを1通受信できれば十分

**避けるべきもの:** TextNow、Google Voice、ほとんどの「無料SMS」サービス — WhatsApp はこれらを積極的にブロックします。

**ヒント:** 番号は認証用SMSを1通受信できれば十分です。その後は、WhatsApp Web のセッションは `creds.json` によって維持されます。

<div id="why-not-twilio">
  ## なぜ Twilio ではないのか？
</div>

* 初期の OpenClaw ビルドでは、Twilio の WhatsApp Business 連携をサポートしていました。
* WhatsApp Business の番号は、個人向けアシスタント用途には不向きです。
* Meta は 24 時間の返信ウィンドウを強制しており、直近 24 時間以内に応答していない場合、そのビジネス番号から新しいメッセージを送って会話を開始することができません。
* 高頻度な利用や「おしゃべり」な使い方をすると、ビジネスアカウントは多数のパーソナルアシスタントメッセージの送信を想定していないため、積極的なブロックが発生します。
* 結果としてメッセージ配信が不安定になり、ブロックも頻発したため、サポート対象から外しました。

<div id="login-credentials">
  ## ログインと認証情報
</div>

* ログインコマンド: `openclaw channels login`（「リンク済み端末」経由のQR）。
* 複数アカウントログイン: `openclaw channels login --account <id>`（`<id>` = `accountId`）。
* デフォルトアカウント（`--account` を省略した場合）: `default` が存在すればそれを使用し、なければ設定済みアカウントIDをソートしたとき最初のもの。
* 認証情報は `~/.openclaw/credentials/whatsapp/<accountId>/creds.json` に保存される。
* `creds.json.bak` にバックアップコピーがあり（破損時にそこから復元される）。
* レガシー互換性: 旧インストールでは Baileys のファイルを `~/.openclaw/credentials/` 直下に保存していた。
* ログアウト: `openclaw channels logout`（または `--account <id>`）は WhatsApp の認証状態を削除する（ただし共有の `oauth.json` は保持される）。
* ログアウト済みソケットの場合、再リンクを指示するエラーが発生する。

<div id="inbound-flow-dm-group">
  ## 受信フロー（DM + グループ）
</div>

* WhatsApp のイベントは `messages.upsert`（Baileys）から届きます。
* テストや再起動時にイベントハンドラが蓄積しないよう、シャットダウン時に inbox リスナーをデタッチします。
* ステータス/ブロードキャストチャットは無視されます。
* 1 対 1 チャットは E.164、グループは group JID を使用します。
* **DM ポリシー**: `channels.whatsapp.dmPolicy` が 1 対 1 チャットへのアクセスを制御します（デフォルト: `pairing`）。
  * Pairing: 未知の送信者にはペアリングコードが発行されます（`openclaw pairing approve whatsapp <code>` で承認します。コードの有効期限は 1 時間です）。
  * open: すべてのユーザーからのメッセージを無制限に受け付ける設定であり、`channels.whatsapp.allowFrom` に `"*"` を含める必要があります。
  * あなたが連携した WhatsApp 番号は暗黙的に信頼されるため、自分自身へのメッセージでは `channels.whatsapp.dmPolicy` および `channels.whatsapp.allowFrom` のチェックはスキップされます。

<div id="personal-number-mode-fallback">
  ### 個人番号モード（フォールバック）
</div>

**自分の WhatsApp 個人番号**で OpenClaw を実行する場合は、`channels.whatsapp.selfChatMode` を有効にします（上記のサンプルを参照）。

動作:

* 送信 DM はペアリング返信を一切トリガーしません（連絡先へのスパム送信を防止）。
* 不明な送信者からの受信メッセージも、`channels.whatsapp.dmPolicy` に従います。
* Self-chat モード（allowFrom に自分の番号が含まれている場合）は、自動の既読レシートを送信せず、メンション JID を無視します。
* 自分とのチャット以外の DM には既読レシートが送信されます。

<div id="read-receipts">
  ## 既読通知
</div>

デフォルトでは、Gateway は受信した WhatsApp メッセージを受け付けた時点で、それらを既読（青いチェックマーク）としてマークします。

グローバルに無効化するには:

```json5
{
  channels: { whatsapp: { sendReadReceipts: false } }
}
```

アカウントごとに無効化:

```json5
{
  channels: {
    whatsapp: {
      accounts: {
        personal: { sendReadReceipts: false }
      }
    }
  }
}
```

注意:

* Self-chat モードでは、既読通知は常に送信されません。

<div id="whatsapp-faq-sending-messages-pairing">
  ## WhatsApp FAQ: メッセージ送信とペアリング
</div>

**WhatsApp をリンクしたら、OpenClaw がランダムな連絡先にメッセージを送ってしまいますか？**\
いいえ。デフォルトの DM ポリシーは **ペアリング** なので、未知の送信者にはペアリングコードだけが返され、そのメッセージは**処理されません**。OpenClaw が返信するのは、受信したチャットか、あなたが明示的にトリガーした送信（エージェント / CLI）のみです。

**WhatsApp でのペアリングはどのように動作しますか？**\
ペアリングは未知の送信者向けの DM ゲートです:

* 新しい送信者からの最初の DM にはショートコードが返されます（メッセージは処理されません）。
* 次のコマンドで承認します: `openclaw pairing approve whatsapp <code>`（一覧表示は `openclaw pairing list whatsapp`）。
* コードは 1 時間で失効し、保留中リクエストはチャネルごとに最大 3 件までです。

**1 つの WhatsApp 番号を、複数人が別々の OpenClaw インスタンスで使えますか？**\
はい。各送信者を `bindings`（`kind: "dm"` の peer、送信者は E.164 形式、例: `+15551234567`）経由で別々のエージェントにルーティングすることで可能です。返信は**同じ WhatsApp アカウント**から行われ、ダイレクトチャットは各エージェントのメインセッションに集約されるため、**1 人につき 1 エージェント**を使用してください。DM アクセス制御（`dmPolicy` / `allowFrom`）は WhatsApp アカウント単位でグローバルです。[Multi-Agent Routing](/ja/concepts/multi-agent) を参照してください。

**ウィザードで電話番号の入力を求めるのはなぜですか？**\
ウィザードはその番号を使ってあなたの **許可リスト / オーナー** を設定し、あなた自身の DM が許可されるようにします。自動送信には使用しません。個人の WhatsApp 番号で実行する場合は、その同じ番号を指定し、`channels.whatsapp.selfChatMode` を有効にしてください。

<div id="message-normalization-what-the-model-sees">
  ## メッセージの正規化（モデルが実際に受け取る内容）
</div>

* `Body` はエンベロープ付きの現在のメッセージ本文です。
* 引用返信コンテキストは**常に末尾に付加されます**:
  ```
  [Replying to +1555 id:ABC123]
  <quoted text or <media:...>>
  [/Replying]
  ```
* 返信メタデータもあわせて設定されます:
  * `ReplyToId` = stanzaId
  * `ReplyToBody` = 引用された本文またはメディアのプレースホルダー
  * `ReplyToSender` = 判明している場合は E.164 形式の番号
* メディアのみの受信メッセージではプレースホルダーが使われます:
  * `<media:image|video|audio|document|sticker>`

<div id="groups">
  ## グループ
</div>

* グループは `agent:<agentId>:whatsapp:group:<jid>` セッションに対応します。
* グループポリシー: `channels.whatsapp.groupPolicy = open|disabled|allowlist`（デフォルトは `allowlist`）。
* アクティベーションモード:
  * `mention`（デフォルト）: @メンションまたは正規表現マッチが必要。
  * `always`: 常にトリガーされる。
* `/activation mention|always` はオーナーのみ実行可能で、単独のメッセージとして送信する必要があります。
* オーナー = `channels.whatsapp.allowFrom`（未設定の場合は自身の E.164）。
* **履歴インジェクション**（保留中のみ）:
  * 最近の*未処理*メッセージ（デフォルト 50 件）が、次の見出しの下に挿入されます:
    `[Chat messages since your last reply - for context]`（すでにセッション内にあるメッセージは再挿入されない）
  * 現在のメッセージは次の見出しの下に挿入されます:
    `[Current message - respond to this]`
  * 送信者情報のサフィックスとして、次の形式が付加されます: `[from: Name (+E164)]`
* グループメタデータは 5 分間キャッシュされます（件名 + 参加者）。

<div id="reply-delivery-threading">
  ## 返信の配信（スレッド）
</div>

* WhatsApp Web は標準メッセージとして送信します（現在の Gateway では、引用返信によるスレッド化は行われません）。
* このチャネルでは返信タグは無視されます。

<div id="acknowledgment-reactions-auto-react-on-receipt">
  ## 受信確認リアクション（受信時の自動リアクション）
</div>

WhatsApp は、ボットが返信を生成する前に、受信メッセージに対して絵文字リアクションを自動で即座に送信できます。これにより、ユーザーに対してメッセージが受信されたことを即座にフィードバックできます。

**設定:**

```json
{
  "whatsapp": {
    "ackReaction": {
      "emoji": "👀",
      "direct": true,
      "group": "mentions"
    }
  }
}
```

**オプション:**

* `emoji` (string): 確認に使用する絵文字（例: &quot;👀&quot;, &quot;✅&quot;, &quot;📨&quot;）。空または未指定の場合はこの機能は無効になります。
* `direct` (boolean, default: `true`): ダイレクト/DM チャットでリアクションを送信するかどうか。
* `group` (string, default: `"mentions"`): グループチャットでの動作:
  * `"always"`: すべてのグループメッセージにリアクションする（@メンションがなくても）
  * `"mentions"`: ボットが @メンションされたときのみリアクションする
  * `"never"`: グループではリアクションしない

**アカウント単位の上書き設定:**

```json
{
  "whatsapp": {
    "accounts": {
      "work": {
        "ackReaction": {
          "emoji": "✅",
          "direct": false,
          "group": "always"
        }
      }
    }
  }
}
```

**動作に関する注意事項:**

* リアクションはメッセージ受信時に**即座に**送信され、タイピングインジケーターやボットの返信よりも先に送られます。
* `requireMention: false`（常時アクティブ）のグループでは、`group: "mentions"` は（@メンションだけでなく）すべてのメッセージにリアクションします。
* 「fire-and-forget」方式: リアクション送信に失敗してもログに記録されるだけで、ボットの返信は妨げられません。
* グループでのリアクションでは Participant JID が自動的に含まれます。
* WhatsApp は `messages.ackReaction` を無視します。代わりに `channels.whatsapp.ackReaction` を使用してください。

<div id="agent-tool-reactions">
  ## エージェントツール（リアクション）
</div>

* ツール: `whatsapp` の `react` アクション（`chatJid`, `messageId`, `emoji`, オプションの `remove`）。
* オプション: `participant`（グループ送信者）、`fromMe`（自分のメッセージへのリアクション）、`accountId`（マルチアカウント）。
* リアクション削除時の挙動: [/tools/reactions](/ja/tools/reactions) を参照。
* ツールのゲーティング: `channels.whatsapp.actions.reactions`（デフォルト: 有効）。

<div id="limits">
  ## 制限
</div>

* 送信テキストは `channels.whatsapp.textChunkLimit`（デフォルト 4000）に基づいてチャンク分割されます。
* 改行単位でのチャンク分割（任意）: 長さによるチャンク分割の前に空行（段落境界）で分割するには、`channels.whatsapp.chunkMode="newline"` を設定します。
* 受信メディアの保存サイズは `channels.whatsapp.mediaMaxMb`（デフォルト 50 MB）で上限が設定されます。
* 送信メディア項目は `agents.defaults.mediaMaxMb`（デフォルト 5 MB）で上限が設定されます。

<div id="outbound-send-text-media">
  ## アウトバウンド送信（テキスト＋メディア）
</div>

* アクティブな Web リスナーを使用します。Gateway が実行中でない場合はエラーになります。
* テキストのチャンク分割: メッセージごとに最大 4k（`channels.whatsapp.textChunkLimit` で設定可能、オプションの `channels.whatsapp.chunkMode`）。
* メディア:
  * 画像 / 動画 / 音声 / ドキュメントをサポート。
  * 音声は PTT として送信されます。`audio/ogg` =&gt; `audio/ogg; codecs=opus`。
  * キャプションは最初のメディア項目にのみ付与されます。
  * メディアの取得は HTTP(S) とローカルパスをサポートします。
  * アニメーション GIF: WhatsApp はインラインループ再生用に `gifPlayback: true` を指定した MP4 を要求します。
    * CLI: `openclaw message send --media <mp4> --gif-playback`
    * Gateway: `send` のパラメータに `gifPlayback: true` を含めます

<div id="voice-notes-ptt-audio">
  ## ボイスメモ（PTT音声）
</div>

WhatsApp は音声を **ボイスメモ**（PTT バブル）として送信します。

* 最適な結果を得るには: OGG/Opus。OpenClaw は `audio/ogg` を `audio/ogg; codecs=opus` に書き換えます。
* `[[audio_as_voice]]` は WhatsApp では無視されます（音声は既にボイスメモとして送信されるため）。

<div id="media-limits-optimization">
  ## メディア制限と最適化
</div>

* デフォルトの送信上限: 5 MB（メディアアイテムごと）。
* 上限の変更: `agents.defaults.mediaMaxMb`。
* 画像は上限内に収まるよう、自動的に JPEG に最適化されます（リサイズ + 画質調整）。
* 上限を超えるメディア =&gt; エラーとなり、メディア返信はテキストでの警告にフォールバックします。

<div id="heartbeats">
  ## ハートビート
</div>

* **Gateway ハートビート** は接続状態の健全性をログに記録します（`web.heartbeatSeconds`、デフォルト 60 秒）。
* **エージェント ハートビート** はエージェントごと（`agents.list[].heartbeat`）またはグローバルに
  `agents.defaults.heartbeat` を介して設定できます（個別エージェントの設定がない場合のフォールバックとして使用）。
  * 設定されたハートビート用プロンプト（デフォルト: `Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.`）と、`HEARTBEAT_OK` によるスキップ挙動を使用します。
  * 配信先のデフォルトは、最後に使用したチャネル（または設定されたターゲット）です。

<div id="reconnect-behavior">
  ## 再接続時の動作
</div>

* バックオフポリシー: `web.reconnect`:
  * `initialMs`, `maxMs`, `factor`, `jitter`, `maxAttempts`.
* `maxAttempts` に達した場合、Web モニタリングは停止（劣化状態）します。
* ログアウト状態 =&gt; 停止し、再リンクが必要になります。

<div id="config-quick-map">
  ## 設定クイックマップ
</div>

* `channels.whatsapp.dmPolicy` (DM ポリシー: pairing/許可リスト/open/disabled)。
* `channels.whatsapp.selfChatMode` (同一電話でのセットアップ。ボットがあなた自身の WhatsApp 番号を使用)。
* `channels.whatsapp.allowFrom` (DM の許可リスト)。WhatsApp は E.164 形式の電話番号を使用します (ユーザー名なし)。
* `channels.whatsapp.mediaMaxMb` (受信メディア保存サイズの上限)。
* `channels.whatsapp.ackReaction` (メッセージ受信時の自動リアクション: `{emoji, direct, group}`)。
* `channels.whatsapp.accounts.<accountId>.*` (アカウントごとの設定 + 任意の `authDir`)。
* `channels.whatsapp.accounts.<accountId>.mediaMaxMb` (アカウントごとの受信メディアサイズ上限)。
* `channels.whatsapp.accounts.<accountId>.ackReaction` (アカウントごとの ack リアクションのオーバーライド)。
* `channels.whatsapp.groupAllowFrom` (グループ送信者の許可リスト)。
* `channels.whatsapp.groupPolicy` (グループポリシー)。
* `channels.whatsapp.historyLimit` / `channels.whatsapp.accounts.<accountId>.historyLimit` (グループ履歴コンテキスト。`0` で無効化)。
* `channels.whatsapp.dmHistoryLimit` (DM の履歴上限 (ユーザーターン数))。ユーザー別オーバーライド: `channels.whatsapp.dms["<phone>"].historyLimit`。
* `channels.whatsapp.groups` (グループ許可リスト + メンション制御のデフォルト値。`"*"` で全許可)。
* `channels.whatsapp.actions.reactions` (WhatsApp ツールのリアクションを制御)。
* `agents.list[].groupChat.mentionPatterns` (または `messages.groupChat.mentionPatterns`)
* `messages.groupChat.historyLimit`
* `channels.whatsapp.messagePrefix` (受信メッセージのプレフィックス。アカウントごと: `channels.whatsapp.accounts.<accountId>.messagePrefix`。非推奨: `messages.messagePrefix`)
* `messages.responsePrefix` (送信メッセージのプレフィックス)
* `agents.defaults.mediaMaxMb`
* `agents.defaults.heartbeat.every`
* `agents.defaults.heartbeat.model` (任意のオーバーライド)
* `agents.defaults.heartbeat.target`
* `agents.defaults.heartbeat.to`
* `agents.defaults.heartbeat.session`
* `agents.list[].heartbeat.*` (エージェントごとのオーバーライド)
* `session.*` (scope、idle、store、mainKey)
* `web.enabled` (false の場合、チャネルの起動を無効化)
* `web.heartbeatSeconds`
* `web.reconnect.*`

<div id="logs-troubleshooting">
  ## ログとトラブルシューティング
</div>

* サブシステム: `whatsapp/inbound`, `whatsapp/outbound`, `web-heartbeat`, `web-reconnect`。
* ログファイル: `/tmp/openclaw/openclaw-YYYY-MM-DD.log`（設定で変更できます）。
* トラブルシューティングガイド: [Gateway のトラブルシューティング](/ja/gateway/troubleshooting)。

<div id="troubleshooting-quick">
  ## トラブルシューティング（クイック）
</div>

**未リンク / QR ログインが必要**

* 症状: `channels status` に `linked: false` と表示されるか、「Not linked」と警告される。
* 対処: Gateway ホスト上で `openclaw channels login` を実行し、QR コードをスキャンする（WhatsApp → Settings → Linked Devices）。

**リンク済みだが切断 / 再接続ループ**

* 症状: `channels status` に `running, disconnected` と表示されるか、「Linked but disconnected」と警告される。
* 対処: `openclaw doctor` を実行する（または Gateway を再起動する）。それでも解消しない場合は、`channels login` で再リンクし、`openclaw logs --follow` を確認する。

**Bun ランタイム**

* Bun は**非推奨**。WhatsApp（Baileys）および Telegram は Bun 上では信頼性が低い。
  Gateway は**Node** で実行すること。（「Getting Started」のランタイムに関する注意を参照。）