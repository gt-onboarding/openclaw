---
title: ダッシュボード
summary: "Gateway ダッシュボード（Control UI）へのアクセスと認証"
read_when:
  - ダッシュボードの認証または公開設定を変更するとき
---

<div id="dashboard-control-ui">
  # Dashboard (Control UI)
</div>

Gateway のダッシュボードは、デフォルトでは `/` で提供されるブラウザベースの Control UI です
（`gateway.controlUi.basePath` で変更可能）。

クイックアクセス（ローカル Gateway）:

* http://127.0.0.1:18789/ （または http://localhost:18789/）

主な参照先:

* [Control UI](/ja/web/control-ui): 使い方と UI 機能。
* [Tailscale](/ja/gateway/tailscale): Serve/Funnel の自動化。
* [Web surfaces](/ja/web): バインドモードとセキュリティに関する注意点。

認証は WebSocket ハンドシェイク時の `connect.params.auth`
（トークンまたはパスワード）で必須となります。`gateway.auth` については [Gateway configuration](/ja/gateway/configuration) を参照してください。

セキュリティ上の注意: Control UI は **管理用のサーフェス** です（チャット、設定、実行承認）。
インターネット上に公開しないでください。UI は初回ロード後、トークンを `localStorage` に保存します。
localhost、Tailscale Serve、または SSH トンネル経由での利用を推奨します。

<div id="fast-path-recommended">
  ## 高速パス（推奨）
</div>

* オンボーディング後、CLI はトークン付きのダッシュボードを自動的に開き、同じトークン付きリンクも出力します。
* いつでも再度開くには: `openclaw dashboard` （リンクをコピーし、可能であればブラウザを開き、ヘッドレス環境では SSH ヒントを表示）。
* トークンはローカルにのみ保持されます（クエリパラメータとしてのみ）。UI は初回ロード後にクエリパラメータからトークンを削除し、localStorage に保存します。

<div id="token-basics-local-vs-remote">
  ## トークンの基本（ローカル vs リモート）
</div>

* **Localhost**: `http://127.0.0.1:18789/` を開きます。`"unauthorized"` と表示された場合は、`openclaw dashboard` を実行し、トークン付きリンク（`?token=...`）を使用します。
* **Token source**: `gateway.auth.token`（または `OPENCLAW_GATEWAY_TOKEN`）。UI は初回ロード時にこの値を保存します。
* **Not localhost**: Tailscale Serve（`gateway.auth.allowTailscale: true` の場合はトークン不要）、トークン付きでの tailnet bind、または SSH トンネルを使用します。[Web surfaces](/ja/web) を参照してください。

<div id="if-you-see-unauthorized-1008">
  ## 「unauthorized」 / 1008 と表示される場合
</div>

* `openclaw dashboard` を実行して、新しいトークン付きリンクを取得します。
* Gateway にアクセス可能であることを確認します（ローカル: `openclaw status`、リモート: SSH トンネル `ssh -N -L 18789:127.0.0.1:18789 user@host` を確立してから `http://127.0.0.1:18789/?token=...` を開く）。
* ダッシュボードの設定で、`gateway.auth.token`（または `OPENCLAW_GATEWAY_TOKEN`）に設定したものと同じトークンを貼り付けます。