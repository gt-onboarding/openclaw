---
title: Webhook
summary: "ウェイクおよび分離エージェント実行のための Webhook 受信"
read_when:
  - Webhook エンドポイントを追加または変更するとき
  - 外部システムを OpenClaw に接続するとき
---

<div id="webhooks">
  # Webhooks
</div>

Gateway は、外部トリガーを受け取るための小さな HTTP webhook エンドポイントを公開できます。

<div id="enable">
  ## 有効化
</div>

```json5
{
  hooks: {
    enabled: true,
    token: "shared-secret",
    path: "/hooks"
  }
}
```

Notes:

* `hooks.token` は `hooks.enabled` が `true` の場合に必須です。
* `hooks.path` のデフォルトは `/hooks` です。

<div id="auth">
  ## 認証
</div>

すべてのリクエストには hook トークンを含める必要があります。HTTP ヘッダーでの指定を推奨します：

* `Authorization: Bearer <token>`（推奨）
* `x-openclaw-token: <token>`
* `?token=<token>`（非推奨。警告がログに出力され、将来のメジャーリリースで削除される予定です）

<div id="endpoints">
  ## エンドポイント
</div>

<div id="post-hookswake">
  ### `POST /hooks/wake`
</div>

リクエストペイロード:

```json
{ "text": "System line", "mode": "now" }
```

* `text` **必須** (string): イベントの説明（例: &quot;New email received&quot;）。
* `mode` 任意 (`now` | `next-heartbeat`): すぐにハートビートをトリガーするか（デフォルトは `now`）、次の定期チェックまで待機するかを指定します。

Effect:

* **main** セッション向けのシステムイベントをキューに投入します
* `mode=now` の場合、即時にハートビートをトリガーします

<div id="post-hooksagent">
  ### `POST /hooks/agent`
</div>

リクエストペイロード：

```json
{
  "message": "Run this",
  "name": "Email",
  "sessionKey": "hook:email:msg-123",
  "wakeMode": "now",
  "deliver": true,
  "channel": "last",
  "to": "+15551234567",
  "model": "openai/gpt-5.2-mini",
  "thinking": "low",
  "timeoutSeconds": 120
}
```

* `message` **必須** (string): エージェントが処理するプロンプトまたはメッセージ。
* `name` 任意 (string): フックの人間可読な名前 (例: &quot;GitHub&quot;)。セッション要約内でプレフィックスとして使用されます。
* `sessionKey` 任意 (string): エージェントのセッションを識別するために使用されるキー。デフォルトはランダムな `hook:<uuid>`。同じキーを使い続けることで、フックのコンテキスト内でマルチターンの会話が可能になります。
* `wakeMode` 任意 (`now` | `next-heartbeat`): すぐにハートビートをトリガーするか (デフォルトは `now`)、次の定期チェックまで待機するかを指定します。
* `deliver` 任意 (boolean): `true` の場合、エージェントの応答はメッセージングチャネルに送信されます。デフォルトは `true`。ハートビートの受信確認のみの応答は自動的にスキップされます。
* `channel` 任意 (string): 配信に使用するメッセージングチャネル。`last`, `whatsapp`, `telegram`, `discord`, `slack`, `mattermost` (プラグイン), `signal`, `imessage`, `msteams` のいずれか。デフォルトは `last`。
* `to` 任意 (string): チャネルの送信先識別子 (例: WhatsApp/Signal の電話番号、Telegram のチャット ID、Discord/Slack/Mattermost (プラグイン) のチャンネル ID、MS Teams の会話 ID)。デフォルトはメインセッションの最後の送信先。
* `model` 任意 (string): モデルの上書き指定 (例: `anthropic/claude-3-5-sonnet` またはエイリアス)。制限されている場合は、許可されたモデル一覧に含まれている必要があります。
* `thinking` 任意 (string): `thinking` レベルの上書き指定 (例: `low`, `medium`, `high`)。
* `timeoutSeconds` 任意 (number): エージェントの実行に許容される最大時間 (秒)。

効果:

* **隔離された** エージェントの 1 ターンを実行します (独自のセッションキー)
* 常に **メイン** セッションに要約を投稿します
* `wakeMode=now` の場合、即時のハートビートをトリガーします

<div id="post-hooksname-mapped">
  ### `POST /hooks/<name>` (mapped)
</div>

カスタムフック名は `hooks.mappings`（設定を参照）によって解決されます。マッピングによって、任意のペイロードを `wake` または `agent` アクションに変換でき、テンプレートやコード変換を任意で組み合わせられます。

マッピングオプション（概要）:

* `hooks.presets: ["gmail"]` は組み込みの Gmail マッピングを有効にします。
* `hooks.mappings` では、`match`、`action`、テンプレートを設定内で定義できます。
* `hooks.transformsDir` + `transform.module` は、カスタムロジック用の JS/TS モジュールを読み込みます。
* `match.source` を使うと、汎用的な取り込みエンドポイントを維持しつつ（ペイロード駆動のルーティング）、振り分けが可能です。
* TS での変換には TS ローダー（例: `bun` または `tsx`）か、実行時に事前コンパイルされた `.js` が必要です。
* マッピングで `deliver: true` と `channel`/`to` を設定すると、応答をチャット面（チャット画面）にルーティングできます
  （`channel` のデフォルトは `last` で、WhatsApp にフォールバックします）。
* `allowUnsafeExternalContent: true` を設定すると、そのフックに対する外部コンテンツ用セーフティラッパーが無効化されます
  （危険です。信頼できる社内ソースにのみ使用してください）。
* `openclaw webhooks gmail setup` は、`openclaw webhooks gmail run` 用の `hooks.gmail` 設定を書き込みます。
  Gmail の完全なウォッチフローについては [Gmail Pub/Sub](/ja/automation/gmail-pubsub) を参照してください。

<div id="responses">
  ## レスポンス
</div>

* `/hooks/wake` に対しては `200`
* `/hooks/agent` に対しては `202`（非同期実行の開始時）
* 認証エラー時は `401`
* 無効なペイロードの場合は `400`
* ペイロードが大きすぎる場合は `413`

<div id="examples">
  ## 使用例
</div>

```bash
curl -X POST http://127.0.0.1:18789/hooks/wake \
  -H 'Authorization: Bearer SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"text":"New email received","mode":"now"}'
```

```bash
curl -X POST http://127.0.0.1:18789/hooks/agent \
  -H 'x-openclaw-token: SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"message":"Summarize inbox","name":"Email","wakeMode":"next-heartbeat"}'
```

<div id="use-a-different-model">
  ### 別のモデルを使用する
</div>

その実行で使用するモデルを上書きするために、`model` キーをエージェントのペイロード（またはマッピング）に追加します。

```bash
curl -X POST http://127.0.0.1:18789/hooks/agent \
  -H 'x-openclaw-token: SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"message":"Summarize inbox","name":"Email","model":"openai/gpt-5.2-mini"}'
```

`agents.defaults.models` を強制適用している場合は、上書き用のモデルが必ずその中に含まれていることを確認してください。

```bash
curl -X POST http://127.0.0.1:18789/hooks/gmail \
  -H 'Authorization: Bearer SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"source":"gmail","messages":[{"from":"Ada","subject":"Hello","snippet":"Hi"}]}'
```

<div id="security">
  ## セキュリティ
</div>

* フックのエンドポイントは、ループバック、Tailnet、または信頼できるリバースプロキシの背後に置くこと。
* 専用のフック用トークンを使用し、Gateway の認証トークンを使い回さないこと。
* Webhook のログに機微な生のペイロードを含めないこと。
* フックのペイロードは信頼されない入力として扱われ、デフォルトで安全な境界でラップされる。
  特定のフックに対してこれを無効化する必要がある場合は、そのフックのマッピング内で
  `allowUnsafeExternalContent: true` を設定すること（危険）。