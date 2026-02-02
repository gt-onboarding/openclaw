---
title: Directory
summary: "Riferimento CLI per `openclaw directory` (self, peers, groups)"
read_when:
  - Vuoi cercare gli ID di contatti/gruppi/self per un canale
  - Stai sviluppando un adattatore di directory per un canale
---

<div id="openclaw-directory">
  # `openclaw directory`
</div>

Consultazione della directory per i canali che la supportano (contatti/peer, gruppi e «me»).

<div id="common-flags">
  ## Opzioni comuni
</div>

- `--channel <name>`: ID/alias del canale (obbligatorio quando sono configurati più canali; automatico quando ne è configurato solo uno)
- `--account <id>`: ID dell'account (predefinito: valore predefinito del canale)
- `--json`: stampa l'output in formato JSON

<div id="notes">
  ## Note
</div>

- `directory` serve per aiutarti a trovare gli ID da incollare in altri comandi (in particolare `openclaw message send --target ...`).
- Per molti canali, i risultati sono forniti dalla configurazione (liste di autorizzati / gruppi configurati) invece che da una directory live del provider.
- L'output predefinito è `id` (e talvolta `name`) separati da una tabulazione; usa `--json` per l'automazione via script.

<div id="using-results-with-message-send">
  ## Uso dei risultati con `message send`
</div>

```bash
openclaw directory peers list --channel slack --query "U0"
openclaw message send --channel slack --target user:U012ABCDEF --message "hello"
```


<div id="id-formats-by-channel">
  ## Formati degli ID (per canale)
</div>

- WhatsApp: `+15551234567` (DM), `1234567890-1234567890@g.us` (gruppo)
- Telegram: `@username` o ID chat numerico; i gruppi hanno ID numerici
- Slack: `user:U…` e `channel:C…`
- Discord: `user:<id>` e `channel:<id>`
- Matrix (plugin): `user:@user:server`, `room:!roomId:server` o `#alias:server`
- Microsoft Teams (plugin): `user:<id>` e `conversation:<id>`
- Zalo (plugin): ID utente (Bot API)
- Zalo Personal / `zalouser` (plugin): ID thread (DM/gruppo) da `zca` (`me`, `friend list`, `group list`)

<div id="self-me">
  ## Me (“io”)
</div>

```bash
openclaw directory self --channel zalouser
```


<div id="peers-contactsusers">
  ## Peer (contatti/utenti)
</div>

```bash
openclaw directory peers list --channel zalouser
openclaw directory peers list --channel zalouser --query "name"
openclaw directory peers list --channel zalouser --limit 50
```


<div id="groups">
  ## Gruppi
</div>

```bash
openclaw directory groups list --channel zalouser
openclaw directory groups list --channel zalouser --query "work"
openclaw directory groups members --channel zalouser --group-id <id>
```
