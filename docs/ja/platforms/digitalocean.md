---
title: DigitalOcean
summary: "DigitalOcean での OpenClaw（シンプルな有料VPSオプション）"
read_when:
  - DigitalOcean 上で OpenClaw をセットアップする場合
  - OpenClaw 用の安価な VPS ホスティングを探している場合
---

<div id="openclaw-on-digitalocean">
  # DigitalOcean での OpenClaw
</div>

<div id="goal">
  ## 目的
</div>

DigitalOcean 上で OpenClaw Gateway を常時稼働させて、**月額 6 ドル**（リザーブド料金なら月額 4 ドル）に収めること。

月額 0 ドルのオプションでもよく、ARM ＋ プロバイダー固有のセットアップをいとわない場合は、[Oracle Cloud ガイド](/ja/platforms/oracle) を参照してください。

<div id="cost-comparison-2026">
  ## コスト比較 (2026)
</div>

| Provider | プラン | スペック | 料金/月 | メモ |
|----------|------|-------|----------|-------|
| Oracle Cloud | Always Free ARM | 最大 4 OCPU、24GB RAM | $0 | ARM、リソース上限あり / サインアップ手順に癖あり |
| Hetzner | CX22 | 2 vCPU、4GB RAM | €3.79 (~$4) | 最安の有料オプション |
| DigitalOcean | Basic | 1 vCPU、1GB RAM | $6 | 使いやすい UI と充実したドキュメント |
| Vultr | Cloud Compute | 1 vCPU、1GB RAM | $6 | 利用可能なリージョンが多い |
| Linode | Nanode | 1 vCPU、1GB RAM | $5 | 現在は Akamai の一部 |

**プロバイダーの選び方:**

* DigitalOcean: UX が最もシンプルで、セットアップも予測しやすい（本ガイド）
* Hetzner: 価格/性能比が良い（[Hetzner ガイド](/ja/platforms/hetzner) を参照）
* Oracle Cloud: 月額 $0 も可能だが、やや扱いづらく ARM のみ対応（[Oracle ガイド](/ja/platforms/oracle) を参照）

***

<div id="prerequisites">
  ## 前提条件
</div>

* DigitalOcean アカウント（[$200 の無料クレジットでサインアップ](https://m.do.co/c/signup)）
* SSH 鍵ペア（またはパスワード認証を利用してもよいこと）
* 作業時間：約 20 分

<div id="1-create-a-droplet">
  ## 1) Droplet を作成する
</div>

1. [DigitalOcean](https://cloud.digitalocean.com/) にログインする
2. **Create → Droplets** をクリックする
3. 次を選択する：
   * **Region:** 自分（またはユーザー）に最も近いリージョン
   * **Image:** Ubuntu 24.04 LTS
   * **Size:** Basic → Regular → **$6/mo** (1 vCPU, 1GB RAM, 25GB SSD)
   * **Authentication:** SSH key（推奨）またはパスワード
4. **Create Droplet** をクリックする
5. IP アドレスをメモしておく

<div id="2-connect-via-ssh">
  ## 2) SSH 経由で接続する
</div>

```bash
ssh root@YOUR_DROPLET_IP
```

<div id="3-install-openclaw">
  ## 3) OpenClaw をインストールする
</div>

```bash
# システムの更新
apt update && apt upgrade -y

# Node.js 22のインストール
curl -fsSL https://deb.nodesource.com/setup_22.x | bash -
apt install -y nodejs

# OpenClawのインストール
curl -fsSL https://openclaw.bot/install.sh | bash

# 確認
openclaw --version
```

<div id="4-run-onboarding">
  ## 4) オンボーディングを実施する
</div>

```bash
openclaw onboard --install-daemon
```

ウィザードでは次の内容を順に案内します:

* モデルの認証設定（APIキーまたはOAuth）
* チャンネル設定（Telegram、WhatsApp、Discordなど）
* Gatewayトークン（自動生成）
* デーモンのインストール（systemd）

<div id="5-verify-the-gateway">
  ## 5) Gateway を確認する
</div>

```bash
# ステータスを確認
openclaw status

# サービスを確認
systemctl --user status openclaw-gateway.service

# ログを確認
journalctl --user -u openclaw-gateway.service -f
```

<div id="6-access-the-dashboard">
  ## 6) ダッシュボードへアクセスする
</div>

Gateway はデフォルトでループバックにバインドされます。Control UI にアクセスするには次のようにします:

**オプション A: SSH トンネル（推奨）**

```bash
# From your local machine
ssh -L 18789:localhost:18789 root@YOUR_DROPLET_IP

# 次にブラウザで開く: http://localhost:18789
```

**オプションB：Tailscale Serve（HTTPS、ループバックのみ）**

```bash
# On the droplet
curl -fsSL https://tailscale.com/install.sh | sh
tailscale up

# GatewayでTailscale Serveを使用するように設定
openclaw config set gateway.tailscale.mode serve
openclaw gateway restart
```

開く: `https://<magicdns>/`

注意:

* Serve は Gateway をループバック専用のままに保ち、Tailscale のアイデンティティヘッダーで認証します。
* 代わりにトークン／パスワード認証を必須にするには、`gateway.auth.allowTailscale: false` を設定するか、`gateway.auth.mode: "password"` に設定します。

**オプション C: Tailnet バインド（Serve なし）**

```bash
openclaw config set gateway.bind tailnet
openclaw gateway restart
```

次のURLを開いてください: `http://<tailscale-ip>:18789`（トークンが必要です）。

<div id="7-connect-your-channels">
  ## 7) チャンネルを連携する
</div>

<div id="telegram">
  ### Telegram
</div>

```bash
openclaw pairing list telegram
openclaw pairing approve telegram <CODE>
```

<div id="whatsapp">
  ### WhatsApp
</div>

```bash
openclaw channels login whatsapp
# QRコードをスキャンする
```

他のプロバイダーについては [Channels](/ja/channels) を参照してください。

***

<div id="optimizations-for-1gb-ram">
  ## 1GB RAM 向けの最適化
</div>

$6 の Droplet には RAM が 1GB しかありません。安定して動作させるために、次の対策を行ってください。

<div id="add-swap-recommended">
  ### スワップを追加する（推奨）
</div>

```bash
fallocate -l 2G /swapfile
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
echo '/swapfile none swap sw 0 0' >> /etc/fstab
```

<div id="use-a-lighter-model">
  ### より軽量なモデルを使う
</div>

OOM が発生する場合は、次を検討してください:

* ローカルモデルではなく、API ベースのモデル（Claude、GPT など）を使用する
* `agents.defaults.model.primary` をより小さなモデルに設定する

<div id="monitor-memory">
  ### メモリの監視
</div>

```bash
free -h
htop
```

***

<div id="persistence">
  ## 永続化
</div>

すべての状態データは次の場所に保存されます。

* `~/.openclaw/` — 設定、認証情報、セッションデータ
* `~/.openclaw/workspace/` — ワークスペース（SOUL.md、メモリなど）

これらは再起動後も保持されます。定期的にバックアップを取得してください。

```bash
tar -czvf openclaw-backup.tar.gz ~/.openclaw ~/.openclaw/workspace
```

***

<div id="oracle-cloud-free-alternative">
  ## Oracle Cloud 無料代替案
</div>

Oracle Cloud は、ここで紹介しているどの有料オプションよりもはるかに強力な **Always Free** ARM インスタンスを、月額 $0 で提供しています。

| 提供内容 | スペック |
|---------|----------|
| **4 OCPU** | ARM Ampere A1 |
| **24GB RAM** | 十分以上 |
| **200GB ストレージ** | ブロックボリューム |
| **永年無料** | クレジットカード請求なし |

**注意点：**

* サインアップがシビアで、失敗することがある（失敗したら再試行する）
* ARM アーキテクチャ — ほとんどのものは動作するが、一部のバイナリは ARM 用ビルドが必要

セットアップの完全な手順については [Oracle Cloud](/ja/platforms/oracle) を参照してください。サインアップ時のコツや登録プロセスのトラブルシュートについては、この [コミュニティガイド](https://gist.github.com/rssnyder/51e3cfedd730e7dd5f4a816143b25dbd) を参照してください。

***

<div id="troubleshooting">
  ## トラブルシューティング
</div>

<div id="gateway-wont-start">
  ### Gateway が起動しない
</div>

```bash
openclaw gateway status
openclaw doctor --non-interactive
journalctl -u openclaw --no-pager -n 50
```

<div id="port-already-in-use">
  ### ポートが既に使用中です
</div>

```bash
lsof -i :18789
kill <PID>
```

<div id="out-of-memory">
  ### メモリ不足
</div>

```bash
# メモリを確認
free -h

# スワップを追加
# または $12/月のドロップレット (2GB RAM) にアップグレード
```

***

<div id="see-also">
  ## 関連情報
</div>

* [Hetzner ガイド](/ja/platforms/hetzner) — 低コストでより高性能
* [Docker インストール](/ja/install/docker) — コンテナ化セットアップ
* [Tailscale](/ja/gateway/tailscale) — 安全なリモートアクセス
* [設定](/ja/gateway/configuration) — 設定の完全なリファレンス