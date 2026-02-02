---
title: Protocollo di configurazione per l'onboarding
summary: "Note sul protocollo RPC per la procedura guidata di onboarding e lo schema di configurazione"
read_when: "Quando modifichi i passaggi della procedura guidata di onboarding o gli endpoint dello schema di configurazione"
---

<div id="onboarding-config-protocol">
  # Protocollo di onboarding e configurazione
</div>

Scopo: superfici di onboarding e configurazione condivise tra CLI, app macOS e Web UI.

<div id="components">
  ## Componenti
</div>

- Motore del wizard (sessione condivisa + prompt + stato di onboarding).
- L'onboarding tramite CLI usa lo stesso flusso guidato dei client UI.
- Il Gateway RPC espone endpoint per il wizard e per lo schema di configurazione.
- L'onboarding su macOS usa il modello a step del wizard.
- La Web UI genera i moduli di configurazione da JSON Schema + UI hints.

<div id="gateway-rpc">
  ## RPC del Gateway
</div>

- `wizard.start` parametri: `{ mode?: "local"|"remote", workspace?: string }`
- `wizard.next` parametri: `{ sessionId, answer?: { stepId, value? } }`
- `wizard.cancel` parametri: `{ sessionId }`
- `wizard.status` parametri: `{ sessionId }`
- `config.schema` parametri: `{}`

Risposte (struttura)

- Wizard: `{ sessionId, done, step?, status?, error? }`
- Schema di configurazione: `{ schema, uiHints, version, generatedAt }`

<div id="ui-hints">
  ## Suggerimenti per la UI
</div>

- `uiHints` indicizzati per percorso; metadati opzionali (label/help/group/order/advanced/sensitive/placeholder).
- I campi sensibili vengono visualizzati come input di tipo password; nessun livello di offuscamento.
- I nodi di schema non supportati vengono gestiti tramite l'editor JSON grezzo.

<div id="notes">
  ## Note
</div>

- Questo documento è l’unica fonte di verità per tenere traccia dei refactoring del protocollo di onboarding/configurazione.