---
title: Appel vocal
summary: "Référence CLI pour `openclaw voicecall` (interface de commandes du plugin voice-call)"
read_when:
  - Vous utilisez le plugin voice-call et souhaitez connaître les points d’entrée de la CLI
  - Vous voulez des exemples rapides de `voicecall call|continue|status|tail|expose`
---

<div id="openclaw-voicecall">
  # `openclaw voicecall`
</div>

`voicecall` est une commande fournie par un plugin. Elle n’apparaît que si le plugin d’appel vocal est installé et activé.

Documentation principale :

* Plugin d’appel vocal : [Voice Call](/fr/plugins/voice-call)

<div id="common-commands">
  ## Commandes courantes
</div>

```bash
openclaw voicecall status --call-id <id>
openclaw voicecall call --to "+15555550123" --message "Hello" --mode notify
openclaw voicecall continue --call-id <id> --message "Any questions?"
openclaw voicecall end --call-id <id>
```

<div id="exposing-webhooks-tailscale">
  ## Exposition des webhooks (Tailscale)
</div>

```bash
openclaw voicecall expose --mode serve
openclaw voicecall expose --mode funnel
openclaw voicecall unexpose
```

Note de sécurité : n’exposez le point de terminaison du webhook qu’aux réseaux de confiance. Privilégiez Tailscale Serve à Funnel dès que possible.
