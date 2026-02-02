---
title: AGENTS.default
summary: "Standardanweisungen und Fähigkeitenübersicht des OpenClaw-Agents für das persönliche Assistenten-Setup"
read_when:
  - Beim Start einer neuen OpenClaw-Agent-Sitzung
  - Beim Aktivieren oder Prüfen von Standardfähigkeiten
---

<div id="agentsmd-openclaw-personal-assistant-default">
  # AGENTS.md — OpenClaw Persönlicher Assistent (Standard)
</div>

<div id="first-run-recommended">
  ## Erster Start (empfohlen)
</div>

OpenClaw verwendet ein eigenes Arbeitsbereichsverzeichnis für den Agent. Standard: `~/.openclaw/workspace` (konfigurierbar über `agents.defaults.workspace`).

1. Erstelle den Arbeitsbereich (falls er noch nicht existiert):

```bash
mkdir -p ~/.openclaw/workspace
```

2. Kopiere die Standardvorlagen für den Arbeitsbereich in den Arbeitsbereich:

```bash
cp docs/reference/templates/AGENTS.md ~/.openclaw/workspace/AGENTS.md
cp docs/reference/templates/SOUL.md ~/.openclaw/workspace/SOUL.md
cp docs/reference/templates/TOOLS.md ~/.openclaw/workspace/TOOLS.md
```

3. Optional: Wenn du die Skill-Liste des persönlichen Assistenten verwenden möchtest, ersetze AGENTS.md durch diese Datei:

```bash
cp docs/reference/AGENTS.default.md ~/.openclaw/workspace/AGENTS.md
```

4. Optional: Wähle einen anderen Arbeitsbereich, indem du `agents.defaults.workspace` setzt (unterstützt `~`):

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } }
}
```


<div id="safety-defaults">
  ## Sicherheits-Defaults
</div>

- Gib keine Verzeichnisse oder Secrets im Chat aus.
- Führe keine destruktiven Befehle aus, es sei denn, du wirst ausdrücklich dazu aufgefordert.
- Sende keine teilweisen bzw. gestreamten Antworten an externe Messaging-Plattformen (nur endgültige Antworten).

<div id="session-start-required">
  ## Sitzungsstart (erforderlich)
</div>

- Lies `SOUL.md`, `USER.md`, `memory.md` und heute und gestern in `memory/`.
- Mach das, bevor du antwortest.

<div id="soul-required">
  ## Soul (erforderlich)
</div>

- `SOUL.md` definiert Identität, Tonfall und Grenzen. Halte sie aktuell.
- Wenn du `SOUL.md` änderst, informiere den:die Nutzer:in.
- Du bist in jeder Sitzung eine frische Instanz; die Kontinuität steckt in diesen Dateien.

<div id="shared-spaces-recommended">
  ## Gemeinsame Bereiche (empfohlen)
</div>

- Du bist nicht die Stimme des Nutzers – sei in Gruppenchats oder öffentlichen Kanälen vorsichtig.
- Gib keine privaten Daten, Kontaktdaten oder internen Notizen weiter.

<div id="memory-system-recommended">
  ## Speichersystem (empfohlen)
</div>

- Tagesprotokoll: `memory/YYYY-MM-DD.md` (lege `memory/` bei Bedarf an).
- Langzeitgedächtnis: `memory.md` für dauerhafte Fakten, Präferenzen und Entscheidungen.
- Beim Start einer Sitzung: Datei für heute + gestern + `memory.md` lesen, falls vorhanden.
- Halte fest: Entscheidungen, Präferenzen, Einschränkungen, offene Aufgaben/Loops.
- Vermeide Geheimnisse, außer wenn ausdrücklich angefordert.

<div id="tools-skills">
  ## Tools & Fähigkeiten
</div>

- Tools befinden sich in den Fähigkeiten; folge bei Bedarf der jeweiligen `SKILL.md`.
- Halte umgebungsspezifische Notizen in `TOOLS.md` fest (Hinweise zu Fähigkeiten).

<div id="backup-tip-recommended">
  ## Backup-Tipp (empfohlen)
</div>

Wenn du diesen Arbeitsbereich als Clawds „Gedächtnis“ behandelst, richte ihn als Git-Repository ein (idealerweise privat), damit `AGENTS.md` und deine Speicherdateien gesichert werden.

```bash
cd ~/.openclaw/workspace
git init
git add AGENTS.md
git commit -m "Add Clawd workspace"
# Optional: privates Remote hinzufügen und pushen
```


<div id="what-openclaw-does">
  ## Was OpenClaw macht
</div>

- Führt WhatsApp-Gateway + Pi-Coding-Agent aus, damit der Assistent Chats lesen und schreiben, Kontext abrufen und Fähigkeiten über den Host-Mac ausführen kann.
- Die macOS-App verwaltet Berechtigungen (Bildschirmaufzeichnung, Benachrichtigungen, Mikrofon) und stellt die `openclaw` CLI über ihr mitgeliefertes Binary bereit.
- Direktchats werden standardmäßig in der `main`-Sitzung des agents zusammengeführt; Gruppen bleiben isoliert als `agent:<agentId>:<channel>:group:<id>` (Räume/Channels: `agent:<agentId>:<channel>:channel:<id>`); Heartbeats halten Hintergrundaufgaben aktiv.

<div id="core-skills-enable-in-settings-skills">
  ## Zentrale Fähigkeiten (in Einstellungen → Fähigkeiten aktivieren)
</div>

- **mcporter** — Toolserver-Laufzeit/CLI zur Verwaltung externer Skill-Backends.
- **Peekaboo** — Schnelle macOS-Screenshots mit optionaler KI‑Bildanalyse.
- **camsnap** — Erfassen von Frames, Clips oder Bewegungswarnungen von RTSP/ONVIF-Sicherheitskameras.
- **oracle** — OpenAI-fähige Agent-CLI mit Sitzungswiedergabe und Browsersteuerung.
- **eightctl** — Steuere deinen Schlaf direkt vom Terminal aus.
- **imsg** — senden, read, streamen von iMessage & SMS.
- **wacli** — WhatsApp-CLI: synchronisieren, suchen, senden.
- **discord** — Discord-Aktionen: Reaktionen, Sticker, Umfragen. Verwende `user:<id>` oder `channel:<id>` als Ziele (reine numerische IDs sind uneindeutig).
- **gog** — Google-Suite-CLI: Gmail, Kalender, Drive, Kontakte.
- **spotify-player** — Spotify-Client fürs Terminal zum Suchen, Einreihen und Steuern der Wiedergabe.
- **sag** — ElevenLabs-Sprachausgabe mit `say`-UX im macOS-Stil; streamt standardmäßig auf Lautsprecher.
- **Sonos CLI** — Steuere Sonos-Lautsprecher (Erkennung/Status/Wiedergabe/Lautstärke/Gruppierung) aus Skripten.
- **blucli** — BluOS-Player aus Skripten abspielen, gruppieren und automatisieren.
- **OpenHue CLI** — Philips-Hue-Lichtsteuerung für Szenen und Automatisierungen.
- **OpenAI Whisper** — Lokale Spracherkennung für schnelle Diktate und Voicemail-Transkripte.
- **Gemini CLI** — Google-Gemini-Modelle vom Terminal aus für schnelle Q&A.
- **bird** — X/Twitter-CLI zum Twittern, Antworten, Lesen von Threads und Suchen ohne Browser.
- **agent-tools** — Toolkit mit Hilfsprogrammen für Automatisierungen und Skripte.

<div id="usage-notes">
  ## Hinweise zur Nutzung
</div>

- Verwende für Scripting bevorzugt die `openclaw` CLI; die Mac‑App kümmert sich um Berechtigungen.
- Führe Installationen im Skills‑Tab aus; der Button wird ausgeblendet, wenn ein Binary bereits vorhanden ist.
- Lass Heartbeats aktiviert, damit der Assistent Erinnerungen planen, Postfächer überwachen und Kameraaufnahmen auslösen kann.
- Die Canvas UI läuft im Vollbild mit nativen Overlays. Platziere keine kritischen Bedienelemente in den oberen Ecken bzw. an den unteren Rändern; füge im Layout explizite Ränder ein und verlasse dich nicht auf „safe‑area insets“.
- Für browsergestützte Verifikation verwende `openclaw browser` (tabs/status/screenshot) mit dem von OpenClaw verwalteten Chrome‑Profil.
- Für DOM‑Inspektion verwende `openclaw browser eval|query|dom|snapshot` (und `--json`/`--out`, wenn du maschinenlesbare Ausgaben benötigst).
- Für Interaktionen verwende `openclaw browser click|type|hover|drag|select|upload|press|wait|navigate|back|evaluate|run` (click/type erfordern Snapshot‑Referenzen; verwende `evaluate` für CSS‑Selektoren).