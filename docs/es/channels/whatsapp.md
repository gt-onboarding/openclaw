---
title: WhatsApp
summary: "Integración de WhatsApp (canal web): inicio de sesión, bandeja de entrada, respuestas, contenido multimedia y operaciones"
read_when:
  - Cuando trabajes en el comportamiento del canal web de WhatsApp o en el enrutamiento de la bandeja de entrada
---

<div id="whatsapp-web-channel">
  # WhatsApp (canal web)
</div>

Estado: WhatsApp Web solo a través de Baileys. Gateway es el propietario de las sesiones.

<div id="quick-setup-beginner">
  ## Configuración rápida (principiante)
</div>

1. Usa un **número de teléfono distinto** si es posible (recomendado).
2. Configura WhatsApp en `~/.openclaw/openclaw.json`.
3. Ejecuta `openclaw channels login` para escanear el código QR (Dispositivos vinculados).
4. Inicia el Gateway.

Configuración mínima:

```json5
{
  channels: {
    whatsapp: {
      dmPolicy: "allowlist",
      allowFrom: ["+15551234567"]
    }
  }
}
```

<div id="goals">
  ## Objetivos
</div>

* Varias cuentas de WhatsApp (multicuenta) en un único proceso de Gateway.
* Enrutamiento determinista: las respuestas vuelven a WhatsApp, sin enrutamiento por modelo.
* El modelo dispone de suficiente contexto para entender respuestas citadas.

<div id="config-writes">
  ## Escrituras de configuración
</div>

De forma predeterminada, WhatsApp tiene permiso para escribir actualizaciones de configuración iniciadas mediante `/config set|unset` (requiere `commands.config: true`).

Desactiva esto con:

```json5
{
  channels: { whatsapp: { configWrites: false } }
}
```

<div id="architecture-who-owns-what">
  ## Arquitectura (quién controla qué)
</div>

* **Gateway** controla el socket de Baileys y el bucle de bandeja de entrada.
* **CLI / app de macOS** se comunican con el Gateway; no usan Baileys directamente.
* **Active listener** es obligatorio para los envíos salientes; de lo contrario, los envíos fallan rápidamente.

<div id="getting-a-phone-number-two-modes">
  ## Obtener un número de teléfono (dos modos)
</div>

WhatsApp requiere un número de teléfono móvil real para la verificación. Los números VoIP y virtuales suelen estar bloqueados. Hay dos modos admitidos para ejecutar OpenClaw en WhatsApp:

<div id="dedicated-number-recommended">
  ### Número dedicado (recomendado)
</div>

Usa un **número de teléfono separado** para OpenClaw. Mejor experiencia de usuario, enrutamiento limpio, sin rarezas de chatear contigo mismo. Configuración ideal: **teléfono Android de repuesto/viejo + eSIM**. Déjalo con Wi‑Fi y conectado a la corriente, y vincúlalo mediante código QR.

**WhatsApp Business:** Puedes usar WhatsApp Business en el mismo dispositivo con un número diferente. Ideal para mantener tu WhatsApp personal separado: instala WhatsApp Business y registra allí el número de OpenClaw.

**Configuración de ejemplo (número dedicado, lista de permitidos con un único usuario):**

```json5
{
  channels: {
    whatsapp: {
      dmPolicy: "allowlist",
      allowFrom: ["+15551234567"]
    }
  }
}
```

**Modo de emparejamiento (opcional):**
Si prefieres el emparejamiento en lugar de la lista de permitidos, configura `channels.whatsapp.dmPolicy` a `pairing`. Los remitentes desconocidos recibirán un código de emparejamiento; apruébalo con:
`openclaw pairing approve whatsapp <code>`

<div id="personal-number-fallback">
  ### Número personal (respaldo)
</div>

Respaldo rápido: ejecuta OpenClaw con **tu propio número**. Envíate mensajes a ti mismo (función de WhatsApp “Message yourself”) para hacer pruebas y no hacer spam a tus contactos. Ten en cuenta que tendrás que leer códigos de verificación en tu teléfono principal durante la configuración y las pruebas. **Debes activar el modo de chat contigo mismo.**
Cuando el asistente de configuración pida tu número personal de WhatsApp, introduce el teléfono desde el que enviarás los mensajes (el propietario/remitente), no el número del asistente.

**Configuración de ejemplo (número personal, chat contigo mismo):**

```json
{
  "whatsapp": {
    "selfChatMode": true,
    "dmPolicy": "allowlist",
    "allowFrom": ["+15551234567"]
  }
}
```

Las respuestas de autochat usan por defecto `[{identity.name}]` cuando ese valor está definido (de lo contrario `[openclaw]`)
si `messages.responsePrefix` no está configurado. Configúralo explícitamente para personalizar o desactivar
el prefijo (usa `""` para eliminarlo).

<div id="number-sourcing-tips">
  ### Consejos para obtener un número
</div>

* **eSIM local** de la operadora de telefonía móvil de tu país (lo más fiable)
  * Austria: [hot.at](https://www.hot.at)
  * Reino Unido: [giffgaff](https://www.giffgaff.com) — SIM gratuita, sin contrato
* **SIM prepago** — barata, solo necesita recibir un SMS para la verificación

**Evita:** TextNow, Google Voice, la mayoría de servicios de &quot;SMS gratis&quot; — WhatsApp los bloquea agresivamente.

**Consejo:** El número solo necesita recibir un SMS de verificación. Después de eso, las sesiones de WhatsApp Web persisten mediante `creds.json`.

<div id="why-not-twilio">
  ## ¿Por qué no Twilio?
</div>

* Las primeras versiones de OpenClaw admitían la integración con WhatsApp Business de Twilio.
* Los números de WhatsApp Business no son adecuados para un asistente personal.
* Meta aplica una ventana de respuesta de 24 horas; si no has respondido en las últimas 24 horas, el número empresarial no puede iniciar nuevas conversaciones.
* Un uso de alto volumen o “muy activo en el chat” provoca bloqueos agresivos, ya que las cuentas empresariales no están diseñadas para enviar decenas de mensajes de un asistente personal.
* Resultado: entregas poco fiables y bloqueos frecuentes, por lo que se retiró esa compatibilidad.

<div id="login-credentials">
  ## Inicio de sesión + credenciales
</div>

* Comando de inicio de sesión: `openclaw channels login` (QR vía Dispositivos vinculados).
* Inicio de sesión con múltiples cuentas: `openclaw channels login --account <id>` (`<id>` = `accountId`).
* Cuenta predeterminada (cuando se omite `--account`): `default` si existe; en caso contrario, el primer identificador de cuenta configurado (en orden).
* Credenciales almacenadas en `~/.openclaw/credentials/whatsapp/<accountId>/creds.json`.
* Copia de seguridad en `creds.json.bak` (restaurada en caso de corrupción).
* Compatibilidad con versiones anteriores: instalaciones antiguas almacenaban archivos de Baileys directamente en `~/.openclaw/credentials/`.
* Cierre de sesión: `openclaw channels logout` (o `--account <id>`) elimina el estado de autenticación de WhatsApp (pero conserva el `oauth.json` compartido).
* Socket con sesión cerrada =&gt; error que indica que vuelvas a vincular.

<div id="inbound-flow-dm-group">
  ## Flujo entrante (DM + grupo)
</div>

* Los eventos de WhatsApp provienen de `messages.upsert` (Baileys).
* Los listeners de la bandeja de entrada se desregistran al apagarse el servicio para evitar acumular manejadores de eventos en pruebas o reinicios.
* Se ignoran los chats de estado/difusión.
* Los chats directos usan E.164; los grupos usan JID de grupo.
* **Política de DM**: `channels.whatsapp.dmPolicy` controla el acceso a chats directos (valor predeterminado: `pairing`).
  * Emparejamiento: los remitentes desconocidos reciben un código de emparejamiento (apruébalo mediante `openclaw pairing approve whatsapp <code>`; los códigos expiran después de 1 hora).
  * open: esta configuración permite aceptar mensajes sin restricciones de cualquier usuario y requiere que `channels.whatsapp.allowFrom` incluya `"*"`.
  * Tu número de WhatsApp vinculado se considera implícitamente de confianza, por lo que los mensajes que te envías a ti mismo omiten las comprobaciones de `channels.whatsapp.dmPolicy` y `channels.whatsapp.allowFrom`.

<div id="personal-number-mode-fallback">
  ### Modo de número personal (fallback)
</div>

Si ejecutas OpenClaw en tu **número personal de WhatsApp**, habilita `channels.whatsapp.selfChatMode` (ver ejemplo anterior).

Comportamiento:

* Los mensajes directos (DM) salientes nunca generan respuestas de emparejamiento (evita enviar spam a tus contactos).
* Los remitentes desconocidos entrantes siguen la configuración de `channels.whatsapp.dmPolicy`.
* El modo self-chat (allowFrom incluye tu número) evita los acuses de lectura automáticos e ignora los JID de menciones.
* Se envían acuses de lectura para mensajes directos que no son self-chat.

<div id="read-receipts">
  ## Confirmaciones de lectura
</div>

De forma predeterminada, el Gateway marca los mensajes entrantes de WhatsApp como leídos (palomitas azules) cuando los acepta.

Desactivar globalmente:

```json5
{
  channels: { whatsapp: { sendReadReceipts: false } }
}
```

Desactivar por cada cuenta:

```json5
{
  channels: {
    whatsapp: {
      accounts: {
        personal: { sendReadReceipts: false }
      }
    }
  }
}
```

Notas:

* El modo de chat contigo mismo no envía confirmaciones de lectura.

<div id="whatsapp-faq-sending-messages-pairing">
  ## Preguntas frecuentes de WhatsApp: envío de mensajes + emparejamiento
</div>

**¿OpenClaw enviará mensajes a contactos aleatorios cuando vincule WhatsApp?**\
No. La política de DM predeterminada es **emparejamiento**, por lo que los remitentes desconocidos solo reciben un código de emparejamiento y su mensaje **no se procesa**. OpenClaw solo responde a los chats que recibe o a envíos que inicies explícitamente (agente/CLI).

**¿Cómo funciona el emparejamiento en WhatsApp?**\
El emparejamiento es un filtro de DM para remitentes desconocidos:

* El primer DM de un nuevo remitente devuelve un código corto (el mensaje no se procesa).
* Aprueba con: `openclaw pairing approve whatsapp <code>` (lista con `openclaw pairing list whatsapp`).
* Los códigos caducan después de 1 hora; las solicitudes pendientes se limitan a 3 por canal.

**¿Pueden varias personas usar distintas instancias de OpenClaw en un mismo número de WhatsApp?**\
Sí, enrutando cada remitente a un agente diferente mediante `bindings` (par `kind: "dm"`, remitente E.164 como `+15551234567`). Las respuestas siguen llegando desde la **misma cuenta de WhatsApp**, y los chats directos se agrupan en la sesión principal de cada agente, así que usa **un agente por persona**. El control de acceso de DM (`dmPolicy`/`allowFrom`) es global por cuenta de WhatsApp. Consulta [Enrutamiento multiagente](/es/concepts/multi-agent).

**¿Por qué me piden mi número de teléfono en el asistente?**\
El asistente lo usa para configurar tu **lista de permitidos/propietario**, de modo que se permitan tus propios DM. No se utiliza para envíos automáticos. Si lo ejecutas en tu número personal de WhatsApp, usa ese mismo número y habilita `channels.whatsapp.selfChatMode`.

<div id="message-normalization-what-the-model-sees">
  ## Normalización de mensajes (lo que ve el modelo)
</div>

* `Body` es el cuerpo del mensaje actual con envoltura.
* El contexto de respuesta citada **siempre se añade**:
  ```
  [Replying to +1555 id:ABC123]
  <quoted text or <media:...>>
  [/Replying]
  ```
* También se establecen los metadatos de respuesta:
  * `ReplyToId` = stanzaId
  * `ReplyToBody` = cuerpo citado o marcador de posición de contenido multimedia
  * `ReplyToSender` = E.164 cuando se conoce
* Los mensajes entrantes que solo contienen contenido multimedia usan marcadores de posición:
  * `<media:image|video|audio|document|sticker>`

<div id="groups">
  ## Grupos
</div>

* Los grupos se asignan a sesiones `agent:<agentId>:whatsapp:group:<jid>`.
* Política de grupo: `channels.whatsapp.groupPolicy = open|disabled|allowlist` (valor predeterminado: `allowlist`).
* Modos de activación:
  * `mention` (predeterminado): requiere @mención o coincidencia de regex.
  * `always`: siempre se activa.
* `/activation mention|always` es solo para el propietario y debe enviarse como mensaje independiente.
* Propietario = `channels.whatsapp.allowFrom` (o tu propio E.164 si no está definido).
* **Inyección de historial** (solo pendientes):
  * Mensajes recientes *no procesados* (50 de forma predeterminada) que se insertan bajo:
    `[Chat messages since your last reply - for context]` (los mensajes que ya están en la sesión no se vuelven a inyectar)
  * Mensaje actual bajo:
    `[Current message - respond to this]`
  * Se añade sufijo del remitente: `[from: Name (+E164)]`
* Metadatos del grupo en caché durante 5 min (asunto + participantes).

<div id="reply-delivery-threading">
  ## Entrega de respuestas (hilos)
</div>

* WhatsApp Web envía mensajes estándar (sin hilos de respuesta con cita en el Gateway actual).
* Las etiquetas de respuesta se ignoran en este canal.

<div id="acknowledgment-reactions-auto-react-on-receipt">
  ## Reacciones de confirmación (reacción automática al recibir)
</div>

WhatsApp puede enviar automáticamente reacciones con emoji a los mensajes entrantes en cuanto se reciben, antes de que el bot genere una respuesta. Esto proporciona una confirmación inmediata a los usuarios de que su mensaje se ha recibido.

**Configuración:**

```json
{
  "whatsapp": {
    "ackReaction": {
      "emoji": "👀",
      "direct": true,
      "group": "mentions"
    }
  }
}
```

**Opciones:**

* `emoji` (string): Emoji que se usará para el acuse de recibo (p. ej., &quot;👀&quot;, &quot;✅&quot;, &quot;📨&quot;). Vacío u omitido = función desactivada.
* `direct` (booleano, valor predeterminado: `true`): Enviar reacciones en chats directos/DM.
* `group` (string, valor predeterminado: `"mentions"`): Comportamiento en chats de grupo:
  * `"always"`: Reaccionar a todos los mensajes de grupo (incluso sin @mención)
  * `"mentions"`: Reaccionar solo cuando se @mencione al bot
  * `"never"`: Nunca reaccionar en grupos

**Anulación por cuenta individual:**

```json
{
  "whatsapp": {
    "accounts": {
      "work": {
        "ackReaction": {
          "emoji": "✅",
          "direct": false,
          "group": "always"
        }
      }
    }
  }
}
```

**Notas sobre el comportamiento:**

* Las reacciones se envían **inmediatamente** al recibir el mensaje, antes de los indicadores de escritura o de las respuestas del bot.
* En grupos con `requireMention: false` (activación: siempre), `group: "mentions"` reaccionará a todos los mensajes (no solo a @menciones).
* Fire-and-forget: los fallos en las reacciones se registran pero no impiden que el bot responda.
* El JID del participante se incluye automáticamente en las reacciones de grupo.
* WhatsApp ignora `messages.ackReaction`; usa `channels.whatsapp.ackReaction` en su lugar.

<div id="agent-tool-reactions">
  ## Herramienta de agente (reacciones)
</div>

* Herramienta: `whatsapp` con acción `react` (`chatJid`, `messageId`, `emoji`, opcional `remove`).
* Opcional: `participant` (remitente en el grupo), `fromMe` (reacción a tu propio mensaje), `accountId` (multi‑cuenta).
* Semántica de eliminación de reacciones: consulta [/tools/reactions](/es/tools/reactions).
* Control de uso de la herramienta: `channels.whatsapp.actions.reactions` (predeterminado: habilitado).

<div id="limits">
  ## Límites
</div>

* El texto saliente se divide en fragmentos según `channels.whatsapp.textChunkLimit` (valor predeterminado: 4000).
* Fragmentación opcional por nueva línea: configura `channels.whatsapp.chunkMode="newline"` para dividir en líneas en blanco (límites de párrafo) antes de fragmentar por longitud.
* El guardado de contenido multimedia entrante se limita con `channels.whatsapp.mediaMaxMb` (valor predeterminado: 50 MB).
* Los elementos multimedia salientes se limitan con `agents.defaults.mediaMaxMb` (valor predeterminado: 5 MB).

<div id="outbound-send-text-media">
  ## Envío saliente (texto + multimedia)
</div>

* Usa el listener web activo; error si el Gateway no se está ejecutando.
* Fragmentación de texto: máximo 4k por mensaje (configurable mediante `channels.whatsapp.textChunkLimit`, opcional `channels.whatsapp.chunkMode`).
* Multimedia:
  * Se admiten imagen, vídeo, audio y documentos.
  * Audio enviado como PTT; `audio/ogg` =&gt; `audio/ogg; codecs=opus`.
  * Texto de pie solo en el primer elemento multimedia.
  * La descarga de contenido multimedia admite URLs HTTP(S) y rutas locales.
  * GIFs animados: WhatsApp espera MP4 con `gifPlayback: true` para reproducción en bucle en línea.
    * CLI: `openclaw message send --media <mp4> --gif-playback`
    * Gateway: los parámetros de `send` incluyen `gifPlayback: true`

<div id="voice-notes-ptt-audio">
  ## Notas de voz (audio PTT)
</div>

WhatsApp envía el audio como **notas de voz** (burbuja PTT).

* Mejores resultados: OGG/Opus. OpenClaw reescribe `audio/ogg` como `audio/ogg; codecs=opus`.
* `[[audio_as_voice]]` se ignora para WhatsApp (el audio ya se envía como nota de voz).

<div id="media-limits-optimization">
  ## Límites y optimización de contenido multimedia
</div>

* Límite máximo de salida predeterminado: 5 MB (por elemento multimedia).
* Se puede sobrescribir mediante: `agents.defaults.mediaMaxMb`.
* Las imágenes se optimizan automáticamente a JPEG dentro del límite (redimensionado + ajuste de calidad).
* Si el contenido multimedia supera el límite =&gt; error; la respuesta multimedia se sustituye por una advertencia en texto.

<div id="heartbeats">
  ## Latidos
</div>

* **Latido del Gateway** registra la salud de la conexión (`web.heartbeatSeconds`, valor predeterminado 60s).
* **Latido del agente** se puede configurar por agente (`agents.list[].heartbeat`) o globalmente
  mediante `agents.defaults.heartbeat` (como alternativa cuando no se definen entradas por agente).
  * Usa el prompt de latido configurado (predeterminado: `Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.`) + el comportamiento de omisión cuando se recibe `HEARTBEAT_OK`.
  * Por defecto, se envía al último canal utilizado (o al destino configurado).

<div id="reconnect-behavior">
  ## Comportamiento de reconexión
</div>

* Política de backoff: `web.reconnect`:
  * `initialMs`, `maxMs`, `factor`, `jitter`, `maxAttempts`.
* Si se alcanza `maxAttempts`, la monitorización web se detiene (estado degradado).
* Si se cierra la sesión =&gt; se detiene y requiere volver a vincular.

<div id="config-quick-map">
  ## Mapa rápido de configuración
</div>

* `channels.whatsapp.dmPolicy` (política de MD: emparejamiento/lista de permitidos/open/deshabilitado).
* `channels.whatsapp.selfChatMode` (configuración en el mismo teléfono; el bot usa tu número personal de WhatsApp).
* `channels.whatsapp.allowFrom` (lista de permitidos para MD). WhatsApp usa números de teléfono E.164 (sin nombres de usuario).
* `channels.whatsapp.mediaMaxMb` (límite de guardado de contenido multimedia entrante).
* `channels.whatsapp.ackReaction` (reacción automática al recibir un mensaje: `{emoji, direct, group}`).
* `channels.whatsapp.accounts.<accountId>.*` (configuración por cuenta + `authDir` opcional).
* `channels.whatsapp.accounts.<accountId>.mediaMaxMb` (límite por cuenta para contenido multimedia entrante).
* `channels.whatsapp.accounts.<accountId>.ackReaction` (anulación por cuenta de la reacción de acuse de recibo).
* `channels.whatsapp.groupAllowFrom` (lista de permitidos de remitentes en grupos).
* `channels.whatsapp.groupPolicy` (política de grupos).
* `channels.whatsapp.historyLimit` / `channels.whatsapp.accounts.<accountId>.historyLimit` (contexto de historial de grupo; `0` lo deshabilita).
* `channels.whatsapp.dmHistoryLimit` (límite de historial de MD en turnos de usuario). Anulaciones por usuario: `channels.whatsapp.dms["<phone>"].historyLimit`.
* `channels.whatsapp.groups` (lista de permitidos de grupos + valores predeterminados de control por menciones; usa `"*"` para permitir todos los grupos)
* `channels.whatsapp.actions.reactions` (controla las reacciones de herramientas en WhatsApp).
* `agents.list[].groupChat.mentionPatterns` (o `messages.groupChat.mentionPatterns`)
* `messages.groupChat.historyLimit`
* `channels.whatsapp.messagePrefix` (prefijo de entrada; por cuenta: `channels.whatsapp.accounts.<accountId>.messagePrefix`; en desuso: `messages.messagePrefix`)
* `messages.responsePrefix` (prefijo de salida)
* `agents.defaults.mediaMaxMb`
* `agents.defaults.heartbeat.every`
* `agents.defaults.heartbeat.model` (anulación opcional)
* `agents.defaults.heartbeat.target`
* `agents.defaults.heartbeat.to`
* `agents.defaults.heartbeat.session`
* `agents.list[].heartbeat.*` (anulaciones por agente)
* `session.*` (ámbito, inactividad, almacenamiento, mainKey)
* `web.enabled` (deshabilita el inicio del canal cuando es falso)
* `web.heartbeatSeconds`
* `web.reconnect.*`

<div id="logs-troubleshooting">
  ## Registros y solución de problemas
</div>

* Subsistemas: `whatsapp/inbound`, `whatsapp/outbound`, `web-heartbeat`, `web-reconnect`.
* Archivo de registro: `/tmp/openclaw/openclaw-YYYY-MM-DD.log` (configurable).
* Guía de solución de problemas: [Guía de solución de problemas de Gateway](/es/gateway/troubleshooting).

<div id="troubleshooting-quick">
  ## Solución rápida de problemas
</div>

**No vinculado / Se requiere inicio de sesión con QR**

* Síntoma: `channels status` muestra `linked: false` o advierte “Not linked”.
* Solución: ejecuta `openclaw channels login` en el host del Gateway y escanea el QR (WhatsApp → Configuración → Dispositivos vinculados).

**Vinculado pero desconectado / bucle de reconexión**

* Síntoma: `channels status` muestra `running, disconnected` o advierte “Linked but disconnected”.
* Solución: ejecuta `openclaw doctor` (o reinicia el Gateway). Si persiste, vuelve a vincular mediante `channels login` e inspecciona `openclaw logs --follow`.

**Entorno de ejecución Bun**

* Bun **no se recomienda**. WhatsApp (Baileys) y Telegram son poco fiables en Bun.
  Ejecuta el Gateway con **Node**. (Consulta la nota sobre el entorno de ejecución en la sección Getting Started).