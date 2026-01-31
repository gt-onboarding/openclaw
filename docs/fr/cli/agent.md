---
title: Agent
summary: "Référence CLI pour la commande `openclaw agent` (envoyer un tour d’agent via le Gateway)"
read_when:
  - Vous souhaitez exécuter un tour d’agent depuis des scripts (et éventuellement transmettre la réponse)
---

<div id="openclaw-agent">
  # `openclaw agent`
</div>

Exécute un tour d’agent via le Gateway (utilisez `--local` pour une exécution embarquée).
Utilisez `--agent <id>` pour cibler directement un agent configuré.

Voir aussi :

* Outil Agent send : [Agent send](/fr/tools/agent-send)

<div id="examples">
  ## Exemples
</div>

```bash
openclaw agent --to +15555550123 --message "status update" --deliver
openclaw agent --agent ops --message "Summarize logs"
openclaw agent --session-id 1234 --message "Summarize inbox" --thinking medium
openclaw agent --agent ops --message "Generate report" --deliver --reply-channel slack --reply-to "#reports"
```
