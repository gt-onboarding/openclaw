---
title: Voicecall
summary: "Referencia de la CLI para `openclaw voicecall` (superficie de comandos del complemento `voice-call`)"
read_when:
  - Usas el complemento `voice-call` y quieres los puntos de entrada en la CLI
  - Quieres ejemplos rápidos de `voicecall call|continue|status|tail|expose`
---

<div id="openclaw-voicecall">
  # `openclaw voicecall`
</div>

`voicecall` es un comando que proporciona un complemento. Solo aparece si el complemento de llamadas de voz está instalado y habilitado.

Documentación principal:

* Complemento de llamadas de voz: [Voice Call](/es/plugins/voice-call)

<div id="common-commands">
  ## Comandos habituales
</div>

```bash
openclaw voicecall status --call-id <id>
openclaw voicecall call --to "+15555550123" --message "Hello" --mode notify
openclaw voicecall continue --call-id <id> --message "Any questions?"
openclaw voicecall end --call-id <id>
```

<div id="exposing-webhooks-tailscale">
  ## Exponer webhooks (Tailscale)
</div>

```bash
openclaw voicecall expose --mode serve
openclaw voicecall expose --mode funnel
openclaw voicecall unexpose
```

Nota de seguridad: expón el endpoint del webhook solo en redes de confianza. Siempre que sea posible, usa Tailscale Serve en lugar de Funnel.
