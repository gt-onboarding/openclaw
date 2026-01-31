---
title: Qwen
summary: "Utiliser Qwen OAuth (niveau gratuit) dans OpenClaw"
read_when:
  - Vous souhaitez utiliser Qwen avec OpenClaw
  - Vous souhaitez un accès OAuth gratuit à Qwen Coder
---

<div id="qwen">
  # Qwen
</div>

Qwen propose un flux d’authentification OAuth gratuit pour les modèles Qwen Coder et Qwen Vision
(2 000 requêtes/jour, sous réserve des limites de débit de Qwen).

<div id="enable-the-plugin">
  ## Activer le plugin
</div>

```bash
openclaw plugins enable qwen-portal-auth
```

Redémarrez Gateway après l’activation.

<div id="authenticate">
  ## Authentification
</div>

```bash
openclaw models auth login --provider qwen-portal --set-default
```

Cela exécute le flux OAuth utilisant un code d’appareil pour Qwen et ajoute une entrée de fournisseur dans votre
`models.json` (ainsi qu’un alias `qwen` pour basculer rapidement).

<div id="model-ids">
  ## Identifiants de modèles
</div>

* `qwen-portal/coder-model`
* `qwen-portal/vision-model`

Basculez de modèle avec :

```bash
openclaw models set qwen-portal/coder-model
```

<div id="reuse-qwen-code-cli-login">
  ## Réutiliser la connexion Qwen Code CLI
</div>

Si vous vous êtes déjà connecté avec la Qwen Code CLI, OpenClaw synchronisera les identifiants
depuis `~/.qwen/oauth_creds.json` lorsqu&#39;il charge le store d&#39;authentification. Vous devez tout de même disposer
d&#39;une entrée `models.providers.qwen-portal` (utilisez la commande de connexion ci-dessus pour en créer une).

<div id="notes">
  ## Notes
</div>

* Les jetons sont actualisés automatiquement ; relancez la commande de connexion si l’actualisation échoue ou si l’accès est révoqué.
* URL de base par défaut : `https://portal.qwen.ai/v1` (remplacez-la par
  `models.providers.qwen-portal.baseUrl` si Qwen fournit un autre point de terminaison).
* Consultez [Fournisseurs de modèles](/fr/concepts/model-providers) pour les règles globales applicables aux fournisseurs.