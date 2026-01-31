---
title: エージェントループ
summary: "エージェントループのライフサイクル、ストリーム、および待機セマンティクス"
read_when:
  - エージェントループやライフサイクルイベントの正確なウォークスルーが必要なとき
---

<div id="agent-loop-openclaw">
  # エージェントループ (OpenClaw)
</div>

エージェントループとは、エージェントが実際に動作するときの 1 回分の完全な実行フローのことです: 入力受付 → コンテキスト組み立て → モデル推論 → ツール実行 → 応答ストリーミング → 永続化。メッセージをアクションと最終応答へと変換しつつ、セッション状態を一貫して保つための正規の処理パスです。

OpenClaw において、ループは 1 セッションあたり 1 回の、直列に実行されるランであり、モデルが思考し、ツールを呼び出し、出力をストリーミングするのに合わせてライフサイクルイベントとストリームイベントを発行します。このドキュメントでは、その実際のループがどのようにエンドツーエンドで構成されているかを説明します。

<div id="entry-points">
  ## エントリーポイント
</div>

* Gateway RPC: `agent` および `agent.wait`。
* CLI: `agent` コマンド。

<div id="how-it-works-high-level">
  ## 仕組み（ハイレベル）
</div>

1. `agent` RPC がパラメータを検証し、セッション（sessionKey/sessionId）を解決し、セッションメタデータを永続化して、直ちに `{ runId, acceptedAt }` を返します。
2. `agentCommand` がエージェントを実行します:
   * モデルと thinking/verbose のデフォルト値を解決する
   * スキルのスナップショットをロードする
   * `runEmbeddedPiAgent`（pi-agent-core ランタイム）を呼び出す
   * 埋め込みループが **lifecycle end/error** を発行しなかった場合に **lifecycle end/error** を発行する
3. `runEmbeddedPiAgent`:
   * セッションごと + グローバルキューによって実行を直列化する
   * モデルと認証/認可プロファイルを解決し、pi セッションを構築する
   * pi イベントを購読し、assistant/tool の差分をストリームする
   * タイムアウトを強制し、超過した場合は実行を中止する
   * ペイロード + 使用状況メタデータを返す
4. `subscribeEmbeddedPiSession` は pi-agent-core のイベントを OpenClaw の `agent` ストリームにブリッジします:
   * tool イベント =&gt; `stream: "tool"`
   * assistant の差分 =&gt; `stream: "assistant"`
   * lifecycle イベント =&gt; `stream: "lifecycle"`（`phase: "start" | "end" | "error"`）
5. `agent.wait` は `waitForAgentJob` を使用します:
   * `runId` の **lifecycle end/error** を待機する
   * `{ status: ok|error|timeout, startedAt, endedAt, error? }` を返す

<div id="queueing-concurrency">
  ## キューイングと並行実行
</div>

* 実行はセッションキー（session lane）ごとに、必要に応じてグローバルレーンを通して直列化されます。
* これにより、ツール／セッション間の競合状態が防がれ、セッション履歴の一貫性が保たれます。
* メッセージングチャネルは、このレーンシステムに流し込むキューモード（collect/steer/followup）を選択できます。
  [Command Queue](/ja/concepts/queue) を参照してください。

<div id="session-workspace-preparation">
  ## セッションとワークスペースの準備
</div>

* ワークスペースが特定・作成され、サンドボックス実行時にはサンドボックス用ワークスペースルートへリダイレクトされる場合があります。
* スキルが読み込まれ（またはスナップショットから再利用され）、env とプロンプトに注入されます。
* ブートストラップ／コンテキストファイルが解決され、システムプロンプトレポートに注入されます。
* セッションの書き込みロックが取得され、ストリーミング前に `SessionManager` がオープンされて準備されます。

<div id="prompt-assembly-system-prompt">
  ## プロンプトの組み立て + システムプロンプト
</div>

* システムプロンプトは、OpenClaw のベースプロンプト、スキル用プロンプト、ブートストラップコンテキスト、および実行ごとのオーバーライドから構成されます。
* モデルごとの制限と、プロンプト圧縮用の予約トークンが適用されます。
* モデルが実際に参照する内容については、[システムプロンプト](/ja/concepts/system-prompt) を参照してください。

<div id="hook-points-where-you-can-intercept">
  ## フックポイント（処理を差し込める場所）
</div>

OpenClaw には 2 つのフックシステムがあります。

* **内部フック**（Gateway フック）：コマンドおよびライフサイクルイベント用のイベント駆動スクリプト。
* **プラグインフック**：エージェント／ツールのライフサイクルおよび Gateway パイプライン内における拡張ポイント。

<div id="internal-hooks-gateway-hooks">
  ### 内部フック（Gateway フック）
</div>

* **`agent:bootstrap`**: システムプロンプトが確定する前、ブートストラップファイルを構築している間に実行されます。
  これを使ってブートストラップ用のコンテキストファイルを追加／削除します。
* **コマンドフック**: `/new`、`/reset`、`/stop`、およびその他のコマンドイベント（Hooks ドキュメントを参照）。

セットアップ方法と例については [Hooks](/ja/hooks) を参照してください。

<div id="plugin-hooks-agent-gateway-lifecycle">
  ### プラグインフック（エージェント + Gateway ライフサイクル）
</div>

これらはエージェントループまたは Gateway パイプライン内で実行されます:

* **`before_agent_start`**: 実行が開始する前にコンテキストを注入したり、system プロンプトを上書きしたりします。
* **`agent_end`**: 完了後に最終メッセージ一覧と実行メタデータを検査します。
* **`before_compaction` / `after_compaction`**: コンパクションサイクルを監視または注釈付けします。
* **`before_tool_call` / `after_tool_call`**: ツールのパラメータ／結果をフックしてインターセプトします。
* **`tool_result_persist`**: ツール結果がセッションのトランスクリプトに書き込まれる前に同期的に変換します。
* **`message_received` / `message_sending` / `message_sent`**: 受信／送信方向のメッセージ用フックです。
* **`session_start` / `session_end`**: セッションのライフサイクル境界です。
* **`gateway_start` / `gateway_stop`**: Gateway のライフサイクルイベントです。

フックの API と登録方法については [プラグイン](/ja/plugin#plugin-hooks) を参照してください。

<div id="streaming-partial-replies">
  ## ストリーミングと部分的な応答
</div>

* Assistant のデルタは pi-agent-core からストリーミングされ、`assistant` イベントとして送出されます。
* ブロックストリーミングでは、`text_end` または `message_end` のタイミングで部分的な応答を出力できます。
* 推論ストリーミングは、別のストリームとして、またはブロック応答として出力できます。
* チャンク化とブロック応答の動作については、[ストリーミング](/ja/concepts/streaming) を参照してください。

<div id="tool-execution-messaging-tools">
  ## ツール実行とメッセージングツール
</div>

* ツールの開始／更新／終了イベントは `tool` ストリームで発行されます。
* ツール結果は、ログ出力／送信前にサイズおよび画像ペイロードを対象としてサニタイズされます。
* メッセージングツールによる送信は、アシスタントによる重複した確認を抑制するために追跡されます。

<div id="reply-shaping-suppression">
  ## 応答の整形と抑制
</div>

* 最終ペイロードは次の要素から組み立てられます:
  * アシスタントテキスト（および任意の reasoning）
  * インラインツールサマリー（詳細表示が有効かつ許可されている場合）
  * モデルがエラーになった場合のアシスタントエラーテキスト
* `NO_REPLY` はサイレントなトークンとして扱われ、送信ペイロードから除外されます。
* Messaging ツールの重複は最終ペイロードのリストから削除されます。
* レンダリング可能なペイロードが残らず、かつツールがエラーになった場合は、
  （Messaging ツールがすでにユーザー向けの返信を送っていないかぎり）
  フォールバックのツールエラー応答が返されます。

<div id="compaction-retries">
  ## コンパクションとリトライ
</div>

* 自動コンパクションは `compaction` ストリームイベントを発行し、リトライをトリガーする場合があります。
* リトライ時には、重複した出力を避けるためにメモリ内バッファとツール要約がリセットされます。
* コンパクションパイプラインについては [Compaction](/ja/concepts/compaction) を参照してください。

<div id="event-streams-today">
  ## イベントストリーム（現行）
</div>

* `lifecycle`: `subscribeEmbeddedPiSession` によって発行される（フォールバックとして `agentCommand` によっても発行）
* `assistant`: pi-agent-core からストリーミングされる差分データ
* `tool`: pi-agent-core からストリーミングされるツールイベント

<div id="chat-channel-handling">
  ## チャットチャネルの処理
</div>

* アシスタントのデルタはチャットの `delta` メッセージとしてバッファリングされます。
* チャットの `final` は **ライフサイクルの終了時またはエラー発生時** に発行されます。

<div id="timeouts">
  ## タイムアウト
</div>

* `agent.wait` のデフォルト: 30s（待ち時間のみ）。`timeoutMs` パラメータで上書き可能。
* エージェントのランタイム: `agents.defaults.timeoutSeconds` のデフォルトは 600s。`runEmbeddedPiAgent` の中の中断タイマーで強制される。

<div id="where-things-can-end-early">
  ## 早期終了し得るポイント
</div>

* エージェントのタイムアウト（中断）
* AbortSignal（キャンセル）
* Gateway の切断または RPC のタイムアウト
* `agent.wait` のタイムアウト（待機のみで、エージェントは停止しない）