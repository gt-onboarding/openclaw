---
title: Broadcast-Gruppen
summary: "WhatsApp-Nachrichten an mehrere Agenten senden"
read_when:
  - Broadcast-Gruppen konfigurieren
  - Antworten mehrerer Agenten in WhatsApp debuggen
status: experimental
---

<div id="broadcast-groups">
  # Broadcast-Gruppen
</div>

**Status:** Experimentell\
**Version:** Eingeführt in 2026.1.9

<div id="overview">
  ## Übersicht
</div>

Broadcast Groups ermöglichen es mehreren Agents, dieselbe Nachricht gleichzeitig zu verarbeiten und zu beantworten. Dadurch kannst du spezialisierte Agent-Teams erstellen, die gemeinsam in einer einzelnen WhatsApp-Gruppe oder DM arbeiten – alle über eine einzige Telefonnummer.

Aktueller Umfang: **nur WhatsApp** (Web-Channel).

Broadcast Groups werden nach Channel-Allowlists und Gruppenaktivierungsregeln ausgewertet. In WhatsApp-Gruppen bedeutet das, dass Broadcasts stattfinden, wenn OpenClaw normalerweise antworten würde (zum Beispiel bei Erwähnung, abhängig von deinen Gruppeneinstellungen).

<div id="use-cases">
  ## Anwendungsfälle
</div>

<div id="1-specialized-agent-teams">
  ### 1. Spezialisierte Agent-Teams
</div>

Setze mehrere Agenten mit klar abgegrenzten, fokussierten Verantwortlichkeiten ein:

```
Group: "Development Team"
Agents:
  - CodeReviewer (reviews code snippets)
  - DocumentationBot (generates docs)
  - SecurityAuditor (checks for vulnerabilities)
  - TestGenerator (suggests test cases)
```

Jeder Agent verarbeitet dieselbe Nachricht und bringt seine spezialisierte Perspektive ein.

<div id="2-multi-language-support">
  ### 2. Unterstützung mehrerer Sprachen
</div>

```
Group: "International Support"
Agents:
  - Agent_EN (responds in English)
  - Agent_DE (responds in German)
  - Agent_ES (responds in Spanish)
```

<div id="3-quality-assurance-workflows">
  ### 3. Qualitätssicherungs-Workflows
</div>

```
Group: "Customer Support"
Agents:
  - SupportAgent (provides answer)
  - QAAgent (reviews quality, only responds if issues found)
```

<div id="4-task-automation">
  ### 4. Automatisierung von Aufgaben
</div>

```
Group: "Project Management"
Agents:
  - TaskTracker (updates task database)
  - TimeLogger (logs time spent)
  - ReportGenerator (creates summaries)
```

<div id="configuration">
  ## Konfiguration
</div>

<div id="basic-setup">
  ### Grundlegende Einrichtung
</div>

Füge einen `broadcast`-Abschnitt auf oberster Konfigurationsebene hinzu (neben `bindings`). Die Schlüssel sind WhatsApp-Peer-IDs:

* Gruppenchats: Gruppen-JID (z. B. `120363403215116621@g.us`)
* DMs (Direktnachrichten): E.164-Telefonnummer (z. B. `+15551234567`)

```json
{
  "broadcast": {
    "120363403215116621@g.us": ["alfred", "baerbel", "assistant3"]
  }
}
```

**Ergebnis:** Wenn OpenClaw in diesem Chat antwortet, werden alle drei Agenten ausgeführt.

<div id="processing-strategy">
  ### Verarbeitungsstrategie
</div>

Steuern Sie, wie Agenten Nachrichten verarbeiten:

<div id="parallel-default">
  #### Parallel (Standard)
</div>

Alle Agenten werden gleichzeitig ausgeführt:

```json
{
  "broadcast": {
    "strategy": "parallel",
    "120363403215116621@g.us": ["alfred", "baerbel"]
  }
}
```

<div id="sequential">
  #### Sequenziell
</div>

Agenten werden nacheinander verarbeitet (jeweils einer wartet, bis der vorherige fertig ist):

```json
{
  "broadcast": {
    "strategy": "sequential",
    "120363403215116621@g.us": ["alfred", "baerbel"]
  }
}
```

<div id="complete-example">
  ### Vollständiges Beispiel
</div>

```json
{
  "agents": {
    "list": [
      {
        "id": "code-reviewer",
        "name": "Code Reviewer",
        "workspace": "/path/to/code-reviewer",
        "sandbox": { "mode": "all" }
      },
      {
        "id": "security-auditor",
        "name": "Security Auditor",
        "workspace": "/path/to/security-auditor",
        "sandbox": { "mode": "all" }
      },
      {
        "id": "docs-generator",
        "name": "Documentation Generator",
        "workspace": "/path/to/docs-generator",
        "sandbox": { "mode": "all" }
      }
    ]
  },
  "broadcast": {
    "strategy": "parallel",
    "120363403215116621@g.us": ["code-reviewer", "security-auditor", "docs-generator"],
    "120363424282127706@g.us": ["support-en", "support-de"],
    "+15555550123": ["assistant", "logger"]
  }
}
```

<div id="how-it-works">
  ## So funktioniert es
</div>

<div id="message-flow">
  ### Nachrichtenfluss
</div>

1. **Eingehende Nachricht** geht in einer WhatsApp-Gruppe ein
2. **Broadcast-Prüfung**: Das System prüft, ob die Peer-ID in `broadcast` enthalten ist
3. **Falls in der Broadcast-Liste**:
   * Alle aufgeführten Agenten verarbeiten die Nachricht
   * Jeder Agent hat seinen eigenen Sitzungsschlüssel und isolierten Kontext
   * Agenten verarbeiten parallel (Standard) oder sequentiell
4. **Falls nicht in der Broadcast-Liste**:
   * Normales Routing kommt zur Anwendung (erstes passendes Binding)

Hinweis: Broadcast-Gruppen umgehen keine Allowlists des Kanals oder Gruppen-Aktivierungsregeln (Erwähnungen/Befehle/etc.). Sie ändern nur, *welche Agenten ausgeführt werden*, wenn eine Nachricht für die Verarbeitung in Frage kommt.

<div id="session-isolation">
  ### Sitzungsisolation
</div>

Jeder Agent in einer Broadcast-Gruppe hält vollständig getrennte:

* **Sitzungsschlüssel** (`agent:alfred:whatsapp:group:120363...` vs. `agent:baerbel:whatsapp:group:120363...`)
* **Konversationsverläufe** (der Agent sieht die Nachrichten anderer Agenten nicht)
* **Arbeitsbereiche** (separate Sandboxes, falls konfiguriert)
* **Tool-Zugriffe** (unterschiedliche Allow-/Deny-Listen)
* **Speicher/Kontexte** (separate IDENTITY.md, SOUL.md usw.)
* **Gruppenkontext-Puffer** (aktuelle Gruppennachrichten, die für Kontext verwendet werden) wird pro Peer geteilt, sodass alle Broadcast-Agenten beim Auslösen denselben Kontext sehen

Dadurch kann jeder Agent Folgendes haben:

* Unterschiedliche Persönlichkeiten
* Unterschiedliche Tool-Zugriffe (z. B. Read-only vs. Read-write)
* Unterschiedliche Modelle (z. B. opus vs. sonnet)
* Unterschiedlich installierte Fähigkeiten

<div id="example-isolated-sessions">
  ### Beispiel: Isolierte Sitzungen
</div>

In Gruppe `120363403215116621@g.us` mit Agenten `["alfred", "baerbel"]`:

**Alfreds Kontext:**

```
Session: agent:alfred:whatsapp:group:120363403215116621@g.us
History: [user message, alfred's previous responses]
Workspace: /Users/pascal/openclaw-alfred/
Tools: read, write, exec
```

**Bärbels Kontext:**

```
Session: agent:baerbel:whatsapp:group:120363403215116621@g.us  
History: [user message, baerbel's previous responses]
Workspace: /Users/pascal/openclaw-baerbel/
Tools: read only
```

<div id="best-practices">
  ## Best Practices
</div>

<div id="1-keep-agents-focused">
  ### 1. Agenten fokussiert halten
</div>

Entwirf jeden Agenten mit einer einzigen, klaren Aufgabe:

```json
{
  "broadcast": {
    "DEV_GROUP": ["formatter", "linter", "tester"]
  }
}
```

✅ **Gut:** Jeder agent hat genau eine Aufgabe
❌ **Schlecht:** Ein generischer „dev-helper“-agent

<div id="2-use-descriptive-names">
  ### 2. Verwende aussagekräftige Namen
</div>

Mache deutlich, was jeder agent tut:

```json
{
  "agents": {
    "security-scanner": { "name": "Security Scanner" },
    "code-formatter": { "name": "Code Formatter" },
    "test-generator": { "name": "Test Generator" }
  }
}
```

<div id="3-configure-different-tool-access">
  ### 3. Unterschiedliche Tool-Zugriffe konfigurieren
</div>

Gib Agenten nur die Tools, die sie wirklich benötigen:

```json
{
  "agents": {
    "reviewer": {
      "tools": { "allow": ["read", "exec"] }  // Read-only
    },
    "fixer": {
      "tools": { "allow": ["read", "write", "edit", "exec"] }  // Lesen und Schreiben
    }
  }
}
```

<div id="4-monitor-performance">
  ### 4. Performance überwachen
</div>

Wenn du viele Agenten hast, solltest du Folgendes beachten:

* Verwende `"strategy": "parallel"` (Standard) für mehr Geschwindigkeit
* Beschränke Broadcast-Gruppen auf 5–10 Agenten
* Verwende schnellere Modelle für einfachere Agenten

<div id="5-handle-failures-gracefully">
  ### 5. Fehler robust handhaben
</div>

Agenten fallen unabhängig voneinander aus. Der Fehler eines Agenten blockiert die anderen nicht:

```
Message → [Agent A ✓, Agent B ✗ error, Agent C ✓]
Result: Agent A and C respond, Agent B logs error
```

<div id="compatibility">
  ## Kompatibilität
</div>

<div id="providers">
  ### Anbieter
</div>

Broadcast-Gruppen unterstützen derzeit:

* ✅ WhatsApp (implementiert)
* 🚧 Telegram (geplant)
* 🚧 Discord (geplant)
* 🚧 Slack (geplant)

<div id="routing">
  ### Routing
</div>

Broadcast-Gruppen arbeiten parallel zum bestehenden Routing:

```json
{
  "bindings": [
    { "match": { "channel": "whatsapp", "peer": { "kind": "group", "id": "GROUP_A" } }, "agentId": "alfred" }
  ],
  "broadcast": {
    "GROUP_B": ["agent1", "agent2"]
  }
}
```

* `GROUP_A`: Nur Alfred antwortet (normales Routing)
* `GROUP_B`: agent1 UND agent2 antworten (Broadcast)

**Priorität:** `broadcast` hat Vorrang vor `bindings`.

<div id="troubleshooting">
  ## Fehlerbehebung
</div>

<div id="agents-not-responding">
  ### Agenten reagieren nicht
</div>

**Prüfen:**

1. Agent-IDs sind in `agents.list` vorhanden
2. Peer-ID-Format ist korrekt (z. B. `120363403215116621@g.us`)
3. Agenten stehen nicht auf Sperrlisten

**Debug:**

```bash
tail -f ~/.openclaw/logs/gateway.log | grep broadcast
```

<div id="only-one-agent-responding">
  ### Nur ein Agent antwortet
</div>

**Ursache:** Die Peer-ID ist möglicherweise in `bindings`, aber nicht in `broadcast`.

**Behebung:** Zur Broadcast-Konfiguration hinzufügen oder aus `bindings` entfernen.

<div id="performance-issues">
  ### Leistungsprobleme
</div>

**Wenn es bei vielen Agenten langsam ist:**

* Anzahl der Agenten pro Gruppe reduzieren
* Leichtere Modelle verwenden (sonnet statt opus)
* Sandbox-Startzeit prüfen

<div id="examples">
  ## Beispiele
</div>

<div id="example-1-code-review-team">
  ### Beispiel 1: Code-Review-Team
</div>

```json
{
  "broadcast": {
    "strategy": "parallel",
    "120363403215116621@g.us": [
      "code-formatter",
      "security-scanner",
      "test-coverage",
      "docs-checker"
    ]
  },
  "agents": {
    "list": [
      { "id": "code-formatter", "workspace": "~/agents/formatter", "tools": { "allow": ["read", "write"] } },
      { "id": "security-scanner", "workspace": "~/agents/security", "tools": { "allow": ["read", "exec"] } },
      { "id": "test-coverage", "workspace": "~/agents/testing", "tools": { "allow": ["read", "exec"] } },
      { "id": "docs-checker", "workspace": "~/agents/docs", "tools": { "allow": ["read"] } }
    ]
  }
}
```

**Benutzer sendet:** Codeausschnitt
**Responses:**

* code-formatter: &quot;Einrückung korrigiert und Typannotationen hinzugefügt&quot;
* security-scanner: &quot;⚠️ SQL-Injection-Schwachstelle in Zeile 12&quot;
* test-coverage: &quot;Testabdeckung liegt bei 45 %, Tests für Fehlerfälle fehlen&quot;
* docs-checker: &quot;Fehlender Docstring für die Funktion `process_data`&quot;

<div id="example-2-multi-language-support">
  ### Beispiel 2: Mehrsprachige Unterstützung
</div>

```json
{
  "broadcast": {
    "strategy": "sequential",
    "+15555550123": ["detect-language", "translator-en", "translator-de"]
  },
  "agents": {
    "list": [
      { "id": "detect-language", "workspace": "~/agents/lang-detect" },
      { "id": "translator-en", "workspace": "~/agents/translate-en" },
      { "id": "translator-de", "workspace": "~/agents/translate-de" }
    ]
  }
}
```

<div id="api-reference">
  ## API-Referenz
</div>

<div id="config-schema">
  ### Konfigurationsschema
</div>

```typescript
interface OpenClawConfig {
  broadcast?: {
    strategy?: "parallel" | "sequential";
    [peerId: string]: string[];
  };
}
```

<div id="fields">
  ### Felder
</div>

* `strategy` (optional): Wie Agenten verarbeitet werden sollen
  * `"parallel"` (Standard): Alle Agenten werden gleichzeitig ausgeführt
  * `"sequential"`: Agenten werden in der Reihenfolge des Arrays ausgeführt

* `[peerId]`: WhatsApp-Gruppen-JID, E.164-Nummer oder andere Peer-ID
  * Wert: Array von Agent-IDs, die Nachrichten verarbeiten sollen

<div id="limitations">
  ## Einschränkungen
</div>

1. **Maximale Anzahl Agenten:** Kein festes Limit, aber mehr als 10 Agenten können zu Verzögerungen führen
2. **Gemeinsamer Kontext:** Agenten sehen die Antworten der anderen nicht (bewusst so gestaltet)
3. **Nachrichtenreihenfolge:** Parallele Antworten können in beliebiger Reihenfolge eintreffen
4. **Rate Limits:** Alle Agenten zählen gegen die WhatsApp-Rate-Limits

<div id="future-enhancements">
  ## Zukünftige Erweiterungen
</div>

Geplante Funktionen:

* [ ] Modus für gemeinsamen Kontext (Agenten sehen die Antworten der anderen)
* [ ] Agenten­koordination (Agenten können sich gegenseitig Signale senden)
* [ ] Dynamische Agentenauswahl (Agenten abhängig vom Nachrichteninhalt auswählen)
* [ ] Agentenprioritäten (einige Agenten antworten vor anderen)

<div id="see-also">
  ## Siehe auch
</div>

* [Multi-Agent-Konfiguration](/de/multi-agent-sandbox-tools)
* [Routing-Konfiguration](/de/concepts/channel-routing)
* [Sitzungsverwaltung](/de/concepts/sessions)