---
title: Gcp
summary: "状態を永続的に保持しながら OpenClaw Gateway を GCP Compute Engine VM（Docker）上で 24 時間 365 日稼働させる"
read_when:
  - OpenClaw を GCP 上で 24 時間 365 日稼働させたい
  - 自前の VM 上で、本番運用レベルの常時稼働 Gateway を動かしたい
  - 永続化方式、バイナリ、再起動時の挙動を完全に自分で制御したい
---

<div id="openclaw-on-gcp-compute-engine-docker-production-vps-guide">
  # GCP Compute Engine 上の OpenClaw（Docker、本番向け VPS ガイド）
</div>

<div id="goal">
  ## 目的
</div>

GCP Compute Engine の VM 上で Docker を使って OpenClaw Gateway を常時稼働させ、永続状態（データ）、バイナリをあらかじめ含んだ環境、安全な再起動動作を確保する。

もし「OpenClaw を 24 時間 365 日、月額およそ 5〜12 ドルで動かしたい」場合、これは Google Cloud 上で信頼性の高いセットアップである。
料金はマシンタイプとリージョンによって異なる。まずはワークロードに見合う最小の VM を選び、メモリ不足（OOM）が発生したらスケールアップすること。

<div id="what-are-we-doing-simple-terms">
  ## 何をするのか（シンプルな説明）
</div>

* GCP プロジェクトを作成し、課金を有効化する
* Compute Engine VM を作成する
* Docker（分離されたアプリ実行環境）をインストールする
* Docker 上で OpenClaw Gateway を起動する
* ホスト側で `~/.openclaw` と `~/.openclaw/workspace` を永続化する（再起動／再構築後もデータが保持されるようにする）
* SSH トンネル経由でノート PC から Control UI にアクセスする

Gateway には次の方法でアクセスできます:

* ノート PC からの SSH ポートフォワーディング
* 自分でファイアウォールとトークンを管理する場合にポートを直接公開する

このガイドでは、GCP Compute Engine 上の Debian を使用します。
Ubuntu でも動作しますが、パッケージ名などは適宜読み替えてください。
汎用的な Docker 手順については [Docker](/ja/install/docker) を参照してください。

***

<div id="quick-path-experienced-operators">
  ## クイックパス（経験豊富なオペレーター向け）
</div>

1. GCP プロジェクトを作成して、Compute Engine API を有効化する
2. Compute Engine VM（e2-small、Debian 12、20GB）を作成する
3. VM に SSH 接続する
4. Docker をインストールする
5. OpenClaw リポジトリをクローンする
6. 永続化用のホストディレクトリを作成する
7. `.env` と `docker-compose.yml` を設定する
8. 必要なバイナリを組み込んだイメージをビルドし、起動する

***

<div id="what-you-need">
  ## 必要なもの
</div>

* GCP アカウント（e2-micro 対象の無料枠）
* gcloud CLI がインストールされていること（または Cloud Console を使用）
* 手元のラップトップからの SSH アクセス
* SSH とコピー＆ペーストの基本的な操作に慣れていること
* 所要時間の目安: 約 20〜30 分
* Docker と Docker Compose
* モデルの認証情報
* オプションのプロバイダー認証情報
  * WhatsApp の QR コード
  * Telegram ボットトークン
  * Gmail OAuth

***

<div id="1-install-gcloud-cli-or-use-console">
  ## 1) gcloud CLI をインストールする（またはコンソールを使用）
</div>

**オプション A: gcloud CLI**（自動化用途に推奨）

https://cloud.google.com/sdk/docs/install からインストールします。

初期化と認証を行います:

```bash
gcloud init
gcloud auth login
```

**オプション B：Cloud Console**

すべての手順は https://console.cloud.google.com の Web UI から実行できます。

***

<div id="2-create-a-gcp-project">
  ## 2) GCP プロジェクトを作成する
</div>

**CLI：**

```bash
gcloud projects create my-openclaw-project --name="OpenClaw Gateway"
gcloud config set project my-openclaw-project
```

https://console.cloud.google.com/billing で課金を有効にしてください（Compute Engine に必須です）。

Compute Engine API を有効にします:

```bash
gcloud services enable compute.googleapis.com
```

**コンソール:**

1. 「IAM &amp; Admin」 &gt; 「Create Project」に移動する
2. プロジェクトに名前を付けて作成する
3. プロジェクトの課金を有効にする
4. 「APIs &amp; Services」 &gt; 「Enable APIs」 &gt; 「Compute Engine API」を検索して有効にする

***

<div id="3-create-the-vm">
  ## 3) VM を作成する
</div>

**マシンタイプ:**

| 種類       | スペック                | コスト     | 備考               |
| -------- | ------------------- | ------- | ---------------- |
| e2-small | 2 vCPU、2GB RAM      | 約 $12/月 | 推奨               |
| e2-micro | 2 vCPU (共有)、1GB RAM | 無料枠の対象  | 高負荷時に OOM になる可能性 |

**CLI:**

```bash
gcloud compute instances create openclaw-gateway \
  --zone=us-central1-a \
  --machine-type=e2-small \
  --boot-disk-size=20GB \
  --image-family=debian-12 \
  --image-project=debian-cloud
```

**コンソール:**

1. Compute Engine &gt; VM インスタンス &gt; インスタンスを作成 に移動します
2. 名前: `openclaw-gateway`
3. リージョン: `us-central1`、ゾーン: `us-central1-a`
4. マシンタイプ: `e2-small`
5. ブートディスク: Debian 12、20GB
6. 「作成」をクリックします

***

<div id="4-ssh-into-the-vm">
  ## 4) VM に SSH で接続する
</div>

**CLI:**

```bash
gcloud compute ssh openclaw-gateway --zone=us-central1-a
```

**コンソール:**

Compute Engine ダッシュボードで VM の横にある「SSH」ボタンをクリックします。

注意: VM 作成後、SSH キーが反映されるまでに 1～2 分かかることがあります。接続が拒否された場合は、少し待ってから再試行してください。

***

<div id="5-install-docker-on-the-vm">
  ## 5) VM 上に Docker をインストールする
</div>

```bash
sudo apt-get update
sudo apt-get install -y git curl ca-certificates
curl -fsSL https://get.docker.com | sudo sh
sudo usermod -aG docker $USER
```

グループ変更を反映させるために、一度ログアウトしてから再度ログインしてください。

```bash
exit
```

その後、再度 SSH で接続します:

```bash
gcloud compute ssh openclaw-gateway --zone=us-central1-a
```

確認:

```bash
docker --version
docker compose version
```

***

<div id="6-clone-the-openclaw-repository">
  ## 6) OpenClaw リポジトリをクローンする
</div>

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
```

このガイドでは、バイナリを永続化できるようカスタムイメージを構築することを前提としています。

***

<div id="7-create-persistent-host-directories">
  ## 7) 永続的なホストディレクトリの作成
</div>

Docker コンテナは本質的にエフェメラル（使い捨て）です。
長期間保持・永続化する必要がある状態は、すべてホスト側に配置してください。

```bash
mkdir -p ~/.openclaw
mkdir -p ~/.openclaw/workspace
```

***

<div id="8-configure-environment-variables">
  ## 8) 環境変数を設定する
</div>

リポジトリのルートディレクトリに `.env` を作成します。

```bash
OPENCLAW_IMAGE=openclaw:latest
OPENCLAW_GATEWAY_TOKEN=change-me-now
OPENCLAW_GATEWAY_BIND=lan
OPENCLAW_GATEWAY_PORT=18789

OPENCLAW_CONFIG_DIR=/home/$USER/.openclaw
OPENCLAW_WORKSPACE_DIR=/home/$USER/.openclaw/workspace

GOG_KEYRING_PASSWORD=change-me-now
XDG_CONFIG_HOME=/home/node/.openclaw
```

強度の高いシークレットを生成する：

```bash
openssl rand -hex 32
```

**このファイルはコミットしないでください。**

***

<div id="9-docker-compose-configuration">
  ## 9) Docker Compose の設定
</div>

`docker-compose.yml` を作成または更新します。

```yaml
services:
  openclaw-gateway:
    image: ${OPENCLAW_IMAGE}
    build: .
    restart: unless-stopped
    env_file:
      - .env
    environment:
      - HOME=/home/node
      - NODE_ENV=production
      - TERM=xterm-256color
      - OPENCLAW_GATEWAY_BIND=${OPENCLAW_GATEWAY_BIND}
      - OPENCLAW_GATEWAY_PORT=${OPENCLAW_GATEWAY_PORT}
      - OPENCLAW_GATEWAY_TOKEN=${OPENCLAW_GATEWAY_TOKEN}
      - GOG_KEYRING_PASSWORD=${GOG_KEYRING_PASSWORD}
      - XDG_CONFIG_HOME=${XDG_CONFIG_HOME}
      - PATH=/home/linuxbrew/.linuxbrew/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
    volumes:
      - ${OPENCLAW_CONFIG_DIR}:/home/node/.openclaw
      - ${OPENCLAW_WORKSPACE_DIR}:/home/node/.openclaw/workspace
    ports:
      # Recommended: keep the Gateway loopback-only on the VM; access via SSH tunnel.
      # To expose it publicly, remove the `127.0.0.1:` prefix and firewall accordingly.
      - "127.0.0.1:${OPENCLAW_GATEWAY_PORT}:18789"

      # オプション: このVMに対してiOS/Androidノードを実行し、Canvasホストが必要な場合のみ。
      # 公開する場合は、/gateway/securityを読み、適切にファイアウォールを設定してください。
      # - "18793:18793"
    command:
      [
        "node",
        "dist/index.js",
        "gateway",
        "--bind",
        "${OPENCLAW_GATEWAY_BIND}",
        "--port",
        "${OPENCLAW_GATEWAY_PORT}"
      ]
```

***

<div id="10-bake-required-binaries-into-the-image-critical">
  ## 10) 必要なバイナリをイメージに焼き込む（重要）
</div>

稼働中のコンテナ内でバイナリをインストールするのはアンチパターンです。
コンテナ起動後にインストールしたものは、再起動時にすべて失われます。

スキルが必要とする外部バイナリは、必ずイメージのビルド時にインストールしてください。

以下の例では、代表的な 3 つのバイナリのみを示しています：

* Gmail へのアクセス用 `gog`
* Google Places 用 `goplaces`
* WhatsApp 用 `wacli`

これらはあくまで例であり、完全な一覧ではありません。
同じパターンで、必要なだけ多くのバイナリをインストールしてかまいません。

後から新しいスキルを追加し、追加のバイナリに依存する場合は、必ず次を行ってください:

1. Dockerfile を更新する
2. イメージを再ビルドする
3. コンテナを再起動する

**Dockerfile の例**

```dockerfile
FROM node:22-bookworm

RUN apt-get update && apt-get install -y socat && rm -rf /var/lib/apt/lists/*

# バイナリ例1: Gmail CLI
RUN curl -L https://github.com/steipete/gog/releases/latest/download/gog_Linux_x86_64.tar.gz \
  | tar -xz -C /usr/local/bin && chmod +x /usr/local/bin/gog

# バイナリ例2: Google Places CLI
RUN curl -L https://github.com/steipete/goplaces/releases/latest/download/goplaces_Linux_x86_64.tar.gz \
  | tar -xz -C /usr/local/bin && chmod +x /usr/local/bin/goplaces

# バイナリ例3: WhatsApp CLI
RUN curl -L https://github.com/steipete/wacli/releases/latest/download/wacli_Linux_x86_64.tar.gz \
  | tar -xz -C /usr/local/bin && chmod +x /usr/local/bin/wacli

# 同じパターンを使用して以下にバイナリを追加

WORKDIR /app
COPY package.json pnpm-lock.yaml pnpm-workspace.yaml .npmrc ./
COPY ui/package.json ./ui/package.json
COPY scripts ./scripts

RUN corepack enable
RUN pnpm install --frozen-lockfile

COPY . .
RUN pnpm build
RUN pnpm ui:install
RUN pnpm ui:build

ENV NODE_ENV=production

CMD ["node","dist/index.js"]
```

***

<div id="11-build-and-launch">
  ## 11) ビルドと起動
</div>

```bash
docker compose build
docker compose up -d openclaw-gateway
```

バイナリの検証

```bash
docker compose exec openclaw-gateway which gog
docker compose exec openclaw-gateway which goplaces
docker compose exec openclaw-gateway which wacli
```

期待される出力：

```
/usr/local/bin/gog
/usr/local/bin/goplaces
/usr/local/bin/wacli
```

***

<div id="12-verify-gateway">
  ## 12) Gateway の動作を確認する
</div>

```bash
docker compose logs -f openclaw-gateway
```

成功：

```
[gateway] listening on ws://0.0.0.0:18789
```

***

<div id="13-access-from-your-laptop">
  ## 13) ローカルマシン（ラップトップ）からのアクセス
</div>

SSH トンネルを作成して Gateway のポートを転送します：

```bash
gcloud compute ssh openclaw-gateway --zone=us-central1-a -- -L 18789:127.0.0.1:18789
```

ブラウザで次のURLを開きます:

`http://127.0.0.1:18789/`

Gateway トークンを貼り付けてください。

***

<div id="what-persists-where-source-of-truth">
  ## どのデータがどこで永続化されるか（ソース・オブ・トゥルース）
</div>

OpenClaw は Docker 上で実行されますが、Docker 自体がソース・オブ・トゥルースではありません。
すべての永続的な状態は、コンテナの再起動・再ビルドやホストの再起動（リブート）をまたいで保持されなければなりません。

| コンポーネント | 場所 | 永続化方法 | 備考 |
|---|---|---|---|
| Gateway 設定 | `/home/node/.openclaw/` | ホストボリュームマウント | `openclaw.json`、トークンを含む |
| モデル認証プロファイル | `/home/node/.openclaw/` | ホストボリュームマウント | OAuth トークン、API キー |
| スキル設定 | `/home/node/.openclaw/skills/` | ホストボリュームマウント | スキル単位の状態 |
| エージェントのワークスペース | `/home/node/.openclaw/workspace/` | ホストボリュームマウント | コードおよびエージェントのアーティファクト |
| WhatsApp セッション | `/home/node/.openclaw/` | ホストボリュームマウント | QR ログインを保持 |
| Gmail キーリング | `/home/node/.openclaw/` | ホストボリューム + パスワード | `GOG_KEYRING_PASSWORD` が必要 |
| 外部バイナリ | `/usr/local/bin/` | Docker イメージ | ビルド時にイメージへ埋め込む必要あり |
| Node.js ランタイム | コンテナファイルシステム | Docker イメージ | イメージビルドごとに再ビルドされる |
| OS パッケージ | コンテナファイルシステム | Docker イメージ | 実行時にインストールしないこと |
| Docker コンテナ | 揮発的 | 再起動可能 | 破棄しても安全 |

***

<div id="updates">
  ## 更新
</div>

VM 上で OpenClaw を更新するには：

```bash
cd ~/openclaw
git pull
docker compose build
docker compose up -d
```

***

<div id="troubleshooting">
  ## トラブルシューティング
</div>

**SSH 接続が拒否される**

VM 作成後、SSH キーの反映には 1〜2 分ほどかかる場合があります。少し待ってから再度試してください。

**OS Login の問題**

OS Login プロファイルを確認してください。

```bash
gcloud compute os-login describe-profile
```

使用しているアカウントに必要な IAM ロール（Compute OS Login または Compute OS Admin Login）が付与されていることを確認してください。

**Out of memory (OOM)**

e2-micro を使用していて OOM が発生する場合は、e2-small または e2-medium にアップグレードしてください。

```bash
# Stop the VM first
gcloud compute instances stop openclaw-gateway --zone=us-central1-a

# マシンタイプを変更する
gcloud compute instances set-machine-type openclaw-gateway \
  --zone=us-central1-a \
  --machine-type=e2-small

# Start the VM
gcloud compute instances start openclaw-gateway --zone=us-central1-a
```

***

<div id="service-accounts-security-best-practice">
  ## サービス アカウント（セキュリティのベストプラクティス）
</div>

個人利用であれば、デフォルトのユーザー アカウントで問題ありません。

自動化や CI/CD パイプライン向けには、最小限の権限だけを付与した専用のサービス アカウントを作成してください。

1. サービス アカウントを作成します:
   ```bash
   gcloud iam service-accounts create openclaw-deploy \
     --display-name="OpenClaw Deployment"
   ```

2. Compute Instance Admin ロール（またはそれより権限を絞ったカスタム ロール）を付与します:
   ```bash
   gcloud projects add-iam-policy-binding my-openclaw-project \
     --member="serviceAccount:openclaw-deploy@my-openclaw-project.iam.gserviceaccount.com" \
     --role="roles/compute.instanceAdmin.v1"
   ```

自動化用途では Owner ロールの使用を避けてください。最小権限の原則を徹底してください。

IAM ロールの詳細については https://cloud.google.com/iam/docs/understanding-roles を参照してください。

***

<div id="next-steps">
  ## 次のステップ
</div>

* メッセージングチャネルを設定する: [Channels](/ja/channels)
* ローカルデバイスをノードとしてペアリングする: [Nodes](/ja/nodes)
* Gateway を設定する: [Gateway configuration](/ja/gateway/configuration)