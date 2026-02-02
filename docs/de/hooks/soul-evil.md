---
title: Soul Evil
summary: "SOUL Evil Hook (SOUL.md durch SOUL_EVIL.md ersetzen)"
read_when:
  - Du möchtest den SOUL Evil Hook aktivieren oder feinabstimmen
  - Du möchtest ein Purge-Zeitfenster oder einen zufallsbasierten Persona-Wechsel
---

<div id="soul-evil-hook">
  # SOUL Evil Hook
</div>

Der SOUL Evil Hook ersetzt den **injizierten** Inhalt von `SOUL.md` während eines Purge-Fensters oder mit einer gewissen Zufallswahrscheinlichkeit durch `SOUL_EVIL.md`. Er verändert **keine** Dateien im Dateisystem.

<div id="how-it-works">
  ## Funktionsweise
</div>

Wenn `agent:bootstrap` ausgeführt wird, kann der Hook den Inhalt von `SOUL.md` im Speicher ersetzen,
bevor der System-Prompt zusammengesetzt wird. Falls `SOUL_EVIL.md` fehlt oder leer ist,
schreibt OpenClaw eine Warnung ins Log und behält das normale `SOUL.md` bei.

Subagent-Ausführungen enthalten **nicht** `SOUL.md` in ihren Bootstrap-Dateien, daher hat dieser Hook
keinen Effekt auf Subagenten.

<div id="enable">
  ## Aktivieren
</div>

```bash
openclaw hooks enable soul-evil
```

Setze dann die Konfiguration:

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "entries": {
        "soul-evil": {
          "enabled": true,
          "file": "SOUL_EVIL.md",
          "chance": 0.1,
          "purge": { "at": "21:00", "duration": "15m" }
        }
      }
    }
  }
}
```

Erstelle `SOUL_EVIL.md` im Wurzelverzeichnis des Agent-Arbeitsbereichs (neben `SOUL.md`).

<div id="options">
  ## Optionen
</div>

* `file` (string): alternativer SOUL-Dateiname (Standard: `SOUL_EVIL.md`)
* `chance` (number 0–1): zufällige Wahrscheinlichkeit pro Lauf, `SOUL_EVIL.md` zu verwenden
* `purge.at` (HH:mm): täglicher Beginn der Bereinigung (24-Stunden-Format)
* `purge.duration` (duration): Länge des Zeitfensters (z. B. `30s`, `10m`, `1h`)

**Priorität:** Das Bereinigungsfenster hat Vorrang vor `chance`.

**Zeitzone:** verwendet `agents.defaults.userTimezone`, falls gesetzt; ansonsten die Host-Zeitzone.

<div id="notes">
  ## Hinweise
</div>

* Es werden keine Dateien auf dem Datenträger geschrieben oder geändert.
* Wenn sich `SOUL.md` nicht in der Bootstrap-Liste befindet, führt der Hook nichts aus.

<div id="see-also">
  ## Siehe auch
</div>

* [Hooks](/de/hooks)