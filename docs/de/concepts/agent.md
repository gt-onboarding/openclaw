---
title: Agent
summary: "Agent-Laufzeit (eingebettetes pi-mono), Arbeitsbereichs-Contract und Sitzungs-Bootstrap"
read_when:
  - Beim Ändern der Agent-Laufzeit, des Arbeitsbereichs-Bootstrap oder des Sitzungsverhaltens
---

<div id="agent-runtime">
  # Agent-Laufzeitumgebung 🤖
</div>

OpenClaw betreibt eine einzelne eingebettete Agent-Laufzeitumgebung, die von **pi-mono** abgeleitet ist.

<div id="workspace-required">
  ## Arbeitsbereich (erforderlich)
</div>

OpenClaw verwendet ein einzelnes Agent-Arbeitsverzeichnis (`agents.defaults.workspace`) als das **einzige** Arbeitsverzeichnis (`cwd`) des agents für Tools und Kontext.

Empfehlung: Verwende `openclaw setup`, um `~/.openclaw/openclaw.json` zu erstellen, falls diese noch nicht existiert, und die Arbeitsbereichsdateien zu initialisieren.

Vollständiges Layout des Arbeitsbereichs + Backup-Anleitung: [Agent-Arbeitsbereich](/de/concepts/agent-workspace)

Wenn `agents.defaults.sandbox` aktiviert ist, können Nicht-Main-Sitzungen dies mit
Arbeitsbereichen pro Sitzung unter `agents.defaults.sandbox.workspaceRoot` überschreiben (siehe
[Gateway-Konfiguration](/de/gateway/configuration)).

<div id="bootstrap-files-injected">
  ## Bootstrap-Dateien (injectiert)
</div>

Innerhalb von `agents.defaults.workspace` erwartet OpenClaw diese vom Benutzer bearbeitbaren Dateien:

* `AGENTS.md` — Bedienungsanleitung + „Speicher“
* `SOUL.md` — Persona, Grenzen, Tonfall
* `TOOLS.md` — vom Benutzer gepflegte Tool-Notizen (z. B. `imsg`, `sag`, Konventionen)
* `BOOTSTRAP.md` — einmaliges Ritual beim ersten Start (nach Abschluss gelöscht)
* `IDENTITY.md` — Agent-Name/Vibe/Emoji
* `USER.md` — Benutzerprofil + bevorzugte Anrede

Beim ersten Turn einer neuen Sitzung injiziert OpenClaw den Inhalt dieser Dateien direkt in den Agent-Kontext.

Leere Dateien werden übersprungen. Große Dateien werden gekürzt und mit einem Marker abgeschnitten, sodass Prompts schlank bleiben (lies die Datei für den vollständigen Inhalt).

Wenn eine Datei fehlt, injiziert OpenClaw eine einzelne „fehlende Datei“-Markerzeile (und `openclaw setup` erstellt eine sichere Standardvorlage).

`BOOTSTRAP.md` wird nur für einen **nagelneuen Arbeitsbereich** erstellt (keine anderen Bootstrap-Dateien vorhanden). Wenn du sie nach Abschluss des Rituals löschst, sollte sie bei späteren Neustarts nicht erneut erstellt werden.

Um die Erstellung von Bootstrap-Dateien vollständig zu deaktivieren (für vorab befüllte Arbeitsbereiche), setze:

```json5
{ agent: { skipBootstrap: true } }
```

<div id="built-in-tools">
  ## Eingebaute Tools
</div>

Kern-Tools (`read`/`exec`/`edit`/`write` und verwandte System-Tools) sind immer verfügbar,
vorbehaltlich der Tool-Policy. `apply_patch` ist optional und wird durch
`tools.exec.applyPatch` gesteuert. `TOOLS.md` steuert **nicht**, welche Tools existieren; es ist
eine Anleitung dafür, wie *du* möchtest, dass sie verwendet werden sollen.

<div id="skills">
  ## Fähigkeiten
</div>

OpenClaw lädt Fähigkeiten aus drei Quellen (bei Namenskonflikten hat der Arbeitsbereich Vorrang):

* Integriert (mit der Installation ausgeliefert)
* Verwaltet/lokal: `~/.openclaw/skills`
* Arbeitsbereich: `<workspace>/skills`

Fähigkeiten können über Konfiguration/Umgebungsvariablen gesteuert bzw. eingeschränkt werden (siehe `skills` in der [Gateway-Konfiguration](/de/gateway/configuration)).

<div id="pi-mono-integration">
  ## pi-mono-Integration
</div>

OpenClaw verwendet Teile der pi-mono-Codebasis wieder (Modelle/Tools), aber **Sitzungsverwaltung, Discovery und Tool-Verdrahtung sind OpenClaw-eigen**.

* Keine pi-coding-Agent-Runtime.
* `~/.pi/agent`- oder `<workspace>/.pi`-Einstellungen werden nicht ausgewertet.

<div id="sessions">
  ## Sitzungen
</div>

Sitzungsprotokolle werden im JSONL-Format gespeichert unter:

* `~/.openclaw/agents/<agentId>/sessions/<SessionId>.jsonl`

Die Sitzungs-ID ist eindeutig und wird von OpenClaw vergeben.
Legacy-Pi/Tau-Sitzungsverzeichnisse werden **nicht** eingelesen.

<div id="steering-while-streaming">
  ## Steuern während des Streamings
</div>

Wenn der Warteschlangenmodus `steer` ist, werden eingehende Nachrichten in den aktuellen Run eingespeist.
Die Warteschlange wird **nach jedem Tool-Aufruf** geprüft; wenn eine Nachricht in der Warteschlange vorhanden ist,
werden verbleibende Tool-Aufrufe aus der aktuellen Assistant-Nachricht übersprungen (Error-Tool
liefert Ergebnisse mit &quot;Skipped due to queued user message.&quot;), dann wird die eingereihte Benutzernachricht
vor der nächsten Assistant-Antwort eingespeist.

Wenn der Warteschlangenmodus `followup` oder `collect` ist, werden eingehende Nachrichten zurückgehalten, bis der
aktuelle Turn endet, dann beginnt ein neuer Agent-Turn mit den eingereihten Payloads. Siehe
[Queue](/de/concepts/queue) für Modus- sowie Debounce-/Cap-Verhalten.

Block-Streaming sendet abgeschlossene Assistant-Blöcke, sobald sie fertig sind; es ist
**standardmäßig deaktiviert** (`agents.defaults.blockStreamingDefault: "off"`).
Justiere die Grenze über `agents.defaults.blockStreamingBreak` (`text_end` vs `message_end`; Standard ist text&#95;end).
Steuere das Soft-Block-Chunking mit `agents.defaults.blockStreamingChunk` (Standard:
800–1200 Zeichen; bevorzugt Absatzumbrüche, dann Zeilenumbrüche, zuletzt Sätze).
Fasse gestreamte Chunks mit `agents.defaults.blockStreamingCoalesce` zusammen, um
Einzeilen-Spam zu reduzieren (leerlaufbasiertes Zusammenführen vor dem Senden). Nicht-Telegram-Kanäle benötigen
explizit `*.blockStreaming: true`, um Block-Antworten zu aktivieren.
Ausführliche Tool-Zusammenfassungen werden beim Start des Tools ausgegeben (kein Debounce); die Control UI
streamt Tool-Ausgaben über Agent-Events, wenn verfügbar.
Weitere Details: [Streaming + chunking](/de/concepts/streaming).

<div id="model-refs">
  ## Modell-Referenzen
</div>

Modell-Referenzen in der Konfiguration (zum Beispiel `agents.defaults.model` und `agents.defaults.models`) werden verarbeitet, indem am **ersten** `/` getrennt wird.

* Verwende `provider/model` bei der Konfiguration von Modellen.
* Wenn die Modell-ID selbst `/` enthält (OpenRouter-Stil), füge das Anbieterpräfix hinzu (Beispiel: `openrouter/moonshotai/kimi-k2`).
* Wenn du den Anbieter weglässt, behandelt OpenClaw die Eingabe als Alias oder als Modell für den **Standardanbieter** (funktioniert nur, wenn kein `/` in der Modell-ID vorkommt).

<div id="configuration-minimal">
  ## Konfiguration (minimal)
</div>

Lege mindestens Folgendes fest:

* `agents.defaults.workspace`
* `channels.whatsapp.allowFrom` (dringend empfohlen)

***

*Weiter: [Gruppenchats](/de/concepts/group-messages)* 🦞