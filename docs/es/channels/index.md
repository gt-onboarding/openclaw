---
title: Canales
summary: "Plataformas de mensajería a las que OpenClaw puede conectarse"
read_when:
  - Quieres elegir un canal de chat para OpenClaw
  - Necesitas un resumen rápido de las plataformas de mensajería compatibles
---

<div id="chat-channels">
  # Canales de chat
</div>

OpenClaw puede hablar contigo en cualquier aplicación de chat que ya uses. Cada canal se conecta a través del Gateway.
El texto se admite en todos los canales; el soporte de contenido multimedia y de reacciones varía según el canal.

<div id="supported-channels">
  ## Canales compatibles
</div>

* [WhatsApp](/es/channels/whatsapp) — El más popular; usa Baileys y requiere emparejamiento por QR.
* [Telegram](/es/channels/telegram) — Bot API vía grammY; admite grupos.
* [Discord](/es/channels/discord) — Discord Bot API + Gateway; admite servidores, canales y mensajes directos.
* [Slack](/es/channels/slack) — SDK Bolt; aplicaciones de espacio de trabajo.
* [Google Chat](/es/channels/googlechat) — Aplicación de Google Chat API vía webhook HTTP.
* [Mattermost](/es/channels/mattermost) — Bot API + WebSocket; canales, grupos, mensajes directos (complemento, se instala por separado).
* [Signal](/es/channels/signal) — signal-cli; centrado en la privacidad.
* [BlueBubbles](/es/channels/bluebubbles) — **Recomendado para iMessage**; usa la REST API del servidor BlueBubbles en macOS con compatibilidad completa de funcionalidades (editar, anular envío, efectos, reacciones, gestión de grupos — actualmente la edición no funciona en macOS 26 Tahoe).
* [iMessage](/es/channels/imessage) — Solo macOS; integración nativa vía imsg (heredada, considera BlueBubbles para nuevas instalaciones).
* [Microsoft Teams](/es/channels/msteams) — Bot Framework; compatibilidad empresarial (complemento, se instala por separado).
* [LINE](/es/channels/line) — Bot de LINE Messaging API (complemento, se instala por separado).
* [Nextcloud Talk](/es/channels/nextcloud-talk) — Chat autoalojado vía Nextcloud Talk (complemento, se instala por separado).
* [Matrix](/es/channels/matrix) — Protocolo Matrix (complemento, se instala por separado).
* [Nostr](/es/channels/nostr) — Mensajes directos descentralizados vía NIP-04 (complemento, se instala por separado).
* [Tlon](/es/channels/tlon) — Mensajería basada en Urbit (complemento, se instala por separado).
* [Twitch](/es/channels/twitch) — Chat de Twitch vía conexión IRC (complemento, se instala por separado).
* [Zalo](/es/channels/zalo) — Zalo Bot API; la aplicación de mensajería más popular de Vietnam (complemento, se instala por separado).
* [Zalo Personal](/es/channels/zalouser) — Cuenta personal de Zalo vía inicio de sesión por QR (complemento, se instala por separado).
* [WebChat](/es/web/webchat) — UI WebChat de Gateway vía WebSocket.

<div id="notes">
  ## Notas
</div>

* Los canales pueden ejecutarse simultáneamente; configura varios y OpenClaw se encargará de enrutar por chat.
* La configuración más rápida suele ser **Telegram** (token de bot sencillo). WhatsApp requiere emparejamiento mediante código QR y almacena más estado en disco.
* El comportamiento en grupos varía según el canal; consulta [Grupos](/es/concepts/groups).
* El emparejamiento por DM y las listas de permitidos se aplican por motivos de seguridad; consulta [Seguridad](/es/gateway/security).
* Detalles internos de Telegram: [Notas sobre grammY](/es/channels/grammy).
* Resolución de problemas: [Solución de problemas de canales](/es/channels/troubleshooting).
* Los proveedores de modelos se documentan por separado; consulta [Proveedores de modelos](/es/providers/models).