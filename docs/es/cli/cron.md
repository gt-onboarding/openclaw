---
title: Cron
summary: "Referencia de la CLI para `openclaw cron` (programar y ejecutar tareas en segundo plano)"
read_when:
  - Necesitas tareas programadas y activaciones
  - Estás depurando la ejecución y los registros de cron
---

<div id="openclaw-cron">
  # `openclaw cron`
</div>

Administra trabajos cron para el planificador del Gateway.

Relacionado:

* Trabajos cron: [Cron jobs](/es/automation/cron-jobs)

Consejo: ejecuta `openclaw cron --help` para ver todas las opciones y subcomandos disponibles.

<div id="common-edits">
  ## Cambios habituales
</div>

Actualiza la configuración de entrega sin modificar el mensaje:

```bash
openclaw cron edit <job-id> --deliver --channel telegram --to "123456789"
```

Desactivar la entrega para un trabajo aislado:

```bash
openclaw cron edit <job-id> --no-deliver
```
