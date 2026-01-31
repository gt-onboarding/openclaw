---
title: システム
summary: "`openclaw system` の CLI リファレンス（システムイベント、ハートビート、プレゼンス）"
read_when:
  - cron ジョブを作成せずにシステムイベントをキューに追加したいとき
  - ハートビートを有効または無効にしたいとき
  - システムのプレゼンスエントリを確認したいとき
---

<div id="openclaw-system">
  # `openclaw system`
</div>

Gateway 向けのシステムレベルのヘルパーです。システムイベントのキュー投入、ハートビートの制御、
およびプレゼンス情報の表示を行います。

<div id="common-commands">
  ## 主なコマンド
</div>

```bash
openclaw system event --text "Check for urgent follow-ups" --mode now
openclaw system heartbeat enable
openclaw system heartbeat last
openclaw system presence
```


<div id="system-event">
  ## `system event`
</div>

**メイン**セッションにシステムイベントをキューイングします。次のハートビートで、
プロンプト内の `System:` 行として挿入されます。`--mode now` を使用すると
ハートビートを即時にトリガーできます。`next-heartbeat` は次回のスケジュール済み
ティックまで待機します。

Flags:

- `--text <text>`: 必須のシステムイベント用テキスト。
- `--mode <mode>`: `now` または `next-heartbeat`（デフォルト）。
- `--json`: 機械可読な形式で出力します。

<div id="system-heartbeat-lastenabledisable">
  ## `system heartbeat last|enable|disable`
</div>

ハートビートの制御:

- `last`: 最後のハートビートイベントを表示します。
- `enable`: ハートビートを再度有効にします（無効になっている場合に使用します）。
- `disable`: ハートビートを一時停止します。

フラグ:

- `--json`: 機械可読な形式で出力します。

<div id="system-presence">
  ## `system presence`
</div>

Gateway が把握している現在の system presence のエントリ（ノード、インスタンス、
その他同様のステータス行）を一覧表示します。

Flags:

- `--json`: マシン判読可能な出力。

<div id="notes">
  ## 注意事項
</div>

- 現在の設定（ローカルまたはリモート）から到達可能な、稼働中の Gateway が必要です。
- システムイベントは一時的なものであり、再起動をまたいで永続的には保存されません。