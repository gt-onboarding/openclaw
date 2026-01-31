---
title: Tarea LLM
summary: "Tareas LLM exclusivamente en JSON para flujos de trabajo (herramienta de complemento opcional)"
read_when:
  - Quieres un paso LLM exclusivamente en JSON dentro de flujos de trabajo
  - Necesitas salida de LLM validada mediante un esquema para automatización
---

<div id="llm-task">
  # Tarea LLM
</div>

`llm-task` es una **herramienta de complemento opcional** que ejecuta una tarea de LLM exclusivamente sobre JSON y
devuelve una salida estructurada (opcionalmente validada contra un JSON Schema).

Esto es ideal para motores de flujo de trabajo como Lobster: puedes añadir un único paso de LLM
sin tener que escribir código personalizado de OpenClaw para cada flujo de trabajo.

<div id="enable-the-plugin">
  ## Habilitar el complemento
</div>

1. Habilita el complemento:

```json
{
  "plugins": {
    "entries": {
      "llm-task": { "enabled": true }
    }
  }
}
```

2. Agrega la herramienta a la lista de permitidos (está registrada con `optional: true`):

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
  ## Configuración (opcional)
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

`allowedModels` es una lista de permitidos de cadenas con formato `provider/model`. Si se define, cualquier solicitud fuera de la lista se rechaza.


<div id="tool-parameters">
  ## Parámetros de la herramienta
</div>

- `prompt` (string, obligatorio)
- `input` (any, opcional)
- `schema` (object, esquema JSON opcional)
- `provider` (string, opcional)
- `model` (string, opcional)
- `authProfileId` (string, opcional)
- `temperature` (number, opcional)
- `maxTokens` (number, opcional)
- `timeoutMs` (number, opcional)

<div id="output">
  ## Output
</div>

Devuelve `details.json`, que contiene el JSON analizado (y lo valida frente a
`schema` cuando se proporciona).

<div id="example-lobster-workflow-step">
  ## Ejemplo: paso de un flujo de trabajo de Lobster
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
  ## Notas de seguridad
</div>

- La herramienta es **únicamente JSON** e indica al modelo que genere solo JSON (sin
  bloques de código, sin comentarios).
- No se exponen herramientas al modelo durante esta ejecución.
- Trata la salida como no confiable, a menos que la valides con `schema`.
- Coloca las aprobaciones antes de cualquier paso que produzca efectos secundarios (send, post, exec).