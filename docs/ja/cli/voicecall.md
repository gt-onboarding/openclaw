---
title: Voicecall
summary: "`openclaw voicecall`（voice-call プラグインのコマンドインターフェイス）向け CLI リファレンス"
read_when:
  - "voice-call プラグインを利用していて、CLI のエントリーポイントを把握したいとき"
  - "`voicecall call|continue|status|tail|expose` の簡単な使用例を知りたいとき"
---

<div id="openclaw-voicecall">
  # `openclaw voicecall`
</div>

`voicecall` はプラグインで提供されるコマンドです。音声通話プラグインがインストールされ、有効になっている場合にのみ表示されます。

主なドキュメント:

* 音声通話プラグイン: [Voice Call](/ja/plugins/voice-call)

<div id="common-commands">
  ## よく使うコマンド
</div>

```bash
openclaw voicecall status --call-id <id>
openclaw voicecall call --to "+15555550123" --message "Hello" --mode notify
openclaw voicecall continue --call-id <id> --message "Any questions?"
openclaw voicecall end --call-id <id>
```

<div id="exposing-webhooks-tailscale">
  ## Webhook を公開する（Tailscale）
</div>

```bash
openclaw voicecall expose --mode serve
openclaw voicecall expose --mode funnel
openclaw voicecall unexpose
```

セキュリティ上の注意: 信頼できるネットワークに対してのみ webhook エンドポイントを公開してください。可能な限り、Funnel ではなく Tailscale Serve を優先して使用してください。
