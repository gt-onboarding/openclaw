---
title: Slack
summary: "Configuración de Slack para el modo socket o webhook HTTP"
read_when: "Configurar o depurar el modo socket/HTTP de Slack"
---

<div id="slack">
  # Slack
</div>

<div id="socket-mode-default">
  ## Modo Socket (predeterminado)
</div>

<div id="quick-setup-beginner">
  ### Configuración rápida (para principiantes)
</div>

1. Crea una aplicación de Slack y habilita **Socket Mode**.
2. Crea un **App Token** (`xapp-...`) y un **Bot Token** (`xoxb-...`).
3. Configura los tokens en OpenClaw e inicia el Gateway.

Configuración mínima:

```json5
{
  channels: {
    slack: {
      enabled: true,
      appToken: "xapp-...",
      botToken: "xoxb-..."
    }
  }
}
```

<div id="setup">
  ### Configuración
</div>

1. Crea una app de Slack (From scratch) en https://api.slack.com/apps.
2. **Socket Mode** → actívalo. Luego ve a **Basic Information** → **App-Level Tokens** → **Generate Token and Scopes** con el ámbito `connections:write`. Copia el **App Token** (`xapp-...`).
3. **OAuth &amp; Permissions** → añade los ámbitos del bot token (usa el manifiesto de abajo). Haz clic en **Install to Workspace**. Copia el **Bot User OAuth Token** (`xoxb-...`).
4. Opcional: **OAuth &amp; Permissions** → añade **User Token Scopes** (consulta la lista de solo lectura más abajo). Reinstala la app y copia el **User OAuth Token** (`xoxp-...`).
5. **Event Subscriptions** → habilita eventos y suscríbete a:
   * `message.*` (incluye ediciones/eliminaciones/difusiones en hilos)
   * `app_mention`
   * `reaction_added`, `reaction_removed`
   * `member_joined_channel`, `member_left_channel`
   * `channel_rename`
   * `pin_added`, `pin_removed`
6. Invita al bot a los canales que quieres que lea.
7. Slash Commands → crea `/openclaw` si usas `channels.slack.slashCommand`. Si habilitas comandos nativos, añade un comando slash por cada comando integrado (mismos nombres que `/help`). Los nativos están desactivados por defecto para Slack a menos que configures `channels.slack.commands.native: true` (el valor global de `commands.native` es `"auto"`, lo que deja Slack desactivado).
8. App Home → habilita la **Messages Tab** para que los usuarios puedan enviar DMs al bot.

Usa el manifiesto de abajo para que los ámbitos y eventos se mantengan sincronizados.

Soporte multicuenta: usa `channels.slack.accounts` con tokens por cuenta y `name` opcional. Consulta [`gateway/configuration`](/es/gateway/configuration#telegramaccounts--discordaccounts--slackaccounts--signalaccounts--imessageaccounts) para el patrón compartido.

<div id="openclaw-config-minimal">
  ### Configuración mínima de OpenClaw
</div>

Configura los tokens mediante variables de entorno (recomendado):

* `SLACK_APP_TOKEN=xapp-...`
* `SLACK_BOT_TOKEN=xoxb-...`

O mediante configuración:

```json5
{
  channels: {
    slack: {
      enabled: true,
      appToken: "xapp-...",
      botToken: "xoxb-..."
    }
  }
}
```

<div id="user-token-optional">
  ### Token de usuario (opcional)
</div>

OpenClaw puede usar un token de usuario de Slack (`xoxp-...`) para operaciones de lectura (historial,
mensajes fijados, reacciones, emoji, información de miembros). De forma predeterminada, esto se mantiene como solo lectura: las lecturas
prefieren el token de usuario cuando está presente y las escrituras siguen usando el token de bot, a menos que
optes explícitamente por lo contrario. Incluso con `userTokenReadOnly: false`, el token de bot sigue
siendo el preferido para escrituras cuando está disponible.

Los tokens de usuario se configuran en el archivo de configuración (sin compatibilidad con variables de entorno). Para
varias cuentas, configura `channels.slack.accounts.<id>.userToken`.

Ejemplo con tokens de bot + app + usuario:

```json5
{
  channels: {
    slack: {
      enabled: true,
      appToken: "xapp-...",
      botToken: "xoxb-...",
      userToken: "xoxp-..."
    }
  }
}
```

Ejemplo con userTokenReadOnly configurado explícitamente (que permite escrituras con el token de usuario):

```json5
{
  channels: {
    slack: {
      enabled: true,
      appToken: "xapp-...",
      botToken: "xoxb-...",
      userToken: "xoxp-...",
      userTokenReadOnly: false
    }
  }
}
```

<div id="token-usage">
  #### Uso de tokens
</div>

* Las operaciones de lectura (historial, lista de reacciones, lista de elementos fijados, lista de emoji, información de miembros, búsqueda) prefieren el token de usuario cuando está configurado; en caso contrario, el token del bot.
* Las operaciones de escritura (enviar/editar/eliminar mensajes, añadir/eliminar reacciones, fijar/desfijar, subida de archivos) usan el token del bot de forma predeterminada. Si `userTokenReadOnly: false` y no hay ningún token de bot disponible, OpenClaw recurre al token de usuario.

<div id="history-context">
  ### Contexto del historial
</div>

* `channels.slack.historyLimit` (o `channels.slack.accounts.*.historyLimit`) controla cuántos mensajes recientes de canal/grupo se incluyen en el prompt.
* Si no se establece, se recurre a `messages.groupChat.historyLimit`. Configura `0` para desactivarlo (valor predeterminado: 50).

<div id="http-mode-events-api">
  ## Modo HTTP (Events API)
</div>

Usa el modo webhook HTTP cuando tu Gateway es accesible desde Slack a través de HTTPS (lo habitual en despliegues en servidor).
El modo HTTP usa Events API + Interactivity + Slash Commands mediante una URL de solicitud compartida.

<div id="setup">
  ### Configuración
</div>

1. Crea una aplicación de Slack y **desactiva Socket Mode** (opcional si solo usas HTTP).
2. **Basic Information** → copia el **Signing Secret**.
3. **OAuth &amp; Permissions** → instala la aplicación y copia el **Bot User OAuth Token** (`xoxb-...`).
4. **Event Subscriptions** → habilita los eventos y configura la **Request URL** con la ruta de webhook de tu Gateway (por defecto `/slack/events`).
5. **Interactivity &amp; Shortcuts** → habilita y configura la misma **Request URL**.
6. **Slash Commands** → configura la misma **Request URL** para tus comandos.

Ejemplo de Request URL:
`https://gateway-host/slack/events`

<div id="openclaw-config-minimal">
  ### Configuración mínima de OpenClaw
</div>

```json5
{
  channels: {
    slack: {
      enabled: true,
      mode: "http",
      botToken: "xoxb-...",
      signingSecret: "your-signing-secret",
      webhookPath: "/slack/events"
    }
  }
}
```

Modo HTTP multi-cuenta: configura `channels.slack.accounts.<id>.mode = "http"` y proporciona un
`webhookPath` único por cuenta para que cada aplicación de Slack pueda apuntar a su propia URL.

<div id="manifest-optional">
  ### Manifiesto (opcional)
</div>

Usa este manifiesto de aplicación de Slack para crear la app rápidamente (ajusta el nombre/comando si lo deseas). Incluye los ámbitos de usuario si planeas configurar un token de usuario.

```json
{
  "display_information": {
    "name": "OpenClaw",
    "description": "Slack connector for OpenClaw"
  },
  "features": {
    "bot_user": {
      "display_name": "OpenClaw",
      "always_online": false
    },
    "app_home": {
      "messages_tab_enabled": true,
      "messages_tab_read_only_enabled": false
    },
    "slash_commands": [
      {
        "command": "/openclaw",
        "description": "Send a message to OpenClaw",
        "should_escape": false
      }
    ]
  },
  "oauth_config": {
    "scopes": {
      "bot": [
        "chat:write",
        "channels:history",
        "channels:read",
        "groups:history",
        "groups:read",
        "groups:write",
        "im:history",
        "im:read",
        "im:write",
        "mpim:history",
        "mpim:read",
        "mpim:write",
        "users:read",
        "app_mentions:read",
        "reactions:read",
        "reactions:write",
        "pins:read",
        "pins:write",
        "emoji:read",
        "commands",
        "files:read",
        "files:write"
      ],
      "user": [
        "channels:history",
        "channels:read",
        "groups:history",
        "groups:read",
        "im:history",
        "im:read",
        "mpim:history",
        "mpim:read",
        "users:read",
        "reactions:read",
        "pins:read",
        "emoji:read",
        "search:read"
      ]
    }
  },
  "settings": {
    "socket_mode_enabled": true,
    "event_subscriptions": {
      "bot_events": [
        "app_mention",
        "message.channels",
        "message.groups",
        "message.im",
        "message.mpim",
        "reaction_added",
        "reaction_removed",
        "member_joined_channel",
        "member_left_channel",
        "channel_rename",
        "pin_added",
        "pin_removed"
      ]
    }
  }
}
```

Si habilitas los comandos nativos, añade una entrada `slash_commands` por cada comando que quieras exponer (debe coincidir con la lista de `/help`). Puedes anularlo con `channels.slack.commands.native`.

<div id="scopes-current-vs-optional">
  ## Ámbitos (actuales vs opcionales)
</div>

La Conversations API de Slack utiliza ámbitos por tipo: solo necesitas los ámbitos para los tipos de conversaciones que realmente utilizas (channels, groups, im, mpim). Consulta https://docs.slack.dev/apis/web-api/using-the-conversations-api/ para una descripción general.

<div id="bot-token-scopes-required">
  ### Ámbitos del token de bot (obligatorios)
</div>

* `chat:write` (enviar/actualizar/eliminar mensajes mediante `chat.postMessage`)
  https://docs.slack.dev/reference/methods/chat.postMessage
* `im:write` (abrir mensajes directos (DM) de usuario mediante `conversations.open`)
  https://docs.slack.dev/reference/methods/conversations.open
* `channels:history`, `groups:history`, `im:history`, `mpim:history`
  https://docs.slack.dev/reference/methods/conversations.history
* `channels:read`, `groups:read`, `im:read`, `mpim:read`
  https://docs.slack.dev/reference/methods/conversations.info
* `users:read` (búsqueda de usuario)
  https://docs.slack.dev/reference/methods/users.info
* `reactions:read`, `reactions:write` (`reactions.get` / `reactions.add`)
  https://docs.slack.dev/reference/methods/reactions.get
  https://docs.slack.dev/reference/methods/reactions.add
* `pins:read`, `pins:write` (`pins.list` / `pins.add` / `pins.remove`)
  https://docs.slack.dev/reference/scopes/pins.read
  https://docs.slack.dev/reference/scopes/pins.write
* `emoji:read` (`emoji.list`)
  https://docs.slack.dev/reference/scopes/emoji.read
* `files:write` (carga de archivos mediante `files.uploadV2`)
  https://docs.slack.dev/messaging/working-with-files/#upload

<div id="user-token-scopes-optional-read-only-by-default">
  ### Ámbitos de tokens de usuario (opcional, de solo lectura por defecto)
</div>

Añade estos en **User Token Scopes** si configuras `channels.slack.userToken`.

* `channels:history`, `groups:history`, `im:history`, `mpim:history`
* `channels:read`, `groups:read`, `im:read`, `mpim:read`
* `users:read`
* `reactions:read`
* `pins:read`
* `emoji:read`
* `search:read`

<div id="not-needed-today-but-likely-future">
  ### No necesario hoy (pero probablemente en el futuro)
</div>

* `mpim:write` (solo si agregamos apertura de MD de grupo/inicio de MD mediante `conversations.open`)
* `groups:write` (solo si agregamos gestión de canales privados: crear/renombrar/invitar/archivar)
* `chat:write.public` (solo si queremos publicar en canales en los que el bot no está)
  https://docs.slack.dev/reference/scopes/chat.write.public
* `users:read.email` (solo si necesitamos campos de correo electrónico de `users.info`)
  https://docs.slack.dev/changelog/2017-04-narrowing-email-access
* `files:read` (solo si empezamos a listar/leer metadatos de archivos)

<div id="config">
  ## Configuración
</div>

Slack utiliza únicamente Socket Mode (sin servidor de webhooks HTTP). Proporciona ambos tokens:

```json
{
  "slack": {
    "enabled": true,
    "botToken": "xoxb-...",
    "appToken": "xapp-...",
    "groupPolicy": "allowlist",
    "dm": {
      "enabled": true,
      "policy": "pairing",
      "allowFrom": ["U123", "U456", "*"],
      "groupEnabled": false,
      "groupChannels": ["G123"],
      "replyToMode": "all"
    },
    "channels": {
      "C123": { "allow": true, "requireMention": true },
      "#general": {
        "allow": true,
        "requireMention": true,
        "users": ["U123"],
        "skills": ["search", "docs"],
        "systemPrompt": "Keep answers short."
      }
    },
    "reactionNotifications": "own",
    "reactionAllowlist": ["U123"],
    "replyToMode": "off",
    "actions": {
      "reactions": true,
      "messages": true,
      "pins": true,
      "memberInfo": true,
      "emojiList": true
    },
    "slashCommand": {
      "enabled": true,
      "name": "openclaw",
      "sessionPrefix": "slack:slash",
      "ephemeral": true
    },
    "textChunkLimit": 4000,
    "mediaMaxMb": 20
  }
}
```

Los tokens también se pueden suministrar mediante variables de entorno:

* `SLACK_BOT_TOKEN`
* `SLACK_APP_TOKEN`

Las reacciones de acuse de recibo se controlan a nivel global mediante `messages.ackReaction` +
`messages.ackReactionScope`. Utiliza `messages.removeAckAfterReply` para eliminar la
reacción de acuse de recibo después de que el bot responda.

<div id="limits">
  ## Límites
</div>

* El texto de salida se segmenta según `channels.slack.textChunkLimit` (valor predeterminado 4000).
* Segmentación opcional por nueva línea: establece `channels.slack.chunkMode="newline"` para dividir en líneas en blanco (límites de párrafo) antes de la segmentación por longitud.
* Las subidas de archivos multimedia tienen un límite definido por `channels.slack.mediaMaxMb` (valor predeterminado 20).

<div id="reply-threading">
  ## Encadenamiento de respuestas
</div>

De forma predeterminada, OpenClaw responde en el canal principal. Usa `channels.slack.replyToMode` para controlar el encadenamiento automático:

| Modo | Comportamiento |
| --- | --- |
| `off` | **Predeterminado.** Responder en el canal principal. Solo encadenar si el mensaje que lo originó ya estaba en un hilo. |
| `first` | La primera respuesta va al hilo (debajo del mensaje que la originó), las respuestas posteriores van al canal principal. Útil para mantener el contexto visible evitando a la vez la proliferación de hilos. |
| `all` | Todas las respuestas van al hilo. Mantiene las conversaciones contenidas, pero puede reducir la visibilidad. |

El modo se aplica tanto a las respuestas automáticas como a las llamadas a herramientas de agente (`slack sendMessage`).

<div id="per-chat-type-threading">
  ### Encadenamiento por tipo de chat
</div>

Puedes configurar un comportamiento de encadenamiento diferente por tipo de chat mediante la opción `channels.slack.replyToModeByChatType`:

```json5
{
  channels: {
    slack: {
      replyToMode: "off",        // default for channels
      replyToModeByChatType: {
        direct: "all",           // DMs always thread
        group: "first"           // DMs de grupo/MPIM crean hilo en la primera respuesta
      },
    }
  }
}
```

Tipos de chat admitidos:

* `direct`: mensajes directos 1:1 (Slack `im`)
* `group`: mensajes directos de grupo / MPIM (Slack `mpim`)
* `channel`: canales estándar (públicos/privados)

Precedencia:

1. `replyToModeByChatType.<chatType>`
2. `replyToMode`
3. Valor predeterminado del proveedor (`off`)

El valor heredado `channels.slack.dm.replyToMode` todavía se acepta como opción de reserva para `direct` cuando no se establece ninguna anulación por tipo de chat.

Ejemplos:

Solo mensajes directos en hilos:

```json5
{
  channels: {
    slack: {
      replyToMode: "off",
      replyToModeByChatType: { direct: "all" }
    }
  }
}
```

Crea hilos en los MD grupales pero mantén los canales en la raíz:

```json5
{
  channels: {
    slack: {
      replyToMode: "off",
      replyToModeByChatType: { group: "first" }
    }
  }
}
```

Usa hilos en los canales y mantén los mensajes directos en la raíz:

```json5
{
  channels: {
    slack: {
      replyToMode: "first",
      replyToModeByChatType: { direct: "off", group: "off" }
    }
  }
}
```

<div id="manual-threading-tags">
  ### Etiquetas de gestión manual de hilos
</div>

Para un control más granular, usa estas etiquetas en las respuestas de los agentes:

* `[[reply_to_current]]` — responde al mensaje que la desencadena (inicia/continúa el hilo).
* `[[reply_to:<id>]]` — responde a un ID de mensaje específico.

<div id="sessions-routing">
  ## Sesiones y enrutamiento
</div>

* Los mensajes directos (DM) comparten la sesión `main` (como en WhatsApp/Telegram).
* Los canales se asignan a sesiones `agent:<agentId>:slack:channel:<channelId>`.
* Los comandos slash usan sesiones `agent:<agentId>:slack:slash:<userId>` (prefijo configurable mediante `channels.slack.slashCommand.sessionPrefix`).
* Si Slack no proporciona `channel_type`, OpenClaw lo infiere a partir del prefijo del ID del canal (`D`, `C`, `G`) y usa `channel` de forma predeterminada para mantener estables las claves de sesión.
* El registro de comandos nativos usa `commands.native` (valor global predeterminado `"auto"` → Slack desactivado) y se puede sobrescribir por espacio de trabajo con `channels.slack.commands.native`. Los comandos de texto requieren mensajes independientes de la forma `/...` y se pueden desactivar con `commands.text: false`. Los comandos slash de Slack se gestionan en la aplicación de Slack y no se eliminan automáticamente. Usa `commands.useAccessGroups: false` para omitir las comprobaciones de grupos de acceso para los comandos.
* Lista completa de comandos y configuración: [Comandos slash](/es/tools/slash-commands)

<div id="dm-security-pairing">
  ## Seguridad de MD (emparejamiento)
</div>

* Predeterminado: `channels.slack.dm.policy="pairing"` — los remitentes de MD desconocidos reciben un código de emparejamiento (expira después de 1 hora).
* Aprobar mediante: `openclaw pairing approve slack <code>`.
* Para permitir a cualquiera: establece `channels.slack.dm.policy="open"` (el valor de la directiva open permite aceptar mensajes sin restricciones de cualquier usuario) y `channels.slack.dm.allowFrom=["*"]`.
* `channels.slack.dm.allowFrom` acepta IDs de usuario, @handles o correos electrónicos (se resuelven al inicio cuando los tokens lo permiten). El asistente acepta nombres de usuario y los resuelve a IDs durante la configuración cuando los tokens lo permiten.

<div id="group-policy">
  ## Política de grupos
</div>

* `channels.slack.groupPolicy` controla el manejo de canales (`open` [acepta mensajes sin restricciones de cualquier usuario] | `disabled` | `allowlist`).
* `allowlist` requiere que los canales estén listados en `channels.slack.channels`.
* Si solo defines `SLACK_BOT_TOKEN`/`SLACK_APP_TOKEN` y nunca creas una sección `channels.slack`,
  el tiempo de ejecución establece por defecto `groupPolicy` en `open`. Agrega `channels.slack.groupPolicy`,
  `channels.defaults.groupPolicy` o una lista de permitidos de canales para restringirlo.
* El asistente de configuración acepta nombres de `#channel` y los resuelve a IDs cuando es posible
  (públicos + privados); si existen múltiples coincidencias, prefiere el canal activo.
* En el arranque, OpenClaw resuelve los nombres de canal/usuario en listas de permitidos a IDs (cuando los tokens lo permiten)
  y registra la asignación en los logs; las entradas no resueltas se mantienen tal como se escribieron.
* Para permitir **ningún canal**, configura `channels.slack.groupPolicy: "disabled"` (o mantén una lista de permitidos vacía).

Opciones de canal (`channels.slack.channels.<id>` o `channels.slack.channels.<name>`):

* `allow`: permite/deniega el canal cuando `groupPolicy="allowlist"`.
* `requireMention`: control de acceso por mención para el canal.
* `tools`: anulaciones opcionales de la política de herramientas por canal (`allow`/`deny`/`alsoAllow`).
* `toolsBySender`: anulaciones opcionales de la política de herramientas por remitente dentro del canal (las claves son IDs de remitente/@handles/correos electrónicos; se admite el comodín `"*"`).
* `allowBots`: permite mensajes generados por bots en este canal (valor predeterminado: false).
* `users`: lista de permitidos de usuarios opcional por canal.
* `skills`: filtro de habilidades (omitir = todas las habilidades, vacío = ninguna).
* `systemPrompt`: prompt de sistema adicional para el canal (combinado con el tema/propósito).
* `enabled`: establece `false` para deshabilitar el canal.

<div id="delivery-targets">
  ## Destinos de envío
</div>

Úsalos con envíos desde cron/CLI:

* `user:<id>` para mensajes directos (DMs)
* `channel:<id>` para canales

<div id="tool-actions">
  ## Acciones de herramientas
</div>

Las acciones de herramientas de Slack se pueden restringir con `channels.slack.actions.*`:

| Grupo de acciones | Valor predeterminado | Notas |
| --- | --- | --- |
| reactions | enabled | Reaccionar + listar reacciones |
| messages | enabled | Leer/enviar/editar/eliminar |
| pins | enabled | Fijar/desfijar/listar |
| memberInfo | enabled | Información de miembros |
| emojiList | enabled | Lista de emojis personalizados |

<div id="security-notes">
  ## Notas de seguridad
</div>

* Las operaciones de escritura usan de forma predeterminada el token del bot, de modo que las acciones que modifican el estado permanecen dentro del ámbito de los permisos e identidad del bot de la app.
* Configurar `userTokenReadOnly: false` permite que el token de usuario se use para operaciones de escritura cuando no hay un token de bot disponible, lo que significa que las acciones se ejecutan con el acceso del usuario que instaló la app. Trata el token de usuario como altamente privilegiado y mantén estrictos los controles de acciones y las listas de permitidos.
* Si habilitas escrituras con token de usuario, asegúrate de que el token de usuario incluya los ámbitos de escritura que esperas (`chat:write`, `reactions:write`, `pins:write`, `files:write`) o esas operaciones fallarán.

<div id="notes">
  ## Notas
</div>

* La restricción por menciones se controla mediante `channels.slack.channels` (establece `requireMention` en `true`); `agents.list[].groupChat.mentionPatterns` (o `messages.groupChat.mentionPatterns`) también cuentan como menciones.
* Anulación multi-agente: define patrones por agente en `agents.list[].groupChat.mentionPatterns`.
* Las notificaciones de reacciones siguen `channels.slack.reactionNotifications` (utiliza `reactionAllowlist` con modo `allowlist`).
* Los mensajes escritos por bots se ignoran de forma predeterminada; actívalos mediante `channels.slack.allowBots` o `channels.slack.channels.<id>.allowBots`.
* Advertencia: si permites respuestas a otros bots (`channels.slack.allowBots=true` o `channels.slack.channels.<id>.allowBots=true`), evita bucles de respuestas entre bots con `requireMention`, listas de permitidos `channels.slack.channels.<id>.users` y/o directrices claras en `AGENTS.md` y `SOUL.md`.
* Para la herramienta de Slack, la semántica de eliminación de reacciones se describe en [/tools/reactions](/es/tools/reactions).
* Los archivos adjuntos se descargan al almacén de medios cuando se permite y no superan el límite de tamaño.