---
summary: "Sandbox- und Tool-Einschränkungen pro Agent, Reihenfolge und Beispiele"
title: Multi-Agent-Sandbox &amp; Tools
read_when: "Du möchtest Sandboxing pro Agent oder Allow-/Deny-Richtlinien für Tools pro Agent in einem Multi-Agent-Gateway einrichten."
status: active
---

<div id="multi-agent-sandbox-tools-configuration">
  # Multi-Agent-Sandbox- &amp; Tools-Konfiguration
</div>

<div id="overview">
  ## Überblick
</div>

Jeder Agent in einem Multi-Agent-Setup kann nun Folgendes eigenständig konfigurieren:

* **Sandbox-Konfiguration** (`agents.list[].sandbox` überschreibt `agents.defaults.sandbox`)
* **Tool-Einschränkungen** (`tools.allow` / `tools.deny` sowie `agents.list[].tools`)

Dadurch kannst du mehrere Agenten mit unterschiedlichen Sicherheitsprofilen betreiben:

* Persönlicher Assistent mit vollem Zugriff
* Familien-/Arbeitsagenten mit eingeschränkten Tools
* Öffentlich erreichbare Agenten in Sandboxes

`setupCommand` gehört unter `sandbox.docker` (global oder pro Agent) und wird einmalig
ausgeführt, wenn der Container erstellt wird.

Die Authentifizierung erfolgt pro Agent: Jeder Agent liest aus seinem eigenen `agentDir`-Auth-Store unter:

```
~/.openclaw/agents/<agentId>/agent/auth-profiles.json
```

Anmeldedaten werden **nicht** zwischen Agenten geteilt. Verwende niemals dasselbe `agentDir` für mehrere Agenten wieder.

Wenn du Anmeldedaten teilen willst, kopiere `auth-profiles.json` in das `agentDir` des anderen Agenten.

Informationen zum Sandbox-Verhalten zur Laufzeit findest du unter [Sandboxing](/de/gateway/sandboxing).
Zum Debuggen von „Warum ist das blockiert?“ siehe [Sandbox vs Tool Policy vs Elevated](/de/gateway/sandbox-vs-tool-policy-vs-elevated) und `openclaw sandbox explain`.

***

<div id="configuration-examples">
  ## Konfigurationsbeispiele
</div>

<div id="example-1-personal-restricted-family-agent">
  ### Beispiel 1: Persönlicher + Eingeschränkter Familienagent
</div>

```json
{
  "agents": {
    "list": [
      {
        "id": "main",
        "default": true,
        "name": "Personal Assistant",
        "workspace": "~/.openclaw/workspace",
        "sandbox": { "mode": "off" }
      },
      {
        "id": "family",
        "name": "Family Bot",
        "workspace": "~/.openclaw/workspace-family",
        "sandbox": {
          "mode": "all",
          "scope": "agent"
        },
        "tools": {
          "allow": ["read"],
          "deny": ["exec", "write", "edit", "apply_patch", "process", "browser"]
        }
      }
    ]
  },
  "bindings": [
    {
      "agentId": "family",
      "match": {
        "provider": "whatsapp",
        "accountId": "*",
        "peer": {
          "kind": "group",
          "id": "120363424282127706@g.us"
        }
      }
    }
  ]
}
```

**Ergebnis:**

* `main`-Agent: Läuft auf dem Host, vollständiger Zugriff auf Tools
* `family`-Agent: Läuft in Docker (ein Container pro agent), nur `read`-Tool

***

<div id="example-2-work-agent-with-shared-sandbox">
  ### Beispiel 2: Arbeitsagent mit geteilter Sandbox
</div>

```json
{
  "agents": {
    "list": [
      {
        "id": "personal",
        "workspace": "~/.openclaw/workspace-personal",
        "sandbox": { "mode": "off" }
      },
      {
        "id": "work",
        "workspace": "~/.openclaw/workspace-work",
        "sandbox": {
          "mode": "all",
          "scope": "shared",
          "workspaceRoot": "/tmp/work-sandboxes"
        },
        "tools": {
          "allow": ["read", "write", "apply_patch", "exec"],
          "deny": ["browser", "gateway", "discord"]
        }
      }
    ]
  }
}
```

***

<div id="example-2b-global-coding-profile-messaging-only-agent">
  ### Beispiel 2b: Globales Coding-Profil + reiner Messaging-agent
</div>

```json
{
  "tools": { "profile": "coding" },
  "agents": {
    "list": [
      {
        "id": "support",
        "tools": { "profile": "messaging", "allow": ["slack"] }
      }
    ]
  }
}
```

**Ergebnis:**

* Standard-Agenten erhalten Coding-Tools
* `support`-Agent ist reines Messaging (+ Slack-Tool)

***

<div id="example-3-different-sandbox-modes-per-agent">
  ### Beispiel 3: Verschiedene Sandbox-Modi je Agent
</div>

```json
{
  "agents": {
    "defaults": {
      "sandbox": {
        "mode": "non-main",  // Global default
        "scope": "session"
      }
    },
    "list": [
      {
        "id": "main",
        "workspace": "~/.openclaw/workspace",
        "sandbox": {
          "mode": "off"  // Override: main never sandboxed
        }
      },
      {
        "id": "public",
        "workspace": "~/.openclaw/workspace-public",
        "sandbox": {
          "mode": "all",  // Überschreibung: public wird immer in der sandbox ausgeführt
          "scope": "agent"
        },
        "tools": {
          "allow": ["read"],
          "deny": ["exec", "write", "edit", "apply_patch"]
        }
      }
    ]
  }
}
```

***

<div id="configuration-precedence">
  ## Konfigurationsvorrang
</div>

Wenn es sowohl globale (`agents.defaults.*`) als auch agentenspezifische (`agents.list[].*`) Konfigurationen gibt:

<div id="sandbox-config">
  ### Sandbox-Konfiguration
</div>

Agent-spezifische Einstellungen überschreiben globale Einstellungen:

```
agents.list[].sandbox.mode > agents.defaults.sandbox.mode
agents.list[].sandbox.scope > agents.defaults.sandbox.scope
agents.list[].sandbox.workspaceRoot > agents.defaults.sandbox.workspaceRoot
agents.list[].sandbox.workspaceAccess > agents.defaults.sandbox.workspaceAccess
agents.list[].sandbox.docker.* > agents.defaults.sandbox.docker.*
agents.list[].sandbox.browser.* > agents.defaults.sandbox.browser.*
agents.list[].sandbox.prune.* > agents.defaults.sandbox.prune.*
```

**Hinweise:**

* `agents.list[].sandbox.{docker,browser,prune}.*` überschreibt `agents.defaults.sandbox.{docker,browser,prune}.*` für diesen Agenten (wird ignoriert, wenn der Sandbox-Scope als „shared“ aufgelöst wird).

<div id="tool-restrictions">
  ### Tool-Einschränkungen
</div>

Die Reihenfolge der Filterung ist:

1. **Tool-Profil** (`tools.profile` oder `agents.list[].tools.profile`)
2. **Anbieter-Tool-Profil** (`tools.byProvider[provider].profile` oder `agents.list[].tools.byProvider[provider].profile`)
3. **Globale Tool-Richtlinie** (`tools.allow` / `tools.deny`)
4. **Anbieter-Tool-Richtlinie** (`tools.byProvider[provider].allow/deny`)
5. **Agent-spezifische Tool-Richtlinie** (`agents.list[].tools.allow/deny`)
6. **Agent-Anbieter-Richtlinie** (`agents.list[].tools.byProvider[provider].allow/deny`)
7. **Sandbox-Tool-Richtlinie** (`tools.sandbox.tools` oder `agents.list[].tools.sandbox.tools`)
8. **Subagent-Tool-Richtlinie** (`tools.subagents.tools`, falls zutreffend)

Jede Ebene kann Tools weiter einschränken, aber keine zuvor verbotenen Tools aus früheren Ebenen wieder zulassen.
Wenn `agents.list[].tools.sandbox.tools` gesetzt ist, ersetzt dies `tools.sandbox.tools` für diesen Agenten.
Wenn `agents.list[].tools.profile` gesetzt ist, überschreibt dies `tools.profile` für diesen Agenten.
Anbieter-Tool-Schlüssel akzeptieren entweder `provider` (z. B. `google-antigravity`) oder `provider/model` (z. B. `openai/gpt-5.2`).

<div id="tool-groups-shorthands">
  ### Toolgruppen (Kurzformen)
</div>

Tool-Richtlinien (global, agent, sandbox) unterstützen `group:*`-Einträge, die zu mehreren konkreten Tools aufgelöst werden:

* `group:runtime`: `exec`, `bash`, `process`
* `group:fs`: `read`, `write`, `edit`, `apply_patch`
* `group:sessions`: `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status`
* `group:memory`: `memory_search`, `memory_get`
* `group:ui`: `browser`, `canvas`
* `group:automation`: `cron`, `gateway`
* `group:messaging`: `message`
* `group:nodes`: `nodes`
* `group:openclaw`: alle integrierten OpenClaw-Tools (schließt Anbieter-Plugins aus)

<div id="elevated-mode">
  ### Erhöhter Modus
</div>

`tools.elevated` ist die globale Basiseinstellung (senderbasierte Allowlist). `agents.list[].tools.elevated` kann den erhöhten Modus für bestimmte Agenten weiter einschränken (beide müssen zulassen).

Muster zur Risikominimierung:

* `exec` für nicht vertrauenswürdige Agenten verweigern (`agents.list[].tools.deny: ["exec"]`)
* Vermeide es, Sender in die Allowlist aufzunehmen, die zu eingeschränkten Agenten geroutet werden
* Erhöhten Modus global deaktivieren (`tools.elevated.enabled: false`), wenn du nur Ausführung in der Sandbox möchtest
* Erhöhten Modus pro Agent deaktivieren (`agents.list[].tools.elevated.enabled: false`) für sensible Profile

***

<div id="migration-from-single-agent">
  ## Migration von einem einzelnen Agent
</div>

**Vorher (einzelner Agent):**

```json
{
  "agents": {
    "defaults": {
      "workspace": "~/.openclaw/workspace",
      "sandbox": {
        "mode": "non-main"
      }
    }
  },
  "tools": {
    "sandbox": {
      "tools": {
        "allow": ["read", "write", "apply_patch", "exec"],
        "deny": []
      }
    }
  }
}
```

**Nachher (Multi-Agent mit verschiedenen Profilen):**

```json
{
  "agents": {
    "list": [
      {
        "id": "main",
        "default": true,
        "workspace": "~/.openclaw/workspace",
        "sandbox": { "mode": "off" }
      }
    ]
  }
}
```

Veraltete `agent.*`-Konfigurationen werden von `openclaw doctor` migriert; künftig solltest du `agents.defaults` und `agents.list` bevorzugen.

***

<div id="tool-restriction-examples">
  ## Beispiele für Tool-Einschränkungen
</div>

<div id="read-only-agent">
  ### Nur-Lese-Agent
</div>

```json
{
  "tools": {
    "allow": ["read"],
    "deny": ["exec", "write", "edit", "apply_patch", "process"]
  }
}
```

<div id="safe-execution-agent-no-file-modifications">
  ### Agent für sichere Ausführung (ohne Dateiänderungen)
</div>

```json
{
  "tools": {
    "allow": ["read", "exec", "process"],
    "deny": ["write", "edit", "apply_patch", "browser", "gateway"]
  }
}
```

<div id="communication-only-agent">
  ### Reiner Kommunikationsagent
</div>

```json
{
  "tools": {
    "allow": ["sessions_list", "sessions_send", "sessions_history", "session_status"],
    "deny": ["exec", "write", "edit", "apply_patch", "read", "browser"]
  }
}
```

***

<div id="common-pitfall-non-main">
  ## Häufige Fehlerquelle: &quot;non-main&quot;
</div>

`agents.defaults.sandbox.mode: "non-main"` basiert auf `session.mainKey` (Standardwert `"main"`),
nicht auf der Agent-ID. Sitzungen für Gruppen/Kanäle erhalten immer eigene Keys und
werden daher als „non-main“ behandelt und in der Sandbox ausgeführt. Wenn du möchtest, dass ein Agent niemals
in der Sandbox läuft, setze `agents.list[].sandbox.mode: "off"`.

***

<div id="testing">
  ## Tests
</div>

Nach der Konfiguration der Multi-Agent-Sandbox und der Tools:

1. **Agent-Auflösung prüfen:**
   ```exec
   openclaw agents list --bindings
   ```

2. **Sandbox-Container überprüfen:**
   ```exec
   docker ps --filter "name=openclaw-sbx-"
   ```

3. **Tool-Einschränkungen testen:**
   * Eine Nachricht senden, die eingeschränkte Tools erfordert
   * Prüfen, dass der Agent verweigerte Tools nicht verwenden kann

4. **Logs überwachen:**
   ```exec
   tail -f "${OPENCLAW_STATE_DIR:-$HOME/.openclaw}/logs/gateway.log" | grep -E "routing|sandbox|tools"
   ```

***

<div id="troubleshooting">
  ## Fehlerbehebung
</div>

<div id="agent-not-sandboxed-despite-mode-all">
  ### Agent wird trotz `mode: "all"` nicht in sandbox ausgeführt
</div>

* Überprüfe, ob es eine globale `agents.defaults.sandbox.mode` gibt, die dies überschreibt
* Agent-spezifische Konfiguration hat Vorrang, setze daher `agents.list[].sandbox.mode: "all"`

<div id="tools-still-available-despite-deny-list">
  ### Tools trotz Deny-Liste weiterhin verfügbar
</div>

* Überprüfe die Reihenfolge der Tool-Filterung: global → agent → sandbox → subagent
* Jede Ebene kann nur weiter einschränken, nicht wieder freigeben
* Überprüfe in den Logs: `[tools] filtering tools for agent:${agentId}`

<div id="container-not-isolated-per-agent">
  ### Container nicht pro Agent isoliert
</div>

* Setze `scope: "agent"` in der agentspezifischen Sandbox-Konfiguration
* Der Standardwert ist `"session"`, wodurch ein Container pro Sitzung erstellt wird

***

<div id="see-also">
  ## Siehe auch
</div>

* [Multi-Agent-Routing](/de/concepts/multi-agent)
* [Sandbox-Konfiguration](/de/gateway/configuration#agentsdefaults-sandbox)
* [Sitzungsverwaltung](/de/concepts/session)