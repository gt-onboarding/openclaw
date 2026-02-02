---
title: インストーラー
summary: "インストーラー・スクリプト（install.sh と install-cli.sh）の動作、フラグ、そして自動化について"
read_when:
  - "You want to understand `openclaw.bot/install.sh`"
  - "You want to automate installs (CI / headless)"
  - "You want to install from a GitHub checkout"
---

<div id="installer-internals">
  # インストーラーの内部仕様
</div>

OpenClaw は 2 つのインストーラースクリプトを提供しています（`openclaw.ai` から配信）:

* `https://openclaw.bot/install.sh` — 「推奨」のインストーラー（デフォルトでグローバルな npm インストール。GitHub のチェックアウトからのインストールも可能）
* `https://openclaw.bot/install-cli.sh` — 非 root 環境向けの CLI インストーラー（独自の Node を含むプレフィックスディレクトリにインストール）
* `https://openclaw.ai/install.ps1` — Windows PowerShell インストーラー（デフォルトは npm。任意で git インストールも可能）

現在のフラグや挙動を確認するには、次を実行します:

```bash
curl -fsSL https://openclaw.bot/install.sh | bash -s -- --help
```

Windows（PowerShell）のヘルプ：

```powershell
& ([scriptblock]::Create((iwr -useb https://openclaw.ai/install.ps1))) -?
```

インストーラーの実行が完了したにもかかわらず、新しいターミナルで `openclaw` が見つからない場合は、通常 Node/npm の PATH の設定に問題があることが原因です。詳しくは [Install](/ja/install#nodejs--npm-path-sanity) を参照してください。

<div id="installsh-recommended">
  ## install.sh (推奨)
</div>

概要としては、次のことを行います:

* OS を検出します（macOS / Linux / WSL）。
* Node.js **22以上** が存在することを確認します（macOS は Homebrew 経由、Linux は NodeSource 経由）。
* インストール方法を選択します:
  * `npm`（デフォルト）：`npm install -g openclaw@latest`
  * `git`: ソースを clone/build し、ラッパースクリプトをインストールします
* Linux では、必要に応じて npm の prefix を `~/.npm-global` に切り替えることで、グローバル npm 権限エラーを回避します。
* 既存インストールをアップグレードする場合: `openclaw doctor --non-interactive` を実行します（ベストエフォート）。
* git インストールの場合: インストール／更新後に `openclaw doctor --non-interactive` を実行します（ベストエフォート）。
* デフォルトで `SHARP_IGNORE_GLOBAL_LIBVIPS=1` を設定し、`sharp` のネイティブインストールに関するトラブルを軽減します（システムの libvips に対してビルドすることを避けます）。

もし *意図的に* グローバルインストールされた libvips に対して `sharp` をリンクさせたい場合（またはデバッグ中の場合）は、次を設定してください:

```bash
SHARP_IGNORE_GLOBAL_LIBVIPS=0 curl -fsSL https://openclaw.bot/install.sh | bash
```

<div id="discoverability-git-install-prompt">
  ### 検出 / 「git install」プロンプト
</div>

**すでに OpenClaw のソースコードをチェックアウトしたディレクトリ内**（`package.json` と `pnpm-workspace.yaml` によって検出）でインストーラを実行すると、次のようなプロンプトが表示されます:

* このチェックアウトを更新して利用する（`git`）
* もしくはグローバルな npm インストールへ移行する（`npm`）

非対話的なコンテキスト（TTY なし / `--no-prompt` 使用時）では、`--install-method git|npm` を指定する（または `OPENCLAW_INSTALL_METHOD` を設定する）必要があります。指定しない場合、スクリプトは終了コード `2` で終了します。

<div id="why-git-is-needed">
  ### Git が必要な理由
</div>

`--install-method git` パス（clone / pull）には Git が必須です。

`npm` でのインストールでは、通常 Git は必須ではありませんが、環境によっては Git が必要になる場合があります（例: パッケージや依存関係が git URL 経由で取得される場合）。インストーラーは現在、特に新規インストール直後のディストリビューションで `spawn git ENOENT` が発生するのを避けるために、Git が存在することを確認します。

<div id="why-npm-hits-eacces-on-fresh-linux">
  ### なぜ新規セットアップの Linux で npm が `EACCES` になるのか
</div>

一部の Linux 環境（特にシステムのパッケージマネージャーや NodeSource 経由で Node をインストールした場合）では、npm のグローバル prefix が root 所有のディレクトリを指していることがあります。この状態だと、`npm install -g ...` が `EACCES` や `mkdir` のパーミッションエラーで失敗します。

`install.sh` は、prefix を次のように切り替えることでこれを回避します:

* `~/.npm-global`（存在する場合は `~/.bashrc` / `~/.zshrc` の `PATH` に追加）

<div id="install-clish-non-root-cli-installer">
  ## install-cli.sh (非root環境向け CLI インストーラー)
</div>

このスクリプトは `openclaw` を指定されたプレフィックス（デフォルト: `~/.openclaw`）にインストールし、そのプレフィックス配下に専用の Node.js ランタイムもインストールします。これにより、システムの Node/npm を変更したくないマシン上でも動作させることができます。

ヘルプ:

```bash
curl -fsSL https://openclaw.bot/install-cli.sh | bash -s -- --help
```

<div id="installps1-windows-powershell">
  ## install.ps1 (Windows PowerShell)
</div>

このスクリプトが行うこと（概要）:

* Node.js **22 以上** がインストールされていることを確認します（winget、Chocolatey、Scoop または手動）。
* インストール方法を選択します:
  * `npm`（デフォルト）：`npm install -g openclaw@latest`
  * `git`：ソースコードを clone / build して、ラッパースクリプトをインストール
* アップグレードおよび git からのインストール時に、`openclaw doctor --non-interactive` を（ベストエフォートで）実行します。

例:

```powershell
iwr -useb https://openclaw.ai/install.ps1 | iex
```

```powershell
iwr -useb https://openclaw.ai/install.ps1 | iex -InstallMethod git
```

```powershell
iwr -useb https://openclaw.ai/install.ps1 | iex -InstallMethod git -GitDir "C:\\openclaw"
```

環境変数:

* `OPENCLAW_INSTALL_METHOD=git|npm`
* `OPENCLAW_GIT_DIR=...`

Git の前提条件:

`-InstallMethod git` を選択していて Git が見つからない場合、インストーラは
Git for Windows のリンク (`https://git-scm.com/download/win`) を表示して終了します。

Windows でよくある問題:

* **npm error spawn git / ENOENT**: Git for Windows をインストールし、PowerShell を再度開いてからインストーラを再実行してください。
* **&quot;openclaw&quot; is not recognized**: npm のグローバル bin フォルダが PATH に含まれていません。多くのシステムでは
  `%AppData%\\npm` が使われます。`npm config get prefix` を実行し、その出力に対して `\\bin` を追加したパスを PATH に設定し、PowerShell を再度開いてください。
