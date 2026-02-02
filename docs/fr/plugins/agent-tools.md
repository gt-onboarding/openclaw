---
title: Outils d’agent
summary: "Développer des outils d’agent dans un plugin (schémas, outils optionnels, listes d’autorisation)"
read_when:
  - Vous voulez ajouter un nouvel outil d’agent dans un plugin
  - Vous devez rendre un outil activable via des listes d’autorisation
---

<div id="plugin-agent-tools">
  # Outils d’agent des plugins
</div>

Les plugins OpenClaw peuvent enregistrer des **outils d’agent** (fonctions au format schéma JSON) qui sont exposés
au LLM pendant l’exécution de l’agent. Les outils peuvent être **obligatoires** (toujours disponibles) ou
**facultatifs** (activables explicitement).

Les outils d’agent sont configurés sous `tools` dans la configuration principale, ou par agent sous
`agents.list[].tools`. La politique de liste d’autorisation/liste de refus détermine quels outils l’agent
peut appeler.

<div id="basic-tool">
  ## Outil de base
</div>

```ts
import { Type } from "@sinclair/typebox";

export default function (api) {
  api.registerTool({
    name: "my_tool",
    description: "Effectue une action",
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
  ## Outil optionnel (activation explicite)
</div>

Les outils optionnels ne sont **jamais** activés automatiquement. Les utilisateurs doivent les ajouter à la liste d’autorisation de l’agent.

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

Activez les outils optionnels dans `agents.list[].tools.allow` (ou au niveau global dans `tools.allow`) :

```json5
{
  agents: {
    list: [
      {
        id: "main",
        tools: {
          allow: [
            "workflow_tool",  // specific tool name
            "workflow",       // id du plugin (active tous les outils de ce plugin)
            "group:plugins"   // all plugin tools
          ]
        }
      }
    ]
  }
}
```

Autres paramètres de configuration qui affectent la disponibilité des outils :

* Les listes d’autorisation qui ne répertorient que des outils de plugin sont interprétées comme des activations explicites de plugins ; les outils principaux restent
  activés, sauf si vous incluez aussi des outils principaux ou des groupes dans la liste d’autorisation.
* `tools.profile` / `agents.list[].tools.profile` (liste d’autorisation de base)
* `tools.byProvider` / `agents.list[].tools.byProvider` (autorisation/interdiction spécifique au fournisseur)
* `tools.sandbox.tools.*` (politique des outils en sandbox lorsque l’agent est sandboxé)


<div id="rules-tips">
  ## Règles + conseils
</div>

- Les noms d’outils ne doivent **pas** entrer en conflit avec les noms des outils cœur ; les outils en conflit sont ignorés.
- Les ID de plugin utilisés dans les listes d’autorisation ne doivent pas entrer en conflit avec les noms des outils cœur.
- Privilégiez `optional: true` pour les outils qui déclenchent des effets de bord ou nécessitent des binaires/identifiants supplémentaires.