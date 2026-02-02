---
title: Lobster
summary: "Typisierte Workflow-Laufzeitumgebung für OpenClaw mit wiederaufnehmbaren Freigabe-Gates."
description: Typisierte Workflow-Laufzeitumgebung für OpenClaw — komponierbare Pipelines mit Freigabe-Gates.
read_when:
  - Du möchtest deterministische, mehrstufige Workflows mit expliziten Freigaben
  - Du musst einen Workflow fortsetzen, ohne frühere Schritte erneut auszuführen
---

<div id="lobster">
  # Lobster
</div>

Lobster ist eine Workflow-Shell, mit der OpenClaw mehrstufige Tool-Sequenzen als eine einzige, deterministische Operation mit expliziten Freigabe-Checkpoints ausführen kann.

<div id="hook">
  ## Hook
</div>

Dein Assistent kann die Tools entwickeln, mit denen er sich selbst verwaltet. Fordere einen Workflow an, und 30 Minuten später hast du eine CLI plus Pipelines, die sich wie ein einziger Aufruf ausführen lassen. Lobster ist das fehlende Puzzleteil: deterministische Pipelines, explizite Freigaben und ein fortsetzbarer Zustand.

<div id="why">
  ## Warum
</div>

Heutige komplexe Workflows erfordern viele Hin-und-her-Aufrufe von Tools. Jeder Aufruf kostet Token, und das LLM muss jeden Schritt orchestrieren. Lobster verschiebt diese Orchestrierung in eine typisierte Laufzeitumgebung:

* **Ein Aufruf statt vieler**: OpenClaw führt einen Lobster-Toolaufruf aus und erhält ein strukturiertes Ergebnis.
* **Integrierte Genehmigungen**: Seiteneffekte (E-Mail senden, Kommentar posten) halten den Workflow an, bis sie explizit genehmigt werden.
* **Fortsetzbar**: An­ge­hal­te­ne Workflows geben ein Token zurück; genehmige und setze fort, ohne alles neu ausführen zu müssen.

<div id="why-a-dsl-instead-of-plain-programs">
  ## Warum eine DSL statt normaler Programme?
</div>

Lobster ist bewusst klein gehalten. Das Ziel ist nicht „eine neue Sprache“, sondern eine vorhersagbare, KI-freundliche Pipeline-Spezifikation mit First-Class-Freigaben und Resume-Tokens.

* **Approve/Resume ist eingebaut**: Ein normales Programm kann einen Menschen um Eingabe bitten, aber es kann nicht mit einem dauerhaften Token *pausieren und fortsetzen*, ohne dass du diese Laufzeitumgebung selbst erfindest.
* **Determinismus + Nachvollziehbarkeit**: Pipelines sind Daten, daher lassen sie sich leicht loggen, diffen, erneut ausführen und überprüfen.
* **Begrenzter Spielraum für KI**: Eine winzige Grammatik + JSON-Piping reduziert „kreative“ Codepfade und macht Validierung realistisch.
* **Sicherheitsrichtlinie eingebaut**: Timeouts, Output-Grenzen, sandbox-Checks und Allowlists werden von der Laufzeitumgebung erzwungen, nicht von jedem einzelnen Skript.
* **Trotzdem programmierbar**: Jeder Schritt kann beliebige CLI-Befehle oder Skripte aufrufen. Wenn du JS/TS willst, generiere `.lobster`-Dateien aus Code.

<div id="how-it-works">
  ## Funktionsweise
</div>

OpenClaw startet die lokale `lobster`-CLI im **Tool-Modus** und liest einen JSON-Envelope aus stdout.
Wenn die Pipeline für eine Genehmigung pausiert, gibt das Tool ein `resumeToken` zurück, damit du den Vorgang später fortsetzen kannst.

<div id="pattern-small-cli-json-pipes-approvals">
  ## Pattern: kleine CLI + JSON-Pipes + Freigaben
</div>

Erstelle kleine Befehle, die JSON sprechen, und verknüpfe sie dann zu einem einzigen Lobster-Aufruf. (Beispiel-Befehlsnamen unten – ersetze sie durch deine eigenen.)

```bash
inbox list --json
inbox categorize --json
inbox apply --json
```

```json
{
  "action": "run",
  "pipeline": "exec --json --shell 'inbox list --json' | exec --stdin json --shell 'inbox categorize --json' | exec --stdin json --shell 'inbox apply --json' | approve --preview-from-stdin --limit 5 --prompt 'Apply changes?'",
  "timeoutMs": 30000
}
```

Wenn die Pipeline eine Freigabe anfordert, setze mit dem Token fort:

```json
{
  "action": "resume",
  "token": "<resumeToken>",
  "approve": true
}
```

Die KI stößt den Workflow an; Lobster führt die Schritte aus. Freigabe-Gates halten Nebeneffekte explizit und nachvollziehbar.

Beispiel: Eingabeelemente in Tool-Aufrufe abbilden:

```bash
gog.gmail.search --query 'newer_than:1d' \
  | openclaw.invoke --tool message --action send --each --item-key message --args-json '{"provider":"telegram","to":"..."}'
```

<div id="json-only-llm-steps-llm-task">
  ## Nur JSON-basierte LLM-Schritte (llm-task)
</div>

Für Workflows, die einen **strukturierten LLM-Schritt** benötigen, aktiviere das optionale
`llm-task`-Plugin-Tool und rufe es aus Lobster auf. So bleibt der Workflow
deterministisch, während du trotzdem mit einem Modell klassifizieren/zusammenfassen/Entwürfe erstellen kannst.

Aktiviere das Tool:

```json
{
  "plugins": {
    "entries": {
      "llm-task": { "enabled": true }
    }
  },
  "agents": {
    "list": [
      {
        "id": "main",
        "tools": { "allow": ["llm-task"] }
      }
    ]
  }
}
```

Verwenden Sie es in einer Pipeline:

```lobster
openclaw.invoke --tool llm-task --action json --args-json '{
  "prompt": "Analysiere die eingehende E-Mail und gib die Absicht sowie einen Entwurf zurück.",
  "input": { "subject": "Hello", "body": "Can you help?" },
  "schema": {
    "type": "object",
    "properties": {
      "intent": { "type": "string" },
      "draft": { "type": "string" }
    },
    "required": ["intent", "draft"],
    "additionalProperties": false
  }
}'
```

Siehe [LLM Task](/de/tools/llm-task) für weitere Details und Konfigurationsoptionen.

<div id="workflow-files-lobster">
  ## Workflow-Dateien (.lobster)
</div>

Lobster kann YAML-/JSON-Workflow-Dateien mit den Feldern `name`, `args`, `steps`, `env`, `condition` und `approval` ausführen. In OpenClaw-Toolaufrufen setzt du das Feld `pipeline` auf den Dateipfad.

```yaml
name: inbox-triage
args:
  tag:
    default: "family"
steps:
  - id: collect
    command: inbox list --json
  - id: categorize
    command: inbox categorize --json
    stdin: $collect.stdout
  - id: approve
    command: inbox apply --approve
    stdin: $categorize.stdout
    approval: required
  - id: execute
    command: inbox apply --execute
    stdin: $categorize.stdout
    condition: $approve.approved
```

Hinweise:

* `stdin: $step.stdout` und `stdin: $step.json` übergeben die Ausgabe eines vorherigen Schritts.
* `condition` (oder `when`) kann die Ausführung von Schritten von `$step.approved` abhängig machen.

<div id="install-lobster">
  ## Lobster installieren
</div>

Installiere die Lobster-CLI auf demselben **Host**, auf dem auch der OpenClaw Gateway läuft (siehe das [Lobster-Repo](https://github.com/openclaw/lobster)), und stelle sicher, dass `lobster` im `PATH` liegt.
Wenn du einen benutzerdefinierten Pfad zur Binärdatei verwenden möchtest, übergib einen **absoluten** `lobsterPath` im Tool-Aufruf.

<div id="enable-the-tool">
  ## Tool aktivieren
</div>

Lobster ist ein **optionales** Plugin-Tool (standardmäßig deaktiviert).

Empfohlen (ergänzend, sicher):

```json
{
  "tools": {
    "alsoAllow": ["lobster"]
  }
}
```

Oder pro agent:

```json
{
  "agents": {
    "list": [
      {
        "id": "main",
        "tools": {
          "alsoAllow": ["lobster"]
        }
      }
    ]
  }
}
```

Vermeide die Verwendung von `tools.allow: ["lobster"]`, es sei denn, du möchtest im restriktiven Allowlist-Modus arbeiten.

Hinweis: Allowlists sind für optionale Plugins ein Opt-in. Wenn deine Allowlist nur
Plugin-Tools (wie `lobster`) aufführt, bleiben die Kern-Tools in OpenClaw weiterhin aktiviert. Um Kern-Tools
einzuschränken, nimm die gewünschten Kern-Tools oder -Gruppen ebenfalls in die Allowlist auf.

<div id="example-email-triage">
  ## Beispiel: E-Mail-Triage
</div>

Ohne Lobster:

```
User: "Check my email and draft replies"
→ openclaw calls gmail.list
→ LLM summarizes
→ User: "draft replies to #2 and #5"
→ LLM drafts
→ User: "send #2"
→ openclaw calls gmail.send
(repeat daily, no memory of what was triaged)
```

Mit Lobster:

```json
{
  "action": "run",
  "pipeline": "email.triage --limit 20",
  "timeoutMs": 30000
}
```

Gibt einen JSON-Envelope zurück (gekürzt):

```json
{
  "ok": true,
  "status": "needs_approval",
  "output": [{ "summary": "5 need replies, 2 need action" }],
  "requiresApproval": {
    "type": "approval_request",
    "prompt": "2 Antwortentwürfe senden?",
    "items": [],
    "resumeToken": "..."
  }
}
```

Benutzer genehmigt → Fortsetzen:

```json
{
  "action": "resume",
  "token": "<resumeToken>",
  "approve": true
}
```

Ein einziger Workflow. Deterministisch. Sicher.

<div id="tool-parameters">
  ## Toolparameter
</div>

<div id="run">
  ### `run`
</div>

Pipeline im Tool-Modus ausführen.

```json
{
  "action": "run",
  "pipeline": "gog.gmail.search --query 'newer_than:1d' | email.triage",
  "cwd": "/path/to/workspace",
  "timeoutMs": 30000,
  "maxStdoutBytes": 512000
}
```

Workflow-Datei mit Argumenten ausführen:

```json
{
  "action": "run",
  "pipeline": "/path/to/inbox-triage.lobster",
  "argsJson": "{\"tag\":\"family\"}"
}
```

<div id="resume">
  ### `resume`
</div>

Setzt einen angehaltenen Workflow nach Freigabe fort.

```json
{
  "action": "resume",
  "token": "<resumeToken>",
  "approve": true
}
```

<div id="optional-inputs">
  ### Optionale Eingaben
</div>

* `lobsterPath`: Absoluter Pfad zur Lobster-Binärdatei (weglassen, um `PATH` zu verwenden).
* `cwd`: Arbeitsverzeichnis für die Pipeline (standardmäßig das aktuelle Arbeitsverzeichnis des Prozesses).
* `timeoutMs`: Beendet den Subprozess, wenn er diese Dauer überschreitet (Standard: 20000).
* `maxStdoutBytes`: Beendet den Subprozess, wenn `stdout` diese Größe überschreitet (Standard: 512000).
* `argsJson`: JSON-String, der an `lobster run --args-json` übergeben wird (nur für Workflow-Dateien).

<div id="output-envelope">
  ## Ausgabe-Envelope
</div>

Lobster gibt einen JSON-Envelope mit einem von drei Statuswerten zurück:

* `ok` → erfolgreich abgeschlossen
* `needs_approval` → pausiert; `requiresApproval.resumeToken` wird zum Fortsetzen benötigt
* `cancelled` → explizit abgelehnt oder abgebrochen

Das Tool stellt den Envelope sowohl in `content` (lesbar formatiertes JSON) als auch in `details` (Rohobjekt) bereit.

<div id="approvals">
  ## Freigaben
</div>

Wenn `requiresApproval` vorhanden ist, prüfe den Prompt und entscheide:

* `approve: true` → fortsetzen und Seiteneffekte ausführen
* `approve: false` → abbrechen und den Workflow finalisieren

Verwende `approve --preview-from-stdin --limit N`, um eine JSON-Vorschau an Freigabeanforderungen anzuhängen, ohne eigenes jq-/Heredoc-Gebastel. Resume-Tokens sind jetzt kompakt: Lobster speichert den Wiederaufnahmestatus des Workflows in seinem Statusverzeichnis und gibt einen kleinen Token-Schlüssel zurück.

<div id="openprose">
  ## OpenProse
</div>

OpenProse lässt sich gut mit Lobster kombinieren: Verwende `/prose`, um die Multi-Agenten-Vorbereitung zu orchestrieren, und führe anschließend eine Lobster-Pipeline für deterministische Freigaben aus. Wenn ein Prose-Programm Lobster benötigt, erlaube das Tool `lobster` für Sub-Agents über `tools.subagents.tools`. Siehe [OpenProse](/de/prose).

<div id="safety">
  ## Sicherheit
</div>

* **Nur lokaler Subprozess** — keine Netzwerkaufrufe durch das Plugin selbst.
* **Keine Secrets** — Lobster verwaltet kein OAuth; es ruft OpenClaw-Tools auf, die das übernehmen.
* **Sandbox-bewusst** — deaktiviert, wenn der Tool-Kontext in einer Sandbox läuft.
* **Gehärtet** — `lobsterPath` muss, falls angegeben, ein absoluter Pfad sein; Timeouts und Ausgabelimits werden erzwungen.

<div id="troubleshooting">
  ## Fehlerbehebung
</div>

* **`lobster subprocess timed out`** → erhöhe `timeoutMs` oder teile eine lange Pipeline auf.
* **`lobster output exceeded maxStdoutBytes`** → erhöhe `maxStdoutBytes` oder reduziere die Ausgabemenge.
* **`lobster returned invalid JSON`** → stelle sicher, dass die Pipeline im Tool-Modus läuft und nur JSON ausgibt.
* **`lobster failed (code …)`** → führe die gleiche Pipeline in einem Terminal aus, um `stderr` zu untersuchen.

<div id="learn-more">
  ## Mehr erfahren
</div>

* [Plugins](/de/plugin)
* [Entwicklung von Plugin-Tools](/de/plugins/agent-tools)

<div id="case-study-community-workflows">
  ## Fallstudie: Community-Workflows
</div>

Ein öffentliches Beispiel: ein „Second Brain“-CLI + Lobster-Pipelines, die drei Markdown-Vaults verwalten (persönlich, Partner, gemeinsam). Die CLI gibt JSON für Statistiken, Posteingangslisten und Scans auf veraltete Einträge aus; Lobster verkettet diese Befehle zu Workflows wie `weekly-review`, `inbox-triage`, `memory-consolidation` und `shared-task-sync`, jeweils mit Genehmigungsschritten. KI übernimmt die Bewertung (Kategorisierung), wenn verfügbar, und fällt andernfalls auf deterministische Regeln zurück.

* Thread: https://x.com/plattenschieber/status/2014508656335770033
* Repo: https://github.com/bloomedai/brain-cli