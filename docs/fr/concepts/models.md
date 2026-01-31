---
title: Modèles
summary: "CLI des modèles : lister, définir, alias, mécanismes de repli, analyse, statut"
read_when:
  - Ajout ou modification de la CLI des modèles (models list/set/scan/aliases/fallbacks)
  - Modification du comportement de repli des modèles ou de l’UX de sélection de modèle
  - Mise à jour des sondes d’analyse des modèles (tools/images)
---

<div id="models-cli">
  # CLI des modèles
</div>

Voir [/concepts/model-failover](/fr/concepts/model-failover) pour la rotation des profils d’authentification, la gestion des délais d’attente et la façon dont cela interagit avec les mécanismes de repli.
Aperçu rapide des fournisseurs + exemples : [/concepts/model-providers](/fr/concepts/model-providers).

<div id="how-model-selection-works">
  ## Fonctionnement de la sélection de modèles
</div>

OpenClaw sélectionne les modèles dans cet ordre :

1. Modèle **primaire** (`agents.defaults.model.primary` ou `agents.defaults.model`).
2. **Modèles de repli** dans `agents.defaults.model.fallbacks` (dans l’ordre).
3. Le **basculement d’authentification du fournisseur** se produit côté fournisseur avant de passer au
   modèle suivant.

À noter :

* `agents.defaults.models` est la liste d’autorisation / le catalogue des modèles qu’OpenClaw peut utiliser (plus les alias).
* `agents.defaults.imageModel` est utilisé **uniquement lorsque** le modèle primaire ne peut pas accepter d’images.
* Les valeurs par défaut spécifiques à un agent peuvent remplacer `agents.defaults.model` via `agents.list[].model` plus les bindings (voir [/concepts/multi-agent](/fr/concepts/multi-agent)).

<div id="quick-model-picks-anecdotal">
  ## Choix rapides de modèles (anecdotique)
</div>

* **GLM** : un peu meilleur pour la programmation et l’appel d’outils.
* **MiniMax** : meilleur pour la rédaction et l’ambiance générale.

<div id="setup-wizard-recommended">
  ## Assistant de configuration (recommandé)
</div>

Si vous ne souhaitez pas modifier la configuration à la main, exécutez l’assistant de démarrage :

```bash
openclaw onboard
```

Il peut configurer les modèles et l’authentification pour les fournisseurs courants, y compris **abonnement OpenAI Code (Codex)
** (OAuth) et **Anthropic** (clé API recommandée ; la commande `claude
setup-token` est également prise en charge).

<div id="config-keys-overview">
  ## Clés de configuration (vue d’ensemble)
</div>

* `agents.defaults.model.primary` et `agents.defaults.model.fallbacks`
* `agents.defaults.imageModel.primary` et `agents.defaults.imageModel.fallbacks`
* `agents.defaults.models` (liste d’autorisation + alias + paramètres de fournisseur)
* `models.providers` (fournisseurs personnalisés écrits dans `models.json`)

Les références de modèles sont normalisées en minuscules. Les alias de fournisseur, comme `z.ai/*`, sont normalisés en
`zai/*`.

Des exemples de configuration de fournisseur (y compris OpenCode Zen) sont disponibles dans
[/gateway/configuration](/fr/gateway/configuration#opencode-zen-multi-model-proxy).

<div id="model-is-not-allowed-and-why-replies-stop">
  ## « Model is not allowed » (et pourquoi les réponses s’arrêtent)
</div>

Si `agents.defaults.models` est défini, il devient la **liste d’autorisation** pour `/model` et pour
les surcharges de session. Lorsqu’un utilisateur sélectionne un modèle qui n’est pas dans cette liste d’autorisation,
OpenClaw renvoie :

```
Model "provider/model" is not allowed. Use /model to list available models.
```

Cela se produit **avant** qu’une réponse normale ne soit générée, donc le message peut donner l’impression
qu’il « ne répond pas ». Pour corriger cela, vous pouvez soit :

* Ajouter le modèle à `agents.defaults.models`, ou
* Effacer la liste d’autorisation (supprimer `agents.defaults.models`), ou
* Choisir un modèle depuis `/model list`.

Exemple de configuration de liste d’autorisation :

```json5
{
  agent: {
    model: { primary: "anthropic/claude-sonnet-4-5" },
    models: {
      "anthropic/claude-sonnet-4-5": { alias: "Sonnet" },
      "anthropic/claude-opus-4-5": { alias: "Opus" }
    }
  }
}
```

<div id="switching-models-in-chat-model">
  ## Changer de modèle dans le chat (`/model`)
</div>

Vous pouvez changer de modèle pour la session en cours sans redémarrer :

```
/model
/model list
/model 3
/model openai/gpt-5.2
/model status
```

Notes :

* `/model` (et `/model list`) est un sélecteur compact et numéroté (famille de modèles + fournisseurs disponibles).
* `/model <#>` sélectionne un élément dans ce sélecteur.
* `/model status` est la vue détaillée (candidats d’authentification et, lorsqu’il est configuré, point de terminaison du fournisseur `baseUrl` + mode `api`).
* Les références de modèle sont analysées en scindant sur le **premier** `/`. Utilisez `provider/model` lorsque vous saisissez `/model <ref>`.
* Si l’ID du modèle contient lui-même `/` (style OpenRouter), vous devez inclure le préfixe de fournisseur (exemple : `/model openrouter/moonshotai/kimi-k2`).
* Si vous omettez le fournisseur, OpenClaw traite la saisie comme un alias ou un modèle pour le **fournisseur par défaut** (ne fonctionne que lorsqu’il n’y a pas de `/` dans l’ID du modèle).

Comportement et configuration complets de la commande : [Slash commands](/fr/tools/slash-commands).

<div id="cli-commands">
  ## Commandes de la CLI
</div>

```bash
openclaw models list
openclaw models status
openclaw models set <fournisseur/modèle>
openclaw models set-image <fournisseur/modèle>

openclaw models aliases list
openclaw models aliases add <alias> <fournisseur/modèle>
openclaw models aliases remove <alias>

openclaw models fallbacks list
openclaw models fallbacks add <fournisseur/modèle>
openclaw models fallbacks remove <fournisseur/modèle>
openclaw models fallbacks clear

openclaw models image-fallbacks list
openclaw models image-fallbacks add <fournisseur/modèle>
openclaw models image-fallbacks remove <fournisseur/modèle>
openclaw models image-fallbacks clear
```

`openclaw models` (sans sous-commande) est un alias de `models status`.

<div id="models-list">
  ### `models list`
</div>

Affiche les modèles configurés par défaut. Options utiles :

* `--all` : catalogue complet
* `--local` : fournisseurs locaux uniquement
* `--provider <name>` : filtrer par fournisseur
* `--plain` : un modèle par ligne
* `--json` : sortie lisible par une machine

<div id="models-status">
  ### `models status`
</div>

Affiche le modèle principal résolu, les modèles de repli, le modèle d’images et un aperçu de l’authentification
des fournisseurs configurés. Il met également en évidence l’état d’expiration OAuth pour les profils trouvés
dans le store d’authentification (alerte dans les 24 h par défaut). `--plain` affiche uniquement le
modèle principal résolu.
L’état OAuth est toujours affiché (et inclus dans la sortie `--json`). Si un fournisseur configuré n’a pas
d’identifiants, `models status` affiche une section **Missing auth**.
Le JSON inclut `auth.oauth` (fenêtre d’alerte + profils) et `auth.providers`
(authentification effective par fournisseur).
Utilisez `--check` pour l’automatisation (code de sortie `1` en cas d’authentification manquante/expirée, `2` en cas d’expiration imminente).

L’authentification Anthropic privilégiée est le jeton de configuration Claude Code CLI `setup-token` (exécutez-le n’importe où ; collez le jeton sur l’hôte Gateway si nécessaire) :

```bash
claude setup-token
openclaw models status
```

<div id="scanning-openrouter-free-models">
  ## Analyse (modèles gratuits OpenRouter)
</div>

`openclaw models scan` inspecte le **catalogue de modèles gratuits** d’OpenRouter et peut,
en option, sonder les modèles pour vérifier la prise en charge des outils et des images.

Options clés :

* `--no-probe` : ignorer les sondes en direct (métadonnées uniquement)
* `--min-params <b>` : taille minimale des paramètres (en milliards)
* `--max-age-days <days>` : ignorer les modèles plus anciens
* `--provider <name>` : filtre sur le préfixe du fournisseur
* `--max-candidates <n>` : taille de la liste de repli
* `--set-default` : définit `agents.defaults.model.primary` sur la première sélection
* `--set-image` : définit `agents.defaults.imageModel.primary` sur la première sélection d’image

Les sondes nécessitent une clé API OpenRouter (provenant des profils d’authentification ou
de `OPENROUTER_API_KEY`). Sans clé, utilisez `--no-probe` pour ne lister que les candidats.

Les résultats de l’analyse sont classés selon :

1. Prise en charge des images
2. Latence des outils
3. Taille du contexte
4. Nombre de paramètres

Entrée

* Liste `/models` d’OpenRouter (filtre `:free`)
* Nécessite une clé API OpenRouter issue des profils d’authentification ou de `OPENROUTER_API_KEY` (voir [/environment](/fr/environment))
* Filtres optionnels : `--max-age-days`, `--min-params`, `--provider`, `--max-candidates`
* Paramètres de sondage : `--timeout`, `--concurrency`

Lorsqu’elle est exécutée dans un TTY, vous pouvez sélectionner les modèles de repli de manière interactive. En mode non interactif, passez `--yes` pour accepter les valeurs par défaut.

<div id="models-registry-modelsjson">
  ## Registre des modèles (`models.json`)
</div>

Les fournisseurs personnalisés dans `models.providers` sont enregistrés dans `models.json` dans le
répertoire de l’agent (par défaut `~/.openclaw/agents/<agentId>/models.json`). Ce fichier
est fusionné par défaut, sauf si `models.mode` est défini sur `replace`.