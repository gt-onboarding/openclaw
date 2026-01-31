---
title: Linux
summary: "Linux サポートおよびコンパニオンアプリの状況"
read_when:
  - Linux コンパニオンアプリの状況を知りたいとき
  - プラットフォームの対応範囲やコントリビューションを計画しているとき
---

<div id="linux-app">
  # Linux App
</div>

Gateway は Linux 上で完全にサポートされています。**Node.js を推奨ランタイムとしています**。
Bun は Gateway では推奨されません（WhatsApp/Telegram 周りで不具合があります）。

ネイティブ Linux 向けのコンパニオンアプリは今後の対応を予定しています。開発に協力したい場合は、ぜひコントリビュートしてください。

<div id="beginner-quick-path-vps">
  ## 初心者向けかんたんセットアップ（VPS）
</div>

1. Node 22 以上をインストールします
2. `npm i -g openclaw@latest` を実行します
3. `openclaw onboard --install-daemon` を実行します
4. 手元のラップトップから次を実行します: `ssh -N -L 18789:127.0.0.1:18789 <user>@<host>`
5. `http://127.0.0.1:18789/` を開いて、トークンを貼り付けます

VPS 向けステップバイステップガイド: [exe.dev](/ja/platforms/exe-dev)

<div id="install">
  ## インストール
</div>

* [はじめに](/ja/start/getting-started)
* [インストールと更新](/ja/install/updating)
* オプションの手順: [Bun（実験的）](/ja/install/bun), [Nix](/ja/install/nix), [Docker](/ja/install/docker)

<div id="gateway">
  ## Gateway
</div>

* [Gateway 運用ランブック](/ja/gateway)
* [設定](/ja/gateway/configuration)

<div id="gateway-service-install-cli">
  ## Gateway サービスのインストール (CLI)
</div>

次のいずれかを使用してください:

```
openclaw onboard --install-daemon
```

または：

```
openclaw gateway install
```

または：

```
openclaw configure
```

指示されたら **Gateway service** を選択してください。

修復/移行:

```
openclaw doctor
```

<div id="system-control-systemd-user-unit">
  ## システム制御（systemd ユーザー ユニット）
</div>

OpenClaw はデフォルトで systemd の **ユーザー** サービスをインストールします。共有サーバーや常時稼働サーバーでは、**システム** サービスを使用してください。完全な unit の例と詳しいガイドは [Gateway ランブック](/ja/gateway) にあります。

最小構成：

`~/.config/systemd/user/openclaw-gateway[-<profile>].service` を作成します：

```
[Unit]
Description=OpenClaw Gateway (profile: <profile>, v<version>)
After=network-online.target
Wants=network-online.target

[Service]
ExecStart=/usr/local/bin/openclaw gateway --port 18789
Restart=always
RestartSec=5

[Install]
WantedBy=default.target
```

有効化する:

```
systemctl --user enable --now openclaw-gateway[-<profile>].service
```
