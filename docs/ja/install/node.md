---
title: "Node.js + npm（PATH の健全性確認）"
summary: "Node.js + npm インストールの健全性確認：バージョン、PATH、グローバルインストール"
read_when:
  - "OpenClaw をインストールしたのに、`openclaw` が「command not found」と表示される"
  - "新しいマシンで Node.js/npm をセットアップしている"
  - "npm install -g ... が権限または PATH 設定の問題で失敗する"
---

<div id="nodejs-npm-path-sanity">
  # Node.js + npm（PATH の確認）
</div>

OpenClaw のランタイムの前提条件は **Node 22 以上** です。

`npm install -g openclaw@latest` は実行できるのに、その後で `openclaw: command not found` が表示される場合、原因はほぼ必ず **PATH** の問題です。npm がグローバルのバイナリを配置しているディレクトリが、シェルの PATH に含まれていません。

<div id="quick-diagnosis">
  ## 簡易診断
</div>

次を実行してください：

```bash
node -v
npm -v
npm prefix -g
echo "$PATH"
```

`echo "$PATH"` の出力に `$(npm prefix -g)/bin`（macOS/Linux）または `$(npm prefix -g)`（Windows）が含まれて **いない** 場合、シェルはグローバル npm バイナリ（`openclaw` を含む）を検出できません。


<div id="fix-put-npms-global-bin-dir-on-path">
  ## 修正: npm のグローバル bin ディレクトリを PATH に通す
</div>

1. npm のグローバル prefix を確認します:

```bash
npm prefix -g
```

2. グローバル npm bin ディレクトリをシェルのスタートアップファイルに追加します：

* zsh: `~/.zshrc`
* bash: `~/.bashrc`

例（パスは `npm prefix -g` の出力結果に置き換えてください）：

```bash
# macOS / Linux
export PATH="/path/from/npm/prefix/bin:$PATH"
```

その後、**新しいターミナル** を開きます（あるいは zsh では `rehash`、bash では `hash -r` を実行します）。

Windows の場合は、`npm prefix -g` の出力結果を PATH 環境変数に追加します。


<div id="fix-avoid-sudo-npm-install-g-permission-errors-linux">
  ## 対処方法：`sudo npm install -g` の使用やパーミッションエラー（Linux）を回避する
</div>

`npm install -g ...` が `EACCES` で失敗する場合、npm のグローバル prefix をユーザーが書き込み可能なディレクトリに変更してください：

```bash
mkdir -p "$HOME/.npm-global"
npm config set prefix "$HOME/.npm-global"
export PATH="$HOME/.npm-global/bin:$PATH"
```

シェルの起動ファイルに `export PATH=...` の行を追記して保存してください。


<div id="recommended-node-install-options">
  ## 推奨される Node のインストール方法
</div>

Node/npm は、次の条件を満たす形でインストールしておくと、想定外のトラブルが最も少なくなります。

- Node を最新版に保ちやすい（22 以上）
- グローバルな npm bin ディレクトリが安定しており、新しいシェルでも PATH に含まれる

一般的な選択肢:

- macOS: Homebrew（`brew install node`）またはバージョンマネージャー
- Linux: 好みのバージョンマネージャー、または Node 22 以上を提供するディストリの公式インストール手段
- Windows: 公式の Node インストーラ、`winget`、または Windows 用の Node バージョンマネージャー

バージョンマネージャー（nvm/fnm/asdf など）を使う場合は、日常的に使うシェル（zsh なのか bash なのか）で確実に初期化されるように設定し、そのマネージャーが設定する PATH がインストーラ実行時にも有効になっていることを確認してください。