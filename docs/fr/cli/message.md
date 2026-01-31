---
title: Message
summary: "Référence CLI pour `openclaw message` (envoi + actions de canal)"
read_when:
  - Ajout ou modification des actions CLI liées aux messages
  - Modification du comportement des canaux de sortie
---

<div id="openclaw-message">
  # `openclaw message`
</div>

Commande sortante unique pour envoyer des messages et des actions sur les canaux
(Discord/Google Chat/Slack/Mattermost (plugin)/Telegram/WhatsApp/Signal/iMessage/MS Teams).

<div id="usage">
  ## Utilisation
</div>

```
openclaw message <subcommand> [flags]
```

Sélection du canal :

* `--channel` est requis si plus d’un canal est configuré.
* S’il n’y a qu’un seul canal configuré, il devient la valeur par défaut.
* Valeurs : `whatsapp|telegram|discord|googlechat|slack|mattermost|signal|imessage|msteams` (Mattermost nécessite un plugin)

Formats cibles (`--target`) :

* WhatsApp : E.164 ou JID de groupe
* Telegram : identifiant de discussion ou `@username`
* Discord : `channel:<id>` ou `user:<id>` (ou mention `<@id>` ; les identifiants numériques bruts sont traités comme des canaux)
* Google Chat : `spaces/<spaceId>` ou `users/<userId>`
* Slack : `channel:<id>` ou `user:<id>` (l’identifiant brut du canal est accepté)
* Mattermost (plugin) : `channel:<id>`, `user:<id>` ou `@username` (les identifiants seuls sont traités comme des canaux)
* Signal : `+E.164`, `group:<id>`, `signal:+E.164`, `signal:group:<id>` ou `username:<name>`/`u:<name>`
* iMessage : handle, `chat_id:<id>`, `chat_guid:<guid>` ou `chat_identifier:<id>`
* MS Teams : identifiant de conversation (`19:...@thread.tacv2`) ou `conversation:<id>` ou `user:<aad-object-id>`

Recherche par nom :

* Pour les fournisseurs pris en charge (Discord/Slack/etc), les noms de canaux comme `Help` ou `#help` sont résolus via le cache d’annuaire.
* En cas d’absence dans le cache, OpenClaw tentera une recherche d’annuaire en temps réel lorsque le fournisseur le prend en charge.


<div id="common-flags">
  ## Options courantes
</div>

- `--channel <name>`
- `--account <id>`
- `--target <dest>` (canal ou utilisateur de destination pour send/poll/read/etc)
- `--targets <name>` (peut être répété ; diffusion uniquement)
- `--json`
- `--dry-run`
- `--verbose`

<div id="actions">
  ## Actions
</div>

<div id="core">
  ### Noyau
</div>

- `send`
  - Canaux : WhatsApp/Telegram/Discord/Google Chat/Slack/Mattermost (plugin)/Signal/iMessage/MS Teams
  - Obligatoire : `--target`, plus `--message` ou `--media`
  - Facultatif : `--media`, `--reply-to`, `--thread-id`, `--gif-playback`
  - Telegram uniquement : `--buttons` (requiert `channels.telegram.capabilities.inlineButtons` pour l’autoriser)
  - Telegram uniquement : `--thread-id` (ID de sujet de forum)
  - Slack uniquement : `--thread-id` (horodatage du fil ; `--reply-to` utilise le même champ)
  - WhatsApp uniquement : `--gif-playback`

- `poll`
  - Canaux : WhatsApp/Discord/MS Teams
  - Obligatoire : `--target`, `--poll-question`, `--poll-option` (répétable)
  - Facultatif : `--poll-multi`
  - Discord uniquement : `--poll-duration-hours`, `--message`

- `react`
  - Canaux : Discord/Google Chat/Slack/Telegram/WhatsApp/Signal
  - Obligatoire : `--message-id`, `--target`
  - Facultatif : `--emoji`, `--remove`, `--participant`, `--from-me`, `--target-author`, `--target-author-uuid`
  - Remarque : `--remove` requiert `--emoji` (omettre `--emoji` pour effacer vos propres réactions lorsque c’est pris en charge ; voir /tools/reactions)
  - WhatsApp uniquement : `--participant`, `--from-me`
  - Réactions de groupe Signal : `--target-author` ou `--target-author-uuid` obligatoire

- `reactions`
  - Canaux : Discord/Google Chat/Slack
  - Obligatoire : `--message-id`, `--target`
  - Facultatif : `--limit`

- `read`
  - Canaux : Discord/Slack
  - Obligatoire : `--target`
  - Facultatif : `--limit`, `--before`, `--after`
  - Discord uniquement : `--around`

- `edit`
  - Canaux : Discord/Slack
  - Obligatoire : `--message-id`, `--message`, `--target`

- `delete`
  - Canaux : Discord/Slack/Telegram
  - Obligatoire : `--message-id`, `--target`

- `pin` / `unpin`
  - Canaux : Discord/Slack
  - Obligatoire : `--message-id`, `--target`

- `pins` (liste)
  - Canaux : Discord/Slack
  - Obligatoire : `--target`

- `permissions`
  - Canaux : Discord
  - Obligatoire : `--target`

- `search`
  - Canaux : Discord
  - Obligatoire : `--guild-id`, `--query`
  - Facultatif : `--channel-id`, `--channel-ids` (répétable), `--author-id`, `--author-ids` (répétable), `--limit`

<div id="threads">
  ### Fils de discussion
</div>

- `thread create`
  - Canaux : Discord
  - Obligatoire : `--thread-name`, `--target` (ID du canal)
  - Facultatif : `--message-id`, `--auto-archive-min`

- `thread list`
  - Canaux : Discord
  - Obligatoire : `--guild-id`
  - Facultatif : `--channel-id`, `--include-archived`, `--before`, `--limit`

- `thread reply`
  - Canaux : Discord
  - Obligatoire : `--target` (ID du fil), `--message`
  - Facultatif : `--media`, `--reply-to`

<div id="emojis">
  ### Émojis
</div>

- `emoji list`
  - Discord : `--guild-id`
  - Slack : aucune option supplémentaire

- `emoji upload`
  - Canaux : Discord
  - Options obligatoires : `--guild-id`, `--emoji-name`, `--media`
  - Options facultatives : `--role-ids` (répéter)

<div id="stickers">
  ### Autocollants
</div>

- `sticker send`
  - Canaux : Discord
  - Requis : `--target`, `--sticker-id` (peut être répété)
  - Optionnel : `--message`

- `sticker upload`
  - Canaux : Discord
  - Requis : `--guild-id`, `--sticker-name`, `--sticker-desc`, `--sticker-tags`, `--media`

<div id="roles-channels-members-voice">
  ### Rôles / Canaux / Membres / Vocal
</div>

- `role info` (Discord) : `--guild-id`
- `role add` / `role remove` (Discord) : `--guild-id`, `--user-id`, `--role-id`
- `channel info` (Discord) : `--target`
- `channel list` (Discord) : `--guild-id`
- `member info` (Discord/Slack) : `--user-id` (+ `--guild-id` pour Discord)
- `voice status` (Discord) : `--guild-id`, `--user-id`

<div id="events">
  ### Événements
</div>

- `event list` (Discord) : `--guild-id`
- `event create` (Discord) : `--guild-id`, `--event-name`, `--start-time`
  - Facultatif : `--end-time`, `--desc`, `--channel-id`, `--location`, `--event-type`

<div id="moderation-discord">
  ### Modération (Discord)
</div>

- `timeout` : `--guild-id`, `--user-id` (optionnel : `--duration-min` ou `--until` ; omettez les deux pour supprimer le délai)
- `kick` : `--guild-id`, `--user-id` (+ `--reason`)
- `ban` : `--guild-id`, `--user-id` (+ `--delete-days`, `--reason`)
  - `timeout` accepte également `--reason`

<div id="broadcast">
  ### Diffusion
</div>

- `broadcast`
  - Canaux : tout canal configuré ; utilisez `--channel all` pour cibler tous les fournisseurs
  - Obligatoire : `--targets` (répétable)
  - Facultatif : `--message`, `--media`, `--dry-run`

<div id="examples">
  ## Exemples
</div>

Envoyer une réponse sur Discord :

```
openclaw message send --channel discord \
  --target channel:123 --message "hi" --reply-to 456
```

Créer un sondage sur Discord :

```
openclaw message poll --channel discord \
  --target channel:123 \
  --poll-question "Snack?" \
  --poll-option Pizza --poll-option Sushi \
  --poll-multi --poll-duration-hours 48
```

Envoyer un message proactif sur Teams :

```
openclaw message send --channel msteams \
  --target conversation:19:abc@thread.tacv2 --message "hi"
```

Créer un sondage dans Teams :

```
openclaw message poll --channel msteams \
  --target conversation:19:abc@thread.tacv2 \
  --poll-question "Lunch?" \
  --poll-option Pizza --poll-option Sushi
```

Réagir sur Slack :

```
openclaw message react --channel slack \
  --target C123 --message-id 456 --emoji "✅"
```

Réagir dans un groupe Signal :

```
openclaw message react --channel signal \
  --target signal:group:abc123 --message-id 1737630212345 \
  --emoji "✅" --target-author-uuid 123e4567-e89b-12d3-a456-426614174000
```

Envoyer des boutons inline Telegram :

```
openclaw message send --channel telegram --target @mychat --message "Choisir :" \
  --buttons '[ [{"text":"Yes","callback_data":"cmd:yes"}], [{"text":"No","callback_data":"cmd:no"}] ]'
```
