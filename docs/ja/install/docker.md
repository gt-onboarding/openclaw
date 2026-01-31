---
title: Docker
summary: "OpenClaw 向けのオプションの Docker ベースセットアップと導入"
read_when:
  - ローカルインストールではなく、コンテナ化された Gateway を利用したい場合
  - Docker ベースのセットアップフローを検証している場合
---

<div id="docker-optional">
  # Docker（任意）
</div>

Docker の利用は**必須ではありません**。コンテナ化された Gateway を使いたい場合、または Docker のワークフローを検証したい場合にのみ使用してください。

<div id="is-docker-right-for-me">
  ## Docker は自分に向いているか？
</div>

* **はい**: 分離された、すぐに破棄できる一時的な Gateway 環境がほしい場合、またはローカルインストールなしでホスト上で OpenClaw を動かしたい場合。
* **いいえ**: 自分のマシン上で動かしており、とにかく最速の開発ループだけが目的の場合。代わりに通常のインストール手順を使ってください。
* **サンドボックスに関する注意**: エージェントのサンドボックス化も Docker を使用しますが、Gateway 全体を Docker コンテナ内で動かす必要は **ありません**。詳しくは [Sandboxing](/ja/gateway/sandboxing) を参照してください。

このガイドでは次の内容を扱います:

* コンテナ化された Gateway（Docker 上でフルの OpenClaw を実行）
* セッションごとのエージェントサンドボックス（ホスト上の Gateway + Docker で分離されたエージェントツール）

サンドボックスの詳細: [Sandboxing](/ja/gateway/sandboxing)

<div id="requirements">
  ## 要件
</div>

* Docker Desktop（または Docker Engine）+ Docker Compose v2
* イメージとログ用の十分なディスク容量

<div id="containerized-gateway-docker-compose">
  ## コンテナ版 Gateway（Docker Compose）
</div>

<div id="quick-start-recommended">
  ### クイックスタート（推奨）
</div>

リポジトリのルートディレクトリで実行します:

```bash
./docker-setup.sh
```

このスクリプトは次の処理を行います:

* Gateway イメージをビルドします
* オンボーディングウィザードを実行します
* プロバイダー設定のためのオプションのヒントを表示します
* Docker Compose 経由で Gateway を起動します
* Gateway トークンを生成し、`.env` に書き込みます

オプションの環境変数:

* `OPENCLAW_DOCKER_APT_PACKAGES` — ビルド中に追加の apt パッケージをインストール
* `OPENCLAW_EXTRA_MOUNTS` — 追加のホスト bind mount を指定
* `OPENCLAW_HOME_VOLUME` — `/home/node` を名前付きボリュームとして永続化

実行が完了したら:

* ブラウザで `http://127.0.0.1:18789/` を開きます。
* Control UI の (Settings → token) にトークンを貼り付けます。

ホスト側に設定/ワークスペースを書き出します:

* `~/.openclaw/`
* `~/.openclaw/workspace`

VPS 上で実行していますか？[Hetzner (Docker VPS)](/ja/platforms/hetzner) を参照してください。

<div id="manual-flow-compose">
  ### 手動手順（Compose）
</div>

```bash
docker build -t openclaw:local -f Dockerfile .
docker compose run --rm openclaw-cli onboard
docker compose up -d openclaw-gateway
```

<div id="extra-mounts-optional">
  ### 追加マウント（任意）
</div>

コンテナにホスト側ディレクトリを追加でマウントしたい場合は、
`docker-setup.sh` を実行する前に `OPENCLAW_EXTRA_MOUNTS` を設定します。これは、
カンマ区切りの Docker の bind mount のリストを受け取り、`docker-compose.extra.yml`
を生成して `openclaw-gateway` と `openclaw-cli` の両方に適用します。

例:

```bash
export OPENCLAW_EXTRA_MOUNTS="$HOME/.codex:/home/node/.codex:ro,$HOME/github:/home/node/github:rw"
./docker-setup.sh
```

注意:

* パスは macOS/Windows の Docker Desktop で共有済みである必要があります。
* `OPENCLAW_EXTRA_MOUNTS` を編集した場合は、追加の compose ファイルを再生成するために `docker-setup.sh` を再実行してください。
* `docker-compose.extra.yml` は自動生成されます。手動で編集しないでください。

<div id="persist-the-entire-container-home-optional">
  ### コンテナのホーム全体を永続化する（任意）
</div>

コンテナを再作成しても `/home/node` を保持したい場合は、`OPENCLAW_HOME_VOLUME` で
名前付きボリュームを設定します。これにより Docker ボリュームが作成されて
`/home/node` にマウントされますが、標準の config/workspace のバインドマウントはそのまま維持されます。
ここでは名前付きボリューム（バインドパスではない）を使用してください。バインドマウントには
`OPENCLAW_EXTRA_MOUNTS` を使用します。

例:

```bash
export OPENCLAW_HOME_VOLUME="openclaw_home"
./docker-setup.sh
```

これを追加のマウント設定と組み合わせて使用することもできます：

```bash
export OPENCLAW_HOME_VOLUME="openclaw_home"
export OPENCLAW_EXTRA_MOUNTS="$HOME/.codex:/home/node/.codex:ro,$HOME/github:/home/node/github:rw"
./docker-setup.sh
```

注意:

* `OPENCLAW_HOME_VOLUME` を変更した場合は、追加の Compose ファイルを再生成するために `docker-setup.sh` を再実行してください。
* 名前付きボリュームは、`docker volume rm <name>` で削除するまで残ります。

<div id="install-extra-apt-packages-optional">
  ### 追加の apt パッケージをインストールする（オプション）
</div>

イメージ内でシステムパッケージ（例: ビルドツールやメディアライブラリ）が必要な場合は、`docker-setup.sh` を実行する前に `OPENCLAW_DOCKER_APT_PACKAGES` を設定しておきます。
これにより、イメージのビルド時にパッケージがインストールされるため、コンテナを削除してもインストール済みのパッケージはイメージ内に保持されます。

例:

```bash
export OPENCLAW_DOCKER_APT_PACKAGES="ffmpeg build-essential"
./docker-setup.sh
```

Notes:

* ここでは、スペース区切りの apt パッケージ名のリストを指定できます。
* `OPENCLAW_DOCKER_APT_PACKAGES` を変更した場合は、イメージを再ビルドするために `docker-setup.sh` を再実行してください。

<div id="faster-rebuilds-recommended">
  ### より高速なリビルド（推奨）
</div>

リビルドを高速化するには、依存関係レイヤーがキャッシュされるように Dockerfile の記述順序を調整します。
これにより、ロックファイルが変更されない限り `pnpm install` を再実行する必要がなくなります。

```dockerfile
FROM node:22-bookworm

# Bunをインストール（ビルドスクリプトに必要）
RUN curl -fsSL https://bun.sh/install | bash
ENV PATH="/root/.bun/bin:${PATH}"

RUN corepack enable

WORKDIR /app

# パッケージメタデータが変更されない限り、依存関係をキャッシュ
COPY package.json pnpm-lock.yaml pnpm-workspace.yaml .npmrc ./
COPY ui/package.json ./ui/package.json
COPY scripts ./scripts

RUN pnpm install --frozen-lockfile

COPY . .
RUN pnpm build
RUN pnpm ui:install
RUN pnpm ui:build

ENV NODE_ENV=production

CMD ["node","dist/index.js"]
```

<div id="channel-setup-optional">
  ### チャンネルのセットアップ（任意）
</div>

CLI コンテナを使用してチャンネルを構成し、必要に応じて Gateway を再起動してください。

WhatsApp（QR）:

```bash
docker compose run --rm openclaw-cli channels login
```

Telegram（ボットトークン）：

```bash
docker compose run --rm openclaw-cli channels add --channel telegram --token "<token>"
```

Discord（ボットトークン）：

```bash
docker compose run --rm openclaw-cli channels add --channel discord --token "<token>"
```

ドキュメント: [WhatsApp](/ja/channels/whatsapp), [Telegram](/ja/channels/telegram), [Discord](/ja/channels/discord)

<div id="health-check">
  ### ヘルスチェック
</div>

```bash
docker compose exec openclaw-gateway node dist/index.js health --token "$OPENCLAW_GATEWAY_TOKEN"
```

<div id="e2e-smoke-test-docker">
  ### E2E スモークテスト（Docker）
</div>

```bash
scripts/e2e/onboard-docker.sh
```

<div id="qr-import-smoke-test-docker">
  ### QR インポートのスモークテスト（Docker）
</div>

```bash
pnpm test:docker:qr
```

<div id="notes">
  ### 注意事項
</div>

* コンテナ利用時の Gateway のバインド先アドレスのデフォルトは `lan` です。
* セッション（`~/.openclaw/agents/<agentId>/sessions/`）に関しては、Gateway コンテナが唯一の正とみなされるソースです。

<div id="agent-sandbox-host-gateway-docker-tools">
  ## エージェント用サンドボックス (ホスト Gateway + Docker ツール)
</div>

詳細: [サンドボックス機能](/ja/gateway/sandboxing)

<div id="what-it-does">
  ### 動作概要
</div>

`agents.defaults.sandbox` が有効な場合、**メイン以外のセッション** はツールを Docker
コンテナ内で実行します。Gateway はホスト上で動作し続けますが、ツールの実行は分離されます。

* scope: デフォルトは `"agent"`（エージェントごとに 1 コンテナ + 1 ワークスペース）
* scope: `"session"` を指定するとセッション単位で分離
* スコープごとのワークスペースフォルダが `/workspace` にマウントされる
* エージェントのワークスペースへのオプションのアクセス設定（`agents.defaults.sandbox.workspaceAccess`）
* ツール用の allow/deny ポリシー（deny が優先）
* 受信メディアは、アクティブなサンドボックスのワークスペース（`media/inbound/*`）にコピーされ、ツールが読み取れるようになる（`workspaceAccess: "rw"` の場合、これはエージェントのワークスペースに入る）

警告: `scope: "shared"` はセッション間の分離を無効にします。すべてのセッションが
1 つのコンテナと 1 つのワークスペースを共有します。

<div id="per-agent-sandbox-profiles-multi-agent">
  ### エージェントごとのサンドボックスプロファイル（マルチエージェント）
</div>

マルチエージェントルーティングを使用する場合、各エージェントはサンドボックスとツール設定を上書き（オーバーライド）できます：
`agents.list[].sandbox` と `agents.list[].tools`（および `agents.list[].tools.sandbox.tools`）。これにより、1つの Gateway 内で
異なるアクセスレベルを混在させて運用できます：

* フルアクセス（個人用エージェント）
* 読み取り専用ツール + 読み取り専用ワークスペース（家族/業務用エージェント）
* ファイルシステム/シェル系ツールなし（公開エージェント）

具体例・優先順位・トラブルシューティングについては [マルチエージェントのサンドボックスとツール](/ja/multi-agent-sandbox-tools) を参照してください。

<div id="default-behavior">
  ### 既定の動作
</div>

* イメージ: `openclaw-sandbox:bookworm-slim`
* エージェントごとに 1 コンテナ
* エージェントのワークスペースアクセス: `workspaceAccess: "none"`（デフォルト）の場合は `~/.openclaw/sandboxes` を使用
  * `"ro"` はサンドボックスのワークスペースを `/workspace` に保持し、エージェントのワークスペースを `/agent` に読み取り専用でマウントします（`write` / `edit` / `apply_patch` を無効化）
  * `"rw"` はエージェントのワークスペースを `/workspace` に読み書き可能でマウントします
* 自動削除: アイドル時間が 24 時間超 または 存続期間が 7 日超
* ネットワーク: デフォルトは `none`（外部への通信が必要な場合は明示的に有効化）
* デフォルト許可: `exec`, `process`, `read`, `write`, `edit`, `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status`
* デフォルト拒否: `browser`, `canvas`, `nodes`, `cron`, `discord`, `gateway`

<div id="enable-sandboxing">
  ### サンドボックス機能を有効化する
</div>

`setupCommand` 内でパッケージをインストールする予定がある場合は、次の点に注意してください:

* デフォルトの `docker.network` は `"none"`（外部ネットワークへの通信なし）です。
* `readOnlyRoot: true` はパッケージのインストールをブロックします。
* `apt-get` を利用するには `user` が root である必要があります（`user` を省略するか、`user: "0:0"` を設定してください）。
  OpenClaw は、コンテナが**直近に使用されていた**（おおよそ直近5分以内に使用）場合を除き、
  `setupCommand`（または Docker 設定）が変更されるとコンテナを自動的に再作成します。
  ホットなコンテナでは、正確な `openclaw sandbox recreate ...` コマンドを含む警告ログが出力されます。

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main", // off | non-main | all
        scope: "agent", // session | agent | shared (agent is default)
        workspaceAccess: "none", // none | ro | rw
        workspaceRoot: "~/.openclaw/sandboxes",
        docker: {
          image: "openclaw-sandbox:bookworm-slim",
          workdir: "/workspace",
          readOnlyRoot: true,
          tmpfs: ["/tmp", "/var/tmp", "/run"],
          network: "none",
          user: "1000:1000",
          capDrop: ["ALL"],
          env: { LANG: "C.UTF-8" },
          setupCommand: "apt-get update && apt-get install -y git curl jq",
          pidsLimit: 256,
          memory: "1g",
          memorySwap: "2g",
          cpus: 1,
          ulimits: {
            nofile: { soft: 1024, hard: 2048 },
            nproc: 256
          },
          seccompProfile: "/path/to/seccomp.json",
          apparmorProfile: "openclaw-sandbox",
          dns: ["1.1.1.1", "8.8.8.8"],
          extraHosts: ["internal.service:10.0.0.5"]
        },
        prune: {
          idleHours: 24, // 0 disables idle pruning
          maxAgeDays: 7  // 0 disables max-age pruning
        }
      }
    }
  },
  tools: {
    sandbox: {
      tools: {
        allow: ["exec", "process", "read", "write", "edit", "sessions_list", "sessions_history", "sessions_send", "sessions_spawn", "session_status"],
        deny: ["browser", "canvas", "nodes", "cron", "discord", "gateway"]
      }
    }
  }
}
```

ハードニング用の設定項目は `agents.defaults.sandbox.docker` 配下に定義します：
`network`, `user`, `pidsLimit`, `memory`, `memorySwap`, `cpus`, `ulimits`,
`seccompProfile`, `apparmorProfile`, `dns`, `extraHosts`。

マルチエージェント構成では、`agents.list[].sandbox.{docker,browser,prune}.*` を使って、エージェントごとに `agents.defaults.sandbox.{docker,browser,prune}.*` を上書きします
（`agents.defaults.sandbox.scope` / `agents.list[].sandbox.scope` が `"shared"` の場合は無視されます）。

<div id="build-the-default-sandbox-image">
  ### デフォルトのサンドボックス用イメージをビルドする
</div>

```bash
scripts/sandbox-setup.sh
```

これにより、`Dockerfile.sandbox` を使用して `openclaw-sandbox:bookworm-slim` イメージをビルドします。

<div id="sandbox-common-image-optional">
  ### サンドボックス共通イメージ（オプション）
</div>

Node、Go、Rust などの一般的なビルドツールを含むサンドボックスイメージが必要な場合は、共通イメージをビルドしてください。

```bash
scripts/sandbox-common-setup.sh
```

これは `openclaw-sandbox-common:bookworm-slim` をビルドします。これを使うには、次のようにします。

```json5
{
  agents: { defaults: { sandbox: { docker: { image: "openclaw-sandbox-common:bookworm-slim" } } } }
}
```

<div id="sandbox-browser-image">
  ### サンドボックス用ブラウザイメージ
</div>

サンドボックス内で browser ツールを実行するには、browser イメージをビルドしてください。

```bash
scripts/sandbox-browser-setup.sh
```

これは `Dockerfile.sandbox-browser` を使って `openclaw-sandbox-browser:bookworm-slim` イメージをビルドします。コンテナは CDP を有効化した Chromium と、任意で noVNC オブザーバー（Xvfb を用いたヘッドフル）を実行します。

Notes:

* ヘッドフル（Xvfb）の方が、ヘッドレスよりもボットとしてのブロック・検知を受けにくくなります。
* `agents.defaults.sandbox.browser.headless=true` を指定すれば、ヘッドレスも引き続き利用できます。
* フルデスクトップ環境（GNOME）は不要で、Xvfb がディスプレイを提供します。

次の設定を使用します:

```json5
{
  agents: {
    defaults: {
      sandbox: {
        browser: { enabled: true }
      }
    }
  }
}
```

カスタムブラウザイメージ：

```json5
{
  agents: {
    defaults: {
      sandbox: { browser: { image: "my-openclaw-browser" } }
    }
  }
}
```

有効化すると、エージェントには次が渡されます:

* サンドボックス内ブラウザ制御用 URL（`browser` ツール用）
* noVNC の URL（有効化されていて、かつ headless=false の場合）

注意：ツールに許可リストを使用している場合は、`browser` を追加し、さらに `deny` から削除しないと、このツールは引き続きブロックされたままになります。
Prune ルール（`agents.defaults.sandbox.prune`）はブラウザ用コンテナにも適用されます。

<div id="custom-sandbox-image">
  ### カスタムサンドボックスイメージ
</div>

独自のイメージをビルドし、設定でそのイメージを指定します:

```bash
docker build -t my-openclaw-sbx -f Dockerfile.sandbox .
```

```json5
{
  agents: {
    defaults: {
      sandbox: { docker: { image: "my-openclaw-sbx" } }
    }
  }
}
```

<div id="tool-policy-allowdeny">
  ### ツールポリシー（allow/deny）
</div>

* `deny` が `allow` よりも優先されます。
* `allow` が空の場合：`deny` に含まれるもの以外のすべてのツールが利用可能です。
* `allow` が空でない場合：`allow` に含まれるツールのみが利用可能です（ただし `deny` に含まれるものを除きます）。

<div id="pruning-strategy">
  ### 削除ポリシー（Pruning strategy）
</div>

調整可能なパラメータは次の 2 つです:

* `prune.idleHours`: X 時間使用されていないコンテナを削除します（0 = 無効）
* `prune.maxAgeDays`: X 日より古いコンテナを削除します（0 = 無効）

例:

* 忙しいセッションは維持しつつ、寿命に上限を設ける:
  `idleHours: 24`, `maxAgeDays: 7`
* 一切削除しない:
  `idleHours: 0`, `maxAgeDays: 0`

<div id="security-notes">
  ### セキュリティに関する注意事項
</div>

* Hard wall は **tools**（exec/read/write/edit/apply&#95;patch）にのみ適用されます。
* browser / camera / canvas のようなホスト専用ツールは、デフォルトでブロックされます。
* サンドボックスで `browser` を許可すると、**分離が破られます**（browser はホスト上で動作します）。

<div id="troubleshooting">
  ## トラブルシューティング
</div>

* イメージが見つからない場合: [`scripts/sandbox-setup.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/sandbox-setup.sh) でビルドするか、`agents.defaults.sandbox.docker.image` を設定します。
* コンテナが起動していない場合: 必要に応じてセッションごとにオンデマンドで自動作成されます。
* サンドボックス内でパーミッションエラーが発生する場合: `docker.user` を、マウントしているワークスペースの所有権と一致する UID:GID に設定します（またはワークスペースフォルダを chown します）。
* カスタムツールが見つからない場合: OpenClaw はコマンドを `sh -lc`（ログインシェル）で実行し、`/etc/profile` を読み込むため、PATH がリセットされる場合があります。`docker.env.PATH` を設定してカスタムツールのパス（例: `/custom/bin:/usr/local/share/npm-global/bin`）を先頭に追加するか、Dockerfile 内で `/etc/profile.d/` 配下にスクリプトを追加します。