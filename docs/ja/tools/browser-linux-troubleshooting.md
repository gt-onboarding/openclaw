---
title: ブラウザ Linux トラブルシューティング
summary: "Linux 上で OpenClaw のブラウザ制御を行う際の Chrome/Brave/Edge/Chromium における CDP 起動問題を解決する"
read_when: "Linux でのブラウザ制御が失敗するとき、特に snap 版 Chromium の場合に読む"
---

<div id="browser-troubleshooting-linux">
  # ブラウザのトラブルシューティング（Linux）
</div>

<div id="problem-failed-to-start-chrome-cdp-on-port-18800">
  ## 問題: &quot;Failed to start Chrome CDP on port 18800&quot;
</div>

OpenClawのブラウザ制御サーバーが Chrome/Brave/Edge/Chromium の起動に失敗し、次のエラーが表示されます:

```
{"error":"Error: Failed to start Chrome CDP on port 18800 for profile \"openclaw\"."}
```


<div id="root-cause">
  ### 根本原因
</div>

Ubuntu（および多くの Linux ディストリビューション）では、Chromium のデフォルトのインストール方法は **snap パッケージ**です。Snap の AppArmor によるサンドボックス（制限）が、OpenClaw がブラウザープロセスを起動・監視する仕組みに干渉します。

`apt install chromium` コマンドは、snap 版のインストールへリダイレクトするスタブ（ダミー）パッケージをインストールします。

```
Note, selecting 'chromium-browser' instead of 'chromium'
chromium-browser is already the newest version (2:1snap1-0ubuntu2).
```

これは本物のブラウザではなく、単なるラッパーに過ぎません。


<div id="solution-1-install-google-chrome-recommended">
  ### 解決策 1: Google Chrome をインストールする（推奨）
</div>

snap のサンドボックスではない公式の Google Chrome の `.deb` パッケージをインストールします:

```bash
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
sudo dpkg -i google-chrome-stable_current_amd64.deb
sudo apt --fix-broken install -y  # 依存関係のエラーがある場合
```

次に OpenClaw の設定ファイル（`~/.openclaw/openclaw.json`）を更新してください。

```json
{
  "browser": {
    "enabled": true,
    "executablePath": "/usr/bin/google-chrome-stable",
    "headless": true,
    "noSandbox": true
  }
}
```


<div id="solution-2-use-snap-chromium-with-attach-only-mode">
  ### 解決策 2: Snap 版 Chromium を Attach-Only モードで使う
</div>

どうしても snap 版 Chromium を使う必要がある場合は、OpenClaw を手動で起動したブラウザにアタッチするように設定します:

1. 設定を更新します:

```json
{
  "browser": {
    "enabled": true,
    "attachOnly": true,
    "headless": true,
    "noSandbox": true
  }
}
```

2. Chromium を手動で起動する:

```bash
chromium-browser --headless --no-sandbox --disable-gpu \
  --remote-debugging-port=18800 \
  --user-data-dir=$HOME/.openclaw/browser/openclaw/user-data \
  about:blank &
```

3. 必要に応じて、Chrome を自動起動させる systemd ユーザーサービスを作成します：

```ini
# ~/.config/systemd/user/openclaw-browser.service
[Unit]
Description=OpenClaw ブラウザ (Chrome CDP)
After=network.target

[Service]
ExecStart=/snap/bin/chromium --headless --no-sandbox --disable-gpu --remote-debugging-port=18800 --user-data-dir=%h/.openclaw/browser/openclaw/user-data about:blank
Restart=on-failure
RestartSec=5

[Install]
WantedBy=default.target
```

次のコマンドで有効化します: `systemctl --user enable --now openclaw-browser.service`


<div id="verifying-the-browser-works">
  ### ブラウザが正常に動作しているか確認する
</div>

ステータスを確認する：

```bash
curl -s http://127.0.0.1:18791/ | jq '{running, pid, chosenBrowser}'
```

ブラウジングのテスト:

```bash
curl -s -X POST http://127.0.0.1:18791/start
curl -s http://127.0.0.1:18791/tabs
```


<div id="config-reference">
  ### 設定リファレンス
</div>

| オプション | 説明 | デフォルト |
|--------|-------------|---------|
| `browser.enabled` | ブラウザ制御を有効にする | `true` |
| `browser.executablePath` | Chromium ベースのブラウザバイナリ（Chrome/Brave/Edge/Chromium）へのパス | 自動検出（Chromium ベースの場合はデフォルトブラウザを優先） |
| `browser.headless` | GUI なしで実行する | `false` |
| `browser.noSandbox` | `--no-sandbox` フラグを追加（一部の Linux 環境で必要） | `false` |
| `browser.attachOnly` | ブラウザを起動せず、既存のブラウザにのみ接続する | `false` |
| `browser.cdpPort` | Chrome DevTools Protocol のポート番号 | `18800` |

<div id="problem-chrome-extension-relay-is-running-but-no-tab-is-connected">
  ### Problem: "Chrome extension relay is running, but no tab is connected"
</div>

あなたは `chrome` プロファイル（拡張機能リレー）を使用しています。このプロファイルは、OpenClaw
ブラウザー拡張機能が開いているタブに接続されていることを前提としています。

対処方法:

1. **管理対象ブラウザーを使う:** `openclaw browser start --browser-profile openclaw`
   （または `browser.defaultProfile: "openclaw"` を設定する）。
2. **拡張リレーを使う:** 拡張機能をインストールし、タブを開いてから OpenClaw
   拡張機能アイコンをクリックし、そのタブに接続する。

補足:

- `chrome` プロファイルは、可能であれば **システムのデフォルトの Chromium ブラウザー** を使用します。
- ローカルの `openclaw` プロファイルでは `cdpPort` / `cdpUrl` は自動割り当てされます。これらを手動で設定するのは、リモート CDP を利用する場合に限ってください。