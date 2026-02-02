---
title: Bun
summary: "Workflow Bun (expérimental)\u00A0: installations et pièges par rapport à pnpm"
read_when:
  - Vous voulez la boucle de développement locale la plus rapide (bun + watch)
  - Vous rencontrez des problèmes d'installation, de patch ou de scripts de cycle de vie avec Bun
---

<div id="bun-experimental">
  # Bun (expérimental)
</div>

Objectif : exécuter ce dépôt avec **Bun** (facultatif, non recommandé pour WhatsApp/Telegram)
sans s’écarter des workflows pnpm.

⚠️ **Non recommandé pour le runtime du Gateway** (bugs WhatsApp/Telegram). Utilisez Node pour la production.

<div id="status">
  ## Statut
</div>

- Bun est un runtime local optionnel pour exécuter TypeScript directement (`bun run …`, `bun --watch …`).
- `pnpm` est l’outil par défaut pour les builds et reste entièrement pris en charge (et utilisé par certains outils de documentation).
- Bun ne peut pas utiliser `pnpm-lock.yaml` et l’ignore.

<div id="install">
  ## Installation
</div>

Par défaut :

```sh
bun install
```

Remarque : `bun.lock`/`bun.lockb` sont ignorés par Git, donc le dépôt ne subit pas de modifications inutiles dans un cas comme dans l’autre. Si vous voulez *aucune écriture dans le fichier de verrouillage* :

```sh
bun install --no-save
```


<div id="build-test-bun">
  ## Build / Test (Bun)
</div>

```sh
bun run build
bun run vitest run
```


<div id="bun-lifecycle-scripts-blocked-by-default">
  ## Scripts de cycle de vie Bun (bloqués par défaut)
</div>

Bun peut bloquer les scripts de cycle de vie des dépendances à moins qu’ils ne soient explicitement approuvés (`bun pm untrusted` / `bun pm trust`).
Pour ce dépôt, les scripts fréquemment bloqués ne sont pas nécessaires :

* `@whiskeysockets/baileys` `preinstall` : vérifie que la version majeure de Node &gt;= 20 (nous utilisons Node 22+).
* `protobufjs` `postinstall` : émet des avertissements concernant des schémas de version incompatibles (ne produit aucun artefact de build).

Si vous rencontrez un véritable problème à l’exécution qui nécessite ces scripts, faites-leur explicitement confiance :

```sh
bun pm trust @whiskeysockets/baileys protobufjs
```


<div id="caveats">
  ## Points à noter
</div>

- Certains scripts utilisent encore pnpm en dur (par exemple `docs:build`, `ui:*`, `protocol:check`). Exécute-les avec pnpm pour l’instant.