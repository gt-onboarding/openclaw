---
title: Hetzner
summary: "安価な Hetzner VPS 上で OpenClaw Gateway を Docker で 24/7 稼働させ、永続的な状態と組み込みバイナリを確保する"
read_when:
  - ノートPCではなくクラウド VPS 上で OpenClaw を 24/7 動かしたい
  - 自前の VPS 上で本番運用グレードの常時稼働 Gateway を動かしたい
  - 永続化、バイナリ、再起動動作を完全に自分で管理したい
  - Hetzner または類似プロバイダー上で Docker を使って OpenClaw を実行している
---

<div id="openclaw-on-hetzner-docker-production-vps-guide">
  # Hetzner での OpenClaw（Docker 本番運用向け VPS ガイド）
</div>

<div id="goal">
  ## 目標
</div>

Hetzner の VPS 上で Docker を用いて、永続的な状態・あらかじめバイナリを組み込んだイメージ・安全な再起動動作を備えた常駐の OpenClaw Gateway を稼働させる。

「約 5 ドルで OpenClaw を 24 時間 365 日動かしたい」場合、これは最も簡単で信頼性の高いセットアップである。
Hetzner の料金は変動するため、最小構成の Debian/Ubuntu VPS を選び、メモリ不足（OOM）が発生したらスケールアップすること。

<div id="what-are-we-doing-simple-terms">
  ## ここでやること（ざっくり）
</div>

* 小さな Linux サーバー（Hetzner VPS）を借りる
* Docker（分離されたアプリ実行環境）をインストールする
* Docker 上で OpenClaw Gateway を起動する
* ホスト側に `~/.openclaw` と `~/.openclaw/workspace` を永続化する（再起動／再構築後も残す）
* SSH トンネル経由でノート PC から Control UI にアクセスする

Gateway には次の方法でアクセスできます:

* ノート PC からの SSH ポートフォワーディング
* 自分でファイアウォールとトークン管理を行う前提でポートを直接公開する

このガイドは Hetzner 上の Ubuntu または Debian を前提としています。\
別の Linux VPS を利用している場合は、パッケージ等を環境に合わせて読み替えてください。\
一般的な Docker 手順については [Docker](/ja/install/docker) を参照してください。

***

<div id="quick-path-experienced-operators">
  ## クイックスタート（経験豊富なオペレーター向け）
</div>

1. Hetzner VPS をプロビジョニングする
2. Docker をインストールする
3. OpenClaw リポジトリをクローンする
4. 永続データ用のホストディレクトリを作成する
5. `.env` と `docker-compose.yml` を設定する
6. 必要なバイナリをイメージに組み込む
7. `docker compose up -d` を実行する
8. データの永続化と Gateway へのアクセスを確認する

***

<div id="what-you-need">
  ## 必要なもの
</div>

* root アクセス権のある Hetzner VPS
* 手元のノートPCからの SSH アクセス
* SSH とコピー＆ペーストの基本的な操作に慣れていること
* 作業時間の目安: 約 20 分
* Docker と Docker Compose
* モデル用の認証情報
* プロバイダー用の認証情報（任意）
  * WhatsApp QR
  * Telegram ボットトークン
  * Gmail OAuth

***

<div id="1-provision-the-vps">
  ## 1) VPS をプロビジョニングする
</div>

Hetzner で Ubuntu または Debian の VPS を作成します。

root ユーザーとして接続します:

```bash
ssh root@YOUR_VPS_IP
```

このガイドでは、VPS をステートフルに運用することを前提としています。
使い捨てのインフラとして扱わないでください。

***

<div id="2-install-docker-on-the-vps">
  ## 2) Docker をインストールする (VPS 上で)
</div>

```bash
apt-get update
apt-get install -y git curl ca-certificates
curl -fsSL https://get.docker.com | sh
```

確認:

```bash
docker --version
docker compose version
```

***

<div id="3-clone-the-openclaw-repository">
  ## 3) OpenClaw のリポジトリをクローンする
</div>

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
```

このガイドでは、バイナリを永続的に保持できるように、カスタムイメージをビルドすることを前提とします。

***

<div id="4-create-persistent-host-directories">
  ## 4) 永続的なホストディレクトリを作成する
</div>

Docker コンテナは基本的に一時的（ephemeral）なものです。
長期間保持する必要のある状態はすべてホスト側に持たせる必要があります。

```bash
mkdir -p /root/.openclaw
mkdir -p /root/.openclaw/workspace

# コンテナユーザー (uid 1000) に所有権を設定:
chown -R 1000:1000 /root/.openclaw
chown -R 1000:1000 /root/.openclaw/workspace
```

***

<div id="5-configure-environment-variables">
  ## 5) 環境変数を設定する
</div>

リポジトリのルートに `.env` を作成します。

```bash
OPENCLAW_IMAGE=openclaw:latest
OPENCLAW_GATEWAY_TOKEN=change-me-now
OPENCLAW_GATEWAY_BIND=lan
OPENCLAW_GATEWAY_PORT=18789

OPENCLAW_CONFIG_DIR=/root/.openclaw
OPENCLAW_WORKSPACE_DIR=/root/.openclaw/workspace

GOG_KEYRING_PASSWORD=change-me-now
XDG_CONFIG_HOME=/home/node/.openclaw
```

強力なシークレットを生成する：

```bash
openssl rand -hex 32
```

**このファイルはコミットしないでください。**

***

<div id="6-docker-compose-configuration">
  ## 6) Docker Compose の設定
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
      # Recommended: keep the Gateway loopback-only on the VPS; access via SSH tunnel.
      # To expose it publicly, remove the `127.0.0.1:` prefix and firewall accordingly.
      - "127.0.0.1:${OPENCLAW_GATEWAY_PORT}:18789"

      # オプション: このVPSに対してiOS/Androidノードを実行し、Canvasホストが必要な場合のみ。
      # 公開する場合は、/gateway/securityを読み、適切にファイアウォールを設定すること。
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

<div id="7-bake-required-binaries-into-the-image-critical">
  ## 7) 必要なバイナリをイメージに焼き込む（重要）
</div>

稼働中のコンテナ内でバイナリをインストールするのは落とし穴です。
コンテナ起動後にインストールしたものは、再起動時にすべて失われます。

スキルが必要とするすべての外部バイナリは、イメージのビルド時にインストールしておく必要があります。

以下の例では、代表的な 3 つのバイナリのみを示しています:

* Gmail アクセス用の `gog`
* Google Places 用の `goplaces`
* WhatsApp 用の `wacli`

これらはあくまで例であり、完全な一覧ではありません。
同じパターンで、必要なだけバイナリをインストールできます。

後から追加のバイナリに依存する新しいスキルを追加した場合は、必ず次を実施してください:

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

<div id="8-build-and-launch">
  ## 8) ビルドと起動
</div>

```bash
docker compose build
docker compose up -d openclaw-gateway
```

バイナリの検証:

```bash
docker compose exec openclaw-gateway which gog
docker compose exec openclaw-gateway which goplaces
docker compose exec openclaw-gateway which wacli
```

想定される出力例:

```
/usr/local/bin/gog
/usr/local/bin/goplaces
/usr/local/bin/wacli
```

***

<div id="9-verify-gateway">
  ## 9) Gateway の動作確認
</div>

```bash
docker compose logs -f openclaw-gateway
```

成功：

```
[gateway] listening on ws://0.0.0.0:18789
```

手元のノートPCから:

```bash
ssh -N -L 18789:127.0.0.1:18789 root@YOUR_VPS_IP
```

次を開く:

`http://127.0.0.1:18789/`

Gateway トークンを貼り付けてください。

***

<div id="what-persists-where-source-of-truth">
  ## どこに何が永続化されるか（信頼できる情報源）
</div>

OpenClaw は Docker 上で動作しますが、信頼できる情報源は Docker ではありません。
すべての長期的な状態は、コンテナの再起動・イメージの再ビルド・ホスト OS のリブート後も保持されていなければなりません。

| コンポーネント | 場所 | 永続化メカニズム | 備考 |
|---|---|---|---|
| Gateway 設定 | `/home/node/.openclaw/` | ホストボリュームのマウント | `openclaw.json` やトークンを含む |
| モデル認証プロファイル | `/home/node/.openclaw/` | ホストボリュームのマウント | OAuth トークン、API キー |
| スキル設定 | `/home/node/.openclaw/skills/` | ホストボリュームのマウント | スキル単位の状態 |
| エージェントのワークスペース | `/home/node/.openclaw/workspace/` | ホストボリュームのマウント | コードとエージェントの成果物 |
| WhatsApp セッション | `/home/node/.openclaw/` | ホストボリュームのマウント | QR ログイン状態を保持 |
| Gmail キーリング | `/home/node/.openclaw/` | ホストボリューム + パスワード | `GOG_KEYRING_PASSWORD` が必要 |
| 外部バイナリ | `/usr/local/bin/` | Docker イメージ | ビルド時にイメージへ組み込む必要あり |
| Node ランタイム | コンテナファイルシステム | Docker イメージ | イメージビルドごとに再作成される |
| OS パッケージ | コンテナファイルシステム | Docker イメージ | コンテナ実行中にインストールしないこと |
| Docker コンテナ | 一時的 | 再起動可能 | 破棄しても安全 |