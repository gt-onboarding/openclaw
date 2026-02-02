---
title: ハートビート
summary: "ハートビートのポーリングメッセージと通知ルール"
read_when:
  - ハートビートの実行間隔やメッセージ内容を調整するとき
  - スケジュールされたタスクにハートビートと cron のどちらを使うか決めるとき
---

<div id="heartbeat-gateway">
  # ハートビート (Gateway)
</div>

> **ハートビートと Cron の違いは？** それぞれをいつ使うべきかについては、[Cron vs Heartbeat](/ja/automation/cron-vs-heartbeat) を参照してください。

ハートビートはメインセッションで**定期的にエージェントのターンを実行**し、大量の通知であなたを煩わせることなく、注意が必要な事項をモデルが拾い上げられるようにします。

<div id="quick-start-beginner">
  ## クイックスタート（初心者向け）
</div>

1. ハートビートを有効のままにしておく（デフォルトは `30m`、Anthropic OAuth/setup-token の場合は `1h`）か、任意の間隔を設定します。
2. エージェントのワークスペースに小さな `HEARTBEAT.md` チェックリストを作成します（任意ですが推奨）。
3. ハートビートメッセージの送信先を決めます（デフォルトは `target: "last"`）。
4. 任意: 透明性のため、ハートビートの推論内容の配信を有効化します。
5. 任意: ハートビートを活動時間帯（ローカル時間）のみに制限します。

設定例:

```json5
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m",
        target: "last",
        // activeHours: { start: "08:00", end: "24:00" },
        // includeReasoning: true, // オプション: 別途 `Reasoning:` メッセージも送信する
      }
    }
  }
}
```

<div id="defaults">
  ## デフォルト
</div>

* 間隔: `30m`（または、認証モードとして Anthropic OAuth/setup-token が検出された場合は `1h`）。`agents.defaults.heartbeat.every` またはエージェント単位の `agents.list[].heartbeat.every` を設定します。無効化するには `0m` を使用します。
* プロンプト本文（`agents.defaults.heartbeat.prompt` で設定可能）:
  `存在する場合は HEARTBEAT.md を read してください（ワークスペースコンテキスト）。その内容に厳密に従ってください。以前のチャットから古いタスクを推測したり繰り返したりしないでください。対応が不要な場合は HEARTBEAT_OK と返信してください。`
* ハートビートプロンプトは、ユーザーメッセージとして **そのまま** 送信されます。システム
  プロンプトには「Heartbeat」セクションが含まれ、この実行は内部的にフラグ付けされます。
* アクティブ時間帯（`heartbeat.activeHours`）は、設定されたタイムゾーンでチェックされます。
  この時間帯の外では、次に時間帯内となるタイミングまでハートビートはスキップされます。

<div id="what-the-heartbeat-prompt-is-for">
  ## ハートビート用プロンプトの目的
</div>

デフォルトのプロンプトは、あえて広めの内容になっています。

* **バックグラウンドタスク**: “Consider outstanding tasks” は、エージェントに
  未処理のフォローアップ（受信箱、カレンダー、リマインダー、キューされた作業）を見直し、
  緊急度の高いものを洗い出すよう促します。
* **人間側の状況確認**: “Checkup sometimes on your human during day time” は、
  時おり軽い「何か必要なものはありますか？」というメッセージを送るよう促しますが、
  設定済みのローカルタイムゾーン（[/concepts/timezone](/ja/concepts/timezone) を参照）を用いることで、
  夜間のスパム送信は避けます。

ハートビートにごく特定の処理（例: “check Gmail PubSub
stats” や “verify gateway health”）だけを行わせたい場合は、
`agents.defaults.heartbeat.prompt`（または
`agents.list[].heartbeat.prompt`）を、（そのまま送信される）
カスタム本文に設定してください。

<div id="response-contract">
  ## レスポンス仕様
</div>

* 特別な対応が不要な場合は、**`HEARTBEAT_OK`** のみを返信してください。
* ハートビート実行中、OpenClaw は返信の **先頭または末尾** に現れる `HEARTBEAT_OK` を ACK（確認応答）として扱います。このトークンは削除され、残りの内容が **`ackMaxChars` 以下**（デフォルト: 300）の場合、その返信は破棄されます。
* `HEARTBEAT_OK` が返信の**途中**に現れる場合は、特別な扱いはされません。
* アラート時は、**`HEARTBEAT_OK` を含めないでください**。アラート文のみを返します。

ハートビート以外では、メッセージの先頭／末尾にある余分な `HEARTBEAT_OK` は削除されてログに記録されます。内容が `HEARTBEAT_OK` のみのメッセージは破棄されます。

<div id="config">
  ## 設定
</div>

```json5
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m",           // default: 30m (0m disables)
        model: "anthropic/claude-opus-4-5",
        includeReasoning: false, // default: false (deliver separate Reasoning: message when available)
        target: "last",         // last | none | <channel id> (core or plugin, e.g. "bluebubbles")
        to: "+15551234567",     // optional channel-specific override
        prompt: "HEARTBEAT.md が存在する場合は読み込んでください(ワークスペースコンテキスト)。厳密に従ってください。以前のチャットから古いタスクを推測したり繰り返したりしないでください。注意が必要なものがない場合は、HEARTBEAT_OK と返信してください。",
        ackMaxChars: 300         // max chars allowed after HEARTBEAT_OK
      }
    }
  }
}
```

<div id="scope-and-precedence">
  ### スコープと優先順位
</div>

* `agents.defaults.heartbeat` はハートビートのグローバルな動作を設定します。
* `agents.list[].heartbeat` はその設定に対してマージされます。いずれかのエージェントに `heartbeat` ブロックがある場合、**そのエージェントのみ** がハートビートを実行します。
* `channels.defaults.heartbeat` はすべてのチャネルに対する可視性のデフォルトを設定します。
* `channels.<channel>.heartbeat` はチャネルのデフォルトを上書きします。
* `channels.<channel>.accounts.<id>.heartbeat`（マルチアカウント対応チャネル）はチャネルごとの設定を上書きします。

<div id="per-agent-heartbeats">
  ### エージェント単位のハートビート
</div>

いずれかの `agents.list[]` エントリに `heartbeat` ブロックが含まれている場合、**そのエージェントだけが**
ハートビートを実行します。エージェント単位のブロックは `agents.defaults.heartbeat`
の上にマージされます（これにより、共通のデフォルトを一度設定し、エージェントごとに上書きできます）。

例: 2つのエージェントがあり、2番目のエージェントのみがハートビートを実行する。

```json5
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m",
        target: "last"
      }
    },
    list: [
      { id: "main", default: true },
      {
        id: "ops",
        heartbeat: {
          every: "1h",
          target: "whatsapp",
          to: "+15551234567",
          prompt: "Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK."
        }
      }
    ]
  }
}
```

<div id="field-notes">
  ### フィールドノート
</div>

* `every`: ハートビート間隔（期間を表す文字列。デフォルトの単位 = 分）。
* `model`: ハートビート実行時に使用する任意のモデル上書き指定（`provider/model`）。
* `includeReasoning`: 有効化すると、利用可能な場合は別メッセージとして `Reasoning:` も配信する（形式は `/reasoning on` と同じ）。
* `session`: ハートビート実行用の任意指定のセッションキー。
  * `main`（デフォルト）：エージェントのメインセッション。
  * セッションキーを明示指定（`openclaw sessions --json` または [sessions CLI](/ja/cli/sessions) からコピー）。
  * セッションキーの形式: [Sessions](/ja/concepts/session) および [Groups](/ja/concepts/groups) を参照。
* `target`:
  * `last`（デフォルト）：直近に使用された外部チャネルへ配信。
  * チャネルを明示指定: `whatsapp` / `telegram` / `discord` / `googlechat` / `slack` / `msteams` / `signal` / `imessage`。
  * `none`: ハートビートは実行するが、外部には**配信しない**。
* `to`: 宛先の任意上書き（チャネル固有の ID。例: WhatsApp の E.164 形式、または Telegram のチャット ID）。
* `prompt`: デフォルトのプロンプト本文を上書き（マージはしない）。
* `ackMaxChars`: 配信前に `HEARTBEAT_OK` の後へ追加できる最大文字数。

<div id="delivery-behavior">
  ## 配信動作
</div>

* ハートビートはデフォルトでエージェントのメインセッション（`agent:<id>:<mainKey>`）で実行されますが、
  `session.scope = "global"` の場合は `global` で実行されます。特定のチャネルセッション（Discord／WhatsApp／など）で実行するには、`session` を設定して切り替えます。
* `session` は実行コンテキストにのみ影響し、配信は `target` と `to` によって制御されます。
* 特定のチャネル／宛先に配信するには、`target` と `to` を設定します。`target: "last"` の場合、そのセッションで最後に使用された外部チャネルが配信先として使われます。
* メインキューがビジー状態の場合、ハートビートはスキップされ、後で再試行されます。
* `target` がいずれの外部宛先にも解決されない場合でも、実行自体は行われますが、外部へのメッセージは送信されません。
* ハートビートのみの返信ではセッションをアクティブな状態には**しません**。最後の `updatedAt` が元に戻されるため、アイドルによる期限切れは通常どおり動作します。

<div id="visibility-controls">
  ## 表示制御
</div>

デフォルトでは、アラート内容が配信されている間、`HEARTBEAT_OK` の確認メッセージは抑制されます。これはチャネル単位またはアカウント単位で調整できます。

```yaml
channels:
  defaults:
    heartbeat:
      showOk: false      # HEARTBEAT_OKを非表示（デフォルト）
      showAlerts: true   # アラートメッセージを表示（デフォルト）
      useIndicator: true # インジケーターイベントを発行（デフォルト）
  telegram:
    heartbeat:
      showOk: true       # TelegramでOK確認応答を表示
  whatsapp:
    accounts:
      work:
        heartbeat:
          showAlerts: false # このアカウントではアラート配信を抑制
```

優先順位: アカウント単位 → チャンネル単位 → チャンネルのデフォルト設定 → 組み込みのデフォルト設定。

<div id="what-each-flag-does">
  ### 各フラグの役割
</div>

* `showOk`: モデルが OK のみの応答を返したときに、`HEARTBEAT_OK` の確認応答を送信します。
* `showAlerts`: モデルが OK 以外の応答を返したときに、アラート内容を送信します。
* `useIndicator`: UI のステータス表示向けにインジケーターイベントを発行します。

**3 つすべて** が false の場合、OpenClaw はハートビートの実行自体をスキップします（モデル呼び出しは行われません）。

<div id="per-channel-vs-per-account-examples">
  ### チャンネル単位とアカウント単位の比較例
</div>

```yaml
channels:
  defaults:
    heartbeat:
      showOk: false
      showAlerts: true
      useIndicator: true
  slack:
    heartbeat:
      showOk: true # 全Slackアカウント
    accounts:
      ops:
        heartbeat:
          showAlerts: false # opsアカウントのみアラートを抑制
  telegram:
    heartbeat:
      showOk: true
```

<div id="common-patterns">
  ### よくあるパターン
</div>

| 目的 | 設定 |
| --- | --- |
| デフォルト動作（OK は通知せず、アラートは通知） | *(設定は不要)* |
| 完全サイレント（メッセージもインジケーターもなし） | `channels.defaults.heartbeat: { showOk: false, showAlerts: false, useIndicator: false }` |
| インジケーターのみ（メッセージなし） | `channels.defaults.heartbeat: { showOk: false, showAlerts: false, useIndicator: true }` |
| OK 通知を 1 つのチャンネルだけで表示 | `channels.telegram.heartbeat: { showOk: true }` |

<div id="heartbeatmd-optional">
  ## HEARTBEAT.md (任意)
</div>

`HEARTBEAT.md` ファイルがワークスペース内に存在する場合、デフォルトのプロンプトは
エージェントにそれを read するよう指示します。これは「ハートビート用チェックリスト」
と考えてください。小さく、安定していて、安全に 30 分ごとにプロンプトへ含められるものです。

`HEARTBEAT.md` が存在していても、実質的に空 (空行と、`# Heading` のような
markdown 見出しだけ) の場合、OpenClaw は API 呼び出しを節約するために
ハートビートの実行をスキップします。ファイルが存在しない場合でも、
ハートビート自体は実行され、モデルが何をするかを決定します。

プロンプトが肥大化しないように、小さく (短いチェックリストやリマインダー程度に)
保ってください。

`HEARTBEAT.md` の例:

```md
# ハートビートチェックリスト

- クイックスキャン: 受信トレイに緊急のものはあるか?
- 日中で他に保留中のものがなければ、軽量なチェックインを実行する。
- タスクがブロックされている場合、*何が不足しているか*を書き留めて、次回Peterに尋ねる。
```

<div id="can-the-agent-update-heartbeatmd">
  ### エージェントは HEARTBEAT.md を更新できますか？
</div>

はい。あなたがそう指示すれば更新できます。

`HEARTBEAT.md` はエージェントのワークスペース内にある通常のファイルなので、
通常のチャットでエージェントに対して次のように指示できます。

* 「`HEARTBEAT.md` を更新して、毎日のカレンダーチェックを追加して。」
* 「`HEARTBEAT.md` を書き直して、もっと短く、受信トレイのフォローアップに集中した内容にして。」

これをこちらから毎回依頼しなくても自発的に行わせたい場合は、ハートビートプロンプトの中に次のような
明示的な一文を含めることもできます：「チェックリストが古くなったら、
より良いものに差し替えるために HEARTBEAT.md を更新すること。」

安全上の注意：`HEARTBEAT.md` には秘密情報（API キー、電話番号、秘密トークンなど）を
記載しないでください。プロンプトコンテキストの一部になります。

<div id="manual-wake-on-demand">
  ## 手動起動（オンデマンド）
</div>

次のコマンドでシステムイベントをキューに登録し、即時にハートビートを実行できます：

```bash
openclaw system event --text "Check for urgent follow-ups" --mode now
```

複数のエージェントで `heartbeat` が設定されている場合、手動ウェイクを実行すると、それらの各エージェントのハートビートが直ちに実行されます。

次にスケジュールされている tick を待機するには、`--mode next-heartbeat` を使用します。

<div id="reasoning-delivery-optional">
  ## 推論内容の配信（オプション）
</div>

デフォルトでは、ハートビートは最終的な「回答」ペイロードだけを配信します。

内部の挙動を把握したい場合は、次を有効化します:

* `agents.defaults.heartbeat.includeReasoning: true`

有効化すると、ハートビートは `Reasoning:` というプレフィックスが付いた
別メッセージも配信します（`/reasoning on` と同じ形式）。これはエージェントが
複数のセッション／codex を管理している場合に、なぜあなたに ping する判断を
行ったのかを確認するのに役立ちます。ただし、望む以上の内部情報が漏れる
可能性もあります。グループチャットではオフのままにしておくことを推奨します。

<div id="cost-awareness">
  ## コスト意識
</div>

ハートビートではエージェントのターンがフルで実行されます。インターバルを短くすると、より多くのトークンを消費します。`HEARTBEAT.md` は小さく保ち、内部状態の更新だけが目的であれば、より安価な `model` を使うか、`target: "none"` を指定することも検討してください。