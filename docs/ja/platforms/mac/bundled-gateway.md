---
title: バンドル版 Gateway
summary: "macOS 上の Gateway ランタイム（外部 launchd サービス）"
read_when:
  - OpenClaw.app をパッケージングする場合
  - macOS Gateway の launchd サービスをデバッグする場合
  - macOS 用の Gateway CLI をインストールする場合
---

<div id="gateway-on-macos-external-launchd">
  # macOS 上の Gateway（外部 launchd）
</div>

OpenClaw.app はもはや Node/Bun や Gateway ランタイムを同梱しません。macOS アプリは
**外部** の `openclaw` CLI のインストールを前提としており、Gateway を子プロセスとして
起動せず、Gateway を動作させ続けるためのユーザーごとの launchd サービスを管理します
（すでにローカルで Gateway が動作している場合は、その Gateway に接続します）。

<div id="install-the-cli-required-for-local-mode">
  ## CLI をインストールする（ローカルモードでは必須）
</div>

Mac に Node.js 22 以降をインストールし、そのうえで `openclaw` をグローバルにインストールします。

```bash
npm install -g openclaw@<version>
```

macOS アプリの **Install CLI** ボタンは、npm/pnpm を使って同じフローを実行します（Gateway ランタイムの実行には bun は推奨されていません）。


<div id="launchd-gateway-as-launchagent">
  ## Launchd (LaunchAgent としての Gateway)
</div>

ラベル:

- `bot.molt.gateway`（または `bot.molt.<profile>`。レガシーな `com.openclaw.*` が残っている場合があります）

Plist の場所（ユーザーごと）:

- `~/Library/LaunchAgents/bot.molt.gateway.plist`
  （または `~/Library/LaunchAgents/bot.molt.<profile>.plist`）

管理:

- macOS アプリが、Local モードでの LaunchAgent のインストール／更新を管理します。
- CLI からもインストールできます: `openclaw gateway install`

動作:

- 「OpenClaw Active」が LaunchAgent の有効化／無効化を制御します。
- アプリを終了しても Gateway は **停止しません**（launchd が生かし続けます）。
- 設定されたポートで既に Gateway が動作している場合、アプリは新規起動せず、
  その Gateway に接続します。

ログ:

- launchd の stdout/err: `/tmp/openclaw/openclaw-gateway.log`

<div id="version-compatibility">
  ## バージョン互換性
</div>

macOS アプリは、自身のバージョンと Gateway のバージョンを照合します。一致しない場合は、グローバル CLI をアプリのバージョンに合わせて更新してください。

<div id="smoke-check">
  ## 簡易動作確認
</div>

```bash
openclaw --version

OPENCLAW_SKIP_CHANNELS=1 \
OPENCLAW_SKIP_CANVAS_HOST=1 \
openclaw gateway --port 18999 --bind loopback
```

次に：

```bash
openclaw gateway call health --url ws://127.0.0.1:18999 --timeout 3000
```
