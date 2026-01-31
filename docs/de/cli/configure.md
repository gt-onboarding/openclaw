---
title: Konfigurieren
summary: "CLI-Referenz für `openclaw configure` (interaktive Konfigurationsdialoge)"
read_when:
  - Du möchtest Zugangsdaten, Geräte oder Agent-Standardwerte interaktiv anpassen
---

<div id="openclaw-configure">
  # `openclaw configure`
</div>

Interaktiver Assistent zum Einrichten von Zugangsdaten, Geräten und Agent-Standardeinstellungen.

Hinweis: Der Abschnitt **Model** enthält jetzt eine Mehrfachauswahl für die
`agents.defaults.models` Allowlist (was in `/model` und in der Modellauswahl angezeigt wird).

Tipp: `openclaw config` ohne Unterbefehl öffnet denselben Assistenten. Verwende
`openclaw config get|set|unset` für nicht-interaktive Änderungen.

Verwandte Themen:

* Gateway-Konfigurationsreferenz: [Configuration](/de/gateway/configuration)
* Config-CLI: [Config](/de/cli/config)

Hinweise:

* Die Auswahl, wo das Gateway ausgeführt wird, aktualisiert immer `gateway.mode`. Du kannst „Continue“ auswählen, ohne andere Abschnitte zu ändern, wenn du nur das benötigst.
* Kanalorientierte Dienste (Slack/Discord/Matrix/Microsoft Teams) fragen während der Einrichtung nach Allowlists für Kanäle/Räume. Du kannst Namen oder IDs eingeben; der Assistent löst Namen, wenn möglich, in IDs auf.

<div id="examples">
  ## Beispiele
</div>

```bash
openclaw configure
openclaw configure --section models --section channels
```
