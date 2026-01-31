---
title: Web
summary: "Gateway の Web インターフェイス: Control UI、バインドモード、セキュリティ"
read_when:
  - Tailscale 経由で Gateway にアクセスしたいとき
  - ブラウザでの Control UI 利用と設定編集を行いたいとき
---

<div id="web-gateway">
  # Web (Gateway)
</div>

Gateway は、小さな **ブラウザベースの Control UI**（Vite + Lit）を Gateway の WebSocket と同じポートで提供します:

* デフォルト: `http://<host>:18789/`
* 任意のプレフィックス: `gateway.controlUi.basePath` を設定（例: `/openclaw`）

各種機能は [Control UI](/ja/web/control-ui) に集約されています。
このページでは、バインドモード、セキュリティ、および外部公開される Web インターフェースについて説明します。

<div id="webhooks">
  ## Webhooks
</div>

`hooks.enabled=true` の場合、Gateway は同じ HTTP サーバー上で簡易な webhook エンドポイントも公開します。
認証とペイロードについては、[Gateway configuration](/ja/gateway/configuration) → `hooks` を参照してください。

<div id="config-default-on">
  ## 設定（デフォルト有効）
</div>

Control UI は、アセットが存在する場合（`dist/control-ui`）に **デフォルトで有効** になります。
設定で管理できます。

```json5
{
  gateway: {
    controlUi: { enabled: true, basePath: "/openclaw" } // basePath はオプション
  }
}
```

<div id="tailscale-access">
  ## Tailscale 経由でのアクセス
</div>

<div id="integrated-serve-recommended">
  ### 統合された Serve（推奨）
</div>

Gateway はループバックアドレスで待ち受けさせ、Tailscale Serve にプロキシさせます。

```json5
{
  gateway: {
    bind: "loopback",
    tailscale: { mode: "serve" }
  }
}
```

次に Gateway を起動します：

```bash
openclaw gateway
```

次を開きます:

* `https://<magicdns>/` (or your configured `gateway.controlUi.basePath`)

<div id="tailnet-bind-token">
  ### Tailnet のバインドとトークン
</div>

```json5
{
  gateway: {
    bind: "tailnet",
    controlUi: { enabled: true },
    auth: { mode: "token", token: "your-token" }
  }
}
```

次に Gateway を起動します（ループバック以外のアドレスへバインドするにはトークンが必要です）:

```bash
openclaw gateway
```

開く:

* `http://<tailscale-ip>:18789/`（または、設定した `gateway.controlUi.basePath`）

<div id="public-internet-funnel">
  ### 公開インターネット（ファネル）
</div>

```json5
{
  gateway: {
    bind: "loopback",
    tailscale: { mode: "funnel" },
    auth: { mode: "password" } // または OPENCLAW_GATEWAY_PASSWORD
  }
}
```

<div id="security-notes">
  ## セキュリティに関する注意事項
</div>

* Gateway 認証はデフォルトで必須です（トークン/パスワード、または Tailscale のアイデンティティヘッダー）。
* ループバック以外のアドレスへバインドする場合も、共有トークン/パスワード（`gateway.auth` または環境変数）が**必須**です。
* ウィザードはデフォルトで gateway トークンを生成します（ループバック接続時も同様）。
* UI は `connect.params.auth.token` または `connect.params.auth.password` を送信します。
* Serve を使用している場合、`gateway.auth.allowTailscale` が `true` のときは Tailscale のアイデンティティヘッダーで認証要件を満たせます（トークン/パスワードは不要です）。明示的な認証情報を必須にするには `gateway.auth.allowTailscale: false` を設定します。詳しくは
  [Tailscale](/ja/gateway/tailscale) と [Security](/ja/gateway/security) を参照してください。
* `gateway.tailscale.mode: "funnel"` を使用する場合は、`gateway.auth.mode: "password"`（共有パスワード）の設定が必要です。

<div id="building-the-ui">
  ## UI のビルド
</div>

Gateway は `dist/control-ui` から静的ファイルを配信します。UI をビルドするには、次のように実行します：

```bash
pnpm ui:build # 初回実行時にUI依存関係を自動インストール
```
