---
title: ヘルスチェック
summary: "`openclaw health` の CLI リファレンス（RPC 経由で Gateway のヘルスチェック用エンドポイントにアクセスします）"
read_when:
  - 稼働中の Gateway のヘルス状態を手早く確認したいとき
---

<div id="openclaw-health">
  # `openclaw health`
</div>

稼働中の Gateway からヘルスチェック情報を取得します。

```bash
openclaw health
openclaw health --json
openclaw health --verbose
```

Notes:

* `--verbose` はライブプローブを実行し、複数のアカウントが設定されている場合はアカウントごとの処理時間を表示します。
* 複数のエージェントが設定されている場合は、出力にエージェントごとのセッションストアが含まれます。
