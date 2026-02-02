---
title: Protocole de configuration de l'onboarding
summary: "Notes sur le protocole RPC pour l'assistant d'onboarding et le schéma de configuration"
read_when: "Modification des étapes de l'assistant d'onboarding ou des points de terminaison du schéma de configuration"
---

<div id="onboarding-config-protocol">
  # Protocole d’onboarding et de configuration
</div>

Objectif : fournir des interfaces d’onboarding et de configuration communes entre la CLI, l’app macOS et l’UI Web.

<div id="components">
  ## Composants
</div>

- Moteur d’assistant (session partagée + prompts + état d’onboarding).
- L’onboarding via la CLI utilise le même flux d’assistant que les clients UI.
- Gateway RPC expose des points de terminaison d’assistant et de schéma de configuration.
- L’onboarding macOS utilise le modèle d’étapes de l’assistant.
- La Web UI rend les formulaires de configuration à partir du JSON Schema et des indications UI.

<div id="gateway-rpc">
  ## RPC du Gateway
</div>

- `wizard.start` Paramètres : `{ mode?: "local"|"remote", workspace?: string }`
- `wizard.next` Paramètres : `{ sessionId, answer?: { stepId, value? } }`
- `wizard.cancel` Paramètres : `{ sessionId }`
- `wizard.status` Paramètres : `{ sessionId }`
- `config.schema` Paramètres : `{}`

Réponses (structure)

- Assistant : `{ sessionId, done, step?, status?, error? }`
- Schéma de configuration : `{ schema, uiHints, version, generatedAt }`

<div id="ui-hints">
  ## Indications pour l’UI
</div>

- `uiHints` indexés par chemin ; métadonnées facultatives (label/help/group/order/advanced/sensitive/placeholder).
- Les champs sensibles sont rendus comme des champs de mot de passe ; aucun mécanisme de masquage.
- Les nœuds de schéma non pris en charge basculent vers l’éditeur JSON brut.

<div id="notes">
  ## Remarques
</div>

- Ce document est la source unique pour suivre les refontes du protocole d’onboarding et de configuration.