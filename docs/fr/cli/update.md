---
title: Mise à jour
summary: "Référence CLI pour `openclaw update` (mise à jour du code source relativement sûre + redémarrage automatique du Gateway)"
read_when:
  - Vous voulez mettre à jour une copie locale du code source en toute sécurité
  - Vous avez besoin de comprendre le comportement de la forme abrégée `--update`
---

<div id="openclaw-update">
  # `openclaw update`
</div>

Mettez à jour OpenClaw en toute sécurité et basculez entre les canaux stable/beta/dev.

Si vous avez installé via **npm/pnpm** (installation globale, sans métadonnées git), les mises à jour se font via le gestionnaire de paquets, comme expliqué dans [Mise à jour](/fr/install/updating).

<div id="usage">
  ## Utilisation
</div>

```bash
openclaw update
openclaw update status
openclaw update wizard
openclaw update --channel beta
openclaw update --channel dev
openclaw update --tag beta
openclaw update --no-restart
openclaw update --json
openclaw --update
```

<div id="options">
  ## Options
</div>

* `--no-restart` : ne pas redémarrer le service Gateway après une mise à jour réussie.
* `--channel <stable|beta|dev>` : définir le canal de mise à jour (git + npm ; conservé dans la configuration).
* `--tag <dist-tag|version>` : remplacer le dist-tag ou la version npm uniquement pour cette mise à jour.
* `--json` : afficher un JSON `UpdateRunResult` lisible par machine.
* `--timeout <seconds>` : délai d&#39;expiration pour chaque étape (1200 s par défaut).

Note : les retours à une version antérieure nécessitent une confirmation, car les versions plus anciennes peuvent rendre la configuration incompatible.

<div id="update-status">
  ## `update status`
</div>

Affiche le canal de mise à jour actif + le tag/branche/SHA Git (pour les checkouts du dépôt source), ainsi que la disponibilité d’une mise à jour.

```bash
openclaw update status
openclaw update status --json
openclaw update status --timeout 10
```

Options :

* `--json` : affiche un statut JSON lisible par une machine.
* `--timeout <seconds>` : délai d’expiration pour les vérifications (3 s par défaut).

<div id="update-wizard">
  ## `update wizard`
</div>

Procédure interactive permettant de choisir un canal de mise à jour et de confirmer s’il faut redémarrer le Gateway
après la mise à jour (par défaut, un redémarrage est effectué). Si vous sélectionnez `dev` sans clone Git, il
propose d’en créer un.

<div id="what-it-does">
  ## Ce que fait cette commande
</div>

Lorsque vous changez explicitement de canal (`--channel ...`), OpenClaw aligne
également la méthode d’installation :

* `dev` → garantit un checkout Git (par défaut : `~/openclaw`, modifiable via `OPENCLAW_GIT_DIR`),
  le met à jour et installe la CLI globale à partir de ce checkout.
* `stable`/`beta` → installe depuis npm en utilisant le dist-tag correspondant.

<div id="git-checkout-flow">
  ## Flux de checkout Git
</div>

Canaux :

* `stable` : checkout du dernier tag non bêta, puis build + doctor.
* `beta` : checkout du dernier tag `-beta`, puis build + doctor.
* `dev` : checkout de `main`, puis fetch + rebase.

Vue d’ensemble :

1. Nécessite un worktree propre (aucune modification non validée).
2. Bascule vers le canal sélectionné (tag ou branche).
3. Fait un fetch depuis l’upstream (dev uniquement).
4. Dev uniquement : préflight lint + build TypeScript dans un worktree temporaire ; si le dernier commit échoue, remonte jusqu’à 10 commits pour trouver le build le plus récent qui passe.
5. Rebase sur le commit sélectionné (dev uniquement).
6. Installe les dépendances (pnpm préféré ; npm en solution de repli).
7. Construit le projet puis la Control UI.
8. Exécute `openclaw doctor` comme contrôle final de « mise à jour sûre ».
9. Synchronise les plugins avec le canal actif (dev utilise les extensions intégrées ; stable/beta utilise npm) et met à jour les plugins installés via npm.

<div id="update-shorthand">
  ## Raccourci `--update`
</div>

`openclaw --update` est équivalent à `openclaw update` (utile pour les shells et les scripts de lancement).

<div id="see-also">
  ## Voir aussi
</div>

* `openclaw doctor` (propose d’exécuter une mise à jour préalable sur les clones Git)
* [Canaux de développement](/fr/install/development-channels)
* [Mise à jour](/fr/install/updating)
* [Référence de la CLI](/fr/cli)