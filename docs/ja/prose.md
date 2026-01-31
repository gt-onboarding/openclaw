---
title: プローズ
summary: "OpenProse: OpenClaw における .prose ワークフロー、スラッシュコマンド、そして状態管理"
read_when:
  - .prose ワークフローを実行または作成したい場合
  - OpenProse プラグインを有効にしたい場合
  - 状態ストレージについて理解する必要がある場合
---

<div id="openprose">
  # OpenProse
</div>

OpenProse は、AI セッションをオーケストレーションするための、可搬性が高い Markdown ファーストのワークフロー形式です。OpenClaw では、OpenProse のスキルパックと `/prose` スラッシュコマンドをインストールするプラグインとして提供されています。プログラムは `.prose` ファイル内に定義され、明示的な制御フローを持つ複数のサブエージェントを起動できます。

公式サイト: https://www.prose.md

<div id="what-it-can-do">
  ## できること
</div>

* 明示的な並列実行によるマルチエージェントでのリサーチと統合・要約。
* 繰り返し実行可能で、承認プロセスに対して安全なワークフロー（コードレビュー、インシデントトリアージ、コンテンツパイプライン）。
* 対応しているエージェントランタイム全体で実行できる、再利用可能な `.prose` プログラム。

<div id="install-enable">
  ## インストールと有効化
</div>

バンドルされているプラグインはデフォルトで無効です。OpenProse を有効にするには:

```bash
openclaw plugins enable open-prose
```

プラグインを有効にしたら Gateway を再起動してください。

開発環境／ローカルチェックアウト: `openclaw plugins install ./extensions/open-prose`

関連ドキュメント: [プラグイン](/ja/plugin)、[プラグインマニフェスト](/ja/plugins/manifest)、[スキル](/ja/tools/skills)。

<div id="slash-command">
  ## スラッシュコマンド
</div>

OpenProse は、ユーザーが呼び出せるスキルコマンドとして `/prose` を登録します。これは OpenProse VM のインストラクションに処理を転送し、内部的には OpenClaw のツールを使用します。

よく使われるコマンド:

```
/prose help
/prose run <file.prose>
/prose run <handle/slug>
/prose run <https://example.com/file.prose>
/prose compile <file.prose>
/prose examples
/prose update
```

<div id="example-a-simple-prose-file">
  ## 簡単な `.prose` ファイルの例
</div>

```prose
# Research + synthesis with two agents running in parallel.

input topic: "What should we research?"

agent researcher:
  model: sonnet
  prompt: "You research thoroughly and cite sources."

agent writer:
  model: opus
  prompt: "You write a concise summary."

parallel:
  findings = session: researcher
    prompt: "Research {topic}."
  draft = session: writer
    prompt: "Summarize {topic}."

session "Merge the findings + draft into a final answer."
context: { findings, draft }
```

<div id="file-locations">
  ## ファイルの場所
</div>

OpenProse は、ワークスペース内の `.prose/` ディレクトリ配下に状態を保存します。

```
.prose/
├── .env
├── runs/
│   └── {YYYYMMDD}-{HHMMSS}-{random}/
│       ├── program.prose
│       ├── state.md
│       ├── bindings/
│       └── agents/ # エージェント群
└── agents/ # エージェント群
```

ユーザーレベルの永続エージェントは次のパスに配置されます：

```
~/.prose/agents/
```

<div id="state-modes">
  ## 状態モード
</div>

OpenProse は複数の状態バックエンドをサポートします:

* **filesystem** (デフォルト): `.prose/runs/...`
* **in-context**: 小規模なプログラム向けの一時的な方式
* **sqlite** (実験的): `sqlite3` バイナリが必要
* **postgres** (実験的): `psql` と接続文字列が必要

注意:

* sqlite/postgres はオプトインかつ実験的機能です。
* postgres の認証情報はサブエージェントのログに記録されます。最小権限の専用 DB を使用してください。

<div id="remote-programs">
  ## リモートプログラム
</div>

`/prose run <handle/slug>` は `https://p.prose.md/<handle>/<slug>` に解決されます。
直接指定された URL はそのままフェッチされます。これには `web_fetch` ツール（POST の場合は `exec`）が使用されます。

<div id="openclaw-runtime-mapping">
  ## OpenClaw ランタイムの対応関係
</div>

OpenProse プログラムは OpenClaw のプリミティブに次のようにマッピングされます:

| OpenProse の概念 | 対応する OpenClaw ツール |
| --- | --- |
| セッションの生成 / Task ツール | `sessions_spawn` |
| ファイルの読み取り/書き込み | `read` / `write` |
| Web 取得 | `web_fetch` |

ツールの許可リストでこれらのツールをブロックしている場合、OpenProse プログラムは実行に失敗します。[Skills config](/ja/tools/skills-config) を参照してください。

<div id="security-approvals">
  ## セキュリティと承認
</div>

`.prose` ファイルはコードと同様に扱ってください。実行前に必ずレビューし、副作用を制御するために OpenClaw のツール許可リストと承認ゲートを使用してください。

決定論的で承認ゲート付きのワークフローについては、[Lobster](/ja/tools/lobster) を参照してください。