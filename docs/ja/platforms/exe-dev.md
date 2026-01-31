---
title: Exe Dev
summary: "OpenClaw Gateway を exe.dev（VM + HTTPS プロキシ）上で実行してリモートアクセスを可能にする"
read_when:
  - Gateway 用に安価な常時稼働 Linux ホストがほしい場合
  - 自前で VPS を用意せずにリモートで Control UI にアクセスしたい場合
---

<div id="exedev">
  # exe.dev
</div>

目標: exe.dev の VM 上で OpenClaw Gateway を稼働させ、ローカルマシンから `https://<vm-name>.exe.xyz` でアクセスできるようにすること。

このページは、exe.dev のデフォルトイメージである **exeuntu** を前提としている。別のディストリビューションを選んだ場合は、パッケージをそれに応じて読み替えること。

<div id="beginner-quick-path">
  ## 初心者向けクイックスタート
</div>

1. [https://exe.new/openclaw](https://exe.new/openclaw)
2. 必要に応じて認証キー／トークンを入力する
3. VM の横の「Agent」をクリックして待つ……
4. ???
5. 利益が出る

<div id="what-you-need">
  ## 必要なもの
</div>

* exe.dev アカウント
* [exe.dev](https://exe.dev) 仮想マシンへの `ssh exe.dev` アクセス権（任意）

<div id="automated-install-with-shelley">
  ## Shelley を使った自動インストール
</div>

[exe.dev](https://exe.dev) のエージェントである Shelley は、用意されたプロンプトを使って
OpenClaw を即座にインストールできます。使用するプロンプトは次のとおりです。

```
Set up OpenClaw (https://docs.openclaw.ai/install) on this VM. Use the non-interactive and accept-risk flags for openclaw onboarding. Add the supplied auth or token as needed. Configure nginx to forward from the default port 18789 to the root location on the default enabled site config, making sure to enable Websocket support. Pairing is done by "openclaw devices list" and "openclaw device approve <request id>". Make sure the dashboard shows that OpenClaw's health is OK. exe.dev handles forwarding from port 8000 to port 80/443 and HTTPS for us, so the final "reachable" should be <vm-name>.exe.xyz, without port specification.
```

<div id="manual-installation">
  ## 手動インストール
</div>

<div id="1-create-the-vm">
  ## 1) VM を作成する
</div>

ローカル端末から:

```bash
ssh exe.dev new 
```

次に接続します:

```bash
ssh <vm-name>.exe.xyz
```

ヒント: この VM は **ステートフル** に維持してください。OpenClaw は `~/.openclaw/` と `~/.openclaw/workspace/` 配下に状態を保存します。

<div id="2-install-prerequisites-on-the-vm">
  ## 2) 前提条件をインストールする（VM 上）
</div>

```bash
sudo apt-get update
sudo apt-get install -y git curl jq ca-certificates openssl
```

<div id="3-install-openclaw">
  ## 3) OpenClaw をインストールする
</div>

OpenClaw のインストールスクリプトを実行してください:

```bash
curl -fsSL https://openclaw.bot/install.sh | bash
```

<div id="4-setup-nginx-to-proxy-openclaw-to-port-8000">
  ## 4) nginx を設定して OpenClaw をポート 8000 へリバースプロキシする
</div>

`/etc/nginx/sites-enabled/default` を次のように編集します

```
server {
    listen 80 default_server;
    listen [::]:80 default_server;
    listen 8000;
    listen [::]:8000;

    server_name _;

    location / {
        proxy_pass http://127.0.0.1:18789;
        proxy_http_version 1.1;

        # WebSocketサポート
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";

        # 標準プロキシヘッダー
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # 長時間接続用タイムアウト設定
        proxy_read_timeout 86400s;
        proxy_send_timeout 86400s;
    }
}
```

<div id="5-access-openclaw-and-grant-privileges">
  ## 5) OpenClaw にアクセスして権限を付与する
</div>

`https://<vm-name>.exe.xyz/?token=YOUR-TOKEN-FROM-TERMINAL` にアクセスします（`YOUR-TOKEN-FROM-TERMINAL` にはターミナルに表示されたトークンを指定します）。`openclaw devices list` と `openclaw device approve` を使ってデバイスを承認します。迷ったときは、ブラウザから Shelley を使いましょう。

<div id="remote-access">
  ## リモートアクセス
</div>

リモートアクセスは [exe.dev](https://exe.dev) の認証機能によって管理されます。デフォルトでは、ポート 8000 で受信した HTTP トラフィックは、メール認証付きで `https://<vm-name>.exe.xyz` に転送されます。

<div id="updating">
  ## アップデート
</div>

```bash
npm i -g openclaw@latest
openclaw doctor
openclaw gateway restart
openclaw health
```

ガイド: [更新](/ja/install/updating)
