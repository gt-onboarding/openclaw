---
title: 语音通话
summary: "用于 `openclaw voicecall` 的 CLI 参考（voice-call 插件命令接口）"
read_when:
  - 你使用 voice-call 插件并且需要对应的 CLI 入口
  - 你想要快速查看 `voicecall call|continue|status|tail|expose` 的示例
---

<div id="openclaw-voicecall">
  # `openclaw voicecall`
</div>

`voicecall` 是由插件提供的命令。该命令仅在安装并启用了语音通话插件后才会出现。

主要文档：

* 语音通话插件：[Voice Call](/zh/plugins/voice-call)

<div id="common-commands">
  ## 常用命令
</div>

```bash
openclaw voicecall status --call-id <id>
openclaw voicecall call --to "+15555550123" --message "Hello" --mode notify
openclaw voicecall continue --call-id <id> --message "Any questions?"
openclaw voicecall end --call-id <id>
```

<div id="exposing-webhooks-tailscale">
  ## 公开 Webhook（Tailscale）
</div>

```bash
openclaw voicecall expose --mode serve
openclaw voicecall expose --mode funnel
openclaw voicecall unexpose
```

安全说明：仅将该 webhook 端点暴露给你信任的网络。在可能的情况下，优先使用 Tailscale Serve 而非 Funnel。
