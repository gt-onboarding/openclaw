---
title: Gmail Pub/Sub
summary: "Gmail Pub/Sub の push 配信を gogcli 経由で OpenClaw の webhook に接続する"
read_when:
  - Gmail 受信トレイのトリガーを OpenClaw に接続したいとき
  - エージェントの起動用に Pub/Sub の push 配信を設定するとき
---

<div id="gmail-pubsub-openclaw">
  # Gmail Pub/Sub -&gt; OpenClaw
</div>

目的: Gmail watch -&gt; Pub/Sub push -&gt; `gog gmail watch serve` -&gt; OpenClaw webhook。

<div id="prereqs">
  ## 前提条件
</div>

* `gcloud` がインストールされ、ログイン済みであること（[インストールガイド](https://docs.cloud.google.com/sdk/docs/install-sdk)を参照）。
* `gog` (gogcli) がインストールされ、対象の Gmail アカウントに対して認可済みであること（[gogcli.sh](https://gogcli.sh/) を参照）。
* OpenClaw hooks が有効化されていること（[Webhooks](/ja/automation/webhook) を参照）。
* `tailscale` にログイン済みであること（[tailscale.com](https://tailscale.com/)）。サポートされている構成では、パブリックな HTTPS エンドポイントとして Tailscale Funnel を使用する。
  他のトンネルサービスでも動作する可能性はあるが、自己責任・サポート対象外であり、手動での設定が必要となる。
  現時点では、Tailscale のみをサポートしている。

フック設定の例（Gmail プリセットマッピングを有効化）:

```json5
{
  hooks: {
    enabled: true,
    token: "OPENCLAW_HOOK_TOKEN",
    path: "/hooks",
    presets: ["gmail"]
  }
}
```

Gmail のサマリーをチャット画面に配信するには、`deliver` と任意の `channel` / `to` を設定するマッピングを指定して、プリセットを上書きします。

```json5
{
  hooks: {
    enabled: true,
    token: "OPENCLAW_HOOK_TOKEN",
    presets: ["gmail"],
    mappings: [
      {
        match: { path: "gmail" },
        action: "agent",
        wakeMode: "now",
        name: "Gmail",
        sessionKey: "hook:gmail:{{messages[0].id}}",
        messageTemplate:
          "New email from {{messages[0].from}}\nSubject: {{messages[0].subject}}\n{{messages[0].snippet}}\n{{messages[0].body}}",
        model: "openai/gpt-5.2-mini",
        deliver: true,
        channel: "last"
        // to: "+15551234567"
      }
    ]
  }
}
```

特定のチャネルを固定で使いたい場合は、`channel` と `to` を設定してください。そうでない場合は `channel: "last"` を使うと、
最後に使われた配信ルートが利用されます（WhatsApp がフォールバック先になります）。

Gmail 実行時の処理で、より安価なモデルを強制したい場合は、マッピング内で `model`
（`provider/model` またはエイリアス）を設定してください。`agents.defaults.models` を強制している場合は、そこにも含めてください。

Gmail フック専用にデフォルトのモデルと thinking レベルを設定するには、
設定ファイルに `hooks.gmail.model` / `hooks.gmail.thinking` を追加します：

```json5
{
  hooks: {
    gmail: {
      model: "openrouter/meta-llama/llama-3.3-70b-instruct:free",
      thinking: "off" // 思考モードを無効化
    }
  }
}
```

Notes:

* マッピング内のフックごとの `model` / `thinking` 設定は、引き続きこれらのデフォルトを上書きします。
* フォールバック順序: `hooks.gmail.model` → `agents.defaults.model.fallbacks` → メイン (認証 / レート制限 / タイムアウト)。
* `agents.defaults.models` が設定されている場合、Gmail で使用するモデルは許可リストに含まれている必要があります。
* Gmail フックのコンテンツは、デフォルトで外部コンテンツ用のセーフティ境界でラップされます。
  これを無効化するには (危険)、`hooks.gmail.allowUnsafeExternalContent: true` を設定します。

ペイロード処理をさらにカスタマイズするには、`hooks.mappings` を追加するか、
`hooks.transformsDir` 配下に JS/TS の変換モジュールを配置します ([Webhooks](/ja/automation/webhook) を参照)。

<div id="wizard-recommended">
  ## ウィザード（推奨）
</div>

OpenClaw のヘルパーを使って一括設定を行います（macOS では Homebrew（brew）経由で依存関係をインストールします）:

```bash
openclaw webhooks gmail setup \
  --account openclaw@gmail.com
```

デフォルト:

* 公開用 push エンドポイントとして Tailscale Funnel を使用します。
* `openclaw webhooks gmail run` 用の `hooks.gmail` 設定を書き出します。
* Gmail フックプリセットを有効化します（`hooks.presets: ["gmail"]`）。

パスに関する注意事項: `tailscale.mode` が有効な場合、OpenClaw は自動的に
`hooks.gmail.serve.path` を `/` に設定し、公開パスは
`hooks.gmail.tailscale.path`（デフォルト `/gmail-pubsub`）のままにします。これは、Tailscale がプロキシする前に設定されたパスプレフィックスを取り除くためです。
プレフィックス付きパスをバックエンド側で受け取りたい場合は、
`hooks.gmail.tailscale.target`（または `--tailscale-target`）を
`http://127.0.0.1:8788/gmail-pubsub` のようなフル URL に設定し、
`hooks.gmail.serve.path` をそれに合わせてください。

独自のエンドポイントを使いたい場合は、`--push-endpoint <url>` または `--tailscale off` を使用してください。

プラットフォームに関する注意: macOS ではウィザードが Homebrew 経由で
`gcloud`、`gogcli`、`tailscale` をインストールします。Linux では先にそれらを手動でインストールしてください。

Gateway の自動起動（推奨）:

* `hooks.enabled=true` かつ `hooks.gmail.account` が設定されている場合、Gateway は起動時に
  `gog gmail watch serve` を開始し、watch を自動更新します。
* オプトアウトするには `OPENCLAW_SKIP_GMAIL_WATCHER=1` を設定します（自分でデーモンを動かす場合に有用です）。
* 手動デーモンを同時に実行しないでください。同時に動かすと
  `listen tcp 127.0.0.1:8788: bind: address already in use`
  というエラーになります。

手動デーモン（`gog gmail watch serve` の起動 + 自動更新）:

```bash
openclaw webhooks gmail run
```

<div id="one-time-setup">
  ## 初回セットアップ
</div>

1. `gog` が利用する **OAuth クライアントの所有元である** GCP プロジェクトを選択します。

```bash
gcloud auth login
gcloud config set project <project-id>
```

注記: Gmail ウォッチ機能では、Pub/Sub トピックは OAuth クライアントと同じプロジェクト内に存在している必要があります。

2. API を有効にします:

```bash
gcloud services enable gmail.googleapis.com pubsub.googleapis.com
```

3. トピックを作成する：

```bash
gcloud pubsub topics create gog-gmail-watch
```

4. Gmail プッシュによる公開を許可する:

```bash
gcloud pubsub topics add-iam-policy-binding gog-gmail-watch \
  --member=serviceAccount:gmail-api-push@system.gserviceaccount.com \
  --role=roles/pubsub.publisher
```

<div id="start-the-watch">
  ## ウォッチを開始する
</div>

```bash
gog gmail watch start \
  --account openclaw@gmail.com \
  --label INBOX \
  --topic projects/<project-id>/topics/gog-gmail-watch
```

出力に含まれる `history_id` を（デバッグ用に）保存しておいてください。

<div id="run-the-push-handler">
  ## プッシュハンドラーを実行する
</div>

ローカル環境での例（共有トークン認証）:

```bash
gog gmail watch serve \
  --account openclaw@gmail.com \
  --bind 127.0.0.1 \
  --port 8788 \
  --path /gmail-pubsub \
  --token <shared> \
  --hook-url http://127.0.0.1:18789/hooks/gmail \
  --hook-token OPENCLAW_HOOK_TOKEN \
  --include-body \
  --max-bytes 20000
```

注意事項:

* `--token` はプッシュエンドポイント（`x-gog-token` または `?token=`）を保護します。
* `--hook-url` は OpenClaw の `/hooks/gmail` を指します（マッピング済み。分離された実行 + メインへの要約）。
* `--include-body` と `--max-bytes` は、OpenClaw に送信される本文スニペットを制御します。

推奨: `openclaw webhooks gmail run` は同じフローをラップし、watch を自動更新します。

<div id="expose-the-handler-advanced-unsupported">
  ## ハンドラーを公開する（上級者向け・サポート対象外）
</div>

Tailscale 以外のトンネルが必要な場合は、自分でトンネルを設定し、push サブスクリプションでパブリック URL を使用してください（サポート対象外・安全策なし）:

```bash
cloudflared tunnel --url http://127.0.0.1:8788 --no-autoupdate
```

生成された URL をプッシュエンドポイントとして使用します：

```bash
gcloud pubsub subscriptions create gog-gmail-watch-push \
  --topic gog-gmail-watch \
  --push-endpoint "https://<public-url>/gmail-pubsub?token=<shared>"
```

本番環境では、安定した HTTPS エンドポイントを使用し、Pub/Sub OIDC JWT を設定してから、次のコマンドを実行します:

```bash
gog gmail watch serve --verify-oidc --oidc-email <svc@...>
```

<div id="test">
  ## テスト
</div>

監視対象の受信トレイ宛てにメッセージを送信します：

```bash
gog gmail send \
  --account openclaw@gmail.com \
  --to openclaw@gmail.com \
  --subject "watch test" \
  --body "ping"
```

ウォッチの状態と履歴を確認する：

```bash
gog gmail watch status --account openclaw@gmail.com
gog gmail history --account openclaw@gmail.com --since <historyId>
```

<div id="troubleshooting">
  ## トラブルシューティング
</div>

* `Invalid topicName`: プロジェクトの不一致（トピックが OAuth クライアントのプロジェクトに存在しない）。
* `User not authorized`: ユーザーにトピックの `roles/pubsub.publisher` ロールが付与されていない。
* 空のメッセージ: Gmail のプッシュは `historyId` のみを提供するため、`gog gmail history` で取得する。

<div id="cleanup">
  ## クリーンアップ
</div>

```bash
gog gmail watch stop --account openclaw@gmail.com
gcloud pubsub subscriptions delete gog-gmail-watch-push
gcloud pubsub topics delete gog-gmail-watch
```
