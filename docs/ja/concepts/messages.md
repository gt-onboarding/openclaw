---
title: メッセージ
summary: "メッセージフロー、セッション、キューイング、および推論の可視性"
read_when:
  - 受信メッセージがどのように返信として処理されるかを説明するとき
  - セッション、キューイングモード、またはストリーミング動作を説明するとき
  - 推論の可視性と利用上の影響を文書化するとき
---

<div id="messages">
  # メッセージ
</div>

このページでは、OpenClaw が受信メッセージ、セッション、キューイング、
ストリーミング、および思考過程（reasoning）の可視化をどのように扱うかを整理して説明します。

<div id="message-flow-high-level">
  ## メッセージフローの概要
</div>

```
Inbound message
  -> routing/bindings -> session key
  -> queue (if a run is active)
  -> agent run (streaming + tools)
  -> outbound replies (channel limits + chunking)
```

主要な調整項目は設定で行います：

* 接頭辞、キューイング、グループ動作には `messages.*`。
* ブロックストリーミングとチャンク分割のデフォルトには `agents.defaults.*`。
* 上限値とストリーミングのオン/オフ切り替えにはチャネルごとのオーバーライド（`channels.whatsapp.*`、`channels.telegram.*` など）。

完全なスキーマについては [Configuration](/ja/gateway/configuration) を参照してください。

<div id="inbound-dedupe">
  ## 受信メッセージの重複排除
</div>

チャネルによっては、再接続後に同じメッセージが再送信されることがあります。OpenClaw は
channel/account/peer/session/message id をキーにした短期間のみ有効なキャッシュを保持し、
重複した配送によってエージェントが再実行されないようにします。

<div id="inbound-debouncing">
  ## 受信デバウンス
</div>

**同じ送信者** からの短時間に連続して送信されたメッセージは、`messages.inbound` を通じて 1 回の
エージェントターンにまとめて処理できます。デバウンスのスコープはチャンネル＋会話ごとで、
返信スレッド／ID には最新のメッセージが使われます。

設定（グローバルデフォルト＋チャンネル単位の上書き）:

```json5
{
  messages: {
    inbound: {
      debounceMs: 2000,
      byChannel: {
        whatsapp: 5000,
        slack: 1500,
        discord: 1500
      }
    }
  }
}
```

Notes:

* デバウンス処理は**テキストのみ**のメッセージに適用され、メディアや添付ファイルは即座に送信されます。
* 制御コマンドはデバウンス処理の対象外となるため、常に単独のメッセージとして扱われます。

<div id="sessions-and-devices">
  ## セッションとデバイス
</div>

セッションはクライアントではなく Gateway によって管理・所有されます。

* 1 対 1 のチャットは、エージェントのメインセッションキーに集約されます。
* グループ／チャンネルは、それぞれ独自のセッションキーを持ちます。
* セッションストアとトランスクリプトは Gateway ホスト上に配置されます。

複数のデバイス／チャンネルを同じセッションにマッピングすることはできますが、履歴はすべてのクライアントに完全には同期されません。コンテキストが分岐するのを避けるため、長い会話には 1 台のメインデバイスを使用することを推奨します。Control UI と TUI は常に Gateway 管理下のセッショントランスクリプトを表示するため、それらが信頼すべき唯一の情報源となります。

詳細: [セッション管理](/ja/concepts/session)

<div id="inbound-bodies-and-history-context">
  ## 受信ボディと履歴コンテキスト
</div>

OpenClaw は **プロンプトボディ** と **コマンドボディ** を分離します:

* `Body`: エージェントに送信されるプロンプトテキスト。チャネルのエンベロープや
  オプションの履歴ラッパーを含む場合があります。
* `CommandBody`: ディレクティブ／コマンド解析のための生のユーザーテキスト。
* `RawBody`: `CommandBody` のレガシー別名（互換性維持のために残されています）。

チャネルが履歴を提供する場合、共通のラッパーを使用します:

* `[Chat messages since your last reply - for context]`
* `[Current message - respond to this]`

**非ダイレクトチャット**（グループ／チャネル／ルーム）の場合、**現在のメッセージボディ** には
送信者ラベルがプレフィックスとして付与されます（履歴エントリと同じスタイル）。これにより、
リアルタイムメッセージとキューイングされた／履歴メッセージがエージェントのプロンプト内で
一貫した形になります。

履歴バッファには **保留中のものだけ** が含まれます。つまり、実行をトリガーしなかった
グループメッセージ（例: メンション必須のメッセージ）を含み、すでに
セッションのトランスクリプトに含まれているメッセージは **除外** されます。

ディレクティブの除去は **現在のメッセージ** セクションにのみ適用されるため、
履歴はそのまま保持されます。履歴をラップするチャネルは、`CommandBody`
（または `RawBody`）を元のメッセージテキストに設定し、`Body` を結合済みの
プロンプトとして保持する必要があります。履歴バッファは `messages.groupChat.historyLimit`
（グローバルデフォルト）および `channels.slack.historyLimit` や
`channels.telegram.accounts.<id>.historyLimit` のようなチャネル単位のオーバーライドで
設定できます（無効にするには `0` を設定します）。

<div id="queueing-and-followups">
  ## キューイングとフォローアップ
</div>

すでにアクティブな run がある場合、受信メッセージはキューに入れるか、現在の run に取り込むか、あるいはフォローアップ用として次のターンに回すために収集できます。

* `messages.queue`（および `messages.queue.byChannel`）で設定します。
* モード：`interrupt`、`steer`、`followup`、`collect` に加え、バックログ向けのバリアントがあります。

詳細: [キューイング](/ja/concepts/queue)。

<div id="streaming-chunking-and-batching">
  ## ストリーミング、チャンク分割、バッチ処理
</div>

ブロックストリーミングは、モデルがテキストブロックを生成するのにあわせて部分的な応答を送信します。
チャンク分割はチャネルのテキスト上限を考慮し、フェンス付きコードブロックを分割しないようにします。

主な設定項目:

* `agents.defaults.blockStreamingDefault` (`on|off`、デフォルトは off)
* `agents.defaults.blockStreamingBreak` (`text_end|message_end`)
* `agents.defaults.blockStreamingChunk` (`minChars|maxChars|breakPreference`)
* `agents.defaults.blockStreamingCoalesce` (アイドル状態に基づくバッチ処理)
* `agents.defaults.humanDelay` (ブロック応答間の人間らしい待ち時間)
* チャネルごとのオーバーライド: `*.blockStreaming` および `*.blockStreamingCoalesce` (Telegram 以外のチャネルでは `*.blockStreaming: true` を明示的に指定する必要があります)

詳細: [ストリーミング + チャンク分割](/ja/concepts/streaming)。

<div id="reasoning-visibility-and-tokens">
  ## 推論の可視性とトークン
</div>

OpenClaw はモデルの推論を表示するか非表示にするかを制御できます:

* `/reasoning on|off|stream` で可視性を制御します。
* モデルが生成した推論コンテンツも、トークン使用量として引き続きカウントされます。
* Telegram はドラフトバブル内への推論のストリーミングをサポートしています。

詳細: [思考 + 推論ディレクティブ](/ja/tools/thinking) および [トークン使用](/ja/token-use)。

<div id="prefixes-threading-and-replies">
  ## プレフィックス、スレッド、返信
</div>

送信メッセージのフォーマットは `messages` セクションで一元管理されます:

* `messages.responsePrefix`（送信プレフィックス）と `channels.whatsapp.messagePrefix`（WhatsApp 受信プレフィックス）
* `replyToMode` とチャネル別のデフォルトによる返信スレッド処理

詳細については、[Configuration](/ja/gateway/configuration#messages) と各チャネルのドキュメントを参照してください。