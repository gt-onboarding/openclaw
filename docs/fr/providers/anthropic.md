---
title: Anthropic
summary: "Utiliser Claude d'Anthropic via des clés API ou un setup-token dans OpenClaw"
read_when:
  - Vous souhaitez utiliser les modèles Anthropic dans OpenClaw
  - Vous souhaitez un setup-token au lieu de clés API
---

<div id="anthropic-claude">
  # Anthropic (Claude)
</div>

Anthropic développe la famille de modèles **Claude** et y donne accès via une API.
Dans OpenClaw, vous pouvez vous authentifier avec une clé d’API ou un **setup-token**.

<div id="option-a-anthropic-api-key">
  ## Option A : clé API Anthropic
</div>

**Idéal pour :** accès standard à l&#39;API et facturation à l&#39;usage.
Créez votre clé API dans la console Anthropic.

<div id="cli-setup">
  ### Configuration de la CLI
</div>

```bash
openclaw onboard
# choisissez : clé API Anthropic

# or non-interactive
openclaw onboard --anthropic-api-key "$ANTHROPIC_API_KEY"
```

<div id="config-snippet">
  ### Exemple de configuration
</div>

```json5
{
  env: { ANTHROPIC_API_KEY: "sk-ant-..." },
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-5" } } }
}
```

<div id="prompt-caching-anthropic-api">
  ## Mise en cache des prompts (API Anthropic)
</div>

OpenClaw **ne** remplace **pas** le TTL de cache par défaut d’Anthropic, sauf si vous le définissez.
Ceci est **réservé à l’API** ; l’authentification par abonnement ne tient pas compte des paramètres de TTL.

Pour définir le TTL par modèle, utilisez `cacheControlTtl` dans les `params` du modèle :

```json5
{
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-5": {
          params: { cacheControlTtl: "5m" } // ou « 1h »
        }
      }
    }
  }
}
```

OpenClaw inclut l’indicateur bêta `extended-cache-ttl-2025-04-11` pour les requêtes vers l’API Anthropic ; conservez-le si vous personnalisez les en-têtes du fournisseur (voir [/gateway/configuration](/fr/gateway/configuration)).

<div id="option-b-claude-setup-token">
  ## Option B : Claude setup-token
</div>

**Idéal pour :** utiliser votre abonnement Claude.

<div id="where-to-get-a-setup-token">
  ### Où obtenir un setup-token
</div>

Les setup-tokens sont générés par le **Claude Code CLI**, et non par la console Anthropic. Vous pouvez exécuter cette commande sur **n&#39;importe quelle machine** :

```bash
claude setup-token
```

Collez le jeton dans OpenClaw (assistant de configuration : **Anthropic token (paste setup-token)**), ou exécutez la commande sur l’hôte du Gateway :

```bash
openclaw models auth setup-token --provider anthropic
```

Si vous avez généré le jeton sur une autre machine, collez-le ici :

```bash
openclaw models auth paste-token --provider anthropic
```

<div id="cli-setup">
  ### Configuration de la CLI
</div>

```bash
# Collez un setup-token lors de l'intégration
openclaw onboard --auth-choice setup-token
```

<div id="config-snippet">
  ### Exemple de configuration
</div>

```json5
{
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-5" } } }
}
```

<div id="notes">
  ## Notes
</div>

* Générez le setup-token avec `claude setup-token` et collez-le, ou exécutez `openclaw models auth setup-token` sur l’hôte qui exécute le Gateway.
* Si vous voyez le message « OAuth token refresh failed … » sur un abonnement Claude, relancez l’authentification avec un setup-token. Voir [/gateway/troubleshooting#oauth-token-refresh-failed-anthropic-claude-subscription](/fr/gateway/troubleshooting#oauth-token-refresh-failed-anthropic-claude-subscription).
* Les détails d’authentification et les règles de réutilisation se trouvent dans [/concepts/oauth](/fr/concepts/oauth).

<div id="troubleshooting">
  ## Dépannage
</div>

**Erreurs 401 / jeton soudainement invalide**

* L’authentification de l’abonnement Claude peut expirer ou être révoquée. Relance la commande `claude setup-token`
  et colle le jeton sur l’**hôte du Gateway**.
* Si la connexion CLI à Claude se trouve sur une autre machine, utilise
  `openclaw models auth paste-token --provider anthropic` sur l’hôte du Gateway.

**Aucune clé API trouvée pour le fournisseur &quot;anthropic&quot;**

* L’authentification est **par agent**. Les nouveaux agents n’héritent pas des clés de l’agent principal.
* Relance la procédure d’onboarding pour cet agent, ou colle un setup-token / une clé API sur
  l’hôte du Gateway, puis vérifie avec `openclaw models status`.

**Aucun identifiant trouvé pour le profil `anthropic:default`**

* Exécute `openclaw models status` pour voir quel profil d’authentification est actif.
* Relance la procédure d’onboarding, ou colle un setup-token / une clé API pour ce profil.

**Aucun profil d’authentification disponible (tous en cooldown/indisponibles)**

* Vérifie la sortie de `openclaw models status --json` pour `auth.unusableProfiles`.
* Ajoute un autre profil Anthropic ou attends la fin du cooldown.

Plus d’infos : [/gateway/troubleshooting](/fr/gateway/troubleshooting) et [/help/faq](/fr/help/faq).