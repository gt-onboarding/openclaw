---
title: Oracle
summary: "Oracle Cloud での OpenClaw（常時無料の ARM）"
read_when:
  - Oracle Cloud 上で OpenClaw をセットアップするとき
  - OpenClaw 用の低コストな VPS ホスティングを探しているとき
  - 小型サーバーで OpenClaw を 24時間365日稼働させたいとき
---

<div id="openclaw-on-oracle-cloud-oci">
  # Oracle Cloud (OCI) 上での OpenClaw
</div>

<div id="goal">
  ## 目的
</div>

Oracle Cloud の **Always Free** ARM ティア上で OpenClaw Gateway を永続的に稼働させる。

Oracle の無料ティアは（すでに OCI アカウントを持っている場合は特に）OpenClaw に非常に適した選択肢になりえますが、いくつかのトレードオフがあります。

* ARM アーキテクチャ（ほとんどは動作するが、一部のバイナリは x86 専用の場合がある）
* キャパシティやサインアップまわりの扱いがやや気難しい

<div id="cost-comparison-2026">
  ## コスト比較 (2026年)
</div>

| プロバイダー | プラン | スペック | 月額料金 | 備考 |
|----------|------|-------|----------|-------|
| Oracle Cloud | Always Free ARM | 最大 4 OCPU、24GB RAM | $0 | ARM、容量に制限あり |
| Hetzner | CX22 | 2 vCPU、4GB RAM | 約 $4 | 最も安価な有料オプション |
| DigitalOcean | Basic | 1 vCPU、1GB RAM | $6 | UI が分かりやすく、ドキュメントも充実 |
| Vultr | Cloud Compute | 1 vCPU、1GB RAM | $6 | リージョンが多い |
| Linode | Nanode | 1 vCPU、1GB RAM | $5 | 現在は Akamai 傘下 |

***

<div id="prerequisites">
  ## 前提条件
</div>

* Oracle Cloud アカウント（[こちらからサインアップ](https://www.oracle.com/cloud/free/)）— 問題が発生した場合は [コミュニティサインアップガイド](https://gist.github.com/rssnyder/51e3cfedd730e7dd5f4a816143b25dbd) を参照してください
* Tailscale アカウント（[tailscale.com](https://tailscale.com) で無料）
* 所要時間：約 30 分

<div id="1-create-an-oci-instance">
  ## 1) OCIインスタンスを作成する
</div>

1. [Oracle Cloud Console](https://cloud.oracle.com/) にログインする
2. **Compute → Instances → Create Instance** に移動する
3. 次のように設定する:
   * **Name:** `openclaw`
   * **Image:** Ubuntu 24.04 (aarch64)
   * **Shape:** `VM.Standard.A1.Flex` (Ampere ARM)
   * **OCPUs:** 2（最大4まで）
   * **Memory:** 12 GB（最大24 GBまで）
   * **Boot volume:** 50 GB（無料枠は最大200 GBまで）
   * **SSH key:** 自身の公開鍵を追加する
4. **Create** をクリックする
5. パブリック IP アドレスを控えておく

**Tip:** インスタンス作成時に &quot;Out of capacity&quot; エラーが出る場合は、別のアベイラビリティ・ドメインを試すか、時間をおいて再試行してください。Free Tier のキャパシティには制限があります。

<div id="2-connect-and-update">
  ## 2) 接続とアップデート
</div>

```bash
# パブリックIPで接続
ssh ubuntu@YOUR_PUBLIC_IP

# システムを更新
sudo apt update && sudo apt upgrade -y
sudo apt install -y build-essential
```

**注意:** 一部の依存関係を ARM 向けにコンパイルするには `build-essential` が必要です。

<div id="3-configure-user-and-hostname">
  ## 3) ユーザーとホスト名の設定
</div>

```bash
# Set hostname
sudo hostnamectl set-hostname openclaw

# Set password for ubuntu user
sudo passwd ubuntu

# lingeringを有効化（ログアウト後もユーザーサービスを実行し続ける）
sudo loginctl enable-linger ubuntu
```

<div id="4-install-tailscale">
  ## 4) Tailscale をインストールする
</div>

```bash
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up --ssh --hostname=openclaw
```

これで Tailscale SSH が有効になり、Tailnet 上の任意のデバイスから `ssh openclaw` で接続できるようになります。パブリック IP は不要です。

検証:

```bash
tailscale status
```

**これ以降は、Tailscale 経由で接続してください:** `ssh ubuntu@openclaw`（または Tailscale の IP アドレスを使用）。

<div id="5-install-openclaw">
  ## 5) OpenClaw をインストールする
</div>

```bash
curl -fsSL https://openclaw.bot/install.sh | bash
source ~/.bashrc
```

「How do you want to hatch your bot?」と表示されたら、**「Do this later」** を選択します。

> Note: ARM ネイティブ向けビルドで問題に遭遇した場合は、Homebrew を使う前に、まずシステムパッケージ（例：`sudo apt install -y build-essential`）をインストールしてください。

<div id="6-configure-gateway-loopback-token-auth-and-enable-tailscale-serve">
  ## 6) Gateway を設定（ループバック + トークン認証）し、Tailscale Serve を有効にする
</div>

デフォルトはトークン認証にします。挙動が予測しやすく、「insecure auth」系の Control UI フラグを有効にする必要がなくなります。

```bash
# Keep the Gateway private on the VM
openclaw config set gateway.bind loopback

# Require auth for the Gateway + Control UI
openclaw config set gateway.auth.mode token
openclaw doctor --generate-gateway-token

# Tailscale Serve 経由で公開する (HTTPS + tailnet アクセス)
openclaw config set gateway.tailscale.mode serve
openclaw config set gateway.trustedProxies '["127.0.0.1"]'

systemctl --user restart openclaw-gateway
```

<div id="7-verify">
  ## 7) 確認
</div>

```bash
# バージョンを確認
openclaw --version

# デーモンステータスを確認
systemctl --user status openclaw-gateway

# Tailscale Serve を確認
tailscale serve status

# ローカル応答をテスト
curl http://localhost:18789
```

<div id="8-lock-down-vcn-security">
  ## 8) VCN セキュリティのロックダウン
</div>

すべてが動作していることを確認したら、Tailscale 以外のすべてのトラフィックをブロックするように VCN をロックダウンします。OCI の Virtual Cloud Network はネットワークエッジで動作するファイアウォールとして機能し、トラフィックはインスタンスに到達する前にブロックされます。

1. OCI コンソールで **Networking → Virtual Cloud Networks** に移動します
2. 対象の VCN をクリック → **Security Lists** → Default Security List を選択します
3. 次のルールを除くすべてのインバウンド (ingress) ルールを**削除**します:
   * `0.0.0.0/0 UDP 41641` (Tailscale)
4. 送信 (egress) ルールはデフォルトのまま維持します (すべてのアウトバウンドを許可)

これにより、ポート 22 の SSH、HTTP、HTTPS を含むすべてのトラフィックがネットワークエッジでブロックされます。今後は、Tailscale 経由でのみ接続できます。

***

<div id="access-the-control-ui">
  ## Control UI へアクセスする
</div>

Tailscale ネットワークに接続されている任意のデバイスから:

```
https://openclaw.<tailnet-name>.ts.net/
```

`tailscale status` に表示される tailnet 名で `<tailnet-name>` を置き換えてください。

SSH トンネルは不要です。Tailscale により以下が提供されます:

* HTTPS 暗号化（証明書は自動発行）
* Tailscale アイデンティティによる認証
* tailnet 上の任意のデバイス（ノートPC、スマートフォンなど）からのアクセス

***

<div id="security-vcn-tailscale-recommended-baseline">
  ## セキュリティ: VCN + Tailscale（推奨ベースライン）
</div>

VCN をロックダウンし（UDP 41641 のみ開放）、Gateway をループバックにバインドしておくことで、強力な多層防御を実現できます。パブリックトラフィックはネットワークエッジで遮断され、管理者アクセスは Tailnet 経由でのみ行われます。

この構成により、多くの場合、インターネット全体からの SSH 総当たり攻撃を止める「だけ」のための追加のホストベースのファイアウォール ルールは不要になります。ただし、OS を最新の状態に保ち、`openclaw security audit` を実行し、誤ってパブリックインターフェースで待ち受けるようになっていないことを必ず確認してください。

<div id="whats-already-protected">
  ### すでに保護されているもの
</div>

| 従来の対策 | 必要か | 理由 |
|------------------|---------|-----|
| UFW ファイアウォール | 不要 | トラフィックがインスタンスに到達する前に VCN がブロックするため |
| fail2ban | 不要 | ポート 22 が VCN でブロックされていれば総当たり攻撃は発生しないため |
| sshd の堅牢化 | 不要 | Tailscale SSH は sshd を使用しないため |
| root ログイン無効化 | 不要 | Tailscale はシステムユーザーではなく Tailscale のアイデンティティを使用するため |
| SSH 鍵のみでの認証 | 不要 | Tailscale は tailnet を通じて認証を行うため |
| IPv6 の堅牢化 | 通常は不要 | VCN/サブネット設定によって異なるため、実際にどのアドレスが割り当てられ／公開されているかを確認してください |

<div id="still-recommended">
  ### 引き続き推奨される作業
</div>

* **認証情報ディレクトリの権限:** `chmod 700 ~/.openclaw` を設定する
* **セキュリティ監査:** `openclaw security audit` を実行する
* **システム更新:** `sudo apt update && sudo apt upgrade` を定期的に実行する
* **Tailscale の監視:** [Tailscale admin console](https://login.tailscale.com/admin) でデバイスを確認する

<div id="verify-security-posture">
  ### セキュリティ体制を確認する
</div>

```bash
# パブリックポートがリッスンしていないことを確認
sudo ss -tlnp | grep -v '127.0.0.1\|::1'

# Tailscale SSHがアクティブであることを検証
tailscale status | grep -q 'offers: ssh' && echo "Tailscale SSH active"

# オプション: sshdを完全に無効化
sudo systemctl disable --now ssh
```

***

<div id="fallback-ssh-tunnel">
  ## フォールバック: SSH トンネル
</div>

Tailscale Serve が使えない場合は、SSH トンネルを使用してください。

```bash
# ローカルマシンから（Tailscale 経由）
ssh -L 18789:127.0.0.1:18789 ubuntu@openclaw
```

次に、`http://localhost:18789` を開きます。

***

<div id="troubleshooting">
  ## トラブルシューティング
</div>

<div id="instance-creation-fails-out-of-capacity">
  ### インスタンス作成が「Out of capacity」で失敗する
</div>

Free Tier の ARM インスタンスは人気があります。次を試してください:

* 別の可用性ドメインを選ぶ
* 混雑していない時間帯（早朝など）に再試行する
* シェイプを選択するときに「Always Free」フィルタを使用する

<div id="tailscale-wont-connect">
  ### Tailscale が接続できない
</div>

```bash
# ステータスを確認
sudo tailscale status

# 再認証
sudo tailscale up --ssh --hostname=openclaw --reset
```

<div id="gateway-wont-start">
  ### Gateway が起動しない
</div>

```bash
openclaw gateway status
openclaw doctor --non-interactive
journalctl --user -u openclaw-gateway -n 50
```

<div id="cant-reach-control-ui">
  ### Control UI にアクセスできない
</div>

```bash
# Tailscale Serveが実行されているか確認
tailscale serve status

# Check gateway is listening
curl http://localhost:18789

# Restart if needed
systemctl --user restart openclaw-gateway
```

<div id="arm-binary-issues">
  ### ARM バイナリに関する問題
</div>

一部のツールには ARM 向けビルドが存在しない場合があります。以下を確認してください。

```bash
uname -m  # aarch64 が表示されるはずです
```

ほとんどの npm パッケージは問題なく動作します。バイナリについては、`linux-arm64` または `aarch64` リリースを探してください。

***

<div id="persistence">
  ## 永続化
</div>

すべての状態は次の場所に保存されます。

* `~/.openclaw/` — 設定、認証情報、セッションデータ
* `~/.openclaw/workspace/` — ワークスペース（SOUL.md、メモリ、アーティファクト）

定期的にバックアップを実行してください。

```bash
tar -czvf openclaw-backup.tar.gz ~/.openclaw ~/.openclaw/workspace
```

***

<div id="see-also">
  ## 関連項目
</div>

* [Gateway のリモートアクセス](/ja/gateway/remote) — その他のリモートアクセスパターン
* [Tailscale 連携](/ja/gateway/tailscale) — Tailscale に関する詳細ドキュメント一式
* [Gateway 設定](/ja/gateway/configuration) — すべての設定オプション
* [DigitalOcean ガイド](/ja/platforms/digitalocean) — 有料でもよいので、より簡単にサインアップしたい場合
* [Hetzner ガイド](/ja/platforms/hetzner) — Docker ベースの代替手段