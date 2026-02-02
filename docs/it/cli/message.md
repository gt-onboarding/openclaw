---
title: Messaggio
summary: "Riferimento CLI per `openclaw message` (invio + azioni sul canale)"
read_when:
  - Aggiunta o modifica delle azioni della CLI per i messaggi
  - Modifica del comportamento dei canali in uscita
---

<div id="openclaw-message">
  # `openclaw message`
</div>

Singolo comando per inviare messaggi in uscita e azioni sui canali
(Discord/Google Chat/Slack/Mattermost (plugin)/Telegram/WhatsApp/Signal/iMessage/MS Teams).

<div id="usage">
  ## Uso
</div>

```
openclaw message <subcommand> [flag]
```

Selezione del canale:

* `--channel` è obbligatorio se è configurato più di un canale.
* Se è configurato un solo canale, questo diventa il predefinito.
* Valori: `whatsapp|telegram|discord|googlechat|slack|mattermost|signal|imessage|msteams` (Mattermost richiede il plugin)

Formati di destinazione (`--target`):

* WhatsApp: E.164 o JID del gruppo
* Telegram: ID chat o `@username`
* Discord: `channel:<id>` oppure `user:<id>` (oppure menzione `<@id>`; gli ID numerici senza prefisso sono trattati come canali)
* Google Chat: `spaces/<spaceId>` oppure `users/<userId>`
* Slack: `channel:<id>` oppure `user:<id>` (l’ID di canale senza prefisso è accettato)
* Mattermost (plugin): `channel:<id>`, `user:<id>` oppure `@username` (gli ID senza prefisso sono trattati come canali)
* Signal: `+E.164`, `group:<id>`, `signal:+E.164`, `signal:group:<id>` oppure `username:<name>`/`u:<name>`
* iMessage: handle, `chat_id:<id>`, `chat_guid:<guid>` oppure `chat_identifier:<id>`
* MS Teams: ID della conversazione (`19:...@thread.tacv2`) oppure `conversation:<id>` oppure `user:<aad-object-id>`

Risoluzione dei nomi:

* Per i provider supportati (Discord/Slack/ecc.), i nomi di canale come `Help` o `#help` vengono risolti tramite la cache della directory.
* In caso di cache miss, OpenClaw tenterà un lookup della directory in tempo reale quando il provider lo supporta.


<div id="common-flags">
  ## Opzioni comuni
</div>

- `--channel <name>`
- `--account <id>`
- `--target <dest>` (canale o utente di destinazione per send/poll/read/etc)
- `--targets <name>` (ripetibile; solo per broadcast)
- `--json`
- `--dry-run`
- `--verbose`

<div id="actions">
  ## Azioni
</div>

<div id="core">
  ### Core
</div>

- `send`
  - Canali: WhatsApp/Telegram/Discord/Google Chat/Slack/Mattermost (plugin)/Signal/iMessage/MS Teams
  - Obbligatori: `--target`, più `--message` o `--media`
  - Opzionali: `--media`, `--reply-to`, `--thread-id`, `--gif-playback`
  - Solo Telegram: `--buttons` (richiede `channels.telegram.capabilities.inlineButtons` per abilitarlo)
  - Solo Telegram: `--thread-id` (ID del topic del forum)
  - Solo Slack: `--thread-id` (timestamp del thread; `--reply-to` usa lo stesso campo)
  - Solo WhatsApp: `--gif-playback`

- `poll`
  - Canali: WhatsApp/Discord/MS Teams
  - Obbligatori: `--target`, `--poll-question`, `--poll-option` (ripetibile)
  - Opzionali: `--poll-multi`
  - Solo Discord: `--poll-duration-hours`, `--message`

- `react`
  - Canali: Discord/Google Chat/Slack/Telegram/WhatsApp/Signal
  - Obbligatori: `--message-id`, `--target`
  - Opzionali: `--emoji`, `--remove`, `--participant`, `--from-me`, `--target-author`, `--target-author-uuid`
  - Nota: `--remove` richiede `--emoji` (ometti `--emoji` per cancellare le tue reazioni dove supportato; vedi /tools/reactions)
  - Solo WhatsApp: `--participant`, `--from-me`
  - Reazioni nei gruppi Signal: `--target-author` o `--target-author-uuid` obbligatori

- `reactions`
  - Canali: Discord/Google Chat/Slack
  - Obbligatori: `--message-id`, `--target`
  - Opzionali: `--limit`

- `read`
  - Canali: Discord/Slack
  - Obbligatori: `--target`
  - Opzionali: `--limit`, `--before`, `--after`
  - Solo Discord: `--around`

- `edit`
  - Canali: Discord/Slack
  - Obbligatori: `--message-id`, `--message`, `--target`

- `delete`
  - Canali: Discord/Slack/Telegram
  - Obbligatori: `--message-id`, `--target`

- `pin` / `unpin`
  - Canali: Discord/Slack
  - Obbligatori: `--message-id`, `--target`

- `pins` (elenco)
  - Canali: Discord/Slack
  - Obbligatori: `--target`

- `permissions`
  - Canali: Discord
  - Obbligatori: `--target`

- `search`
  - Canali: Discord
  - Obbligatori: `--guild-id`, `--query`
  - Opzionali: `--channel-id`, `--channel-ids` (ripetibile), `--author-id`, `--author-ids` (ripetibile), `--limit`

<div id="threads">
  ### Thread
</div>

- `thread create`
  - Canali: Discord
  - Obbligatori: `--thread-name`, `--target` (ID canale)
  - Opzionali: `--message-id`, `--auto-archive-min`

- `thread list`
  - Canali: Discord
  - Obbligatori: `--guild-id`
  - Opzionali: `--channel-id`, `--include-archived`, `--before`, `--limit`

- `thread reply`
  - Canali: Discord
  - Obbligatori: `--target` (ID thread), `--message`
  - Opzionali: `--media`, `--reply-to`

<div id="emojis">
  ### Emoji
</div>

- `emoji list`
  - Discord: `--guild-id`
  - Slack: nessun flag aggiuntivo

- `emoji upload`
  - Canali: Discord
  - Richiesti: `--guild-id`, `--emoji-name`, `--media`
  - Opzionale: `--role-ids` (ripetibile)

<div id="stickers">
  ### Sticker
</div>

- `sticker send`
  - Canali: Discord
  - Obbligatori: `--target`, `--sticker-id` (ripetibile)
  - Opzionale: `--message`

- `sticker upload`
  - Canali: Discord
  - Obbligatori: `--guild-id`, `--sticker-name`, `--sticker-desc`, `--sticker-tags`, `--media`

<div id="roles-channels-members-voice">
  ### Ruoli / Canali / Membri / Voce
</div>

- `role info` (Discord): `--guild-id`
- `role add` / `role remove` (Discord): `--guild-id`, `--user-id`, `--role-id`
- `channel info` (Discord): `--target`
- `channel list` (Discord): `--guild-id`
- `member info` (Discord/Slack): `--user-id` (+ `--guild-id` per Discord)
- `voice status` (Discord): `--guild-id`, `--user-id`

<div id="events">
  ### Eventi
</div>

- `event list` (Discord): `--guild-id`
- `event create` (Discord): `--guild-id`, `--event-name`, `--start-time`
  - Opzionali: `--end-time`, `--desc`, `--channel-id`, `--location`, `--event-type`

<div id="moderation-discord">
  ### Moderazione (Discord)
</div>

- `timeout`: `--guild-id`, `--user-id` (facoltativo `--duration-min` o `--until`; ometti entrambi per rimuovere il timeout)
- `kick`: `--guild-id`, `--user-id` (+ `--reason`)
- `ban`: `--guild-id`, `--user-id` (+ `--delete-days`, `--reason`)
  - `timeout` accetta anche `--reason`

<div id="broadcast">
  ### Broadcast
</div>

- `broadcast`
  - Canali: qualsiasi canale configurato; usa `--channel all` per inviare a tutti i provider
  - Obbligatorio: `--targets` (ripetibile)
  - Opzionale: `--message`, `--media`, `--dry-run`

<div id="examples">
  ## Esempi
</div>

Invia una risposta su Discord:

```
openclaw message send --channel discord \
  --target channel:123 --message "hi" --reply-to 456
```

Crea un sondaggio su Discord:

```
openclaw message poll --channel discord \
  --target channel:123 \
  --poll-question "Snack?" \
  --poll-option Pizza --poll-option Sushi \
  --poll-multi --poll-duration-hours 48
```

Invia un messaggio proattivo su Teams:

```
openclaw message send --channel msteams \
  --target conversation:19:abc@thread.tacv2 --message "hi"
```

Crea un sondaggio su Teams:

```
openclaw message poll --channel msteams \
  --target conversation:19:abc@thread.tacv2 \
  --poll-question "Lunch?" \
  --poll-option Pizza --poll-option Sushi
```

Reagisci su Slack:

```
openclaw message react --channel slack \
  --target C123 --message-id 456 --emoji "✅"
```

Reagire in un gruppo su Signal:

```
openclaw message react --channel signal \
  --target signal:group:abc123 --message-id 1737630212345 \
  --emoji "✅" --target-author-uuid 123e4567-e89b-12d3-a456-426614174000
```

Invia pulsanti inline di Telegram:

```
openclaw message send --channel telegram --target @mychat --message "Scegli:" \
  --buttons '[ [{"text":"Yes","callback_data":"cmd:yes"}], [{"text":"No","callback_data":"cmd:no"}] ]'
```
