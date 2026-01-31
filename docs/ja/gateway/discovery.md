---
title: ディスカバリ
summary: "Gateway を検出するためのノードディスカバリとトランスポート（Bonjour、Tailscale、SSH）"
read_when:
  - Bonjour ディスカバリ/アドバタイズを実装または変更するとき
  - リモート接続モード（直接接続 vs SSH）を調整するとき
  - リモートノード用のノードディスカバリとペアリングを設計するとき
---

<div id="discovery-transports">
  # Discovery &amp; transports
</div>

OpenClaw には、表面的にはよく似ているものの、本質的には別個の 2 つの課題があります:

1. **Operator remote control**: 別の場所で動作している Gateway を制御するための macOS メニューバーアプリ。
2. **Node pairing**: iOS/Android（および将来のノード）が Gateway を検出し、安全にペアリングすること。

設計目標は、すべてのネットワークのディスカバリー／アドバタイズ機能を **Node Gateway** (`openclaw gateway`) に集約し、クライアント（mac アプリ、iOS）はその利用者（コンシューマ）として扱うことです。

<div id="terms">
  ## 用語
</div>

* **Gateway**: 状態（セッション、ペアリング、ノードレジストリ）を保持し、チャネルを実行する単一の長期間稼働 Gateway プロセス。多くの構成ではホストごとに 1 つを使用するが、分離された複数 Gateway 構成も可能。
* **Gateway WS（コントロールプレーン）**: デフォルトでは `127.0.0.1:18789` 上の WebSocket エンドポイント。`gateway.bind` によって LAN/tailnet にバインドできる。
* **Direct WS transport**: LAN/tailnet 向けの Gateway WS エンドポイント（SSH 不要）。
* **SSH transport（フォールバック）**: `127.0.0.1:18789` を SSH 経由で転送することで行うリモート制御。
* **Legacy TCP bridge（非推奨／削除済み）**: 旧式のノード用トランスポート（[Bridge protocol](/ja/gateway/bridge-protocol) を参照）。ディスカバリにはもう利用されない。

プロトコルの詳細:

* [Gateway protocol](/ja/gateway/protocol)
* [Bridge protocol（レガシー）](/ja/gateway/bridge-protocol)

<div id="why-we-keep-both-direct-and-ssh">
  ## 「direct」と SSH の両方を維持する理由
</div>

* **Direct WS** は、同一ネットワーク内や tailnet 内では最良のユーザー体験:
  * Bonjour を使った LAN 上での自動ディスカバリ
  * Gateway が管理する pairing token と ACL
  * シェルアクセスが不要で、プロトコルの公開範囲を絞って監査しやすくできる
* **SSH** は引き続き汎用的なフォールバック手段:
  * SSH アクセスさえあればどこでも動作（無関係なネットワーク間でも可）
  * マルチキャストや mDNS 周りの問題があっても動作する
  * SSH 以外に新たな受信ポートを開ける必要がない

<div id="discovery-inputs-how-clients-learn-where-the-gateway-is">
  ## ディスカバリの入力元（クライアントが Gateway の場所を知る方法）
</div>

<div id="1-bonjour-mdns-lan-only">
  ### 1) Bonjour / mDNS (LAN のみ)
</div>

Bonjour はベストエフォートであり、ネットワークをまたいで動作することはありません。「同一 LAN」内での利便性のためだけに使用されます。

通信方向:

* **Gateway** は Bonjour を使って自身の WS エンドポイントをアドバタイズします。
* クライアントは探索して「Gateway を選択する」一覧を表示し、選択したエンドポイントを保存します。

トラブルシューティングおよびビーコンの詳細については、[Bonjour](/ja/gateway/bonjour) を参照してください。

<div id="service-beacon-details">
  #### サービスビーコンの詳細
</div>

* サービスタイプ:
  * `_openclaw-gw._tcp` (Gateway トランスポート用ビーコン)
* TXT キー（非機密）:
  * `role=gateway`
  * `lanHost=<hostname>.local`
  * `sshPort=22`（またはアドバタイズしている任意のポート）
  * `gatewayPort=18789`（Gateway の WS + HTTP）
  * `gatewayTls=1`（TLS 有効時のみ）
  * `gatewayTlsSha256=<sha256>`（TLS 有効かつフィンガープリントが利用可能な場合のみ）
  * `canvasPort=18793`（デフォルトの canvas ホストポート。`/__openclaw__/canvas/` を配信）
  * `cliPath=<path>`（オプション。実行可能な `openclaw` エントリポイントまたはバイナリへの絶対パス）
  * `tailnetDns=<magicdns>`（オプションのヒント。Tailscale が利用可能な場合に自動検出）

無効化／上書き:

* `OPENCLAW_DISABLE_BONJOUR=1` はアドバタイズを無効化します。
* `~/.openclaw/openclaw.json` の `gateway.bind` が Gateway のバインドモードを制御します。
* `OPENCLAW_SSH_PORT` は TXT でアドバタイズされる SSH ポートを上書きします（デフォルトは 22）。
* `OPENCLAW_TAILNET_DNS` は `tailnetDns` ヒント（MagicDNS）を公開します。
* `OPENCLAW_CLI_PATH` はアドバタイズされる CLI パスを上書きします。

<div id="2-tailnet-cross-network">
  ### 2) Tailnet（クロスネットワーク）
</div>

London/Vienna スタイルの構成では、Bonjour は役に立ちません。推奨される「ダイレクト」ターゲットは次のとおりです。

* Tailscale の MagicDNS 名（推奨）または安定した Tailnet IP

Gateway が Tailscale 配下で動作していることを検出できる場合、クライアント向けのオプションのヒントとして（広域ビーコンを含む）`tailnetDns` を公開します。

<div id="3-manual-ssh-target">
  ### 3) 手動 / SSH ターゲット
</div>

ダイレクトルートがない場合（または direct が無効化されている場合）、クライアントはループバック上の Gateway ポートをポートフォワードすることで、常に SSH 経由で接続できます。

[リモートアクセス](/ja/gateway/remote) を参照してください。

<div id="transport-selection-client-policy">
  ## トランスポート選択（クライアントポリシー）
</div>

推奨されるクライアントの挙動は次のとおり:

1. ペアリング済みのダイレクトエンドポイントが設定されており、到達可能な場合はそれを使用する。
2. それ以外の場合、Bonjour が LAN 上で Gateway を検出したら、「この Gateway を使う」というワンタップの選択肢を提示し、ダイレクトエンドポイントとして保存する。
3. それ以外の場合、tailnet の DNS/IP が設定されていれば、ダイレクト接続を試みる。
4. それ以外の場合、SSH にフォールバックする。

<div id="pairing-auth-direct-transport">
  ## ペアリング + 認証（ダイレクトトランスポート）
</div>

Gateway はノード／クライアントの受け入れに関する唯一の信頼できる情報源です。

* ペアリングリクエストの作成／承認／拒否は Gateway 側で行われます（[Gateway pairing](/ja/gateway/pairing) を参照）。
* Gateway は次を適用します:
  * 認証（トークン / 鍵ペア）
  * スコープ / ACL（Gateway はあらゆるメソッドをそのまま中継するだけのプロキシではない）
  * レート制限

<div id="responsibilities-by-component">
  ## コンポーネントごとの責務
</div>

* **Gateway**: ディスカバリ用ビーコンをブロードキャストし、ペアリングの承認を行い、WS エンドポイントをホストします。
* **macOS app**: 接続先 Gateway の選択を支援し、ペアリングプロンプトを表示し、フォールバック手段としてのみ SSH を使用します。
* **iOS/Android ノード**: 補助的な機能として Bonjour で探索を行い、ペアリング済みの Gateway の WS に接続します。