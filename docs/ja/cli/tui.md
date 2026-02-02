---
title: Tui
summary: "CLI リファレンス: `openclaw tui`（Gateway に接続するターミナル UI）"
read_when:
  - Gateway 用のターミナル UI（リモート環境からの利用に適したもの）が必要な場合
  - スクリプトから url/token/session を渡す必要がある場合
---

<div id="openclaw-tui">
  # `openclaw tui`
</div>

Gateway に接続されたターミナルベースの UI を開きます。

関連:

* TUI ガイド: [TUI](/ja/tui)

<div id="examples">
  ## 使用例
</div>

```bash
openclaw tui
openclaw tui --url ws://127.0.0.1:18789 --token <token>
openclaw tui --session main --deliver
```
