---
title: リモート Gateway 向け README
summary: "リモート Gateway に接続する OpenClaw.app 用 SSH トンネルの設定"
read_when: "macOS アプリから SSH 経由でリモート Gateway に接続するとき"
---

<div id="running-openclawapp-with-a-remote-gateway">
  # リモート Gateway と連携して OpenClaw.app を実行する
</div>

OpenClaw.app は SSH トンネル経由でリモート Gateway に接続します。このガイドでは、その設定方法を説明します。

<div id="overview">
  ## 概要
</div>

```
┌─────────────────────────────────────────────────────────────┐
│                        Client Machine                          │
│                                                              │
│  OpenClaw.app ──► ws://127.0.0.1:18789 (local port)           │
│                     │                                        │
│                     ▼                                        │
│  SSH Tunnel ────────────────────────────────────────────────│
│                     │                                        │
└─────────────────────┼──────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────┐
│                         Remote Machine                        │
│                                                              │
│  Gateway WebSocket ──► ws://127.0.0.1:18789 ──►              │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```


<div id="quick-setup">
  ## クイックセットアップ
</div>

<div id="step-1-add-ssh-config">
  ### ステップ 1: SSH 設定を追加する
</div>

`~/.ssh/config` を編集して、次の内容を追加します：

```ssh
Host remote-gateway
    HostName <REMOTE_IP>          # 例: 172.27.187.184
    User <REMOTE_USER>            # 例: jefferson
    LocalForward 18789 127.0.0.1:18789
    IdentityFile ~/.ssh/id_rsa
```

`<REMOTE_IP>` と `<REMOTE_USER>` を、ご自身の環境に合わせた値に置き換えてください。


<div id="step-2-copy-ssh-key">
  ### ステップ 2: SSH 鍵をコピーする
</div>

公開鍵をリモートマシンにコピーします（パスワードの入力は一度だけで済みます）:

```bash
ssh-copy-id -i ~/.ssh/id_rsa <REMOTE_USER>@<REMOTE_IP>
```


<div id="step-3-set-gateway-token">
  ### 手順 3: Gateway トークンを設定する
</div>

```bash
launchctl setenv OPENCLAW_GATEWAY_TOKEN "<your-token>"
```


<div id="step-4-start-ssh-tunnel">
  ### ステップ 4: SSH トンネルを開始
</div>

```bash
ssh -N remote-gateway &
```


<div id="step-5-restart-openclawapp">
  ### ステップ 5：OpenClaw.app を再起動する
</div>

```bash
# OpenClaw.app を終了し (⌘Q)、その後再度開く:
open /path/to/OpenClaw.app
```

これでアプリは SSH トンネル経由でリモート Gateway に接続します。

***


<div id="auto-start-tunnel-on-login">
  ## ログイン時にトンネルを自動起動する
</div>

ログイン時に SSH トンネルが自動で開始されるようにするには、Launch Agent を作成します。

<div id="create-the-plist-file">
  ### PLIST ファイルを作成する
</div>

これを `~/Library/LaunchAgents/bot.molt.ssh-tunnel.plist` として保存します。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>bot.molt.ssh-tunnel</string>
    <key>ProgramArguments</key>
    <array>
        <string>/usr/bin/ssh</string>
        <string>-N</string>
        <string>remote-gateway</string>
    </array>
    <key>KeepAlive</key>
    <true/>
    <key>RunAtLoad</key>
    <true/>
</dict>
</plist>
```


<div id="load-the-launch-agent">
  ### Launch Agent をロードする
</div>

```bash
launchctl bootstrap gui/$UID ~/Library/LaunchAgents/bot.molt.ssh-tunnel.plist
```

以降、トンネルは次のように動作します:

* ログイン時に自動的に起動する
* クラッシュした場合に再起動する
* バックグラウンドで動作し続ける

レガシー設定に関する注意: 残っている場合は、`com.openclaw.ssh-tunnel` LaunchAgent を削除してください。

***


<div id="troubleshooting">
  ## トラブルシューティング
</div>

**トンネルが稼働しているか確認する:**

```bash
ps aux | grep "ssh -N remote-gateway" | grep -v grep
lsof -i :18789
```

**トンネルを再起動する:**

```bash
launchctl kickstart -k gui/$UID/bot.molt.ssh-tunnel
```

**トンネルを停止する：**

```bash
launchctl bootout gui/$UID/bot.molt.ssh-tunnel
```

***


<div id="how-it-works">
  ## 動作の仕組み
</div>

| コンポーネント | 役割 |
|-----------|--------------|
| `LocalForward 18789 127.0.0.1:18789` | ローカルポート 18789 の通信をリモートポート 18789 に転送します |
| `ssh -N` | リモートコマンドを実行せずに SSH 接続のみを行います（ポートフォワーディング専用） |
| `KeepAlive` | トンネルがクラッシュした場合に自動的に再起動します |
| `RunAtLoad` | エージェント読み込み時にトンネルを起動します |

OpenClaw.app はクライアントマシン上の `ws://127.0.0.1:18789` に接続します。SSH トンネルがその接続を、Gateway が稼働しているリモートマシン上のポート 18789 に転送します。