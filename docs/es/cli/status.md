---
title: Estado
summary: "Referencia de la CLI para `openclaw status` (diagnósticos, comprobaciones, instantáneas de uso)"
read_when:
  - Quieres un diagnóstico rápido del estado de los canales y de los destinatarios de sesiones recientes
  - Quieres un estado “all” general, fácil de pegar, para depuración
---

<div id="openclaw-status">
  # `openclaw status`
</div>

Diagnóstico de canales y sesiones.

```bash
openclaw status
openclaw status --all
openclaw status --deep
openclaw status --usage
```

Notas:

* `--deep` ejecuta sondeos en vivo (WhatsApp Web + Telegram + Discord + Google Chat + Slack + Signal).
* La salida incluye almacenes de sesiones por agente cuando se configuran múltiples agentes.
* El resumen incluye el estado de instalación/ejecución del servicio host del Gateway y del nodo cuando está disponible.
* El resumen incluye el canal de actualización y el SHA de Git (para checkouts desde el código fuente).
* La información de actualización se muestra en el resumen; si hay una actualización disponible, `status` imprime una sugerencia para ejecutar `openclaw update` (consulta [Updating](/es/install/updating)).
