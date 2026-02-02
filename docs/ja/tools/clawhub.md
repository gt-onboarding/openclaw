---
title: Clawhub
summary: "ClawHub ガイド: 公開スキルレジストリ + CLI ワークフロー"
read_when:
  - 新規ユーザーに ClawHub を紹介するとき
  - スキルをインストール・検索・公開するとき
  - ClawHub CLI のフラグと同期動作を説明するとき
---

<div id="clawhub">
  # ClawHub
</div>

ClawHub は **OpenClaw 向けの公開スキルレジストリ** です。無料のサービスであり、すべてのスキルはパブリックで、共有や再利用のために誰でも閲覧できます。スキルは `SKILL.md` ファイル（および関連するテキストファイル）を含む 1 つのフォルダに過ぎません。スキルは Web アプリから閲覧できるほか、CLI を使って検索、インストール、更新、公開を行うこともできます。

Site: [clawhub.com](https://clawhub.com)

<div id="who-this-is-for-beginner-friendly">
  ## 対象読者（初心者向け）
</div>

OpenClaw のエージェントに新しい機能を追加したい場合、ClawHub はスキルを見つけてインストールするための最も簡単な方法です。バックエンドがどのように動作しているかを知る必要はありません。次のことができます。

* 自然な言葉でスキルを検索する。
* スキルを自分のワークスペースにインストールする。
* 後から 1 つのコマンドでスキルを更新する。
* 自分のスキルを公開してバックアップとして保存する。

<div id="quick-start-non-technical">
  ## クイックスタート（非技術者向け）
</div>

1. CLI をインストールします（次のセクションを参照）。
2. 必要なものを検索します：
   * `clawhub search "calendar"`
3. スキルをインストールします：
   * `clawhub install <skill-slug>`
4. 新しいスキルを読み込むために、新しい OpenClaw セッションを開始します。

<div id="install-the-cli">
  ## CLI をインストールする
</div>

次のいずれかを選んでください:

```bash
npm i -g clawhub
```

```bash
pnpm add -g clawhub
```

<div id="how-it-fits-into-openclaw">
  ## OpenClaw における位置づけ
</div>

デフォルトでは、CLI は現在の作業ディレクトリ配下の `./skills` にスキルをインストールします。OpenClaw のワークスペースが設定されている場合、`clawhub` は `--workdir`（または `CLAWHUB_WORKDIR`）で明示的に指定しない限り、そのワークスペースをインストール先として使用します。OpenClaw は `<workspace>/skills` からワークスペースのスキルを読み込み、**次回**のセッションでそれらを認識します。すでに `~/.openclaw/skills` やバンドルされているスキルを使用している場合でも、ワークスペースのスキルが優先されます。

スキルの読み込み方法、共有方法、およびアクセス制御方法の詳細については、
[Skills](/ja/tools/skills) を参照してください。

<div id="what-the-service-provides-features">
  ## サービスが提供するもの（機能）
</div>

* スキルおよびその `SKILL.md` コンテンツの**公開閲覧**。
* キーワード検索だけでなく、埋め込みベクトル（ベクター検索）による**検索**。
* semver、変更履歴（changelog）、タグ（`latest` を含む）による**バージョニング**。
* バージョンごとの zip 形式での**ダウンロード**。
* コミュニティからのフィードバックのための**スターとコメント**。
* 承認および監査のための**モデレーションフック**。
* 自動化およびスクリプト実行のための、**CLI から扱いやすい API**。

<div id="cli-commands-and-parameters">
  ## CLI コマンドとパラメーター
</div>

グローバルオプション（すべてのコマンドに適用）:

* `--workdir <dir>`: 作業ディレクトリ（デフォルト: 現在のディレクトリ。指定がない場合は OpenClaw のワークスペースを使用）。
* `--dir <dir>`: スキルディレクトリ。workdir からの相対パス（デフォルト: `skills`）。
* `--site <url>`: サイトのベース URL（ブラウザログイン用）。
* `--registry <url>`: Registry API のベース URL。
* `--no-input`: プロンプトを無効化（非対話モード）。
* `-V, --cli-version`: CLI のバージョンを表示。

認証:

* `clawhub login`（ブラウザフロー）または `clawhub login --token <token>`
* `clawhub logout`
* `clawhub whoami`

オプション:

* `--token <token>`: API トークンを貼り付ける。
* `--label <label>`: ブラウザログインのトークンに保存されるラベル（デフォルト: `CLI token`）。
* `--no-browser`: ブラウザを開かない（`--token` が必須）。

検索:

* `clawhub search "query"`
* `--limit <n>`: 最大件数。

インストール:

* `clawhub install <slug>`
* `--version <version>`: 特定バージョンをインストール。
* `--force`: ディレクトリがすでに存在する場合に上書き。

更新:

* `clawhub update <slug>`
* `clawhub update --all`
* `--version <version>`: 特定バージョンに更新（単一の slug のみ）。
* `--force`: ローカルファイルがいずれの公開バージョンとも一致しない場合に上書き。

一覧:

* `clawhub list`（`.clawhub/lock.json` を読み込む）

公開:

* `clawhub publish <path>`
* `--slug <slug>`: スキルの slug。
* `--name <name>`: 表示名。
* `--version <version>`: SemVer 形式のバージョン。
* `--changelog <text>`: 変更履歴テキスト（空でも可）。
* `--tags <tags>`: カンマ区切りのタグ（デフォルト: `latest`）。

削除/削除取り消し（オーナー/管理者のみ）:

* `clawhub delete <slug> --yes`
* `clawhub undelete <slug> --yes`

同期（ローカルスキルをスキャン + 新規/更新分を公開）:

* `clawhub sync`
* `--root <dir...>`: 追加のスキャンルートディレクトリ。
* `--all`: すべてをプロンプトなしでアップロード。
* `--dry-run`: アップロード対象を表示のみ。
* `--bump <type>`: 更新時の `patch|minor|major`（デフォルト: `patch`）。
* `--changelog <text>`: 非対話更新用の変更履歴。
* `--tags <tags>`: カンマ区切りのタグ（デフォルト: `latest`）。
* `--concurrency <n>`: Registry チェックの並列数（デフォルト: 4）。

<div id="common-workflows-for-agents">
  ## エージェントに関する一般的なワークフロー
</div>

<div id="search-for-skills">
  ### スキルを検索
</div>

```bash
clawhub search "postgres backups"
```

<div id="download-new-skills">
  ### 新しいスキルをダウンロード
</div>

```bash
clawhub install my-skill-pack
```

<div id="update-installed-skills">
  ### インストール済みのスキルを更新する
</div>

```bash
clawhub update --all
```

<div id="back-up-your-skills-publish-or-sync">
  ### スキルをバックアップする（公開または同期）
</div>

1つのスキルフォルダの場合:

```bash
clawhub publish ./my-skill --slug my-skill --name "My Skill" --version 1.0.0 --tags latest
```

多くのスキルをまとめてスキャンしてバックアップするには：

```bash
clawhub sync --all
```

<div id="advanced-details-technical">
  ## 高度な技術的な詳細
</div>

<div id="versioning-and-tags">
  ### バージョン管理とタグ
</div>

* 各 `publish` によって新しい **semver** 形式の `SkillVersion` が作成されます。
* タグ（`latest` など）は特定のバージョンを指し、タグを付け替えることでロールバックできます。
* 変更ログはバージョンごとに関連付けられ、同期や更新を公開する際には空でもかまいません。

<div id="local-changes-vs-registry-versions">
  ### ローカルの変更とレジストリ上のバージョンの比較
</div>

アップデート時には、コンテンツハッシュを使ってローカルのスキル内容とレジストリ上のバージョンを比較します。ローカルファイルが公開済みのいずれのバージョンとも一致しない場合、CLI は上書き前に確認を求めるか、非対話モードの実行では `--force` オプションが必要になります。

<div id="sync-scanning-and-fallback-roots">
  ### 同期スキャンとフォールバックルート
</div>

`clawhub sync` は、まず現在の作業ディレクトリをスキャンします。スキルが見つからない場合は、既知のレガシーなパス（例: `~/openclaw/skills` や `~/.openclaw/skills`）にフォールバックします。これは、追加のフラグを付けずに、以前にインストールしたスキルを検出できるように設計されています。

<div id="storage-and-lockfile">
  ### ストレージとロックファイル
</div>

* インストール済みのスキルは、作業ディレクトリ配下の `.clawhub/lock.json` に記録されます。
* 認証トークンは ClawHub CLI の設定ファイルに保存されます（`CLAWHUB_CONFIG_PATH` で変更できます）。

<div id="telemetry-install-counts">
  ### テレメトリ（インストール数）
</div>

ログインした状態で `clawhub sync` を実行すると、CLI はインストール数を集計するための最小限のスナップショットを送信します。これは完全に無効化することもできます：

```bash
export CLAWHUB_DISABLE_TELEMETRY=1
```

<div id="environment-variables">
  ## 環境変数
</div>

* `CLAWHUB_SITE`: サイト URL を上書きします。
* `CLAWHUB_REGISTRY`: レジストリ API の URL を上書きします。
* `CLAWHUB_CONFIG_PATH`: CLI がトークン／設定ファイルを保存するパスを上書きします。
* `CLAWHUB_WORKDIR`: デフォルトの作業ディレクトリを上書きします。
* `CLAWHUB_DISABLE_TELEMETRY=1`: `sync` 実行時のテレメトリ送信を無効にします。