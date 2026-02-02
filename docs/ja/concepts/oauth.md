---
title: OAuth
summary: "OpenClaw における OAuth: トークンの交換、保存、マルチアカウントパターン"
read_when:
  - OpenClaw の OAuth をエンドツーエンドで理解したいとき
  - トークンの無効化やログアウトの問題に遭遇しているとき
  - setup-token や OAuth 認証フローを設定・利用したいとき
  - 複数アカウントやプロファイルルーティングを行いたいとき
---

<div id="oauth">
  # OAuth
</div>

OpenClaw は、OAuth を用いた「サブスクリプション認証」を、これを提供しているプロバイダーに対してサポートしています（特に **OpenAI Codex (ChatGPT OAuth)**）。Anthropic のサブスクリプションについては、**setup-token** フローを使用してください。このページでは次の内容を説明します:

* OAuth の **トークン交換**（PKCE）がどのように動作するか
* トークンがどこに**保存**されるか（およびその理由）
* **複数アカウント**をどのように扱うか（プロファイル + セッションごとのオーバーライド）

OpenClaw は、独自の OAuth や API キー フローを提供する **プロバイダー用プラグイン** もサポートします。これらは次のように実行します:

```bash
openclaw models auth login --provider <id>
```

<div id="the-token-sink-why-it-exists">
  ## トークンシンク（なぜ存在するか）
</div>

OAuth プロバイダーは、ログイン／リフレッシュフローの途中で一般的に**新しいリフレッシュトークン**を発行します。プロバイダー（や OAuth クライアント）によっては、同じユーザー／アプリ用に新しいリフレッシュトークンが発行されたときに、古いリフレッシュトークンを無効化する場合があります。

実際に起きがちな症状:

* OpenClaw *と* Claude Code / Codex CLI の両方でログインしている → 後でどちらか一方がランダムに「ログアウト」される

これを抑えるために、OpenClaw は `auth-profiles.json` を **トークンシンク**として扱います:

* ランタイムは認証情報を**1 箇所**から読み取る
* 複数のプロファイルを保持し、決まったルールで安定してルーティングできる

<div id="storage-where-tokens-live">
  ## ストレージ（トークンの保存場所）
</div>

シークレットは**エージェントごと**に保存されます:

* 認証プロファイル（OAuth + API キー）: `~/.openclaw/agents/<agentId>/agent/auth-profiles.json`
* ランタイムキャッシュ（自動管理されるため編集しないこと）: `~/.openclaw/agents/<agentId>/agent/auth.json`

レガシーなインポート専用ファイル（まだサポートされていますが、メインのストアではありません）:

* `~/.openclaw/credentials/oauth.json`（初回利用時に `auth-profiles.json` にインポートされます）

上記のすべては `$OPENCLAW_STATE_DIR`（state ディレクトリの上書き設定）にも従います。詳細は [/gateway/configuration](/ja/gateway/configuration#auth-storage-oauth--api-keys) を参照してください。

<div id="anthropic-setup-token-subscription-auth">
  ## Anthropic setup-token（サブスクリプション認証）
</div>

任意のマシン上で `claude setup-token` を実行し、表示されたトークンを OpenClaw に貼り付けてください。

```bash
openclaw models auth setup-token --provider anthropic
```

トークンを別の場所で生成した場合は、手動で貼り付けてください。

```bash
openclaw models auth paste-token --provider anthropic
```

検証:

```bash
openclaw models status
```

<div id="oauth-exchange-how-login-works">
  ## OAuth エクスチェンジ（ログインの仕組み）
</div>

OpenClaw のインタラクティブなログインフローは `@mariozechner/pi-ai` 内で実装されており、ウィザードやコマンドに組み込まれています。

<div id="anthropic-claude-promax-setup-token">
  ### Anthropic (Claude Pro/Max) setup-token
</div>

フローの流れ:

1. `claude setup-token` を実行する
2. トークンを OpenClaw に貼り付ける
3. トークン認証プロファイル（リフレッシュなし）として保存する

ウィザードのパスは `openclaw onboard` → 認証方式の選択で `setup-token` (Anthropic) を選択する。

<div id="openai-codex-chatgpt-oauth">
  ### OpenAI Codex (ChatGPT OAuth)
</div>

フローの流れ（PKCE）:

1. PKCE verifier/challenge とランダムな `state` を生成する
2. `https://auth.openai.com/oauth/authorize?...` を開く
3. `http://127.0.0.1:1455/auth/callback` でコールバックを受信することを試みる
4. コールバックをバインドできない場合（リモート/ヘッドレス環境など）は、リダイレクト URL またはコードを貼り付ける
5. `https://auth.openai.com/oauth/token` でトークン交換を行う
6. アクセストークンから `accountId` を取得し、`{ access, refresh, expires, accountId }` を保存する

Wizard のパスは `openclaw onboard` → 認証方式の選択で `openai-codex` を選択する。

<div id="refresh-expiry">
  ## リフレッシュと有効期限
</div>

プロファイルには `expires` タイムスタンプが保存されます。

実行時には:

* `expires` が将来の時刻の場合 → 保存されているアクセストークンを使用
* 有効期限が切れている場合 → （ファイルロックを取得した上で）リフレッシュし、保存済みの認証情報を上書き

リフレッシュ処理は自動で行われるため、通常はトークンを手動で管理する必要はありません。

<div id="multiple-accounts-profiles-routing">
  ## 複数アカウント（プロファイル）とルーティング
</div>

次の 2 つのパターンがあります：

<div id="1-preferred-separate-agents">
  ### 1) 推奨: エージェントを分ける
</div>

「個人用」と「仕事用」を一切交わらせたくない場合は、分離されたエージェント（セッション + 認証情報 + ワークスペースをそれぞれ別々にしたもの）を使用してください。

```bash
openclaw agents add work
openclaw agents add personal
```

その後、エージェントごとに認証をウィザードで設定し、チャットを適切なエージェントにルーティングします。

<div id="2-advanced-multiple-profiles-in-one-agent">
  ### 2) 上級編: 1つのエージェントで複数のプロファイルを使う
</div>

`auth-profiles.json` は、同じプロバイダーに対して複数のプロファイル ID をサポートします。

どのプロファイルを使うかを選択します:

* 設定内の順序 (`auth.order`) によるグローバル指定
* `/model ...@<profileId>` によるセッション単位の指定

例 (セッション側での上書き):

* `/model Opus@anthropic:work`

利用可能なプロファイル ID を確認するには:

* `openclaw channels list --json` (`auth[]` が表示される)

関連ドキュメント:

* [/concepts/model-failover](/ja/concepts/model-failover) (ローテーション + クールダウンのルール)
* [/tools/slash-commands](/ja/tools/slash-commands) (利用可能なコマンド)