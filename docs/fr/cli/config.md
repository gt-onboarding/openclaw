---
title: Configuration
summary: "Référence de la CLI pour `openclaw config` (lire/définir/réinitialiser des valeurs de configuration)"
read_when:
  - Vous voulez lire ou modifier la configuration de façon non interactive
---

<div id="openclaw-config">
  # `openclaw config`
</div>

Outils de configuration : consulter/définir/supprimer des valeurs par chemin. Exécutez cette commande sans sous-commande pour lancer l’assistant de configuration (équivaut à `openclaw configure`).

<div id="examples">
  ## Exemples
</div>

```bash
openclaw config get browser.executablePath
openclaw config set browser.executablePath "/usr/bin/google-chrome"
openclaw config set agents.defaults.heartbeat.every "2h"
openclaw config set agents.list[0].tools.exec.node "node-id-or-name"
openclaw config unset tools.web.search.apiKey
```


<div id="paths">
  ## Chemins
</div>

Les chemins utilisent la notation à points ou entre crochets :

```bash
openclaw config get agents.defaults.workspace
openclaw config get agents.list[0].id
```

Utilisez l’index de la liste d’agents pour cibler un agent précis :

```bash
openclaw config get agents.list
openclaw config set agents.list[1].tools.exec.node "node-id-or-name"
```


<div id="values">
  ## Valeurs
</div>

Les valeurs sont analysées en JSON5 lorsque c’est possible ; sinon, elles sont traitées comme des chaînes de caractères.
Utilisez `--json` pour imposer l’analyse au format JSON5.

```bash
openclaw config set agents.defaults.heartbeat.every "0m"
openclaw config set gateway.port 19001 --json
openclaw config set channels.whatsapp.groups '["*"]' --json
```

Redémarrez le Gateway après les modifications.
