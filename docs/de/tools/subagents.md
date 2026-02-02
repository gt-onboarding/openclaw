---
title: Subagenten
summary: "Subagenten: Starten isolierter agent-Ausführungen, die ihre Ergebnisse an den anfordernden Chat zurückmelden"
read_when:
  - Du möchtest Hintergrund- oder Parallelaufgaben mit dem agent ausführen
  - Du änderst die sessions_spawn- oder die Tool-Richtlinie für Subagenten
---

<div id="sub-agents">
  # Sub-Agenten
</div>

Sub-Agenten sind Hintergrundläufe eines Agents, die aus einem bestehenden Agent-Lauf gestartet werden. Sie laufen in ihrer eigenen Sitzung (`agent:<agentId>:subagent:<uuid>`) und **melden** ihr Ergebnis nach Abschluss im anfordernden Chat-Kanal zurück.

<div id="slash-command">
  ## Slash-Befehl
</div>

Verwende `/subagents`, um Subagent-Ausführungen für die **aktuelle Sitzung** zu inspizieren oder zu steuern:

- `/subagents list`
- `/subagents stop <id|#|all>`
- `/subagents log <id|#> [limit] [tools]`
- `/subagents info <id|#>`
- `/subagents send <id|#> <message>`

`/subagents info` zeigt Ausführungsmetadaten (Status, Zeitstempel, Sitzungs-ID, Transkriptpfad, Bereinigung).

Hauptziele:

- „Research / lange Aufgabe / langsames Tool“-Arbeit parallelisieren, ohne den Hauptlauf zu blockieren.
- Subagenten standardmäßig isoliert halten (Sitzungstrennung + optionale sandbox).
- Die Tool-Oberfläche gegen Fehlbedienung absichern: Subagenten erhalten standardmäßig **keine** Sitzungs-Tools.
- Verschachteltes Fan-out vermeiden: Subagenten können keine weiteren Subagenten starten.

Hinweis zu Kosten: Jeder Subagent hat seinen **eigenen** Kontext und Tokenverbrauch. Für schwere oder
wiederholte Aufgaben setze ein günstigeres Modell für Subagenten ein und lasse deinen Hauptagenten auf einem höherwertigen Modell laufen.
Du kannst dies über `agents.defaults.subagents.model` oder agent-spezifische Overrides konfigurieren.

<div id="tool">
  ## Tool
</div>

Verwende `sessions_spawn`:

- Startet einen Sub-Agent-Run (`deliver: false`, globale Lane: `subagent`)
- Führt dann einen Announce-Schritt aus und postet die Announce-Antwort in den Chat-Kanal des Anfragenden
- Standardmodell: übernimmt das Modell des Aufrufers, sofern du nicht `agents.defaults.subagents.model` (oder pro Agent `agents.list[].subagents.model`) setzt; ein explizites `sessions_spawn.model` hat immer Vorrang.

Tool-Parameter:

- `task` (erforderlich)
- `label?` (optional)
- `agentId?` (optional; startet unter einer anderen Agent-ID, sofern erlaubt)
- `model?` (optional; überschreibt das Sub-Agent-Modell; ungültige Werte werden übersprungen und der Sub-Agent läuft mit dem Standardmodell weiter, mit einer Warnung im Tool-Ergebnis)
- `thinking?` (optional; überschreibt das Thinking-Level für den Sub-Agent-Run)
- `runTimeoutSeconds?` (Standard `0`; wenn gesetzt, wird der Sub-Agent-Run nach N Sekunden abgebrochen)
- `cleanup?` (`delete|keep`, Standard `keep`)

Allowlist:

- `agents.list[].subagents.allowAgents`: Liste von Agent-IDs, die über `agentId` adressiert werden dürfen (`["*"]`, um alle zu erlauben). Standard: nur der anfragende Agent.

Discovery:

- Verwende `agents_list`, um zu sehen, welche Agent-IDs aktuell für `sessions_spawn` erlaubt sind.

Auto-Archivierung:

- Sub-Agent-Sitzungen werden automatisch nach `agents.defaults.subagents.archiveAfterMinutes` archiviert (Standard: 60).
- Die Archivierung verwendet `sessions.delete` und benennt das Transkript in `*.deleted.<timestamp>` um (gleicher Ordner).
- `cleanup: "delete"` archiviert direkt nach dem Announce (das Transkript bleibt über die Umbenennung erhalten).
- Auto-Archivierung erfolgt nach dem Best-effort-Prinzip; ausstehende Timer gehen verloren, wenn das Gateway neu startet.
- `runTimeoutSeconds` führt **nicht** zur Auto-Archivierung; es stoppt nur den Run. Die Sitzung bleibt bestehen, bis die Auto-Archivierung greift.

<div id="authentication">
  ## Authentifizierung
</div>

Die Authentifizierung von Sub-agents wird über die **Agent-ID** aufgelöst, nicht über den Sitzungstyp:

- Der Sitzungsschlüssel des Sub-agents ist `agent:<agentId>:subagent:<uuid>`.
- Der Auth-Store wird aus dem `agentDir` dieses Agents geladen.
- Die Auth-Profile des Haupt-Agents werden als **Fallback** hinzugefügt; Agent-Profile überschreiben Haupt-Profile bei Konflikten.

Hinweis: Die Zusammenführung ist additiv, daher sind Haupt-Profile immer als Fallbacks verfügbar. Vollständig isolierte Authentifizierung pro Agent wird derzeit noch nicht unterstützt.

<div id="announce">
  ## Announce
</div>

Sub-Agenten melden sich über einen Announce-Schritt zurück:

- Der Announce-Schritt läuft innerhalb der Sub-Agent-Sitzung (nicht in der Sitzung des Anforderers).
- Wenn der Sub-Agent exakt mit `ANNOUNCE_SKIP` antwortet, wird nichts gepostet.
- Andernfalls wird die Announce-Antwort über einen nachgelagerten `agent`-Aufruf (`deliver=true`) an den Chat-Kanal des Anforderers gesendet.
- Announce-Antworten erhalten das Thread-/Themen-Routing, sofern verfügbar (Slack-Threads, Telegram-Themen, Matrix-Threads).
- Announce-Nachrichten werden in eine stabile Vorlage normalisiert:
  - `Status:` abgeleitet aus dem Laufergebnis (`success`, `error`, `timeout` oder `unknown`).
  - `Result:` der zusammengefasste Inhalt aus dem Announce-Schritt (oder `(not available)`, falls nicht vorhanden).
  - `Notes:` Fehlerdetails und weiterer nützlicher Kontext.
- `Status` wird nicht aus der Modellausgabe abgeleitet; er stammt aus Laufzeitsignalen zum Ausführungsergebnis.

Announce-Payloads enthalten am Ende eine Statistikzeile (auch in umbrochener Darstellung):

- Laufzeit (z. B. `runtime 5m12s`)
- Tokenverbrauch (Input/Output/Gesamt)
- Geschätzte Kosten, wenn die Modellbepreisung konfiguriert ist (`models.providers.*.models[].cost`)
- `sessionKey`, `sessionId` und Transkriptpfad (damit der Hauptagent die Historie über `sessions_history` abrufen oder die Datei auf dem Datenträger inspizieren kann)

<div id="tool-policy-sub-agent-tools">
  ## Tool-Richtlinie (Sub-Agent-Tools)
</div>

Standardmäßig haben Sub-Agenten Zugriff auf **alle Tools mit Ausnahme der Sitzungs-Tools**:

* `sessions_list`
* `sessions_history`
* `sessions_send`
* `sessions_spawn`

Setze dies bei Bedarf per Konfiguration außer Kraft:

```json5
{
  agents: {
    defaults: {
      subagents: {
        maxConcurrent: 1
      }
    }
  },
  tools: {
    subagents: {
      tools: {
        // deny hat Vorrang
        deny: ["gateway", "cron"],
        // wenn allow gesetzt ist, wird es zu allow-only (deny hat trotzdem Vorrang)
        // allow: ["read", "exec", "process"]
      }
    }
  }
}
```


<div id="concurrency">
  ## Parallelität
</div>

Sub-Agents verwenden eine eigene In-Process-Queue-Lane:

- Name der Lane: `subagent`
- Parallelität: `agents.defaults.subagents.maxConcurrent` (Standardwert: `8`)

<div id="stopping">
  ## Anhalten
</div>

- Das Senden des Befehls `/stop` im Requester-Chat bricht die Requester-Sitzung ab und stoppt alle aktiven Sub-Agent-Ausführungen, die daraus gestartet wurden.

<div id="limitations">
  ## Einschränkungen
</div>

- Sub-Agent-Ankündigungen erfolgen nach dem **Best-Effort-Prinzip**. Wenn der Gateway neu gestartet wird, gehen ausstehende „announce back“-Aufgaben verloren.
- Sub-Agenten teilen sich weiterhin dieselben Gateway-Prozessressourcen; behandle `maxConcurrent` als Sicherheitsventil.
- `sessions_spawn` ist immer nicht blockierend: Es gibt `{ status: "accepted", runId, childSessionKey }` sofort zurück.
- Der Sub-Agent-Kontext injiziert nur `AGENTS.md` + `TOOLS.md` (kein `SOUL.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md` oder `BOOTSTRAP.md`).