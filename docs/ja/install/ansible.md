---
title: Ansible
summary: "Ansible、Tailscale VPN、ファイアウォール分離による自動化された堅牢な OpenClaw のインストール"
read_when:
  - セキュリティを強化したサーバー環境を自動デプロイしたい場合
  - VPN 経由でアクセスするファイアウォール分離環境が必要な場合
  - リモートの Debian/Ubuntu サーバーにデプロイする場合
---

<div id="ansible-installation">
  # Ansible によるインストール
</div>

本番環境のサーバーに OpenClaw をデプロイする推奨方法は、セキュリティを最優先に設計された自動インストーラーである **[openclaw-ansible](https://github.com/openclaw/openclaw-ansible)** を利用することです。

<div id="quick-start">
  ## クイックスタート
</div>

1 コマンドでインストール:

```bash
curl -fsSL https://raw.githubusercontent.com/openclaw/openclaw-ansible/main/install.sh | bash
```

> **📦 完全ガイド: [github.com/openclaw/openclaw-ansible](https://github.com/openclaw/openclaw-ansible)**
>
> openclaw-ansible リポジトリが、Ansible によるデプロイの公式な情報源です。このページは、その簡単な概要です。

<div id="what-you-get">
  ## 得られるもの
</div>

* 🔒 **ファイアウォール優先のセキュリティ**: UFW + Docker による分離（SSH と Tailscale のみアクセス可能）
* 🔐 **Tailscale VPN**: サービスをインターネットに公開せずに安全なリモートアクセス
* 🐳 **Docker**: 分離されたサンドボックスコンテナと localhost のみへのバインド
* 🛡️ **多層防御**: 4 層のセキュリティアーキテクチャ
* 🚀 **ワンコマンドセットアップ**: 数分で完了するデプロイ
* 🔧 **systemd 連携**: 起動時の自動起動とハードニング

<div id="requirements">
  ## 要件
</div>

* **OS**: Debian 11+ または Ubuntu 20.04+
* **アクセス権限**: root または sudo 権限
* **ネットワーク**: パッケージのインストールに必要なインターネット接続
* **Ansible**: 2.14+（クイックスタートスクリプトにより自動的にインストールされます）

<div id="what-gets-installed">
  ## 何がインストールされるか
</div>

Ansible の playbook は、以下をインストールおよび設定します:

1. **Tailscale**（安全なリモートアクセスのためのメッシュ VPN）
2. **UFW ファイアウォール**（SSH と Tailscale 用ポートのみ許可）
3. **Docker CE + Compose V2**（エージェントサンドボックス用）
4. **Node.js 22.x + pnpm**（実行環境の依存関係）
5. **OpenClaw**（コンテナ化ではなくホスト上で動作）
6. **Systemd サービス**（セキュリティ強化を施した自動起動）

注意: Gateway は **Docker ではなくホスト上で直接** 実行されますが、エージェントサンドボックスは分離のために Docker を使用します。詳細は [Sandboxing](/ja/gateway/sandboxing) を参照してください。

<div id="post-install-setup">
  ## インストール後のセットアップ
</div>

インストールが完了したら、`openclaw` ユーザーに切り替えてください。

```bash
sudo -i -u openclaw
```

インストール後のスクリプトでは、次のセットアップを順番に行います:

1. **オンボーディングウィザード**: OpenClaw の基本設定を行う
2. **プロバイダーへのログイン**: WhatsApp/Telegram/Discord/Signal を接続する
3. **Gateway のテスト**: インストールが正しく行われたか確認する
4. **Tailscale のセットアップ**: 自身の VPN メッシュネットワークに接続する

<div id="quick-commands">
  ### よく使うコマンド
</div>

```bash
# サービスステータスを確認
sudo systemctl status openclaw

# ライブログを表示
sudo journalctl -u openclaw -f

# Gatewayを再起動
sudo systemctl restart openclaw

# プロバイダーログイン（openclawユーザーとして実行）
sudo -i -u openclaw
openclaw channels login
```

<div id="security-architecture">
  ## セキュリティアーキテクチャ
</div>

<div id="4-layer-defense">
  ### 4層防御
</div>

1. **Firewall (UFW)**: 外部に公開されるのは SSH (22) と Tailscale (41641/udp) のみ
2. **VPN (Tailscale)**: Gateway にはメッシュ VPN 経由でのみアクセス可能
3. **Docker Isolation**: DOCKER-USER iptables チェインにより外部へのポート公開を防止
4. **Systemd Hardening**: NoNewPrivileges、PrivateTmp、非特権ユーザー

<div id="verification">
  ### 検証
</div>

外部向けの攻撃面をテストする：

```bash
nmap -p- YOUR_SERVER_IP
```

**ポート 22 のみ**（SSH）が開いていると表示されている必要があります。その他のサービス（Gateway や Docker）はすべて閉じられている必要があります。

<div id="docker-availability">
  ### Docker のサポート状況
</div>

Docker は **エージェントのサンドボックス**（ツールを隔離して実行するため）のためにインストールされています。Gateway 自体を実行するためのものではありません。Gateway は localhost のみにバインドされ、Tailscale VPN 経由でアクセスできます。

サンドボックスの設定については、[Multi-Agent Sandbox &amp; Tools](/ja/multi-agent-sandbox-tools) を参照してください。

<div id="manual-installation">
  ## 手動インストール
</div>

自動化ではなく手動で制御したい場合は:

```bash
# 1. Install prerequisites
sudo apt update && sudo apt install -y ansible git

# 2. Clone repository
git clone https://github.com/openclaw/openclaw-ansible.git
cd openclaw-ansible

# 3. Install Ansible collections
ansible-galaxy collection install -r requirements.yml

# 4. Run playbook
./run-playbook.sh

# または直接実行（実行後、手動で /tmp/openclaw-setup.sh を実行する必要があります）
# ansible-playbook playbook.yml --ask-become-pass
```

<div id="updating-openclaw">
  ## OpenClaw の更新
</div>

Ansible インストーラーは、OpenClaw を手動で更新できるように構成します。標準的な更新手順については、[Updating](/ja/install/updating) を参照してください。

Ansible の playbook を再実行する場合（例: 設定変更を反映するため）:

```bash
cd openclaw-ansible
./run-playbook.sh
```

注: この処理は冪等であり、複数回実行しても問題ありません。

<div id="troubleshooting">
  ## トラブルシューティング
</div>

<div id="firewall-blocks-my-connection">
  ### ファイアウォールが接続をブロックしている
</div>

ロックアウトされてしまった場合は、次の点を確認してください:

* まず Tailscale VPN 経由でアクセスできるか確認してください
* SSH アクセス（ポート 22）は常に許可されています
* Gateway には、設計上 **Tailscale 経由でのみ** アクセスできます

<div id="service-wont-start">
  ### サービスが起動しない
</div>

```bash
# ログを確認する
sudo journalctl -u openclaw -n 100

# パーミッションを確認する
sudo ls -la /opt/openclaw

# 手動起動をテストする
sudo -i -u openclaw
cd ~/openclaw
pnpm start
```

<div id="docker-sandbox-issues">
  ### Docker サンドボックス関連の問題
</div>

```bash
# Dockerが実行中であることを確認
sudo systemctl status docker

# サンドボックスイメージを確認
sudo docker images | grep openclaw-sandbox

# サンドボックスイメージが存在しない場合はビルド
cd /opt/openclaw/openclaw
sudo -u openclaw ./scripts/sandbox-setup.sh
```

<div id="provider-login-fails">
  ### プロバイダーへのログインに失敗する
</div>

`openclaw` ユーザーとして実行していることを確認してください。

```bash
sudo -i -u openclaw
openclaw channels login
```

<div id="advanced-configuration">
  ## 高度な設定
</div>

セキュリティアーキテクチャおよびトラブルシューティングに関する詳細情報については、次を参照してください。

* [セキュリティアーキテクチャ](https://github.com/openclaw/openclaw-ansible/blob/main/docs/security.md)
* [技術詳細](https://github.com/openclaw/openclaw-ansible/blob/main/docs/architecture.md)
* [トラブルシューティングガイド](https://github.com/openclaw/openclaw-ansible/blob/main/docs/troubleshooting.md)

<div id="related">
  ## 関連項目
</div>

* [openclaw-ansible](https://github.com/openclaw/openclaw-ansible) — デプロイメントのフルガイド
* [Docker](/ja/install/docker) — Gateway のコンテナ化セットアップ
* [Sandboxing](/ja/gateway/sandboxing) — エージェント用サンドボックス設定
* [Multi-Agent Sandbox &amp; Tools](/ja/multi-agent-sandbox-tools) — エージェント単位の分離