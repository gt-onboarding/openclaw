---
title: iMessage
summary: "Soporte de iMessage mediante imsg (JSON-RPC a través de stdio), configuración y enrutamiento de chat_id"
read_when:
  - Configurar el soporte de iMessage
  - Depurar el envío y la recepción de iMessage
---

<div id="imessage-imsg">
  # iMessage (imsg)
</div>

Estado: integración externa con la CLI. El Gateway inicia `imsg rpc` (JSON-RPC a través de stdio).

<div id="quick-setup-beginner">
  ## Configuración rápida (principiante)
</div>

1. Asegúrate de que Mensajes tenga la sesión iniciada en este Mac.
2. Instala `imsg`:
   * `brew install steipete/tap/imsg`
3. Configura OpenClaw con `channels.imessage.cliPath` y `channels.imessage.dbPath`.
4. Inicia el Gateway y acepta cualquier solicitud de macOS (Automatización + Acceso completo al disco).

Configuración mínima:

```json5
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "/usr/local/bin/imsg",
      dbPath: "/Users/<you>/Library/Messages/chat.db"
    }
  }
}
```

<div id="what-it-is">
  ## Qué es
</div>

* Canal de iMessage basado en `imsg` en macOS.
* Enrutamiento determinista: las respuestas siempre se envían de vuelta a iMessage.
* Los DMs (mensajes directos) comparten la sesión principal del agente; los grupos están aislados (`agent:<agentId>:imessage:group:<chat_id>`).
* Si llega un hilo con varios participantes con `is_group=false`, aún puedes aislarlo por `chat_id` usando `channels.imessage.groups` (consulta “Hilos tipo grupo” más abajo).

<div id="config-writes">
  ## Escrituras de configuración
</div>

De forma predeterminada, iMessage tiene permiso para escribir actualizaciones de configuración activadas por `/config set|unset` (requiere `commands.config: true`).

Desactiva con:

```json5
{
  channels: { imessage: { configWrites: false } }
}
```

<div id="requirements">
  ## Requisitos
</div>

* macOS con sesión iniciada en Mensajes.
* Acceso total al disco para OpenClaw + `imsg` (acceso a la base de datos de Mensajes).
* Permiso de Automatización para enviar.
* `channels.imessage.cliPath` puede apuntar a cualquier comando que haga de proxy de stdin/stdout (por ejemplo, un script wrapper que se conecte por SSH a otro Mac y ejecute `imsg rpc`).

<div id="setup-fast-path">
  ## Configuración (vía rápida)
</div>

1. Asegúrate de que Mensajes tenga la sesión iniciada en este Mac.
2. Configura iMessage e inicia el Gateway.

<div id="dedicated-bot-macos-user-for-isolated-identity">
  ### Usuario de macOS dedicado para el bot (para aislar la identidad)
</div>

Si quieres que el bot envíe desde una **identidad de iMessage separada** (y mantener limpia tu app Mensajes personal), usa un Apple ID dedicado y un usuario de macOS dedicado.

1. Crea un Apple ID dedicado (ejemplo: `my-cool-bot@icloud.com`).
   * Es posible que Apple requiera un número de teléfono para la verificación / 2FA.
2. Crea un usuario de macOS (ejemplo: `openclawhome`) e inicia sesión con él.
3. Abre Mensajes en ese usuario de macOS e inicia sesión en iMessage usando el Apple ID del bot.
4. Activa el inicio de sesión remoto (System Settings → General → Sharing → Remote Login).
5. Instala `imsg`:
   * `brew install steipete/tap/imsg`
6. Configura SSH de forma que `ssh <bot-macos-user>@localhost true` funcione sin contraseña.
7. Haz que `channels.imessage.accounts.bot.cliPath` apunte a un script wrapper de SSH que ejecute `imsg` como el usuario del bot.

Nota para la primera ejecución: el envío/recepción puede requerir aprobaciones en la GUI (Automation + Full Disk Access) en el *usuario de macOS del bot*. Si `imsg rpc` parece quedarse colgado o salir, inicia sesión en ese usuario (Screen Sharing ayuda), ejecuta una vez `imsg chats --limit 1` / `imsg send ...`, aprueba los avisos y vuelve a intentarlo.

Ejemplo de wrapper (`chmod +x`). Sustituye `<bot-macos-user>` por tu nombre de usuario real de macOS:

```bash
#!/usr/bin/env bash
set -euo pipefail

# Ejecuta SSH interactivo una vez primero para aceptar las claves del host:
#   ssh <bot-macos-user>@localhost true
exec /usr/bin/ssh -o BatchMode=yes -o ConnectTimeout=5 -T <bot-macos-user>@localhost \
  "/usr/local/bin/imsg" "$@"
```

Ejemplo de configuración:

```json5
{
  channels: {
    imessage: {
      enabled: true,
      accounts: {
        bot: {
          name: "Bot",
          enabled: true,
          cliPath: "/path/to/imsg-bot",
          dbPath: "/Users/<bot-macos-user>/Library/Messages/chat.db"
        }
      }
    }
  }
}
```

Para configuraciones con una sola cuenta, usa las opciones planas (`channels.imessage.cliPath`, `channels.imessage.dbPath`) en lugar del mapa `accounts`.

<div id="remotessh-variant-optional">
  ### Variante remota/SSH (opcional)
</div>

Si quieres usar iMessage en otro Mac, configura `channels.imessage.cliPath` como un script envoltorio que ejecute `imsg` en el host remoto de macOS a través de SSH. OpenClaw solo necesita stdio (entrada/salida estándar).

Ejemplo de script envoltorio:

```bash
#!/usr/bin/env bash
exec ssh -T gateway-host imsg "$@"
```

**Adjuntos remotos:** Cuando `cliPath` apunta a un host remoto vía SSH, las rutas de los archivos adjuntos en la base de datos de Messages hacen referencia a archivos en la máquina remota. OpenClaw puede recuperarlos automáticamente mediante SCP configurando `channels.imessage.remoteHost`:

```json5
{
  channels: {
    imessage: {
      cliPath: "~/imsg-ssh",                     // SSH wrapper to remote Mac
      remoteHost: "user@gateway-host",           // para transferencia de archivos por SCP
      includeAttachments: true
    }
  }
}
```

Si no defines `remoteHost`, OpenClaw intenta detectarlo automáticamente analizando el comando SSH en tu script wrapper. Se recomienda configurarlo explícitamente para mayor confiabilidad.

<div id="remote-mac-via-tailscale-example">
  #### Mac remoto vía Tailscale (ejemplo)
</div>

Si el Gateway se ejecuta en un host/VM con Linux pero iMessage debe ejecutarse en un Mac, Tailscale es el puente más sencillo: el Gateway se comunica con el Mac a través del tailnet, ejecuta `imsg` vía SSH y transfiere los archivos adjuntos de vuelta con SCP.

Arquitectura:

```
┌──────────────────────────────┐          SSH (imsg rpc)          ┌──────────────────────────┐
│ Gateway host (Linux/VM)      │──────────────────────────────────▶│ Mac with Messages + imsg │
│ - openclaw gateway           │          SCP (attachments)        │ - Messages signed in     │
│ - channels.imessage.cliPath  │◀──────────────────────────────────│ - Remote Login enabled   │
└──────────────────────────────┘                                   └──────────────────────────┘
              ▲
              │ Tailscale tailnet (hostname or 100.x.y.z)
              ▼
        user@gateway-host
```

Ejemplo concreto de configuración (nombre de host de Tailscale):

```json5
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "~/.openclaw/scripts/imsg-ssh",
      remoteHost: "bot@mac-mini.tailnet-1234.ts.net",
      includeAttachments: true,
      dbPath: "/Users/bot/Library/Messages/chat.db"
    }
  }
}
```

Ejemplo de script *wrapper* (`~/.openclaw/scripts/imsg-ssh`):

```bash
#!/usr/bin/env bash
exec ssh -T bot@mac-mini.tailnet-1234.ts.net imsg "$@"
```

Notas:

* Asegúrate de que el Mac haya iniciado sesión en Mensajes y que el Inicio de sesión remoto esté habilitado.
* Usa claves SSH para que `ssh bot@mac-mini.tailnet-1234.ts.net` funcione sin pedir contraseña.
* `remoteHost` debe coincidir con el destino SSH para que SCP pueda recuperar los archivos adjuntos.

Compatibilidad con varias cuentas: usa `channels.imessage.accounts` con configuración por cuenta y `name` opcional. Consulta [`gateway/configuration`](/es/gateway/configuration#telegramaccounts--discordaccounts--slackaccounts--signalaccounts--imessageaccounts) para el patrón compartido. No hagas commit de `~/.openclaw/openclaw.json` (suele contener tokens).

<div id="access-control-dms-groups">
  ## Control de acceso (MD + grupos)
</div>

MD (mensajes directos):

* Predeterminado: `channels.imessage.dmPolicy = "pairing"`.
* Los remitentes desconocidos reciben un código de emparejamiento; los mensajes se ignoran hasta que se aprueben (los códigos caducan después de 1 hora).
* Aprueba mediante:
  * `openclaw pairing list imessage`
  * `openclaw pairing approve imessage <CODE>`
* El emparejamiento es el intercambio de tokens predeterminado para los MD de iMessage. Detalles: [Pairing](/es/start/pairing)

Grupos:

* `channels.imessage.groupPolicy = open | allowlist | disabled`.
* `channels.imessage.groupAllowFrom` controla quién puede activar en grupos cuando se establece en `allowlist` (lista de permitidos).
* El control de acceso por mención usa `agents.list[].groupChat.mentionPatterns` (o `messages.groupChat.mentionPatterns`) porque iMessage no tiene metadatos nativos de menciones.
* Anulación multiagente: configura patrones por agente en `agents.list[].groupChat.mentionPatterns`.

<div id="how-it-works-behavior">
  ## Cómo funciona (comportamiento)
</div>

* `imsg` emite eventos de mensajes; el Gateway los normaliza al formato de canal compartido.
* Las respuestas siempre se envían de vuelta al mismo ID de chat o handle.

<div id="group-ish-threads-is_groupfalse">
  ## Hilos tipo grupo (`is_group=false`)
</div>

Algunos hilos de iMessage pueden tener varios participantes pero seguir apareciendo con `is_group=false`, dependiendo de cómo Messages almacena el identificador del chat.

Si configuras explícitamente un `chat_id` en `channels.imessage.groups`, OpenClaw trata ese hilo como un “grupo” para:

* aislamiento de sesión (clave de sesión separada `agent:<agentId>:imessage:group:<chat_id>`)
* comportamiento de lista de permitidos de grupo / control de acceso por menciones

Ejemplo:

```json5
{
  channels: {
    imessage: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15555550123"],
      groups: {
        "42": { "requireMention": false }
      }
    }
  }
}
```

Esto es útil cuando quieres una personalidad/modelo aislado para un hilo específico (consulta [Multi-agent routing](/es/concepts/multi-agent)). Para el aislamiento del sistema de archivos, ve a [Sandboxing](/es/gateway/sandboxing).

<div id="media-limits">
  ## Contenido multimedia + límites
</div>

* Incorporación opcional de archivos adjuntos mediante `channels.imessage.includeAttachments`.
* Límite de tamaño de contenido multimedia mediante `channels.imessage.mediaMaxMb`.

<div id="limits">
  ## Límites
</div>

* El texto de salida se segmenta según `channels.imessage.textChunkLimit` (valor predeterminado 4000).
* Segmentación opcional por saltos de línea: establece `channels.imessage.chunkMode="newline"` para dividir en líneas en blanco (límites de párrafo) antes de la segmentación por longitud.
* Las subidas de contenido multimedia están limitadas por `channels.imessage.mediaMaxMb` (valor predeterminado 16).

<div id="addressing-delivery-targets">
  ## Direccionamiento / destinos de entrega
</div>

Es preferible usar `chat_id` para el enrutamiento estable:

* `chat_id:123` (preferido)
* `chat_guid:...`
* `chat_identifier:...`
* identificadores directos: `imessage:+1555` / `sms:+1555` / `user@example.com`

Lista los chats:

```
imsg chats --limit 20
```

<div id="configuration-reference-imessage">
  ## Referencia de configuración (iMessage)
</div>

Configuración completa: [Configuration](/es/gateway/configuration)

Opciones del proveedor:

* `channels.imessage.enabled`: habilitar/deshabilitar el inicio del canal.
* `channels.imessage.cliPath`: ruta a `imsg`.
* `channels.imessage.dbPath`: ruta a la base de datos de Messages.
* `channels.imessage.remoteHost`: host SSH para la transferencia de adjuntos mediante SCP cuando `cliPath` apunta a un Mac remoto (por ejemplo, `user@gateway-host`). Se detecta automáticamente a partir del wrapper SSH si no se establece.
* `channels.imessage.service`: `imessage | sms | auto`.
* `channels.imessage.region`: región de SMS.
* `channels.imessage.dmPolicy`: `pairing | allowlist | open | disabled` (valor predeterminado: pairing).
* `channels.imessage.allowFrom`: lista de permitidos para MD (handles, correos electrónicos, números E.164 o `chat_id:*`). `open` requiere `"*"`. iMessage no tiene nombres de usuario; utiliza handles o destinos de chat.
* `channels.imessage.groupPolicy`: `open | allowlist | disabled` (valor predeterminado: allowlist).
* `channels.imessage.groupAllowFrom`: lista de permitidos para remitentes de grupo.
* `channels.imessage.historyLimit` / `channels.imessage.accounts.*.historyLimit`: máximo de mensajes de grupo que se incluirán como contexto (0 lo deshabilita).
* `channels.imessage.dmHistoryLimit`: límite de historial de MD en turnos de usuario. Anulaciones por usuario: `channels.imessage.dms["<handle>"].historyLimit`.
* `channels.imessage.groups`: valores predeterminados por grupo + lista de permitidos (usa `"*"` para valores predeterminados globales).
* `channels.imessage.includeAttachments`: incorpora adjuntos en el contexto.
* `channels.imessage.mediaMaxMb`: límite de medios entrantes/salientes (MB).
* `channels.imessage.textChunkLimit`: tamaño de fragmento saliente (caracteres).
* `channels.imessage.chunkMode`: `length` (predeterminado) o `newline` para dividir en líneas en blanco (límites de párrafo) antes de fragmentar por longitud.

Opciones globales relacionadas:

* `agents.list[].groupChat.mentionPatterns` (o `messages.groupChat.mentionPatterns`).
* `messages.responsePrefix`.