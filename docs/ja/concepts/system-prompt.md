---
title: システムプロンプト
summary: "OpenClaw のシステムプロンプトの内容と、その組み立て方"
read_when:
  - システムプロンプトのテキスト、tools のリスト、time/ハートビートのセクションを編集するとき
  - ワークスペースのブートストラップ処理やスキル注入の動作を変更するとき
---

<div id="system-prompt">
  # システムプロンプト
</div>

OpenClaw は、エージェントの各実行ごとにカスタムのシステムプロンプトを構築します。このプロンプトは **OpenClaw が所有・管理するもの** であり、p-coding-agent のデフォルトプロンプトは使用しません。

このプロンプトは OpenClaw によって組み立てられ、各エージェント実行に挿入されます。

<div id="structure">
  ## 構造
</div>

このプロンプトは意図的にコンパクトで、固定されたセクションで構成されています:

* **Tooling**: 現在のツール一覧と短い説明。
* **Skills** (利用可能な場合): スキル用の指示をオンデマンドで読み込む方法をモデルに伝えます。
* **OpenClaw Self-Update**: `config.apply` と `update.run` の実行方法。
* **Workspace**: 作業ディレクトリ (`agents.defaults.workspace`)。
* **Documentation**: OpenClaw ドキュメントのローカルパス (リポジトリまたは npm パッケージ) と、いつそれらを read すべきか。
* **Workspace Files (injected)**: ブートストラップ用ファイルが以下に含まれていることを示します。
* **Sandbox** (有効な場合): サンドボックス化されたランタイム、サンドボックスのパス、および特権付き exec が利用可能かどうかを示します。
* **Current Date &amp; Time**: ユーザーのローカル時刻、タイムゾーン、時刻フォーマット。
* **Reply Tags**: 対応しているプロバイダー向けの任意指定の返信タグ構文。
* **Heartbeats**: ハートビート用プロンプトと ACK の動作。
* **Runtime**: ホスト、OS、ノード、モデル、リポジトリルート (検出されている場合)、思考レベル (1 行)。
* **Reasoning**: 現在の可視性レベルと `/reasoning` によるトグルのヒント。

<div id="prompt-modes">
  ## プロンプトモード
</div>

OpenClaw はサブエージェント向けに、より短いシステムプロンプトをレンダリングできます。ランタイムは各実行ごとに
`promptMode` を設定します（これはユーザー向けの設定ではありません）:

* `full`（デフォルト）: 上記のすべてのセクションを含みます。
* `minimal`: サブエージェント向けに使用されます。**Skills**、**Memory Recall**、**OpenClaw
  Self-Update**、**Model Aliases**、**User Identity**、**Reply Tags**、
  **Messaging**、**Silent Replies**、**Heartbeats** を省略します。Tooling、Workspace、
  Sandbox、既知であれば現在の日付と時刻、Runtime、およびインジェクトされたコンテキストは
  引き続き利用可能です。
* `none`: ベースのアイデンティティ行のみを返します。

`promptMode=minimal` の場合、追加でインジェクトされるプロンプトは **Group Chat Context** ではなく
**Subagent Context** としてラベル付けされます。

<div id="workspace-bootstrap-injection">
  ## ワークスペースのブートストラップ挿入
</div>

ブートストラップファイルは切り詰められたうえで **Project Context** の下に追記されるため、モデルは明示的な `read` を行わなくても、アイデンティティとプロファイルに関するコンテキストを参照できます:

* `AGENTS.md`
* `SOUL.md`
* `TOOLS.md`
* `IDENTITY.md`
* `USER.md`
* `HEARTBEAT.md`
* `BOOTSTRAP.md`（新規ワークスペースでのみ）

大きなファイルはマーカー付きで切り詰められます。ファイルごとの最大サイズは
`agents.defaults.bootstrapMaxChars`（デフォルト: 20000）で制御されます。存在しないファイルについては、短い欠落ファイル用マーカーが挿入されます。

内部フックは `agent:bootstrap` を介してこのステップをフックし、挿入されるブートストラップファイルを変更または置き換えることができます（例: `SOUL.md` を別のペルソナ用ファイルに差し替えるなど）。

各挿入ファイルがどの程度コンテキストに寄与しているか（元の内容 vs 挿入後、切り詰め量、さらにツールスキーマのオーバーヘッドを含む）を確認するには、`/context list` または `/context detail` を使用してください。詳細は [Context](/ja/concepts/context) を参照してください。

<div id="time-handling">
  ## 時刻の扱い
</div>

ユーザーのタイムゾーンがわかっている場合、System Prompt には専用の **Current Date &amp; Time** セクションが含まれます。プロンプトのキャッシュ安定性を保つため、現在は **time zone** のみを含めるようになっており、動的な時計や時刻フォーマットは含まれません。

エージェントが現在時刻を取得する必要がある場合は `session_status` を使用してください。ステータスカードにはタイムスタンプ行が含まれています。

設定項目:

* `agents.defaults.userTimezone`
* `agents.defaults.timeFormat` (`auto` | `12` | `24`)

完全な動作仕様については [Date &amp; Time](/ja/date-time) を参照してください。

<div id="skills">
  ## スキル
</div>

利用可能なスキルが存在する場合、OpenClaw は各スキルの**ファイルパス**を含む、コンパクトな**利用可能なスキル一覧**
（`formatSkillsForPrompt`）を挿入します。
そのプロンプトは、モデルに対して、記載された場所（ワークスペース、管理対象、またはバンドル済み）にある SKILL.md を読み込むために `read` を使うよう指示します。
利用可能なスキルがない場合は、「スキル」セクションは省略されます。

```
<available_skills>
  <skill>
    <name>...</name>
    <description>...</description>
    <location>...</location>
  </skill>
</available_skills>
```

これにより、ベースプロンプトを小さく保ったまま、必要なスキルだけを選択的に利用できるようになります。

<div id="documentation">
  ## ドキュメント
</div>

利用可能な場合、system prompt には **Documentation** セクションが含まれます。このセクションでは、ローカルの OpenClaw ドキュメントディレクトリ（リポジトリのワークスペース内の `docs/` または同梱の npm パッケージドキュメント）の場所を示すほか、公開ミラー、ソースリポジトリ、コミュニティ Discord、スキル探索のための ClawHub (https://clawhub.com) についても言及します。プロンプトはモデルに対し、OpenClaw の動作、コマンド、設定、アーキテクチャに関しては、まずローカルドキュメントを参照するよう指示し、可能な場合には自分自身で `openclaw status` を実行し、アクセスできない場合にのみユーザーへ尋ねるよう求めます。