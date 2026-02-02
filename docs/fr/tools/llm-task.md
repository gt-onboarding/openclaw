---
title: Tâche LLM
summary: "Tâches LLM au format JSON uniquement pour les workflows (plugin d’outil optionnel)"
read_when:
  - Vous voulez une étape LLM au format JSON uniquement dans vos workflows
  - Vous avez besoin d'une sortie LLM validée par un schéma pour l'automatisation
---

<div id="llm-task">
  # Tâche LLM
</div>

`llm-task` est un **outil de plugin optionnel** qui exécute une tâche LLM produisant uniquement du JSON et
renvoie une sortie structurée (qui peut éventuellement être validée par rapport à un schéma JSON).

C'est idéal pour les moteurs de workflow comme Lobster : vous pouvez ajouter une seule étape LLM
sans écrire de code OpenClaw personnalisé pour chaque workflow.

<div id="enable-the-plugin">
  ## Activer le plugin
</div>

1. Activez le plugin :

```json
{
  "plugins": {
    "entries": {
      "llm-task": { "enabled": true }
    }
  }
}
```

2. Ajoutez l’outil à la liste d’autorisation (il est enregistré avec `optional: true`) :

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
  ## Configuration (optionnelle)
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

`allowedModels` est une liste d’autorisation contenant des chaînes `provider/model`. Si ce paramètre est défini, toute requête qui n’y figure pas est rejetée.


<div id="tool-parameters">
  ## Paramètres de l’outil
</div>

- `prompt` (chaîne de caractères, obligatoire)
- `input` (tout type, facultatif)
- `schema` (objet, schéma JSON facultatif)
- `provider` (chaîne de caractères, facultatif)
- `model` (chaîne de caractères, facultatif)
- `authProfileId` (chaîne de caractères, facultatif)
- `temperature` (nombre, facultatif)
- `maxTokens` (nombre, facultatif)
- `timeoutMs` (nombre, facultatif)

<div id="output">
  ## Sortie
</div>

Renvoie `details.json` contenant le JSON analysé (et le valide par rapport au
`schema` lorsqu'il est fourni).

<div id="example-lobster-workflow-step">
  ## Exemple : étape du workflow Lobster
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
  ## Notes de sécurité
</div>

- L'outil est **uniquement JSON** et demande au modèle de produire uniquement du JSON (aucun
  délimiteur de code, aucun commentaire).
- Aucun outil n'est exposé au modèle pour cette exécution.
- Considérez toute sortie comme non fiable tant que vous ne l'avez pas validée avec `schema`.
- Placez les validations/approbations avant toute étape produisant des effets de bord (send, post, exec).