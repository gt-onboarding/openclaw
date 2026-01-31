---
title: Reacciones
summary: "Semántica de las reacciones compartida entre canales"
read_when:
  - Al trabajar con reacciones en cualquier canal
---

<div id="reaction-tooling">
  # Herramientas de reacciones
</div>

Semántica común de reacciones entre canales:

- `emoji` es obligatorio al añadir una reacción.
- `emoji=""` elimina las reacciones del bot cuando el canal lo admite.
- `remove: true` elimina el emoji especificado cuando el canal lo admite (requiere `emoji`).

Notas por canal:

- **Discord/Slack**: un `emoji` vacío elimina todas las reacciones del bot en el mensaje; `remove: true` elimina solo ese emoji.
- **Google Chat**: un `emoji` vacío elimina las reacciones de la app en el mensaje; `remove: true` elimina solo ese emoji.
- **Telegram**: un `emoji` vacío elimina las reacciones del bot; `remove: true` también elimina reacciones, pero sigue requiriendo un `emoji` no vacío para la validación de la herramienta.
- **WhatsApp**: un `emoji` vacío elimina la reacción del bot; `remove: true` equivale a un emoji vacío (sigue requiriendo `emoji`).
- **Signal**: las notificaciones de reacciones entrantes emiten eventos del sistema cuando `channels.signal.reactionNotifications` está habilitado.