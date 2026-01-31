---
title: ステータス
summary: "`openclaw status` の CLI リファレンス（診断、プローブ、利用状況スナップショット）"
read_when:
  - チャネルの正常性と最近のセッション宛先を手早く確認・診断したいとき
  - デバッグ用にペーストしやすい「all」ステータスがほしいとき
---

<div id="openclaw-status">
  # `openclaw status`
</div>

チャンネルおよびセッションの診断。

```bash
openclaw status
openclaw status --all
openclaw status --deep
openclaw status --usage
```

補足:

* `--deep` を指定すると、ライブプローブ（WhatsApp Web + Telegram + Discord + Google Chat + Slack + Signal）を実行します。
* 複数のエージェントが設定されている場合、出力にはエージェントごとのセッションストアが含まれます。
* 概要には、利用可能な場合 Gateway およびノードホストサービスのインストール状況／実行時ステータスが含まれます。
* 概要には、更新チャネルと git SHA（ソースからチェックアウトした場合）が含まれます。
* 更新情報は概要に表示され、アップデートが利用可能な場合は、`openclaw update` を実行するよう促すヒントが status の出力に表示されます（[Updating](/ja/install/updating) を参照）。
