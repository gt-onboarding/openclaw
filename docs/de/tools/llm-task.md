---
title: LLM-Task
summary: "Rein JSON-basierte LLM-Tasks für Workflows (optionales Plugin-Tool)"
read_when:
  - Du möchtest einen rein JSON-basierten LLM-Schritt innerhalb von Workflows verwenden
  - Du benötigst schema-validierte LLM-Ausgabe für die Automatisierung
---

<div id="llm-task">
  # LLM-Aufgabe
</div>

`llm-task` ist ein **optionales Plugin-Tool**, das eine LLM-Aufgabe ausführt, die ausschließlich mit JSON arbeitet, und strukturierte Ausgaben zurückgibt (optional gegen ein JSON-Schema validiert).

Dies ist ideal für Workflow-Engines wie Lobster: Du kannst einen einzelnen LLM-Schritt hinzufügen, ohne für jeden Workflow eigenen OpenClaw-Code zu schreiben.

<div id="enable-the-plugin">
  ## Plugin aktivieren
</div>

1. Aktivieren Sie das Plugin:

```json
{
  "plugins": {
    "entries": {
      "llm-task": { "enabled": true }
    }
  }
}
```

2. Nimm das Tool in die Allowlist auf (es ist mit `optional: true` registriert):

```json
{
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


<div id="config-optional">
  ## Konfiguration (optional)
</div>

```json
{
  "plugins": {
    "entries": {
      "llm-task": {
        "enabled": true,
        "config": {
          "defaultProvider": "openai-codex",
          "defaultModel": "gpt-5.2",
          "defaultAuthProfileId": "main",
          "allowedModels": ["openai-codex/gpt-5.2"],
          "maxTokens": 800,
          "timeoutMs": 30000
        }
      }
    }
  }
}
```

`allowedModels` ist eine Allowlist mit `provider/model`-Strings. Wenn gesetzt, wird jede Anfrage, die nicht in der Liste enthalten ist, abgelehnt.


<div id="tool-parameters">
  ## Toolparameter
</div>

- `prompt` (string, erforderlich)
- `input` (any, optional)
- `schema` (object, optionales JSON-Schema)
- `provider` (string, optional)
- `model` (string, optional)
- `authProfileId` (string, optional)
- `temperature` (number, optional)
- `maxTokens` (number, optional)
- `timeoutMs` (number, optional)

<div id="output">
  ## Ausgabe
</div>

Gibt `details.json` zurück, das das geparste JSON enthält (und bei Angabe gegen `schema` validiert).

<div id="example-lobster-workflow-step">
  ## Beispiel: Schritt im Lobster-Workflow
</div>

```lobster
openclaw.invoke --tool llm-task --action json --args-json '{
  "prompt": "Given the input email, return intent and draft.",
  "input": {
    "subject": "Hello",
    "body": "Can you help?"
  },
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


<div id="safety-notes">
  ## Sicherheitshinweise
</div>

- Das Tool arbeitet **ausschließlich mit JSON** und weist das Modell an, nur JSON auszugeben (keine
  Codeblöcke, keine Kommentare).
- Für diesen Durchlauf werden dem Modell keine Tools zur Verfügung gestellt.
- Betrachte Ausgaben als nicht vertrauenswürdig, solange du sie nicht mit `schema` validierst.
- Hole Freigaben ein, bevor du Schritte mit Seiteneffekten ausführst (`send`, `post`, `exec`).