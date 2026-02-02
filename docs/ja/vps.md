---
title: VPS
summary: "OpenClaw 向け VPS ホスティング ハブ (Oracle/Fly/Hetzner/GCP/exe.dev)"
read_when:
  - Gateway をクラウド上で実行したいとき
  - VPS/ホスティングガイドの一覧を手早く把握したいとき
---

<div id="vps-hosting">
  # VPS ホスティング
</div>

このハブは、サポートされている VPS／ホスティング向けガイドへのリンクをまとめるとともに、クラウド環境へのデプロイの仕組みを高レベルに説明します。

<div id="pick-a-provider">
  ## プロバイダーを選ぶ
</div>

* **Railway**（ワンクリック + ブラウザーからのセットアップ）：[Railway](/ja/railway)
* **Northflank**（ワンクリック + ブラウザーからのセットアップ）：[Northflank](/ja/northflank)
* **Oracle Cloud (Always Free)**：[Oracle](/ja/platforms/oracle) — 月額 $0（Always Free、ARM。キャパシティ／サインアップの可用性にばらつきが出る場合あり）
* **Fly.io**：[Fly.io](/ja/platforms/fly)
* **Hetzner (Docker)**：[Hetzner](/ja/platforms/hetzner)
* **GCP (Compute Engine)**：[GCP](/ja/platforms/gcp)
* **exe.dev**（VM + HTTPS プロキシ）：[exe.dev](/ja/platforms/exe-dev)
* **AWS (EC2/Lightsail/free tier)**：これも問題なく利用できます。動画ガイド：
  https://x.com/techfrenAJ/status/2014934471095812547

<div id="how-cloud-setups-work">
  ## クラウド構成の仕組み
</div>

* **Gateway は VPS 上で動作**し、状態とワークスペースを保持します。
* ノートPCやスマートフォンから **Control UI** または **Tailscale/SSH** 経由で接続します。
* VPS を状態の唯一の「正」として扱い、状態とワークスペースを**バックアップ**してください。
* デフォルトの安全な構成: Gateway はループバックインターフェース上のみにバインドし、SSH トンネルまたは Tailscale Serve 経由でアクセスします。\
  `lan` / `tailnet` にバインドする場合は、`gateway.auth.token` または `gateway.auth.password` の設定を必須にします。

リモートアクセス: [Gateway remote](/ja/gateway/remote)\
プラットフォームハブ: [Platforms](/ja/platforms)

<div id="using-nodes-with-a-vps">
  ## VPS でノードを使う
</div>

Gateway をクラウド上で動かしつつ、ローカルデバイス
（Mac / iOS / Android / headless）上の **ノード** をペアリングして利用できます。
Gateway をクラウドに置いたまま、ノードからローカルの画面（screen）/カメラ（camera）/キャンバス（canvas）と `system.run`
の機能を提供させることができます。

ドキュメント: [Nodes](/ja/nodes), [Nodes CLI](/ja/cli/nodes)