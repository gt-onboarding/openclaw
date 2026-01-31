---
title: ストリーミング
summary: "ストリーミングとチャンク処理の動作（ブロック応答、ドラフトストリーミング、各種制限）"
read_when:
  - チャンネルでのストリーミングやチャンク処理の仕組みを説明するとき
  - ブロックストリーミングやチャンネルのチャンク処理の挙動を変更するとき
  - 重複した／早すぎるブロック応答やドラフトストリーミングをデバッグするとき
---

<div id="streaming-chunking">
  # ストリーミングとチャンク化
</div>

OpenClaw には、別々の「ストリーミング」レイヤーが 2 つあります:

- **ブロックストリーミング（チャネル）:** アシスタントが文章を生成する際に、完成した **ブロック** を単位として送信します。これは通常のチャネルメッセージであり（トークンの差分ではありません）。
- **トークン風ストリーミング（Telegram のみ）:** 生成中の部分テキストで **ドラフトバブル** を更新し続け、最後に確定メッセージを送信します。

現時点では、外部チャネルメッセージに対する **本物のトークン単位ストリーミング** はありません。Telegram のドラフトストリーミングだけが、部分的なストリーミングが行われる唯一の手段です。

<div id="block-streaming-channel-messages">
  ## ブロックストリーミング（チャネルメッセージ）
</div>

ブロックストリーミングでは、アシスタントの出力を、利用可能になり次第、比較的粗いチャンク単位で順次送信します。

```
Model output
  └─ text_delta/events
       ├─ (blockStreamingBreak=text_end)
       │    └─ chunker emits blocks as buffer grows
       └─ (blockStreamingBreak=message_end)
            └─ chunker flushes at message_end
                   └─ channel send (block replies)
```

凡例:

* `text_delta/events`: モデルのストリームイベント（ストリーミング非対応モデルではスパースになる場合あり）。
* `chunker`: `EmbeddedBlockChunker` が最小/最大境界値と改行優先度を適用。
* `channel send`: 実際のアウトバウンドメッセージ（ブロック単位の返信）。

**制御項目:**

* `agents.defaults.blockStreamingDefault`: `"on"`/`"off"`（デフォルトは off）。
* チャネルごとの上書き: `*.blockStreaming`（およびアカウント単位のバリアント）でチャネルごとに `"on"`/`"off"` を強制。
* `agents.defaults.blockStreamingBreak`: `"text_end"` または `"message_end"`。
* `agents.defaults.blockStreamingChunk`: `{ minChars, maxChars, breakPreference? }`。
* `agents.defaults.blockStreamingCoalesce`: `{ minChars?, maxChars?, idleMs? }`（送信前にストリームされたブロックをマージ）。
* チャネルのハード上限: `*.textChunkLimit`（例: `channels.whatsapp.textChunkLimit`）。
* チャネルのチャンクモード: `*.chunkMode`（デフォルトは `length`、`newline` は長さによるチャンク分割の前に空行（段落境界）で分割）。
* Discord のソフト上限: `channels.discord.maxLinesPerMessage`（デフォルト 17）により、縦長の返信を分割して UI でのクリップを回避。

**境界セマンティクス:**

* `text_end`: chunker がチャンクを出力したら即座にブロックをストリームし、各 `text_end` ごとにフラッシュする。
* `message_end`: アシスタントメッセージが完了するまで待ち、その後でバッファ済み出力をフラッシュする。

`message_end` でも、バッファ済みテキストが `maxChars` を超える場合には chunker を使用するため、最後に複数チャンクを出力する場合がある。


<div id="chunking-algorithm-lowhigh-bounds">
  ## チャンク化アルゴリズム（下限／上限）
</div>

ブロック単位でのチャンク化は `EmbeddedBlockChunker` によって実装されています:

- **下限:** バッファが `minChars` 以上になるまで（強制されない限り）はチャンクを出力しない。
- **上限:** 可能であれば `maxChars` に達する前に分割し、強制される場合は `maxChars` で分割する。
- **分割の優先度:** `paragraph` → `newline` → `sentence` → `whitespace` → ハードブレイク。
- **コードフェンス:** フェンスの内部では決して分割しない。`maxChars` での分割が避けられない場合は、Markdown の整合性を保つためにフェンスを一度閉じてから開き直す。

`maxChars` はチャネルの `textChunkLimit` を上限として制限されるため、チャネルごとの上限を超えることはできません。

<div id="coalescing-merge-streamed-blocks">
  ## コアレッシング（ストリームされたブロックのマージ）
</div>

ブロックストリーミングが有効な場合、OpenClaw は **連続するブロックチャンクを送出前にまとめてマージ**できます。これにより、「1 行ずつのスパム」を抑えつつ、逐次的な出力は維持されます。

- コアレッシングはフラッシュする前に、**アイドル時間のギャップ**（`idleMs`）が発生するのを待ちます。
- バッファには `maxChars` による上限があり、これを超えるとフラッシュされます。
- `minChars` によって、十分なテキストが蓄積されるまで小さな断片が送信されないようにします
  （最終フラッシュでは残りのテキストが必ず送信されます）。
- 結合方法は `blockStreamingChunk.breakPreference` から決定されます
  （`paragraph` → `\n\n`、`newline` → `\n`、`sentence` → 半角スペース）。
- チャンネルごとのオーバーライドは `*.blockStreamingCoalesce` で指定できます（アカウント単位の設定も含む）。
- 既定のコアレッシングの `minChars` は、上書きされない限り Signal/Slack/Discord では 1500 に引き上げられます。

<div id="human-like-pacing-between-blocks">
  ## 人間らしいブロック間のペース
</div>

ブロックストリーミングが有効な場合、ブロック返信（最初のブロック以降）の間に
**ランダムな一時停止** を挿入できます。これにより、複数バブル表示のレスポンスが
より自然に感じられます。

- 設定: `agents.defaults.humanDelay`（`agents.list[].humanDelay` でエージェントごとに上書き可能）
- モード: `off`（デフォルト）、`natural`（800〜2500ms）、`custom`（`minMs` / `maxMs`）
- **ブロック返信** のみに適用され、最終返信やツールのサマリーには適用されません。

<div id="stream-chunks-or-everything">
  ## 「チャンク単位でストリーミング」か「最後にまとめてストリーミング」か
</div>

これは次のように対応します:

- **チャンクをストリーミング:** `blockStreamingDefault: "on"` + `blockStreamingBreak: "text_end"`（生成しながら順次送出）。Telegram 以外のチャネルでは `*.blockStreaming: true` も必要です。
- **最後にすべてストリーミング:** `blockStreamingBreak: "message_end"`（最後にまとめてフラッシュ。非常に長い場合は複数チャンクになる可能性あり）。
- **ブロックストリーミングなし:** `blockStreamingDefault: "off"`（最終応答のみ）。

**チャネルに関する注意:** Telegram 以外のチャネルでは、`*.blockStreaming` が明示的に `true` に設定されていない限り、ブロックストリーミングは **オフ** です。Telegram はブロック返信なしでドラフト
(`channels.telegram.streamMode`) をストリーミングできます。

設定場所に関する注意: `blockStreaming*` のデフォルトはルート設定ではなく、
`agents.defaults` の下にあります。

<div id="telegram-draft-streaming-token-ish">
  ## Telegram のドラフトストリーミング（トークン風）
</div>

Telegram はドラフトストリーミングをサポートしている唯一のチャネルです。

* **トピック付きのプライベートチャット**で Bot API `sendMessageDraft` を使用します。
* `channels.telegram.streamMode: "partial" | "block" | "off"`。
  * `partial`: 最新のストリームテキストでドラフトを更新します。
  * `block`: チャンク化されたブロック単位でドラフトを更新します（チャンク処理ルールは他のブロックストリーミングと同じ）。
  * `off`: ドラフトストリーミングを行いません。
* ドラフトチャンク設定（`streamMode: "block"` のときのみ有効）: `channels.telegram.draftChunk`（デフォルト: `minChars: 200`, `maxChars: 800`）。
* ドラフトストリーミングはブロックストリーミングとは別物であり、ブロックストリーミングによる返信はデフォルトでは無効で、Telegram 以外のチャネルで `*.blockStreaming: true` にした場合のみ有効になります。
* 最終返信は通常のメッセージとして送信されます。
* `/reasoning stream` は推論内容をドラフトバブルに書き込みます（Telegram のみ）。

ドラフトストリーミングが有効な場合、その返信については二重ストリーミングを避けるために、OpenClaw はブロックストリーミングを無効化します。

```
Telegram (プライベート + トピック)
  └─ sendMessageDraft (下書きバブル)
       ├─ streamMode=partial → 最新テキストを更新
       └─ streamMode=block   → チャンカーが下書きを更新
  └─ 最終返信 → 通常メッセージ
```

凡例:

* `sendMessageDraft`: Telegram の下書きバブル（実際のメッセージではない）。
* `final reply`: 通常の Telegram メッセージ送信。
