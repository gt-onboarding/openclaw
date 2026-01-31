---
title: インストール
summary: "OpenClaw をインストールする（推奨インストーラー、グローバルインストール、またはソースからのインストール）"
read_when:
  - OpenClaw をインストールする
  - GitHub からインストールしたいとき
---

<div id="install">
  # インストール
</div>

特別な理由がない限り、インストーラーを使用してください。CLI のセットアップとオンボーディングを自動で実行します。

<div id="quick-install-recommended">
  ## クイックインストール（推奨）
</div>

```bash
curl -fsSL https://openclaw.bot/install.sh | bash
```

Windows（PowerShell）:

```powershell
iwr -useb https://openclaw.ai/install.ps1 | iex
```

次の手順（オンボーディングをスキップした場合）：

```bash
openclaw onboard --install-daemon
```

<div id="system-requirements">
  ## システム要件
</div>

* **Node &gt;=22**
* macOS、Linux、または WSL2 経由の Windows
* ソースからビルドする場合にのみ `pnpm` が必要です

<div id="choose-your-install-path">
  ## インストール方法を選ぶ
</div>

<div id="1-installer-script-recommended">
  ### 1) インストールスクリプト（推奨）
</div>

`openclaw` を npm 経由でグローバルインストールし、オンボーディング（初期セットアップ）を実行します。

```bash
curl -fsSL https://openclaw.bot/install.sh | bash
```

インストール用フラグ:

```bash
curl -fsSL https://openclaw.bot/install.sh | bash -s -- --help
```

詳細: [インストーラーの内部動作](/ja/install/installer)。

非対話モード (オンボーディングをスキップ):

```bash
curl -fsSL https://openclaw.bot/install.sh | bash -s -- --no-onboard
```

<div id="2-global-install-manual">
  ### 2) グローバルインストール（手動）
</div>

すでに Node.js がインストールされている場合：

```bash
npm install -g openclaw@latest
```

システム全体に libvips がインストールされている場合（macOS では Homebrew 経由などでよくあります）に `sharp` のインストールが失敗する場合は、プリビルド済みバイナリの使用を強制してください：

```bash
SHARP_IGNORE_GLOBAL_LIBVIPS=1 npm install -g openclaw@latest
```

`sharp: Please add node-gyp to your dependencies` と表示された場合は、ビルドツールをインストールする（macOS: Xcode CLT + `npm install -g node-gyp`）か、上記の `SHARP_IGNORE_GLOBAL_LIBVIPS=1` 回避策を使ってネイティブビルドをスキップしてください。

または:

```bash
pnpm add -g openclaw@latest
```

次に行うこと:

```bash
openclaw onboard --install-daemon
```

<div id="3-from-source-contributorsdev">
  ### 3) ソースコードからインストール（コントリビューター／開発者向け）
</div>

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm ui:build # 初回実行時にUI依存関係を自動インストールします
pnpm build
openclaw onboard --install-daemon
```

ヒント：まだグローバルインストールしていない場合は、`pnpm openclaw ...` でリポジトリのコマンドを実行してください。

<div id="4-other-install-options">
  ### 4) その他のインストール方法
</div>

* Docker: [Docker](/ja/install/docker)
* Nix: [Nix](/ja/install/nix)
* Ansible: [Ansible](/ja/install/ansible)
* Bun（CLI のみ）: [Bun](/ja/install/bun)

<div id="after-install">
  ## インストール後
</div>

* オンボーディングを実行する: `openclaw onboard --install-daemon`
* クイックチェックを行う: `openclaw doctor`
* Gateway の状態を確認する: `openclaw status` + `openclaw health`
* ダッシュボードを開く: `openclaw dashboard`

<div id="install-method-npm-vs-git-installer">
  ## インストール方法: npm と git（インストーラー）
</div>

インストーラーは次の 2 つの方法をサポートしています:

* `npm`（デフォルト）: `npm install -g openclaw@latest`
* `git`: GitHub からクローンしてビルドし、チェックアウトしたソースから実行

<div id="cli-flags">
  ### CLI フラグ
</div>

```bash
# npm を明示的に指定
curl -fsSL https://openclaw.bot/install.sh | bash -s -- --install-method npm

# GitHub からインストール（ソースをチェックアウト）
curl -fsSL https://openclaw.bot/install.sh | bash -s -- --install-method git
```

共通フラグ:

* `--install-method npm|git`
* `--git-dir <path>` (デフォルト: `~/openclaw`)
* `--no-git-update` (既存のチェックアウトを使用する場合に `git pull` をスキップ)
* `--no-prompt` (対話プロンプトを無効化; CI/自動化環境では必須)
* `--dry-run` (実行予定の処理を表示するだけで、変更は行わない)
* `--no-onboard` (オンボーディング処理をスキップ)

<div id="environment-variables">
  ### 環境変数
</div>

同等の環境変数（自動化に便利）:

* `OPENCLAW_INSTALL_METHOD=git|npm`
* `OPENCLAW_GIT_DIR=...`
* `OPENCLAW_GIT_UPDATE=0|1`
* `OPENCLAW_NO_PROMPT=1`
* `OPENCLAW_DRY_RUN=1`
* `OPENCLAW_NO_ONBOARD=1`
* `SHARP_IGNORE_GLOBAL_LIBVIPS=0|1`（デフォルト: `1`。`sharp` がシステムの libvips に対してビルドされるのを回避します）

<div id="troubleshooting-openclaw-not-found-path">
  ## トラブルシューティング：`openclaw` が見つからない（PATH）
</div>

簡易診断：

```bash
node -v
npm -v
npm prefix -g
echo "$PATH"
```

`echo "$PATH"` を実行した結果に `$(npm prefix -g)/bin`（macOS/Linux）または `$(npm prefix -g)`（Windows）が含まれて**いない**場合、シェルはグローバル npm バイナリ（`openclaw` を含む）を見つけられません。

修正方法: シェルの起動ファイルにこれを追加します（zsh: `~/.zshrc`, bash: `~/.bashrc`）:

```bash
# macOS / Linux
export PATH="$(npm prefix -g)/bin:$PATH"
```

Windows では、`npm prefix -g` の出力結果を PATH 環境変数に追加してください。

その後、新しいターミナルを開くか、zsh では `rehash`、bash では `hash -r` を実行してください。

<div id="update-uninstall">
  ## 更新 / アンインストール
</div>

* 更新: [更新手順](/ja/install/updating)
* 新しいマシンへの移行: [移行手順](/ja/install/migrating)
* アンインストール: [アンインストール手順](/ja/install/uninstall)