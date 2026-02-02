---
title: リモート
summary: "SSH トンネル（Gateway WS）および tailnet を利用したリモートアクセス"
read_when:
  - Gateway のリモートセットアップを実行またはトラブルシューティングしているとき
---

<div id="remote-access-ssh-tunnels-and-tailnets">
  # リモートアクセス（SSH、トンネル、tailnet）
</div>

このリポジトリでは、専用ホスト（デスクトップ/サーバー）上で単一の Gateway（マスター）を常時稼働させ、クライアントをそこに接続することで「SSH 経由のリモート」をサポートします。

* **オペレーター（あなた / macOS アプリ）**向け: SSH トンネリングは汎用的なフォールバック手段です。
* **ノード（iOS/Android および将来のデバイス）**向け: 必要に応じて LAN/tailnet または SSH トンネル経由で Gateway の **WebSocket** に接続します。

<div id="the-core-idea">
  ## コアとなる考え方
</div>

* Gateway の WebSocket は、設定されたポート（デフォルトは 18789）で **ループバック** にバインドされます。
* リモートから利用する場合は、そのループバックポートを SSH 経由でフォワードするか、tailnet/VPN を使ってトンネル構成をよりシンプルにします。

<div id="common-vpntailnet-setups-where-the-agent-lives">
  ## 一般的な VPN／tailnet 構成（エージェントが常駐する場所）
</div>

**Gateway ホスト**を「エージェントが常駐する場所」と考えてください。Gateway ホストは、セッション、認証プロファイル、チャネル、および状態を管理します。
あなたのノート PC／デスクトップ PC（およびノード）は、そのホストに接続します。

<div id="1-always-on-gateway-in-your-tailnet-vps-or-home-server">
  ### 1) tailnet 内の常時稼働 Gateway（VPS またはホームサーバー）
</div>

Gateway を常時稼働しているホスト上で動かし、**Tailscale** または SSH 経由でアクセスします。

* **ベストな UX:** `gateway.bind: "loopback"` を維持し、Control UI 用に **Tailscale Serve** を使用します。
* **フォールバック:** loopback のままにしつつ、アクセスが必要な任意のマシンから SSH トンネルを張ります。
* **例:** [exe.dev](/ja/platforms/exe-dev)（簡易 VM）や [Hetzner](/ja/platforms/hetzner)（本番向け VPS）。

ノート PC が頻繁にスリープする一方で、エージェントは常時稼働させておきたい場合に最適です。

<div id="2-home-desktop-runs-the-gateway-laptop-is-remote-control">
  ### 2) 自宅のデスクトップで Gateway を動かし、ノート PC からリモート操作する
</div>

ノート PC ではエージェントを**実行しません**。ノート PC からリモート接続するだけです:

* macOS アプリの **Remote over SSH** モード（Settings → General → “OpenClaw runs”）を使用します。
* アプリがトンネルを開いて管理するため、WebChat とヘルスチェックがそのまま利用できます。

運用手順書: [macOS リモートアクセス](/ja/platforms/mac/remote).

<div id="3-laptop-runs-the-gateway-remote-access-from-other-machines">
  ### 3) ラップトップで Gateway を動かし、他のマシンからリモートアクセスする
</div>

Gateway はローカルに保ちつつ、安全に外部からアクセスできるようにする:

* 他のマシンからラップトップへ SSH トンネルで接続する、または
* Tailscale Serve で Control UI を公開し、Gateway 自体はループバック専用のままにしておく。

ガイド: [Tailscale](/ja/gateway/tailscale) と [Web の概要](/ja/web)。

<div id="command-flow-what-runs-where">
  ## コマンドフロー（どこで何が動くか）
</div>

1 つの Gateway サービスが状態とチャネルを管理します。ノードは周辺デバイスです。

フロー例（Telegram → ノード）:

* Telegram メッセージが **Gateway** に到着します。
* Gateway が **エージェント** を実行し、ノードツールを呼び出すかどうかを判断します。
* Gateway が Gateway WebSocket（`node.*` RPC）経由で **ノード** を呼び出します。
* ノードが結果を返し、Gateway が Telegram に返信します。

注意事項:

* **ノードは Gateway サービスを実行しません。** 意図的に分離されたプロファイルを実行する場合を除き、1 ホストあたり実行する Gateway は 1 つにしてください（[Multiple gateways](/ja/gateway/multiple-gateways) を参照）。
* macOS アプリの「ノードモード」は、Gateway WebSocket 上のノードクライアントにすぎません。

<div id="ssh-tunnel-cli-tools">
  ## SSH トンネル (CLI + tools)
</div>

ローカルからリモート Gateway の WS へのトンネルを作成します：

```bash
ssh -N -L 18789:127.0.0.1:18789 user@host
```

トンネルが有効になっている場合:

* `openclaw health` と `openclaw status --deep` は、`ws://127.0.0.1:18789` 経由でリモート Gateway にアクセスできます。
* 必要に応じて、`openclaw gateway {status,health,send,agent,call}` も `--url` を使って転送された URL を指定できます。

注意: `18789` は、設定した `gateway.port`（または `--port` / `OPENCLAW_GATEWAY_PORT`）に置き換えてください。

<div id="cli-remote-defaults">
  ## CLI のリモートのデフォルト設定
</div>

リモート先を保存しておき、CLI コマンドがそれをデフォルトで使うようにできます。

```json5
{
  gateway: {
    mode: "remote",
    remote: {
      url: "ws://127.0.0.1:18789",
      token: "your-token"
    }
  }
}
```

Gateway がループバック専用で動作している場合は、URL は `ws://127.0.0.1:18789` のままにしておき、先に SSH トンネルを開いてください。

<div id="chat-ui-over-ssh">
  ## SSH 経由のチャット UI
</div>

WebChat は、もはや別の HTTP ポートを使用していません。SwiftUI のチャット UI は Gateway の WebSocket に直接接続します。

* SSH で `18789` ポートを転送し（上記参照）、クライアントを `ws://127.0.0.1:18789` に接続します。
* macOS では、トンネルを自動的に管理するアプリの「Remote over SSH」モードを優先して使用してください。

<div id="macos-app-remote-over-ssh">
  ## macOS アプリ「Remote over SSH」
</div>

macOS メニューバーアプリを使うと、同じ構成（リモートステータスチェック、WebChat、Voice Wake の転送）をエンドツーエンドで制御できます。

Runbook: [macOS リモートアクセス](/ja/platforms/mac/remote)。

<div id="security-rules-remotevpn">
  ## セキュリティルール（リモート/VPN）
</div>

要点: **本当にバインドが必要な場合を除き、Gateway はループバック専用のままにしておくこと。**

* **ループバック + SSH/Tailscale Serve** が最も安全なデフォルト（パブリック公開なし）。
* **ループバック以外のバインド**（`lan`/`tailnet`/`custom`、またはループバックが利用できない場合の `auto`）では、必ず認証トークン/パスワードを使用すること。
* `gateway.remote.token` はリモート CLI 呼び出し専用であり、ローカル認証を **有効化しない**。
* `gateway.remote.tlsFingerprint` は、`wss://` 利用時にリモート TLS 証明書をピン固定する。
* **Tailscale Serve** は、`gateway.auth.allowTailscale: true` の場合、アイデンティティヘッダー経由で認証できる。
  トークン/パスワードによる認証を使いたい場合は `false` に設定すること。
* ブラウザからの制御はオペレーターアクセスと同等に扱うこと：tailnet 限定 + 明示的なノードのペアリング。

詳細解説: [Security](/ja/gateway/security).