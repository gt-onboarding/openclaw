---
title: セットアップ
summary: "セットアップガイド: 自分好みに調整した OpenClaw 環境を保ちながら、常に最新の状態を維持する"
read_when:
  - 新しいマシンをセットアップするとき
  - 自分の環境を壊さずに「最新かつ最良」を維持したいとき
---

<div id="setup">
  # セットアップ
</div>

最終更新: 2026-01-01

<div id="tldr">
  ## TL;DR
</div>

* **テーラリング（カスタマイズ）はリポジトリの外に配置する:** `~/.openclaw/workspace`（ワークスペース） + `~/.openclaw/openclaw.json`（設定）。
* **安定運用向けワークフロー:** macOS アプリをインストールし、同梱の Gateway を実行させておく。
* **最新機能重視のワークフロー:** `pnpm gateway:watch` で自分で Gateway を起動してから、macOS アプリをローカルモードで接続させる。

<div id="prereqs-from-source">
  ## 前提条件（元ドキュメントより）
</div>

* Node `>=22`
* `pnpm`
* Docker（任意。コンテナ化されたセットアップ／e2e 用のみ — [Docker](/ja/install/docker) を参照）

<div id="tailoring-strategy-so-updates-dont-hurt">
  ## カスタマイズ戦略（アップデートで困らないように）
</div>

「自分専用に100%カスタマイズ」しつつアップデートも簡単にしたい場合は、カスタマイズ内容を次の場所にまとめておきます。

* **Config:** `~/.openclaw/openclaw.json`（JSON/JSON5 風）
* **Workspace:** `~/.openclaw/workspace`（スキル、プロンプト、メモリ。プライベートな git リポジトリにする）

初回だけ以下を実行します:

```bash
openclaw setup
```

このリポジトリ内から、ローカルの CLI エントリポイントを使用してください:

```bash
openclaw setup
```

まだグローバルインストールしていない場合は、`pnpm openclaw setup` を実行してください。

<div id="stable-workflow-macos-app-first">
  ## 安定したワークフロー（macOS アプリ優先）
</div>

1. **OpenClaw.app**（メニューバー）をインストールして起動します。
2. オンボーディング／権限チェックリスト（TCC プロンプト）を完了します。
3. Gateway が **Local** として起動していることを確認します（アプリ側で管理されます）。
4. サーフェスをリンクします（例: WhatsApp）:

```bash
openclaw channels login
```

5. 動作確認:

```bash
openclaw health
```

お使いのビルドでオンボーディングが利用できない場合は、次の手順を実行します。

* `openclaw setup` を実行し、続けて `openclaw channels login` を実行し、最後に `openclaw gateway` で Gateway を手動起動します。

<div id="bleeding-edge-workflow-gateway-in-a-terminal">
  ## 最新開発版ワークフロー（ターミナルで Gateway を実行）
</div>

目的：TypeScript 製 Gateway を開発しながらホットリロードを有効にし、macOS アプリの UI を接続したままにしておく。

<div id="0-optional-run-the-macos-app-from-source-too">
  ### 0)（オプション）macOS アプリもソースから実行する
</div>

最新の開発版の macOS アプリも利用したい場合は：

```bash
./scripts/restart-mac.sh
```

<div id="1-start-the-dev-gateway">
  ### 1) 開発用の Gateway を起動する
</div>

```bash
pnpm install
pnpm gateway:watch
```

`gateway:watch` は Gateway をウォッチモードで起動し、TypeScript の変更を検知するとリロードします。

<div id="2-point-the-macos-app-at-your-running-gateway">
  ### 2) 動作中の Gateway を macOS アプリの接続先として指定する
</div>

**OpenClaw.app** で:

* 接続モード (Connection Mode): **Local**
  アプリは、設定されたポートで動作中の Gateway に接続します。

<div id="3-verify">
  ### 3) 検証
</div>

* アプリ内の Gateway のステータスは **“Using existing gateway …”** と表示されているはずです
* あるいは CLI から確認します:

```bash
openclaw health
```

<div id="common-footguns">
  ### よくあるやりがちな落とし穴
</div>

* **ポートの誤り:** Gateway の WS のデフォルトは `ws://127.0.0.1:18789` です。アプリと CLI は同じポートを使うようにしてください。
* **状態が保存される場所:**
  * 認証情報: `~/.openclaw/credentials/`
  * セッション: `~/.openclaw/agents/<agentId>/sessions/`
  * ログ: `/tmp/openclaw/`

<div id="credential-storage-map">
  ## 認証情報ストレージマップ
</div>

認証関連のデバッグを行うときや、どのファイルをバックアップすべきか判断するときの参考にしてください：

* **WhatsApp**: `~/.openclaw/credentials/whatsapp/<accountId>/creds.json`
* **Telegram bot token**: config/env または `channels.telegram.tokenFile`
* **Discord bot token**: config/env（トークンファイルはまだ未対応）
* **Slack tokens**: config/env（`channels.slack.*`）
* **ペアリング許可リスト**: `~/.openclaw/credentials/<channel>-allowFrom.json`
* **モデル認証プロファイル**: `~/.openclaw/agents/<agentId>/agent/auth-profiles.json`
* **レガシー OAuth インポート**: `~/.openclaw/credentials/oauth.json`

詳しくは [セキュリティ](/ja/gateway/security#credential-storage-map) を参照してください。

<div id="updating-without-wrecking-your-setup">
  ## 更新（セットアップを壊さずに行う）
</div>

* `~/.openclaw/workspace` と `~/.openclaw/` は「あなたのもの」として使い、個人用のプロンプトや設定は `openclaw` リポジトリ内に置かないでください。
* ソースコードの更新: `git pull` ＋（lockfile が変更された場合は）`pnpm install` ＋ 引き続き `pnpm gateway:watch` を使用します。

<div id="linux-systemd-user-service">
  ## Linux（systemd ユーザーサービス）
</div>

Linux 環境へのインストールでは、systemd の **ユーザー**サービスを使用します。デフォルトでは、systemd はログアウト／アイドル時にユーザーサービスを停止するため、Gateway が終了してしまいます。オンボーディング時に lingering を有効化しようと試みます（sudo の入力を求められる場合があります）。それでも無効なままの場合は、次を実行してください：

```bash
sudo loginctl enable-linger $USER
```

常時稼働させるサーバーやマルチユーザー環境のサーバーでは、ユーザーサービスではなく **system** サービスの利用を検討してください（lingering は不要です）。systemd に関する注意点については [Gateway runbook](/ja/gateway) を参照してください。

<div id="related-docs">
  ## 関連ドキュメント
</div>

* [Gateway ランブック](/ja/gateway)（フラグ、監視、ポート）
* [Gateway 設定](/ja/gateway/configuration)（設定スキーマとサンプル）
* [Discord](/ja/channels/discord) と [Telegram](/ja/channels/telegram)（返信タグと replyToMode 設定）
* [OpenClaw アシスタントのセットアップ](/ja/start/openclaw)
* [macOS アプリ](/ja/platforms/macos)（Gateway のライフサイクル）