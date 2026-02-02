---
title: Agente
summary: "Referencia de la CLI para `openclaw agent` (enviar un turno de un agente a través del Gateway)"
read_when:
  - Quieres ejecutar un turno de un agente desde scripts (con entrega opcional de la respuesta)
---

<div id="openclaw-agent">
  # `openclaw agent`
</div>

Ejecuta un turno de un agente a través del Gateway (usa `--local` para modo integrado).
Usa `--agent <id>` para dirigirte directamente a un agente configurado.

Relacionado:

* Herramienta Agent send: [Agent send](/es/tools/agent-send)

<div id="examples">
  ## Ejemplos
</div>

```bash
openclaw agent --to +15555550123 --message "status update" --deliver
openclaw agent --agent ops --message "Summarize logs"
openclaw agent --session-id 1234 --message "Summarize inbox" --thinking medium
openclaw agent --agent ops --message "Generate report" --deliver --reply-channel slack --reply-to "#reports"
```
