---
title: Signal
summary: "Soporte de Signal mediante signal-cli (JSON-RPC + SSE), configuración y modelo de numeración"
read_when:
  - Configurar el soporte de Signal
  - Depurar el envío y la recepción en Signal
---

<div id="signal-signal-cli">
  # Signal (signal-cli)
</div>

Estado: integración de CLI externa. El Gateway se comunica con `signal-cli` sobre HTTP JSON-RPC + SSE.

<div id="quick-setup-beginner">
  ## Configuración rápida (para principiantes)
</div>

1. Usa un **número de Signal separado** para el bot (recomendado).
2. Instala `signal-cli` (requiere Java).
3. Vincula el dispositivo del bot e inicia el proceso daemon:
   * `signal-cli link -n "OpenClaw"`
4. Configura OpenClaw e inicia el Gateway.

Configuración mínima:

```json5
{
  channels: {
    signal: {
      enabled: true,
      account: "+15551234567",
      cliPath: "signal-cli",
      dmPolicy: "pairing",
      allowFrom: ["+15557654321"]
    }
  }
}
```

<div id="what-it-is">
  ## Qué es
</div>

* Canal de Signal a través de `signal-cli` (no la biblioteca integrada `libsignal`).
* Enrutamiento determinista: las respuestas siempre regresan a Signal.
* Los mensajes directos (DMs) comparten la sesión principal del agente; los grupos están aislados (`agent:<agentId>:signal:group:<groupId>`).

<div id="config-writes">
  ## Escrituras de configuración
</div>

De forma predeterminada, Signal puede escribir actualizaciones de configuración iniciadas por `/config set|unset` (requiere `commands.config: true`).

Desactívalo con:

```json5
{
  channels: { signal: { configWrites: false } }
}
```

<div id="the-number-model-important">
  ## El modelo de numeración (importante)
</div>

* El Gateway se conecta a un **dispositivo de Signal** (la cuenta de `signal-cli`).
* Si ejecutas el bot en **tu cuenta personal de Signal**, ignorará tus propios mensajes (protección contra bucles).
* Para el caso &quot;yo le escribo al bot y me responde&quot;, usa un **número de bot separado**.

<div id="setup-fast-path">
  ## Configuración (vía rápida)
</div>

1. Instala `signal-cli` (se requiere Java).
2. Enlaza una cuenta de bot:
   * `signal-cli link -n "OpenClaw"` y luego escanea el código QR en Signal.
3. Configura Signal e inicia el Gateway.

Ejemplo:

```json5
{
  channels: {
    signal: {
      enabled: true,
      account: "+15551234567",
      cliPath: "signal-cli",
      dmPolicy: "pairing",
      allowFrom: ["+15557654321"]
    }
  }
}
```

Compatibilidad con múltiples cuentas: utiliza `channels.signal.accounts` con configuración por cuenta y un `name` opcional. Consulta [`gateway/configuration`](/es/gateway/configuration#telegramaccounts--discordaccounts--slackaccounts--signalaccounts--imessageaccounts) para ver el patrón común.

<div id="external-daemon-mode-httpurl">
  ## Modo de daemon externo (httpUrl)
</div>

Si quieres administrar `signal-cli` tú mismo (arranques en frío lentos de la JVM, inicialización de contenedores o procesadores compartidos), ejecuta el daemon por separado y configura OpenClaw para que apunte a él:

```json5
{
  channels: {
    signal: {
      httpUrl: "http://127.0.0.1:8080",
      autoStart: false
    }
  }
}
```

Esto omite el auto-spawn y la espera de arranque dentro de OpenClaw. Para arranques lentos al usar auto-spawn, configura `channels.signal.startupTimeoutMs`.

<div id="access-control-dms-groups">
  ## Control de acceso (MDs + grupos)
</div>

MDs:

* Predeterminado: `channels.signal.dmPolicy = "pairing"`.
* Los remitentes desconocidos reciben un código de emparejamiento; los mensajes se ignoran hasta que se aprueban (los códigos caducan al cabo de 1 hora).
* Aprobar a través de:
  * `openclaw pairing list signal`
  * `openclaw pairing approve signal <CODE>`
* El emparejamiento es el mecanismo de intercambio de tokens predeterminado para las MDs de Signal. Detalles: [Emparejamiento](/es/start/pairing)
* Los remitentes identificados solo por UUID (desde `sourceUuid`) se almacenan como `uuid:<id>` en `channels.signal.allowFrom`.

Grupos:

* `channels.signal.groupPolicy = open | allowlist | disabled`.
* `channels.signal.groupAllowFrom` controla quién puede activar en grupos cuando `allowlist` está configurado.

<div id="how-it-works-behavior">
  ## Cómo funciona (comportamiento)
</div>

* `signal-cli` se ejecuta como un daemon; el Gateway lee eventos mediante SSE.
* Los mensajes entrantes se normalizan en el envoltorio de canal compartido.
* Las respuestas siempre se enrutan de nuevo al mismo número o grupo.

<div id="media-limits">
  ## Contenido multimedia + límites
</div>

* El texto saliente se fragmenta según `channels.signal.textChunkLimit` (valor predeterminado: 4000).
* Fragmentación opcional por saltos de línea: establece `channels.signal.chunkMode="newline"` para dividir en líneas en blanco (límites de párrafo) antes de fragmentar por longitud.
* Se admiten archivos adjuntos (base64 obtenido de `signal-cli`).
* Límite predeterminado de contenido multimedia: `channels.signal.mediaMaxMb` (valor predeterminado: 8).
* Usa `channels.signal.ignoreAttachments` para omitir la descarga de contenido multimedia.
* El contexto de historial de grupo usa `channels.signal.historyLimit` (o `channels.signal.accounts.*.historyLimit`); en caso contrario, recurre a `messages.groupChat.historyLimit`. Establece `0` para deshabilitarlo (valor predeterminado: 50).

<div id="typing-read-receipts">
  ## Indicadores de escritura + confirmaciones de lectura
</div>

* **Indicadores de escritura**: OpenClaw envía señales de escritura mediante `signal-cli sendTyping` y las actualiza mientras una respuesta está en curso.
* **Confirmaciones de lectura**: cuando `channels.signal.sendReadReceipts` es true, OpenClaw reenvía confirmaciones de lectura para los mensajes directos (DM) permitidos.
* Signal-cli no expone confirmaciones de lectura para grupos.

<div id="reactions-message-tool">
  ## Reacciones (message tool)
</div>

* Usa `message action=react` con `channel=signal`.
* Destinos: remitente E.164 o UUID (usa `uuid:&lt;id&gt;` de la salida de emparejamiento; un UUID sin prefijo también funciona).
* `messageId` es la marca de tiempo de Signal del mensaje al que estás reaccionando.
* Las reacciones de grupo requieren `targetAuthor` o `targetAuthorUuid`.

Ejemplos:

```
message action=react channel=signal target=uuid:123e4567-e89b-12d3-a456-426614174000 messageId=1737630212345 emoji=🔥
message action=react channel=signal target=+15551234567 messageId=1737630212345 emoji=🔥 remove=true
message action=react channel=signal target=signal:group:<groupId> targetAuthor=uuid:<sender-uuid> messageId=1737630212345 emoji=✅
```

Configuración:

* `channels.signal.actions.reactions`: activar/desactivar acciones de reacción (valor predeterminado: true).
* `channels.signal.reactionLevel`: `off | ack | minimal | extensive`.
  * `off`/`ack` desactiva las reacciones del agente (la herramienta de mensajes `react` generará un error).
  * `minimal`/`extensive` activa las reacciones del agente y establece el nivel de orientación.
* Anulaciones por cuenta: `channels.signal.accounts.<id>.actions.reactions`, `channels.signal.accounts.<id>.reactionLevel`.

<div id="delivery-targets-clicron">
  ## Destinos de entrega (CLI/cron)
</div>

* DM: `signal:+15551234567` (o en formato E.164 sin prefijo).
* DM por UUID: `uuid:<id>` (o UUID sin prefijo).
* Grupos: `signal:group:<groupId>`.
* Nombres de usuario: `username:<name>` (si tu cuenta de Signal los admite).

<div id="configuration-reference-signal">
  ## Referencia de configuración (Signal)
</div>

Configuración completa: [Configuration](/es/gateway/configuration)

Opciones del proveedor:

* `channels.signal.enabled`: habilita/deshabilita el arranque del canal.
* `channels.signal.account`: E.164 para la cuenta del bot.
* `channels.signal.cliPath`: ruta a `signal-cli`.
* `channels.signal.httpUrl`: URL completa del daemon (anula host/port).
* `channels.signal.httpHost`, `channels.signal.httpPort`: bind del daemon (por defecto 127.0.0.1:8080).
* `channels.signal.autoStart`: inicia automáticamente el daemon (por defecto true si `httpUrl` no está definido).
* `channels.signal.startupTimeoutMs`: tiempo de espera de inicio en ms (límite 120000).
* `channels.signal.receiveMode`: `on-start | manual`.
* `channels.signal.ignoreAttachments`: omite la descarga de adjuntos.
* `channels.signal.ignoreStories`: ignora historias del daemon.
* `channels.signal.sendReadReceipts`: reenvía acuses de lectura.
* `channels.signal.dmPolicy`: `pairing | allowlist | open | disabled` (por defecto: pairing).
* `channels.signal.allowFrom`: lista de permitidos para DM (E.164 o `uuid:<id>`). `open` requiere `"*"`. Signal no tiene nombres de usuario; utiliza identificadores de teléfono/UUID.
* `channels.signal.groupPolicy`: `open | allowlist | disabled` (por defecto: allowlist).
* `channels.signal.groupAllowFrom`: lista de permitidos para remitentes de grupo.
* `channels.signal.historyLimit`: máximo de mensajes de grupo a incluir como contexto (0 lo desactiva).
* `channels.signal.dmHistoryLimit`: límite de historial de DM en turnos de usuario. Anulaciones por usuario: `channels.signal.dms["<phone_or_uuid>"].historyLimit`.
* `channels.signal.textChunkLimit`: tamaño de fragmento saliente (caracteres).
* `channels.signal.chunkMode`: `length` (por defecto) o `newline` para dividir en líneas en blanco (límites de párrafo) antes de fragmentar por longitud.
* `channels.signal.mediaMaxMb`: límite de tamaño de medios de entrada/salida (MB).

Opciones globales relacionadas:

* `agents.list[].groupChat.mentionPatterns` (Signal no admite menciones nativas).
* `messages.groupChat.mentionPatterns` (respaldo global).
* `messages.responsePrefix`.