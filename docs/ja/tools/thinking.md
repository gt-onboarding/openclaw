---
title: 思考
summary: "/think および /verbose のディレクティブ構文と、それらがモデルの推論プロセスに与える影響"
read_when:
  - /think または /verbose ディレクティブの構文解析やデフォルト設定を調整するとき
---

<div id="thinking-levels-think-directives">
  # 思考レベル（/think ディレクティブ）
</div>

<div id="what-it-does">
  ## 動作概要
</div>

* 任意の受信メッセージ本文内でのインラインディレクティブ: `/t <level>`、`/think:<level>`、または `/thinking <level>`。
* レベル（エイリアス）: `off | minimal | low | medium | high | xhigh`（GPT-5.2 + Codex モデルのみ）
  * minimal → 「think」
  * low → 「think hard」
  * medium → 「think harder」
  * high → 「ultrathink」（最大予算）
  * xhigh → 「ultrathink+」（GPT-5.2 + Codex モデルのみ）
  * `highest`、`max` は `high` にマップされる。
* プロバイダーに関する注意:
  * Z.AI（`zai/*`）は二値の thinking（`on`/`off`）のみをサポートする。`off` 以外の任意のレベルは `on` として扱われ（`low` にマップされる）。

<div id="resolution-order">
  ## 優先順位
</div>

1. メッセージ内のインラインディレクティブ（そのメッセージにのみ適用）。
2. セッションのオーバーライド（ディレクティブのみのメッセージを送信して設定）。
3. グローバルデフォルト（設定内の `agents.defaults.thinkingDefault`）。
4. フォールバック：推論対応モデルでは low、それ以外では off。

<div id="setting-a-session-default">
  ## セッションのデフォルトを設定する
</div>

* ディレクティブ**だけ**を含むメッセージを送信します（空白は可）。例: `/think:medium` や `/t high`。
* それが現在のセッションに対して有効になります（デフォルトでは送信者単位）。`/think:off` またはセッションのアイドル状態によるリセットで解除されます。
* 確認の返信が送信されます（`Thinking level set to high.` / `Thinking disabled.`）。レベルが不正な場合（例: `/thinking big`）、コマンドはヒント付きで拒否され、セッション状態は変更されません。
* 現在の思考レベルを確認するには、引数なしで `/think`（または `/think:`）を送信します。

<div id="application-by-agent">
  ## エージェントごとの適用
</div>

* **Embedded Pi**: 確定したレベルは、インプロセスの Pi エージェントランタイムに渡されます。

<div id="verbose-directives-verbose-or-v">
  ## 冗長モードディレクティブ（/verbose または /v）
</div>

* レベル: `on`（最小） | `full` | `off`（デフォルト）。
* ディレクティブだけを含むメッセージは、そのセッションの冗長モードを切り替え、`Verbose logging enabled.` / `Verbose logging disabled.` と返信する。無効なレベルの場合は状態を変更せずにヒントを返す。
* `/verbose off` は、明示的なセッションの上書き設定を保存する。Sessions UI で `inherit` を選択するとクリアできる。
* インラインディレクティブはそのメッセージにのみ影響し、それ以外はセッション/グローバルのデフォルトが適用される。
* 現在の冗長レベルを確認するには、引数なしで `/verbose`（または `/verbose:`）を送信する。
* 冗長モードが `on` の場合、構造化されたツール結果を出力するエージェント（Pi やその他の JSON エージェント）は、各ツール呼び出しをメタデータのみの個別メッセージとして送り返す。利用可能な場合（パス/コマンド）、`<emoji> <tool-name>: <arg>` という接頭辞が付く。これらのツール要約は、各ツールが開始されるたびにすぐに（別バブルで）送信され、ストリーミングの差分にはならない。
* 冗長モードが `full` の場合、ツールの出力も完了後に転送される（別バブルで、安全な長さまで切り詰められる）。実行中に `/verbose on|full|off` を切り替えた場合、その後のツールバブルには新しい設定が適用される。

<div id="reasoning-visibility-reasoning">
  ## 推論の表示レベル (/reasoning)
</div>

* レベル: `on|off|stream`。
* ディレクティブ専用メッセージで、思考ブロックを返信に表示するかどうかを切り替えます。
* 有効にすると、推論は `Reasoning:` というプレフィックス付きの**別メッセージ**として送信されます。
* `stream`（Telegram 限定）: 返信を生成している間、Telegram の下書きバブルに推論をストリーミングし、その後、推論なしの最終回答を送信します。
* エイリアス: `/reason`。
* 引数なしで `/reasoning`（または `/reasoning:`）を送信すると、現在の推論レベルを確認できます。

<div id="related">
  ## 関連項目
</div>

* Elevated モードについては、[Elevated mode](/ja/tools/elevated) のドキュメントを参照してください。

<div id="heartbeats">
  ## ハートビート
</div>

* ハートビートのプローブ本文には、設定されたハートビート用プロンプトが使用されます（デフォルト: `Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.`）。ハートビートメッセージ内のインラインディレクティブは通常どおり適用されます（ただし、ハートビート経由でセッションのデフォルトを変更することは避けてください）。
* ハートビートの送信は、デフォルトでは最終ペイロードのみです。別個の `Reasoning:` メッセージ（利用可能な場合）も送信するには、`agents.defaults.heartbeat.includeReasoning: true` を設定するか、エージェントごとに `agents.list[].heartbeat.includeReasoning: true` を設定します。

<div id="web-chat-ui">
  ## Web chat UI
</div>

* Web chat の思考レベルセレクターは、ページ読み込み時に、受信セッションストア／設定に保存されているセッションレベルを反映します。
* 別のレベルを選択すると、それは次のメッセージにのみ適用されます（`thinkingOnce`）。送信後、セレクターは保存されているセッションレベルに戻ります。
* セッションのデフォルトを変更するには、（これまで通り）`/think:&lt;level&gt;` ディレクティブを送信します。次回の再読み込み後にセレクターがその内容を反映します。