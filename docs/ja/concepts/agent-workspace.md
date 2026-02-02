---
title: エージェント・ワークスペース
summary: "エージェント・ワークスペース：場所、構成、バックアップ戦略"
read_when:
  - エージェント・ワークスペースやそのファイル構成について説明する必要があるとき
  - エージェント・ワークスペースのバックアップや移行を行いたいとき
---

<div id="agent-workspace">
  # Agent workspace
</div>

ワークスペースはエージェントの「ホーム」となる場所です。ファイル関連ツールとワークスペースコンテキストに使用される唯一の作業ディレクトリとして使われます。プライベートに保ち、メモリとして扱ってください。

これは、設定、認証情報、およびセッションを保存する `~/.openclaw/` とは別です。

**Important:** ワークスペースは **デフォルトの cwd** であり、厳密なサンドボックスではありません。ツールは相対パスをワークスペースを基準に解決しますが、サンドボックスが有効になっていない限り、絶対パスを使うとホスト上の他の場所にも到達できてしまいます。分離が必要な場合は、[`agents.defaults.sandbox`](/ja/gateway/sandboxing)（および／またはエージェント単位のサンドボックス設定）を使用してください。サンドボックスが有効になっていて `workspaceAccess` が `"rw"` でない場合、ツールはホスト側のワークスペースではなく、`~/.openclaw/sandboxes` 配下のサンドボックスワークスペース内で動作します。

<div id="default-location">
  ## デフォルトの場所
</div>

* デフォルト: `~/.openclaw/workspace`
* `OPENCLAW_PROFILE` が設定されていて `"default"` でない場合、デフォルトは
  `~/.openclaw/workspace-<profile>` になる。
* `~/.openclaw/openclaw.json` で上書きできる。

```json5
{
  agent: {
    workspace: "~/.openclaw/workspace"
  }
}
```

`openclaw onboard`、`openclaw configure`、または `openclaw setup` は、
ワークスペースが存在しない場合にワークスペースを作成し、ブートストラップ用ファイルを生成します。

すでに自分でワークスペース内のファイルを管理している場合は、ブートストラップ
ファイルの作成を無効にできます。

```json5
{ agent: { skipBootstrap: true } }
```

<div id="extra-workspace-folders">
  ## 余分なワークスペースフォルダ
</div>

古いインストールでは `~/openclaw` が作成されている場合があります。複数のワークスペース
ディレクトリが存在すると、同時にアクティブになるワークスペースは 1 つだけのため、
認証や状態が食い違って分かりにくくなることがあります。

**推奨:** アクティブなワークスペースは 1 つに保ってください。もう使っていない
余分なフォルダがある場合は、アーカイブするかゴミ箱に移動してください（例: `trash ~/openclaw`）。
意図的に複数のワークスペースを保持する場合は、
`agents.defaults.workspace` がアクティブなものを指していることを必ず確認してください。

`openclaw doctor` は、余分なワークスペースディレクトリを検出すると警告します。

<div id="workspace-file-map-what-each-file-means">
  ## ワークスペースのファイルマップ（各ファイルの意味）
</div>

これは、OpenClaw がワークスペース内に存在することを想定している標準ファイルです：

* `AGENTS.md`
  * エージェント向けの運用手順と、メモリの使い方に関する指示。
  * すべてのセッション開始時にロードされる。
  * ルール、優先順位、「どのように振る舞うべきか」の詳細を書くのに適した場所。

* `SOUL.md`
  * ペルソナ、口調、境界条件。
  * すべてのセッションでロードされる。

* `USER.md`
  * ユーザーが誰で、どのように呼びかけるべきか。
  * すべてのセッションでロードされる。

* `IDENTITY.md`
  * エージェントの名前、雰囲気、絵文字。
  * ブートストラップ儀式の最中に作成／更新される。

* `TOOLS.md`
  * ローカルツールと運用上の慣習に関するメモ。
  * ツールの利用可否は制御せず、あくまでガイダンスのみ。

* `HEARTBEAT.md`
  * ハートビート実行用の小さなチェックリスト（任意）。
  * トークン消費を抑えるため、短く保つこと。

* `BOOT.md`
  * 内部フックが有効な場合に、Gateway の再起動時に実行される任意の起動チェックリスト。
  * 短く保ち、外向きの送信には `message` ツールを使用すること。

* `BOOTSTRAP.md`
  * 初回実行時の一度きりの儀式。
  * 新規作成されたワークスペースにのみ作成される。
  * 儀式が完了したら削除すること。

* `memory/YYYY-MM-DD.md`
  * 日次メモリログ（1 日 1 ファイル）。
  * セッション開始時に、今日＋昨日分を読み込むことを推奨。

* `MEMORY.md` (任意)
  * キュレートされた長期メモリ。
  * 共有／グループコンテキストではなく、メインのプライベートセッションでのみロードすること。

ワークフローと自動メモリフラッシュについては、[Memory](/ja/concepts/memory) を参照。

* `skills/` (任意)
  * ワークスペース固有のスキル。
  * 名前が衝突した場合、管理／バンドル済みスキルよりこちらが優先される。

* `canvas/` (任意)
  * ノードディスプレイ向けの Canvas UI ファイル（例：`canvas/index.html`）。

ブートストラップ用ファイルが欠けている場合、OpenClaw はセッション内に「missing file」マーカーを挿入して処理を継続します。大きなブートストラップファイルは挿入時に切り詰められます。
上限は `agents.defaults.bootstrapMaxChars` で調整できます（デフォルト: 20000）。
`openclaw setup` を実行すると、既存ファイルを上書きせずに欠けているデフォルトファイルを再作成できます。

<div id="what-is-not-in-the-workspace">
  ## ワークスペースに含めないもの
</div>

これらは `~/.openclaw/` 配下にあり、ワークスペースのリポジトリには **コミットしてはいけません**:

* `~/.openclaw/openclaw.json` (設定)
* `~/.openclaw/credentials/` (OAuth トークン、API キー)
* `~/.openclaw/agents/<agentId>/sessions/` (セッションのログ + メタデータ)
* `~/.openclaw/skills/` (管理対象スキル)

セッションや設定を移行する必要がある場合は、別途コピーし、
バージョン管理の対象外にしておいてください。

<div id="git-backup-recommended-private">
  ## Git バックアップ（推奨・非公開）
</div>

ワークスペースはプライベートなメモリとして扱ってください。バックアップされて復元できるように、**非公開**の git リポジトリに保存してください。

Gateway が動作しているマシン上（つまりワークスペースが存在する場所）で、次の手順を実行します。

<div id="1-initialize-the-repo">
  ### 1) リポジトリを初期化する
</div>

git がインストールされていれば、新しいワークスペースは自動的にリポジトリとして初期化されます。このワークスペースがまだリポジトリになっていない場合は、次を実行してください：

```bash
cd ~/.openclaw/workspace
git init
git add AGENTS.md SOUL.md TOOLS.md IDENTITY.md USER.md HEARTBEAT.md memory/
git commit -m "Add agent workspace"
```

<div id="2-add-a-private-remote-beginner-friendly-options">
  ### 2) プライベートリモートを追加する（初心者向けオプション）
</div>

オプション A: GitHub の Web UI

1. GitHub で新しい **プライベート** リポジトリを作成します。
2. README を作成して初期化しないでください（マージコンフリクトを避けるため）。
3. HTTPS リモート URL をコピーします。
4. リモートを追加して push します。

```bash
git branch -M main
git remote add origin <https-url>
git push -u origin main
```

オプション B: GitHub CLI (`gh`)

```bash
gh auth login
gh repo create openclaw-workspace --private --source . --remote origin --push
```

Option C: GitLab の Web UI

1. GitLab で新しい **プライベート** リポジトリを作成します。
2. README を作成して初期化しないでください（マージコンフリクトを避けるため）。
3. HTTPS のリモート URL をコピーします。
4. リモートを追加して push します。

```bash
git branch -M main
git remote add origin <https-url>
git push -u origin main
```

<div id="3-ongoing-updates">
  ### 3) 継続的な更新
</div>

```bash
git status
git add .
git commit -m "Update memory"
git push
```

<div id="do-not-commit-secrets">
  ## シークレットをコミットしない
</div>

プライベートリポジトリであっても、ワークスペースにシークレットを保存することは避けてください：

* APIキー、OAuthトークン、パスワード、その他の秘密情報。
* `~/.openclaw/` 配下のあらゆるもの。
* チャットログの未加工ダンプや機微情報を含む添付ファイル。

どうしても機微情報への参照を保存する必要がある場合は、プレースホルダーを使い、
本物のシークレットは別の場所（パスワードマネージャー、環境変数、または `~/.openclaw/`）に保持してください。

推奨される `.gitignore` のひな形:

```gitignore
.DS_Store
.env
**/*.key
**/*.pem
**/secrets*
```

<div id="moving-the-workspace-to-a-new-machine">
  ## ワークスペースを新しいマシンに移動する
</div>

1. リポジトリを目的のパス（デフォルトは `~/.openclaw/workspace`）にクローンします。
2. `~/.openclaw/openclaw.json` 内の `agents.defaults.workspace` をそのパスに設定します。
3. `openclaw setup --workspace <path>` を実行して、不足しているファイルを作成します。
4. セッションが必要な場合は、古いマシンから `~/.openclaw/agents/<agentId>/sessions/` を
   別途コピーします。

<div id="advanced-notes">
  ## 高度な注意事項
</div>

* マルチエージェントルーティングでは、エージェントごとに別々のワークスペースを使用できます。ルーティング設定については
  [チャネルルーティング](/ja/concepts/channel-routing) を参照してください。
* `agents.defaults.sandbox` が有効な場合、メインではないセッションは `agents.defaults.sandbox.workspaceRoot` 配下の、セッションごとのサンドボックス用ワークスペースを使用できます。