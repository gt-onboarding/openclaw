---
title: バックグラウンドプロセス
summary: "バックグラウンド exec の実行とプロセス管理"
read_when:
  - バックグラウンド exec の挙動を追加・変更するとき
  - 長時間実行中の exec タスクをデバッグするとき
---

<div id="background-exec-process-tool">
  # バックグラウンド Exec と Process ツール
</div>

OpenClaw は `exec` ツールを介してシェルコマンドを実行し、長時間実行されるタスクをメモリ内に保持します。`process` ツールは、それらのバックグラウンドセッションを管理します。

<div id="exec-tool">
  ## exec ツール
</div>

主なパラメータ:

- `command` (必須)
- `yieldMs` (デフォルト 10000): この待ち時間後に自動でバックグラウンド実行に移行
- `background` (bool): 即座にバックグラウンド実行にする
- `timeout` (秒, デフォルト 1800): このタイムアウト後にプロセスを kill する
- `elevated` (bool): 昇格モードが有効/許可されている場合はホスト上で実行
- 実際の TTY が必要な場合は `pty: true` を設定
- `workdir`, `env`

動作:

- フォアグラウンド実行では、出力が直接返される。
- バックグラウンド化された場合（明示的指定またはタイムアウト）、ツールは `status: "running"` と `sessionId`、および末尾の短い出力を返す。
- 出力は、そのセッションがポーリングされるかクリアされるまでメモリ上に保持される。
- `process` ツールが無効化されている場合、`exec` は同期実行となり、`yieldMs` / `background` は無視される。

<div id="child-process-bridging">
  ## 子プロセスブリッジ
</div>

exec/process ツールの外で長時間動作する子プロセス（例: CLI の再起動や Gateway ヘルパー）を起動する場合は、子プロセスブリッジヘルパーを接続して、終了シグナルがフォワードされ、終了/エラー時にリスナーが解除されるようにします。これにより、systemd 上での孤立プロセスを防ぎ、プラットフォーム間でシャットダウン動作を一貫させます。

環境変数によるオーバーライド:

- `PI_BASH_YIELD_MS`: デフォルトのイールド時間 (ミリ秒)
- `PI_BASH_MAX_OUTPUT_CHARS`: メモリ内出力の上限 (文字数)
- `OPENCLAW_BASH_PENDING_MAX_OUTPUT_CHARS`: ストリームごとの保留中 stdout/stderr の上限 (文字数)
- `PI_BASH_JOB_TTL_MS`: 完了したセッションの TTL (ミリ秒、1 分〜3 時間に制限)

設定（推奨）:

- `tools.exec.backgroundMs` (デフォルト 10000)
- `tools.exec.timeoutSec` (デフォルト 1800)
- `tools.exec.cleanupMs` (デフォルト 1800000)
 - `tools.exec.notifyOnExit` (デフォルト true): バックグラウンド実行中の exec が終了したときに、システムイベントをキューに入れ、ハートビートをリクエストします。

<div id="process-tool">
  ## process tool
</div>

アクション:

- `list`: 実行中 + 完了済みのセッション
- `poll`: セッションの新しい出力をすべて取り出す（終了ステータスも報告）
- `log`: 集約された出力を読み取る（`offset` + `limit` をサポート）
- `write`: stdin を送信する（`data` と、任意指定の `eof`）
- `kill`: バックグラウンドセッションを終了する
- `clear`: 完了済みセッションをメモリから削除する
- `remove`: 実行中なら kill、完了済みなら clear

注意事項:

- バックグラウンド化されたセッションのみが一覧表示され、メモリに保持されます。
- プロセスを再起動するとセッションは失われます（ディスクへの永続化は行われません）。
- セッションログは、`process poll/log` を実行してツール結果が記録された場合にのみチャット履歴に保存されます。
- `process` はエージェント単位でスコープが設定されており、そのエージェントが開始したセッションのみを参照できます。
- `process list` には、クイックスキャン用に派生された `name`（コマンド動詞 + 対象）が含まれます。
- `process log` は行ベースの `offset` / `limit` を使用します（最後の N 行を取得するには `offset` を省略）。

<div id="examples">
  ## 例
</div>

長時間実行されるタスクを開始し、後からポーリングする:

```json
{"tool": "exec", "command": "sleep 5 && echo done", "yieldMs": 1000}
```

```json
{"tool": "process", "action": "poll", "sessionId": "<id>"}
```

バックグラウンドですぐに開始：

```json
{"tool": "exec", "command": "npm run build", "background": true}
```

標準入力を送信:

```json
{"tool": "process", "action": "write", "sessionId": "<id>", "data": "y\n"}
```
