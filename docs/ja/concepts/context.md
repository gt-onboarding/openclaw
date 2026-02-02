---
title: コンテキスト
summary: "コンテキスト: モデルが何を見ているか、それがどのように構築されるか、そしてどのように確認できるか"
read_when:
  - OpenClawにおける「コンテキスト」の意味を理解したいとき
  - モデルがなぜ何かを「知っている」のか（または忘れたのか）をデバッグしているとき
  - /context、/status、/compact でコンテキストのオーバーヘッドを減らしたいとき
---

<div id="context">
  # コンテキスト
</div>

「コンテキスト」とは、**1 回の実行(run)のために OpenClaw がモデルに送るすべて**を指します。これはモデルの **コンテキストウィンドウ**（トークン上限）によって制限されます。

初心者向けのメンタルモデル：

* **システムプロンプト**（OpenClaw が構築したもの）：ルール、ツール、スキル一覧、現在時刻／ランタイム情報、注入されたワークスペースファイルなど。
* **会話履歴**：このセッションにおける、あなたのメッセージ + アシスタントのメッセージ。
* **ツール呼び出し／結果 + 添付ファイル**：コマンド出力、ファイルの `read`、画像／音声など。

コンテキストは「メモリ」とは*同じものではありません*。メモリはディスクに保存して後から再読み込みできますが、コンテキストはモデルの現在のコンテキストウィンドウ内に収まっている情報です。

<div id="quick-start-inspect-context">
  ## クイックスタート（コンテキストを確認）
</div>

* `/status` → 「ウィンドウがどれくらい埋まっているか」の簡易ビュー＋セッション設定。
* `/context list` → 何がインジェクトされているか＋おおよそのサイズ（ファイルごと＋合計）。
* `/context detail` → さらに詳しい内訳：ファイル単位、ツールごとのスキーマサイズ、スキルエントリごとのサイズ、システムプロンプトのサイズ。
* `/usage tokens` → 通常の返信に、返信ごとのトークン使用量のフッターを付与。
* `/compact` → 古い履歴を要約してコンパクトなエントリにし、ウィンドウの空き容量を確保。

あわせて参照: [Slash commands](/ja/tools/slash-commands)、[トークン使用量とコスト](/ja/token-use)、[コンパクション](/ja/concepts/compaction)。

<div id="example-output">
  ## 出力例
</div>

値は、モデル、プロバイダー、ツールポリシー、そしてワークスペースの中身によって変わります。

<div id="context-list">
  ### `/context list`
</div>

```
🧠 コンテキスト内訳
ワークスペース: <workspaceDir>
Bootstrap max/file: 20,000 chars
サンドボックス: mode=non-main sandboxed=false
System prompt (run): 38,412 chars (~9,603 tok) (Project Context 23,901 chars (~5,976 tok))

Injected workspace files:
- AGENTS.md: OK | raw 1,742 chars (~436 tok) | injected 1,742 chars (~436 tok)
- SOUL.md: OK | raw 912 chars (~228 tok) | injected 912 chars (~228 tok)
- TOOLS.md: TRUNCATED | raw 54,210 chars (~13,553 tok) | injected 20,962 chars (~5,241 tok)
- IDENTITY.md: OK | raw 211 chars (~53 tok) | injected 211 chars (~53 tok)
- USER.md: OK | raw 388 chars (~97 tok) | injected 388 chars (~97 tok)
- HEARTBEAT.md: MISSING | raw 0 | injected 0
- BOOTSTRAP.md: OK | raw 0 chars (~0 tok) | injected 0 chars (~0 tok)

Skills list (system prompt text): 2,184 chars (~546 tok) (12 skills)
Tools: read, edit, write, exec, process, browser, message, sessions_send, …
Tool list (system prompt text): 1,032 chars (~258 tok)
Tool schemas (JSON): 31,988 chars (~7,997 tok) (counts toward context; not shown as text)
Tools: (same as above)

Session tokens (cached): 14,250 total / ctx=32,000
```

<div id="context-detail">
  ### `/context detail`
</div>

```
🧠 Context breakdown (detailed)
…
Top skills (prompt entry size):
- frontend-design: 412 chars (~103 tok)
- oracle: 401 chars (~101 tok)
… (+10 more skills)

Top tools (schema size):
- browser: 9,812 chars (~2,453 tok)
- exec: 6,240 chars (~1,560 tok)
… (+N more tools)
```

<div id="what-counts-toward-the-context-window">
  ## コンテキストウィンドウにカウントされるもの
</div>

モデルが受け取るものはすべてカウントされます。具体的には次のとおりです:

* システムプロンプト（すべてのセクション）。
* 会話履歴。
* ツール呼び出しとツールの結果。
* 添付ファイル/書き起こし（画像/音声/ファイル）。
* 圧縮サマリおよび剪定によるアーティファクト。
* プロバイダーの「ラッパー」や非表示ヘッダー（ユーザーからは見えませんが、カウントされます）。

<div id="how-openclaw-builds-the-system-prompt">
  ## OpenClaw がシステムプロンプトを構築する方法
</div>

システムプロンプトは **OpenClaw が所有する** ものであり、実行のたびに再構築されます。これには次の内容が含まれます:

* ツール一覧 + 短い説明。
* スキル一覧（メタデータのみ。詳細は後述）。
* ワークスペースの場所。
* 時刻（UTC と、設定されていればユーザーのローカル時刻への変換）。
* ランタイムメタデータ（host/OS/model/thinking）。
* **Project Context** 配下に挿入されるワークスペースのブートストラップファイル。

詳細な内訳: [System Prompt](/ja/concepts/system-prompt)。

<div id="injected-workspace-files-project-context">
  ## 注入されるワークスペースファイル（プロジェクトコンテキスト）
</div>

デフォルトでは、OpenClaw は（存在する場合）あらかじめ決まったワークスペースファイル一式を注入します：

* `AGENTS.md`
* `SOUL.md`
* `TOOLS.md`
* `IDENTITY.md`
* `USER.md`
* `HEARTBEAT.md`
* `BOOTSTRAP.md`（初回実行時のみ）

サイズの大きいファイルは、ファイルごとに `agents.defaults.bootstrapMaxChars`（デフォルトは `20000` 文字）を使って切り詰められます。`/context` では、**生（未注入）データと注入後**のサイズ、および切り詰めが行われたかどうかを確認できます。

<div id="skills-whats-injected-vs-loaded-on-demand">
  ## スキル: 事前に埋め込まれるものとオンデマンドで読み込まれるもの
</div>

システムプロンプトには、コンパクトな **スキル一覧**（名前 + 説明 + 配置場所）が含まれます。この一覧にはそれなりのオーバーヘッドがあります。

スキルの指示内容（instructions）はデフォルトでは *含まれません*。モデルは、必要になったときにだけ、そのスキルの `SKILL.md` を `read` することが期待されています。

<div id="tools-there-are-two-costs">
  ## ツール：コストは 2 種類ある
</div>

ツールはコンテキストに 2 通りの影響を与えます：

1. システムプロンプト内の **ツール一覧テキスト**（「Tooling」として表示される部分）。
2. **ツールスキーマ**（JSON）。これはモデルがツールを呼び出せるように送信されるもので、プレーンテキストとしては見えなくてもコンテキストとしてカウントされます。

`/context detail` は、最大のツールスキーマを内訳として表示し、どれがコンテキストを支配しているかを確認できるようにします。

<div id="commands-directives-and-inline-shortcuts">
  ## コマンド、ディレクティブ、および「インラインショートカット」
</div>

スラッシュコマンドは Gateway によって処理されます。挙動にはいくつかの種類があります:

* **スタンドアロンコマンド**: メッセージが `/...` のみの場合、コマンドとして実行されます。
* **ディレクティブ**: `/think`、`/verbose`、`/reasoning`、`/elevated`、`/model`、`/queue` は、モデルがメッセージを受け取る前に取り除かれます。
  * ディレクティブのみのメッセージは、セッション設定として保存されます。
  * 通常のメッセージ内のインラインディレクティブは、そのメッセージに対するヒントとして機能します。
* **インラインショートカット**（許可リストに登録された送信者のみ）: 通常のメッセージ内の特定の `/...` トークンは即座に実行され（例: 「hey /status」）、モデルが残りのテキストを受け取る前に取り除かれます。

詳細: [スラッシュコマンド](/ja/tools/slash-commands)。

<div id="sessions-compaction-and-pruning-what-persists">
  ## セッション、圧縮、プルーニング（何が保持されるか）
</div>

メッセージ間でどの情報が保持されるかは、その仕組みによって異なります。

* **通常の履歴** は、ポリシーによって圧縮／プルーニングされるまで、セッションのトランスクリプトに保持されます。
* **圧縮（Compaction）** は要約をトランスクリプトに永続化し、直近のメッセージはそのまま保持します。
* **プルーニング（Pruning）** は、ある実行用の *インメモリ* プロンプトから古いツール結果を削除しますが、トランスクリプト自体は書き換えません。

ドキュメント: [Session](/ja/concepts/session), [Compaction](/ja/concepts/compaction), [Session pruning](/ja/concepts/session-pruning).

<div id="what-context-actually-reports">
  ## `/context` が実際に報告する内容
</div>

`/context` は、利用可能な場合は最新の **run-built** なシステムプロンプトレポートを優先して使用します:

* `System prompt (run)` = 直近の埋め込み（tool-capable）run の実行時に構築され、キャプチャされてセッションストアに永続化されたもの。
* `System prompt (estimate)` = run レポートが存在しない場合（またはレポートを生成しない CLI バックエンド経由で実行している場合）に、その場で算出されるもの。

いずれの場合も、サイズと主な寄与元を報告しますが、システムプロンプト全体やツールスキーマをダンプすることは**ありません**。