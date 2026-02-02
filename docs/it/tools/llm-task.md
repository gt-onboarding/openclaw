---
title: Task LLM
summary: "Task LLM solo JSON per i workflow (strumento plugin opzionale)"
read_when:
  - Vuoi uno step LLM solo JSON all'interno dei workflow
  - Hai bisogno di output LLM convalidato tramite schema per l'automazione
---

<div id="llm-task">
  # LLM Task
</div>

`llm-task` è uno **strumento plugin opzionale** che esegue un task LLM che utilizza esclusivamente JSON e
restituisce output strutturato (facoltativamente convalidato rispetto a uno schema JSON).

Questo è ideale per motori di workflow come Lobster: puoi aggiungere una singola fase LLM
senza scrivere codice OpenClaw personalizzato per ogni workflow.

<div id="enable-the-plugin">
  ## Attiva il plugin
</div>

1. Attiva il plugin:

```json
{
  "plugins": {
    "entries": {
      "llm-task": { "enabled": true }
    }
  }
}
```

2. Aggiungi lo strumento alla lista di autorizzati (viene registrato con `optional: true`):

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
  ## Config (opzionale)
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

`allowedModels` è una lista di autorizzati contenente stringhe `provider/model`. Se configurata, qualsiasi richiesta al di fuori della lista viene rifiutata.


<div id="tool-parameters">
  ## Parametri dello strumento
</div>

- `prompt` (string, obbligatorio)
- `input` (any, facoltativo)
- `schema` (object, JSON Schema opzionale)
- `provider` (string, facoltativo)
- `model` (string, facoltativo)
- `authProfileId` (string, facoltativo)
- `temperature` (number, facoltativo)
- `maxTokens` (number, facoltativo)
- `timeoutMs` (number, facoltativo)

<div id="output">
  ## Output
</div>

Restituisce `details.json`, che contiene il JSON analizzato (e, quando fornito, lo convalida rispetto allo
`schema`).

<div id="example-lobster-workflow-step">
  ## Esempio: passaggio del workflow di Lobster
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
  ## Note sulla sicurezza
</div>

- Lo strumento è **solo JSON** e istruisce il modello a produrre solo JSON (nessun
  blocco di codice, nessun commento).
- Nessuno strumento è esposto al modello in questa esecuzione.
- Considera l'output come non attendibile a meno che tu non lo convalidi con `schema`.
- Inserisci le approvazioni prima di qualsiasi passaggio con effetti collaterali (send, post, exec).