---
title: Cron ジョブ
summary: "Cron ジョブ + Gateway スケジューラ用のウェイクアップ"
read_when:
  - バックグラウンド ジョブやウェイクアップをスケジューリングするとき
  - ハートビートと連動して、またはハートビートと並行して実行する自動化を設定するとき
  - スケジュールされたタスクにハートビートと Cron のどちらを使うか決めるとき
---

<div id="cron-jobs-gateway-scheduler">
  # Cron ジョブ (Gateway スケジューラ)
</div>

> **Cron とハートビートの違いは？** それぞれをいつ使うべきかは、[Cron vs Heartbeat](/ja/automation/cron-vs-heartbeat) を参照してください。

Cron は Gateway に組み込まれたスケジューラです。ジョブを保持し、適切なタイミングでエージェントを起動し、必要に応じて出力をチャットに返すことができます。

*「毎朝これを実行する」* や *「20 分後にエージェントをつつく」* といったことを行いたい場合、cron がそのための仕組みです。

<div id="tldr">
  ## 要約
</div>

* Cron は **Gateway の内部で**動作します（モデル内では動作しません）。
* ジョブは `~/.openclaw/cron/` に永続化されるため、再起動してもスケジュールは失われません。
* 実行スタイルは 2 種類です:
  * **メインセッション**: システムイベントをキューに入れ、次のハートビートで実行します。
  * **分離実行**: `cron:<jobId>` 内で専用のエージェントのターンを実行し、必要に応じて出力を配信します。
* ウェイクアップは第一級の機能として扱われます: ジョブは「今すぐ起こす」か「次のハートビートまで待つか」を指定できます。

<div id="beginner-friendly-overview">
  ## 初心者向けの概要
</div>

cron ジョブは「**いつ** 実行するか」+「**何を** するか」と考えると分かりやすいです。

1. **スケジュールを決める**
   * 一度きりのリマインダー → `schedule.kind = "at"` (CLI: `--at`)
   * 繰り返しジョブ → `schedule.kind = "every"` または `schedule.kind = "cron"`
   * ISO タイムスタンプにタイムゾーンが含まれていない場合は、**UTC** として扱われます。

2. **どこで実行するかを決める**
   * `sessionTarget: "main"` → 次回のハートビート時に main セッションのコンテキストで実行します。
   * `sessionTarget: "isolated"` → `cron:<jobId>` 内で専用のエージェントのターンを実行します。

3. **ペイロードの内容を決める**
   * メインセッション → `payload.kind = "systemEvent"`
   * 分離セッション → `payload.kind = "agentTurn"`

オプション：`deleteAfterRun: true` を指定すると、正常終了した一度きりのジョブはストアから削除されます。

<div id="concepts">
  ## 基本概念
</div>

<div id="jobs">
  ### ジョブ
</div>

cron ジョブは、次の要素を持つレコードです:

* **schedule**（いつ実行するか）
* **payload**（何を行うか）
* 任意の **delivery**（出力をどこへ送るか）
* 任意の **agent binding**（`agentId`）：特定のエージェントのコンテキストでジョブを実行する。\
  指定がない場合や不明な場合は、Gateway はデフォルトのエージェントにフォールバックする。

ジョブは、不変の `jobId`（CLI / Gateway API で使用）によって識別されます。
エージェントのツール呼び出しでは、`jobId` が正準的な識別子です。互換性のためにレガシーな `id` も受け付けられます。
ジョブは、1 回限りの実行が成功した後に `deleteAfterRun: true` を指定することで自動削除させることもできます。

<div id="schedules">
  ### スケジュール
</div>

Cron は 3 種類のスケジュール種別をサポートします:

* `at`: 1 回限りのタイムスタンプ（エポックからのミリ秒値）。Gateway は ISO 8601 を受け付け、UTC に変換します。
* `every`: 固定間隔（ミリ秒）。
* `cron`: オプションの IANA タイムゾーン付き、5 フィールドからなる cron 式。

Cron 式の評価には `croner` を使用します。タイムゾーンが省略された場合、Gateway ホストのローカルタイムゾーンが使用されます。

<div id="main-vs-isolated-execution">
  ### メイン実行と分離実行
</div>

<div id="main-session-jobs-system-events">
  #### メインセッションジョブ（システムイベント）
</div>

メインジョブはシステムイベントをキューに入れ、必要に応じてハートビートランナーを起動します。
`payload.kind = "systemEvent"` を必ず設定してください。

* `wakeMode: "next-heartbeat"`（デフォルト）：イベントは次にスケジュールされたハートビートまで待機します。
* `wakeMode: "now"`：イベントがただちにハートビートの実行をトリガーします。

通常のハートビート用プロンプトとメインセッションのコンテキストを使いたい場合に最適です。
[Heartbeat](/ja/gateway/heartbeat) を参照してください。

<div id="isolated-jobs-dedicated-cron-sessions">
  #### 分離ジョブ（専用の cron セッション）
</div>

分離ジョブは、セッション `cron:<jobId>` 内で専用のエージェントのターン処理を実行します。

主な動作:

* トレース可能にするため、プロンプトの先頭に `[cron:<jobId> <job name>]` が付与されます。
* 各実行は **新しいセッション ID** から開始されます（それ以前の会話は引き継がれません）。
* 要約がメインセッションに投稿されます（接頭辞は `Cron`、設定可能）。
* `wakeMode: "now"` は、要約投稿後に即時のハートビートをトリガーします。
* `payload.deliver: true` の場合、出力はチャネルに配信され、それ以外の場合は内部に留まります。

分離ジョブは、ノイズが多い処理、高頻度の処理、またはメインチャット履歴をスパムさせたくない
「バックグラウンドの雑用」的な処理に使用してください。

<div id="payload-shapes-what-runs">
  ### ペイロードの形式（何が実行されるか）
</div>

2 種類のペイロードがサポートされています:

* `systemEvent`: メインセッション専用で、ハートビートプロンプト経由でルーティングされます。
* `agentTurn`: 分離セッション専用で、専用のエージェントターンを実行します。

共通の `agentTurn` フィールド:

* `message`: 必須のテキストプロンプト。
* `model` / `thinking`: 任意の上書き（後述）。
* `timeoutSeconds`: 任意のタイムアウト上書き。
* `deliver`: 出力をチャネルのターゲットへ送信する場合は `true`。
* `channel`: `last` または特定のチャネル。
* `to`: チャネル固有のターゲット（電話／チャット／チャネル ID）。
* `bestEffortDeliver`: 配信に失敗してもジョブ全体を失敗扱いにしないようにします。

分離セッション用オプション（`session=isolated` の場合のみ）:

* `postToMainPrefix` (CLI: `--post-prefix`): メイン側の system event のプレフィックス。
* `postToMainMode`: `summary`（デフォルト）または `full`。
* `postToMainMaxChars`: `postToMainMode=full` のときの最大文字数（デフォルト 8000）。

<div id="model-and-thinking-overrides">
  ### モデルと思考レベルのオーバーライド
</div>

分離されたジョブ（`agentTurn`）では、モデルと思考レベルをオーバーライドできます:

* `model`: プロバイダー/モデル文字列（例: `anthropic/claude-sonnet-4-20250514`）またはエイリアス（例: `opus`）
* `thinking`: 思考レベル（`off`, `minimal`, `low`, `medium`, `high`, `xhigh`。GPT-5.2 + Codex モデルのみ）

注: メインセッションのジョブでも `model` を設定できますが、その場合は共有メイン
セッションのモデルが変更されます。予期しないコンテキストシフトを避けるため、
モデルのオーバーライドは分離されたジョブに限定することを推奨します。

解決順序の優先度:

1. ジョブのペイロードでのオーバーライド（最優先）
2. フック固有のデフォルト（例: `hooks.gmail.model`）
3. エージェント設定のデフォルト

<div id="delivery-channel-target">
  ### Delivery (channel + target)
</div>

独立したジョブは、出力をチャネルに配信できます。ジョブのペイロードでは次を指定できます:

* `channel`: `whatsapp` / `telegram` / `discord` / `slack` / `mattermost` (plugin) / `signal` / `imessage` / `last`
* `to`: チャネル固有の宛先ターゲット

`channel` または `to` が省略された場合、cron はメインセッションの「last route」
（エージェントが最後に返信した場所）にフォールバックします。

配信に関する注意事項:

* `to` が設定されている場合、`deliver` が省略されていても、cron はエージェントの最終出力を自動的に配信します。
* 明示的な `to` なしで last route への配信を行いたい場合は、`deliver: true` を使用します。
* `to` があっても出力を内部に留めたい場合は、`deliver: false` を使用します。

ターゲット形式に関する注意事項:

* Slack/Discord/Mattermost (plugin) のターゲットは、あいまいさを避けるために、`channel:<id>`、`user:<id>` のような明示的なプレフィックスを使用する必要があります。
* Telegram のトピックは `:topic:` 形式を使用してください（後述）。

<div id="telegram-delivery-targets-topics-forum-threads">
  #### Telegram の配信ターゲット（トピック / フォーラムスレッド）
</div>

Telegram は `message_thread_id` によるフォーラムのトピックをサポートします。cron 配信では、
トピック / スレッドを `to` フィールドにエンコードして指定できます:

* `-1001234567890`（チャット ID のみ）
* `-1001234567890:topic:123`（推奨: 明示的なトピックマーカー）
* `-1001234567890:123`（短縮形: 数値サフィックス）

`telegram:...` / `telegram:group:...` のようなプレフィックス付きターゲットも利用できます:

* `telegram:group:-1001234567890:topic:123`

<div id="storage-history">
  ## ストレージと履歴
</div>

* ジョブストア: `~/.openclaw/cron/jobs.json`（Gateway 管理の JSON）。
* 実行履歴: `~/.openclaw/cron/runs/<jobId>.jsonl`（JSONL。古いエントリは自動的に削除される）。
* ストアパスの上書き: 設定の `cron.store`。

<div id="configuration">
  ## 設定
</div>

```json5
{
  cron: {
    enabled: true, // default true
    store: "~/.openclaw/cron/jobs.json",
    maxConcurrentRuns: 1 // デフォルトは 1
  }
}
```

cron を完全に無効にする:

* `cron.enabled: false` (設定)
* `OPENCLAW_SKIP_CRON=1` (環境変数)

<div id="cli-quickstart">
  ## CLI クイックスタート
</div>

ワンショットのリマインダー（UTC ISO 形式、成功後に自動削除）：

```bash
openclaw cron add \
  --name "Send reminder" \
  --at "2026-01-12T18:00:00Z" \
  --session main \
  --system-event "Reminder: submit expense report." \
  --wake now \
  --delete-after-run
```

ワンショット リマインダー（メインセッション、即時起動）：

```bash
openclaw cron add \
  --name "Calendar check" \
  --at "20m" \
  --session main \
  --system-event "Next heartbeat: check calendar." \
  --wake now
```

定期実行の独立ジョブ（WhatsApp への配信）：

```bash
openclaw cron add \
  --name "Morning status" \
  --cron "0 7 * * *" \
  --tz "America/Los_Angeles" \
  --session isolated \
  --message "Summarize inbox + calendar for today." \
  --deliver \
  --channel whatsapp \
  --to "+15551234567"
```

定期実行の単独ジョブ（Telegram のトピックへの配信）:

```bash
openclaw cron add \
  --name "Nightly summary (topic)" \
  --cron "0 22 * * *" \
  --tz "America/Los_Angeles" \
  --session isolated \
  --message "Summarize today; send to the nightly topic." \
  --deliver \
  --channel telegram \
  --to "-1001234567890:topic:123"
```

model と thinking を上書きする単体ジョブ:

````bash
openclaw cron add \
  --name "Deep analysis" \
  --cron "0 6 * * 1" \
  --tz "America/Los_Angeles" \
  --session isolated \
  --message "Weekly deep analysis of project progress." \
  --model "opus" \
  --thinking high \
  --deliver \
  --channel whatsapp \
  --to "+15551234567"

Agent selection (multi-agent setups):
```bash
# ジョブをエージェント "ops" に固定(該当エージェントが存在しない場合はデフォルトにフォールバック)
openclaw cron add --name "Ops sweep" --cron "0 6 * * *" --session isolated --message "Check ops queue" --agent ops

# Switch or clear the agent on an existing job
openclaw cron edit <jobId> --agent ops
openclaw cron edit <jobId> --clear-agent
````

````

手動実行（デバッグ用）:
```bash
openclaw cron run <jobId> --force
````

既存ジョブの編集（フィールドのパッチ更新）:

```bash
openclaw cron edit <jobId> \
  --message "Updated prompt" \
  --model "opus" \
  --thinking low
```

実行履歴：

```bash
openclaw cron runs --id <jobId> --limit 50
```

ジョブを作成せずに即時発火するシステムイベント:

```bash
openclaw system event --mode now --text "Next heartbeat: check battery."
```

<div id="gateway-api-surface">
  ## Gateway API インターフェース
</div>

* `cron.list`, `cron.status`, `cron.add`, `cron.update`, `cron.remove`
* `cron.run`（強制実行または期限到来時）、`cron.runs`

ジョブを作成せずに即時のシステムイベントを発火させたい場合は、[`openclaw system event`](/ja/cli/system) を使用します。

<div id="troubleshooting">
  ## トラブルシューティング
</div>

<div id="nothing-runs">
  ### 「何も動かない」
</div>

* `cron` が有効か確認する: `cron.enabled` と `OPENCLAW_SKIP_CRON` をチェックする。
* Gateway が常時稼働しているか確認する（cron は Gateway プロセス内で実行される）。
* `cron` スケジュールについて: タイムゾーン（`--tz`）がホスト側のタイムゾーンと一致しているか確認する。

<div id="telegram-delivers-to-the-wrong-place">
  ### Telegram が誤った場所に送信される
</div>

* フォーラムのトピックには、あいまいさがなく明確になるように `-100…:topic:<id>` を使用してください。
* ログや保存された「last route」のターゲットに `telegram:...` プレフィックスが表示されても正常です。
  cron 経由の配信ではそれらをそのまま受け取りつつ、トピック ID を正しく解析します。