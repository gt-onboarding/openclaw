---
title: ログ
summary: "`openclaw logs` の CLI リファレンス（RPC 経由で Gateway ログを tail する）"
read_when:
  - SSH なしでリモートから Gateway ログを tail したいとき
  - ツール連携用に JSON 形式のログ行が欲しいとき
---

<div id="openclaw-logs">
  # `openclaw logs`
</div>

RPC 経由で Gateway のファイルログを tail 表示します（リモートモードでも使用できます）。

関連:

* ログの概要: [Logging](/ja/logging)

<div id="examples">
  ## 使用例
</div>

```bash
openclaw logs
openclaw logs --follow
openclaw logs --json
openclaw logs --limit 500
```
