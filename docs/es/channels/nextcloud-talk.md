---
title: Nextcloud Talk
summary: "Estado de soporte, capacidades y configuración de Nextcloud Talk"
read_when:
  - Trabajando en las características del canal de Nextcloud Talk
---

<div id="nextcloud-talk-plugin">
  # Nextcloud Talk (complemento)
</div>

Estado: compatible a través de un complemento (bot de webhook). Se admiten mensajes directos, salas, reacciones y mensajes en Markdown.

<div id="plugin-required">
  ## Complemento requerido
</div>

Nextcloud Talk se distribuye como un complemento y no está incluido en la instalación básica.

Instálalo mediante la CLI (registro de npm):

```bash
openclaw plugins install @openclaw/nextcloud-talk
```

Checkout local (cuando se ejecuta desde un repositorio Git):

```bash
openclaw plugins install ./extensions/nextcloud-talk
```

Si seleccionas Nextcloud Talk durante la configuración/onboarding y se detecta un checkout de Git,
OpenClaw sugerirá automáticamente la ruta de instalación local.

Detalles: [Complementos](/es/plugin)

<div id="quick-setup-beginner">
  ## Configuración rápida (principiantes)
</div>

1. Instala el complemento Nextcloud Talk.
2. En tu servidor Nextcloud, crea un bot:
   ```bash
   ./occ talk:bot:install "OpenClaw" "<shared-secret>" "<webhook-url>" --feature reaction
   ```
3. Habilita el bot en la configuración de la sala de destino.
4. Configura OpenClaw:
   * Config: `channels.nextcloud-talk.baseUrl` + `channels.nextcloud-talk.botSecret`
   * O bien variable de entorno: `NEXTCLOUD_TALK_BOT_SECRET` (solo cuenta predeterminada)
5. Reinicia el Gateway (o termina el onboarding).

Configuración mínima:

```json5
{
  channels: {
    "nextcloud-talk": {
      enabled: true,
      baseUrl: "https://cloud.example.com",
      botSecret: "shared-secret",
      dmPolicy: "pairing"
    }
  }
}
```

<div id="notes">
  ## Notas
</div>

* Los bots no pueden iniciar DMs. El usuario debe enviar primero un mensaje al bot.
* La URL del webhook debe ser accesible desde el Gateway; configura `webhookPublicUrl` si está detrás de un proxy.
* Las subidas de archivos multimedia no son compatibles con la API del bot; el contenido multimedia se envía como URLs.
* El payload del webhook no distingue entre DMs y salas; configura `apiUser` + `apiPassword` para habilitar las búsquedas por tipo de sala (de lo contrario, los DMs se tratan como salas).

<div id="access-control-dms">
  ## Control de acceso (DMs)
</div>

* Valor predeterminado: `channels.nextcloud-talk.dmPolicy = "pairing"`. Los remitentes desconocidos reciben un código de emparejamiento.
* Aprobar mediante:
  * `openclaw pairing list nextcloud-talk`
  * `openclaw pairing approve nextcloud-talk <CODE>`
* DMs públicas: `channels.nextcloud-talk.dmPolicy="open"` (permite aceptar mensajes de cualquier usuario sin restricciones) más `channels.nextcloud-talk.allowFrom=["*"]`.

<div id="rooms-groups">
  ## Salas (grupos)
</div>

* Valor predeterminado: `channels.nextcloud-talk.groupPolicy = "allowlist"` (controlado por menciones).
* Incluye salas en la lista de permitidos con `channels.nextcloud-talk.rooms`:

```json5
{
  channels: {
    "nextcloud-talk": {
      rooms: {
        "room-token": { requireMention: true }
      }
    }
  }
}
```

* Para no permitir ninguna sala, deja la lista de permitidos vacía o establece `channels.nextcloud-talk.groupPolicy="disabled"`.

<div id="capabilities">
  ## Capacidades
</div>

| Función | Estado |
|---------|--------|
| Mensajes directos | Disponible |
| Salas | Disponible |
| Hilos | No disponible |
| Contenido multimedia | Solo por URL |
| Reacciones | Disponible |
| Comandos nativos | No disponible |

<div id="configuration-reference-nextcloud-talk">
  ## Referencia de configuración (Nextcloud Talk)
</div>

Configuración completa: [Configuración](/es/gateway/configuration)

Opciones del proveedor:

* `channels.nextcloud-talk.enabled`: habilitar/deshabilitar el inicio del canal.
* `channels.nextcloud-talk.baseUrl`: URL de la instancia de Nextcloud.
* `channels.nextcloud-talk.botSecret`: secreto compartido del bot.
* `channels.nextcloud-talk.botSecretFile`: ruta del archivo de secreto.
* `channels.nextcloud-talk.apiUser`: usuario de API para búsquedas de salas (detección de mensajes directos).
* `channels.nextcloud-talk.apiPassword`: contraseña de API/app para búsquedas de salas.
* `channels.nextcloud-talk.apiPasswordFile`: ruta del archivo de contraseña de API.
* `channels.nextcloud-talk.webhookPort`: puerto del listener de webhook (predeterminado: 8788).
* `channels.nextcloud-talk.webhookHost`: host del webhook (predeterminado: 0.0.0.0).
* `channels.nextcloud-talk.webhookPath`: ruta del webhook (predeterminado: /nextcloud-talk-webhook).
* `channels.nextcloud-talk.webhookPublicUrl`: URL de webhook accesible externamente.
* `channels.nextcloud-talk.dmPolicy`: `pairing | allowlist | open | disabled`.
* `channels.nextcloud-talk.allowFrom`: lista de permitidos de mensajes directos (IDs de usuario). `open` requiere `"*"`.
* `channels.nextcloud-talk.groupPolicy`: `allowlist | open | disabled`.
* `channels.nextcloud-talk.groupAllowFrom`: lista de permitidos de grupos (IDs de usuario).
* `channels.nextcloud-talk.rooms`: configuración y lista de permitidos por sala.
* `channels.nextcloud-talk.historyLimit`: límite de historial de grupos (0 lo deshabilita).
* `channels.nextcloud-talk.dmHistoryLimit`: límite de historial de mensajes directos (0 lo deshabilita).
* `channels.nextcloud-talk.dms`: anulaciones por mensaje directo (historyLimit).
* `channels.nextcloud-talk.textChunkLimit`: tamaño del fragmento de texto saliente (caracteres).
* `channels.nextcloud-talk.chunkMode`: `length` (predeterminado) o `newline` para dividir en líneas en blanco (límites de párrafo) antes de fragmentar por longitud.
* `channels.nextcloud-talk.blockStreaming`: deshabilitar el streaming por bloques para este canal.
* `channels.nextcloud-talk.blockStreamingCoalesce`: ajuste de combinación de streaming por bloques.
* `channels.nextcloud-talk.mediaMaxMb`: límite de tamaño de contenido multimedia entrante (MB).