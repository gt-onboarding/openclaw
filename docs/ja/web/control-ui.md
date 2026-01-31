---
title: Control UI
summary: "Gateway 用のブラウザベースの Control UI（チャット、ノード、設定）"
read_when:
  - Gateway をブラウザから操作したいとき
  - SSH トンネルなしで Tailnet へアクセスしたいとき
---

<div id="control-ui-browser">
  # Control UI (ブラウザ)
</div>

Control UI は、Gateway によって提供される小さな **Vite + Lit** 製のシングルページアプリです：

* デフォルト: `http://<host>:18789/`
* 任意のプレフィックス: `gateway.controlUi.basePath` を設定 (例: `/openclaw`)

同じポートで **Gateway の WebSocket** と直接通信します。

<div id="quick-open-local">
  ## クイックオープン（ローカル）
</div>

Gateway が同じコンピュータ上で動作している場合は、次を開きます:

* http://127.0.0.1:18789/ （または http://localhost:18789/）

ページが読み込めない場合は、先に Gateway を起動してください: `openclaw gateway`。

認証情報は WebSocket ハンドシェイク時に次のパラメータで送信されます:

* `connect.params.auth.token`
* `connect.params.auth.password`
  ダッシュボードの設定パネルでは、トークンを保存できます。パスワードは永続化されません。
  オンボーディングウィザードはデフォルトで Gateway トークンを生成するため、初回接続時にここに貼り付けてください。

<div id="what-it-can-do-today">
  ## 現時点でできること
</div>

* Gateway の WS 経由でモデルとチャット（`chat.history`、`chat.send`、`chat.abort`、`chat.inject`）
* チャット内でのツール呼び出しストリーミング + ライブツール出力カード（エージェントイベント）
* チャンネル: WhatsApp/Telegram/Discord/Slack + プラグインチャンネル（Mattermost など）のステータス表示 + QR ログイン + チャンネルごとの設定（`channels.status`、`web.login.*`、`config.patch`）
* インスタンス: プレゼンスリスト表示 + 更新（`system-presence`）
* セッション: 一覧表示 + セッション単位での thinking/verbose の上書き設定（`sessions.list`、`sessions.patch`）
* Cron ジョブ: 一覧/追加/実行/有効化/無効化 + 実行履歴（`cron.*`）
* スキル: ステータス確認、有効化/無効化、インストール、API キー更新（`skills.*`）
* ノード: 一覧 + capabilities の取得（`node.list`）
* Exec 承認: Gateway またはノードの許可リスト編集 + `exec host=gateway/node` に対するポリシー問い合わせ（`exec.approvals.*`）
* 設定: `~/.openclaw/openclaw.json` の表示/編集（`config.get`、`config.set`）
* 設定: バリデーション付きで適用 + 再起動（`config.apply`）、および最後にアクティブだったセッションのウェイクアップ
* 設定の書き込み時には、同時編集の上書きを防ぐための base-hash ガードを含める
* 設定スキーマ + フォームレンダリング（プラグイン + チャンネルスキーマを含む `config.schema`）；生の JSON エディタも引き続き利用可能
* デバッグ: status/health/models のスナップショット + イベントログ + 手動 RPC 呼び出し（`status`、`health`、`models.list`）
* ログ: Gateway ファイルログのライブ tail（追尾）表示（フィルタ/エクスポート付き）（`logs.tail`）
* 更新: パッケージ/git アップデートの実行 + 再起動（`update.run`）、および再起動レポートの表示

<div id="chat-behavior">
  ## チャットの挙動
</div>

* `chat.send` は **ノンブロッキング**です。`{ runId, status: "started" }` をすぐに ACK し、応答は `chat` イベント経由でストリーミングされます。
* 同じ `idempotencyKey` で再送すると、実行中は `{ status: "in_flight" }` を、完了後は `{ status: "ok" }` を返します。
* `chat.inject` は、セッションのトランスクリプトにアシスタントメモを追記し、UI だけを更新するための `chat` イベントをブロードキャストします（エージェントの実行やチャネル配信は行われません）。
* 停止:
  * **Stop** をクリック（`chat.abort` を呼び出し）
  * `/stop`（または `stop|esc|abort|wait|exit|interrupt`）と入力して、通常フローの外側から中断
  * `chat.abort` は `{ sessionKey }`（`runId` なし）をサポートしており、そのセッション内のすべてのアクティブな実行を中断します

<div id="tailnet-access-recommended">
  ## Tailnet アクセス（推奨）
</div>

<div id="integrated-tailscale-serve-preferred">
  ### Tailscale Serve の統合（推奨）
</div>

Gateway をループバックアドレス上で稼働させ、Tailscale Serve に HTTPS でプロキシさせます。

```bash
openclaw gateway --tailscale serve
```

Open:

* `https://<magicdns>/`（または設定した `gateway.controlUi.basePath`）

デフォルトでは、`gateway.auth.allowTailscale` が `true` のとき、Serve リクエストは
Tailscale のアイデンティティヘッダー（`tailscale-user-login`）で認証できます。OpenClaw は
`x-forwarded-for` アドレスを `tailscale whois` で解決し、その結果をヘッダーと照合して
アイデンティティを検証し、かつリクエストが Tailscale の `x-forwarded-*` ヘッダー付きで
ループバックに到達した場合にのみそのリクエストを受け入れます。Serve トラフィックに対しても
トークン／パスワードを必須にしたい場合は、`gateway.auth.allowTailscale: false`
（あるいは `gateway.auth.mode: "password"` を強制）に設定してください。

<div id="bind-to-tailnet-token">
  ### tailnet とトークンの関連付け
</div>

```bash
openclaw gateway --bind tailnet --token "$(openssl rand -hex 32)"
```

次に、以下を開きます:

* `http://<tailscale-ip>:18789/`（または設定済みの `gateway.controlUi.basePath`）

トークンを UI 設定に貼り付けます（`connect.params.auth.token` として送信されます）。

<div id="insecure-http">
  ## 安全でない HTTP
</div>

ダッシュボードをプレーンな HTTP（`http://<lan-ip>` または `http://<tailscale-ip>`）で開くと、
ブラウザは **非セキュアなコンテキスト** で動作し、WebCrypto をブロックします。デフォルトでは、
OpenClaw はデバイスのアイデンティティがない Control UI への接続を **ブロック** します。

**推奨される対処方法:** HTTPS（Tailscale Serve）を使用するか、UI をローカルで開いてください:

* `https://<magicdns>/`（Serve）
* `http://127.0.0.1:18789/`（Gateway ホスト上）

**ダウングレード例（HTTP 上でトークンのみを使用する場合）:**

```json5
{
  gateway: {
    controlUi: { allowInsecureAuth: true },
    bind: "tailnet",
    auth: { mode: "token", token: "replace-me" }
  }
}
```

これは Control UI のデバイス ID とペアリングを（HTTPS 上でも）無効にします。
信頼できるネットワーク環境でのみ使用してください。

HTTPS 設定方法については [Tailscale](/ja/gateway/tailscale) を参照してください。

<div id="building-the-ui">
  ## UI のビルド
</div>

Gateway は `dist/control-ui` から静的ファイルを配信します。次のコマンドでビルドしてください:

```bash
pnpm ui:build # 初回実行時にUI依存関係を自動インストール
```

オプションの絶対ベース URL（アセットの URL を固定したい場合）:

```bash
OPENCLAW_CONTROL_UI_BASE_PATH=/openclaw/ pnpm ui:build
```

ローカル開発向け（別の開発サーバーを使用）:

```bash
pnpm ui:dev # 初回実行時にUI依存関係を自動インストール
```

次に、UI を Gateway の WS URL（例: `ws://127.0.0.1:18789`）に設定します。

<div id="debuggingtesting-dev-server-remote-gateway">
  ## デバッグ/テスト: dev サーバー + リモート Gateway
</div>

Control UI は静的ファイルであり、WebSocket の接続先は設定可能で、HTTP オリジンとは別の値に設定できます。これは、ローカルでは Vite の dev サーバーを使いつつ、Gateway を別の環境で動かしたい場合に便利です。

1. UI の dev サーバーを起動します: `pnpm ui:dev`
2. 次のような URL を開きます:

```text
http://localhost:5173/?gatewayUrl=ws://<gateway-host>:18789
```

必要に応じて一度だけ実施する認証:

```text
http://localhost:5173/?gatewayUrl=wss://<gateway-host>:18789&token=<gateway-token>
```

注意:

* `gatewayUrl` は読み込み後に localStorage に保存され、URL からは削除されます。
* `token` は localStorage に保存され、`password` はメモリ上にのみ保持されます。
* Gateway が TLS（Tailscale Serve、HTTPS プロキシなど）で保護されている場合は `wss://` を使用してください。

リモートアクセスのセットアップの詳細については、[Remote access](/ja/gateway/remote) を参照してください。
