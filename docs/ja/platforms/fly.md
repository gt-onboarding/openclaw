---
title: Fly.io
description: OpenClaw を Fly.io にデプロイする
---

<div id="flyio-deployment">
  # Fly.io へのデプロイ
</div>

**目的:** 永続ストレージ、自動 HTTPS、Discord などのチャネルへのアクセスを備えた OpenClaw Gateway を [Fly.io](https://fly.io) 上のマシンで稼働させること。

<div id="what-you-need">
  ## 必要なもの
</div>

- [flyctl CLI](https://fly.io/docs/hands-on/install-flyctl/) がインストールされていること
- Fly.io アカウント（無料プランで可）
- モデル用認証情報: Anthropic API キー（またはその他のプロバイダーキー）
- チャネル用認証情報: Discord ボットトークン、Telegram トークンなど

<div id="beginner-quick-path">
  ## 初心者向けクイックスタート
</div>

1. リポジトリをクローン → `fly.toml` をカスタマイズ
2. アプリとボリュームを作成 → シークレットを設定
3. `fly deploy` でデプロイ
4. SSH で接続して設定を作成するか、Control UI を使用

<div id="1-create-the-fly-app">
  ## 1) Fly アプリの作成
</div>

```bash
# Clone the repo
git clone https://github.com/openclaw/openclaw.git
cd openclaw

# Create a new Fly app (pick your own name)
fly apps create my-openclaw

# 永続ボリュームを作成(通常1GBで十分)
fly volumes create openclaw_data --size 1 --region iad
```

**ヒント:** 自分の拠点に近いリージョンを選択してください。よく使われるオプション: `lhr`（ロンドン）、`iad`（バージニア）、`sjc`（サンノゼ）。


<div id="2-configure-flytoml">
  ## 2) fly.toml を設定する
</div>

`fly.toml` を編集し、アプリ名や要件に合わせてください。

**セキュリティに関する注意:** デフォルトの設定では公開 URL がインターネットに公開されます。パブリック IP を持たない堅牢なデプロイが必要な場合は、[Private Deployment](#private-deployment-hardened) を参照するか、`fly.private.toml` を使用してください。

```toml
app = "my-openclaw"  # アプリ名
primary_region = "iad"

[build]
  dockerfile = "Dockerfile"

[env]
  NODE_ENV = "production"
  OPENCLAW_PREFER_PNPM = "1"
  OPENCLAW_STATE_DIR = "/data"
  NODE_OPTIONS = "--max-old-space-size=1536"

[processes]
  app = "node dist/index.js gateway --allow-unconfigured --port 3000 --bind lan"

[http_service]
  internal_port = 3000
  force_https = true
  auto_stop_machines = false
  auto_start_machines = true
  min_machines_running = 1
  processes = ["app"]

[[vm]]
  size = "shared-cpu-2x"
  memory = "2048mb"

[mounts]
  source = "openclaw_data"
  destination = "/data"
```

**主要な設定:**

| Setting                        | 理由                                                                     |
| ------------------------------ | ---------------------------------------------------------------------- |
| `--bind lan`                   | `0.0.0.0` にバインドして、Fly のプロキシから Gateway に到達できるようにするため                    |
| `--allow-unconfigured`         | 設定ファイルなしで起動するため（このあと作成するため）                                            |
| `internal_port = 3000`         | Fly のヘルスチェック用に `--port 3000`（または `OPENCLAW_GATEWAY_PORT`）と一致させる必要があるため |
| `memory = "2048mb"`            | 512MB では小さすぎるため。2GB を推奨                                                |
| `OPENCLAW_STATE_DIR = "/data"` | ボリューム上に状態を永続化するため                                                      |


<div id="3-set-secrets">
  ## 3) シークレットを設定
</div>

```bash
# 必須: Gatewayトークン(非ループバックバインディング用)
fly secrets set OPENCLAW_GATEWAY_TOKEN=$(openssl rand -hex 32)

# Model provider API keys
fly secrets set ANTHROPIC_API_KEY=sk-ant-...

# Optional: Other providers
fly secrets set OPENAI_API_KEY=sk-...
fly secrets set GOOGLE_API_KEY=...

# Channel tokens
fly secrets set DISCORD_BOT_TOKEN=MTQ...
```

**注意:**

* ループバック以外のアドレスへのバインド（`--bind lan`）では、セキュリティのために `OPENCLAW_GATEWAY_TOKEN` が必要です。
* これらのトークンはパスワードと同様に扱ってください。
* すべての API キーおよびトークンについて、**config ファイルではなく環境変数を優先して使用**してください。これにより、`openclaw.json` に機密情報を含めずに済み、誤って公開されたりログに記録されたりするリスクを減らせます。


<div id="4-deploy">
  ## 4) デプロイする
</div>

```bash
fly deploy
```

最初のデプロイでは Docker イメージのビルドが行われるため（約 2〜3 分かかります）、2 回目以降のデプロイはより高速になります。

デプロイ後、次の点を確認してください:

```bash
fly status
fly logs
```

次のように表示されるはずです:

```
[gateway] listening on ws://0.0.0.0:3000 (PID xxx)
[discord] logged in to discord as xxx
```


<div id="5-create-config-file">
  ## 5) 設定ファイルを作成する
</div>

適切な設定ファイルを作成するために、SSH でマシンに接続します:

```bash
fly ssh console
```

設定用のディレクトリとファイルを作成します:

```bash
mkdir -p /data
cat > /data/openclaw.json << 'EOF'
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "anthropic/claude-opus-4-5",
        "fallbacks": ["anthropic/claude-sonnet-4-5", "openai/gpt-4o"]
      },
      "maxConcurrent": 4
    },
    "list": [
      {
        "id": "main",
        "default": true
      }
    ]
  },
  "auth": {
    "profiles": {
      "anthropic:default": { "mode": "token", "provider": "anthropic" },
      "openai:default": { "mode": "token", "provider": "openai" }
    }
  },
  "bindings": [
    {
      "agentId": "main",
      "match": { "channel": "discord" }
    }
  ],
  "channels": {
    "discord": {
      "enabled": true,
      "groupPolicy": "allowlist",
      "guilds": {
        "YOUR_GUILD_ID": {
          "channels": { "general": { "allow": true } },
          "requireMention": false
        }
      }
    }
  },
  "gateway": {
    "mode": "local",
    "bind": "auto"
  },
  "meta": {
    "lastTouchedVersion": "2026.1.29"
  }
}
EOF
```

**注意:** `OPENCLAW_STATE_DIR=/data` の場合、設定ファイルのパスは `/data/openclaw.json` になります。

**注意:** Discord トークンは次のいずれかから取得できます:

* 環境変数: `DISCORD_BOT_TOKEN`（シークレットにはこちらを推奨）
* 設定ファイル: `channels.discord.token`

環境変数を使う場合、設定ファイルにトークンを追加する必要はありません。Gateway は `DISCORD_BOT_TOKEN` を自動的に読み込みます。

設定を反映するには再起動してください:

```bash
exit
fly machine restart <machine-id>
```


<div id="6-access-the-gateway">
  ## 6) Gateway にアクセスする
</div>

<div id="control-ui">
  ### Control UI
</div>

ブラウザで開く：

```bash
fly open
```

または `https://my-openclaw.fly.dev/` にアクセスしてください。

認証のため、Gateway トークン（`OPENCLAW_GATEWAY_TOKEN` の値）を貼り付けてください。


<div id="logs">
  ### ログ
</div>

```bash
fly logs              # ライブログ
fly logs --no-tail    # 最近のログ
```


<div id="ssh-console">
  ### SSH コンソール
</div>

```bash
fly ssh console
```


<div id="troubleshooting">
  ## トラブルシューティング
</div>

<div id="app-is-not-listening-on-expected-address">
  ### "アプリが想定されたアドレスで待ち受けしていない"
</div>

Gateway が `0.0.0.0` ではなく `127.0.0.1` にバインドされています。

**対処方法:** `fly.toml` 内のプロセスコマンドに `--bind lan` を追加します。

<div id="health-checks-failing-connection-refused">
  ### ヘルスチェックの失敗 / 接続拒否
</div>

Fly が設定されたポートで Gateway に接続できていません。

**対処:** `internal_port` が Gateway のポートと一致していることを確認してください（`--port 3000` または `OPENCLAW_GATEWAY_PORT=3000` を設定します）。

<div id="oom-memory-issues">
  ### OOM / メモリ問題
</div>

コンテナが再起動を繰り返す、または頻繁に強制終了される場合。兆候としては、`SIGABRT`、`v8::internal::Runtime_AllocateInYoungGeneration`、あるいはログも出さずに勝手に再起動される、といったものがある。

**対処:** `fly.toml` でメモリを増やす:

```toml
[[vm]]
  memory = "2048mb"
```

または、既存のマシンを更新します:

```bash
fly machine update <machine-id> --vm-memory 2048 -y
```

**注意:** 512MB では小さすぎます。1GB でも動作する場合はありますが、高負荷時や詳細なログ出力が有効な場合にはメモリ不足（OOM）が発生することがあります。**2GB を推奨します。**


<div id="gateway-lock-issues">
  ### Gateway ロックの問題
</div>

Gateway が「すでに実行中」といったエラーを出して起動しない。

これは、コンテナが再起動されたにもかかわらず、PID ロックファイルがボリューム上に残り続けている場合に発生する。

**対処:** ロックファイルを削除する:

```bash
fly ssh console --command "rm -f /data/gateway.*.lock"
fly machine restart <machine-id>
```

ロックファイルは `/data/gateway.*.lock` にあります（サブディレクトリではなく、`/data` 直下です）。


<div id="config-not-being-read">
  ### 設定が読み込まれない
</div>

`--allow-unconfigured` を使用している場合、Gateway は最小限の設定ファイルを作成します。再起動時には、`/data/openclaw.json` にあるカスタム設定が読み込まれるはずです。

設定ファイルが存在することを確認してください:

```bash
fly ssh console --command "cat /data/openclaw.json"
```


<div id="writing-config-via-ssh">
  ### SSH 経由での設定ファイル書き込み
</div>

`fly ssh console -C` コマンドはシェルのリダイレクトをサポートしていません。設定ファイルを書き込むには、次のようにします:

```bash
# echo + tee を使用 (ローカルからリモートへパイプ)
echo '{"your":"config"}' | fly ssh console -C "tee /data/openclaw.json"

# Or use sftp
fly sftp shell
> put /local/path/config.json /data/openclaw.json
```

**注意:** ファイルがすでに存在する場合、`fly sftp` が失敗することがあります。まず削除してください：

```bash
fly ssh console --command "rm /data/openclaw.json"
```


<div id="state-not-persisting">
  ### 状態が永続化されない
</div>

再起動後に認証情報やセッションが失われる場合、state ディレクトリがコンテナのファイルシステム上に書き込まれています。

**対処法:** `OPENCLAW_STATE_DIR=/data` が `fly.toml` に設定されていることを確認し、再デプロイしてください。

<div id="updates">
  ## 更新
</div>

```bash
# 最新の変更をプル
git pull

# 再デプロイ
fly deploy

# ヘルスチェック
fly status
fly logs
```


<div id="updating-machine-command">
  ### Machine コマンドの更新
</div>

フル再デプロイを行わずに起動コマンドだけを変更したい場合は、次のようにします。

```bash
# マシンIDを取得
fly machines list

# コマンドを更新
fly machine update <machine-id> --command "node dist/index.js gateway --port 3000 --bind lan" -y

# メモリを増やす場合
fly machine update <machine-id> --vm-memory 2048 --command "node dist/index.js gateway --port 3000 --bind lan" -y
```

**注意:** `fly deploy` の後、machine コマンドは `fly.toml` の設定内容に戻ってしまう場合があります。手動で変更した箇所があれば、デプロイ後に再度適用してください。


<div id="private-deployment-hardened">
  ## プライベートデプロイ（ハードニング）
</div>

デフォルトでは、Fly はパブリック IP を割り当てるため、`https://your-app.fly.dev` から Gateway にアクセスできるようになります。これは便利ですが、同時にインターネットスキャナ（Shodan、Censys など）によってデプロイが検出されうることも意味します。

**パブリックに一切公開しない**堅牢化されたデプロイを行う場合は、プライベートテンプレートを使用してください。

<div id="when-to-use-private-deployment">
  ### プライベートデプロイを使うべきタイミング
</div>

- **アウトバウンド** な呼び出し／メッセージ送信のみを行う場合（受信側の Webhook は不要な場合）
- Webhook コールバックに **ngrok または Tailscale** のトンネルのみを使用する場合
- ブラウザではなく **SSH、プロキシ、または WireGuard** 経由で Gateway にアクセスする場合
- デプロイを **インターネット上のスキャナから見えないようにしたい** 場合

<div id="setup">
  ### セットアップ
</div>

標準の設定ファイルではなく、`fly.private.toml` を使用してください。

```bash
# プライベート設定でデプロイ
fly deploy -c fly.private.toml
```

または既存のデプロイを変換する：

```bash
# List current IPs
fly ips list -a my-openclaw

# Release public IPs
fly ips release <public-ipv4> -a my-openclaw
fly ips release <public-ipv6> -a my-openclaw

# プライベート設定に切り替えて、今後のデプロイでパブリックIPアドレスが再割り当てされないようにする
# ([http_service]を削除するか、プライベートテンプレートでデプロイする)
fly deploy -c fly.private.toml

# Allocate private-only IPv6
fly ips allocate-v6 --private -a my-openclaw
```

その後に `fly ips list` を実行すると、`private` タイプの IP だけが表示されるはずです：

```
VERSION  IP                   TYPE             REGION
v6       fdaa:x:x:x:x::x      private          global
```


<div id="accessing-a-private-deployment">
  ### プライベートデプロイへのアクセス
</div>

パブリック URL がないため、次のいずれかの方法でアクセスします：

**オプション 1：ローカルプロキシ（最も簡単）**

```bash
# ローカルポート 3000 をアプリに転送
fly proxy 3000:3000 -a my-openclaw

# その後、ブラウザで http://localhost:3000 を開く
```

**オプション 2: WireGuard VPN**

```bash
# WireGuard設定を作成（1回のみ）
fly wireguard create

# WireGuardクライアントにインポートし、内部IPv6経由でアクセス
# 例: http://[fdaa:x:x:x:x::x]:3000
```

**オプション3：SSH のみ**

```bash
fly ssh console -a my-openclaw
```


<div id="webhooks-with-private-deployment">
  ### プライベートデプロイメントでの webhook 利用
</div>

外部に公開せずに webhook のコールバック（Twilio、Telnyx など）が必要な場合:

1. **ngrok トンネル** - コンテナ内、またはサイドカーとして ngrok を実行する
2. **Tailscale Funnel** - Tailscale 経由で特定のパスのみを公開する
3. **アウトバウンド専用** - 一部のプロバイダー（Twilio）は、webhook なしでもアウトバウンド通話のみで問題なく動作する

ngrok を使った音声通話向け設定例:

```json
{
  "plugins": {
    "entries": {
      "voice-call": {
        "enabled": true,
        "config": {
          "provider": "twilio",
          "tunnel": { "provider": "ngrok" }
        }
      }
    }
  }
}
```

ngrok トンネルはコンテナ内で実行され、Fly アプリ本体をインターネット上に公開することなく、パブリックな webhook URL を提供します。


<div id="security-benefits">
  ### セキュリティ上の利点
</div>

| 項目 | パブリック | プライベート |
|--------|--------|---------|
| インターネットスキャナー | 発見可能 | 非公開 |
| 直接攻撃 | 受けやすい | ブロックされる |
| Control UI へのアクセス | ブラウザから直接 | プロキシ / VPN 経由 |
| Webhook 配信 | 直接 | トンネル経由 |

<div id="notes">
  ## 注意事項
</div>

- Fly.io は **x86 アーキテクチャ**（ARM ではない）を使用します
- この Dockerfile は x86/ARM の両方のアーキテクチャに対応しています
- WhatsApp/Telegram のオンボーディングには `fly ssh console` を使用してください
- 永続データは `/data` ボリューム上に保存されます
- Signal を利用するには Java と signal-cli が必要です。カスタムイメージを使用し、メモリは 2GB 以上を確保してください。

<div id="cost">
  ## コスト
</div>

推奨構成（`shared-cpu-2x`、2GB RAM）の場合：

- 利用状況に応じて月額約 $10〜15 程度
- 無料枠に一定の利用量が含まれる

詳細は [Fly.io の料金ページ](https://fly.io/docs/about/pricing/) を参照してください。