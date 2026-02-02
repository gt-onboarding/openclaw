---
title: ネットワーク
summary: "ネットワークハブ: Gateway のエンドポイント、ペアリング、ディスカバリー、およびセキュリティ"
read_when:
  - ネットワークアーキテクチャとセキュリティの概要が必要なとき
  - ローカルアクセスと Tailnet アクセス、またはペアリングをデバッグしているとき
  - ネットワーク関連ドキュメントの公式な一覧を確認したいとき
---

<div id="network-hub">
  # ネットワークハブ
</div>

このハブでは、OpenClaw が localhost、LAN、tailnet 上のデバイスをどのように接続・ペアリング・セキュア化するかを説明するコアドキュメントをまとめて参照できます。

<div id="core-model">
  ## コアモデル
</div>

* [Gateway アーキテクチャ](/ja/concepts/architecture)
* [Gateway プロトコル](/ja/gateway/protocol)
* [Gateway ランブック](/ja/gateway)
* [Web サーフェスとバインドモード](/ja/web)

<div id="pairing-identity">
  ## ペアリングとアイデンティティ
</div>

* [ペアリング概要（DM + ノード）](/ja/start/pairing)
* [Gateway 管理ノードのペアリング](/ja/gateway/pairing)
* [Devices CLI（ペアリング + トークンローテーション）](/ja/cli/devices)
* [Pairing CLI（DM 承認）](/ja/cli/pairing)

ローカル信頼:

* ローカル接続（ループバック、または Gateway ホスト自身の tailnet アドレス）は、
  同一ホストでのユーザー体験をスムーズに保つため、自動的にペアリングが承認されます。
* 非ローカルの tailnet/LAN クライアントについては、引き続き明示的なペアリング承認が必要です。

<div id="discovery-transports">
  ## ディスカバリとトランスポート
</div>

* [ディスカバリとトランスポート](/ja/gateway/discovery)
* [Bonjour / mDNS](/ja/gateway/bonjour)
* [リモートアクセス (SSH)](/ja/gateway/remote)
* [Tailscale](/ja/gateway/tailscale)

<div id="nodes-transports">
  ## ノードとトランスポート
</div>

* [ノード概要](/ja/nodes)
* [ブリッジプロトコル（レガシーノード）](/ja/gateway/bridge-protocol)
* [ノード運用ガイド：iOS](/ja/platforms/ios)
* [ノード運用ガイド：Android](/ja/platforms/android)

<div id="security">
  ## セキュリティ
</div>

* [セキュリティ概要](/ja/gateway/security)
* [Gateway 設定リファレンス](/ja/gateway/configuration)
* [トラブルシューティング](/ja/gateway/troubleshooting)
* [Doctor](/ja/gateway/doctor)