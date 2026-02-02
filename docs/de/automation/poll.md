---
title: "Umfrage"
summary: "Versand von Umfragen über Gateway und CLI"
read_when:
  - Hinzufügen oder Ändern der Unterstützung für Umfragen
  - Debugging des Versands von Umfragen über die CLI oder das Gateway
---

<div id="polls">
  # Umfragen
</div>

<div id="supported-channels">
  ## Unterstützte Kanäle
</div>

- WhatsApp (Web-Kanal)
- Discord
- MS Teams (Adaptive Cards)

<div id="cli">
  ## CLI
</div>

```bash
# WhatsApp
openclaw message poll --target +15555550123 \
  --poll-question "Lunch today?" --poll-option "Yes" --poll-option "No" --poll-option "Maybe"
openclaw message poll --target 123456789@g.us \
  --poll-question "Meeting time?" --poll-option "10am" --poll-option "2pm" --poll-option "4pm" --poll-multi

# Discord
openclaw message poll --channel discord --target channel:123456789 \
  --poll-question "Snack?" --poll-option "Pizza" --poll-option "Sushi"
openclaw message poll --channel discord --target channel:123456789 \
  --poll-question "Plan?" --poll-option "A" --poll-option "B" --poll-duration-hours 48

# MS Teams
openclaw message poll --channel msteams --target conversation:19:abc@thread.tacv2 \
  --poll-question "Lunch?" --poll-option "Pizza" --poll-option "Sushi"
```

Optionen:

* `--channel`: `whatsapp` (Standard), `discord` oder `msteams`
* `--poll-multi`: ermöglicht die Auswahl mehrerer Optionen
* `--poll-duration-hours`: nur für Discord (Standardwert ist 24, falls nicht gesetzt)


<div id="gateway-rpc">
  ## Gateway RPC
</div>

Methode: `poll`

Parameter:

- `to` (string, erforderlich)
- `question` (string, erforderlich)
- `options` (string[], erforderlich)
- `maxSelections` (number, optional)
- `durationHours` (number, optional)
- `channel` (string, optional, Standardwert: `whatsapp`)
- `idempotencyKey` (string, erforderlich)

<div id="channel-differences">
  ## Kanalunterschiede
</div>

- WhatsApp: 2–12 Optionen, `maxSelections` muss innerhalb der Anzahl der Optionen liegen, `durationHours` wird ignoriert.
- Discord: 2–10 Optionen, `durationHours` wird auf 1–768 Stunden begrenzt (Standard ist 24). `maxSelections > 1` aktiviert Mehrfachauswahl; Discord unterstützt keine feste Vorgabe für die Anzahl der Auswahlen.
- MS Teams: Adaptive-Card-Umfragen (von OpenClaw verwaltet). Keine native Umfrage-API; `durationHours` wird ignoriert.

<div id="agent-tool-message">
  ## Agent-Tool (Nachricht)
</div>

Verwende das `message`-Tool mit der Aktion `poll` (`to`, `pollQuestion`, `pollOption`, optional `pollMulti`, `pollDurationHours`, `channel`).

Hinweis: Discord hat keinen Modus „genau N auswählen“; `pollMulti` entspricht einer Mehrfachauswahl.
Teams-Umfragen werden als Adaptive Cards gerendert und erfordern, dass das Gateway online bleiben muss,
um Stimmen in `~/.openclaw/msteams-polls.json` zu protokollieren.