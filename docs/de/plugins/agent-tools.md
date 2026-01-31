---
title: Agent-Tools
summary: "Agent-Tools in einem Plugin implementieren (Schemas, optionale Tools, Allowlists)"
read_when:
  - Du möchtest in einem Plugin ein neues Agent-Tool hinzufügen
  - Du musst ein Tool per Allowlist als Opt-in freigeben
---

<div id="plugin-agent-tools">
  # Plugin-Agent-Tools
</div>

OpenClaw-Plugins können **Agent-Tools** (JSON-Schema-Funktionen) registrieren, die
dem LLM während Agent-Ausführungen zur Verfügung stehen. Tools können
**erforderlich** (immer verfügbar) oder **optional** (nur bei Bedarf aktiviert) sein.

Agent-Tools werden in der Hauptkonfiguration unter `tools` oder pro Agent unter
`agents.list[].tools` konfiguriert. Die Allowlist-/Denylist-Richtlinie steuert, welche
Tools der Agent aufrufen kann.

<div id="basic-tool">
  ## Standard-Tool
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
  ## Optionales Tool (Opt-in)
</div>

Optionale Tools werden **nie** automatisch aktiviert. Nutzer müssen sie zu einer agent-Allowlist hinzufügen.

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

Aktiviere optionale Tools über `agents.list[].tools.allow` (oder global über `tools.allow`):

```json5
{
  agents: {
    list: [
      {
        id: "main",
        tools: {
          allow: [
            "workflow_tool",  // specific tool name
            "workflow",       // Plugin-ID (aktiviert alle Tools dieses Plugins)
            "group:plugins"   // all plugin tools
          ]
        }
      }
    ]
  }
}
```

Andere Konfigurationsparameter, die die Tool‑Verfügbarkeit beeinflussen:

* Allowlists, die nur Plugin‑Tools enthalten, werden als Plugin‑Opt‑ins behandelt; Core‑Tools bleiben
  weiterhin aktiviert, es sei denn, du nimmst auch Core‑Tools oder Gruppen in die Allowlist auf.
* `tools.profile` / `agents.list[].tools.profile` (Basis‑Allowlist)
* `tools.byProvider` / `agents.list[].tools.byProvider` (anbieterspezifische Allow-/Deny‑Regeln)
* `tools.sandbox.tools.*` (sandbox‑Tool‑Richtlinie bei Ausführung in einer sandbox)


<div id="rules-tips">
  ## Regeln + Tipps
</div>

- Tool-Namen dürfen **nicht** mit Core-Tool-Namen kollidieren; kollidierende Tools werden übersprungen.
- Plugin-IDs, die in Allowlists verwendet werden, dürfen nicht mit Core-Tool-Namen kollidieren.
- Verwende nach Möglichkeit `optional: true` für Tools, die Seiteneffekte auslösen oder zusätzliche
  Binaries/Zugangsdaten erfordern.