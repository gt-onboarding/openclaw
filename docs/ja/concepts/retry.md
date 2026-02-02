---
title: 再試行
summary: "外向きのプロバイダー呼び出し向け再試行ポリシー"
read_when:
  - プロバイダーの再試行動作またはデフォルト設定を更新するとき
  - プロバイダーの送信エラーやレート制限の問題をデバッグするとき
---

<div id="retry-policy">
  # 再試行ポリシー
</div>

<div id="goals">
  ## 目的
</div>

- 複数ステップからなるフロー単位ではなく、HTTP リクエスト単位でリトライする。
- 現在のステップのみをリトライすることで処理の順序を維持する。
- 冪等ではない操作を重複実行しないようにする。

<div id="defaults">
  ## デフォルト
</div>

- 試行回数: 3
- 最大遅延上限: 30000 ms
- ジッター係数: 0.1（10パーセント）
- プロバイダーのデフォルト:
  - Telegram 最小遅延時間: 400 ms
  - Discord 最小遅延時間: 500 ms

<div id="behavior">
  ## 挙動
</div>

<div id="discord">
  ### Discord
</div>

- レート制限エラー（HTTP 429）の場合にのみ再試行します。
- Discord の `retry_after` が利用可能な場合はそれを使用し、それ以外の場合には指数バックオフで対応します。

<div id="telegram">
  ### Telegram
</div>

- 一時的なエラー（429、タイムアウト、接続リセット／切断、接続クローズ、一時的なサービス停止など）の場合は再試行を行います。
- `retry_after` が利用可能な場合はそれを使用し、ない場合は指数バックオフを行います。
- Markdown のパースエラーは再試行せず、プレーンテキストにフォールバックします。

<div id="configuration">
  ## 設定
</div>

プロバイダーごとのリトライポリシーは `~/.openclaw/openclaw.json` で設定します。

```json5
{
  channels: {
    telegram: {
      retry: {
        attempts: 3,
        minDelayMs: 400,
        maxDelayMs: 30000,
        jitter: 0.1
      }
    },
    discord: {
      retry: {
        attempts: 3,
        minDelayMs: 500,
        maxDelayMs: 30000,
        jitter: 0.1
      }
    }
  }
}
```


<div id="notes">
  ## 注意事項
</div>

- 再試行はリクエストごとに適用されます（メッセージ送信、メディアアップロード、リアクション、投票、スタンプ）。
- 複合フローでは、完了済みのステップは再試行されません。