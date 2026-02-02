---
title: アーキテクチャ
summary: "WebSocket Gateway のアーキテクチャ、コンポーネント、およびクライアント フロー"
read_when:
  - Gateway のプロトコル、クライアント、またはトランスポートに関する作業をしているとき
---

<div id="gateway-architecture">
  # Gateway アーキテクチャ
</div>

最終更新日: 2026-01-22

<div id="overview">
  ## 概要
</div>

* 単一の長期間稼働する **Gateway** が、すべてのメッセージングチャネル（Baileys 経由の WhatsApp、grammY 経由の Telegram、Slack、Discord、Signal、iMessage、WebChat）を一元管理します。
* コントロールプレーンのクライアント（macOS アプリ、CLI、Web UI、自動化）は、設定されたバインドホストの **WebSocket**（デフォルトは `127.0.0.1:18789`）経由で Gateway に接続します。
* **ノード**（macOS/iOS/Android/ヘッドレス）も **WebSocket** 経由で接続しますが、明示的な機能／コマンドとともに `role: node` を宣言します。
* 各ホストにつき Gateway は 1 つだけで、WhatsApp セッションを開くのはその Gateway のみです。
* **キャンバスホスト**（デフォルト `18793`）が、エージェントが編集可能な HTML と A2UI を配信します。

<div id="components-and-flows">
  ## コンポーネントと処理フロー
</div>

<div id="gateway-daemon">
  ### Gateway (デーモン)
</div>

* プロバイダーとの接続を維持する。
* 型付きの WS API（リクエスト、レスポンス、サーバープッシュイベント）を提供する。
* 受信フレームを JSON Schema に対して検証する。
* `agent`、`chat`、`presence`、`health`、`heartbeat`、`cron` などのイベントを送出する。

<div id="clients-mac-app-cli-web-admin">
  ### クライアント（mac アプリ / CLI / Web 管理画面）
</div>

* クライアントごとに 1 つの WS 接続。
* リクエストを送信（`health`、`status`、`send`、`agent`、`system-presence`）。
* イベントをサブスクライブ（`tick`、`agent`、`presence`、`shutdown`）。

<div id="nodes-macos-ios-android-headless">
  ### ノード (macOS / iOS / Android / ヘッドレス)
</div>

* `role: node` を指定して、**同じ WS サーバー** に接続します。
* `connect` でデバイス ID を指定します。ペアリングは **デバイス単位**（ロールは `node`）で行われ、
  承認はデバイスのペアリングストアに保存されます。
* `canvas.*`、`camera.*`、`screen.record`、`location.get` のようなコマンドを提供します。

プロトコルの詳細:

* [Gateway protocol](/ja/gateway/protocol)

<div id="webchat">
  ### WebChat
</div>

* チャット履歴の取得と送信のために Gateway WS API を使用する静的な UI。
* リモート環境では、他のクライアントと同じ SSH/Tailscale トンネル経由で接続する。

<div id="connection-lifecycle-single-client">
  ## 接続ライフサイクル（単一クライアントの場合）
</div>

```
Client                    Gateway
  |                          |
  |---- req:connect -------->|
  |<------ res (ok) ---------|   (or res error + close)
  |   (payload=hello-ok carries snapshot: presence + health)
  |                          |
  |<------ event:presence ---|
  |<------ event:tick -------|
  |                          |
  |------- req:agent ------->|
  |<------ res:agent --------|   (ack: {runId,status:"accepted"})
  |<------ event:agent ------|   (streaming)
  |<------ res:agent --------|   (final: {runId,status,summary})
  |                          |
```

<div id="wire-protocol-summary">
  ## ワイヤプロトコル（概要）
</div>

* トランスポート: WebSocket（JSON ペイロードを含むテキストフレーム）。
* 最初のフレームは必ず `connect` である必要がある。
* ハンドシェイク後:
  * リクエスト: `{type:"req", id, method, params}` → `{type:"res", id, ok, payload|error}`
  * イベント: `{type:"event", event, payload, seq?, stateVersion?}`
* `OPENCLAW_GATEWAY_TOKEN`（または `--token`）が設定されている場合、
  `connect.params.auth.token` が一致しないとソケットはクローズされる。
* 副作用を伴うメソッド（`send`、`agent`）を安全に再試行するために、冪等性キーが必須となる。
  サーバーは短寿命の重複排除キャッシュを保持する。
* ノードは `connect` に `role: "node"` を含め、そのうえで対応する機能／コマンド／権限を宣言しなければならない。

<div id="pairing-local-trust">
  ## ペアリングとローカル信頼
</div>

* すべての WS クライアント（オペレーター + ノード）は、`connect` 時に **デバイス識別子（device identity）** を含めます。
* 新しいデバイス ID にはペアリングによる承認が必要で、Gateway は以後の接続用に **デバイストークン** を発行します。
* **ローカル** 接続（ループバックまたは Gateway ホスト自身の tailnet アドレス）は、
  同一ホスト上での UX をスムーズに保つため自動承認できます。
* **非ローカル** 接続は `connect.challenge` ノンスに署名する必要があり、明示的な承認が必要です。
* Gateway 認証（`gateway.auth.*`）は、ローカル・リモートを問わず **すべて** の接続に適用されます。

詳細: [Gateway protocol](/ja/gateway/protocol), [Pairing](/ja/start/pairing),
[Security](/ja/gateway/security).

<div id="protocol-typing-and-codegen">
  ## プロトコルの型定義とコード生成
</div>

* TypeBox スキーマがプロトコルを定義します。
* そのスキーマから JSON Schema が生成されます。
* JSON Schema から Swift モデルが生成されます。

<div id="remote-access">
  ## リモートアクセス
</div>

* 推奨: Tailscale または VPN。
* 代替: SSH トンネル
  ```bash
  ssh -N -L 18789:127.0.0.1:18789 user@host
  ```
* 同じハンドシェイクと認証トークンがトンネル経由でも有効です。
* リモート環境では、WS に対して TLS とオプションの証明書ピンニングを有効化できます。

<div id="operations-snapshot">
  ## 運用スナップショット
</div>

* 起動: `openclaw gateway`（フォアグラウンドで実行し、stdout にログ出力）。
* ヘルスチェック: WS 経由の `health`（`hello-ok` にも含まれる）。
* プロセス監視: 自動再起動のために launchd/systemd を使用。

<div id="invariants">
  ## 不変条件
</div>

* 各ホスト上では、1つの Gateway が 1つの Baileys セッションのみを制御する。
* ハンドシェイクは必須であり、最初のフレームが JSON でも connect でもない場合は即座に強制切断する。
* イベントは再送・再配信されないため、ギャップが生じた場合はクライアントがリフレッシュしなければならない。