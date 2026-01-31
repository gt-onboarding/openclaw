---
title: 更新
summary: "OpenClaw を安全に更新する方法（グローバルインストールまたはソースからのインストール）、およびロールバック戦略"
read_when:
  - OpenClaw を更新しているとき
  - 更新後に何かが壊れたとき
---

<div id="updating">
  # アップデート
</div>

OpenClaw は「1.0」前の段階で、開発が非常に活発に進んでいます。アップデートは本番インフラのリリース運用と同じように扱ってください: アップデート → チェックの実行 → 再起動（あるいは再起動まで自動で行う `openclaw update` を使用） → 検証。

<div id="recommended-re-run-the-website-installer-upgrade-in-place">
  ## 推奨: Web サイトのインストーラーを再実行する（インプレースアップグレード）
</div>

**推奨される**アップデート方法は、Web サイトからインストーラーを再実行することです。これにより、既存のインストールを検出してそのままアップグレードし、必要に応じて `openclaw doctor` を実行します。

```bash
curl -fsSL https://openclaw.bot/install.sh | bash
```

注意:

* オンボーディングウィザードを再度実行したくない場合は、`--no-onboard` を追加します。
* **ソースからのインストール**の場合は、次を使用します:
  ```bash
  curl -fsSL https://openclaw.bot/install.sh | bash -s -- --install-method git --no-onboard
  ```
  インストーラーは、リポジトリがクリーンな場合に **のみ** `git pull --rebase` を実行します。
* **グローバルインストール**の場合、このスクリプトは内部的に `npm install -g openclaw@latest` を使用します。
* 旧バージョンに関する注意: `openclaw` は互換性維持用のシムとして引き続き利用可能です。

<div id="before-you-update">
  ## アップデート前に確認しておくこと
</div>

* インストール方法を把握しておく: **グローバルインストール**（npm/pnpm）か **ソースから**（git clone）か。
* Gateway の起動方法を把握しておく: **フォアグラウンドのターミナル** か **常駐サービス**（launchd/systemd）か。
* カスタマイズ内容のスナップショットを取る:
  * 設定: `~/.openclaw/openclaw.json`
  * 認証情報: `~/.openclaw/credentials/`
  * ワークスペース: `~/.openclaw/workspace`

<div id="update-global-install">
  ## アップデート（グローバルインストール）
</div>

グローバルインストールの場合（いずれか一つを選択）:

```bash
npm i -g openclaw@latest
```

```bash
pnpm add -g openclaw@latest
```

Gateway のランタイムとして Bun を使用することは **推奨しません**（WhatsApp/Telegram 関連の不具合があります）。

更新チャネルを切り替えるには（git + npm でインストールした場合）:

```bash
openclaw update --channel beta
openclaw update --channel dev
openclaw update --channel stable
```

単発のインストールで特定のタグ／バージョンを指定する場合は、`--tag <dist-tag|version>` を使用します。

チャネルの意味や挙動、リリースノートについては、[Development channels](/ja/install/development-channels) を参照してください。

注意：npm インストールでは、Gateway は起動時にアップデート案内のログを出力します（現在のチャネルタグをチェックします）。`update.checkOnStart: false` で無効化できます。

次に:

```bash
openclaw doctor
openclaw gateway restart
openclaw health
```

Notes:

* Gateway をサービスとして実行している場合は、PID を kill するのではなく、`openclaw gateway restart` を実行してください。
* 特定バージョンに固定している場合は、以下の「Rollback / pinning」を参照してください。

<div id="update-openclaw-update">
  ## 更新（`openclaw update`）
</div>

**ソースからインストールした環境**（git checkout）の場合は、次の方法を推奨します。

```bash
openclaw update
```

これは比較的安全なアップデートフローを実行します：

* クリーンな作業ツリーが必要です。
* 選択したチャネル（タグまたはブランチ）に切り替えます。
* 設定された upstream（dev チャネル）に対して fetch と rebase を実行します。
* 依存関係をインストールし、ビルドし、Control UI をビルドし、`openclaw doctor` を実行します。
* デフォルトで Gateway を再起動します（スキップするには `--no-restart` を使用）。

**npm/pnpm**（git メタデータなし）でインストールしている場合、`openclaw update` は使用中のパッケージマネージャー経由での更新を試みます。インストール方法を検出できない場合は、代わりに「Update (global install)」を使用してください。

<div id="update-control-ui-rpc">
  ## アップデート（Control UI / RPC）
</div>

Control UI には **Update &amp; Restart**（RPC: `update.run`）があります。これは次の処理を行います。

1. `openclaw update` と同じソース更新フローを実行します（git checkout のみ）。
2. 構造化されたレポート（stdout/stderr の末尾）付きで、再起動用センチネルを書き込みます。
3. Gateway を再起動し、最後にアクティブだったセッションへレポート付きで ping を送ります。

rebase に失敗した場合、Gateway はアップデートを適用せずに中止して再起動します。

<div id="update-from-source">
  ## 更新（ソースから）
</div>

リポジトリをチェックアウトしたディレクトリで実行:

推奨:

```bash
openclaw update
```

手動（ほぼ同等の手順）:

```bash
git pull
pnpm install
pnpm build
pnpm ui:build # 初回実行時にUI依存関係を自動インストールする
openclaw doctor
openclaw health
```

Notes:

* パッケージ済みの `openclaw` バイナリ（[`openclaw.mjs`](https://github.com/openclaw/openclaw/blob/main/openclaw.mjs)）を実行する場合や、Node.js を使って `dist/` を実行する場合は、`pnpm build` が重要です。
* グローバルインストールなしでリポジトリのチェックアウトから実行する場合、CLI コマンドには `pnpm openclaw ...` を使用してください。
* TypeScript から直接実行する場合（`pnpm openclaw ...`）、通常は再ビルドは不要ですが、**設定マイグレーションは引き続き必要です** → `openclaw doctor` を実行してください。
* グローバルインストールと git インストールの切り替えは簡単です。もう一方の方式をインストールしてから、`openclaw doctor` を実行し、Gateway サービスのエントリポイントを現在のインストールに合わせて書き換えます。

<div id="always-run-openclaw-doctor">
  ## 常に実行する: `openclaw doctor`
</div>

Doctor は「安全なアップデート」用のコマンドです。あえて地味な動作になるよう設計されており、修復 + マイグレーション + 警告を行います。

注意：**ソースからのインストール**（git checkout）の場合、`openclaw doctor` は最初に `openclaw update` を実行するよう提案します。

主な処理内容:

* 非推奨の設定キーやレガシーな設定ファイル配置のマイグレーション。
* DM ポリシーをチェックし、リスクの高い「open」設定（すべてのユーザーからのメッセージを無制限に受け付けることを許可する設定）について警告。
* Gateway のヘルスチェックを行い、必要に応じて再起動を提案。
* 旧バージョンの Gateway サービス（launchd/systemd、レガシーな schtasks）を検出し、現在の OpenClaw サービスへマイグレーション。
* Linux では、systemd の user lingering を有効化し（ログアウト後も Gateway が動作し続けるようにする）。

詳細: [Doctor](/ja/gateway/doctor)

<div id="start-stop-restart-the-gateway">
  ## Gateway の起動 / 停止 / 再起動
</div>

CLI（OS を問わず動作）:

```bash
openclaw gateway status
openclaw gateway stop
openclaw gateway restart
openclaw gateway --port 18789
openclaw logs --follow
```

`supervised` 環境で実行している場合:

* macOS launchd (アプリ同梱の LaunchAgent): `launchctl kickstart -k gui/$UID/bot.molt.gateway`（`bot.molt.<profile>` を使用。従来の `com.openclaw.*` も引き続き有効）
* Linux systemd ユーザーサービス: `systemctl --user restart openclaw-gateway[-<profile>].service`
* Windows (WSL2): `systemctl --user restart openclaw-gateway[-<profile>].service`
  * `launchctl` / `systemctl` はサービスがインストールされている場合にのみ動作します。未インストールの場合は `openclaw gateway install` を実行してください。

Runbook および正確なサービスラベルについては [Gateway runbook](/ja/gateway) を参照してください。

<div id="rollback-pinning-when-something-breaks">
  ## ロールバック / バージョン固定（トラブル発生時）
</div>

<div id="pin-global-install">
  ### ピン留め（グローバルインストール）
</div>

既知の正常動作バージョンをインストールします（`<version>` を最後に問題なく動作していたバージョンに置き換えてください）:

```bash
npm i -g openclaw@<version>
```

```bash
pnpm add -g openclaw@<version>
```

Tip: 現在公開されているバージョンを確認するには、`npm view openclaw version` を実行してください。

その後、再起動してから doctor を再実行してください。

```bash
openclaw doctor
openclaw gateway restart
```

<div id="pin-source-by-date">
  ### 日付でピン留め（ソース）
</div>

特定の日付時点のコミットを選択します（例: 「2026-01-01 時点の main の状態」）:

```bash
git fetch origin
git checkout "$(git rev-list -n 1 --before=\"2026-01-01\" origin/main)"
```

その後、依存パッケージを再インストールして再起動してください:

```bash
pnpm install
pnpm build
openclaw gateway restart
```

あとで最新版に戻したくなった場合は：

```bash
git checkout main
git pull
```

<div id="if-youre-stuck">
  ## 行き詰まったときは
</div>

* `openclaw doctor` をもう一度実行し、出力をよく確認してください（多くの場合、解決策がそのまま書かれています）。
* 次を確認してください: [トラブルシューティング](/ja/gateway/troubleshooting)
* Discord で質問: https://channels.discord.gg/clawd