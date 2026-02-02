---
title: "Vercel AI Gateway"
summary: "Configuration de Vercel AI Gateway (authentification + sélection de modèle)"
read_when:
  - Vous souhaitez utiliser Vercel AI Gateway avec OpenClaw
  - Vous avez besoin de la variable d’environnement de clé API ou de l’option d’authentification via la CLI
---

<div id="vercel-ai-gateway">
  # Vercel AI Gateway
</div>

Le [Vercel AI Gateway](https://vercel.com/ai-gateway) fournit une API unifiée pour accéder à des centaines de modèles via un seul point de terminaison. 

- Fournisseur : `vercel-ai-gateway`
- Auth : `AI_GATEWAY_API_KEY`
- API : compatible avec Anthropic Messages

<div id="quick-start">
  ## Démarrage rapide
</div>

1. Définissez la clé API (recommandé : la stocker pour Gateway) :

```bash
openclaw onboard --auth-choice ai-gateway-api-key
```

2. Définissez un modèle par défaut :

```json5
{
  agents: {
    defaults: {
      model: { primary: "vercel-ai-gateway/anthropic/claude-opus-4.5" }
    }
  }
}
```


<div id="non-interactive-example">
  ## Exemple non interactif
</div>

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice ai-gateway-api-key \
  --ai-gateway-api-key "$AI_GATEWAY_API_KEY"
```


<div id="environment-note">
  ## Remarque sur l'environnement
</div>

Si Gateway s'exécute en tant que démon (launchd/systemd), assurez-vous que `AI_GATEWAY_API_KEY`
est disponible pour ce processus (par exemple dans `~/.openclaw/.env` ou via
`env.shellEnv`).