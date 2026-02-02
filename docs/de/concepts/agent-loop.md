---
title: Agent Loop
summary: "Lebenszyklus des Agent Loops, Streams und Warte-Semantik"
read_when:
  - Sie benötigen eine präzise Schritt-für-Schritt-Darstellung des Agent Loops oder der Lebenszyklus-Ereignisse
---

<div id="agent-loop-openclaw">
  # Agent Loop (OpenClaw)
</div>

Ein Agent-Loop ist der vollständige „echte“ Durchlauf eines Agents: Intake → Kontextaufbau → Modellinferenz →
Tool-Ausführung → Streaming-Antworten → Persistenz. Er ist der maßgebliche Pfad, der eine Nachricht
in Aktionen und eine finale Antwort verwandelt und dabei den Sitzungszustand konsistent hält.

In OpenClaw ist ein Loop ein einzelner, serialisierter Durchlauf pro Sitzung, der Lifecycle- und Stream-Events
emittiert, während das Modell „nachdenkt“, Tools aufruft und Ausgaben streamt. Dieses Dokument erklärt, wie dieser eigentliche Loop
Ende-zu-Ende implementiert ist.

<div id="entry-points">
  ## Einstiegspunkte
</div>

* Gateway-RPC: `agent` und `agent.wait`.
* CLI: `agent`-Befehl.

<div id="how-it-works-high-level">
  ## Funktionsweise (auf hoher Ebene)
</div>

1. `agent`-RPC validiert Parameter, ermittelt die Sitzung (sessionKey/sessionId), persistiert Sitzungsmetadaten und gibt sofort `{ runId, acceptedAt }` zurück.
2. `agentCommand` führt den agent aus:
   * ermittelt Modell + Defaultwerte für Thinking/Verbose
   * lädt den Fähigkeiten-Snapshot
   * ruft `runEmbeddedPiAgent` (pi-agent-core-Runtime) auf
   * sendet **Lifecycle end/error**, falls die eingebettete Schleife keines sendet
3. `runEmbeddedPiAgent`:
   * serialisiert Ausführungen über sitzungsbezogene und globale Queues
   * ermittelt Modell + Auth-Profil und baut die pi-Sitzung
   * abonniert pi-Events und streamt Assistant-/Tool-Deltas
   * erzwingt ein Timeout und bricht die Ausführung ab, wenn es überschritten wird
   * gibt Payloads + Nutzungsmetadaten zurück
4. `subscribeEmbeddedPiSession` überbrückt pi-agent-core-Events zum OpenClaw-`agent`-Stream:
   * Tool-Events =&gt; `stream: "tool"`
   * Assistant-Deltas =&gt; `stream: "assistant"`
   * Lifecycle-Events =&gt; `stream: "lifecycle"` (`phase: "start" | "end" | "error"`)
5. `agent.wait` verwendet `waitForAgentJob`:
   * wartet auf **Lifecycle end/error** für `runId`
   * gibt `{ status: ok|error|timeout, startedAt, endedAt, error? }` zurück

<div id="queueing-concurrency">
  ## Queueing + Nebenläufigkeit
</div>

* Läufe werden pro Sitzungsschlüssel (Sitzungs-Lane) und optional über eine globale Lane serialisiert.
* Dies verhindert Race-Conditions zwischen Tools und Sitzungen und hält den Sitzungsverlauf konsistent.
* Nachrichtenkanäle können Warteschlangenmodi (collect/steer/followup) wählen, die dieses Lane-System bedienen.
  Siehe [Command Queue](/de/concepts/queue).

<div id="session-workspace-preparation">
  ## Sitzung + Vorbereitung des Arbeitsbereichs
</div>

* Arbeitsbereich wird ermittelt und erstellt; Ausführungen in der sandbox können auf das Root des sandbox-Arbeitsbereichs umleiten.
* Fähigkeiten werden geladen (oder aus einem Snapshot wiederverwendet) und in `env` und Prompt eingespeist.
* Bootstrap-/Kontextdateien werden ermittelt und in den System-Prompt-Bericht eingespeist.
* Es wird eine Schreibsperre für die Sitzung gesetzt; der `SessionManager` wird geöffnet und vor dem Streaming vorbereitet.

<div id="prompt-assembly-system-prompt">
  ## Prompt-Zusammenstellung + System-Prompt
</div>

* Der System-Prompt wird aus OpenClaws Basis-Prompt, dem Fähigkeiten-Prompt, dem Bootstrap-Kontext und laufbezogenen Überschreibungen aufgebaut.
* Modellspezifische Grenzwerte und Token-Reserven für Kompaktierung werden durchgesetzt.
* Siehe [System prompt](/de/concepts/system-prompt), um zu sehen, was das Modell erhält.

<div id="hook-points-where-you-can-intercept">
  ## Hook-Punkte (Stellen, an denen du eingreifen kannst)
</div>

OpenClaw bietet zwei Hook-Systeme:

* **Interne Hooks** (Gateway-Hooks): ereignisgesteuerte Skripte für Befehle und Lebenszyklusereignisse.
* **Plugin-Hooks**: Erweiterungspunkte innerhalb des Agent-/Tool-Lebenszyklus und der Gateway-Pipeline.

<div id="internal-hooks-gateway-hooks">
  ### Interne Hooks (Gateway-Hooks)
</div>

* **`agent:bootstrap`**: wird ausgeführt, während Bootstrap-Dateien generiert werden, bevor der System-Prompt finalisiert ist.
  Verwenden Sie dies, um Bootstrap-Kontextdateien hinzuzufügen oder zu entfernen.
* **Command-Hooks**: `/new`, `/reset`, `/stop` und andere Command-Ereignisse (siehe Hooks-Dokumentation).

Siehe [Hooks](/de/hooks) für Einrichtung und Beispiele.

<div id="plugin-hooks-agent-gateway-lifecycle">
  ### Plugin-Hooks (Agent- und Gateway-Lebenszyklus)
</div>

Diese laufen innerhalb der Agent-Schleife oder der Gateway-Pipeline:

* **`before_agent_start`**: Kontext injizieren oder System-Prompt überschreiben, bevor die Ausführung startet.
* **`agent_end`**: die finale Nachrichtenliste und Metadaten der Ausführung nach Abschluss inspizieren.
* **`before_compaction` / `after_compaction`**: Kompaktierungszyklen beobachten oder annotieren.
* **`before_tool_call` / `after_tool_call`**: Tool-Parameter und -Ergebnisse abfangen.
* **`tool_result_persist`**: Tool-Ergebnisse synchron transformieren, bevor sie im Sitzungsprotokoll protokolliert werden.
* **`message_received` / `message_sending` / `message_sent`**: Hooks für eingehende und ausgehende Nachrichten.
* **`session_start` / `session_end`**: Grenzen des Sitzungslebenszyklus.
* **`gateway_start` / `gateway_stop`**: Gateway-Lebenszyklusereignisse.

Siehe [Plugins](/de/plugin#plugin-hooks) für die Hook-API und Details zur Registrierung.

<div id="streaming-partial-replies">
  ## Streaming + partielle Antworten
</div>

* Assistant-Deltas werden von pi-agent-core gestreamt und als `assistant`-Events emittiert.
* Block-Streaming kann partielle Antworten entweder bei `text_end` oder `message_end` ausgeben.
* Reasoning-Streaming kann entweder als separater Stream oder in Form von Blockantworten erfolgen.
* Siehe [Streaming](/de/concepts/streaming) für das Chunking- und Blockantwort-Verhalten.

<div id="tool-execution-messaging-tools">
  ## Toolausführung + Messaging-Tools
</div>

* Tool-Start-, -Update- und -End-Events werden auf dem `tool`-Stream ausgegeben.
* Tool-Ergebnisse werden vor der Protokollierung/Ausgabe im Hinblick auf Größe und Bild-Payloads bereinigt.
* Sendevorgänge (Senden) von Messaging-Tools werden nachverfolgt, um doppelte Bestätigungen des Assistenten zu unterdrücken.

<div id="reply-shaping-suppression">
  ## Antwortgestaltung + Unterdrückung
</div>

* Finale Payloads werden zusammengesetzt aus:
  * Assistant-Text (und optionaler Begründung)
  * Inline-Tool-Zusammenfassungen (wenn ausführlich + erlaubt)
  * Assistant-Fehlertext, wenn das Modell einen Fehler erzeugt
* `NO_REPLY` wird als stummes Token behandelt und aus ausgehenden Payloads herausgefiltert.
* Duplikate von Messaging-Tools werden aus der finalen Payload-Liste entfernt.
* Wenn keine darstellbaren Payloads verbleiben und ein Tool einen Fehler ausgelöst hat, wird eine Fallback-Tool-Fehlerantwort ausgegeben
  (es sei denn, ein Messaging-Tool hat bereits eine für den Benutzer sichtbare Antwort gesendet).

<div id="compaction-retries">
  ## Compaction + Retries
</div>

* Die automatische Compaction sendet `compaction`-Stream-Events und kann einen Retry auslösen.
* Bei einem Retry werden In-Memory-Puffer und Tool-Zusammenfassungen zurückgesetzt, um doppelte Ausgaben zu vermeiden.
* Siehe [Compaction](/de/concepts/compaction) für die Compaction-Pipeline.

<div id="event-streams-today">
  ## Ereignisstreams (derzeit)
</div>

* `lifecycle`: ausgegeben von `subscribeEmbeddedPiSession` (und als Fallback von `agentCommand`)
* `assistant`: gestreamte Delta-Updates von pi-agent-core
* `tool`: gestreamte Tool-Ereignisse von pi-agent-core

<div id="chat-channel-handling">
  ## Verarbeitung von Chat-Kanälen
</div>

* Assistant-Deltas werden in Chat-`delta`-Nachrichten zwischengespeichert.
* Ein Chat-`final` wird am **Ende des Lifecycles oder bei einem Fehler** ausgegeben.

<div id="timeouts">
  ## Timeouts
</div>

* Standardwert für `agent.wait`: 30s (nur die Wartezeit). Der `timeoutMs`‑Parameter überschreibt diesen Wert.
* Agent-Laufzeit: Standardwert für `agents.defaults.timeoutSeconds` ist 600s; wird durch den Abbruch-Timer in `runEmbeddedPiAgent` durchgesetzt.

<div id="where-things-can-end-early">
  ## Wo der Ablauf frühzeitig enden kann
</div>

* Agent-Timeout (Abbruch)
* AbortSignal (Abbruch)
* Gateway-Trennung oder RPC-Timeout
* `agent.wait`-Timeout (nur Warten, beendet den agent nicht)