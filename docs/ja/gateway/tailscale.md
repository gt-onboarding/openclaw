---
title: Tailscale
summary: "Gateway ダッシュボード向け統合 Tailscale Serve/Funnel"
read_when:
  - localhost 外部から Gateway Control UI を公開するとき
  - tailnet または公開ダッシュボードへのアクセスを自動化するとき
---

<div id="tailscale-gateway-dashboard">
  # Tailscale（Gateway ダッシュボード）
</div>

OpenClaw は、Gateway ダッシュボードと WebSocket ポート向けに、Tailscale の **Serve**（tailnet 用）または **Funnel**（パブリック公開用）を自動的に設定できます。これにより Gateway はループバックアドレスにバインドされたままになり、Tailscale が HTTPS、ルーティング、および（Serve の場合）アイデンティティヘッダーを提供します。

<div id="modes">
  ## モード
</div>

- `serve`: `tailscale serve` を使った Tailnet 内限定の Serve。Gateway プロセスはローカルホスト (`127.0.0.1`) 上で動作し続けます。
- `funnel`: `tailscale funnel` を使ったパブリック HTTPS。OpenClaw では共有パスワードが必須です。
- `off`: デフォルト値（Tailscale の自動化なし）。

<div id="auth">
  ## 認証
</div>

ハンドシェイク方法を制御するには `gateway.auth.mode` を設定します:

- `token`（`OPENCLAW_GATEWAY_TOKEN` が設定されている場合のデフォルト）
- `password`（`OPENCLAW_GATEWAY_PASSWORD` または設定ファイルで共有シークレットを指定）

`tailscale.mode = "serve"` かつ `gateway.auth.allowTailscale` が `true` の場合、
有効な Serve プロキシリクエストは、トークン/パスワードを送らずに
Tailscale のアイデンティティヘッダ（`tailscale-user-login`）経由で認証できます。
OpenClaw は、ローカルの Tailscale デーモン（`tailscale whois`）を使って
`x-forwarded-for` アドレスを解決し、その結果がヘッダと一致することを確認してから、
そのリクエスト送信元のアイデンティティを検証します。
OpenClaw は、ループバックから到着し、かつ Tailscale の
`x-forwarded-for`、`x-forwarded-proto`、`x-forwarded-host`
ヘッダが付与されている場合にのみ、そのリクエストを Serve 経由のものとして扱います。
明示的な認証情報を必須にするには、`gateway.auth.allowTailscale: false` を設定するか、
`gateway.auth.mode: "password"` を強制してください。

<div id="config-examples">
  ## 設定例
</div>

<div id="tailnet-only-serve">
  ### Tailnet 内のみ（Serve）
</div>

```json5
{
  gateway: {
    bind: "loopback",
    tailscale: { mode: "serve" }
  }
}
```

次の URL を開きます: `https://<magicdns>/`（または設定済みの `gateway.controlUi.basePath`）


<div id="tailnet-only-bind-to-tailnet-ip">
  ### Tailnet のみ（Tailnet IP にバインド）
</div>

Gateway を Tailnet の IP アドレスで直接待ち受けさせたい場合に使用します（Serve/Funnel は使用しません）。

```json5
{
  gateway: {
    bind: "tailnet",
    auth: { mode: "token", token: "your-token" }
  }
}
```

別の Tailnet デバイスから接続する場合:

* Control UI: `http://<tailscale-ip>:18789/`
* WebSocket: `ws://<tailscale-ip>:18789`

注意：ループバックアドレス（`http://127.0.0.1:18789`）ではこのモードでは**接続できません**。


<div id="public-internet-funnel-shared-password">
  ### 公開インターネット (Funnel + 共有パスワード)
</div>

```json5
{
  gateway: {
    bind: "loopback",
    tailscale: { mode: "funnel" },
    auth: { mode: "password", password: "replace-me" }
  }
}
```

パスワードをディスクに保存するのではなく、`OPENCLAW_GATEWAY_PASSWORD` を使用してください。


<div id="cli-examples">
  ## CLI の使用例
</div>

```bash
openclaw gateway --tailscale serve
openclaw gateway --tailscale funnel --auth password
```


<div id="notes">
  ## 注意事項
</div>

- Tailscale Serve/Funnel を利用するには、`tailscale` CLI がインストールされており、ログイン済みである必要があります。
- 公開されてしまうのを避けるため、認証モードが `password` でない場合は `tailscale.mode: "funnel"` は起動しません。
- シャットダウン時に OpenClaw によって `tailscale serve` や
  `tailscale funnel` の設定を元に戻したい場合は、`gateway.tailscale.resetOnExit` を設定してください。
- `gateway.bind: "tailnet"` は Tailnet への直接バインドです（HTTPS なし、Serve/Funnel なし）。
- `gateway.bind: "auto"` はループバックを優先します。Tailnet のみを使いたい場合は `tailnet` を使用してください。
- Serve/Funnel が公開するのは **Gateway control UI + WS** のみです。ノードは同じ Gateway の WS エンドポイントに接続するため、Serve 経由でノードにアクセスできます。

<div id="browser-control-remote-gateway-local-browser">
  ## ブラウザ制御（リモート Gateway + ローカルブラウザ）
</div>

Gateway をあるマシン上で動かしつつ、別のマシン上のブラウザを操作したい場合は、
ブラウザ側のマシンで **ノードホスト** を実行し、両方を同じ tailnet 上に接続します。
Gateway はブラウザ操作をノードにプロキシするため、別途制御用サーバーや Serve URL は不要です。

ブラウザ制御には Funnel を使わず、ノードのペアリングはオペレーターアクセスと同様に扱ってください。

<div id="tailscale-prerequisites-limits">
  ## Tailscale の前提条件と制限事項
</div>

- Serve を利用するには、tailnet で HTTPS が有効になっている必要があります。未設定の場合は、CLI がプロンプトを表示します。
- Serve は Tailscale のアイデンティティヘッダーを付与しますが、Funnel は付与しません。
- Funnel を利用するには、Tailscale v1.38.3 以上、MagicDNS、HTTPS の有効化、および Funnel 用のノード属性が必要です。
- Funnel は TLS 経由のポート `443`、`8443`、`10000` のみをサポートします。
- macOS で Funnel を利用するには、オープンソース版の Tailscale アプリが必要です。

<div id="learn-more">
  ## 詳細情報
</div>

- Tailscale Serve の概要: https://tailsscale.com/kb/1312/serve
- `tailscale serve` コマンド: https://tailscale.com/kb/1242/tailscale-serve
- Tailscale Funnel の概要: https://tailscale.com/kb/1223/tailscale-funnel
- `tailscale funnel` コマンド: https://tailscale.com/kb/1311/tailscale-funnel