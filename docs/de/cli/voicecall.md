---
title: Voicecall
summary: "CLI-Referenz für `openclaw voicecall` (Befehlsoberfläche des Voicecall-Plugins)"
read_when:
  - Du nutzt das Voicecall-Plugin und möchtest die CLI-Einstiegspunkte
  - Du möchtest kurze Beispiele für `voicecall call|continue|status|tail|expose`
---

<div id="openclaw-voicecall">
  # `openclaw voicecall`
</div>

`voicecall` ist ein Befehl, der von einem Plugin bereitgestellt wird. Er wird nur angezeigt, wenn das Voice-Call-Plugin installiert und aktiviert ist.

Primäres Dokument:

* Voice-Call-Plugin: [Voice Call](/de/plugins/voice-call)

<div id="common-commands">
  ## Häufig verwendete Befehle
</div>

```bash
openclaw voicecall status --call-id <id>
openclaw voicecall call --to "+15555550123" --message "Hello" --mode notify
openclaw voicecall continue --call-id <id> --message "Any questions?"
openclaw voicecall end --call-id <id>
```

<div id="exposing-webhooks-tailscale">
  ## Exponieren von Webhooks (Tailscale)
</div>

```bash
openclaw voicecall expose --mode serve
openclaw voicecall expose --mode funnel
openclaw voicecall unexpose
```

Sicherheitshinweis: Mache den Webhook-Endpunkt nur in Netzen zugänglich, denen du vertraust. Verwende nach Möglichkeit Tailscale Serve statt Funnel.
