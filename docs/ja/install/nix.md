---
title: Nix
summary: "Nix を使って宣言的に OpenClaw をインストールする"
read_when:
  - 再現可能でロールバック可能なインストールを行いたい場合
  - すでに Nix/NixOS/Home Manager を使っている場合
  - すべてのバージョンを固定し、宣言的に管理したい場合
---

<div id="nix-installation">
  # Nix によるインストール
</div>

Nix で OpenClaw を実行する推奨方法は、必要なものが一通り揃った Home Manager モジュールである **[nix-openclaw](https://github.com/openclaw/nix-openclaw)** を利用することです。

<div id="quick-start">
  ## クイックスタート
</div>

これを AI エージェント（Claude や Cursor など）に貼り付けてください：

```text
I want to set up nix-openclaw on my Mac.
Repository: github:openclaw/nix-openclaw

What I need you to do:
1. Check if Determinate Nix is installed (if not, install it)
2. Create a local flake at ~/code/openclaw-local using templates/agent-first/flake.nix
3. Help me create a Telegram bot (@BotFather) and get my chat ID (@userinfobot)
4. Set up secrets (bot token, Anthropic key) - plain files at ~/.secrets/ is fine
5. Fill in the template placeholders and run home-manager switch
6. Verify: launchd running, bot responds to messages

Reference the nix-openclaw README for module options.
```

> **📦 完全なガイド: [github.com/openclaw/nix-openclaw](https://github.com/openclaw/nix-openclaw)**
>
> nix-openclaw リポジトリが、Nix のインストール方法に関する唯一の信頼できる情報源です。このページはその簡単な概要にすぎません。

<div id="what-you-get">
  ## 得られるもの
</div>

* Gateway + macOSアプリ + ツール（Whisper、Spotify、カメラ） — すべてのバージョンを固定
* 再起動後も自動起動する launchd サービス
* 宣言的な設定によるプラグインシステム
* 即時ロールバック：`home-manager switch --rollback`

***

<div id="nix-mode-runtime-behavior">
  ## Nix モードの実行時動作
</div>

`OPENCLAW_NIX_MODE=1` が設定されている場合（nix-openclaw では自動で設定されます）:

OpenClaw は、設定を決定的なものにし、自動インストールフローを無効にする **Nix モード** をサポートしています。
これを有効にするには、次を export してください:

```bash
OPENCLAW_NIX_MODE=1
```

macOS では、GUI アプリはシェル環境変数を自動的には継承されません。
`defaults` コマンド経由で Nix モードを有効にすることもできます：

```bash
defaults write bot.molt.mac openclaw.nixMode -bool true
```

<div id="config-state-paths">
  ### 設定と状態パス
</div>

OpenClaw は `OPENCLAW_CONFIG_PATH` から JSON5 設定を読み込み、`OPENCLAW_STATE_DIR` に可変データを保存します。

* `OPENCLAW_STATE_DIR` （デフォルト: `~/.openclaw`）
* `OPENCLAW_CONFIG_PATH` （デフォルト: `$OPENCLAW_STATE_DIR/openclaw.json`）

Nix 環境下で実行する場合は、実行時の状態と設定が Nix のイミュータブルストアの外側に保持されるよう、これらを Nix 管理のディレクトリに明示的に設定してください。

<div id="runtime-behavior-in-nix-mode">
  ### Nixモードにおけるランタイム時の挙動
</div>

* 自動インストールおよび自己更新フローは無効になります
* 不足している依存関係に対しては、Nix 固有の修復メッセージが表示されます
* UI は、該当する場合に読み取り専用の Nix モードバナーを表示します

<div id="packaging-note-macos">
  ## パッケージングに関する注意事項（macOS）
</div>

macOS のパッケージングフローでは、次の場所に安定した Info.plist テンプレートが存在することを前提としています。

```
apps/macos/Sources/OpenClaw/Resources/Info.plist
```

[`scripts/package-mac-app.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/package-mac-app.sh) はこのテンプレートをアプリバンドル内にコピーし、動的フィールド
（bundle ID、version/build、Git SHA、Sparkle keys）にパッチを当てます。これにより、SwiftPM
パッケージングおよび Nix ビルド（フル Xcode ツールチェーンに依存しない）に対して plist の内容を決定的なものとして保てます。

<div id="related">
  ## 関連情報
</div>

* [nix-openclaw](https://github.com/openclaw/nix-openclaw) — 完全なセットアップガイド
* [Wizard](/ja/start/wizard) — Nix を使わない CLI セットアップ
* [Docker](/ja/install/docker) — コンテナ環境でのセットアップ