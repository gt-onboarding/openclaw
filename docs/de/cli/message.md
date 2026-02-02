---
title: Nachricht
summary: "CLI-Referenz für `openclaw message` (Sende- und Kanalaktionen)"
read_when:
  - Hinzufügen oder Ändern von Nachrichten-CLI-Aktionen
  - Ändern des Verhaltens ausgehender Kanäle
---

<div id="openclaw-message">
  # `openclaw message`
</div>

Ein einzelner ausgehender Befehl zum Senden von Nachrichten und Kanalaktionen
(Discord/Google Chat/Slack/Mattermost (Plugin)/Telegram/WhatsApp/Signal/iMessage/MS Teams).

<div id="usage">
  ## Verwendung
</div>

```
openclaw message <subcommand> [flags]
```

Kanalauswahl:

* `--channel` ist erforderlich, wenn mehr als ein Kanal konfiguriert ist.
* Wenn genau ein Kanal konfiguriert ist, wird dieser zum Standard.
* Werte: `whatsapp|telegram|discord|googlechat|slack|mattermost|signal|imessage|msteams` (Mattermost erfordert Plugin)

Zielformate (`--target`):

* WhatsApp: E.164 oder Gruppen-JID
* Telegram: Chat-ID oder `@username`
* Discord: `channel:<id>` oder `user:<id>` (oder `<@id>`-Erwähnung; rohe numerische IDs werden als Kanäle behandelt)
* Google Chat: `spaces/<spaceId>` oder `users/<userId>`
* Slack: `channel:<id>` oder `user:<id>` (rohe Kanal-ID wird akzeptiert)
* Mattermost (Plugin): `channel:<id>`, `user:<id>` oder `@username` (bloße IDs werden als Kanäle behandelt)
* Signal: `+E.164`, `group:<id>`, `signal:+E.164`, `signal:group:<id>` oder `username:<name>`/`u:<name>`
* iMessage: Handle, `chat_id:<id>`, `chat_guid:<guid>` oder `chat_identifier:<id>`
* MS Teams: Konversations-ID (`19:...@thread.tacv2`) oder `conversation:<id>` oder `user:<aad-object-id>`

Namensauflösung:

* Für unterstützte Anbieter (Discord/Slack/etc.) werden Kanalnamen wie `Help` oder `#help` über den Verzeichnis-Cache aufgelöst.
* Wenn der Cache keinen Treffer liefert, versucht OpenClaw eine Live-Verzeichnisabfrage, sofern der Anbieter dies unterstützt.


<div id="common-flags">
  ## Allgemeine Flags
</div>

- `--channel <name>`
- `--account <id>`
- `--target <dest>` (Zielkanal oder Benutzer für send/poll/read/usw.)
- `--targets <name>` (wiederholbar; nur für Broadcast)
- `--json`
- `--dry-run`
- `--verbose`

<div id="actions">
  ## Aktionen
</div>

<div id="core">
  ### Kern
</div>

- `send`
  - Kanäle: WhatsApp/Telegram/Discord/Google Chat/Slack/Mattermost (Plugin)/Signal/iMessage/MS Teams
  - Erforderlich: `--target` sowie `--message` oder `--media`
  - Optional: `--media`, `--reply-to`, `--thread-id`, `--gif-playback`
  - Nur Telegram: `--buttons` (erfordert `channels.telegram.capabilities.inlineButtons`, um dies zu erlauben)
  - Nur Telegram: `--thread-id` (Forum-Thread-ID)
  - Nur Slack: `--thread-id` (Thread-Zeitstempel; `--reply-to` verwendet dasselbe Feld)
  - Nur WhatsApp: `--gif-playback`

- `poll`
  - Kanäle: WhatsApp/Discord/MS Teams
  - Erforderlich: `--target`, `--poll-question`, `--poll-option` (wiederholt)
  - Optional: `--poll-multi`
  - Nur Discord: `--poll-duration-hours`, `--message`

- `react`
  - Kanäle: Discord/Google Chat/Slack/Telegram/WhatsApp/Signal
  - Erforderlich: `--message-id`, `--target`
  - Optional: `--emoji`, `--remove`, `--participant`, `--from-me`, `--target-author`, `--target-author-uuid`
  - Hinweis: `--remove` erfordert `--emoji` (lassen Sie `--emoji` weg, um eigene Reaktionen zu löschen, sofern unterstützt; siehe /tools/reactions)
  - Nur WhatsApp: `--participant`, `--from-me`
  - Signal-Gruppenreaktionen: `--target-author` oder `--target-author-uuid` erforderlich

- `reactions`
  - Kanäle: Discord/Google Chat/Slack
  - Erforderlich: `--message-id`, `--target`
  - Optional: `--limit`

- `read`
  - Kanäle: Discord/Slack
  - Erforderlich: `--target`
  - Optional: `--limit`, `--before`, `--after`
  - Nur Discord: `--around`

- `edit`
  - Kanäle: Discord/Slack
  - Erforderlich: `--message-id`, `--message`, `--target`

- `delete`
  - Kanäle: Discord/Slack/Telegram
  - Erforderlich: `--message-id`, `--target`

- `pin` / `unpin`
  - Kanäle: Discord/Slack
  - Erforderlich: `--message-id`, `--target`

- `pins` (Liste)
  - Kanäle: Discord/Slack
  - Erforderlich: `--target`

- `permissions`
  - Kanäle: Discord
  - Erforderlich: `--target`

- `search`
  - Kanäle: Discord
  - Erforderlich: `--guild-id`, `--query`
  - Optional: `--channel-id`, `--channel-ids` (wiederholt), `--author-id`, `--author-ids` (wiederholt), `--limit`

<div id="threads">
  ### Threads
</div>

- `thread create`
  - Kanäle: Discord
  - Erforderlich: `--thread-name`, `--target` (Kanal-ID)
  - Optional: `--message-id`, `--auto-archive-min`

- `thread list`
  - Kanäle: Discord
  - Erforderlich: `--guild-id`
  - Optional: `--channel-id`, `--include-archived`, `--before`, `--limit`

- `thread reply`
  - Kanäle: Discord
  - Erforderlich: `--target` (Thread-ID), `--message`
  - Optional: `--media`, `--reply-to`

<div id="emojis">
  ### Emojis
</div>

- `emoji list`
  - Discord: `--guild-id`
  - Slack: keine zusätzlichen Flags

- `emoji upload`
  - Kanäle: Discord
  - Erforderlich: `--guild-id`, `--emoji-name`, `--media`
  - Optional: `--role-ids` (wiederholbar)

<div id="stickers">
  ### Sticker
</div>

- `sticker send`
  - Kanäle: Discord
  - Erforderlich: `--target`, `--sticker-id` (mehrfach wiederholbar)
  - Optional: `--message`

- `sticker upload`
  - Kanäle: Discord
  - Erforderlich: `--guild-id`, `--sticker-name`, `--sticker-desc`, `--sticker-tags`, `--media`

<div id="roles-channels-members-voice">
  ### Rollen / Kanäle / Mitglieder / Voice
</div>

- `role info` (Discord): `--guild-id`
- `role add` / `role remove` (Discord): `--guild-id`, `--user-id`, `--role-id`
- `channel info` (Discord): `--target`
- `channel list` (Discord): `--guild-id`
- `member info` (Discord/Slack): `--user-id` (+ `--guild-id` für Discord)
- `voice status` (Discord): `--guild-id`, `--user-id`

<div id="events">
  ### Events
</div>

- `event list` (Discord): `--guild-id`
- `event create` (Discord): `--guild-id`, `--event-name`, `--start-time`
  - Optional: `--end-time`, `--desc`, `--channel-id`, `--location`, `--event-type`

<div id="moderation-discord">
  ### Moderation (Discord)
</div>

- `timeout`: `--guild-id`, `--user-id` (optional `--duration-min` oder `--until`; beide weglassen, um das Timeout aufzuheben)
- `kick`: `--guild-id`, `--user-id` (+ `--reason`)
- `ban`: `--guild-id`, `--user-id` (+ `--delete-days`, `--reason`)
  - `timeout` unterstützt ebenfalls `--reason`

<div id="broadcast">
  ### Broadcast
</div>

- `broadcast`
  - Channels: beliebiger konfigurierter Kanal; Verwende `--channel all`, um alle anbieter anzusprechen
  - Erforderlich: `--targets` (wiederholbar)
  - Optional: `--message`, `--media`, `--dry-run`

<div id="examples">
  ## Beispiele
</div>

Eine Discord-Antwort senden:

```
openclaw message send --channel discord \
  --target channel:123 --message "hi" --reply-to 456
```

Erstelle eine Umfrage auf Discord:

```
openclaw message poll --channel discord \
  --target channel:123 \
  --poll-question "Snack?" \
  --poll-option Pizza --poll-option Sushi \
  --poll-multi --poll-duration-hours 48
```

Senden Sie eine proaktive Nachricht in Teams:

```
openclaw message send --channel msteams \
  --target conversation:19:abc@thread.tacv2 --message "hi"
```

Erstellen Sie eine Teams-Umfrage:

```
openclaw message poll --channel msteams \
  --target conversation:19:abc@thread.tacv2 \
  --poll-question "Lunch?" \
  --poll-option Pizza --poll-option Sushi
```

In Slack reagieren:

```
openclaw message react --channel slack \
  --target C123 --message-id 456 --emoji "✅"
```

In einer Signal-Gruppe reagieren:

```
openclaw message react --channel signal \
  --target signal:group:abc123 --message-id 1737630212345 \
  --emoji "✅" --target-author-uuid 123e4567-e89b-12d3-a456-426614174000
```

Telegram-Inline-Buttons senden:

```
openclaw message send --channel telegram --target @mychat --message "Wähle:" \
  --buttons '[ [{"text":"Yes","callback_data":"cmd:yes"}], [{"text":"No","callback_data":"cmd:no"}] ]'
```
