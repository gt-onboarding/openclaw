---
title: Refactorización del mirroring de sesiones salientes (Issue #1520)
description: Hacer seguimiento de notas, decisiones, pruebas y temas abiertos de la refactorización del mirroring de sesiones salientes.
---

<div id="outbound-session-mirroring-refactor-issue-1520">
  # Refactorización de la replicación de sesiones salientes (Issue #1520)
</div>

<div id="status">
  ## Estado
</div>

- En curso.
- Enrutamiento de canales del núcleo y complementos actualizado para el mirroring de salida.
- El método send de Gateway ahora deriva la sesión de destino cuando se omite sessionKey.

<div id="context">
  ## Contexto
</div>

Los sends salientes se replicaban en la sesión de agente *actual* (clave de sesión de herramienta) en lugar de en la sesión del canal de destino. El enrutamiento entrante usa claves de sesión de canal/peer, por lo que las respuestas salientes terminaban en la sesión incorrecta y los destinos de primer contacto a menudo carecían de entradas de sesión.

<div id="goals">
  ## Objetivos
</div>

- Reflejar los mensajes salientes en la clave de sesión del canal de destino.
- Crear entradas de sesión para los mensajes salientes cuando no existan.
- Mantener el alcance de los hilos/temas alineado con las claves de sesión entrantes.
- Cubrir los canales principales más las extensiones incluidas.

<div id="implementation-summary">
  ## Resumen de la implementación
</div>

- Nueva utilidad de enrutamiento de sesiones salientes:
  - `src/infra/outbound/outbound-session.ts`
  - `resolveOutboundSessionRoute` construye el `sessionKey` de destino usando `buildAgentSessionKey` (dmScope + identityLinks).
  - `ensureOutboundSessionEntry` escribe un `MsgContext` mínimo mediante `recordSessionMetaFromInbound`.
- `runMessageAction` (send) deriva el `sessionKey` de destino y se lo pasa a `executeSendAction` para la replicación.
- `message-tool` ya no replica directamente; solo resuelve `agentId` a partir de la clave de sesión actual.
- La ruta de send del complemento replica mediante `appendAssistantMessageToSessionTranscript` usando el `sessionKey` derivado.
- El send del Gateway deriva una clave de sesión de destino cuando no se proporciona ninguna (agente predeterminado) y garantiza una entrada de sesión.

<div id="threadtopic-handling">
  ## Gestión de hilos/temas
</div>

- Slack: replyTo/threadId -> `resolveThreadSessionKeys` (sufijo).
- Discord: threadId/replyTo -> `resolveThreadSessionKeys` con `useSuffix=false` para coincidir con el tráfico entrante (el ID del canal del hilo ya define el ámbito de la sesión).
- Telegram: los IDs de tema se asignan a `chatId:topic:<id>` mediante `buildTelegramGroupPeerId`.

<div id="extensions-covered">
  ## Extensiones cubiertas
</div>

- Matrix, MS Teams, Mattermost, BlueBubbles, Nextcloud Talk, Zalo, Zalo Personal, Nostr, Tlon.
- Notas:
  - Los destinos de Mattermost ahora eliminan `@` para el enrutamiento de claves de sesión de DM.
  - Zalo Personal usa el tipo de par de DM para destinos 1:1 (solo grupo cuando `group:` está presente).
  - Los destinos de grupo de BlueBubbles eliminan los prefijos `chat_*` para que coincidan con las claves de sesión entrantes.
  - El mirroring automático de hilos de Slack hace coincidir los ID de canal sin distinguir mayúsculas de minúsculas.
  - Gateway send convierte a minúsculas las claves de sesión proporcionadas antes de realizar el mirroring.

<div id="decisions">
  ## Decisiones
</div>

- **Derivación de sesión en Gateway send**: si se proporciona `sessionKey`, úsala. Si se omite, deriva una sessionKey a partir del destino + agente predeterminado y refleja la sesión allí.
- **Creación de entrada de sesión**: usa siempre `recordSessionMetaFromInbound` con `Provider/From/To/ChatType/AccountId/Originating*` alineados con los formatos de entrada.
- **Normalización de destino**: el enrutamiento saliente usa destinos resueltos (después de `resolveChannelTarget`) cuando están disponibles.
- **Convención de mayúsculas/minúsculas en la clave de sesión**: normaliza las claves de sesión a minúsculas al escribir y durante las migraciones.

<div id="tests-addedupdated">
  ## Pruebas añadidas/actualizadas
</div>

- `src/infra/outbound/outbound-session.test.ts`
  - Clave de sesión de hilo de Slack.
  - Clave de sesión de tema de Telegram.
  - identityLinks de dmScope con Discord.
- `src/agents/tools/message-tool.test.ts`
  - Deriva el agentId a partir de la clave de sesión (sin sessionKey transmitida).
- `src/gateway/server-methods/send.test.ts`
  - Deriva la clave de sesión cuando se omite y crea la entrada de sesión.

<div id="open-items-follow-ups">
  ## Elementos pendientes / Seguimiento
</div>

- El complemento de llamada de voz usa claves de sesión personalizadas `voice:&lt;phone&gt;`. La asignación de salida no está estandarizada aquí; si `message-tool` debe admitir el envío de llamadas de voz, agrega una asignación explícita.
- Confirma si algún complemento externo usa formatos `From/To` no estándar más allá del conjunto incluido.

<div id="files-touched">
  ## Archivos modificados
</div>

- `src/infra/outbound/outbound-session.ts`
- `src/infra/outbound/outbound-send-service.ts`
- `src/infra/outbound/message-action-runner.ts`
- `src/agents/tools/message-tool.ts`
- `src/gateway/server-methods/send.ts`
- Pruebas en:
  - `src/infra/outbound/outbound-session.test.ts`
  - `src/agents/tools/message-tool.test.ts`
  - `src/gateway/server-methods/send.test.ts`