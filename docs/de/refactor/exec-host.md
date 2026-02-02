---
title: Exec-Host
summary: "Refactoring-Plan: Exec-Host-Routing, Node-Freigaben und Headless-Runner"
read_when:
  - Beim Entwurf von Exec-Host-Routing oder Exec-Host-Freigaben
  - Bei der Implementierung eines Node-Runners + UI-IPC
  - Beim Hinzufügen von Exec-Host-Sicherheitsmodi und Slash-Commands
---

<div id="exec-host-refactor-plan">
  # Refactoring-Plan für den Exec-Host
</div>

<div id="goals">
  ## Ziele
</div>

* `exec.host` + `exec.security` hinzufügen, um die Ausführung über **Sandbox**, **Gateway** und **Knoten** zu routen.
* Standardwerte **sicher** halten: keine Host-übergreifende Ausführung, es sei denn, sie wird explizit aktiviert.
* Ausführung in einen **Headless-Runner-Dienst** mit optionaler UI (macOS-App) über lokale IPC aufteilen.
* **Richtlinien pro agent**, Allowlist, Ask-Modus und Knotenbindung bereitstellen.
* **Ask-Modi** unterstützen, die *mit* oder *ohne* Allowlist funktionieren.
* Plattformübergreifend: Unix-Socket + Token-Authentifizierung (gleiche Unterstützung für macOS/Linux/Windows).

<div id="non-goals">
  ## Nicht-Ziele
</div>

* Keine Migration von Legacy-Allowlists und keine Unterstützung für Legacy-Schemas.
* Kein PTY-/Streaming für node exec (nur aggregierte Ausgabe).
* Keine neue Netzwerkschicht über die bestehende Bridge + Gateway hinaus.

<div id="decisions-locked">
  ## Entscheidungen (festgelegt)
</div>

* **Config-Schlüssel:** `exec.host` + `exec.security` (pro-Agent-Override erlaubt).
* **Elevation:** `/elevated` als Alias für vollen Gateway-Zugriff beibehalten.
* **Standard für Nachfragen:** `on-miss`.
* **Genehmigungsspeicher:** `~/.openclaw/exec-approvals.json` (JSON, keine Legacy-Migration).
* **Runner:** headless Systemdienst; UI-App stellt einen Unix-Socket für Genehmigungen bereit.
* **Knotenidentität:** vorhandene `nodeId` verwenden.
* **Socket-Authentifizierung:** Unix-Socket + Token (plattformübergreifend); bei Bedarf später aufteilen.
* **Knoten-Hostzustand:** `~/.openclaw/node.json` (Knoten-ID + Kopplungs-Token).
* **macOS-Exec-Host:** `system.run` innerhalb der macOS-App ausführen; Knoten-Hostdienst leitet Anfragen über lokale IPC weiter.
* **Kein XPC-Helper:** bei Unix-Socket + Token + Peer-Checks bleiben.

<div id="key-concepts">
  ## Kernkonzepte
</div>

<div id="host">
  ### Host
</div>

* `sandbox`: Docker exec (aktuelles Verhalten).
* `gateway`: exec auf dem Gateway-Host.
* `node`: exec auf dem Knoten-Runner über die Bridge (`system.run`).

<div id="security-mode">
  ### Sicherheitsmodus
</div>

* `deny`: immer blockieren.
* `allowlist`: nur passende Einträge erlauben.
* `full`: alles erlauben (entspricht `elevated`).

<div id="ask-mode">
  ### Anfrage-Modus
</div>

* `off`: niemals nachfragen.
* `on-miss`: nur nachfragen, wenn die Allowlist nicht passt.
* `always`: jedes Mal nachfragen.

Anfragen sind **unabhängig** von der Allowlist; die Allowlist kann mit `always` oder `on-miss` verwendet werden.

<div id="policy-resolution-per-exec">
  ### Richtlinienauflösung (pro Ausführungsvorgang)
</div>

1. Ermittle `exec.host` (Tool-Parameter → Agent-Override → globaler Standard).
2. Ermittle `exec.security` und `exec.ask` (gleiche Priorität).
3. Wenn der Host `sandbox` ist, mit lokaler Ausführung in der sandbox fortfahren.
4. Wenn der Host `gateway` oder `node` ist, Sicherheits- und Ask-Richtlinie auf diesem Host anwenden.

<div id="default-safety">
  ## Standard-Sicherheit
</div>

* Standardmäßig `exec.host = sandbox`.
* Standardmäßig `exec.security = deny` für `Gateway` und `Knoten`.
* Standardmäßig `exec.ask = on-miss` (nur relevant, wenn die Sicherheitsrichtlinie dies zulässt).
* Wenn keine Knoten-Bindung gesetzt ist, **kann der agent jeden Knoten ansteuern**, aber nur, wenn die Richtlinie dies erlaubt.

<div id="config-surface">
  ## Konfigurationsumfang
</div>

<div id="tool-parameters">
  ### Toolparameter
</div>

* `exec.host` (optional): `sandbox | gateway | node`.
* `exec.security` (optional): `deny | allowlist | full`.
* `exec.ask` (optional): `off | on-miss | always`.
* `exec.node` (optional): Knoten-ID/-Name, der verwendet werden soll, wenn `host=node`.

<div id="config-keys-global">
  ### Globale Konfigurationsschlüssel
</div>

* `tools.exec.host`
* `tools.exec.security`
* `tools.exec.ask`
* `tools.exec.node` (Standard-Knotenbindung)

<div id="config-keys-per-agent">
  ### Konfigurationsschlüssel (je agent)
</div>

* `agents.list[].tools.exec.host`
* `agents.list[].tools.exec.security`
* `agents.list[].tools.exec.ask`
* `agents.list[].tools.exec.node`

<div id="alias">
  ### Alias
</div>

* `/elevated on` = setzt `tools.exec.host=gateway`, `tools.exec.security=full` für die Agent-Sitzung.
* `/elevated off` = stellt die vorherigen Exec-Einstellungen für die Agent-Sitzung wieder her.

<div id="approvals-store-json">
  ## Approvals-Speicher (JSON)
</div>

Pfad: `~/.openclaw/exec-approvals.json`

Zweck:

* Lokale Richtlinie + Allowlists für den **Ausführungs-Host** (Gateway oder Knoten-Runner).
* Fallback zum Nachfragen, wenn keine UI verfügbar ist.
* IPC-Zugangsdaten für UI-Clients.

Vorgeschlagenes Schema (v1):

```json
{
  "version": 1,
  "socket": {
    "path": "~/.openclaw/exec-approvals.sock",
    "token": "base64-opaque-token"
  },
  "defaults": {
    "security": "deny",
    "ask": "on-miss",
    "askFallback": "deny"
  },
  "agents": {
    "agent-id-1": {
      "security": "allowlist",
      "ask": "on-miss",
      "allowlist": [
        {
          "pattern": "~/Projects/**/bin/rg",
          "lastUsedAt": 0,
          "lastUsedCommand": "rg -n TODO",
          "lastResolvedPath": "/Users/user/Projects/.../bin/rg"
        }
      ]
    }
  }
}
```

Hinweise:

* Keine veralteten Allowlist-Formate.
* `askFallback` gilt nur, wenn `ask` erforderlich ist und keine UI erreichbar ist.
* Dateiberechtigungen: `0600`.

<div id="runner-service-headless">
  ## Runner-Service (headless)
</div>

<div id="role">
  ### Rolle
</div>

* Erzwingt `exec.security` + `exec.ask` lokal.
* Führt Systembefehle aus und gibt die Ausgabe zurück.
* Löst Bridge-Events für den Exec-Lebenszyklus aus (optional, aber empfohlen).

<div id="service-lifecycle">
  ### Service-Lebenszyklus
</div>

* launchd/Daemon auf macOS; Systemdienst auf Linux/Windows.
* Das Approvals-JSON ist lokal auf dem Ausführungs-Host.
* Die UI stellt einen lokalen Unix-Socket bereit; Runner verbinden sich bei Bedarf.

<div id="ui-integration-macos-app">
  ## UI-Integration (macOS-App)
</div>

<div id="ipc">
  ### IPC
</div>

* Unix-Socket bei `~/.openclaw/exec-approvals.sock` (0600).
* Token wird in `exec-approvals.json` gespeichert (0600).
* Gegenstellenprüfung: nur gleiche UID.
* Challenge/Response: Nonce + HMAC(Token, Request-Hash), um Replay-Angriffe zu verhindern.
* Kurze TTL (z. B. 10 s) + maximale Payload-Größe + Rate-Limit.

<div id="ask-flow-macos-app-exec-host">
  ### Ask-Flow (macOS-App-Exec-Host)
</div>

1. Node-Service empfängt `system.run` vom Gateway.
2. Node-Service verbindet sich mit dem lokalen Socket und sendet die Prompt/Exec-Anfrage.
3. Die App validiert Peer + Token + HMAC + TTL und zeigt bei Bedarf einen Dialog an.
4. Die App führt den Befehl im UI-Kontext aus und gibt die Ausgabe zurück.
5. Der Node-Service gibt die Ausgabe an das Gateway zurück.

Wenn keine UI verfügbar ist:

* Wende `askFallback` an (`deny|allowlist|full`).

<div id="diagram-sci">
  ### Diagramm (SCI)
</div>

```
Agent -> Gateway -> Bridge -> Node Service (TS)
                         |  IPC (UDS + token + HMAC + TTL)
                         v
                     Mac App (UI + TCC + system.run)
```

<div id="node-identity-binding">
  ## Knotenidentität + Bindung
</div>

* Verwende die bestehende `nodeId` aus der Bridge-Kopplung.
* Bindungsmodell:
  * `tools.exec.node` beschränkt den Agent auf einen bestimmten Knoten.
  * Falls nicht gesetzt, kann der Agent jeden Knoten auswählen (die Richtlinie erzwingt weiterhin die Standardwerte).
* Auflösung bei der Knotenauswahl:
  * exakte Übereinstimmung von `nodeId`
  * `displayName` (normalisiert)
  * `remoteIp`
  * `nodeId`-Präfix (&gt;= 6 Zeichen)

<div id="eventing">
  ## Eventing
</div>

<div id="who-sees-events">
  ### Wer sieht Events
</div>

* System-Events sind **pro Sitzung** und werden dem agent beim nächsten Prompt angezeigt.
* In der In-Memory-Warteschlange des Gateways (`enqueueSystemEvent`) gespeichert.

<div id="event-text">
  ### Ereignistext
</div>

* `Exec started (node=<id>, id=<runId>)`
* `Exec finished (node=<id>, id=<runId>, code=<code>)` + optional output tail
* `Exec denied (node=<id>, id=<runId>, <reason>)`

<div id="transport">
  ### Transport
</div>

Option A (empfohlen):

* Runner sendet `event`-Frames `exec.started` / `exec.finished` an die Bridge.
* Gateway `handleBridgeEvent` ordnet diese `enqueueSystemEvent` zu.

Option B:

* Das Gateway-`exec`-Tool übernimmt den Lebenszyklus direkt (nur synchron).

<div id="exec-flows">
  ## Exec-Flows
</div>

<div id="sandbox-host">
  ### Sandbox-Host
</div>

* Bestehendes `exec`-Verhalten (Docker oder Host, wenn ohne sandbox ausgeführt).
* PTY wird nur im Nicht-sandbox-Modus unterstützt.

<div id="gateway-host">
  ### Gateway-Host
</div>

* Der Gateway-Prozess läuft auf einem eigenen Host.
* Erzwingt die lokale `exec-approvals.json` (Sicherheit/Rückfrage/Allowlist).

<div id="node-host">
  ### Knoten-Host
</div>

* Gateway ruft `node.invoke` mit `system.run` auf.
* Runner erzwingt lokale Freigaben.
* Runner gibt zusammengeführtes stdout/stderr zurück.
* Optionale Bridge-Events für Start/Ende/Ablehnung.

<div id="output-caps">
  ## Ausgabe-Limits
</div>

* Begrenze kombiniertes stdout+stderr auf **200k**; behalte die **letzten 20k** für Events.
* Schneide mit einem klaren Suffix ab (z. B. `"… (truncated)"`).

<div id="slash-commands">
  ## Slash-Befehle
</div>

* `/exec host=<sandbox|gateway|node> security=<deny|allowlist|full> ask=<off|on-miss|always> node=<id>`
* Agent- und Sitzungs-spezifische Overrides; nicht persistent, sofern sie nicht über die Konfiguration gespeichert werden.
* `/elevated on|off|ask|full` bleibt eine Abkürzung für `host=gateway security=full` (wobei `full` Genehmigungen überspringt).

<div id="cross-platform-story">
  ## Plattformübergreifendes Konzept
</div>

* Der Runner-Dienst ist das portable Ausführungsziel.
* Die UI ist optional; falls sie fehlt, wird `askFallback` verwendet.
* Windows/Linux unterstützen dasselbe Approvals-JSON- und Socket-Protokoll.

<div id="implementation-phases">
  ## Implementierungsphasen
</div>

<div id="phase-1-config-exec-routing">
  ### Phase 1: Konfiguration + Exec-Routing
</div>

* Konfigurationsschema für `exec.host`, `exec.security`, `exec.ask`, `exec.node` hinzufügen.
* Tool-Anbindung so aktualisieren, dass `exec.host` berücksichtigt wird.
* Slash-Befehl `/exec` hinzufügen und Alias `/elevated` beibehalten.

<div id="phase-2-approvals-store-gateway-enforcement">
  ### Phase 2: Genehmigungsspeicher + Gateway-Durchsetzung
</div>

* `exec-approvals.json` Reader/Writer implementieren.
* Allowlist + Ask-Modi für `gateway`-Host erzwingen.
* Ausgabegrenzen hinzufügen.

<div id="phase-3-node-runner-enforcement">
  ### Phase 3: Erzwingen im Node-Runner
</div>

* Node-Runner aktualisieren, um Allowlist + ask durchzusetzen.
* Unix-Socket-Prompt-Bridge an die macOS-App-UI anbinden.
* `askFallback` anbinden.

<div id="phase-4-events">
  ### Phase 4: Ereignisse
</div>

* Füge Bridge-Events Knoten → Gateway für den Exec-Lebenszyklus hinzu.
* Bilde auf `enqueueSystemEvent` für agent-Prompts ab.

<div id="phase-5-ui-polish">
  ### Phase 5: UI-Feinschliff
</div>

* Mac-App: Allowlist-Editor, agent-spezifischer Umschalter, Ask-Policy-UI.
* Steuerelemente für Knoten-Bindungen (optional).

<div id="testing-plan">
  ## Testplan
</div>

* Unit-Tests: Allowlist-Matching (Glob + ohne Beachtung der Groß-/Kleinschreibung).
* Unit-Tests: Reihenfolge der Richtlinienauflösung (Tool-Parameter → agent-Override → global).
* Integrationstests: Node-Runner-Deny/Allow/Ask-Flows.
* Bridge-Event-Tests: Routing von Node-Events zu System-Events.

<div id="open-risks">
  ## Offene Risiken
</div>

* UI-Nichtverfügbarkeit: Stelle sicher, dass `askFallback` berücksichtigt wird.
* Lange laufende Befehle: Verlass dich auf Timeouts und Ausgabe-Limits.
* Mehrdeutigkeit bei mehreren Knoten: führt zu einem Fehler, außer bei Knotenbindung oder explizitem Knoten-Parameter.

<div id="related-docs">
  ## Verwandte Dokumente
</div>

* [Exec-Tool](/de/tools/exec)
* [Exec-Genehmigungen](/de/tools/exec-approvals)
* [Knoten](/de/nodes)
* [Erhöhter Berechtigungsmodus](/de/tools/elevated)