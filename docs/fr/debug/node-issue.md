---
title: Problème avec Node
summary: Notes et solutions de contournement pour le plantage Node + tsx « __name is not a function »
read_when:
  - Débogage de scripts de développement réservés à Node ou d'échecs du mode watch
  - Analyse des plantages du chargeur tsx/esbuild dans OpenClaw
---

<div id="node-tsx-__name-is-not-a-function-crash">
  # Plantage de Node + tsx « __name is not a function »
</div>

<div id="summary">
  ## Résumé
</div>

L’exécution d’OpenClaw via Node avec `tsx` échoue au démarrage avec l’erreur suivante :

```
[openclaw] Failed to start CLI: TypeError: __name is not a function
    at createSubsystemLogger (.../src/logging/subsystem.ts:203:25)
    at .../src/agents/auth-profiles/constants.ts:25:20
```

Cela a commencé après avoir remplacé les scripts de développement Bun par `tsx` (commit `2871657e`, 2026-01-06). Le même chemin d’exécution fonctionnait avec Bun.


<div id="environment">
  ## Environnement
</div>

- Node : v25.x (constaté sur v25.3.0)
- tsx : 4.21.0
- OS : macOS (probablement reproductible aussi sur d'autres plates-formes exécutant Node 25)

<div id="repro-node-only">
  ## Repro (nœud uniquement)
</div>

```bash
# dans la racine du dépôt
node --version
pnpm install
node --import tsx src/entry.ts status
```


<div id="minimal-repro-in-repo">
  ## Exemple minimal reproductible dans le dépôt
</div>

```bash
node --import tsx scripts/repro/tsx-name-repro.ts
```


<div id="node-version-check">
  ## Vérification de la version de Node
</div>

- Node 25.3.0 : échec
- Node 22.22.0 (Homebrew `node@22`) : échec
- Node 24 : pas encore installé ici ; à vérifier

<div id="notes-hypothesis">
  ## Notes / hypothèses
</div>

- `tsx` utilise esbuild pour transformer TS/ESM. L’option `keepNames` d’esbuild émet un helper `__name` et enveloppe les définitions de fonctions avec `__name(...)`.
- Le crash indique que `__name` existe mais n’est pas une fonction à l’exécution, ce qui implique que le helper est manquant ou a été écrasé pour ce module dans le chemin de chargement du chargeur Node 25.
- Des problèmes similaires avec le helper `__name` ont été signalés dans d’autres outils utilisant esbuild lorsque le helper est manquant ou réécrit.

<div id="regression-history">
  ## Historique des régressions
</div>

- `2871657e` (2026-01-06) : les scripts sont passés de Bun à tsx afin de rendre Bun optionnel.
- Avant cela (chemin de code Bun), `openclaw status` et `gateway:watch` fonctionnaient.

<div id="workarounds">
  ## Solutions de contournement
</div>

- Utiliser Bun pour les scripts de développement (repli temporaire actuel).
- Utiliser Node + `tsc` en mode watch, puis exécuter le code compilé :
  ```bash
  pnpm exec tsc --watch --preserveWatchOutput
  node --watch openclaw.mjs status
  ```
- Confirmé en local : `pnpm exec tsc -p tsconfig.json` + `node openclaw.mjs status` fonctionne avec Node 25.
- Désactiver `esbuild keepNames` dans le chargeur TypeScript si possible (empêche l’insertion du helper `__name`) ; `tsx` ne l’expose pas actuellement.
- Tester Node LTS (22/24) avec `tsx` pour vérifier si le problème est spécifique à Node 25.

<div id="references">
  ## Références
</div>

- https://opennext.js.org/cloudflare/howtos/keep_names
- https://esbuild.github.io/api/#keep-names
- https://github.com/evanw/esbuild/issues/1031

<div id="next-steps">
  ## Prochaines étapes
</div>

- Reproduire sous Node 22/24 pour confirmer une régression sous Node 25.
- Tester la version nightly de `tsx` ou figer sur une version antérieure s’il existe une régression connue.
- Si le problème se reproduit sous Node LTS, ouvrir un ticket de reproduction minimal en amont avec la trace de pile `__name`.