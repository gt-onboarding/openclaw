---
title: Strumenti dell'Agente
summary: "Definisci gli strumenti dell'agente in un plugin (schemi, strumenti opzionali, liste di autorizzati)"
read_when:
  - Vuoi aggiungere un nuovo strumento dell'agente in un plugin
  - Devi rendere uno strumento disponibile solo tramite liste di autorizzati
---

<div id="plugin-agent-tools">
  # Strumenti di agente dei plugin
</div>

I plugin di OpenClaw possono registrare **strumenti di agente** (funzioni JSON‑schema) che vengono esposti
all'LLM durante l'esecuzione dell'agente. Gli strumenti possono essere **obbligatori** (sempre disponibili) oppure
**opzionali** (attivati esplicitamente).

Gli strumenti di agente sono configurati in `tools` nella configurazione principale, oppure per singolo agente in
`agents.list[].tools`. La politica di allowlist (lista di autorizzati)/denylist controlla quali strumenti l'agente
può chiamare.

<div id="basic-tool">
  ## Tool di base
</div>

```ts
import { Type } from "@sinclair/typebox";

export default function (api) {
  api.registerTool({
    name: "my_tool",
    description: "Esegui un'operazione",
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
  ## Strumento opzionale (opt‑in)
</div>

Gli strumenti opzionali non vengono **mai** abilitati automaticamente. Gli utenti devono aggiungerli alla lista di autorizzati di un agente.

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

Attiva gli strumenti opzionali in `agents.list[].tools.allow` (o nel `tools.allow` globale):

```json5
{
  agents: {
    list: [
      {
        id: "main",
        tools: {
          allow: [
            "workflow_tool",  // specific tool name
            "workflow",       // id del plugin (abilita tutti gli strumenti di quel plugin)
            "group:plugins"   // all plugin tools
          ]
        }
      }
    ]
  }
}
```

Altre impostazioni di configurazione che influiscono sulla disponibilità degli strumenti:

* Le liste di autorizzati che elencano solo strumenti plugin sono trattate come opt‑in ai plugin; gli strumenti core restano
  abilitati a meno che tu non includa nella lista di autorizzati anche strumenti core o gruppi.
* `tools.profile` / `agents.list[].tools.profile` (lista di autorizzati di base)
* `tools.byProvider` / `agents.list[].tools.byProvider` (consenti/nega specifico per provider)
* `tools.sandbox.tools.*` (criteri sugli strumenti della sandbox quando è attiva)


<div id="rules-tips">
  ## Regole + suggerimenti
</div>

- I nomi degli strumenti **non devono** coincidere con i nomi degli strumenti core; gli strumenti in conflitto vengono ignorati.
- Gli ID dei plugin usati nelle liste di autorizzati non devono entrare in conflitto con i nomi degli strumenti core.
- Usa preferibilmente `optional: true` per gli strumenti che producono effetti collaterali o richiedono binari/credenziali aggiuntivi.