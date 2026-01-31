---
title: Environnement
summary: "Où OpenClaw charge les variables d'environnement et leur ordre de priorité"
read_when:
  - Vous devez savoir quelles variables d'environnement sont chargées, et dans quel ordre
  - Vous recherchez la cause de clés API manquantes dans le Gateway
  - Vous documentez l'authentification d'un fournisseur ou les environnements de déploiement
---

<div id="environment-variables">
  # Variables d&#39;environnement
</div>

OpenClaw lit les variables d&#39;environnement à partir de plusieurs sources. La règle est : **ne jamais remplacer les valeurs existantes**.

<div id="precedence-highest-lowest">
  ## Priorité (de la plus élevée à la plus faible)
</div>

1. **Environnement du processus** (ce que le processus Gateway reçoit déjà du shell ou du démon parent).
2. **`.env` dans le répertoire de travail courant** (comportement par défaut de dotenv ; n’écrase pas les valeurs existantes).
3. **`.env` global** à `~/.openclaw/.env` (alias `$OPENCLAW_STATE_DIR/.env` ; n’écrase pas les valeurs existantes).
4. **Bloc `env` de la config** dans `~/.openclaw/openclaw.json` (appliqué uniquement si la clé est absente).
5. **Importation optionnelle de l’environnement du shell de connexion** (`env.shellEnv.enabled` ou `OPENCLAW_LOAD_SHELL_ENV=1`), appliquée uniquement pour les clés attendues absentes.

Si le fichier de configuration est totalement absent, l’étape 4 est ignorée ; l’import depuis le shell est tout de même exécuté s’il est activé.

<div id="config-env-block">
  ## Bloc `env` de configuration
</div>

Deux manières équivalentes de définir des variables d&#39;environnement inline (dans les deux cas, sans écraser les valeurs existantes) :

```json5
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: {
      GROQ_API_KEY: "gsk-..."
    }
  }
}
```

<div id="shell-env-import">
  ## Import de l’environnement du shell
</div>

`env.shellEnv` exécute votre shell de connexion et importe uniquement les clés attendues **manquantes** :

```json5
{
  env: {
    shellEnv: {
      enabled: true,
      timeoutMs: 15000
    }
  }
}
```

Équivalents de variables d’environnement :

* `OPENCLAW_LOAD_SHELL_ENV=1`
* `OPENCLAW_SHELL_ENV_TIMEOUT_MS=15000`

<div id="env-var-substitution-in-config">
  ## Substitution de variables d&#39;environnement dans la configuration
</div>

Vous pouvez faire référence directement à des variables d&#39;environnement dans les valeurs de chaîne de caractères de la configuration en utilisant la syntaxe `${VAR_NAME}` :

```json5
{
  models: {
    providers: {
      "vercel-gateway": {
        apiKey: "${VERCEL_GATEWAY_API_KEY}"
      }
    }
  }
}
```

Voir [Configuration : substitution de variables d&#39;environnement](/fr/gateway/configuration#env-var-substitution-in-config) pour tous les détails.

<div id="related">
  ## Rubriques associées
</div>

* [Configuration du Gateway](/fr/gateway/configuration)
* [FAQ : variables d’environnement et chargement des fichiers .env](/fr/help/faq#env-vars-and-env-loading)
* [Présentation des modèles](/fr/concepts/models)