---
title: Verzeichnis
summary: "CLI-Referenz für `openclaw directory` (self, peers, groups)"
read_when:
  - Du möchtest IDs von Kontakten/Gruppen/dir selbst für einen Kanal nachschlagen
  - Du entwickelst einen Channel-Verzeichnis-Adapter
---

<div id="openclaw-directory">
  # `openclaw directory`
</div>

Verzeichnissuchen für Kanäle, die dies unterstützen (Kontakte/Peers, Gruppen und „me“).

<div id="common-flags">
  ## Gängige Flags
</div>

- `--channel <name>`: Channel-ID/Alias (erforderlich, wenn mehrere Channels konfiguriert sind; wird automatisch gesetzt, wenn nur einer konfiguriert ist)
- `--account <id>`: Account-ID (Standard: Vorgabe des Channels)
- `--json`: Ausgabe als JSON

<div id="notes">
  ## Hinweise
</div>

- `directory` soll dir helfen, IDs zu finden, die du in andere Befehle einfügen kannst (insbesondere `openclaw message send --target ...`).
- Für viele Kanäle sind die Ergebnisse konfigurationsbasiert (Allowlists / konfigurierte Gruppen) statt eines Live-Verzeichnisses eines Anbieters.
- Die Standardausgabe ist `id` (und manchmal `name`), jeweils durch ein Tabulatorzeichen getrennt; verwende `--json` für Skripte.

<div id="using-results-with-message-send">
  ## Nutzung der Ergebnisse mit `message send`
</div>

```bash
openclaw directory peers list --channel slack --query "U0"
openclaw message send --channel slack --target user:U012ABCDEF --message "hello"
```


<div id="id-formats-by-channel">
  ## ID-Formate (je Channel)
</div>

- WhatsApp: `+15551234567` (DM), `1234567890-1234567890@g.us` (Gruppe)
- Telegram: `@username` oder numerische Chat-ID; Gruppen verwenden numerische IDs
- Slack: `user:U…` und `channel:C…`
- Discord: `user:<id>` und `channel:<id>`
- Matrix (Plugin): `user:@user:server`, `room:!roomId:server` oder `#alias:server`
- Microsoft Teams (Plugin): `user:<id>` und `conversation:<id>`
- Zalo (Plugin): Benutzer-ID (Bot API)
- Zalo Personal / `zalouser` (Plugin): Thread-ID (DM/Gruppe) von `zca` (`me`, `friend list`, `group list`)

<div id="self-me">
  ## Ich („mich“)
</div>

```bash
openclaw directory self --channel zalouser
```


<div id="peers-contactsusers">
  ## Peers (Kontakte/Nutzer)
</div>

```bash
openclaw directory peers list --channel zalouser
openclaw directory peers list --channel zalouser --query "name"
openclaw directory peers list --channel zalouser --limit 50
```


<div id="groups">
  ## Gruppen
</div>

```bash
openclaw directory groups list --channel zalouser
openclaw directory groups list --channel zalouser --query "work"
openclaw directory groups members --channel zalouser --group-id <id>
```
