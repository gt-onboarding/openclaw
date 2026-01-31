---
title: Agent-Senden
summary: "Direkte `openclaw agent` CLI-Ausführungen (mit optionalem Versand)"
read_when:
  - Hinzufügen oder Ändern des Agent-CLI-Einstiegspunkts
---

<div id="openclaw-agent-direct-agent-runs">
  # `openclaw agent` (direkte Agent-Ausführungen)
</div>

`openclaw agent` führt einen einzelnen Agent-Schritt aus, ohne dass eine eingehende Chat-Nachricht erforderlich ist.
Standardmäßig läuft der Befehl **über das Gateway**; füge `--local` hinzu, um die eingebettete Runtime
auf der aktuellen Maschine zu erzwingen.

<div id="behavior">
  ## Verhalten
</div>

- Erforderlich: `--message <text>`
- Auswahl der Sitzung:
  - `--to <dest>` leitet den Sitzungsschlüssel ab (Gruppen-/Channel-Ziele bleiben isoliert; Direkt-Chats werden auf `main` konsolidiert), **oder**
  - `--session-id <id>` verwendet eine bestehende Sitzung per ID weiter, **oder**
  - `--agent <id>` adressiert einen konfigurierten agent direkt (verwendet den `main`-Sitzungsschlüssel dieses agents)
- Führt dieselbe eingebettete Agent-Laufzeitumgebung aus wie normale eingehende Antworten.
- Thinking-/verbose-Flags bleiben im Sitzungsspeicher erhalten.
- Ausgabe:
  - Standard: gibt Antworttext aus (plus `MEDIA:<url>`-Zeilen)
  - `--json`: gibt strukturierte Payload + Metadaten aus
- Optionale Zustellung zurück an einen Channel mit `--deliver` + `--channel` (Zielformate entsprechen `openclaw message --target`).
- Verwende `--reply-channel`/`--reply-to`/`--reply-account`, um die Zustellung zu übersteuern, ohne die Sitzung zu ändern.

Wenn das Gateway nicht erreichbar ist, fällt die CLI auf die eingebettete lokale Ausführung zurück.

<div id="examples">
  ## Beispiele
</div>

```bash
openclaw agent --to +15555550123 --message "status update"
openclaw agent --agent ops --message "Summarize logs"
openclaw agent --session-id 1234 --message "Summarize inbox" --thinking medium
openclaw agent --to +15555550123 --message "Trace logs" --verbose on --json
openclaw agent --to +15555550123 --message "Summon reply" --deliver
openclaw agent --agent ops --message "Generate report" --deliver --reply-channel slack --reply-to "#reports"
```


<div id="flags">
  ## Flags
</div>

- `--local`: lokal ausführen (erfordert API-Schlüssel deines Modellanbieters in deiner Shell)
- `--deliver`: Antwort an den gewählten Kanal senden
- `--channel`: Zustellkanal (`whatsapp|telegram|discord|googlechat|slack|signal|imessage`, Standard: `whatsapp`)
- `--reply-to`: Zustellziel überschreiben
- `--reply-channel`: Zustellkanal überschreiben
- `--reply-account`: Zustellkonto-ID überschreiben
- `--thinking <off|minimal|low|medium|high|xhigh>`: Denkstufe speichern (nur GPT-5.2- und Codex-Modelle)
- `--verbose <on|full|off>`: Verbositätsstufe speichern
- `--timeout <seconds>`: agent-Timeout überschreiben
- `--json`: strukturiertes JSON ausgeben