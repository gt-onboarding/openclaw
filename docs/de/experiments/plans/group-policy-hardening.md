---
title: Härtung von Gruppenrichtlinien
summary: "Telegram-Allowlist-Härtung: Präfix- und Leerraum-Normalisierung"
read_when:
  - Beim Überprüfen früherer Änderungen an der Telegram-Allowlist
---

<div id="telegram-allowlist-hardening">
  # Härtung der Telegram-Allowlist
</div>

**Datum**: 2026-01-05\
**Status**: Abgeschlossen\
**PR**: #216

<div id="summary">
  ## Zusammenfassung
</div>

Telegram-Allowlists akzeptieren jetzt die Präfixe `telegram:` und `tg:` unabhängig von der Groß-/Kleinschreibung und tolerieren versehentlich eingefügte Leerzeichen. Dadurch werden eingehende Allowlist-Prüfungen an die Normalisierung ausgehender Sendevorgänge angeglichen.

<div id="what-changed">
  ## Was sich geändert hat
</div>

* Präfixe `telegram:` und `tg:` werden gleich behandelt (Groß- und Kleinschreibung werden nicht beachtet).
* Allowlist-Einträge werden von Leerzeichen bereinigt; leere Einträge werden ignoriert.

<div id="examples">
  ## Beispiele
</div>

Alle folgenden Schreibweisen werden für dieselbe ID akzeptiert:

* `telegram:123456`
* `TG:123456`
* `tg:123456`

<div id="why-it-matters">
  ## Warum das wichtig ist
</div>

Copy/Paste aus Logs oder Chat-IDs enthält oft Präfixe und Leerzeichen. Durch Normalisierung werden
falsch-negative Treffer vermieden, wenn entschieden wird, ob in DMs oder Gruppen geantwortet werden soll.

<div id="related-docs">
  ## Zugehörige Dokumentation
</div>

* [Gruppenchats](/de/concepts/groups)
* [Telegram-Anbieter](/de/channels/telegram)