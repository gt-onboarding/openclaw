---
title: セッション管理とコンパクション
summary: "詳細解説：セッションストアとトランスクリプト、ライフサイクル、および（自動）コンパクションの内部動作"
read_when:
  - セッションID、トランスクリプトJSONL、または sessions.json のフィールドをデバッグする必要がある
  - 自動コンパクションの挙動を変更するか、「事前コンパクション」用のハウスキーピング処理を追加しようとしている
  - メモリのフラッシュ処理やサイレントなシステムターンを実装したい
---

<div id="session-management-compaction-deep-dive">
  # セッション管理とコンパクション（詳細解説）
</div>

このドキュメントでは、OpenClaw がセッションをエンドツーエンドでどのように管理するかを説明します。

* **セッションルーティング**（受信メッセージが `sessionKey` にどのように対応付けられるか）
* **セッションストア**（`sessions.json`）と、その管理内容
* **トランスクリプトの永続化**（`*.jsonl`）とその構造
* **トランスクリプトの衛生処理**（実行前に行うプロバイダー固有の修正）
* **コンテキスト制限**（コンテキストウィンドウと追跡トークン数の関係）
* **コンパクション**（手動 + 自動コンパクション）と、コンパクション前処理をフックする場所
* **サイレントなハウスキーピング**（ユーザー向け出力を生成すべきでないメモリ書き込みなど）

より高レベルな概要を先に確認したい場合は、次から始めてください:

* [/concepts/session](/ja/concepts/session)
* [/concepts/compaction](/ja/concepts/compaction)
* [/concepts/session-pruning](/ja/concepts/session-pruning)
* [/reference/transcript-hygiene](/ja/reference/transcript-hygiene)

***

<div id="source-of-truth-the-gateway">
  ## 信頼できる情報源: Gateway
</div>

OpenClaw は、セッション状態を管理する単一の **Gateway プロセス** を中心に設計されています。

* UI（macOS アプリ、Web Control UI、TUI）は、セッション一覧とトークン数を Gateway に問い合わせる必要があります。
* リモートモードでは、セッションファイルはリモートホスト上にあり、「ローカルの Mac のファイルを確認」しても、Gateway が使用している状態は反映されません。

***

<div id="two-persistence-layers">
  ## 2 つの永続化レイヤー
</div>

OpenClaw はセッションを 2 つのレイヤーで永続化します:

1. **Session store (`sessions.json`)**
   * キー/値マップ: `sessionKey -> SessionEntry`
   * 小さく、変更可能で、編集（またはエントリの削除）しても安全
   * セッションのメタデータ（現在のセッション ID、最終アクティビティ、トグル、トークンカウンターなど）を追跡

2. **Transcript (`<sessionId>.jsonl`)**
   * 追記専用でツリー構造を持つトランスクリプト（各エントリは `id` と `parentId` を持つ）
   * 実際の会話内容 + ツール呼び出し + コンパクションのサマリーを保存
   * 将来のターンのためにモデルコンテキストを再構築する際に使用

***

<div id="on-disk-locations">
  ## ディスク上の保存場所
</div>

Gateway ホスト上ではエージェントごとに:

* ストア: `~/.openclaw/agents/<agentId>/sessions/sessions.json`
* トランスクリプト: `~/.openclaw/agents/<agentId>/sessions/<sessionId>.jsonl`
  * Telegram のトピックセッション: `.../<sessionId>-topic-<threadId>.jsonl`

OpenClaw はこれらを `src/config/sessions.ts` で解決します。

<div id="session-keys-sessionkey">
  ## Session keys (`sessionKey`)
</div>

`sessionKey` は、あなたが *どの会話バケットに属しているか*（ルーティング + アイソレーション）を識別します。

一般的なパターン例:

* メイン / 直接チャット（エージェントごと）: `agent:<agentId>:<mainKey>`（デフォルトは `main`）
* グループ: `agent:<agentId>:<channel>:group:<id>`
* ルーム / チャンネル（Discord/Slack）: `agent:<agentId>:<channel>:channel:<id>` または `...:room:<id>`
* Cron: `cron:<job.id>`
* Webhook: `hook:<uuid>`（上書きされない限り）

正準的なルールは [/concepts/session](/ja/concepts/session) に記載されています。

***

<div id="session-ids-sessionid">
  ## セッション ID (`sessionId`)
</div>

各 `sessionKey` は現在の `sessionId`（会話が継続されるトランスクリプトファイル）を指します。

運用上の目安:

* **Reset** (`/new`, `/reset`) は、その `sessionKey` に対して新しい `sessionId` を作成します。
* **Daily reset**（デフォルトでは Gateway ホストのローカル時間で午前 4:00）は、リセット境界時刻をまたいだ後、次のメッセージ受信時に新しい `sessionId` を作成します。
* **Idle expiry**（`session.reset.idleMinutes` またはレガシーな `session.idleMinutes`）は、アイドル期間経過後にメッセージが到着したときに新しい `sessionId` を作成します。Daily と idle の両方が設定されている場合は、先に有効期限を迎えた方が適用されます。

実装上の詳細: 判定は `src/auto-reply/reply/session.ts` の `initSessionState()` 内で行われます。

***

<div id="session-store-schema-sessionsjson">
  ## セッションストアスキーマ (`sessions.json`)
</div>

ストアの値の型は `src/config/sessions.ts` の `SessionEntry` です。

主なフィールド（網羅的ではない）:

* `sessionId`: 現在のトランスクリプト ID（`sessionFile` が設定されていない場合、この値からファイル名が決定される）
* `updatedAt`: 直近のアクティビティのタイムスタンプ
* `sessionFile`: 任意指定のトランスクリプトパス（デフォルトのパスを明示的に上書き）
* `chatType`: `direct | group | room`（UI と送信ポリシーの判定に使用）
* `provider`, `subject`, `room`, `space`, `displayName`: グループ／チャネルのラベル付け用メタデータ
* トグル:
  * `thinkingLevel`, `verboseLevel`, `reasoningLevel`, `elevatedLevel`
  * `sendPolicy`（セッション単位での上書き）
* モデル選択:
  * `providerOverride`, `modelOverride`, `authProfileOverride`
* トークンカウンタ（ベストエフォート／プロバイダー依存）:
  * `inputTokens`, `outputTokens`, `totalTokens`, `contextTokens`
* `compactionCount`: このセッションキーに対して自動コンパクションが完了した回数
* `memoryFlushAt`: 直近のコンパクション前メモリフラッシュのタイムスタンプ
* `memoryFlushCompactionCount`: 直近のフラッシュが実行されたときのコンパクション回数

ストアは編集しても安全ですが、最終的な正として扱われるのは Gateway 側です。セッションの実行中にエントリを書き換えたり、再読み込み（リハイドレート）したりする場合があります。

***

<div id="transcript-structure-jsonl">
  ## トランスクリプト構造 (`*.jsonl`)
</div>

トランスクリプトは `@mariozechner/pi-coding-agent` の `SessionManager` によって管理されます。

ファイル形式は JSONL です：

* 1 行目: セッションヘッダー（`type: "session"`、`id`、`cwd`、`timestamp`、任意の `parentSession` を含む）
* それ以降: `id` と `parentId` を持つセッションエントリ（ツリー構造）

主なエントリ種別:

* `message`: user/assistant/toolResult のメッセージ
* `custom_message`: 拡張によって挿入されたメッセージで、モデルコンテキストには *入る* が（UI からは非表示にできる）
* `custom`: モデルコンテキストには *入らない* 拡張の状態
* `compaction`: `firstKeptEntryId` と `tokensBefore` を含む、コンパクション処理の結果として永続化された要約
* `branch_summary`: ツリーのブランチを移動した際に永続化される要約

OpenClaw は意図的にトランスクリプトを**「修正」することはありません**。Gateway は `SessionManager` を使ってトランスクリプトを読み書きします。

***

<div id="context-windows-vs-tracked-tokens">
  ## コンテキストウィンドウと追跡トークンの違い
</div>

重要になる概念は 2 つあります：

1. **モデルのコンテキストウィンドウ**：モデルごとのハード上限（モデルが参照できるトークン）
2. **セッションストアのカウンタ**：`sessions.json` に書き込まれるローリングな統計情報（/status やダッシュボードで利用）

制限値を調整する場合：

* コンテキストウィンドウはモデルカタログに定義されており（設定で上書き可能）ます。
* ストア内の `contextTokens` は実行時の推定／レポート用の値であり、厳密な上限としては扱わないでください。

詳細は [/token-use](/ja/token-use) を参照してください。

***

<div id="compaction-what-it-is">
  ## コンパクション: 何か
</div>

コンパクションは、古い会話内容を要約してトランスクリプト内の永続的な `compaction` エントリとして保存しつつ、直近のメッセージはそのまま保持します。

コンパクションの後、以降のターンでは次のものがコンテキストとして参照されます:

* コンパクションサマリー
* `firstKeptEntryId` 以降のメッセージ

コンパクションは **永続的** です（セッション・プルーニングとは異なります）。[/concepts/session-pruning](/ja/concepts/session-pruning) を参照してください。

***

<div id="when-auto-compaction-happens-pi-runtime">
  ## 自動コンパクションが発生するタイミング（Pi ランタイム）
</div>

組み込みの Pi エージェントでは、自動コンパクションは次の 2 つのケースでトリガーされます。

1. **オーバーフローからの復旧**: モデルがコンテキストオーバーフローエラーを返した場合 → コンパクション → リトライ。
2. **しきい値維持**: 成功したターンの後、次の条件を満たすとき:

`contextTokens > contextWindow - reserveTokens`

ここで:

* `contextWindow` はモデルのコンテキストウィンドウ
* `reserveTokens` はプロンプト + 次のモデル出力のために予約しているヘッドルーム

これらは Pi ランタイムの動作仕様です（OpenClaw はこれらのイベントを取り込みますが、いつコンパクションするかは Pi が決定します）。

***

<div id="compaction-settings-reservetokens-keeprecenttokens">
  ## コンパクション設定 (`reserveTokens`, `keepRecentTokens`)
</div>

Piのコンパクション設定は Pi の設定内にあります。

```json5
{
  compaction: {
    enabled: true,
    reserveTokens: 16384,
    keepRecentTokens: 20000
  }
}
```

OpenClaw は、埋め込み実行に対して安全下限値も強制します:

* `compaction.reserveTokens < reserveTokensFloor` の場合、OpenClaw が値を引き上げます。
* デフォルトの下限値は `20000` トークンです。
* 下限値を無効にするには、`agents.defaults.compaction.reserveTokensFloor: 0` を設定します。
* すでに値がそれより高い場合、OpenClaw はそのままにします。

理由: 圧縮が避けられなくなる前に、マルチターンの「ハウスキーピング」（メモリ書き込みなど）のための十分な余裕を残すためです。

実装: `src/agents/pi-settings.ts` 内の `ensurePiCompactionReserveTokens()`
（`src/agents/pi-embedded-runner.ts` から呼び出されます）。

***

<div id="user-visible-surfaces">
  ## ユーザーに表示される UI
</div>

コンパクションとセッション状態は、次の方法で確認できます：

* `/status`（任意のチャットセッション内）
* `openclaw status`（CLI）
* `openclaw sessions` / `sessions --json`
* 詳細モード: `🧹 Auto-compaction complete` ＋ コンパクション実行回数

***

<div id="silent-housekeeping-no_reply">
  ## サイレント・ハウスキーピング（`NO_REPLY`）
</div>

OpenClaw は、ユーザーに途中経過の出力を見せるべきでないバックグラウンドタスク向けに、「サイレント」なターンをサポートします。

規約:

* アシスタントは、出力を `NO_REPLY` で開始することで、「ユーザーへの返信を配信しない」ことを示します。
* OpenClaw は配信レイヤーでこれを削除・抑止します。

`2026.1.10` 時点では、OpenClaw は部分チャンクが `NO_REPLY` で始まる場合、**draft/typing ストリーミング**も抑止するため、サイレントな処理でターン途中の部分出力が漏れることはありません。

***

<div id="pre-compaction-memory-flush-implemented">
  ## コンパクション前の「メモリフラッシュ」（実装済み）
</div>

目的: 自動コンパクションが走る前に、サイレントなエージェントターンを実行して
永続的な状態をディスクに書き出す（例: エージェントのワークスペース内の `memory/YYYY-MM-DD.md`）
ことで、コンパクションによって重要なコンテキストが消されないようにする。

OpenClaw は **閾値前フラッシュ（pre-threshold flush）** アプローチを採用しています:

1. セッションコンテキストの使用状況を監視する。
2. それが「ソフトスレッショルド」（Pi のコンパクション閾値より低い値）を超えたら、
   エージェントに対してサイレントな「今すぐメモリを書き出せ」という指示を実行する。
3. ユーザーに何も表示されないように `NO_REPLY` を使用する。

設定 (`agents.defaults.compaction.memoryFlush`):

* `enabled`（デフォルト: `true`）
* `softThresholdTokens`（デフォルト: `4000`）
* `prompt`（フラッシュターンで用いるユーザーメッセージ）
* `systemPrompt`（フラッシュターンに付加される追加のシステムプロンプト）

注意事項:

* デフォルトの prompt/systemPrompt には、ユーザーへの表示を抑制するための `NO_REPLY` ヒントが含まれます。
* フラッシュはコンパクションサイクルごとに 1 回実行されます（`sessions.json` で追跡）。
* フラッシュは組み込みの Pi セッションに対してのみ実行されます（CLI バックエンドではスキップされます）。
* セッションのワークスペースが読み取り専用（`workspaceAccess: "ro"` または `"none"`）の場合、フラッシュはスキップされます。
* ワークスペースのファイルレイアウトおよび書き込みパターンについては [Memory](/ja/concepts/memory) を参照してください。

Pi は拡張 API に `session_before_compact` フックも公開していますが、
OpenClaw のフラッシュロジックは現時点では Gateway 側に実装されています。

***

<div id="troubleshooting-checklist">
  ## トラブルシューティング・チェックリスト
</div>

* セッションキーが誤っていないか？まず [/concepts/session](/ja/concepts/session) を確認し、`/status` 内の `sessionKey` を確認する。
* ストアとトランスクリプトの不整合？Gateway ホストと、`openclaw status` に表示されるストアパスを確認する。
* コンパクションが頻発する？次を確認する:
  * モデルのコンテキストウィンドウ（小さすぎないか）
  * コンパクション設定（`reserveTokens` がモデルのウィンドウに対して大きすぎると、早期コンパクションの原因になる）
  * ツール結果の肥大化：セッションのプルーニングを有効化／調整する。
* サイレントターンが漏れている？返信が `NO_REPLY`（このトークンと完全一致）で始まっていること、およびストリーミング抑制の修正を含むビルドを使用していることを確認する。