---
title: Discord
summary: "Estado de compatibilidad, capacidades y configuración del bot de Discord"
read_when:
  - Trabajando en las funcionalidades del canal de Discord
---

<div id="discord-bot-api">
  # Discord (Bot API)
</div>

Estado: listo para usar con mensajes directos (DM) y canales de texto de servidores (guild) a través del gateway oficial de bots de Discord.

<div id="quick-setup-beginner">
  ## Configuración rápida (principiantes)
</div>

1. Crea un bot de Discord y copia el token del bot.
2. En la configuración de la app de Discord, habilita **Message Content Intent** (y **Server Members Intent** si planeas usar lista de permitidos o búsquedas por nombre).
3. Establece el token para OpenClaw:
   * Entorno: `DISCORD_BOT_TOKEN=...`
   * O config: `channels.discord.token: "..."`.
   * Si ambos están definidos, la configuración tiene prioridad (la variable de entorno se usa solo como valor predeterminado para la cuenta principal).
4. Invita al bot a tu servidor con permisos de mensajes (crea un servidor privado si solo quieres DMs).
5. Inicia el Gateway.
6. El acceso por DM usa emparejamiento de forma predeterminada; aprueba el código de emparejamiento en el primer contacto.

Configuración mínima:

```json5
{
  channels: {
    discord: {
      enabled: true,
      token: "TU_TOKEN_DE_BOT"
    }
  }
}
```

<div id="goals">
  ## Objetivos
</div>

* Hablar con OpenClaw a través de DMs de Discord o canales de servidor.
* Los chats directos se consolidan en la sesión principal del agente (predeterminada `agent:main:main`); los canales de servidor permanecen aislados como `agent:<agentId>:discord:channel:<channelId>` (los nombres visibles usan `discord:<guildSlug>#<channelSlug>`).
* Los DMs grupales se ignoran de forma predeterminada; habilítalos mediante `channels.discord.dm.groupEnabled` y, opcionalmente, restrínge su uso con `channels.discord.dm.groupChannels`.
* Mantén el enrutamiento determinista: las respuestas siempre vuelven al canal por el que llegaron.

<div id="how-it-works">
  ## Cómo funciona
</div>

1. Crea una aplicación de Discord → Bot, habilita las intents que necesites (DMs + mensajes de guild + contenido de mensajes) y obtén el token del bot.
2. Invita al bot a tu servidor con los permisos necesarios para leer/enviar mensajes donde quieras usarlo.
3. Configura OpenClaw con `channels.discord.token` (o `DISCORD_BOT_TOKEN` como alternativa).
4. Ejecuta el Gateway; inicia automáticamente el canal de Discord cuando hay un token disponible (primero config, luego variable de entorno) y `channels.discord.enabled` no es `false`.
   * Si prefieres variables de entorno, establece `DISCORD_BOT_TOKEN` (un bloque de config es opcional).
5. Chats directos: usa `user:<id>` (o una mención `<@id>`) al enviar; todas las interacciones van a la sesión compartida `main`. Los IDs numéricos “desnudos” son ambiguos y se rechazan.
6. Canales de guild: usa `channel:<channelId>` para el envío. Las menciones son obligatorias por defecto y se pueden configurar por guild o por canal.
7. Chats directos: seguros por defecto mediante `channels.discord.dm.policy` (valor predeterminado: `"pairing"`). Los remitentes desconocidos reciben un código de emparejamiento (expira después de 1 hora); aprueba mediante `openclaw pairing approve discord <code>`.
   * Para mantener el comportamiento antiguo “open para cualquiera” (aceptación de mensajes sin restricciones de cualquier usuario): establece `channels.discord.dm.policy="open"` y `channels.discord.dm.allowFrom=["*"]`.
   * Para una lista de permitidos estricta: establece `channels.discord.dm.policy="allowlist"` y lista los remitentes en `channels.discord.dm.allowFrom`.
   * Para ignorar todos los DMs: establece `channels.discord.dm.enabled=false` o `channels.discord.dm.policy="disabled"`.
8. Los DMs grupales se ignoran por defecto; habilítalos mediante `channels.discord.dm.groupEnabled` y, opcionalmente, restríngelos con `channels.discord.dm.groupChannels`.
9. Reglas opcionales por guild: establece `channels.discord.guilds` indexado por id de guild (preferido) o slug, con reglas por canal.
10. Comandos nativos opcionales: `commands.native` tiene por defecto `"auto"` (activado para Discord/Telegram, desactivado para Slack). Sobrescribe con `channels.discord.commands.native: true|false|"auto"`; `false` borra los comandos registrados previamente. Los comandos de texto se controlan con `commands.text` y deben enviarse como mensajes `/...` independientes. Usa `commands.useAccessGroups: false` para omitir las comprobaciones de grupos de acceso para comandos.
    * Lista completa de comandos + configuración: [Slash commands](/es/tools/slash-commands)
11. Historial opcional de contexto de guild: establece `channels.discord.historyLimit` (por defecto 20, recurriendo a `messages.groupChat.historyLimit`) para incluir los últimos N mensajes de la guild como contexto al responder a una mención. Establece `0` para deshabilitar.
12. Reacciones: el agente puede activar reacciones mediante la herramienta `discord` (protegida por `channels.discord.actions.*`).
    * Semántica de eliminación de reacciones: consulta [/tools/reactions](/es/tools/reactions).
    * La herramienta `discord` solo se expone cuando el canal actual es Discord.
13. Los comandos nativos usan claves de sesión aisladas (`agent:<agentId>:discord:slash:<userId>`) en lugar de la sesión compartida `main`.

Nota: La resolución nombre → id usa la búsqueda de miembros de guild y requiere Server Members Intent; si el bot no puede buscar miembros, usa ids o menciones `<@id>`.
Nota: Los slugs están en minúsculas con los espacios reemplazados por `-`. Los nombres de canales se convierten a slug sin el `#` inicial.
Nota: Las líneas de contexto de guild `[from:]` incluyen `author.tag` + `id` para facilitar respuestas listas para hacer ping.

<div id="config-writes">
  ## Escrituras de configuración
</div>

De forma predeterminada, Discord puede escribir actualizaciones de configuración iniciadas mediante `/config set|unset` (requiere `commands.config: true`).

Desactívalo con:

```json5
{
  channels: { discord: { configWrites: false } }
}
```

<div id="how-to-create-your-own-bot">
  ## Cómo crear tu propio bot
</div>

Esta es la configuración del “Discord Developer Portal” para ejecutar OpenClaw en un canal de un servidor (guild), como `#help`.

<div id="1-create-the-discord-app-bot-user">
  ### 1) Crea la aplicación de Discord y el bot
</div>

1. Discord Developer Portal → **Applications** → **New Application**
2. En tu aplicación:
   * **Bot** → **Add Bot**
   * Copia el **token del bot** (esto es lo que debes poner en `DISCORD_BOT_TOKEN`)

<div id="2-enable-the-gateway-intents-openclaw-needs">
  ### 2) Habilitar los intents de Gateway que OpenClaw necesita
</div>

Discord bloquea los “privileged intents” a menos que los habilites explícitamente.

En **Bot** → **Privileged Gateway Intents**, habilita:

* **Message Content Intent** (necesaria para leer el texto de los mensajes en la mayoría de los servidores; sin ella verás “Used disallowed intents” o el bot se conectará pero no reaccionará a los mensajes)
* **Server Members Intent** (recomendada; necesaria para algunas búsquedas de miembros/usuarios y para hacer coincidir con la lista de permitidos en servidores)

Por lo general **no** necesitas **Presence Intent**.

<div id="3-generate-an-invite-url-oauth2-url-generator">
  ### 3) Generar una URL de invitación (OAuth2 URL Generator)
</div>

En tu app: **OAuth2** → **URL Generator**

**Scopes**

* ✅ `bot`
* ✅ `applications.commands` (requerido para comandos nativos)

**Permisos del bot** (mínimo recomendado)

* ✅ Ver canales
* ✅ Enviar mensajes
* ✅ Leer el historial de mensajes
* ✅ Insertar enlaces
* ✅ Adjuntar archivos
* ✅ Agregar reacciones (opcional, pero recomendado)
* ✅ Usar emojis y stickers externos (opcional; solo si los necesitas)

Evita conceder **Administrator** a menos que estés depurando y confíes completamente en el bot.

Copia la URL generada, ábrela, elige tu servidor e instala el bot.

<div id="4-get-the-ids-guilduserchannel">
  ### 4) Obtén las IDs (guild/user/channel)
</div>

Discord usa identificadores numéricos en todas partes; la configuración de OpenClaw prefiere usar IDs.

1. Discord (escritorio/web) → **User Settings** → **Advanced** → habilita **Developer Mode**
2. Haz clic con el botón derecho:
   * Nombre del servidor → **Copy Server ID** (guild id)
   * Canal (p. ej., `#help`) → **Copy Channel ID**
   * Tu usuario → **Copy User ID**

<div id="5-configure-openclaw">
  ### 5) Configura OpenClaw
</div>

<div id="token">
  #### Token
</div>

Configura el token del bot mediante una variable de entorno (recomendado en servidores):

* `DISCORD_BOT_TOKEN=...`

O a través de la configuración:

```json5
{
  channels: {
    discord: {
      enabled: true,
      token: "YOUR_BOT_TOKEN"
    }
  }
}
```

Compatibilidad con varias cuentas: usa `channels.discord.accounts` con tokens individuales por cuenta y un `name` opcional. Consulta [`gateway/configuration`](/es/gateway/configuration#telegramaccounts--discordaccounts--slackaccounts--signalaccounts--imessageaccounts) para ver el patrón común.

<div id="allowlist-channel-routing">
  #### Lista de permitidos + enrutamiento de canales
</div>

Ejemplo “servidor único, solo permitirme a mí, solo permitir #help”:

```json5
{
  channels: {
    discord: {
      enabled: true,
      dm: { enabled: false },
      guilds: {
        "YOUR_GUILD_ID": {
          users: ["YOUR_USER_ID"],
          requireMention: true,
          channels: {
            help: { allow: true, requireMention: true }
          }
        }
      },
      retry: {
        attempts: 3,
        minDelayMs: 500,
        maxDelayMs: 30000,
        jitter: 0.1
      }
    }
  }
}
```

Notas:

* `requireMention: true` significa que el bot solo responde cuando se le menciona (recomendado para canales compartidos).
* `agents.list[].groupChat.mentionPatterns` (o `messages.groupChat.mentionPatterns`) también cuentan como menciones para mensajes de servidor (guild).
* Anulación multiagente: configura patrones por agente en `agents.list[].groupChat.mentionPatterns`.
* Si `channels` está presente, cualquier canal que no esté listado se deniega por defecto.
* Usa una entrada de canal `"*"` para aplicar valores predeterminados en todos los canales; las entradas de canal explícitas tienen prioridad sobre el comodín.
* Los threads heredan la configuración del canal padre (lista de permitidos, `requireMention`, habilidades, prompts, etc.) a menos que añadas explícitamente el ID del canal del thread.
* Los mensajes escritos por bots se ignoran por defecto; establece `channels.discord.allowBots=true` para permitirlos (los mensajes propios siguen filtrados).
* Advertencia: Si permites respuestas a otros bots (`channels.discord.allowBots=true`), evita bucles de respuestas entre bots con `requireMention`, listas de permitidos `channels.discord.guilds.*.channels.<id>.users` y/o reglas de seguridad claras en `AGENTS.md` y `SOUL.md`.

<div id="6-verify-it-works">
  ### 6) Verifica que funciona
</div>

1. Inicia el Gateway.
2. En el canal de tu servidor, envía: `@Krill hello` (o el nombre que tenga tu bot).
3. Si no ocurre nada, revisa **Troubleshooting** a continuación.

<div id="troubleshooting">
  ### Resolución de problemas
</div>

* Primero: ejecuta `openclaw doctor` y `openclaw channels status --probe` (avisos accionables + auditorías rápidas).
* **&quot;Used disallowed intents&quot;**: habilita **Message Content Intent** (y probablemente **Server Members Intent**) en el Developer Portal y luego reinicia el Gateway.
* **El bot se conecta pero nunca responde en un canal de servidor (guild)**:
  * Falta **Message Content Intent**, o
  * El bot no tiene permisos en el canal (View/Send/Read History), o
  * Tu configuración requiere menciones y no lo mencionaste, o
  * La lista de permitidos de tu servidor/canal rechaza el canal/usuario.
* **`requireMention: false` pero sigue sin responder**:
* `channels.discord.groupPolicy` tiene como valor predeterminado **allowlist**; establécelo en `"open"` (permite aceptar mensajes sin restricciones de cualquier usuario) o añade una entrada de servidor bajo `channels.discord.guilds` (opcionalmente enumera canales bajo `channels.discord.guilds.<id>.channels` para restringir).
  * Si solo configuras `DISCORD_BOT_TOKEN` y nunca creas una sección `channels.discord`, el runtime
    establece `groupPolicy` en `open` de forma predeterminada (acepta mensajes sin restricciones de cualquier usuario). Añade `channels.discord.groupPolicy`,
    `channels.defaults.groupPolicy` o una lista de permitidos de servidor/canal para restringirlo.
* `requireMention` debe estar bajo `channels.discord.guilds` (o un canal específico). `channels.discord.requireMention` en el nivel superior se ignora.
* Las **auditorías de permisos** (`channels status --probe`) solo revisan IDs numéricos de canales. Si usas slugs/nombres como claves en `channels.discord.guilds.*.channels`, la auditoría no puede verificar los permisos.
* **Los mensajes directos no funcionan**: `channels.discord.dm.enabled=false`, `channels.discord.dm.policy="disabled"` o aún no has sido aprobado (`channels.discord.dm.policy="pairing"`).

<div id="capabilities-limits">
  ## Capacidades y límites
</div>

* MD y canales de texto de servidor (los hilos se tratan como canales separados; voz no admitida).
* Los indicadores de escritura se envían en la medida de lo posible; la segmentación de mensajes usa `channels.discord.textChunkLimit` (por defecto 2000) y divide las respuestas largas por número de líneas (`channels.discord.maxLinesPerMessage`, por defecto 17).
* Segmentación opcional por saltos de línea: establece `channels.discord.chunkMode="newline"` para dividir en líneas en blanco (límites de párrafo) antes de la segmentación por longitud.
* Carga de archivos admitida hasta el valor configurado en `channels.discord.mediaMaxMb` (por defecto 8 MB).
* Las respuestas en servidores se condicionan a menciones de forma predeterminada para evitar bots ruidosos.
* El contexto de respuesta se inserta cuando un mensaje hace referencia a otro mensaje (contenido citado + IDs).
* El enhebrado nativo de respuestas está **desactivado de forma predeterminada**; actívalo con `channels.discord.replyToMode` y etiquetas de respuesta.

<div id="retry-policy">
  ## Política de reintentos
</div>

Las llamadas salientes a la API de Discord se reintentan cuando hay límites de frecuencia (429) usando `retry_after` de Discord cuando está disponible, con backoff exponencial y jitter. Configúralo mediante `channels.discord.retry`. Consulta la sección [Política de reintentos](/es/concepts/retry).

<div id="config">
  ## Configuración
</div>

```json5
{
  channels: {
    discord: {
      enabled: true,
      token: "abc.123",
      groupPolicy: "allowlist",
      guilds: {
        "*": {
          channels: {
            general: { allow: true }
          }
        }
      },
      mediaMaxMb: 8,
      actions: {
        reactions: true,
        stickers: true,
        emojiUploads: true,
        stickerUploads: true,
        polls: true,
        permissions: true,
        messages: true,
        threads: true,
        pins: true,
        search: true,
        memberInfo: true,
        roleInfo: true,
        roles: false,
        channelInfo: true,
        channels: true,
        voiceStatus: true,
        events: true,
        moderation: false
      },
      replyToMode: "off",
      dm: {
        enabled: true,
        policy: "pairing", // emparejamiento | lista de permitidos | abierto | deshabilitado
        allowFrom: ["123456789012345678", "steipete"],
        groupEnabled: false,
        groupChannels: ["openclaw-dm"]
      },
      guilds: {
        "*": { requireMention: true },
        "123456789012345678": {
          slug: "friends-of-openclaw",
          requireMention: false,
          reactionNotifications: "own",
          users: ["987654321098765432", "steipete"],
          channels: {
            general: { allow: true },
            help: {
              allow: true,
              requireMention: true,
              users: ["987654321098765432"],
              skills: ["search", "docs"],
              systemPrompt: "Keep answers short."
            }
          }
        }
      }
    }
  }
}
```

Las reacciones de acuse de recibo se controlan globalmente mediante `messages.ackReaction` +
`messages.ackReactionScope`. Usa `messages.removeAckAfterReply` para eliminar la
reacción de acuse de recibo después de que el bot responda.

* `dm.enabled`: establece `false` para ignorar todos los MD (valor predeterminado `true`).
* `dm.policy`: control de acceso a MD (`pairing` recomendado). `"open"` requiere `dm.allowFrom=["*"]`.
* `dm.allowFrom`: lista de permitidos de MD (ids o nombres de usuario). La usan `dm.policy="allowlist"` y la validación de `dm.policy="open"`. El asistente acepta nombres de usuario y los resuelve a ids cuando el bot puede buscar miembros.
* `dm.groupEnabled`: habilita MD de grupo (valor predeterminado `false`).
* `dm.groupChannels`: lista de permitidos opcional para ids o slugs de canales de MD de grupo.
* `groupPolicy`: controla el manejo de canales de servidor (`open|disabled|allowlist`); `allowlist` requiere listas de permitidos de canales.
* `guilds`: reglas por servidor indexadas por id de servidor (preferido) o slug.
* `guilds."*"`: configuración predeterminada por servidor aplicada cuando no existe una entrada explícita.
* `guilds.<id>.slug`: slug descriptivo opcional usado para nombres visibles.
* `guilds.<id>.users`: lista de permitidos de usuarios opcional por servidor (ids o nombres).
* `guilds.<id>.tools`: anulaciones opcionales de la política de herramientas por servidor (`allow`/`deny`/`alsoAllow`) usadas cuando falta la anulación a nivel de canal.
* `guilds.<id>.toolsBySender`: anulaciones opcionales de la política de herramientas por remitente a nivel de servidor (se aplican cuando falta la anulación a nivel de canal; comodín `"*"` admitido).
* `guilds.<id>.channels.<channel>.allow`: permite/deniega el canal cuando `groupPolicy="allowlist"`.
* `guilds.<id>.channels.<channel>.requireMention`: requisito de mención para el canal.
* `guilds.<id>.channels.<channel>.tools`: anulaciones opcionales de la política de herramientas por canal (`allow`/`deny`/`alsoAllow`).
* `guilds.<id>.channels.<channel>.toolsBySender`: anulaciones opcionales de la política de herramientas por remitente dentro del canal (comodín `"*"` admitido).
* `guilds.<id>.channels.<channel>.users`: lista de permitidos opcional de usuarios por canal.
* `guilds.<id>.channels.<channel>.skills`: filtro de habilidades (omitir = todas las habilidades, vacío = ninguna).
* `guilds.<id>.channels.<channel>.systemPrompt`: mensaje de sistema adicional para el canal (se combina con el tema del canal).
* `guilds.<id>.channels.<channel>.enabled`: establece `false` para deshabilitar el canal.
* `guilds.<id>.channels`: reglas de canal (las claves son slugs o ids de canal).
* `guilds.<id>.requireMention`: requisito de mención por servidor (se puede anular por canal).
* `guilds.<id>.reactionNotifications`: modo de eventos de sistema de reacciones (`off`, `own`, `all`, `allowlist`).
* `textChunkLimit`: tamaño de fragmento de texto saliente (caracteres). Predeterminado: 2000.
* `chunkMode`: `length` (predeterminado) divide solo cuando se excede `textChunkLimit`; `newline` divide en líneas en blanco (límites de párrafo) antes de fragmentar por longitud.
* `maxLinesPerMessage`: máximo suave de líneas por mensaje. Predeterminado: 17.
* `mediaMaxMb`: límite del tamaño de medios entrantes guardados en disco.
* `historyLimit`: número de mensajes recientes del servidor que se incluyen como contexto al responder a una mención (predeterminado 20; recurre a `messages.groupChat.historyLimit`; `0` deshabilita).
* `dmHistoryLimit`: límite de historial de MD en turnos de usuario. Anulaciones por usuario: `dms["<user_id>"].historyLimit`.
* `retry`: política de reintentos para llamadas salientes a la API de Discord (attempts, minDelayMs, maxDelayMs, jitter).
* `actions`: controles de herramientas por acción; omitir para permitir todo (establece `false` para deshabilitar).
  * `reactions` (cubre reaccionar + leer reacciones)
  * `stickers`, `emojiUploads`, `stickerUploads`, `polls`, `permissions`, `messages`, `threads`, `pins`, `search`
  * `memberInfo`, `roleInfo`, `channelInfo`, `voiceStatus`, `events`
  * `channels` (crear/editar/eliminar canales + categorías + permisos)
  * `roles` (agregar/quitar roles, predeterminado `false`)
  * `moderation` (timeout/kick/ban, predeterminado `false`)

Las notificaciones de reacciones usan `guilds.<id>.reactionNotifications`:

* `off`: sin eventos de reacción.
* `own`: reacciones en los propios mensajes del bot (predeterminado).
* `all`: todas las reacciones en todos los mensajes.
* `allowlist`: reacciones desde `guilds.<id>.users` en todos los mensajes (una lista vacía deshabilita).

<div id="tool-action-defaults">
  ### Valores predeterminados de acciones de herramientas
</div>

| Action group | Default | Notes |
| --- | --- | --- |
| reactions | enabled | Reaccionar + listar reacciones + emojiList |
| stickers | enabled | Enviar stickers |
| emojiUploads | enabled | Subir emojis |
| stickerUploads | enabled | Subir stickers |
| polls | enabled | Crear encuestas |
| permissions | enabled | Instantánea de permisos del canal |
| messages | enabled | Leer/send/editar/eliminar |
| threads | enabled | Crear/listar/responder |
| pins | enabled | Fijar/desfijar/listar |
| search | enabled | Búsqueda de mensajes (función en vista previa) |
| memberInfo | enabled | Información del miembro |
| roleInfo | enabled | Lista de roles |
| channelInfo | enabled | Información y lista de canales |
| channels | enabled | Gestión de canales/categorías |
| voiceStatus | enabled | Consulta del estado de voz |
| events | enabled | Listar/crear eventos programados |
| roles | disabled | Agregar/quitar roles |
| moderation | disabled | Tiempo de espera/expulsar/banear |

* `replyToMode`: `off` (predeterminado), `first` o `all`. Se aplica solo cuando el modelo incluye una etiqueta de respuesta.

<div id="reply-tags">
  ## Etiquetas de respuesta
</div>

Para solicitar una respuesta en hilo, el modelo puede incluir una etiqueta en su salida:

* `[[reply_to_current]]` — responder al mensaje de Discord que originó la acción.
* `[[reply_to:<id>]]` — responder a un ID de mensaje específico del contexto/historial.
  Los IDs de los mensajes actuales se añaden a los prompts como `[message_id: …]`; las entradas de historial ya incluyen IDs.

El comportamiento se controla mediante `channels.discord.replyToMode`:

* `off`: ignora las etiquetas.
* `first`: solo el primer fragmento/adjunto saliente es una respuesta.
* `all`: cada fragmento/adjunto saliente es una respuesta.

Notas sobre coincidencia con la lista de permitidos:

* `allowFrom`/`users`/`groupChannels` aceptan IDs, nombres, etiquetas o menciones como `<@id>`.
* Se admiten prefijos como `discord:`/`user:` (usuarios) y `channel:` (DMs grupales).
* Usa `*` para permitir cualquier remitente/canal.
* Cuando `guilds.<id>.channels` está presente, los canales no listados se rechazan de forma predeterminada.
* Cuando se omite `guilds.<id>.channels`, se permiten todos los canales en el servidor incluido en la lista de permitidos.
* Para permitir **ningún canal**, establece `channels.discord.groupPolicy: "disabled"` (o mantén una lista de permitidos vacía).
* El asistente de configuración acepta nombres de `Guild/Channel` (públicos + privados) y los resuelve a IDs cuando es posible.
* Al inicio, OpenClaw resuelve los nombres de canales/usuarios en las listas de permitidos a IDs (cuando el bot puede buscar miembros)
  y registra la asignación en los logs; las entradas no resueltas se mantienen tal como se escribieron.

Notas sobre comandos nativos:

* Los comandos registrados reflejan los comandos de chat de OpenClaw.
* Los comandos nativos respetan las mismas listas de permitidos que los mensajes de DM/servidor (`channels.discord.dm.allowFrom`, `channels.discord.guilds`, reglas por canal).
* Los comandos de barra diagonal (slash commands) pueden seguir siendo visibles en la UI de Discord para usuarios que no estén en la lista de permitidos; OpenClaw aplica las listas de permitidos en la ejecución y responde «no autorizado».

<div id="tool-actions">
  ## Acciones de la herramienta
</div>

El agente puede llamar a `discord` con acciones como:

* `react` / `reactions` (agregar o listar reacciones)
* `sticker`, `poll`, `permissions`
* `readMessages`, `sendMessage`, `editMessage`, `deleteMessage`
* Los payloads de herramientas de lectura/búsqueda/fijado incluyen `timestampMs` normalizado (milisegundos desde la época UTC) y `timestampUtc`, junto con el `timestamp` sin procesar de Discord.
* `threadCreate`, `threadList`, `threadReply`
* `pinMessage`, `unpinMessage`, `listPins`
* `searchMessages`, `memberInfo`, `roleInfo`, `roleAdd`, `roleRemove`, `emojiList`
* `channelInfo`, `channelList`, `voiceStatus`, `eventList`, `eventCreate`
* `timeout`, `kick`, `ban`

Los ID de mensaje de Discord se exponen en el contexto inyectado (`[discord message id: …]` y líneas de historial) para que el agente pueda dirigirse a ellos específicamente.
Los emojis pueden ser unicode (por ejemplo, `✅`) o usar sintaxis de emoji personalizada como `<:party_blob:1234567890>`.

<div id="safety-ops">
  ## Seguridad y operaciones
</div>

* Trata el token del bot como una contraseña; utiliza preferentemente la variable de entorno `DISCORD_BOT_TOKEN` en hosts supervisados o restringe los permisos del archivo de configuración.
* Concede al bot solo los permisos que necesita (normalmente Read/Send Messages).
* Si el bot se queda bloqueado o limitado por *rate limiting*, reinicia el Gateway (`openclaw gateway --force`) después de confirmar que ningún otro proceso posee la sesión de Discord.