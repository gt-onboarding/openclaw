---
title: Sitzungstool
summary: "Agent-Sitzungstools zum Auflisten von Sitzungen, Abrufen des Verlaufs und Senden sitzungsübergreifender Nachrichten"
read_when:
  - Beim Hinzufügen oder Ändern von Sitzungstools
---

<div id="session-tools">
  # Sitzungs-Tools
</div>

Ziel: ein kleines, schwer falsch zu verwendendes Tool-Set, damit Agenten Sitzungen auflisten, den Verlauf abrufen und an eine andere Sitzung senden können.

<div id="tool-names">
  ## Toolnamen
</div>

- `sessions_list`
- `sessions_history`
- `sessions_send`
- `sessions_spawn`

<div id="key-model">
  ## Schlüsselmodell
</div>

- Der Haupt-Bucket für Direktchats ist immer der wörtliche Schlüssel `"main"` (aufgelöst zum Hauptschlüssel des aktuellen Agents).
- Gruppenchats verwenden `agent:<agentId>:<channel>:group:<id>` oder `agent:<agentId>:<channel>:channel:<id>` (den vollständigen Schlüssel übergeben).
- Cron-Jobs verwenden `cron:<job.id>`.
- Hooks verwenden `hook:<uuid>`, sofern nicht explizit gesetzt.
- Node-Sitzungen verwenden `node-<nodeId>`, sofern nicht explizit gesetzt.

`global` und `unknown` sind reservierte Werte und werden niemals aufgelistet. Wenn `session.scope = "global"` ist, mappen wir ihn auf `main` für alle Tools, sodass Aufrufer niemals `global` sehen.

<div id="sessions_list">
  ## sessions_list
</div>

Listet Sitzungen als Array von Zeilen auf.

Parameter:

- `kinds?: string[]` Filter: eines von `"main" | "group" | "cron" | "hook" | "node" | "other"`
- `limit?: number` maximale Zeilenanzahl (Standard: Server-Standard, begrenzt z. B. auf 200)
- `activeMinutes?: number` nur Sitzungen, die innerhalb der letzten N Minuten aktualisiert wurden
- `messageLimit?: number` 0 = keine Nachrichten (Standard 0); >0 = letzte N Nachrichten einbeziehen

Verhalten:

- `messageLimit > 0` ruft `chat.history` pro Sitzung ab und fügt die letzten N Nachrichten ein.
- Tool-Ergebnisse werden in der Listen-Ausgabe herausgefiltert; verwende `sessions_history` für Tool-Nachrichten.
- Beim Ausführen in einer **sandboxed** Agent-Sitzung haben Sitzungstools standardmäßig **Sichtbarkeit nur für gespawnte Tools** (siehe unten).

Zeilenstruktur (JSON):

- `key`: Sitzungsschlüssel (string)
- `kind`: `main | group | cron | hook | node | other`
- `channel`: `whatsapp | telegram | discord | signal | imessage | webchat | internal | unknown`
- `displayName` (Gruppenanzeigelabel, falls verfügbar)
- `updatedAt` (ms)
- `sessionId`
- `model`, `contextTokens`, `totalTokens`
- `thinkingLevel`, `verboseLevel`, `systemSent`, `abortedLastRun`
- `sendPolicy` (Sitzungs-Override, falls gesetzt)
- `lastChannel`, `lastTo`
- `deliveryContext` (normalisiertes `{ channel, to, accountId }`, falls verfügbar)
- `transcriptPath` (Best-Effort-Pfad, abgeleitet aus Store-Verzeichnis + sessionId)
- `messages?` (nur wenn `messageLimit > 0`)

<div id="sessions_history">
  ## sessions_history
</div>

Transkript für eine Sitzung abrufen.

Parameter:

- `sessionKey` (erforderlich; akzeptiert Sitzungsschlüssel oder `sessionId` aus `sessions_list`)
- `limit?: number` maximale Anzahl von Nachrichten (Server begrenzt bei Bedarf)
- `includeTools?: boolean` (Standardwert `false`)

Verhalten:

- `includeTools=false` filtert Nachrichten mit `role: "toolResult"`.
- Gibt ein Nachrichten-Array im Roh-Transkriptformat zurück.
- Wenn eine `sessionId` übergeben wird, löst OpenClaw diese in den entsprechenden Sitzungsschlüssel auf (Fehler bei fehlenden IDs).

<div id="sessions_send">
  ## sessions_send
</div>

Sende eine Nachricht in eine andere Sitzung.

Parameter:

- `sessionKey` (erforderlich; akzeptiert Sitzungsschlüssel oder `sessionId` aus `sessions_list`)
- `message` (erforderlich)
- `timeoutSeconds?: number` (Standard > 0; 0 = fire-and-forget)

Verhalten:

- `timeoutSeconds = 0`: in die Warteschlange stellen und `{ runId, status: "accepted" }` zurückgeben.
- `timeoutSeconds > 0`: bis zu N Sekunden auf Abschluss warten, dann `{ runId, status: "ok", reply }` zurückgeben.
- Wenn die Wartezeit abläuft: `{ runId, status: "timeout", error }`. Der Run läuft weiter; rufe später `sessions_history` auf.
- Wenn der Run fehlschlägt: `{ runId, status: "error", error }`.
- Ankündigungs-Runs werden ausgeführt, nachdem der primäre Run abgeschlossen ist, und sind Best-Effort; `status: "ok"` garantiert nicht, dass die Ankündigung zugestellt wurde.
- Wartet über das Gateway mittels `agent.wait` (serverseitig), sodass Verbindungsabbrüche das Warten nicht abbrechen.
- Agent-zu-Agent-Nachrichtenkontext wird für den primären Run injiziert.
- Nachdem der primäre Run abgeschlossen ist, führt OpenClaw eine **Reply-Back-Schleife** aus:
  - Ab Runde 2 wird zwischen anfragendem und Zielagenten abgewechselt.
  - Antworte exakt mit `REPLY_SKIP`, um das Ping-Pong zu stoppen.
  - Die maximale Anzahl an Runden ist `session.agentToAgent.maxPingPongTurns` (0–5, Standard 5).
- Sobald die Schleife endet, führt OpenClaw den **Agent-zu-Agent-Ankündigungsschritt** aus (nur Zielagent):
  - Antworte exakt mit `ANNOUNCE_SKIP`, um still zu bleiben.
  - Jede andere Antwort wird an den Zielkanal gesendet.
  - Der Ankündigungsschritt enthält die ursprüngliche Anfrage + die Antwort aus Runde 1 + die letzte Ping-Pong-Antwort.

<div id="channel-field">
  ## Channel-Feld
</div>

- Für Gruppen ist `channel` der Channel, der im Sitzungseintrag protokolliert ist.
- Für Direktchats wird `channel` aus `lastChannel` abgeleitet.
- Für cron/hook/knoten ist `channel` `internal`.
- Wenn `channel` fehlt, ist der Wert `unknown`.

<div id="security-send-policy">
  ## Sicherheit / Send-Richtlinie
</div>

Richtlinienbasiertes Blockieren nach Kanal- bzw. Chat-Typ (nicht pro Sitzungs-ID).

```json
{
  "session": {
    "sendPolicy": {
      "rules": [
        {
          "match": { "channel": "discord", "chatType": "group" },
          "action": "deny"
        }
      ],
      "default": "allow"
    }
  }
}
```

Laufzeit-Override (pro Sitzungseintrag):

* `sendPolicy: "allow" | "deny"` (nicht gesetzt = Konfiguration wird übernommen)
* Konfigurierbar über `sessions.patch` oder nur durch den Besitzer via `/send on|off|inherit` (eigenständige Nachricht).

Stellen der Durchsetzung:

* `chat.send` / `agent` (Gateway)
* Logik zur Zustellung automatischer Antworten


<div id="sessions_spawn">
  ## sessions_spawn
</div>

Startet eine Sub-Agent-Ausführung in einer isolierten Sitzung und kündigt das Ergebnis im anfragenden Chat-Kanal an.

Parameter:

- `task` (erforderlich)
- `label?` (optional; für Logs/UI verwendet)
- `agentId?` (optional; startet unter einer anderen Agent-ID, falls erlaubt)
- `model?` (optional; überschreibt das Sub-Agent-Modell; ungültige Werte führen zu einem Fehler)
- `runTimeoutSeconds?` (Standardwert 0; wenn gesetzt, bricht die Sub-Agent-Ausführung nach N Sekunden ab)
- `cleanup?` (`delete|keep`, Standardwert `keep`)

Allowlist:

- `agents.list[].subagents.allowAgents`: Liste von Agent-IDs, die über `agentId` erlaubt sind (`["*"]`, um alle zu erlauben). Standard: nur der anfragende Agent.

Discovery:

- Verwenden Sie `agents_list`, um herauszufinden, welche Agent-IDs für `sessions_spawn` erlaubt sind.

Verhalten:

- Startet eine neue `agent:<agentId>:subagent:<uuid>`-Sitzung mit `deliver: false`.
- Sub-Agenten verwenden standardmäßig das vollständige Toolset **ohne Session-Tools** (konfigurierbar über `tools.subagents.tools`).
- Sub-Agenten dürfen `sessions_spawn` nicht aufrufen (kein Sub-Agent-→-Sub-Agent-Spawning).
- Immer nicht-blockierend: gibt sofort `{ status: "accepted", runId, childSessionKey }` zurück.
- Nach Abschluss führt OpenClaw einen **Announce-Schritt** für den Sub-Agent aus und postet das Ergebnis im anfragenden Chat-Kanal.
- Antworten Sie während des Announce-Schritts exakt mit `ANNOUNCE_SKIP`, um keine Ausgabe zu erzeugen.
- Announce-Antworten werden auf `Status`/`Result`/`Notes` normalisiert; `Status` stammt aus dem Laufzeitergebnis (nicht aus dem Modelltext).
- Sub-Agent-Sitzungen werden nach `agents.defaults.subagents.archiveAfterMinutes` automatisch archiviert (Standardwert: 60).
- Announce-Antworten enthalten eine Statistikzeile (Laufzeit, Tokens, sessionKey/sessionId, Transkriptpfad und optionale Kosten).

<div id="sandbox-session-visibility">
  ## Sichtbarkeit von Sandbox-Sitzungen
</div>

Sandbox-Sitzungen können Sitzungstools verwenden, sehen jedoch standardmäßig nur Sitzungen, die sie selbst über `sessions_spawn` erzeugt haben.

Konfiguration:

```json5
{
  agents: {
    defaults: {
      sandbox: {
        // default: "spawned"
        sessionToolsVisibility: "spawned" // oder "all"
      }
    }
  }
}
```
