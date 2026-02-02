---
title: Registros
summary: "Referencia de la CLI para `openclaw logs` (monitorizar en tiempo real los registros del Gateway mediante RPC)"
read_when:
  - Necesitas monitorizar en tiempo real los registros del Gateway de forma remota (sin SSH)
  - Quieres líneas de logs en formato JSON para herramientas
---

<div id="openclaw-logs">
  # `openclaw logs`
</div>

Muestra en tiempo real los registros en archivos del Gateway mediante RPC (funciona en modo remoto).

Relacionado:

* Información general sobre el registro: [Logging](/es/logging)

<div id="examples">
  ## Ejemplos
</div>

```bash
openclaw logs
openclaw logs --follow
openclaw logs --json
openclaw logs --limit 500
```
