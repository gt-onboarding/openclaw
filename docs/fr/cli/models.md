---
title: Modèles
summary: "Référence CLI pour `openclaw models` (status/list/set/scan, alias, solutions de repli, authentification)"
read_when:
  - Vous souhaitez modifier les modèles par défaut ou afficher l’état d’authentification des fournisseurs
  - Vous souhaitez analyser les modèles/fournisseurs disponibles et déboguer les profils d’authentification
---

<div id="openclaw-models">
  # `openclaw models`
</div>

Découverte, analyse et configuration des modèles (modèle par défaut, solutions de repli, profils d&#39;authentification).

Contenu associé :

* Fournisseurs + modèles : [Modèles](/fr/providers/models)
* Configuration de l&#39;authentification des fournisseurs : [Premiers pas](/fr/start/getting-started)

<div id="common-commands">
  ## Commandes courantes
</div>

```bash
openclaw models status
openclaw models list
openclaw models set <model-or-alias>
openclaw models scan
```

`openclaw models status` affiche les valeurs par défaut/de repli résolues ainsi qu’un aperçu de l’état de l’authentification.
Lorsque des instantanés d’utilisation des fournisseurs sont disponibles, la section d’état OAuth/jeton inclut
des en-têtes d’utilisation des fournisseurs.
Ajoutez `--probe` pour exécuter des sondes d’authentification en temps réel sur chaque profil de fournisseur configuré.
Les sondes sont de vraies requêtes (elles peuvent consommer des jetons et déclencher des limitations de débit).
Utilisez `--agent <id>` pour inspecter l’état modèle/authentification d’un agent configuré. Lorsque cet argument est omis,
la commande utilise `OPENCLAW_AGENT_DIR`/`PI_CODING_AGENT_DIR` si elles sont définies, sinon l’agent par défaut configuré.

Notes :

* `models set <model-or-alias>` accepte `provider/model` ou un alias.
* Les références de modèle sont analysées en scindant sur le **premier** `/`. Si l’ID du modèle contient `/` (style OpenRouter), incluez le préfixe fournisseur (exemple : `openrouter/moonshotai/kimi-k2`).
* Si vous omettez le fournisseur, OpenClaw considère l’entrée comme un alias ou un modèle pour le **fournisseur par défaut** (ne fonctionne que lorsqu’il n’y a pas de `/` dans l’ID du modèle).

<div id="models-status">
  ### `models status`
</div>

Options :

* `--json`
* `--plain`
* `--check` (code de sortie 1 = expiré/manquant, 2 = bientôt expiré)
* `--probe` (sondage en direct des profils d’authentification configurés)
* `--probe-provider <name>` (sonder un fournisseur)
* `--probe-profile <id>` (répéter ou fournir une liste d’identifiants de profil séparés par des virgules)
* `--probe-timeout <ms>`
* `--probe-concurrency <n>`
* `--probe-max-tokens <n>`
* `--agent <id>` (identifiant d’agent configuré ; remplace `OPENCLAW_AGENT_DIR`/`PI_CODING_AGENT_DIR`)

<div id="aliases-fallbacks">
  ## Alias et solutions de repli
</div>

```bash
openclaw models aliases list
openclaw models fallbacks list
```

<div id="auth-profiles">
  ## Profils d&#39;authentification
</div>

```bash
openclaw models auth add
openclaw models auth login --provider <id>
openclaw models auth setup-token
openclaw models auth paste-token
```

`models auth login` exécute le flux d’authentification (OAuth/API key) d’un plugin de fournisseur. Utilisez
`openclaw plugins list` pour voir quels fournisseurs sont installés.

Remarques :

* `setup-token` vous invite à saisir une valeur de setup-token (générez-la avec `claude setup-token` sur n’importe quelle machine).
* `paste-token` accepte une chaîne de caractères de jeton générée ailleurs ou par automatisation.
