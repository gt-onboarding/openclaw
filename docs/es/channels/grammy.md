---
title: Grammy
summary: "Integración de Telegram Bot API mediante grammY con notas de configuración"
read_when:
  - Al trabajar en flujos de Telegram o grammY
---

<div id="grammy-integration-telegram-bot-api">
  # Integración con grammY (Telegram Bot API)
</div>

<div id="why-grammy">
  # Por qué grammY
</div>

- Cliente de Bot API orientado a TS con helpers integrados para long-polling + webhooks, middleware, manejo de errores y limitador de velocidad (rate limiter).
- Helpers de medios más limpios que implementar a mano con fetch + FormData; admite todos los métodos de Bot API.
- Extensible: soporte para proxy mediante fetch personalizado, middleware de sesión (opcional), contexto con tipado seguro.

<div id="what-we-shipped">
  # Lo que lanzamos
</div>

- **Ruta de cliente única:** se eliminó la implementación basada en `fetch`; grammY es ahora el único cliente de Telegram (send + gateway) con el limitador de velocidad (throttler) de grammY habilitado de forma predeterminada.
- **Gateway:** `monitorTelegramProvider` construye un `Bot` de grammY, conecta el filtrado por menciones/lista de permitidos, la descarga de medios vía `getFile`/`download`, y entrega respuestas con `sendMessage/sendPhoto/sendVideo/sendAudio/sendDocument`. Admite long‑polling o webhook mediante `webhookCallback`.
- **Proxy:** el `channels.telegram.proxy` opcional utiliza `undici.ProxyAgent` a través de `client.baseFetch` de grammY.
- **Compatibilidad con webhook:** `webhook-set.ts` envuelve `setWebhook/deleteWebhook`; `webhook.ts` expone el callback con comprobación de estado y apagado gradual. Gateway habilita el modo webhook cuando se establece `channels.telegram.webhookUrl` (de lo contrario, utiliza long‑polling).
- **Sesiones:** los chats directos se consolidan en la sesión principal del agente (`agent:&lt;agentId&gt;:&lt;mainKey&gt;`); los grupos usan `agent:&lt;agentId&gt;:telegram:group:&lt;chatId&gt;`; las respuestas se enrutan de vuelta al mismo canal.
- **Controles de configuración:** `channels.telegram.botToken`, `channels.telegram.dmPolicy`, `channels.telegram.groups` (lista de permitidos + valores predeterminados de mención), `channels.telegram.allowFrom`, `channels.telegram.groupAllowFrom`, `channels.telegram.groupPolicy`, `channels.telegram.mediaMaxMb`, `channels.telegram.linkPreview`, `channels.telegram.proxy`, `channels.telegram.webhookSecret`, `channels.telegram.webhookUrl`.
- **Streaming de borradores:** el `channels.telegram.streamMode` opcional utiliza `sendMessageDraft` en chats de temas privados (Bot API 9.3+). Esto es independiente del streaming de bloques de canal.
- **Pruebas:** los mocks de grammY cubren el filtrado por DM + mención en grupo y el envío saliente; aún son bienvenidos más fixtures de medios/webhook.

Preguntas abiertas

- Complementos opcionales de grammY (throttler) si encontramos respuestas 429 de la Bot API.
- Añadir pruebas de medios más estructuradas (stickers, notas de voz).
- Hacer configurable el puerto de escucha del webhook (actualmente fijado en 8787 a menos que se enrute a través del Gateway).