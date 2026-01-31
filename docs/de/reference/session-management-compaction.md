---
title: Sitzungsverwaltung &amp; Kompaktierung
summary: "Deep Dive: Sitzungs-Store und Transkripte, Lebenszyklus und Interna der (Auto-)Kompaktierung"
read_when:
  - Du Sitzungs-IDs, Transcript-JSONL oder Felder in sessions.json debuggen musst
  - Du das Verhalten der Auto-Kompaktierung änderst oder „Pre-Compaction“-Aufräumaufgaben hinzufügst
  - Du Speicher-Flushes oder stille System-Turns implementieren möchtest
---

<div id="session-management-compaction-deep-dive">
  # Sitzungsverwaltung &amp; Kompaktierung (Deep Dive)
</div>

Dieses Dokument erklärt, wie OpenClaw Sitzungen Ende-zu-Ende verwaltet:

* **Sitzungsrouting** (wie eingehende Nachrichten einem `sessionKey` zugeordnet werden)
* **Sitzungsspeicher** (`sessions.json`) und was darin nachverfolgt wird
* **Persistenz von Transkripten** (`*.jsonl`) und deren Struktur
* **Transkripthygiene** (anbieterspezifische Korrekturen vor Ausführungen)
* **Kontextgrenzen** (Kontextfenster vs. verfolgte Tokens)
* **Kompaktierung** (manuelle + automatische Kompaktierung) und wo du Arbeiten vor der Kompaktierung einhängen kannst
* **Stille Hintergrundaufgaben** (z. B. Memory-Schreibvorgänge, die keine für Nutzer sichtbare Ausgabe erzeugen sollen)

Wenn du zuerst eine Übersicht auf höherer Ebene möchtest, beginne hier:

* [/concepts/session](/de/concepts/session)
* [/concepts/compaction](/de/concepts/compaction)
* [/concepts/session-pruning](/de/concepts/session-pruning)
* [/reference/transcript-hygiene](/de/reference/transcript-hygiene)

***

<div id="source-of-truth-the-gateway">
  ## Verbindliche Referenz: der Gateway
</div>

OpenClaw ist um einen einzelnen **Gateway-Prozess** zentriert, der den Sitzungszustand verwaltet.

* UIs (macOS-App, webbasierte Control UI, TUI) sollten den Gateway nach Sitzungslisten und Token-Zahlen abfragen.
* Im Remote-Modus befinden sich Sitzungsdateien auf dem Remote-Host; „das Überprüfen deiner lokalen Mac-Dateien“ spiegelt nicht wider, welche Daten der Gateway tatsächlich verwendet.

***

<div id="two-persistence-layers">
  ## Zwei Persistenzschichten
</div>

OpenClaw speichert Sitzungen in zwei Schichten:

1. **Session Store (`sessions.json`)**
   * Key/Value-Map: `sessionKey -> SessionEntry`
   * Klein, veränderbar, gefahrlos zu bearbeiten (oder Einträge zu löschen)
   * Enthält Sitzungsmetadaten (aktuelle Sitzungs-ID, letzte Aktivität, Schalter, Token-Zähler usw.)

2. **Transcript (`<sessionId>.jsonl`)**
   * Anhänge-only-Protokoll mit Baumstruktur (Einträge haben `id` + `parentId`)
   * Speichert die eigentliche Konversation + Tool-Aufrufe + Kompaktierungszusammenfassungen
   * Wird verwendet, um den Modellkontext für zukünftige Interaktionen wiederherzustellen

***

<div id="on-disk-locations">
  ## Speicherorte auf dem Datenträger
</div>

Pro Agent auf dem Gateway-Host:

* Speicher: `~/.openclaw/agents/<agentId>/sessions/sessions.json`
* Transkripte: `~/.openclaw/agents/<agentId>/sessions/<sessionId>.jsonl`
  * Telegram-Themen-Sitzungen: `.../<sessionId>-topic-<threadId>.jsonl`

OpenClaw löst diese über `src/config/sessions.ts` auf.

***

<div id="session-keys-sessionkey">
  ## Sitzungsschlüssel (`sessionKey`)
</div>

Ein `sessionKey` identifiziert, *in welchem Gesprächskontext* du dich befindest (Routing + Isolation).

Gängige Muster:

* Haupt-/Direktchat (pro Agent): `agent:<agentId>:<mainKey>` (Standard: `main`)
* Gruppe: `agent:<agentId>:<channel>:group:<id>`
* Raum/Kanal (Discord/Slack): `agent:<agentId>:<channel>:channel:<id>` oder `...:room:<id>`
* Cron: `cron:<job.id>`
* Webhook: `hook:<uuid>` (sofern nicht überschrieben)

Die kanonischen Regeln sind unter [/concepts/session](/de/concepts/session) dokumentiert.

***

<div id="session-ids-sessionid">
  ## Sitzungs-IDs (`sessionId`)
</div>

Jeder `sessionKey` verweist auf eine aktuelle `sessionId` (die Transkriptdatei, die den Gesprächsverlauf fortführt).

Faustregeln:

* **Reset** (`/new`, `/reset`) erzeugt eine neue `sessionId` für diesen `sessionKey`.
* **Täglicher Reset** (standardmäßig um 4:00 Uhr Ortszeit auf dem Gateway-Host) erzeugt eine neue `sessionId` bei der nächsten Nachricht nach dem Reset-Grenzzeitpunkt.
* **Ablauf bei Inaktivität** (`session.reset.idleMinutes` oder älteres `session.idleMinutes`) erzeugt eine neue `sessionId`, wenn eine Nachricht nach Ablauf des Inaktivitätsfensters eintrifft. Wenn sowohl täglicher Reset als auch Inaktivität konfiguriert sind, setzt sich die Regel durch, die zuerst greift.

Implementierungsdetail: Die Entscheidung erfolgt in `initSessionState()` in `src/auto-reply/reply/session.ts`.

***

<div id="session-store-schema-sessionsjson">
  ## Sitzungsspeicher-Schema (`sessions.json`)
</div>

Der Werttyp des Speichers ist `SessionEntry` in `src/config/sessions.ts`.

Wichtige Felder (nicht vollständig):

* `sessionId`: aktuelle Transkript-ID (Dateiname wird hiervon abgeleitet, sofern `sessionFile` nicht gesetzt ist)
* `updatedAt`: Zeitstempel der letzten Aktivität
* `sessionFile`: optionale explizite Überschreibung des Transkriptpfads
* `chatType`: `direct | group | room` (hilft UIs und der Send-Richtlinie)
* `provider`, `subject`, `room`, `space`, `displayName`: Metadaten für Gruppen-/Kanalbezeichnungen
* Umschalter:
  * `thinkingLevel`, `verboseLevel`, `reasoningLevel`, `elevatedLevel`
  * `sendPolicy` (sitzungsspezifische Überschreibung)
* Modellauswahl:
  * `providerOverride`, `modelOverride`, `authProfileOverride`
* Token-Zähler (Best Effort / anbieterabhängig):
  * `inputTokens`, `outputTokens`, `totalTokens`, `contextTokens`
* `compactionCount`: wie oft die automatische Kompaktierung für diesen Sitzungsschlüssel abgeschlossen wurde
* `memoryFlushAt`: Zeitstempel des letzten Speicher-Flush vor der Kompaktierung
* `memoryFlushCompactionCount`: Kompaktierungszähler zum Zeitpunkt, als der letzte Flush ausgeführt wurde

Der Speicher kann gefahrlos bearbeitet werden, aber das Gateway ist die maßgebliche Instanz: Es kann Einträge umschreiben oder rehydrieren, während Sitzungen laufen.

***

<div id="transcript-structure-jsonl">
  ## Transkriptstruktur (`*.jsonl`)
</div>

Transkripte werden vom `SessionManager` von `@mariozechner/pi-coding-agent` verwaltet.

Die Datei liegt im JSONL-Format vor:

* Erste Zeile: Sitzungs-Header (`type: "session"`, enthält `id`, `cwd`, `timestamp`, optional `parentSession`)
* Danach: Sitzungs-Einträge mit `id` + `parentId` (Baumstruktur)

Wichtige Eintragstypen:

* `message`: user/assistant/toolResult-Nachrichten
* `custom_message`: von Erweiterungen injizierte Nachrichten, die *tatsächlich* in den Modellkontext eingehen (können in der UI verborgen werden)
* `custom`: Erweiterungszustand, der *nicht* in den Modellkontext eingeht
* `compaction`: persistierte Kompaktierungszusammenfassung mit `firstKeptEntryId` und `tokensBefore`
* `branch_summary`: persistierte Zusammenfassung beim Navigieren eines Baumzweigs

OpenClaw „bereinigt“ Transkripte absichtlich **nicht**; das Gateway verwendet `SessionManager`, um sie zu lesen und zu schreiben.

***

<div id="context-windows-vs-tracked-tokens">
  ## Kontextfenster vs. nachverfolgte Tokens
</div>

Zwei unterschiedliche Konzepte sind relevant:

1. **Kontextfenster des Modells**: harte Obergrenze pro Modell (Tokens, die für das Modell sichtbar sind)
2. **Zähler im Sitzungsspeicher**: gleitende Statistiken, die in `sessions.json` geschrieben werden (verwendet für /status und Dashboards)

Wenn du Grenzwerte feinjustierst:

* Das Kontextfenster stammt aus dem Modellkatalog (und kann über die Konfiguration überschrieben werden).
* `contextTokens` im Speicher ist ein Laufzeit-Schätzwert/Reporting-Wert; behandle ihn nicht als strikte Garantie.

Weitere Informationen findest du unter [/token-use](/de/token-use).

***

<div id="compaction-what-it-is">
  ## Compaction: Was ist das?
</div>

Compaction fasst ältere Teile einer Unterhaltung in einem dauerhaft gespeicherten `compaction`-Eintrag im Transkript zusammen und lässt aktuelle Nachrichten unverändert.

Nach der Compaction sehen spätere Dialogrunden:

* Die Compaction-Zusammenfassung
* Nachrichten nach `firstKeptEntryId`

Compaction ist **persistent** (im Unterschied zur Sitzungsbereinigung). Siehe [/concepts/session-pruning](/de/concepts/session-pruning).

***

<div id="when-auto-compaction-happens-pi-runtime">
  ## Wann Auto-Komprimierung stattfindet (Pi-Runtime)
</div>

Im eingebetteten Pi-Agent wird die Auto-Komprimierung in zwei Fällen ausgelöst:

1. **Overflow-Wiederherstellung**: Das Modell gibt einen Kontextüberlauf-Fehler zurück → komprimieren → erneut versuchen.
2. **Schwellenwert-Wartung**: nach einem erfolgreichen Turn, wenn:

`contextTokens > contextWindow - reserveTokens`

Dabei gilt:

* `contextWindow` ist das Kontextfenster des Modells
* `reserveTokens` ist der Puffer, der für Prompts + die nächste Modellausgabe reserviert wird

Dies sind Pi-Runtime-Semantiken (OpenClaw konsumiert die Events, aber Pi entscheidet, wann komprimiert wird).

***

<div id="compaction-settings-reservetokens-keeprecenttokens">
  ## Einstellungen zur Kompaktierung (`reserveTokens`, `keepRecentTokens`)
</div>

Die Einstellungen zur Kompaktierung von Pi befinden sich in den Pi-Einstellungen:

```json5
{
  compaction: {
    enabled: true,
    reserveTokens: 16384,
    keepRecentTokens: 20000
  }
}
```

OpenClaw erzwingt außerdem eine Sicherheitsuntergrenze für eingebettete Runs:

* Wenn `compaction.reserveTokens < reserveTokensFloor` ist, erhöht OpenClaw diesen Wert.
* Die Standarduntergrenze beträgt `20000` Token.
* Setze `agents.defaults.compaction.reserveTokensFloor: 0`, um die Untergrenze zu deaktivieren.
* Wenn der Wert bereits höher ist, lässt OpenClaw ihn unverändert.

Warum: Es soll genug Spielraum für mehrfache „Housekeeping“-Runden (wie Speicherschreibvorgänge) bleiben, bevor Kompaktierung unvermeidbar wird.

Implementierung: `ensurePiCompactionReserveTokens()` in `src/agents/pi-settings.ts`
(aufgerufen von `src/agents/pi-embedded-runner.ts`).

***

<div id="user-visible-surfaces">
  ## Für Nutzer sichtbare Oberflächen
</div>

Du kannst den Kompaktierungs- und Sitzungszustand über Folgendes einsehen:

* `/status` (in jeder Chatsitzung)
* `openclaw status` (CLI)
* `openclaw sessions` / `sessions --json`
* Ausführlicher Modus: `🧹 Auto-compaction complete` + Anzahl der Kompaktierungen

***

<div id="silent-housekeeping-no_reply">
  ## Stilles Housekeeping (`NO_REPLY`)
</div>

OpenClaw unterstützt „stille“ Turns für Hintergrundaufgaben, bei denen der Nutzer keine Zwischenausgaben sehen soll.

Konvention:

* Der Assistant beginnt seine Ausgabe mit `NO_REPLY`, um anzuzeigen: „Keine Antwort an den Nutzer ausliefern“.
* OpenClaw entfernt/unterdrückt dies in der Auslieferungsschicht.

Seit `2026.1.10` unterdrückt OpenClaw außerdem **Draft-/Typing-Streaming**, wenn ein Teil-Chunk mit `NO_REPLY` beginnt, sodass stille Operationen keine Teilausgaben während eines Turns preisgeben.

<div id="pre-compaction-memory-flush-implemented">
  ## „Memory Flush“ vor der Komprimierung (implementiert)
</div>

Ziel: Bevor die automatische Komprimierung stattfindet, eine stille Agenten-Ausführung durchführen, die dauerhaften
Zustand auf die Festplatte schreibt (z. B. `memory/YYYY-MM-DD.md` im Arbeitsbereich des Agenten), damit die Komprimierung keinen
kritischen Kontext löschen kann.

OpenClaw verwendet den **Pre-Threshold-Flush**-Ansatz:

1. Nutzung des Sitzungskontexts überwachen.
2. Wenn ein „Soft-Threshold“ überschritten wird (unterhalb von Pis Komprimierungsschwelle), eine stille
   „Schreibe jetzt Speicher“-Direktive an den agent senden.
3. `NO_REPLY` verwenden, damit der Nutzer nichts sieht.

Konfiguration (`agents.defaults.compaction.memoryFlush`):

* `enabled` (Standard: `true`)
* `softThresholdTokens` (Standard: `4000`)
* `prompt` (Nutzernachricht für den Flush-Turn)
* `systemPrompt` (zusätzlicher System-Prompt, der für den Flush-Turn angehängt wird)

Hinweise:

* Der Standard-Prompt/System-Prompt enthält einen `NO_REPLY`-Hinweis, um die Zustellung zu unterdrücken.
* Der Flush wird einmal pro Komprimierungszyklus ausgeführt (nachverfolgt in `sessions.json`).
* Der Flush wird nur für eingebettete Pi-Sitzungen ausgeführt (CLI-Backends überspringen ihn).
* Der Flush wird übersprungen, wenn der Sitzungsarbeitsbereich schreibgeschützt ist (`workspaceAccess: "ro"` oder `"none"`).
* Siehe [Memory](/de/concepts/memory) für das Dateilayout des Arbeitsbereichs und die Schreibmuster.

Pi stellt außerdem einen `session_before_compact`-Hook in der Extension-API bereit, aber OpenClaws
Flush-Logik befindet sich derzeit auf der Gateway-Seite.

<div id="troubleshooting-checklist">
  ## Checkliste zur Fehlerbehebung
</div>

* Sitzungsschlüssel falsch? Starte mit [/concepts/session](/de/concepts/session) und bestätige den `sessionKey` in `/status`.
* Abgleich zwischen Store und Transkript fehlerhaft? Bestätige den Gateway-Host und den Store-Pfad aus `openclaw status`.
* Zu viele Kompaktierungsvorgänge? Prüfe:
  * Kontextfenster des Modells (zu klein)
  * Kompaktierungseinstellungen (`reserveTokens` zu hoch im Verhältnis zum Kontextfenster des Modells kann zu früherer Kompaktierung führen)
  * Aufblähung von Tool-Ergebnissen: Sitzungsbereinigung aktivieren/feinjustieren
* „Silent turns“ werden trotzdem gesendet? Bestätige, dass die Antwort mit `NO_REPLY` (exaktes Token) beginnt und dass du einen Build verwendest, der den Fix zur Unterdrückung beim Streaming enthält.