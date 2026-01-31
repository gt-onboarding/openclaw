---
title: Bonjour
summary: "Bonjour/mDNS ディスカバリとデバッグ（Gateway ビーコン、クライアント、よくある障害パターン）"
read_when:
  - macOS/iOS で Bonjour ディスカバリの問題をトラブルシュートするとき
  - mDNS サービスタイプ、TXT レコード、またはディスカバリ UX を変更するとき
---

<div id="bonjour-mdns-discovery">
  # Bonjour / mDNS ディスカバリ
</div>

OpenClaw は、アクティブな Gateway（WebSocket エンドポイント）を検出するための **LAN 内限定の利便機能** として Bonjour（mDNS / DNS‑SD）を使用します。これはベストエフォートの仕組みであり、SSH や Tailnet ベースの接続手段を置き換えるものでは **ありません**。

<div id="widearea-bonjour-unicast-dnssd-over-tailscale">
  ## Tailscale 上での広域 Bonjour（ユニキャスト DNS‑SD）
</div>

ノードと Gateway が異なるネットワーク上にある場合、マルチキャスト mDNS は
境界を越えません。Tailscale 上で **ユニキャスト DNS‑SD**
（「Wide‑Area Bonjour」）に切り替えることで、同じディスカバリ UX を維持できます。

全体的な手順は次のとおりです:

1. Gateway ホスト上で DNS サーバーを起動し（Tailnet 経由で到達可能にする）。
2. 専用ゾーン配下（例: `openclaw.internal.`）で `_openclaw-gw._tcp` の DNS‑SD
   レコードを公開する。
3. Tailscale の **split DNS** を設定し、クライアント（iOS を含む）が
   選択したドメインをその DNS サーバー経由で名前解決するようにする。

OpenClaw は任意のディスカバリドメインをサポートします。`openclaw.internal.` は単なる例です。
iOS/Android ノードは `local.` と、設定した広域ドメインの両方を探索します。

<div id="gateway-config-recommended">
  ### Gateway の設定（推奨）
</div>

```json5
{
  gateway: { bind: "tailnet" }, // tailnet-only (recommended)
  discovery: { wideArea: { enabled: true } } // wide-area DNS-SD公開を有効化
}
```

<div id="onetime-dns-server-setup-gateway-host">
  ### 一度だけ行う DNS サーバー設定（Gateway ホスト）
</div>

```bash
openclaw dns setup --apply
```

これにより CoreDNS をインストールし、次のように設定します:

* Gateway の Tailscale インターフェイス上でのみポート 53 をリッスンする
* 選択したドメイン（例: `openclaw.internal.`）の名前解決を `~/.openclaw/dns/<domain>.db` から行う

Tailnet に接続されているマシンから検証します:

```bash
dns-sd -B _openclaw-gw._tcp openclaw.internal.
dig @<TAILNET_IPV4> -p 53 _openclaw-gw._tcp.openclaw.internal PTR +short
```

<div id="tailscale-dns-settings">
  ### Tailscale の DNS 設定
</div>

Tailscale の管理コンソールで次を実施します:

* Gateway の tailnet IP（UDP/TCP 53）を指すネームサーバーを追加します。
* split DNS を追加し、ディスカバリ用ドメインがそのネームサーバーを利用するように設定します。

クライアントが tailnet DNS を利用するように構成されていれば、iOS ノードはマルチキャストなしで
ディスカバリ用ドメイン内の `_openclaw-gw._tcp` を参照できます。

<div id="gateway-listener-security-recommended">
  ### Gateway リスナーのセキュリティ（推奨）
</div>

Gateway の WS ポート（デフォルト `18789`）は、デフォルトではループバックインターフェイスにバインドされます。LAN／tailnet からアクセスする場合は、バインド先を明示的に指定し、認証を有効のままにしておきます。

tailnet 専用構成の場合：

* `~/.openclaw/openclaw.json` の `gateway.bind: "tailnet"` を設定します。
* Gateway を再起動します（または macOS メニューバーアプリを再起動します）。

<div id="what-advertises">
  ## どれがアドバタイズするか
</div>

`_openclaw-gw._tcp` をアドバタイズするのは Gateway のみです。

<div id="service-types">
  ## サービス種別
</div>

* `_openclaw-gw._tcp` — Gateway のトランスポート用ビーコン（macOS/iOS/Android ノードで使用される）。

<div id="txt-keys-nonsecret-hints">
  ## TXT キー（非秘密のヒント）
</div>

Gateway は、UI フローを便利にするために、少量の非秘密のヒントを公開します:

* `role=gateway`
* `displayName=<friendly name>`
* `lanHost=<hostname>.local`
* `gatewayPort=<port>` (Gateway の WS + HTTP)
* `gatewayTls=1` (TLS が有効な場合のみ)
* `gatewayTlsSha256=<sha256>` (TLS が有効でフィンガープリントが利用可能な場合のみ)
* `canvasPort=<port>` (canvas ホストが有効な場合のみ; デフォルトは `18793`)
* `sshPort=<port>` (上書き指定がない場合はデフォルトで 22)
* `transport=gateway`
* `cliPath=<path>` (オプション; 実行可能な `openclaw` エントリポイントへの絶対パス)
* `tailnetDns=<magicdns>` (Tailnet が利用可能な場合のオプションのヒント)

<div id="debugging-on-macos">
  ## macOS でのデバッグ
</div>

便利な組み込みツール:

* インスタンスを一覧表示:
  ```bash
  dns-sd -B _openclaw-gw._tcp local.
  ```
* 特定のインスタンスを解決（`<instance>` を実際のインスタンス名に置き換え）:
  ```bash
  dns-sd -L "<instance>" _openclaw-gw._tcp local.
  ```

ブラウズはできるのに解決に失敗する場合は、たいてい LAN ポリシーか
mDNS リゾルバの問題が原因です。

<div id="debugging-in-gateway-logs">
  ## Gateway ログでのデバッグ
</div>

Gateway はローテーションされるログファイルを出力します（起動時に
`gateway log file: ...` として表示されます）。`bonjour:` 行を確認し、特に次のものを探してください:

* `bonjour: advertise failed ...`
* `bonjour: ... name conflict resolved` / `hostname conflict resolved`
* `bonjour: watchdog detected non-announced service ...`

<div id="debugging-on-ios-node">
  ## iOSノードでのデバッグ
</div>

iOSノードは `NWBrowser` を使用して `_openclaw-gw._tcp` を検出します。

ログを取得するには:

* 設定 → Gateway → Advanced → **Discovery Debug Logs**
* 設定 → Gateway → Advanced → **Discovery Logs** → 現象を再現 → **Copy**

ログには、ブラウザの状態遷移と結果セットの変更が含まれます。

<div id="common-failure-modes">
  ## よくある障害モード
</div>

* **Bonjour はネットワークをまたげない**: Tailnet か SSH を使用してください。
* **マルチキャストがブロックされている**: 一部の Wi‑Fi ネットワークでは mDNS が無効化されています。
* **スリープ／インターフェースの切り替え**: macOS が一時的に mDNS の結果を見失うことがあります。再試行してください。
* **Browse は成功するが resolve が失敗する**: マシン名はシンプルに保ってください（絵文字や句読点は避ける）。そのうえで Gateway を再起動してください。サービスインスタンス名はホスト名から生成されるため、複雑すぎる名前は一部のリゾルバを混乱させることがあります。

<div id="escaped-instance-names-032">
  ## エスケープされたインスタンス名（`\032`）
</div>

Bonjour/DNS‑SD は、サービスインスタンス名内のバイトを 10 進数の `\DDD`
シーケンスとしてエスケープすることがあります（例: 空白は `\032` になる）。

* プロトコルレベルではこれは正常な動作です。
* UI は表示用にデコードする必要があります（iOS は `BonjourEscapes.decode` を使用します）。

<div id="disabling-configuration">
  ## 無効化 / 設定
</div>

* `OPENCLAW_DISABLE_BONJOUR=1` はアドバタイズを無効にします（旧環境変数: `OPENCLAW_DISABLE_BONJOUR`）。
* `~/.openclaw/openclaw.json` 内の `gateway.bind` は Gateway のバインドモードを制御します。
* `OPENCLAW_SSH_PORT` は TXT レコードでアドバタイズされる SSH ポートを上書きします（旧環境変数: `OPENCLAW_SSH_PORT`）。
* `OPENCLAW_TAILNET_DNS` は TXT レコードに MagicDNS のヒントを公開します（旧環境変数: `OPENCLAW_TAILNET_DNS`）。
* `OPENCLAW_CLI_PATH` はアドバタイズされる CLI のパスを上書きします（旧環境変数: `OPENCLAW_CLI_PATH`）。

<div id="related-docs">
  ## 関連ドキュメント
</div>

* ディスカバリポリシーとトランスポート選択: [Discovery](/ja/gateway/discovery)
* ノードのペアリングと承認: [Gateway pairing](/ja/gateway/pairing)