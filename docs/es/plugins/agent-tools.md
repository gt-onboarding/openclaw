---
title: Herramientas de Agente
summary: "Escribe herramientas de agente en un complemento (esquemas, herramientas opcionales, listas de permitidos)"
read_when:
  - Quieres añadir una nueva herramienta de agente en un complemento
  - Necesitas que una herramienta solo esté disponible para identidades incluidas en una lista de permitidos
---

<div id="plugin-agent-tools">
  # Herramientas de agente de complementos
</div>

Los complementos de OpenClaw pueden registrar **herramientas de agente** (funciones definidas mediante JSON Schema) que se exponen
al LLM durante las ejecuciones del agente. Las herramientas pueden ser **obligatorias** (siempre disponibles) u
**opcionales** (de activación voluntaria).

Las herramientas de agente se configuran en `tools` en la configuración principal, o por agente en
`agents.list[].tools`. La política de lista de permitidos/lista de denegados controla qué herramientas puede invocar el agente.

<div id="basic-tool">
  ## Herramienta básica
</div>

```ts
import { Type } from "@sinclair/typebox";

export default function (api) {
  api.registerTool({
    name: "my_tool",
    description: "Do a thing",
    parameters: Type.Object({
      input: Type.String(),
    }),
    async execute(_id, params) {
      return { content: [{ type: "text", text: params.input }] };
    },
  });
}
```


<div id="optional-tool-optin">
  ## Herramienta opcional (opt‑in)
</div>

Las herramientas opcionales **nunca** se habilitan automáticamente. Los usuarios deben agregarlas a la lista de permitidos del agente.

```ts
export default function (api) {
  api.registerTool(
    {
      name: "workflow_tool",
      description: "Run a local workflow",
      parameters: {
        type: "object",
        properties: {
          pipeline: { type: "string" },
        },
        required: ["pipeline"],
      },
      async execute(_id, params) {
        return { content: [{ type: "text", text: params.pipeline }] };
      },
    },
    { optional: true },
  );
}
```

Habilita las herramientas opcionales en `agents.list[].tools.allow` (o en `tools.allow` global):

```json5
{
  agents: {
    list: [
      {
        id: "main",
        tools: {
          allow: [
            "workflow_tool",  // nombre de herramienta específica
            "workflow",       // id del complemento (habilita todas las herramientas de ese complemento)
            "group:plugins"   // todas las herramientas de complementos
          ]
        }
      }
    ]
  }
}
```

Otros parámetros de configuración que afectan la disponibilidad de herramientas:

* Las listas de permitidos que incluyen únicamente herramientas de complementos se tratan como inclusiones explícitas de complementos; las herramientas principales siguen
  habilitadas a menos que también incluyas herramientas principales o grupos en la lista de permitidos.
* `tools.profile` / `agents.list[].tools.profile` (lista de permitidos base)
* `tools.byProvider` / `agents.list[].tools.byProvider` (permitir/denegar específico por proveedor)
* `tools.sandbox.tools.*` (política de herramientas del sandbox cuando se ejecuta en sandbox)


<div id="rules-tips">
  ## Reglas y consejos
</div>

- Los nombres de herramientas **no** deben entrar en conflicto con los nombres de herramientas principales; las herramientas en conflicto se omiten.
- Los ID de complementos usados en listas de permitidos no deben entrar en conflicto con los nombres de herramientas principales.
- Es preferible usar `optional: true` en las herramientas que producen efectos secundarios o requieren binarios/credenciales adicionales.