---
title: Nœuds
summary: "Référence CLI pour `openclaw nodes` (liste/état/approbation/appel, caméra/canevas/écran)"
read_when:
  - Vous gérez des nœuds appairés (caméras, écran, canevas)
  - Vous devez approuver des requêtes ou invoquer des commandes des nœuds
---

<div id="openclaw-nodes">
  # `openclaw nodes`
</div>

Gérer les nœuds (appareils) appairés et appeler les capacités des nœuds.

Voir aussi :

* Vue d’ensemble des nœuds : [Nodes](/fr/nodes)
* Caméra : [Camera nodes](/fr/nodes/camera)
* Images : [Image nodes](/fr/nodes/images)

Options courantes :

* `--url`, `--token`, `--timeout`, `--json`

<div id="common-commands">
  ## Commandes courantes
</div>

```bash
openclaw nodes list
openclaw nodes list --connected
openclaw nodes list --last-connected 24h
openclaw nodes pending
openclaw nodes approve <requestId>
openclaw nodes status
openclaw nodes status --connected
openclaw nodes status --last-connected 24h
```

`nodes list` affiche les tableaux en attente/appairés. Les lignes appairées incluent l’intervalle depuis la dernière connexion (Last Connect).
Utilisez `--connected` pour n’afficher que les nœuds actuellement connectés. Utilisez `--last-connected <duration>` pour
filtrer les nœuds qui se sont connectés au cours d’une durée donnée (par ex. `24h`, `7d`).

<div id="invoke-run">
  ## Invocation / exécution
</div>

```bash
openclaw nodes invoke --node <id|name|ip> --command <command> --params <json>
openclaw nodes run --node <id|name|ip> <command...>
openclaw nodes run --raw "git status"
openclaw nodes run --agent main --node <id|name|ip> --raw "git status"
```

Options d&#39;invocation :

* `--params <json>`: chaîne JSON représentant un objet (par défaut `{}`).
* `--invoke-timeout <ms>`: délai d&#39;expiration d&#39;invocation du nœud (par défaut `15000`).
* `--idempotency-key <key>`: clé d&#39;idempotence facultative.

<div id="exec-style-defaults">
  ### Valeurs par défaut de type exec
</div>

`nodes run` reflète le comportement exec du modèle (valeurs par défaut + approbations) :

* Lit `tools.exec.*` (plus les surcharges `agents.list[].tools.exec.*`).
* Utilise les approbations exec (`exec.approval.request`) avant d’invoquer `system.run`.
* `--node` peut être omis lorsque `tools.exec.node` est défini.
* Nécessite un nœud qui expose `system.run` (app compagnon macOS ou hôte de nœud sans interface).

Flags :

* `--cwd <path>` : répertoire de travail.
* `--env <key=val>` : surcharge d’environnement (option répétable).
* `--command-timeout <ms>` : délai d’expiration de la commande.
* `--invoke-timeout <ms>` : délai d’expiration d’invocation du nœud (par défaut `30000`).
* `--needs-screen-recording` : requiert l’autorisation d’enregistrement de l’écran.
* `--raw <command>` : exécute une chaîne shell (`/bin/sh -lc` ou `cmd.exe /c`).
* `--agent <id>` : approbations/listes d’autorisation propres à l’agent (par défaut, l’agent configuré).
* `--ask <off|on-miss|always>`, `--security <deny|allowlist|full>` : surcharges.