---
title: Mensaje
summary: "Referencia de la CLI para `openclaw message` (envío + acciones de canal)"
read_when:
  - Añadir o modificar acciones de la CLI para mensajes
  - Cambiar el comportamiento del canal saliente
---

<div id="openclaw-message">
  # `openclaw message`
</div>

Comando único de salida para enviar mensajes y realizar acciones en canales
(Discord/Google Chat/Slack/Mattermost (complemento)/Telegram/WhatsApp/Signal/iMessage/MS Teams).

<div id="usage">
  ## Uso
</div>

```
openclaw message <subcommand> [flags]
```

Selección de canal:

* `--channel` es obligatorio si hay más de un canal configurado.
* Si hay exactamente un canal configurado, se convierte en el valor predeterminado.
* Valores: `whatsapp|telegram|discord|googlechat|slack|mattermost|signal|imessage|msteams` (Mattermost requiere complemento)

Formatos de destino (`--target`):

* WhatsApp: E.164 o JID de grupo
* Telegram: id de chat o `@username`
* Discord: `channel:&lt;id&gt;` o `user:&lt;id&gt;` (o mención `&lt;@id&gt;`; los IDs numéricos sin formato se tratan como canales)
* Google Chat: `spaces/&lt;spaceId&gt;` o `users/&lt;userId&gt;`
* Slack: `channel:&lt;id&gt;` o `user:&lt;id&gt;` (se acepta el ID de canal sin formato)
* Mattermost (complemento): `channel:&lt;id&gt;`, `user:&lt;id&gt;`, o `@username` (los IDs sin prefijo se tratan como canales)
* Signal: `+E.164`, `group:&lt;id&gt;`, `signal:+E.164`, `signal:group:&lt;id&gt;`, o `username:&lt;name&gt;`/`u:&lt;name&gt;`
* iMessage: handle, `chat_id:&lt;id&gt;`, `chat_guid:&lt;guid&gt;`, o `chat_identifier:&lt;id&gt;`
* MS Teams: id de conversación (`19:...@thread.tacv2`) o `conversation:&lt;id&gt;` o `user:&lt;aad-object-id&gt;`

Búsqueda por nombre:

* Para proveedores compatibles (Discord, Slack, etc.), los nombres de canal como `Help` o `#help` se resuelven mediante la caché de directorio.
* Si la caché no contiene el valor, OpenClaw intentará una búsqueda de directorio en tiempo real cuando el proveedor lo admita.


<div id="common-flags">
  ## Opciones comunes
</div>

- `--channel <name>`
- `--account <id>`
- `--target <dest>` (canal o usuario de destino para send/poll/read/etc)
- `--targets <name>` (repetible; solo difusión)
- `--json`
- `--dry-run`
- `--verbose`

<div id="actions">
  ## Acciones
</div>

<div id="core">
  ### Núcleo
</div>

- `send`
  - Canales: WhatsApp/Telegram/Discord/Google Chat/Slack/Mattermost (complemento)/Signal/iMessage/MS Teams
  - Obligatorio: `--target`, más `--message` o `--media`
  - Opcional: `--media`, `--reply-to`, `--thread-id`, `--gif-playback`
  - Solo Telegram: `--buttons` (requiere `channels.telegram.capabilities.inlineButtons` para permitirlo)
  - Solo Telegram: `--thread-id` (ID de tema de foro)
  - Solo Slack: `--thread-id` (marca de tiempo del hilo; `--reply-to` usa el mismo campo)
  - Solo WhatsApp: `--gif-playback`

- `poll`
  - Canales: WhatsApp/Discord/MS Teams
  - Obligatorio: `--target`, `--poll-question`, `--poll-option` (puede repetirse)
  - Opcional: `--poll-multi`
  - Solo Discord: `--poll-duration-hours`, `--message`

- `react`
  - Canales: Discord/Google Chat/Slack/Telegram/WhatsApp/Signal
  - Obligatorio: `--message-id`, `--target`
  - Opcional: `--emoji`, `--remove`, `--participant`, `--from-me`, `--target-author`, `--target-author-uuid`
  - Nota: `--remove` requiere `--emoji` (omite `--emoji` para eliminar tus propias reacciones donde se admita; consulta /tools/reactions)
  - Solo WhatsApp: `--participant`, `--from-me`
  - Reacciones en grupos de Signal: `--target-author` o `--target-author-uuid` obligatorio

- `reactions`
  - Canales: Discord/Google Chat/Slack
  - Obligatorio: `--message-id`, `--target`
  - Opcional: `--limit`

- `read`
  - Canales: Discord/Slack
  - Obligatorio: `--target`
  - Opcional: `--limit`, `--before`, `--after`
  - Solo Discord: `--around`

- `edit`
  - Canales: Discord/Slack
  - Obligatorio: `--message-id`, `--message`, `--target`

- `delete`
  - Canales: Discord/Slack/Telegram
  - Obligatorio: `--message-id`, `--target`

- `pin` / `unpin`
  - Canales: Discord/Slack
  - Obligatorio: `--message-id`, `--target`

- `pins` (lista)
  - Canales: Discord/Slack
  - Obligatorio: `--target`

- `permissions`
  - Canales: Discord
  - Obligatorio: `--target`

- `search`
  - Canales: Discord
  - Obligatorio: `--guild-id`, `--query`
  - Opcional: `--channel-id`, `--channel-ids` (puede repetirse), `--author-id`, `--author-ids` (puede repetirse), `--limit`

<div id="threads">
  ### Hilos
</div>

- `thread create`
  - Canales: Discord
  - Obligatorio: `--thread-name`, `--target` (ID de canal)
  - Opcional: `--message-id`, `--auto-archive-min`

- `thread list`
  - Canales: Discord
  - Obligatorio: `--guild-id`
  - Opcional: `--channel-id`, `--include-archived`, `--before`, `--limit`

- `thread reply`
  - Canales: Discord
  - Obligatorio: `--target` (ID de hilo), `--message`
  - Opcional: `--media`, `--reply-to`

<div id="emojis">
  ### Emojis
</div>

- `emoji list`
  - Discord: `--guild-id`
  - Slack: sin opciones adicionales

- `emoji upload`
  - Canales: Discord
  - Obligatorios: `--guild-id`, `--emoji-name`, `--media`
  - Opcional: `--role-ids` (repetible)

<div id="stickers">
  ### Stickers
</div>

- `sticker send`
  - Canales: Discord
  - Requeridos: `--target`, `--sticker-id` (repetible)
  - Opcional: `--message`

- `sticker upload`
  - Canales: Discord
  - Requeridos: `--guild-id`, `--sticker-name`, `--sticker-desc`, `--sticker-tags`, `--media`

<div id="roles-channels-members-voice">
  ### Roles / Canales / Miembros / Voz
</div>

- `role info` (Discord): `--guild-id`
- `role add` / `role remove` (Discord): `--guild-id`, `--user-id`, `--role-id`
- `channel info` (Discord): `--target`
- `channel list` (Discord): `--guild-id`
- `member info` (Discord/Slack): `--user-id` (+ `--guild-id` en Discord)
- `voice status` (Discord): `--guild-id`, `--user-id`

<div id="events">
  ### Eventos
</div>

- `event list` (Discord): `--guild-id`
- `event create` (Discord): `--guild-id`, `--event-name`, `--start-time`
  - Opcionales: `--end-time`, `--desc`, `--channel-id`, `--location`, `--event-type`

<div id="moderation-discord">
  ### Moderación (Discord)
</div>

- `timeout`: `--guild-id`, `--user-id` (opcional `--duration-min` o `--until`; omite ambos para quitar el timeout)
- `kick`: `--guild-id`, `--user-id` (+ `--reason`)
- `ban`: `--guild-id`, `--user-id` (+ `--delete-days`, `--reason`)
  - `timeout` también admite `--reason`

<div id="broadcast">
  ### Difusión
</div>

- `broadcast`
  - Canales: cualquier canal configurado; usa `--channel all` para dirigirte a todos los proveedores
  - Requerido: `--targets` (se puede repetir)
  - Opcional: `--message`, `--media`, `--dry-run`

<div id="examples">
  ## Ejemplos
</div>

Envía una respuesta en Discord:

```
openclaw message send --channel discord \
  --target channel:123 --message "hi" --reply-to 456
```

Crea una encuesta en Discord:

```
openclaw message poll --channel discord \
  --target channel:123 \
  --poll-question "Snack?" \
  --poll-option Pizza --poll-option Sushi \
  --poll-multi --poll-duration-hours 48
```

Envía un mensaje proactivo en Teams:

```
openclaw message send --channel msteams \
  --target conversation:19:abc@thread.tacv2 --message "hi"
```

Crea una encuesta en Teams:

```
openclaw message poll --channel msteams \
  --target conversation:19:abc@thread.tacv2 \
  --poll-question "Lunch?" \
  --poll-option Pizza --poll-option Sushi
```

Añadir una reacción en Slack:

```
openclaw message react --channel slack \
  --target C123 --message-id 456 --emoji "✅"
```

Reaccionar en un grupo de Signal:

```
openclaw message react --channel signal \
  --target signal:group:abc123 --message-id 1737630212345 \
  --emoji "✅" --target-author-uuid 123e4567-e89b-12d3-a456-426614174000
```

Enviar botones inline en Telegram:

```
openclaw message send --channel telegram --target @mychat --message "Choose:" \
  --buttons '[ [{"text":"Yes","callback_data":"cmd:yes"}], [{"text":"No","callback_data":"cmd:no"}] ]'
```
