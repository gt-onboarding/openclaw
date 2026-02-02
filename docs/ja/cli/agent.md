---
title: エージェント
summary: "`openclaw agent` のCLIリファレンス（Gateway経由でエージェントのターンを1回送信する）"
read_when:
  - スクリプトからエージェントのターンを1回実行したい（必要に応じて返信も送信したい）とき
---

<div id="openclaw-agent">
  # `openclaw agent`
</div>

Gateway 経由でエージェントのターンを実行します（組み込みモードで実行する場合は `--local` を使用）。
特定の設定済みエージェントを直接指定するには `--agent <id>` を使用します。

関連:

* エージェント送信ツール: [Agent send](/ja/tools/agent-send)

<div id="examples">
  ## 使用例
</div>

```bash
openclaw agent --to +15555550123 --message "status update" --deliver
openclaw agent --agent ops --message "Summarize logs"
openclaw agent --session-id 1234 --message "Summarize inbox" --thinking medium
openclaw agent --agent ops --message "Generate report" --deliver --reply-channel slack --reply-to "#reports"
```
