---
title: Lobster
summary: "OpenClaw 向けの型付きワークフロー実行環境（再開可能な承認ゲート付き）。"
description: OpenClaw 向けの型付きワークフロー実行環境 — 承認ゲートを備えたコンポーザブルなパイプライン。
read_when:
  - 明示的な承認付きの決定論的なマルチステップワークフローが必要な場合
  - 前のステップを再実行せずにワークフローを再開する必要がある場合
---

<div id="lobster">
  # Lobster
</div>

Lobster は、OpenClaw が複数ステップからなるツールシーケンスを、明示的な承認チェックポイント付きの単一の決定的な処理として実行できるようにするワークフローシェルです。

<div id="hook">
  ## Hook
</div>

あなたのアシスタントは、自身を管理するためのツールを自分で構築できます。ワークフローを依頼すれば、30分後には、1回の呼び出しで動く CLI とパイプライン一式が手に入ります。Lobster はそこに欠けていたピースです: 決定論的なパイプライン、明示的な承認、そして再開可能な状態。

<div id="why">
  ## なぜ
</div>

現在、複雑なワークフローには、ツール呼び出しのやり取りが何度も発生します。各呼び出しにはトークンコストがかかり、LLM がすべてのステップをオーケストレーションしなければなりません。Lobster は、そのオーケストレーションを型付きランタイム側に移します。

* **多数の呼び出しではなく 1 回の呼び出しで済む**: OpenClaw は 1 回の Lobster ツール呼び出しを実行し、構造化された結果を受け取ります。
* **承認が組み込まれている**: 副作用（メールの送信、コメントの投稿）は、明示的に承認されるまでワークフローを停止します。
* **再開可能**: 停止したワークフローはトークンを返します。承認すれば、すべてを再実行することなく再開できます。

<div id="why-a-dsl-instead-of-plain-programs">
  ## なぜ通常のプログラムではなく DSL なのか？
</div>

Lobster は意図的に小さく作られています。目標は「新しい言語」を作ることではなく、予測可能で AI に適した、承認と再開トークンを第一級の概念として備えたパイプライン仕様を提供することです。

* **承認と再開が組み込み**: 通常のプログラムでも人間に確認を促すことはできますが、あなた自身でそのランタイムを作り込まない限り、永続的なトークン付きで*一時停止して再開する*ことはできません。
* **決定性 + 監査可能性**: パイプラインはデータなので、ログ化、差分比較、リプレイ、レビューが容易です。
* **AI が扱う領域を制約**: 小さな文法と JSON パイプ処理により、「創造的」なコードパスを減らし、検証可能な範囲に収めます。
* **セーフティポリシーが組み込み**: タイムアウト、出力上限、サンドボックス検査、許可リストなどが、各スクリプトではなくランタイムによって強制されます。
* **それでもプログラマブル**: 各ステップは任意の CLI やスクリプトを呼び出せます。JS/TS を使いたい場合は、コードから `.lobster` ファイルを生成してください。

<div id="how-it-works">
  ## 動作の仕組み
</div>

OpenClaw はローカルの `lobster` CLI を **ツールモード** で起動し、stdout の出力から JSON エンベロープをパースします。
パイプラインが承認待ちで一時停止すると、ツールは後で処理を再開できるように `resumeToken` を返します。

<div id="pattern-small-cli-json-pipes-approvals">
  ## パターン: 小さな CLI + JSON パイプ + 承認
</div>

JSON を扱う小さなコマンドを作り、それらをつないで 1 回の Lobster 呼び出しにまとめます。（以下のコマンド名は例なので、自分のものに差し替えて構いません。）

```bash
inbox list --json
inbox categorize --json
inbox apply --json
```

```json
{
  "action": "run",
  "pipeline": "exec --json --shell 'inbox list --json' | exec --stdin json --shell 'inbox categorize --json' | exec --stdin json --shell 'inbox apply --json' | approve --preview-from-stdin --limit 5 --prompt 'Apply changes?'",
  "timeoutMs": 30000
}
```

パイプラインが承認を要求してきた場合は、次のトークンを使って再開してください：

```json
{
  "action": "resume",
  "token": "<resumeToken>",
  "approve": true
}
```

AI がワークフローを開始し、Lobster が各ステップを実行します。承認ゲートによって、副作用は常に明示的で監査可能なものとして扱われます。

例: 入力アイテムをツール呼び出しに対応付ける:

```bash
gog.gmail.search --query 'newer_than:1d' \
  | openclaw.invoke --tool message --action send --each --item-key message --args-json '{"provider":"telegram","to":"..."}'
```

<div id="json-only-llm-steps-llm-task">
  ## JSON 専用の LLM ステップ (llm-task)
</div>

**構造化された LLM ステップ** が必要なワークフローの場合は、オプションの
`llm-task` プラグインツールを有効にして、Lobster から呼び出します。これにより、
モデルを使って分類・要約・ドラフト生成を行いつつ、ワークフローを決定論的に維持できます。

ツールを有効にします:

```json
{
  "plugins": {
    "entries": {
      "llm-task": { "enabled": true }
    }
  },
  "agents": {
    "list": [
      {
        "id": "main",
        "tools": { "allow": ["llm-task"] }
      }
    ]
  }
}
```

パイプラインで使用する:

```lobster
openclaw.invoke --tool llm-task --action json --args-json '{
  "prompt": "Given the input email, return intent and draft.",
  "input": { "subject": "Hello", "body": "Can you help?" },
  "schema": {
    "type": "object",
    "properties": {
      "intent": { "type": "string" },
      "draft": { "type": "string" }
    },
    "required": ["intent", "draft"],
    "additionalProperties": false
  }
}'
```

詳細と設定オプションについては、[LLM Task](/ja/tools/llm-task) を参照してください。

<div id="workflow-files-lobster">
  ## ワークフローファイル (.lobster)
</div>

Lobster は、`name`、`args`、`steps`、`env`、`condition`、`approval` フィールドを持つ YAML/JSON 形式のワークフローファイルを実行できます。OpenClaw のツール呼び出しで使用する場合は、`pipeline` にそのファイルパスを指定します。

```yaml
name: inbox-triage
args:
  tag:
    default: "family"
steps:
  - id: collect
    command: inbox list --json
  - id: categorize
    command: inbox categorize --json
    stdin: $collect.stdout
  - id: approve
    command: inbox apply --approve
    stdin: $categorize.stdout
    approval: required
  - id: execute
    command: inbox apply --execute
    stdin: $categorize.stdout
    condition: $approve.approved
```

注記:

* `stdin: $step.stdout` と `stdin: $step.json` は、前のステップの出力を渡します。
* `condition`（または `when`）は、`$step.approved` に基づいてステップを実行するかどうかを制御できます。

<div id="install-lobster">
  ## Lobster のインストール
</div>

[Lobster repo](https://github.com/openclaw/lobster) を参照して、OpenClaw Gateway を実行している **同じホスト** に Lobster CLI をインストールし、`lobster` が `PATH` に通っていることを確認してください。
バイナリを任意の場所に配置して使用したい場合は、ツール呼び出しで **絶対パス** の `lobsterPath` を指定してください。

<div id="enable-the-tool">
  ## ツールを有効にする
</div>

Lobster は **オプション** のプラグインツールです（デフォルトでは有効になっていません）。

推奨（追加しても安全）：

```json
{
  "tools": {
    "alsoAllow": ["lobster"]
  }
}
```

または、エージェント単位で:

```json
{
  "agents": {
    "list": [
      {
        "id": "main",
        "tools": {
          "alsoAllow": ["lobster"]
        }
      }
    ]
  }
}
```

制限的な許可リストモードで実行するつもりがない限り、`tools.allow: ["lobster"]` の使用は避けてください。

注意: 許可リストはオプションのプラグインに対しては明示的に有効化した場合にのみ適用されます。許可リストに
プラグインのツール（`lobster` など）だけを指定した場合でも、OpenClaw はコアツールを引き続き有効にします。コアツールを制限したい場合は、許可リストに対象とするコアツールやコアツールのグループも含めてください。

<div id="example-email-triage">
  ## 例：メールのトリアージ
</div>

Lobster なしの場合：

```
ユーザー: "メールを確認して返信の下書きを作成して"
→ openclawがgmail.listを呼び出す
→ LLMが要約する
→ ユーザー: "#2と#5への返信の下書きを作成して"
→ LLMが下書きを作成する
→ ユーザー: "#2を送信して"
→ openclawがgmail.sendを呼び出す
(毎日繰り返し、トリアージされた内容の記憶なし)
```

Lobster を使う場合は:

```json
{
  "action": "run",
  "pipeline": "email.triage --limit 20",
  "timeoutMs": 30000
}
```

（一部のみを示した）JSON エンベロープを返します:

```json
{
  "ok": true,
  "status": "needs_approval",
  "output": [{ "summary": "5 need replies, 2 need action" }],
  "requiresApproval": {
    "type": "approval_request",
    "prompt": "下書き返信2件を送信しますか?",
    "items": [],
    "resumeToken": "..."
  }
}
```

ユーザーが承認すると再開:

```json
{
  "action": "resume",
  "token": "<resumeToken>",
  "approve": true
}
```

単一のワークフロー。決定論的。安全。

<div id="tool-parameters">
  ## ツールのパラメータ
</div>

<div id="run">
  ### `run`
</div>

ツールモードでパイプラインを実行します。

```json
{
  "action": "run",
  "pipeline": "gog.gmail.search --query 'newer_than:1d' | email.triage",
  "cwd": "/path/to/workspace",
  "timeoutMs": 30000,
  "maxStdoutBytes": 512000
}
```

引数付きでワークフローファイルを実行:

```json
{
  "action": "run",
  "pipeline": "/path/to/inbox-triage.lobster",
  "argsJson": "{\"tag\":\"family\"}"
}
```

<div id="resume">
  ### `resume`
</div>

承認後に一時停止していたワークフローを再開します。

```json
{
  "action": "resume",
  "token": "<resumeToken>",
  "approve": true
}
```

<div id="optional-inputs">
  ### オプション入力
</div>

* `lobsterPath`: Lobster バイナリへの絶対パス（省略時は `PATH` を使用）。
* `cwd`: パイプラインの作業ディレクトリ（デフォルトは現在のプロセスの作業ディレクトリ）。
* `timeoutMs`: この時間を超えた場合にサブプロセスを強制終了する（デフォルト: 20000）。
* `maxStdoutBytes`: stdout がこのサイズを超えた場合にサブプロセスを強制終了する（デフォルト: 512000）。
* `argsJson`: `lobster run --args-json` に渡される JSON 文字列（ワークフローファイルでのみ使用）。

<div id="output-envelope">
  ## 出力エンベロープ
</div>

Lobster は、次のいずれか 3 つのステータスを持つ JSON エンベロープを返します：

* `ok` → 正常に完了
* `needs_approval` → 一時停止中。再開するには `requiresApproval.resumeToken` が必要
* `cancelled` → 明示的に拒否されたか、キャンセル済み

このツールは、そのエンベロープを `content`（整形済み JSON）と `details`（生のオブジェクト）の両方として提供します。

<div id="approvals">
  ## 承認
</div>

`requiresApproval` がある場合は、プロンプトを確認して次のように判断します:

* `approve: true` → 再開して副作用処理を継続する
* `approve: false` → ワークフローをキャンセルして確定する

`approve --preview-from-stdin --limit N` を使用すると、カスタムの jq/heredoc で自前の「つなぎ」を書かなくても、承認リクエストに JSON プレビューを添付できます。再開トークンは現在コンパクトになっており、Lobster はワークフローの再開状態を自身の state ディレクトリ配下に保存し、小さなトークンキーだけを返します。

<div id="openprose">
  ## OpenProse
</div>

OpenProse は Lobster と組み合わせて使うと効果的です。`/prose` を使ってマルチエージェントの準備処理をオーケストレーションし、その後に Lobster パイプラインを実行して決定的な承認フローを実行してください。Prose プログラムが Lobster を必要とする場合は、`tools.subagents.tools` でサブエージェントに `lobster` ツールの利用を許可します。[OpenProse](/ja/prose) も参照してください。

<div id="safety">
  ## セキュリティ
</div>

* **ローカルサブプロセスのみ** — プラグイン自体から外部ネットワークへの呼び出しは行いません。
* **シークレットなし** — Lobster は OAuth を管理せず、その処理は対応する OpenClaw のツールに委ねます。
* **サンドボックス対応** — ツールコンテキストがサンドボックス化されている場合は無効化されます。
* **堅牢化** — `lobsterPath` を指定する場合は絶対パスでなければならず、タイムアウトと出力上限が強制されます。

<div id="troubleshooting">
  ## トラブルシューティング
</div>

* **`lobster subprocess timed out`** → `timeoutMs` の値を増やすか、長いパイプラインを分割します。
* **`lobster output exceeded maxStdoutBytes`** → `maxStdoutBytes` を引き上げるか、出力サイズを減らします。
* **`lobster returned invalid JSON`** → パイプラインがツールモードで実行され、JSON だけを出力していることを確認します。
* **`lobster failed (code …)`** → 同じパイプラインをターミナルで実行して、標準エラー出力（stderr）を確認します。

<div id="learn-more">
  ## さらに詳しく
</div>

* [プラグイン](/ja/plugin)
* [プラグインツールの作成方法](/ja/plugins/agent-tools)

<div id="case-study-community-workflows">
  ## ケーススタディ: コミュニティワークフロー
</div>

公開されている例として、「セカンドブレイン」向け CLI と Lobster パイプラインを使い、3 つの Markdown ボルト（個人用、パートナー用、共有）を管理するものがあります。CLI は統計情報、インボックス一覧、古い項目のスキャンのための JSON を出力し、Lobster はそれらのコマンドを `weekly-review`、`inbox-triage`、`memory-consolidation`、`shared-task-sync` といったワークフローに連結し、それぞれに承認ゲートを設けます。AI が利用可能な場合は判断（カテゴリ分類）を任せ、利用できない場合はルールベースの決定的なロジックにフォールバックします。

* スレッド: https://x.com/plattenschieber/status/2014508656335770033
* リポジトリ: https://github.com/bloomedai/brain-cli