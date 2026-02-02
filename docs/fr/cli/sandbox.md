---
title: Sandbox CLI
summary: "Gérer les conteneurs de sandbox et examiner la politique de sandbox appliquée"
read_when: "Vous gérez des conteneurs de sandbox ou dépannez le comportement de la sandbox et des politiques d’outils."
status: active
---

<div id="sandbox-cli">
  # CLI sandbox
</div>

Gérez des conteneurs sandbox Docker pour exécuter des agents de manière isolée.

<div id="overview">
  ## Vue d’ensemble
</div>

OpenClaw peut exécuter des agents dans des conteneurs Docker isolés pour des raisons de sécurité. Les commandes `sandbox` vous permettent de gérer ces conteneurs, en particulier après des mises à jour ou des modifications de configuration.

<div id="commands">
  ## Commandes
</div>

<div id="openclaw-sandbox-explain">
  ### `openclaw sandbox explain`
</div>

Inspecte le mode de sandbox **effectif**, la portée, l&#39;accès à l&#39;espace de travail, la politique d&#39;utilisation des outils dans la sandbox et les garde-fous renforcés (avec les chemins de clés de configuration permettant de les corriger).

```bash
openclaw sandbox explain
openclaw sandbox explain --session agent:main:main
openclaw sandbox explain --agent work
openclaw sandbox explain --json
```

<div id="openclaw-sandbox-list">
  ### `openclaw sandbox list`
</div>

Liste tous les conteneurs sandbox avec leur état et leur configuration.

```bash
openclaw sandbox list
openclaw sandbox list --browser  # Liste uniquement les conteneurs navigateur
openclaw sandbox list --json     # Sortie JSON
```

**La sortie inclut :**

* Nom et état du conteneur (en cours d’exécution/arrêté)
* Image Docker et indication de sa conformité à la configuration
* Âge (temps écoulé depuis la création)
* Temps d’inactivité (temps écoulé depuis la dernière utilisation)
* Session/agent associé

<div id="openclaw-sandbox-recreate">
  ### `openclaw sandbox recreate`
</div>

Supprime les conteneurs de sandbox afin de les recréer avec des images/configurations à jour.

```bash
openclaw sandbox recreate --all                # Recréer tous les conteneurs
openclaw sandbox recreate --session main       # Session spécifique
openclaw sandbox recreate --agent mybot        # Agent spécifique
openclaw sandbox recreate --browser            # Conteneurs de navigateur uniquement
openclaw sandbox recreate --all --force        # Ignorer la confirmation
```

**Options :**

* `--all` : recréer tous les conteneurs sandbox
* `--session <key>` : recréer le conteneur pour une session spécifique
* `--agent <id>` : recréer les conteneurs pour un agent spécifique
* `--browser` : ne recréer que les conteneurs de navigateur
* `--force` : ignorer l’invite de confirmation

**Important :** les conteneurs sont recréés automatiquement lors de la prochaine utilisation de l’agent.

<div id="use-cases">
  ## Cas d’utilisation
</div>

<div id="after-updating-docker-images">
  ### Après avoir mis à jour les images Docker
</div>

```bash
# Récupérer la nouvelle image
docker pull openclaw-sandbox:latest
docker tag openclaw-sandbox:latest openclaw-sandbox:bookworm-slim

# Mettre à jour la config pour utiliser la nouvelle image
# Modifier la config : agents.defaults.sandbox.docker.image (ou agents.list[].sandbox.docker.image)

# Recréer les conteneurs
openclaw sandbox recreate --all
```

<div id="after-changing-sandbox-configuration">
  ### Après modification de la configuration du sandbox
</div>

```bash
# Modifiez la config : agents.defaults.sandbox.* (ou agents.list[].sandbox.*)

# Recréez pour appliquer la nouvelle config
openclaw sandbox recreate --all
```

<div id="after-changing-setupcommand">
  ### Après modification de setupCommand
</div>

```bash
openclaw sandbox recreate --all
# ou pour un seul agent :
openclaw sandbox recreate --agent family
```

<div id="for-a-specific-agent-only">
  ### Pour un agent donné uniquement
</div>

```bash
# Mettre à jour uniquement les conteneurs d'un agent
openclaw sandbox recreate --agent alfred
```

<div id="why-is-this-needed">
  ## Pourquoi est-ce nécessaire ?
</div>

**Problème :** Lorsque vous mettez à jour les images Docker de sandbox ou la configuration :

* Les conteneurs existants continuent de tourner avec les anciens paramètres
* Les conteneurs ne sont purgés qu’après 24 h d’inactivité
* Les agents utilisés régulièrement conservent leurs anciens conteneurs indéfiniment

**Solution :** Utilisez `openclaw sandbox recreate` pour forcer la suppression des anciens conteneurs. Ils seront recréés automatiquement avec les paramètres actuels lors de leur prochaine utilisation.

Astuce : privilégiez `openclaw sandbox recreate` à un `docker rm` manuel. Cette commande utilise le schéma de nommage des conteneurs du Gateway et évite les incohérences lorsque les clés scope/session changent.

<div id="configuration">
  ## Configuration
</div>

Les paramètres de sandbox se trouvent dans `~/.openclaw/openclaw.json` sous `agents.defaults.sandbox` (les surcharges par agent se définissent dans `agents.list[].sandbox`) :

```jsonc
{
  "agents": {
    "defaults": {
      "sandbox": {
        "mode": "all",                    // off, non-main, all
        "scope": "agent",                 // session, agent, shared
        "docker": {
          "image": "openclaw-sandbox:bookworm-slim",
          "containerPrefix": "openclaw-sbx-"
          // ... more Docker options
        },
        "prune": {
          "idleHours": 24,               // Nettoyage auto après 24h d'inactivité
          "maxAgeDays": 7                // Auto-prune after 7 days
        }
      }
    }
  }
}
```

<div id="see-also">
  ## Voir également
</div>

* [Documentation sur la sandbox](/fr/gateway/sandboxing)
* [Configuration de l&#39;Agent](/fr/concepts/agent-workspace)
* [Commande doctor](/fr/gateway/doctor) - Vérifier la configuration de la sandbox