---
title: Cron
summary: "`openclaw cron` の CLI リファレンス（バックグラウンドジョブのスケジュールと実行）"
read_when:
  - 定期実行ジョブやウェイクアップ処理を設定・実行したいとき
  - cron の実行やログをデバッグしているとき
---

<div id="openclaw-cron">
  # `openclaw cron`
</div>

Gateway スケジューラの cron ジョブを管理します。

関連:

* Cron ジョブ: [Cron jobs](/ja/automation/cron-jobs)

ヒント: 利用可能なコマンドをすべて確認するには、`openclaw cron --help` を実行します。

<div id="common-edits">
  ## よく行う操作
</div>

メッセージ内容を変えずに配信設定だけを更新する：

```bash
openclaw cron edit <job-id> --deliver --channel telegram --to "123456789"
```

特定のジョブの配信を無効にする：

```bash
openclaw cron edit <job-id> --no-deliver
```
