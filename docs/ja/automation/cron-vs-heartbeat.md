---
title: Cron と ハートビート
summary: "自動化のためにハートビートと cron ジョブのどちらを選択するかの指針"
read_when:
  - 定期実行タスクのスケジュール方法を検討するとき
  - バックグラウンド監視や通知を設定するとき
  - 定期チェックにおけるトークン使用量を最適化したいとき
---

<div id="cron-vs-heartbeat-when-to-use-each">
  # Cron とハートビート: どちらを使うべきか
</div>

ハートビートと cron ジョブはいずれも、タスクを定期的に実行できます。このガイドでは、ユースケースに応じて適切な仕組みを選ぶための判断材料を示します。

<div id="quick-decision-guide">
  ## クイック判断ガイド
</div>

| ユースケース | 推奨 | 理由 |
|----------|-------------|-----|
| 30分ごとに受信トレイを確認 | Heartbeat | 他のチェックとまとめて実行でき、コンテキストも考慮できる |
| 毎日9時ちょうどにレポートを送信 | Cron（単独） | 厳密な時刻指定が必要 |
| 直近の予定のためにカレンダーを監視 | Heartbeat | 定期的な状況把握に向いている |
| 毎週の詳細な分析を実行 | Cron（単独） | 単独タスクであり、別のモデルを使える |
| 20分後にリマインド | Cron（メイン、`--at`） | 一度きりで、正確なタイミングが必要 |
| プロジェクトのヘルスチェックをバックグラウンドで実行 | Heartbeat | 既存のサイクルに便乗できる |

<div id="heartbeat-periodic-awareness">
  ## ハートビート: 定期的な状況把握
</div>

ハートビートは **メインセッション** で、一定間隔（デフォルト: 30分）ごとに実行されます。エージェントが状況を確認し、重要な事項があれば表に出せるように設計されています。

<div id="when-to-use-heartbeat">
  ### ハートビートを利用する場面
</div>

* **複数の定期チェック**: inbox、カレンダー、天気、通知、プロジェクトのステータスを確認する 5 個の個別の cron ジョブを実行する代わりに、1 回のハートビートでこれらをまとめて処理できます。
* **コンテキストを考慮した判断**: エージェントはメインセッション全体のコンテキストを保持しているため、何が緊急で、何を後回しにできるかを賢く判断できます。
* **会話の継続性**: ハートビートの実行は同じセッションを共有するため、エージェントは最近の会話内容を把握しており、自然にフォローアップできます。
* **低オーバーヘッドな監視**: 1 つのハートビートで、多数の小さなポーリングタスクを置き換えられます。

<div id="heartbeat-advantages">
  ### ハートビートの利点
</div>

* **複数のチェックをまとめて実行**: 1 回のエージェントのターンで、受信トレイ、カレンダー、通知をまとめて確認できます。
* **API 呼び出し数を削減**: 単一のハートビートのほうが、個別に分かれた 5 つの cron ジョブよりもコストが低くなります。
* **コンテキストを理解**: エージェントはあなたが何に取り組んでいたかを把握しており、それに応じて優先順位を付けられます。
* **スマートな抑制**: 対応が必要なものがなければ、エージェントは `HEARTBEAT_OK` と返信し、メッセージは送信されません。
* **自然なタイミング**: キュー負荷に応じてわずかにずれますが、ほとんどの監視用途では問題ありません。

<div id="heartbeat-example-heartbeatmd-checklist">
  ### ハートビートの例: HEARTBEAT.md 用チェックリスト
</div>

```md
# ハートビートチェックリスト

- 緊急メッセージのメールを確認
- 今後2時間以内のイベントをカレンダーで確認
- バックグラウンドタスクが完了した場合は結果を要約
- 8時間以上アイドル状態の場合は簡単な確認メッセージを送信
```

エージェントは各ハートビートのたびにこれを読み込み、すべての項目を一度のターンで処理します。

<div id="configuring-heartbeat">
  ### ハートビートの設定
</div>

```json5
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m",        // interval
        target: "last",      // where to deliver alerts
        activeHours: { start: "08:00", end: "22:00" }  // 任意
      }
    }
  }
}
```

詳細な設定については、[ハートビート](/ja/gateway/heartbeat) を参照してください。

<div id="cron-precise-scheduling">
  ## Cron: 厳密なスケジューリング
</div>

Cron ジョブは**正確な時刻**に実行され、メインのコンテキストに影響を与えることなく、分離されたセッション内で実行できます。

<div id="when-to-use-cron">
  ### cron を使うとよい場面
</div>

* **厳密な時刻指定が必要な場合**: 「毎週月曜の午前 9:00 にこれを送信して」のように（「9 時ごろ」のような曖昧さではない場合）。
* **単体で完結するタスク**: 会話コンテキストを必要としないタスク。
* **別のモデルや思考モードが必要な場合**: より高性能なモデルを使うに値する重い分析。
* **一度きりのリマインダー**: `--at` を使った「20 分後にリマインドして」のようなもの。
* **ノイズが多い／頻度の高いタスク**: メインのセッション履歴を散らかしてしまうタスク。
* **外部トリガー**: エージェントがアクティブかどうかとは無関係に実行されるべきタスク。

<div id="cron-advantages">
  ### Cron の利点
</div>

* **正確なタイミング**: タイムゾーン対応の 5 フィールドの cron 式。
* **セッション分離**: メインの履歴を汚さずに `cron:<jobId>` で実行。
* **モデル上書き**: ジョブごとに、より安価または高性能なモデルを使用可能。
* **配信制御**: チャンネルへ直接配信でき、デフォルトではメインにも要約を投稿（設定可能）。
* **エージェントコンテキスト不要**: メインセッションがアイドル状態や圧縮済みでも実行される。
* **ワンショット対応**: 将来時刻を正確に指定する `--at` を利用可能。

<div id="cron-example-daily-morning-briefing">
  ### Cron の例: 毎朝のブリーフィング通知
</div>

```bash
openclaw cron add \
  --name "Morning briefing" \
  --cron "0 7 * * *" \
  --tz "America/New_York" \
  --session isolated \
  --message "Generate today's briefing: weather, calendar, top emails, news summary." \
  --model opus \
  --deliver \
  --channel whatsapp \
  --to "+15551234567"
```

これはニューヨーク時間の午前7時ちょうどに実行され、音質向上のために Opus を使用し、WhatsApp に直接配信します。

<div id="cron-example-one-shot-reminder">
  ### Cron の例: 単発リマインダー
</div>

```bash
openclaw cron add \
  --name "Meeting reminder" \
  --at "20m" \
  --session main \
  --system-event "Reminder: standup meeting starts in 10 minutes." \
  --wake now \
  --delete-after-run
```

CLI の完全なリファレンスについては、[Cron jobs](/ja/automation/cron-jobs) を参照してください。

<div id="decision-flowchart">
  ## 意思決定フローチャート
</div>

```
Does the task need to run at an EXACT time?
  YES -> Use cron
  NO  -> Continue...

Does the task need isolation from main session?
  YES -> Use cron (isolated)
  NO  -> Continue...

Can this task be batched with other periodic checks?
  YES -> Use heartbeat (add to HEARTBEAT.md)
  NO  -> Use cron

Is this a one-shot reminder?
  YES -> Use cron with --at
  NO  -> Continue...

Does it need a different model or thinking level?
  YES -> Use cron (isolated) with --model/--thinking
  NO  -> Use heartbeat
```

<div id="combining-both">
  ## 両方を組み合わせる
</div>

最も効率的な構成は、**両方**を使うことです：

1. **ハートビート** は受信箱・カレンダー・通知などの定期的な監視を、30分ごとに1回のバッチ処理としてまとめて実行します。
2. **Cron** は日次レポートや週次レビューなどの正確なスケジュールと、1回限りのリマインダーを担当します。

<div id="example-efficient-automation-setup">
  ### 例：効率的な自動化のセットアップ
</div>

**HEARTBEAT.md**（30分ごとにチェック）:

```md
# ハートビートチェックリスト
- 受信トレイで緊急メールをスキャン
- 今後2時間のイベントをカレンダーで確認
- 保留中のタスクを確認
- 8時間以上静かな場合は軽く確認
```

**Cron ジョブ**（正確なスケジューリング）：

```bash
# Daily morning briefing at 7am
openclaw cron add --name "Morning brief" --cron "0 7 * * *" --session isolated --message "..." --deliver

# 毎週月曜日9時のプロジェクトレビュー
openclaw cron add --name "Weekly review" --cron "0 9 * * 1" --session isolated --message "..." --model opus

# One-shot reminder
openclaw cron add --name "Call back" --at "2h" --session main --system-event "Call back the client" --wake now
```

<div id="lobster-deterministic-workflows-with-approvals">
  ## Lobster: 明示的な承認付きの決定論的ワークフロー
</div>

Lobster は、決定論的な実行と明示的な承認が必要な **複数ステップのツールパイプライン** 向けのワークフローランタイムです。
タスクが単一のエージェントターンを超えており、人によるチェックポイントを挟みつつ再開可能なワークフローにしたい場合に使用します。

<div id="when-lobster-fits">
  ### Lobster が適しているケース
</div>

* **複数ステップの自動化**: 一度きりのプロンプトではなく、決まったツール呼び出しパイプラインが必要な場合。
* **承認ゲート**: 副作用を伴う処理は、あなたが承認するまで一時停止させ、その後に再開させたい場合。
* **再開可能な実行**: 途中で停止したワークフローを、前のステップを再実行せずに続行したい場合。

<div id="how-it-pairs-with-heartbeat-and-cron">
  ### ハートビートと cron との連携方法
</div>

* **ハートビート/cron** は、実行を「いつ」行うかを決定します。
* **Lobster** は、実行開始後に「どのステップ」を実行するかを定義します。

スケジュール実行のワークフローでは、cron またはハートビートを使って、Lobster を呼び出すエージェントのターンをトリガーします。
アドホックなワークフローでは、Lobster を直接呼び出します。

<div id="operational-notes-from-the-code">
  ### 運用上の注意（コードから）
</div>

* Lobster はツールモードで **ローカルサブプロセス**（`lobster` CLI）として動作し、**JSON エンベロープ**を返します。
* ツールが `needs_approval` を返した場合、`resumeToken` と `approve` フラグを指定して再開します。
* このツールは **任意のプラグイン** です。`tools.alsoAllow: ["lobster"]` で追加で有効化してください（推奨）。
* `lobsterPath` を指定する場合、それは **絶対パス** でなければなりません。

詳しい使い方と例については [Lobster](/ja/tools/lobster) を参照してください。

<div id="main-session-vs-isolated-session">
  ## メインセッション vs 分離セッション
</div>

ハートビートと cron のどちらもメインセッションとやり取りできますが、その方法は異なります：

| | ハートビート | cron（メイン） | cron（分離） |
|---|---|---|---|
| セッション | メイン | メイン（システムイベント経由） | `cron:<jobId>` |
| 履歴 | 共有 | 共有 | 実行ごとに新規 |
| コンテキスト | フル | フル | なし（クリーンな状態から開始） |
| モデル | メインセッションのモデル | メインセッションのモデル | 上書き可能 |
| 出力 | `HEARTBEAT_OK` でない場合に配信 | ハートビートプロンプト＋イベント | サマリーをメインセッションに投稿 |

<div id="when-to-use-main-session-cron">
  ### メインセッションの cron を使うべきタイミング
</div>

`--system-event` と併せて `--session main` を使用するのは、次のような場合です。

* リマインダーやイベントをメインセッションのコンテキスト内で発生させたいとき
* エージェントに、次回のハートビート時に完全なコンテキスト付きで処理させたいとき
* 別の独立した実行を走らせたくないとき

```bash
openclaw cron add \
  --name "Check project" \
  --every "4h" \
  --session main \
  --system-event "Time for a project health check" \
  --wake now
```

<div id="when-to-use-isolated-cron">
  ### isolated cron を使うタイミング
</div>

`--session isolated` を使うのは、次のような場合です:

* これまでのコンテキストを持たないクリーンな状態にしたいとき
* 別のモデルや推論設定を使いたいとき
* 出力を直接チャネルに送信したいとき（要約はデフォルトでメインにも投稿されます）
* 履歴でメインのセッションを散らかしたくないとき

```bash
openclaw cron add \
  --name "Deep analysis" \
  --cron "0 6 * * 0" \
  --session isolated \
  --message "Weekly codebase analysis..." \
  --model opus \
  --thinking high \
  --deliver
```

<div id="cost-considerations">
  ## コストに関する考慮事項
</div>

| Mechanism | コスト特性 |
|-----------|------------|
| Heartbeat | N 分ごとに 1 ターン；`HEARTBEAT.md` のサイズに応じて増加 |
| Cron (main) | 次回のハートビートにイベントを追加（単独のターンは発生しない） |
| Cron (isolated) | ジョブごとにエージェントのフルターンが実行される；より安価なモデルを使用可能 |

**Tips**:

* トークンのオーバーヘッドを抑えるために `HEARTBEAT.md` を小さく保つ。
* 複数の cron ジョブに分けるのではなく、類似のチェックはハートビートにまとめる。
* 内部処理だけを行いたい場合は、ハートビートで `target: "none"` を使用する。
* 定型的なタスクには、より安価なモデルを使った分離 cron を使用する。

<div id="related">
  ## 関連
</div>

* [Heartbeat](/ja/gateway/heartbeat) - ハートビートの詳細な設定リファレンス
* [Cron jobs](/ja/automation/cron-jobs) - Cron ジョブの CLI および API の詳細リファレンス
* [System](/ja/cli/system) - システムイベントおよびハートビートの制御