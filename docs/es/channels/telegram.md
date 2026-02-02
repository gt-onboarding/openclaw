---
title: Telegram
summary: "Estado de soporte, capacidades y configuración del bot de Telegram"
read_when:
  - Al trabajar en funcionalidades o webhooks de Telegram
---

<div id="telegram-bot-api">
  # Telegram (Bot API)
</div>

Estado: preparado para producción para mensajes directos (DM) al bot y grupos mediante grammY. Long polling por defecto; webhook opcional.

<div id="quick-setup-beginner">
  ## Configuración rápida (para principiantes)
</div>

1. Crea un bot con **@BotFather** y copia el token.
2. Configura el token:
   * Entorno: `TELEGRAM_BOT_TOKEN=...`
   * O configuración: `channels.telegram.botToken: "..."`.
   * Si ambos están definidos, la configuración tiene prioridad (la variable de entorno solo se usa como respaldo para la cuenta predeterminada).
3. Inicia el Gateway.
4. El acceso mediante DM utiliza el emparejamiento de forma predeterminada; aprueba el código de emparejamiento en el primer contacto.

Configuración mínima:

```json5
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "123:abc",
      dmPolicy: "pairing"
    }
  }
}
```

<div id="what-it-is">
  ## Qué es
</div>

* Un canal de Telegram Bot API perteneciente al Gateway.
* Enrutamiento determinista: las respuestas se envían de vuelta a Telegram; el modelo nunca elige canales.
* Los mensajes directos (DM) comparten la sesión principal del agente; los grupos permanecen aislados (`agent:<agentId>:telegram:group:<chatId>`).

<div id="setup-fast-path">
  ## Configuración rápida
</div>

<div id="1-create-a-bot-token-botfather">
  ### 1) Crea un token de bot (BotFather)
</div>

1. Abre Telegram e inicia una conversación con **@BotFather**.
2. Ejecuta `/newbot` y luego sigue las indicaciones (nombre + nombre de usuario que termine en `bot`).
3. Copia el token y guárdalo de forma segura.

Ajustes opcionales en BotFather:

* `/setjoingroups` — permitir o denegar que se agregue el bot a grupos.
* `/setprivacy` — controlar si el bot puede ver todos los mensajes del grupo.

<div id="2-configure-the-token-env-or-config">
  ### 2) Configura el token (env o config)
</div>

Ejemplo:

```json5
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "123:abc",
      dmPolicy: "pairing",
      groups: { "*": { requireMention: true } }
    }
  }
}
```

Opción de entorno: `TELEGRAM_BOT_TOKEN=...` (funciona para la cuenta predeterminada).
Si se definen tanto la variable de entorno como la configuración, la configuración tiene prioridad.

Soporte de multicuentas: utiliza `channels.telegram.accounts` con tokens por cuenta y un `name` opcional. Consulta [`gateway/configuration`](/es/gateway/configuration#telegramaccounts--discordaccounts--slackaccounts--signalaccounts--imessageaccounts) para ver el patrón compartido.

3. Inicia el Gateway. Telegram se inicia cuando se resuelve un token (primero desde la configuración y, en su defecto, desde la variable de entorno).
4. El acceso por DM usa emparejamiento de forma predeterminada. Aprueba el código cuando contacten al bot por primera vez.
5. Para grupos: añade el bot, decide el comportamiento de privacidad/admin (más abajo) y luego configura `channels.telegram.groups` para controlar la restricción por menciones + listas de permitidos.

<div id="token-privacy-permissions-telegram-side">
  ## Token + privacidad + permisos (del lado de Telegram)
</div>

<div id="token-creation-botfather">
  ### Creación del token (BotFather)
</div>

* `/newbot` crea el bot y devuelve el token (manténlo en secreto).
* Si se filtra un token, revócalo o regénéralo con @BotFather y actualiza tu configuración.

<div id="group-message-visibility-privacy-mode">
  ### Visibilidad de mensajes en grupos (Modo de privacidad)
</div>

De forma predeterminada, los bots de Telegram usan el **modo de privacidad**, lo que limita qué mensajes de grupo reciben.
Si tu bot debe ver *todos* los mensajes del grupo, tienes dos opciones:

* Desactivar el modo de privacidad con `/setprivacy` **o**
* Agregar el bot como **administrador** del grupo (los bots administradores reciben todos los mensajes).

**Nota:** Cuando cambias el modo de privacidad, Telegram requiere quitar y volver a agregar el bot
en cada grupo para que el cambio tenga efecto.

<div id="group-permissions-admin-rights">
  ### Permisos de grupo (privilegios de administrador)
</div>

El estado de administrador se configura dentro del grupo (UI de Telegram). Los bots administradores siempre reciben todos los mensajes del grupo, así que usa administrador si necesitas visibilidad total.

<div id="how-it-works-behavior">
  ## Cómo funciona (comportamiento)
</div>

* Los mensajes entrantes se normalizan en el envoltorio de canal compartido con contexto de respuesta y marcadores de posición para medios.
* Las respuestas en grupos requieren una mención de forma predeterminada (mención nativa con @ o `agents.list[].groupChat.mentionPatterns` / `messages.groupChat.mentionPatterns`).
* Override multi‑agente: define patrones por agente en `agents.list[].groupChat.mentionPatterns`.
* Las respuestas siempre se enrutan de vuelta al mismo chat de Telegram.
* El long-polling usa grammY runner con secuenciación por chat; la concurrencia global está limitada por `agents.defaults.maxConcurrent`.
* La API de bots de Telegram no admite confirmaciones de lectura; no hay opción `sendReadReceipts`.

<div id="formatting-telegram-html">
  ## Formato (HTML de Telegram)
</div>

* Los mensajes salientes de Telegram usan `parse_mode: "HTML"` (el subconjunto de etiquetas que Telegram admite).
* La entrada con sintaxis tipo Markdown se renderiza como **HTML compatible con Telegram** (negrita/cursiva/tachado/código/enlaces); los elementos de bloque se aplanan a texto con saltos de línea/viñetas.
* El HTML en bruto generado por los modelos se escapa para evitar errores de análisis de Telegram.
* Si Telegram rechaza el payload HTML, OpenClaw reintenta el mismo mensaje como texto sin formato.

<div id="commands-native-custom">
  ## Comandos (nativos + personalizados)
</div>

OpenClaw registra comandos nativos (como `/status`, `/reset`, `/model`) en el menú del bot de Telegram al iniciarse.
Puedes agregar comandos personalizados al menú mediante la configuración:

```json5
{
  channels: {
    telegram: {
      customCommands: [
        { command: "backup", description: "Git backup" },
        { command: "generate", description: "Create an image" }
      ]
    }
  }
}
```

## Solución de problemas

* `setMyCommands failed` en los registros normalmente indica que el tráfico HTTPS/DNS saliente está bloqueado hacia `api.telegram.org`.
* Si ves errores de `sendMessage` o `sendChatAction`, revisa el enrutamiento IPv6 y el DNS.

Más ayuda: [Solución de problemas de canales](/es/channels/troubleshooting).

Notas:

* Los comandos personalizados son **solo entradas de menú**; OpenClaw no los procesa a menos que los gestiones en otro lugar.
* Los nombres de comando se normalizan (se elimina la `/` inicial, se convierten a minúsculas) y deben ajustarse a `a-z`, `0-9`, `_` (1–32 caracteres).
* Los comandos personalizados **no pueden sobrescribir los comandos nativos**. Los conflictos se ignoran y se registran en los logs.
* Si `commands.native` está deshabilitado, solo se registran los comandos personalizados (o se eliminan si no hay ninguno).

<div id="limits">
  ## Límites
</div>

* El texto saliente se fragmenta según `channels.telegram.textChunkLimit` (valor predeterminado 4000).
* Fragmentación opcional por saltos de línea: configura `channels.telegram.chunkMode="newline"` para dividir en líneas en blanco (límites de párrafo) antes de fragmentar por longitud.
* Las descargas y subidas de archivos multimedia se limitan con `channels.telegram.mediaMaxMb` (valor predeterminado 5).
* Las solicitudes a la Telegram Bot API caducan después de `channels.telegram.timeoutSeconds` (valor predeterminado 500 vía grammY). Establece un valor menor para evitar bloqueos prolongados.
* El contexto de historial de grupo usa `channels.telegram.historyLimit` (o `channels.telegram.accounts.*.historyLimit`), y si no, recurre a `messages.groupChat.historyLimit`. Establece `0` para desactivar (valor predeterminado 50).
* El historial de DM puede limitarse con `channels.telegram.dmHistoryLimit` (turnos del usuario). Anulaciones por usuario: `channels.telegram.dms["<user_id>"].historyLimit`.

<div id="group-activation-modes">
  ## Modos de activación en grupos
</div>

De forma predeterminada, el bot solo responde a menciones en los grupos (`@botname` o patrones en `agents.list[].groupChat.mentionPatterns`). Para cambiar este comportamiento:

<div id="via-config-recommended">
  ### Mediante la configuración (recomendado)
</div>

```json5
{
  channels: {
    telegram: {
      groups: {
        "-1001234567890": { requireMention: false }  // responder siempre en este grupo
      }
    }
  }
}
```

**Importante:** Configurar `channels.telegram.groups` crea una **lista de permitidos**: solo se aceptarán los grupos listados (o `"*"`).
Los temas del foro heredan la configuración de su grupo principal (allowFrom, requireMention, habilidades, prompts) a menos que agregues anulaciones específicas por tema en `channels.telegram.groups.<groupId>.topics.<topicId>`.

Para permitir todos los grupos con always-respond:

```json5
{
  channels: {
    telegram: {
      groups: {
        "*": { requireMention: false }  // todos los grupos, responder siempre
      }
    }
  }
}
```

Para mantener el modo solo por mención en todos los grupos (comportamiento predeterminado):

```json5
{
  channels: {
    telegram: {
      groups: {
        "*": { requireMention: true }  // o omitir groups completamente
      }
    }
  }
}
```

<div id="via-command-session-level">
  ### Mediante comando (a nivel de sesión)
</div>

Envía en el grupo:

* `/activation always` - responder a todos los mensajes
* `/activation mention` - requiere menciones (valor predeterminado)

**Nota:** Los comandos solo actualizan el estado de la sesión. Para un comportamiento persistente entre reinicios, usa la configuración.

<div id="getting-the-group-chat-id">
  ### Obtener el ID del chat de grupo
</div>

Reenvía cualquier mensaje del grupo a `@userinfobot` o `@getidsbot` en Telegram para ver el ID del chat (un número negativo como `-1001234567890`).

**Consejo:** Para obtener tu propio ID de usuario, envíale un mensaje directo (MD) al bot y te responderá con tu ID de usuario (mensaje de emparejamiento), o usa `/whoami` una vez que los comandos estén habilitados.

**Nota de privacidad:** `@userinfobot` es un bot de terceros. Si lo prefieres, añade el bot al grupo, envía un mensaje y usa `openclaw logs --follow` para leer el `chat.id`, o usa la Bot API `getUpdates`.

<div id="config-writes">
  ## Escrituras de configuración
</div>

De forma predeterminada, Telegram puede escribir actualizaciones de configuración provocadas por eventos del canal o por `/config set|unset`.

Esto sucede cuando:

* Un grupo se convierte en un supergrupo y Telegram emite `migrate_to_chat_id` (el ID de chat cambia). OpenClaw puede migrar `channels.telegram.groups` automáticamente.
* Ejecutas `/config set` o `/config unset` en un chat de Telegram (requiere `commands.config: true`).

Desactívalo con:

```json5
{
  channels: { telegram: { configWrites: false } }
}
```

<div id="topics-forum-supergroups">
  ## Temas (supergrupos de foros)
</div>

Los temas de foros de Telegram incluyen un `message_thread_id` por mensaje. OpenClaw:

* Agrega `:topic:<threadId>` a la clave de sesión del grupo de Telegram para que cada tema quede aislado.
* Envía indicadores de escritura y respuestas con `message_thread_id` para que las respuestas permanezcan en el tema.
* El tema general (id de hilo `1`) es especial: los envíos de mensajes omiten `message_thread_id` (Telegram lo rechaza), pero los indicadores de escritura siguen incluyéndolo.
* Expone `MessageThreadId` + `IsForum` en el contexto de plantillas para enrutamiento/templating.
* La configuración específica de temas está disponible en `channels.telegram.groups.<chatId>.topics.<threadId>` (habilidades, listas de permitidos, respuesta automática, prompts del sistema, deshabilitar).
* Las configuraciones de temas heredan los ajustes del grupo (requireMention, listas de permitidos, habilidades, prompts, enabled) a menos que se sobrescriban por tema.

Los chats privados pueden incluir `message_thread_id` en algunos casos límite. OpenClaw mantiene sin cambios la clave de sesión del DM, pero sigue usando el id de hilo para respuestas/streaming de borradores cuando está presente.

<div id="inline-buttons">
  ## Botones inline
</div>

Telegram es compatible con teclados inline con botones de callback.

```json5
{
  "channels": {
    "telegram": {
      "capabilities": {
        "inlineButtons": "allowlist"
      }
    }
  }
}
```

Para la configuración específica de cada cuenta:

```json5
{
  "channels": {
    "telegram": {
      "accounts": {
        "main": {
          "capabilities": {
            "inlineButtons": "allowlist"
          }
        }
      }
    }
  }
}
```

Ámbitos:

* `off` — botones inline desactivados
* `dm` — solo DMs (destinos de grupos bloqueados)
* `group` — solo grupos (destinos de DM bloqueados)
* `all` — DMs + grupos
* `allowlist` — DMs + grupos, pero solo remitentes permitidos por `allowFrom`/`groupAllowFrom` (mismas reglas que para los comandos de control)

Predeterminado: `allowlist`.
Legado: `capabilities: ["inlineButtons"]` = `inlineButtons: "all"`.

<div id="sending-buttons">
  ### Enviar botones
</div>

Usa la herramienta de mensajes con el parámetro `buttons`:

```json5
{
  "action": "send",
  "channel": "telegram",
  "to": "123456789",
  "message": "Choose an option:",
  "buttons": [
    [
      {"text": "Yes", "callback_data": "yes"},
      {"text": "No", "callback_data": "no"}
    ],
    [
      {"text": "Cancel", "callback_data": "cancel"}
    ]
  ]
}
```

Cuando un usuario hace clic en un botón, los datos de callback se envían de vuelta al agente como un mensaje con el siguiente formato:
`callback_data: value`

<div id="configuration-options">
  ### Opciones de configuración
</div>

Las capacidades de Telegram se pueden configurar en dos niveles (forma de objeto mostrada arriba; los arrays de cadenas heredados siguen siendo compatibles):

* `channels.telegram.capabilities`: Configuración global predeterminada de capacidades que se aplica a todas las cuentas de Telegram, a menos que se sobrescriba.
* `channels.telegram.accounts.<account>.capabilities`: Capacidades por cuenta que sobrescriben los valores predeterminados globales para esa cuenta específica.

Usa la configuración global cuando todos los bots y cuentas de Telegram deban comportarse igual. Usa la configuración por cuenta cuando distintos bots necesiten comportamientos diferentes (por ejemplo, una cuenta solo gestiona mensajes directos (DM) mientras que otra está permitida en grupos).

<div id="access-control-dms-groups">
  ## Control de acceso (mensajes directos y grupos)
</div>

<div id="dm-access">
  ### Acceso a MD
</div>

* Predeterminado: `channels.telegram.dmPolicy = "pairing"`. Los remitentes desconocidos reciben un código de emparejamiento; los mensajes se ignoran hasta que se aprueban (los códigos caducan después de 1 hora).
* Aprobar mediante:
  * `openclaw pairing list telegram`
  * `openclaw pairing approve telegram <CODE>`
* El emparejamiento es el intercambio de tokens predeterminado usado para los MD de Telegram. Detalles: [Emparejamiento](/es/start/pairing)
* `channels.telegram.allowFrom` acepta ID numéricos de usuario (recomendado) o entradas `@username`. **No** es el nombre de usuario del bot; utiliza el ID del remitente humano. El asistente acepta `@username` y lo resuelve a su ID numérico cuando es posible.

<div id="finding-your-telegram-user-id">
  #### Encontrar tu ID de usuario de Telegram
</div>

Más seguro (sin bot de terceros):

1. Inicia el Gateway y envía un mensaje directo (DM) a tu bot.
2. Ejecuta `openclaw logs --follow` y busca `from.id`.

Alternativa (Bot API oficial):

1. Envía un mensaje directo (DM) a tu bot.
2. Obtén las actualizaciones con el token de tu bot y lee `message.from.id`:
   ```bash
   curl "https://api.telegram.org/bot<bot_token>/getUpdates"
   ```

De terceros (menos privado):

* Envía un mensaje directo (DM) a `@userinfobot` o `@getidsbot` y usa el ID de usuario devuelto.

<div id="group-access">
  ### Acceso a grupos
</div>

Dos controles independientes:

**1. Qué grupos están permitidos** (lista de permitidos de grupos mediante `channels.telegram.groups`):

* Sin configuración de `groups` = todos los grupos permitidos
* Con configuración de `groups` = solo se permiten los grupos listados o `"*"`
* Ejemplo: `"groups": { "-1001234567890": {}, "*": {} }` permite todos los grupos

**2. Qué remitentes están permitidos** (filtrado de remitentes mediante `channels.telegram.groupPolicy`):

* `"open"` = todos los remitentes en los grupos permitidos pueden enviar mensajes (configuración que permite aceptar mensajes sin restricciones de cualquier usuario)
* `"allowlist"` = solo los remitentes en `channels.telegram.groupAllowFrom` pueden enviar mensajes
* `"disabled"` = no se aceptan mensajes de grupo en absoluto\
  El valor predeterminado es `groupPolicy: "allowlist"` (bloqueado a menos que añadas `groupAllowFrom`).

La mayoría de los usuarios utilizan: `groupPolicy: "allowlist"` + `groupAllowFrom` + grupos específicos listados en `channels.telegram.groups`

<div id="long-polling-vs-webhook">
  ## Long-polling vs webhook
</div>

* Predeterminado: long-polling (no se requiere URL pública).
* Modo webhook: configura `channels.telegram.webhookUrl` (opcionalmente `channels.telegram.webhookSecret` + `channels.telegram.webhookPath`).
  * El listener local se vincula a `0.0.0.0:8787` y expone `POST /telegram-webhook` de forma predeterminada.
  * Si tu URL pública es diferente, usa un proxy inverso y apunta `channels.telegram.webhookUrl` al endpoint público.

<div id="reply-threading">
  ## Respuestas en hilo
</div>

Telegram admite respuestas en hilo opcionales mediante etiquetas:

* `[[reply_to_current]]` -- responder al mensaje que las originó.
* `[[reply_to:<id>]]` -- responder a un ID de mensaje específico.

Controlado por `channels.telegram.replyToMode`:

* `first` (predeterminado), `all`, `off`.

<div id="audio-messages-voice-vs-file">
  ## Mensajes de audio (voz vs archivo)
</div>

Telegram distingue entre **notas de voz** (burbuja redonda) y **archivos de audio** (tarjeta con metadatos).
OpenClaw usa archivos de audio de forma predeterminada por compatibilidad con versiones anteriores.

Para forzar una burbuja de nota de voz en las respuestas del agente, incluye esta etiqueta en cualquier parte de la respuesta:

* `[[audio_as_voice]]` — enviar audio como nota de voz en lugar de archivo.

La etiqueta se elimina del texto entregado. Otros canales ignoran esta etiqueta.

Para envíos mediante la herramienta de mensajes, establece `asVoice: true` con una URL `media` de audio compatible con notas de voz
(`message` es opcional cuando `media` está presente):

```json5
{
  "action": "send",
  "channel": "telegram",
  "to": "123456789",
  "media": "https://example.com/voice.ogg",
  "asVoice": true
}
```

<div id="stickers">
  ## Stickers
</div>

OpenClaw admite la recepción y el envío de stickers de Telegram con caché inteligente.

<div id="receiving-stickers">
  ### Recepción de stickers
</div>

Cuando un usuario envía un sticker, OpenClaw lo maneja según el tipo de sticker:

* **Stickers estáticos (WEBP):** Se descargan y se procesan con visión. El sticker aparece como un marcador de posición `<media:sticker>` en el contenido del mensaje.
* **Stickers animados (TGS):** Se omiten (el formato Lottie no es compatible con el procesamiento).
* **Stickers de video (WEBM):** Se omiten (el formato de video no es compatible con el procesamiento).

Campo de contexto de plantilla disponible al recibir stickers:

* `Sticker` — objeto con:
  * `emoji` — emoji asociado al sticker
  * `setName` — nombre del conjunto de stickers
  * `fileId` — ID de archivo de Telegram (para volver a enviar el mismo sticker)
  * `fileUniqueId` — ID estable para búsqueda en caché
  * `cachedDescription` — descripción generada por visión en caché, cuando esté disponible

<div id="sticker-cache">
  ### Caché de stickers
</div>

Los stickers se procesan mediante las capacidades de visión de la IA para generar descripciones. Dado que los mismos stickers suelen enviarse repetidamente, OpenClaw almacena en caché estas descripciones para evitar llamadas redundantes a la API.

**Cómo funciona:**

1. **Primer encuentro:** La imagen del sticker se envía a la IA para un análisis de visión. La IA genera una descripción (por ejemplo, &quot;Un gato de dibujos animados saludando con entusiasmo&quot;).
2. **Almacenamiento en caché:** La descripción se guarda junto con el ID de archivo del sticker, el emoji y el nombre del paquete.
3. **Encuentros posteriores:** Cuando se vuelve a ver el mismo sticker, se utiliza directamente la descripción en caché. La imagen no se envía a la IA.

**Ubicación de la caché:** `~/.openclaw/telegram/sticker-cache.json`

**Formato de las entradas de la caché:**

```json
{
  "fileId": "CAACAgIAAxkBAAI...",
  "fileUniqueId": "AgADBAADb6cxG2Y",
  "emoji": "👋",
  "setName": "CoolCats",
  "description": "Un gato de caricatura saludando con entusiasmo",
  "cachedAt": "2026-01-15T10:30:00.000Z"
}
```

**Beneficios:**

* Reduce los costos de la API al evitar llamadas de visión repetidas para el mismo sticker
* Tiempos de respuesta más rápidos para stickers en caché (sin retraso por procesamiento de visión)
* Permite la funcionalidad de búsqueda de stickers basada en descripciones almacenadas en caché

La caché se llena automáticamente a medida que se reciben los stickers. No se requiere ninguna gestión manual de la caché.

<div id="sending-stickers">
  ### Envío de stickers
</div>

El agente puede enviar y buscar stickers mediante las acciones `sticker` y `sticker-search`. Están desactivadas por defecto y deben activarse en la configuración:

```json5
{
  channels: {
    telegram: {
      actions: {
        sticker: true
      }
    }
  }
}
```

**Enviar un sticker:**

```json5
{
  "action": "sticker",
  "channel": "telegram",
  "to": "123456789",
  "fileId": "CAACAgIAAxkBAAI..."
}
```

Parámetros:

* `fileId` (obligatorio): el ID de archivo de Telegram del sticker. Obtén este valor de `Sticker.fileId` al recibir un sticker, o de un resultado de `sticker-search`.
* `replyTo` (opcional): ID del mensaje al que responder.
* `threadId` (opcional): ID del hilo de mensajes para temas de foro.

**Buscar stickers:**

El agente puede buscar stickers en caché por descripción, emoji o nombre del conjunto:

```json5
{
  "action": "sticker-search",
  "channel": "telegram",
  "query": "cat waving",
  "limit": 5
}
```

Devuelve los stickers coincidentes de la caché:

```json5
{
  "ok": true,
  "count": 2,
  "stickers": [
    {
      "fileId": "CAACAgIAAxkBAAI...",
      "emoji": "👋",
      "description": "Un gato de caricatura saludando con entusiasmo",
      "setName": "CoolCats"
    }
  ]
}
```

La búsqueda utiliza coincidencia aproximada en el texto de descripción, los caracteres emoji y los nombres de los conjuntos.

**Ejemplo con hilos (threading):**

```json5
{
  "action": "sticker",
  "channel": "telegram",
  "to": "-1001234567890",
  "fileId": "CAACAgIAAxkBAAI...",
  "replyTo": 42,
  "threadId": 123
}
```

<div id="streaming-drafts">
  ## Streaming (borradores)
</div>

Telegram puede hacer *streaming* de **burbujas de borrador** mientras el agente genera una respuesta.
OpenClaw usa la Bot API `sendMessageDraft` (no son mensajes reales) y luego envía la
respuesta final como un mensaje normal.

Requisitos (Telegram Bot API 9.3+):

* **Chats privados con temas habilitados** (modo de temas de foro para el bot).
* Los mensajes entrantes deben incluir `message_thread_id` (hilo de tema privado).
* El *streaming* se ignora para grupos/supergrupos/canales.

Configuración:

* `channels.telegram.streamMode: "off" | "partial" | "block"` (predeterminado: `partial`)
  * `partial`: actualiza la burbuja de borrador con el texto más reciente del *streaming*.
  * `block`: actualiza la burbuja de borrador en bloques más grandes (por fragmentos).
  * `off`: desactiva el *streaming* de borradores.
* Opcional (solo para `streamMode: "block"`):
  * `channels.telegram.draftChunk: { minChars?, maxChars?, breakPreference? }`
    * valores predeterminados: `minChars: 200`, `maxChars: 800`, `breakPreference: "paragraph"` (limitado por `channels.telegram.textChunkLimit`).

Nota: el *streaming* de borradores es independiente del **streaming por bloques** (mensajes del canal).
El *streaming* por bloques está desactivado de forma predeterminada y requiere `channels.telegram.blockStreaming: true`
si quieres mensajes anticipados de Telegram en lugar de actualizaciones de borrador.

Streaming de razonamiento (solo Telegram):

* `/reasoning stream` hace *streaming* del razonamiento en la burbuja de borrador mientras se
  genera la respuesta, y luego envía la respuesta final sin el razonamiento.
* Si `channels.telegram.streamMode` es `off`, el *streaming* de razonamiento queda deshabilitado.
  Más contexto: [Streaming + segmentación](/es/concepts/streaming).

<div id="retry-policy">
  ## Política de reintentos
</div>

Las llamadas salientes a la API de Telegram se reintentan ante errores transitorios de red o 429 con backoff exponencial y jitter. Configúralo mediante `channels.telegram.retry`. Consulta la [política de reintentos](/es/concepts/retry).

<div id="agent-tool-messages-reactions">
  ## Herramienta del agente (mensajes + reacciones)
</div>

* Herramienta: `telegram` con la acción `sendMessage` (`to`, `content`, opcional `mediaUrl`, `replyToMessageId`, `messageThreadId`).
* Herramienta: `telegram` con la acción `react` (`chatId`, `messageId`, `emoji`).
* Herramienta: `telegram` con la acción `deleteMessage` (`chatId`, `messageId`).
* Semántica de eliminación de reacciones: consulta [/tools/reactions](/es/tools/reactions).
* Control de acceso a herramientas: `channels.telegram.actions.reactions`, `channels.telegram.actions.sendMessage`, `channels.telegram.actions.deleteMessage` (predeterminado: habilitado) y `channels.telegram.actions.sticker` (predeterminado: deshabilitado).

<div id="reaction-notifications">
  ## Notificaciones de reacciones
</div>

**Cómo funcionan las reacciones:**
Las reacciones de Telegram llegan como **eventos `message_reaction` separados**, no como propiedades en los payloads de mensajes. Cuando un usuario añade una reacción, OpenClaw:

1. Recibe la actualización `message_reaction` desde la API de Telegram
2. La convierte en un **evento del sistema** con el formato: `"Telegram reaction added: {emoji} by {user} on msg {id}"`
3. Encola el evento del sistema usando la **misma clave de sesión** que los mensajes normales
4. Cuando llega el siguiente mensaje en esa conversación, los eventos del sistema se consumen y se anteponen al contexto del agente

El agente ve las reacciones como **notificaciones del sistema** en el historial de la conversación, no como metadatos del mensaje.

**Configuración:**

* `channels.telegram.reactionNotifications`: Controla qué reacciones generan notificaciones
  * `"off"` — ignora todas las reacciones
  * `"own"` — notifica cuando los usuarios reaccionan a mensajes del bot (best-effort; en memoria) (predeterminado)
  * `"all"` — notifica para todas las reacciones

* `channels.telegram.reactionLevel`: Controla la capacidad de reacción del agente
  * `"off"` — el agente no puede reaccionar a mensajes
  * `"ack"` — el bot envía reacciones de acuse de recibo (👀 mientras procesa) (predeterminado)
  * `"minimal"` — el agente puede reaccionar con moderación (guía: 1 por cada 5-10 intercambios)
  * `"extensive"` — el agente puede reaccionar libremente cuando sea apropiado

**Grupos de foro:** Las reacciones en grupos de foro incluyen `message_thread_id` y usan claves de sesión como `agent:main:telegram:group:{chatId}:topic:{threadId}`. Esto garantiza que las reacciones y los mensajes en el mismo tema se mantengan juntos.

**Ejemplo de configuración:**

```json5
{
  channels: {
    telegram: {
      reactionNotifications: "all",  // Ver todas las reacciones
      reactionLevel: "minimal"        // El agente puede reaccionar de forma moderada
    }
  }
}
```

**Requisitos:**

* Los bots de Telegram deben solicitar explícitamente `message_reaction` en `allowed_updates` (configurado automáticamente por OpenClaw)
* En el modo webhook, las reacciones se incluyen en el `allowed_updates` del webhook
* En el modo polling (sondeo), las reacciones se incluyen en el `allowed_updates` de `getUpdates`

<div id="delivery-targets-clicron">
  ## Destinos de entrega (CLI/cron)
</div>

* Usa un ID de chat (`123456789`) o un nombre de usuario (`@name`) como destino.
* Ejemplo: `openclaw message send --channel telegram --target 123456789 --message "hi"`.

<div id="troubleshooting">
  ## Resolución de problemas
</div>

**El bot no responde a mensajes sin mención en un grupo:**

* Si configuras `channels.telegram.groups.*.requireMention=false`, el **modo de privacidad** de la Bot API de Telegram debe estar desactivado.
  * BotFather: `/setprivacy` → **Disable** (luego elimina y vuelve a añadir el bot al grupo)
* `openclaw channels status` muestra una advertencia cuando la configuración espera mensajes de grupo sin mención.
* `openclaw channels status --probe` además puede comprobar la pertenencia para IDs de grupo numéricos explícitos (no puede auditar reglas comodín `"*"`).
* Prueba rápida: `/activation always` (solo afecta a la sesión; usa la configuración para hacerlo persistente)

**El bot no ve los mensajes del grupo en absoluto:**

* Si `channels.telegram.groups` está configurado, el grupo debe estar listado o usar `"*"`
* Revisa la configuración de privacidad en @BotFather → &quot;Group Privacy&quot; debería estar **OFF**
* Verifica que el bot sea realmente miembro (no solo un administrador sin acceso de lectura)
* Revisa los registros del Gateway: `openclaw logs --follow` (busca &quot;skipping group message&quot;)

**El bot responde a menciones pero no a `/activation always`:**

* El comando `/activation` actualiza el estado de la sesión pero no lo persiste en la configuración
* Para un comportamiento persistente, añade el grupo a `channels.telegram.groups` con `requireMention: false`

**Comandos como `/status` no funcionan:**

* Asegúrate de que tu ID de usuario de Telegram esté autorizado (mediante emparejamiento o `channels.telegram.allowFrom`)
* Los comandos requieren autorización incluso en grupos con `groupPolicy: "open"`

**El long-polling se aborta inmediatamente en Node 22+ (a menudo con proxies/fetch personalizado):**

* Node 22+ es más estricto con las instancias de `AbortSignal`; las señales externas pueden abortar llamadas `fetch` de inmediato.
* Actualiza a una versión de OpenClaw que normalice las señales de aborto, o ejecuta el Gateway en Node 20 hasta que puedas actualizar.

**El bot arranca y luego deja de responder en silencio (o registra `HttpError: Network request ... failed`):**

* Algunos hosts resuelven `api.telegram.org` primero a IPv6. Si tu servidor no tiene salida IPv6 funcional, grammY puede quedarse bloqueado en solicitudes solo-IPv6.
* Soluciónalo habilitando salida IPv6 **o** forzando la resolución IPv4 para `api.telegram.org` (por ejemplo, añadiendo una entrada en `/etc/hosts` usando el registro A de IPv4, o dando preferencia a IPv4 en la pila DNS de tu sistema operativo), luego reinicia el Gateway.
* Comprobación rápida: `dig +short api.telegram.org A` y `dig +short api.telegram.org AAAA` para confirmar qué devuelve el DNS.

<div id="configuration-reference-telegram">
  ## Referencia de configuración (Telegram)
</div>

Configuración completa: [Configuration](/es/gateway/configuration)

Opciones del proveedor:

* `channels.telegram.enabled`: habilitar/deshabilitar el inicio del canal.
* `channels.telegram.botToken`: token del bot (BotFather).
* `channels.telegram.tokenFile`: leer el token desde una ruta de archivo.
* `channels.telegram.dmPolicy`: `pairing | allowlist | open | disabled` (predeterminado: pairing).
* `channels.telegram.allowFrom`: lista de permitidos de DM (ids/nombres de usuario). `open` requiere `"*"`.
* `channels.telegram.groupPolicy`: `open | allowlist | disabled` (predeterminado: allowlist).
* `channels.telegram.groupAllowFrom`: lista de permitidos de remitentes en grupos (ids/nombres de usuario).
* `channels.telegram.groups`: valores predeterminados por grupo + lista de permitidos (usa `"*"` para valores predeterminados globales).
  * `channels.telegram.groups.<id>.requireMention`: valor predeterminado de restricción por mención.
  * `channels.telegram.groups.<id>.skills`: filtro de habilidades (omitir = todas las habilidades, vacío = ninguna).
  * `channels.telegram.groups.<id>.allowFrom`: anulación de la lista de permitidos de remitentes por grupo.
  * `channels.telegram.groups.<id>.systemPrompt`: prompt de sistema adicional para el grupo.
  * `channels.telegram.groups.<id>.enabled`: deshabilitar el grupo cuando es `false`.
  * `channels.telegram.groups.<id>.topics.<threadId>.*`: anulaciones por tema (mismas propiedades que el grupo).
  * `channels.telegram.groups.<id>.topics.<threadId>.requireMention`: anulación por tema de la restricción por mención.
* `channels.telegram.capabilities.inlineButtons`: `off | dm | group | all | allowlist` (predeterminado: allowlist).
* `channels.telegram.accounts.<account>.capabilities.inlineButtons`: anulación por cuenta.
* `channels.telegram.replyToMode`: `off | first | all` (predeterminado: `first`).
* `channels.telegram.textChunkLimit`: tamaño de fragmento saliente (caracteres).
* `channels.telegram.chunkMode`: `length` (predeterminado) o `newline` para dividir en líneas en blanco (límites de párrafo) antes de fragmentar por longitud.
* `channels.telegram.linkPreview`: activar/desactivar vistas previas de enlaces para mensajes salientes (predeterminado: true).
* `channels.telegram.streamMode`: `off | partial | block` (streaming de borradores).
* `channels.telegram.mediaMaxMb`: límite de medios entrantes/salientes (MB).
* `channels.telegram.retry`: política de reintentos para llamadas salientes a la API de Telegram (attempts, minDelayMs, maxDelayMs, jitter).
* `channels.telegram.network.autoSelectFamily`: anular `Node autoSelectFamily` (true=habilitar, false=deshabilitar). De forma predeterminada está deshabilitado en Node 22 para evitar timeouts de Happy Eyeballs.
* `channels.telegram.proxy`: URL de proxy para llamadas Bot API (SOCKS/HTTP).
* `channels.telegram.webhookUrl`: habilitar el modo webhook.
* `channels.telegram.webhookSecret`: secreto de webhook (opcional).
* `channels.telegram.webhookPath`: ruta local de webhook (predeterminado `/telegram-webhook`).
* `channels.telegram.actions.reactions`: controlar las reacciones de herramientas de Telegram.
* `channels.telegram.actions.sendMessage`: controlar los envíos de mensajes de herramientas de Telegram.
* `channels.telegram.actions.deleteMessage`: controlar los borrados de mensajes de herramientas de Telegram.
* `channels.telegram.actions.sticker`: controlar las acciones de stickers de Telegram — envío y búsqueda (predeterminado: false).
* `channels.telegram.reactionNotifications`: `off | own | all` — controla qué reacciones generan eventos de sistema (predeterminado: `own` cuando no se establece).
* `channels.telegram.reactionLevel`: `off | ack | minimal | extensive` — controla la capacidad de reacción del agente (predeterminado: `minimal` cuando no se establece).

Opciones globales relacionadas:

* `agents.list[].groupChat.mentionPatterns` (patrones de mención para el filtrado por mención).
* `messages.groupChat.mentionPatterns` (valor global de reserva).
* `commands.native` (predeterminado `"auto"` → activado para Telegram/Discord, desactivado para Slack), `commands.text`, `commands.useAccessGroups` (comportamiento de comandos). Anula con `channels.telegram.commands.native`.
* `messages.responsePrefix`, `messages.ackReaction`, `messages.ackReactionScope`, `messages.removeAckAfterReply`.