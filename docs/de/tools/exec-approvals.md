---
title: Exec-Freigaben
summary: "Exec-Freigaben, Allowlists und Prompts zu sandbox-Escapes"
read_when:
  - Konfiguration von Exec-Freigaben oder Allowlists
  - Implementierung der Exec-Freigabe-UX in der macOS-App
  - Überprüfung von Prompts zu sandbox-Escapes und deren Auswirkungen
---

<div id="exec-approvals">
  # Exec-Freigaben
</div>

Exec-Freigaben sind der **Schutzmechanismus der Companion-App / des Knoten-Hosts**, der es einem in einer sandbox ausgeführten agent erlaubt,
Befehle auf einem echten Host (`gateway` oder `node`) auszuführen. Du kannst sie dir wie eine Sicherheitsverriegelung vorstellen:
Befehle sind nur erlaubt, wenn Richtlinie + Allowlist + (optionale) Benutzerbestätigung übereinstimmen.
Exec-Freigaben gelten **zusätzlich** zur Tool-Richtlinie und zum erhöhten Gating (es sei denn, `elevated` ist auf `full` gesetzt, was Freigaben überspringt).
Die wirksame Richtlinie ist die **strengere** von `tools.exec.*` und den Standardwerten für Freigaben; wenn ein Freigabefeld weggelassen wird, wird der `tools.exec`-Wert verwendet.

Wenn die Companion-App-UI **nicht verfügbar** ist, wird jede Anforderung, die eine Rückfrage erfordert,
über den **ask-Fallback** entschieden (Standard: verweigern).

<div id="where-it-applies">
  ## Wo sie gelten
</div>

Exec-Bestätigungen werden lokal auf dem Ausführungs-Host erzwungen:

* **Gateway-Host** → `openclaw`-Prozess auf der Gateway-Maschine
* **Knoten-Host** → Node-Runner (macOS-Companion-App oder Headless-Knoten-Host)

macOS-Split:

* **Knoten-Host-Dienst** leitet `system.run` per lokalem IPC an die **macOS-App** weiter.
* **macOS-App** erzwingt die Bestätigungen und führt den Befehl im UI-Kontext aus.

<div id="settings-and-storage">
  ## Einstellungen und Speicherung
</div>

Freigaben werden in einer lokalen JSON-Datei auf dem Ausführungshost gespeichert:

`~/.openclaw/exec-approvals.json`

Beispielschema:

```json
{
  "version": 1,
  "socket": {
    "path": "~/.openclaw/exec-approvals.sock",
    "token": "base64url-token"
  },
  "defaults": {
    "security": "deny",
    "ask": "on-miss",
    "askFallback": "deny",
    "autoAllowSkills": false
  },
  "agents": {
    "main": {
      "security": "allowlist",
      "ask": "on-miss",
      "askFallback": "deny",
      "autoAllowSkills": true,
      "allowlist": [
        {
          "id": "B0C8C0B3-2C2D-4F8A-9A3C-5A4B3C2D1E0F",
          "pattern": "~/Projects/**/bin/rg",
          "lastUsedAt": 1737150000000,
          "lastUsedCommand": "rg -n TODO",
          "lastResolvedPath": "/Users/user/Projects/.../bin/rg"
        }
      ]
    }
  }
}
```

<div id="policy-knobs">
  ## Richtlinien-Parameter
</div>

<div id="security-execsecurity">
  ### Sicherheit (`exec.security`)
</div>

* **deny**: blockiert alle Exec-Anfragen auf dem Host.
* **allowlist**: erlaubt nur Befehle aus der Allowlist.
* **full**: erlaubt alles (entspricht elevated).

<div id="ask-execask">
  ### Ask (`exec.ask`)
</div>

* **off**: niemals nach Bestätigung fragen.
* **on-miss**: nur nach Bestätigung fragen, wenn die Allowlist nicht greift.
* **always**: bei jedem Befehl nach Bestätigung fragen.

<div id="ask-fallback-askfallback">
  ### Ask-Fallback (`askFallback`)
</div>

Wenn ein Prompt erforderlich ist, aber keine UI erreichbar ist, entscheidet der Fallback:

* **deny**: blockieren.
* **allowlist**: nur zulassen, wenn die Allowlist passt.
* **full**: zulassen.

<div id="allowlist-per-agent">
  ## Allowlist (pro Agent)
</div>

Allowlists sind **pro Agent** definiert. Wenn mehrere Agenten existieren, wechsle in der macOS-App zu dem Agent,
den du bearbeitest. Patterns sind **Groß-/Kleinschreibung ignorierende Glob-Muster**.
Patterns sollten auf **Binärdateipfade** aufgelöst werden (Einträge nur mit Basename werden ignoriert).
Veraltete `agents.default`-Einträge werden beim Laden nach `agents.main` migriert.

Beispiele:

* `~/Projects/**/bin/bird`
* `~/.local/bin/*`
* `/opt/homebrew/bin/rg`

Jeder Allowlist-Eintrag erfasst:

* **id** stabile UUID, die für die UI-Identität verwendet wird (optional)
* **zuletzt verwendet** Zeitstempel
* **zuletzt verwendeter Befehl**
* **zuletzt aufgelöster Pfad**

<div id="auto-allow-skill-clis">
  ## Fähigkeiten-CLIs automatisch zulassen
</div>

Wenn **Fähigkeiten-CLIs automatisch zulassen** aktiviert ist, werden von bekannten Fähigkeiten referenzierte ausführbare Programme auf Knoten (macOS-Knoten oder Headless-Knoten-Host) so behandelt, als stünden sie auf der Allowlist. Dabei wird `skills.bins` über das Gateway-RPC verwendet, um die Liste der Fähigkeits-Binaries abzurufen. Deaktiviere diese Option, wenn du strikte manuelle Allowlists verwenden möchtest.

<div id="safe-bins-stdin-only">
  ## Sichere Binaries (nur stdin)
</div>

`tools.exec.safeBins` definiert eine kleine Liste von **nur-stdin**-Binaries (zum Beispiel `jq`),
die im Allowlist-Modus **ohne** explizite Allowlist-Einträge ausgeführt werden können. Sichere
Binaries lehnen positionale Dateiargumente und pfadähnliche Token ab, sodass sie nur auf
den eingehenden Stream wirken können. Shell-Chaining und Umleitungen werden im Allowlist-Modus
nicht automatisch erlaubt.

Shell-Chaining (`&&`, `||`, `;`) ist erlaubt, wenn jedes Top-Level-Segment die Allowlist erfüllt
(einschließlich sicherer Binaries oder Skill-Auto-Allow). Umleitungen bleiben im Allowlist-Modus
nicht unterstützt.

Standardmäßig sichere Binaries: `jq`, `grep`, `cut`, `sort`, `uniq`, `head`, `tail`, `tr`, `wc`.

<div id="control-ui-editing">
  ## Bearbeitung in der Control UI
</div>

Verwende die Karte **Control UI → Nodes → Exec approvals**, um Standardwerte, agent‑spezifische
Overrides und Allowlist‑Einträge zu bearbeiten. Wähle einen Scope (Defaults oder einen Agent), passe die Richtlinie an,
füge Allowlist‑Muster hinzu oder entferne sie und klicke dann auf **Save**. Die UI zeigt pro Muster
„last used“-Metadaten an, damit du die Liste übersichtlich halten kannst.

Der Ziel‑Selektor wählt **Gateway** (lokale Freigaben) oder einen **Node**. Nodes
müssen `system.execApprovals.get/set` bekanntgeben (macOS‑App oder Headless‑Node‑Host).
Falls ein Node noch keine exec approvals bekanntgibt, bearbeite seine lokale
`~/.openclaw/exec-approvals.json` direkt.

CLI: `openclaw approvals` unterstützt die Bearbeitung am Gateway oder an einem Node (siehe [Approvals CLI](/de/cli/approvals)).

<div id="approval-flow">
  ## Genehmigungsablauf
</div>

Wenn eine Bestätigung erforderlich ist, sendet das Gateway `exec.approval.requested` an Operator-Clients.
Die Control UI und die macOS-App lösen dies über `exec.approval.resolve`, danach leitet das Gateway die
genehmigte Anforderung an den Knoten-Host weiter.

Wenn Genehmigungen erforderlich sind, gibt das `exec`-Tool sofort eine Genehmigungs-ID zurück. Verwende diese ID,
um spätere Systemereignisse zu korrelieren (`Exec finished` / `Exec denied`). Wenn vor Ablauf des Timeouts keine
Entscheidung eintrifft, wird die Anforderung als Genehmigungs-Timeout behandelt und als Ablehnungsgrund ausgegeben.

Der Bestätigungsdialog enthält:

* command + args
* cwd
* Agent-ID
* aufgelösten ausführbaren Pfad
* Host- und Richtlinien-Metadaten

Aktionen:

* **Allow once** → jetzt ausführen
* **Always allow** → zur Allowlist hinzufügen + ausführen
* **Deny** → blockieren

<div id="approval-forwarding-to-chat-channels">
  ## Weiterleitung von Genehmigungen an Chat-Kanäle
</div>

Du kannst Exec-Genehmigungsanfragen an jeden Chat-Kanal (einschließlich Plugin-Kanälen) weiterleiten und sie dort mit `/approve` freigeben. Dabei wird die normale Outbound-Delivery-Pipeline verwendet.

Konfiguration:

```json5
{
  approvals: {
    exec: {
      enabled: true,
      mode: "session", // "Sitzung" | "Ziele" | "beides"
      agentFilter: ["main"],
      sessionFilter: ["discord"], // substring or regex
      targets: [
        { channel: "slack", to: "U12345678" },
        { channel: "telegram", to: "123456789" }
      ]
    }
  }
}
```

Im Chat antworten:

```
/approve <id> allow-once
/approve <id> allow-always
/approve <id> deny
```

<div id="macos-ipc-flow">
  ### Ablauf der macOS-IPC
</div>

```
Gateway -> Node Service (WS)
                 |  IPC (UDS + token + HMAC + TTL)
                 v
             Mac App (UI + approvals + system.run)
```

Sicherheitshinweise:

* Unix-Socket-Modus `0600`, Token wird in `exec-approvals.json` gespeichert.
* Peer-Prüfung auf gleiche UID.
* Challenge/Response (Nonce + HMAC-Token + Request-Hash) mit kurzer TTL.

<div id="system-events">
  ## Systemereignisse
</div>

Der Exec-Lebenszyklus wird als Systemnachrichten dargestellt:

* `Exec running` (nur wenn der Befehl die Schwelle für die Laufzeitbenachrichtigung überschreitet)
* `Exec finished`
* `Exec denied`

Diese werden in die Sitzung des agents gepostet, nachdem der knoten das Ereignis gemeldet hat.
Exec-Genehmigungen auf dem Gateway-Host erzeugen dieselben Lebenszyklusereignisse, wenn der Befehl abgeschlossen ist (und optional, wenn er länger als die Schwelle ausgeführt wird).
Execs mit Genehmigungspflicht verwenden die Genehmigungs-ID erneut als `runId` in diesen Nachrichten, um eine einfache Korrelation zu ermöglichen.

<div id="implications">
  ## Auswirkungen
</div>

* **full** ist mächtig; verwende nach Möglichkeit Allowlists.
* **ask** hält dich auf dem Laufenden und ermöglicht trotzdem schnelle Genehmigungen.
* Allowlists pro agent verhindern, dass die Genehmigungen eines agents auf andere agents übergreifen.
* Genehmigungen gelten nur für Host-Exec-Anfragen von **autorisierten Absendern**. Nicht autorisierte Absender können kein `/exec` ausführen.
* `/exec security=full` ist eine komfortable Einstellung auf Sitzungsebene für autorisierte Operatoren und überspringt Genehmigungen absichtlich.
  Um Host-Exec strikt zu blockieren, setze die Sicherheitsstufe für Genehmigungen auf `deny` oder verweigere das `exec`-Tool über die Tool-Richtlinie.

Verwandt:

* [Exec-Tool](/de/tools/exec)
* [Erhöhter Modus](/de/tools/elevated)
* [Fähigkeiten](/de/tools/skills)