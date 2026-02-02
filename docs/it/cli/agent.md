---
title: Agente
summary: "Riferimento CLI per `openclaw agent` (invio di un singolo turno di un agente tramite il Gateway)"
read_when:
  - Vuoi eseguire un singolo turno di un agente da script (con consegna facoltativa della risposta)
---

<div id="openclaw-agent">
  # `openclaw agent`
</div>

Esegui un turno di un agente tramite il Gateway (usa `--local` per l&#39;esecuzione incorporata).
Usa `--agent <id>` per indirizzare direttamente un agente configurato.

Correlato:

* Strumento Agent send: [Agent send](/it/tools/agent-send)

<div id="examples">
  ## Esempi
</div>

```bash
openclaw agent --to +15555550123 --message "status update" --deliver
openclaw agent --agent ops --message "Summarize logs"
openclaw agent --session-id 1234 --message "Summarize inbox" --thinking medium
openclaw agent --agent ops --message "Generate report" --deliver --reply-channel slack --reply-to "#reports"
```
