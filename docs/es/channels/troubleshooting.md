---
title: Solución de problemas
summary: "Atajos de solución de problemas específicos de cada canal (Discord/Telegram/WhatsApp)"
read_when:
  - Cuando un canal se conecta pero los mensajes no se envían
  - Cuando investigas una configuración incorrecta del canal (intents, permisos, modo de privacidad)
---

<div id="channel-troubleshooting">
  # Resolución de problemas de canales
</div>

Empieza por:

```bash
openclaw doctor
openclaw channels status --probe
```

`channels status --probe` muestra advertencias cuando detecta errores de configuración habituales en los canales e incluye pequeñas comprobaciones en tiempo real (credenciales, algunos permisos/membresías).

<div id="channels">
  ## Canales
</div>

* Discord: [/channels/discord#troubleshooting](/es/channels/discord#troubleshooting)
* Telegram: [/channels/telegram#troubleshooting](/es/channels/telegram#troubleshooting)
* WhatsApp: [/channels/whatsapp#troubleshooting-quick](/es/channels/whatsapp#troubleshooting-quick)

<div id="telegram-quick-fixes">
  ## Soluciones rápidas para Telegram
</div>

* Los registros muestran `HttpError: Network request for 'sendMessage' failed` o `sendChatAction` → revisa el DNS de IPv6. Si `api.telegram.org` se resuelve primero a IPv6 y el host no tiene conectividad saliente por IPv6, fuerza IPv4 o habilita IPv6. Consulta [/channels/telegram#troubleshooting](/es/channels/telegram#troubleshooting).
* Los registros muestran `setMyCommands failed` → comprueba la conectividad saliente HTTPS y DNS hacia `api.telegram.org` (habitual en VPS o proxies muy restringidos).