---
title: OpenAI
summary: "Utiliser OpenAI via des clés d'API ou un abonnement Codex dans OpenClaw"
read_when:
  - Vous souhaitez utiliser les modèles OpenAI dans OpenClaw
  - Vous souhaitez une authentification par abonnement Codex au lieu de clés d'API
---

<div id="openai">
  # OpenAI
</div>

OpenAI fournit des API de développement pour les modèles GPT. Codex prend en charge la **connexion via ChatGPT** pour un accès par abonnement ou la connexion via **clé API** pour un accès basé sur l’utilisation. Le cloud Codex nécessite une connexion via ChatGPT.

<div id="option-a-openai-api-key-openai-platform">
  ## Option A : clé API OpenAI (plateforme OpenAI)
</div>

**Idéal pour :** un accès direct à l&#39;API et une facturation à l&#39;usage.
Obtenez votre clé API depuis le tableau de bord OpenAI.

<div id="cli-setup">
  ### Configuration de la CLI
</div>

```bash
openclaw onboard --auth-choice openai-api-key
# ou en mode non-interactif
openclaw onboard --openai-api-key "$OPENAI_API_KEY"
```

<div id="config-snippet">
  ### Exemple de configuration
</div>

```json5
{
  env: { OPENAI_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "openai/gpt-5.2" } } }
}
```

<div id="option-b-openai-code-codex-subscription">
  ## Option B : abonnement OpenAI Code (Codex)
</div>

**Idéal pour :** utiliser un abonnement ChatGPT/Codex plutôt qu&#39;une clé API.
Le service Codex dans le cloud nécessite une connexion via ChatGPT, tandis que l&#39;outil CLI Codex prend en charge la connexion via ChatGPT ou via une clé API.

<div id="cli-setup">
  ### Configuration de la CLI
</div>

```bash
# Exécuter OAuth Codex dans l'assistant
openclaw onboard --auth-choice openai-codex

# Ou exécuter OAuth directement
openclaw models auth login --provider openai-codex
```

<div id="config-snippet">
  ### Exemple de configuration
</div>

```json5
{
  agents: { defaults: { model: { primary: "openai-codex/gpt-5.2" } } }
}
```

<div id="notes">
  ## Remarques
</div>

* Les références de modèles utilisent toujours la forme `provider/model` (voir [/concepts/models](/fr/concepts/models)).
* Les détails d’authentification et les règles de réutilisation se trouvent dans [/concepts/oauth](/fr/concepts/oauth).