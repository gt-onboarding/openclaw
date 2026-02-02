---
title: リセット
summary: "CLI リファレンス: `openclaw reset`（ローカル状態／設定をリセット）"
read_when:
  - CLI をインストールしたままローカル状態を消去しておきたいとき
  - 何が削除されるかのドライランを実行したいとき
---

<div id="openclaw-reset">
  # `openclaw reset`
</div>

ローカルの設定／状態をリセットします（CLI はインストールされたままです）。

```bash
openclaw reset
openclaw reset --dry-run
openclaw reset --scope config+creds+sessions --yes --non-interactive
```
