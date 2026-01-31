---
title: Estado de salud
summary: "Pasos de comprobación de salud para la conectividad del canal"
read_when:
  - Diagnóstico del estado del canal de WhatsApp
---

<div id="health-checks-cli">
  # Comprobaciones de estado (CLI)
</div>

Guía breve para comprobar la conectividad de los canales sin conjeturas.

<div id="quick-checks">
  ## Comprobaciones rápidas
</div>

- `openclaw status` — resumen local: conectividad/alcance del Gateway y modo, sugerencia de actualización, antigüedad de la autenticación de canales enlazados, sesiones + actividad reciente.
- `openclaw status --all` — diagnóstico local completo (solo lectura, con color, seguro para pegar en informes de depuración).
- `openclaw status --deep` — también sondea el Gateway en ejecución (sondeos por canal cuando sea compatible).
- `openclaw health --json` — solicita al Gateway en ejecución una instantánea completa de estado de salud (solo WS; sin socket Baileys directo).
- Envía `/status` como mensaje independiente en WhatsApp/WebChat para obtener una respuesta de estado sin invocar al agente.
- Registros: usa `tail` sobre `/tmp/openclaw/openclaw-*.log` y filtra por `web-heartbeat`, `web-reconnect`, `web-auto-reply`, `web-inbound`.

<div id="deep-diagnostics">
  ## Diagnósticos avanzados
</div>

- Credenciales en disco: `ls -l ~/.openclaw/credentials/whatsapp/<accountId>/creds.json` (la marca de tiempo mtime debería ser reciente).
- Almacén de sesiones: `ls -l ~/.openclaw/agents/<agentId>/sessions/sessions.json` (la ruta se puede anular en la configuración). El número total y los destinatarios recientes se muestran a través de `status`.
- Flujo de reconexión: `openclaw channels logout && openclaw channels login --verbose` cuando aparezcan en los logs códigos de estado 409–515 o `loggedOut`. (Nota: el flujo de inicio de sesión con QR se reinicia automáticamente una vez para el estado 515 después del emparejamiento).

<div id="when-something-fails">
  ## Cuando algo falla
</div>

- `logged out` o código de estado 409–515 → vuelve a vincular con `openclaw channels logout` y luego `openclaw channels login`.
- Gateway inaccesible → inícialo: `openclaw gateway --port 18789` (usa `--force` si el puerto está ocupado).
- Sin mensajes entrantes → confirma que el teléfono vinculado está en línea y que el remitente está en la lista de permitidos (`channels.whatsapp.allowFrom`); para chats grupales, asegúrate de que la lista de permitidos y las reglas de mención coincidan (`channels.whatsapp.groups`, `agents.list[].groupChat.mentionPatterns`).

<div id="dedicated-health-command">
  ## Comando dedicado "health"
</div>

`openclaw health --json` consulta al Gateway en ejecución para obtener una instantánea de su estado de salud (sin sockets de canal directos desde la CLI). Informa la antigüedad de las credenciales/autenticación vinculadas cuando estén disponibles, resúmenes de sondeo por canal, un resumen del almacén de sesiones y la duración del sondeo. Sale con un código de salida distinto de cero si el Gateway no es accesible o si el sondeo falla/se agota el tiempo. Usa `--timeout <ms>` para reemplazar el valor predeterminado de 10 s.