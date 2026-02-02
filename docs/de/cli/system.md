---
title: System
summary: "CLI-Referenz für `openclaw system` (Systemereignisse, Herzschlag, Präsenz)"
read_when:
  - Du möchtest ein Systemereignis einreihen, ohne einen Cronjob zu erstellen
  - Du musst den Herzschlag aktivieren oder deaktivieren
  - Du möchtest System-Präsenzeinträge einsehen
---

<div id="openclaw-system">
  # `openclaw system`
</div>

Systembezogene Hilfsbefehle für das Gateway: Systemereignisse in die Warteschlange einreihen, Herzschlag steuern
und Präsenz anzeigen.

<div id="common-commands">
  ## Gängige Befehle
</div>

```bash
openclaw system event --text "Check for urgent follow-ups" --mode now
openclaw system heartbeat enable
openclaw system heartbeat last
openclaw system presence
```


<div id="system-event">
  ## `system event`
</div>

Reiht ein Systemereignis in die Warteschlange der **Haupt**sitzung ein. Der nächste Herzschlag
fügt es als `System:`-Zeile im Prompt ein. Verwende `--mode now`, um den Herzschlag
sofort auszulösen; `next-heartbeat` wartet auf den nächsten geplanten Takt.

Flags:

- `--text <text>`: erforderlicher Systemereignistext.
- `--mode <mode>`: `now` oder `next-heartbeat` (Standardwert).
- `--json`: maschinenlesbare Ausgabe.

<div id="system-heartbeat-lastenabledisable">
  ## `system heartbeat last|enable|disable`
</div>

Herzschlag-Steuerung:

- `last`: zeigt das letzte Herzschlag-Ereignis an.
- `enable`: schaltet den Herzschlag wieder ein (verwende dies, wenn er deaktiviert wurde).
- `disable`: pausiert den Herzschlag.

Flags:

- `--json`: maschinenlesbare Ausgabe.

<div id="system-presence">
  ## `system presence`
</div>

Listet die aktuellen Systempräsenz-Einträge auf, die dem Gateway bekannt sind (Knoten,
Instanzen und ähnliche Status-Einträge).

Flags:

- `--json`: maschinenlesbare Ausgabe.

<div id="notes">
  ## Hinweise
</div>

- Erfordert ein laufendes Gateway, das über deine aktuelle Konfiguration (lokal oder remote) erreichbar ist.
- Systemereignisse sind flüchtig und werden über Neustarts hinweg nicht gespeichert.