---
title: Voicecall
summary: "Riferimento CLI per `openclaw voicecall` (superficie di comandi del plugin voice-call)"
read_when:
  - Usi il plugin voice-call e ti servono i punti di ingresso via CLI
  - Vuoi esempi rapidi dei comandi `voicecall call|continue|status|tail|expose`
---

<div id="openclaw-voicecall">
  # `openclaw voicecall`
</div>

`voicecall` è un comando messo a disposizione da un plugin. Compare solo se il plugin voice-call è installato e abilitato.

Documentazione principale:

* Plugin voice-call: [Voice Call](/it/plugins/voice-call)

<div id="common-commands">
  ## Comandi più comuni
</div>

```bash
openclaw voicecall status --call-id <id>
openclaw voicecall call --to "+15555550123" --message "Hello" --mode notify
openclaw voicecall continue --call-id <id> --message "Any questions?"
openclaw voicecall end --call-id <id>
```

<div id="exposing-webhooks-tailscale">
  ## Esposizione dei webhook con Tailscale
</div>

```bash
openclaw voicecall expose --mode serve
openclaw voicecall expose --mode funnel
openclaw voicecall unexpose
```

Nota di sicurezza: esporre l&#39;endpoint webhook solo a reti attendibili. Quando possibile, preferire Tailscale Serve a Funnel.
