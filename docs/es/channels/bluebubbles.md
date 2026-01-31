---
title: Bluebubbles
summary: "iMessage mediante el servidor BlueBubbles para macOS (envío y recepción REST, indicadores de escritura, reacciones, emparejamiento, acciones avanzadas)."
read_when:
  - Configurar el canal de BlueBubbles
  - Solucionar problemas de emparejamiento del webhook
  - Configurar iMessage en macOS
---

<div id="bluebubbles-macos-rest">
  # BlueBubbles (macOS REST)
</div>

Estado: complemento integrado que se comunica con el servidor BlueBubbles de macOS mediante HTTP. **Recomendado para la integración con iMessage** debido a su API más completa y a una configuración más sencilla en comparación con el canal imsg legado.

<div id="overview">
  ## Descripción general
</div>

* Se ejecuta en macOS mediante la app auxiliar de BlueBubbles ([bluebubbles.app](https://bluebubbles.app)).
* Recomendado/probado: macOS Sequoia (15). macOS Tahoe (26) funciona; actualmente la función de edición no funciona en Tahoe y las actualizaciones de iconos de grupo pueden indicar que se realizaron correctamente pero no sincronizarse.
* OpenClaw se comunica con él a través de su API REST (`GET /api/v1/ping`, `POST /message/text`, `POST /chat/:id/*`).
* Los mensajes entrantes llegan mediante webhooks; las respuestas salientes, los indicadores de escritura, las confirmaciones de lectura y los tapbacks son llamadas REST.
* Los archivos adjuntos y stickers se procesan como contenido multimedia entrante (y se exponen al agente cuando es posible).
* El emparejamiento/lista de permitidos funciona igual que en otros canales (`/start/pairing`, etc.) con `channels.bluebubbles.allowFrom` + códigos de emparejamiento.
* Las reacciones se exponen como eventos del sistema igual que en Slack/Telegram, para que los agentes puedan &quot;mencionarlas&quot; antes de responder.
* Funciones avanzadas: editar, deshacer envío, hilos de respuesta, efectos de mensaje, gestión de grupos.

<div id="quick-start">
  ## Inicio rápido
</div>

1. Instala el servidor de BlueBubbles en tu Mac (sigue las instrucciones en [bluebubbles.app/install](https://bluebubbles.app/install)).
2. En la configuración de BlueBubbles, activa la API web y define una contraseña.
3. Ejecuta `openclaw onboard` y selecciona BlueBubbles, o configúralo manualmente:
   ```json5
   {
     channels: {
       bluebubbles: {
         enabled: true,
         serverUrl: "http://192.168.1.100:1234",
         password: "example-password",
         webhookPath: "/bluebubbles-webhook"
       }
     }
   }
   ```
4. Configura los webhooks de BlueBubbles para que apunten a tu Gateway (ejemplo: `https://your-gateway-host:3000/bluebubbles-webhook?password=<password>`).
5. Inicia el Gateway; registrará el controlador del webhook y comenzará el emparejamiento.

<div id="onboarding">
  ## Introducción
</div>

BlueBubbles está disponible en el asistente interactivo de configuración:

```
openclaw onboard
```

El asistente solicita:

* **Server URL** (obligatorio): Dirección del servidor BlueBubbles (por ejemplo, `http://192.168.1.100:1234`)
* **Password** (obligatorio): Contraseña de la API en la configuración de BlueBubbles Server
* **Webhook path** (opcional): El valor predeterminado es `/bluebubbles-webhook`
* **DM policy**: pairing, allowlist, open (ajuste que permite la aceptación de mensajes sin restricciones desde cualquier usuario) o disabled
* **Allow list**: Lista de permitidos: números de teléfono, direcciones de correo electrónico o destinos de chat

También puedes agregar BlueBubbles mediante la CLI:

```
openclaw channels add bluebubbles --http-url http://192.168.1.100:1234 --password <password>
```

<div id="access-control-dms-groups">
  ## Control de acceso (DMs + grupos)
</div>

DMs:

* Valor predeterminado: `channels.bluebubbles.dmPolicy = "pairing"`.
* Los remitentes desconocidos reciben un código de emparejamiento; los mensajes se ignoran hasta que los apruebes (los códigos expiran después de 1 hora).
* Aprobar mediante:
  * `openclaw pairing list bluebubbles`
  * `openclaw pairing approve bluebubbles <CODE>`
* El emparejamiento es el intercambio de tokens predeterminado. Detalles: [Pairing](/es/start/pairing)

Grupos:

* `channels.bluebubbles.groupPolicy = open | allowlist | disabled` (valor predeterminado: `allowlist`).
* `channels.bluebubbles.groupAllowFrom` controla quién puede activar en grupos cuando `allowlist` está establecido.

<div id="mention-gating-groups">
  ### Restricción por menciones (grupos)
</div>

BlueBubbles admite la restricción por menciones para chats de grupo, replicando el comportamiento de iMessage/WhatsApp:

* Usa `agents.list[].groupChat.mentionPatterns` (o `messages.groupChat.mentionPatterns`) para detectar menciones.
* Cuando `requireMention` está habilitado para un grupo, el agente solo responde cuando se le menciona.
* Los comandos de control de remitentes autorizados no están sujetos a la restricción por menciones.

Configuración por grupo:

```json5
{
  channels: {
    bluebubbles: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15555550123"],
      groups: {
        "*": { requireMention: true },  // default for all groups
        "iMessage;-;chat123": { requireMention: false }  // anulación para grupo específico
      }
    }
  }
}
```

<div id="command-gating">
  ### Control de acceso a comandos
</div>

* Los comandos de control (por ejemplo, `/config`, `/model`) requieren autorización.
* Utiliza `allowFrom` y `groupAllowFrom` para determinar si un comando está autorizado.
* Los remitentes autorizados pueden ejecutar comandos de control incluso sin ser mencionados en grupos.

<div id="typing-read-receipts">
  ## Indicadores de escritura y confirmaciones de lectura
</div>

* **Indicadores de escritura**: Se envían automáticamente antes y durante la generación de la respuesta.
* **Confirmaciones de lectura**: Se controlan con `channels.bluebubbles.sendReadReceipts` (valor predeterminado: `true`).
* **Indicadores de escritura**: OpenClaw envía eventos de inicio de escritura; BlueBubbles borra el estado de escritura automáticamente al enviar o al agotarse el tiempo de espera (la detención manual mediante DELETE no es fiable).

```json5
{
  channels: {
    bluebubbles: {
      sendReadReceipts: false  // deshabilitar confirmaciones de lectura
    }
  }
}
```

<div id="advanced-actions">
  ## Acciones avanzadas
</div>

BlueBubbles admite acciones avanzadas para mensajes cuando se habilitan en la configuración:

```json5
{
  channels: {
    bluebubbles: {
      actions: {
        reactions: true,       // tapbacks (default: true)
        edit: true,            // editar mensajes enviados (macOS 13+, no funciona en macOS 26 Tahoe)
        unsend: true,          // unsend messages (macOS 13+)
        reply: true,           // reply threading by message GUID
        sendWithEffect: true,  // message effects (slam, loud, etc.)
        renameGroup: true,     // rename group chats
        setGroupIcon: true,    // set group chat icon/photo (flaky on macOS 26 Tahoe)
        addParticipant: true,  // add participants to groups
        removeParticipant: true, // remove participants from groups
        leaveGroup: true,      // leave group chats
        sendAttachment: true   // send attachments/media
      }
    }
  }
}
```

Acciones disponibles:

* **react**: Agregar/eliminar reacciones de tapback (`messageId`, `emoji`, `remove`)
* **edit**: Editar un mensaje enviado (`messageId`, `text`)
* **unsend**: Anular el envío de un mensaje (`messageId`)
* **reply**: Responder a un mensaje específico (`messageId`, `text`, `to`)
* **sendWithEffect**: Enviar con efecto de iMessage (`text`, `to`, `effectId`)
* **renameGroup**: Renombrar un chat grupal (`chatGuid`, `displayName`)
* **setGroupIcon**: Establecer el ícono/foto de un chat grupal (`chatGuid`, `media`) — poco confiable en macOS 26 Tahoe (la API puede devolver éxito, pero el ícono no se sincroniza).
* **addParticipant**: Agregar a alguien a un grupo (`chatGuid`, `address`)
* **removeParticipant**: Eliminar a alguien de un grupo (`chatGuid`, `address`)
* **leaveGroup**: Salir de un chat grupal (`chatGuid`)
* **sendAttachment**: Enviar contenido multimedia/archivos (`to`, `buffer`, `filename`, `asVoice`)
  * Mensajes de voz: configura `asVoice: true` con audio **MP3** o **CAF** para enviarlo como un mensaje de voz de iMessage. BlueBubbles convierte MP3 → CAF al enviar mensajes de voz.

<div id="message-ids-short-vs-full">
  ### IDs de mensaje (cortos vs completos)
</div>

OpenClaw puede mostrar IDs de mensaje *cortos* (p. ej., `1`, `2`) para ahorrar tokens.

* `MessageSid` / `ReplyToId` pueden ser IDs cortos.
* `MessageSidFull` / `ReplyToIdFull` contienen los IDs completos del proveedor.
* Los IDs cortos están solo en memoria; pueden caducar al reiniciar o al vaciar la caché.
* Las acciones aceptan `messageId` corto o completo, pero los IDs cortos producirán un error si ya no están disponibles.

Usa IDs completos para automatizaciones y almacenamiento duraderos:

* Plantillas: `{{MessageSidFull}}`, `{{ReplyToIdFull}}`
* Contexto: `MessageSidFull` / `ReplyToIdFull` en los payloads de entrada

Consulta [Configuración](/es/gateway/configuration) para ver las variables de plantilla.

<div id="block-streaming">
  ## Transmisión en bloques
</div>

Controla si las respuestas se envían como un único mensaje o se transmiten en bloques:

```json5
{
  channels: {
    bluebubbles: {
      blockStreaming: true  // habilitar transmisión por bloques (comportamiento predeterminado)
    }
  }
}
```

<div id="media-limits">
  ## Contenido multimedia + límites
</div>

* Los archivos adjuntos recibidos se descargan y almacenan en la caché multimedia.
* Límite de tamaño de contenido multimedia mediante `channels.bluebubbles.mediaMaxMb` (valor predeterminado: 8 MB).
* El texto saliente se divide en fragmentos según `channels.bluebubbles.textChunkLimit` (valor predeterminado: 4000 caracteres).

<div id="configuration-reference">
  ## Referencia de configuración
</div>

Configuración completa: [Configuration](/es/gateway/configuration)

Opciones del proveedor:

* `channels.bluebubbles.enabled`: Habilitar/deshabilitar el canal.
* `channels.bluebubbles.serverUrl`: URL base de la API REST de BlueBubbles.
* `channels.bluebubbles.password`: Contraseña de la API.
* `channels.bluebubbles.webhookPath`: Ruta del endpoint del webhook (valor predeterminado: `/bluebubbles-webhook`).
* `channels.bluebubbles.dmPolicy`: `pairing | allowlist | open | disabled` (valor predeterminado: `pairing`).
* `channels.bluebubbles.allowFrom`: Lista de permitidos para MD (handles, correos electrónicos, números E.164, `chat_id:*`, `chat_guid:*`).
* `channels.bluebubbles.groupPolicy`: `open | allowlist | disabled` (valor predeterminado: `allowlist`).
* `channels.bluebubbles.groupAllowFrom`: Lista de permitidos de remitentes de grupos.
* `channels.bluebubbles.groups`: Configuración por grupo (`requireMention`, etc.).
* `channels.bluebubbles.sendReadReceipts`: Enviar confirmaciones de lectura (valor predeterminado: `true`).
* `channels.bluebubbles.blockStreaming`: Habilitar transmisión por bloques (valor predeterminado: `true`).
* `channels.bluebubbles.textChunkLimit`: Tamaño de bloque saliente en caracteres (valor predeterminado: 4000).
* `channels.bluebubbles.chunkMode`: `length` (predeterminado) divide solo cuando se excede `textChunkLimit`; `newline` divide en líneas en blanco (límites de párrafo) antes de la segmentación por longitud.
* `channels.bluebubbles.mediaMaxMb`: Límite de medios entrantes en MB (valor predeterminado: 8).
* `channels.bluebubbles.historyLimit`: Máximo de mensajes de grupo para contexto (0 lo deshabilita).
* `channels.bluebubbles.dmHistoryLimit`: Límite de historial de MD.
* `channels.bluebubbles.actions`: Habilitar/deshabilitar acciones específicas.
* `channels.bluebubbles.accounts`: Configuración multicuenta.

Opciones globales relacionadas:

* `agents.list[].groupChat.mentionPatterns` (o `messages.groupChat.mentionPatterns`).
* `messages.responsePrefix`.

<div id="addressing-delivery-targets">
  ## Direccionamiento / destinos de entrega
</div>

Prefiere usar `chat_guid` para un enrutamiento estable:

* `chat_guid:iMessage;-;+15555550123` (preferido para grupos)
* `chat_id:123`
* `chat_identifier:...`
* Handles directos: `+15555550123`, `user@example.com`
  * Si un handle directo no tiene un chat de mensaje directo (DM) existente, OpenClaw creará uno mediante `POST /api/v1/chat/new`. Esto requiere que la BlueBubbles Private API esté habilitada.

<div id="security">
  ## Seguridad
</div>

* Las solicitudes de webhook se autentican comparando los parámetros de consulta o encabezados `guid`/`password` con `channels.bluebubbles.password`. También se aceptan solicitudes desde `localhost`.
* Mantén en secreto la contraseña de la API y el endpoint de webhook (trátalos como credenciales).
* La confianza en localhost implica que un reverse proxy en el mismo host puede eludir la contraseña de forma no intencionada. Si pones el Gateway detrás de un proxy, exige autenticación en el proxy y configura `gateway.trustedProxies`. Consulta [Seguridad del Gateway](/es/gateway/security#reverse-proxy-configuration).
* Habilita HTTPS y reglas de firewall en el servidor de BlueBubbles si lo expones fuera de tu LAN.

<div id="troubleshooting">
  ## Solución de problemas
</div>

* Si los eventos de escritura/read dejan de funcionar, revisa los registros del webhook de BlueBubbles y verifica que la ruta del Gateway coincida con `channels.bluebubbles.webhookPath`.
* Los códigos de emparejamiento caducan después de una hora; usa `openclaw pairing list bluebubbles` y `openclaw pairing approve bluebubbles <code>`.
* Las reacciones requieren la API privada de BlueBubbles (`POST /api/v1/message/react`); asegúrate de que la versión del servidor la exponga.
* Editar/anular envío (unsend) requiere macOS 13+ y una versión compatible del servidor BlueBubbles. En macOS 26 (Tahoe), la edición no funciona actualmente debido a cambios en la API privada.
* Las actualizaciones del icono de grupo pueden ser inestables en macOS 26 (Tahoe): la API puede devolver éxito pero el nuevo icono no se sincroniza.
* OpenClaw oculta automáticamente las acciones conocidas como defectuosas según la versión de macOS del servidor BlueBubbles. Si la edición sigue apareciendo en macOS 26 (Tahoe), desactívala manualmente con `channels.bluebubbles.actions.edit=false`.
* Para información sobre estado y salud: `openclaw status --all` o `openclaw status --deep`.

Para una referencia general del flujo de trabajo de canales, consulta [Channels](/es/channels) y la guía de [Plugins](/es/plugins).