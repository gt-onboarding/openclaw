---
title: PUBLICATION
summary: "Checklist de publication pas à pas pour npm et l'app macOS"
read_when:
  - Préparer une nouvelle version npm
  - Préparer une nouvelle version de l'app macOS
  - Vérifier les métadonnées avant la publication
---

<div id="release-checklist-npm-macos">
  # Checklist de publication (npm + macOS)
</div>

Utilisez `pnpm` (Node 22+) depuis la racine du dépôt. Assurez-vous que l’arbre de travail est propre avant de créer le tag et de publier.

<div id="operator-trigger">
  ## Déclenchement par l’opérateur
</div>

Quand l’opérateur dit « release », exécute immédiatement cette pré‑vérification (aucune question supplémentaire sauf en cas de blocage) :

* Lis ce document et `docs/platforms/mac/release.md`.
* Charge l’environnement depuis `~/.profile` et confirme que `SPARKLE_PRIVATE_KEY_FILE` + les variables App Store Connect sont définies (`SPARKLE_PRIVATE_KEY_FILE` doit se trouver dans `~/.profile`).
* Utilise les clés Sparkle depuis `~/Library/CloudStorage/Dropbox/Backup/Sparkle` si nécessaire.

1. **Version &amp; métadonnées**

* [ ] Incrémente la version dans `package.json` (par ex. `2026.1.29`).
* [ ] Lance `pnpm plugins:sync` pour aligner les versions des paquets d’extension + leurs changelogs.
* [ ] Mets à jour les chaînes de version de la CLI : [`src/cli/program.ts`](https://github.com/openclaw/openclaw/blob/main/src/cli/program.ts) et l’agent utilisateur Baileys dans [`src/provider-web.ts`](https://github.com/openclaw/openclaw/blob/main/src/provider-web.ts).
* [ ] Confirme les métadonnées du paquet (name, description, repository, keywords, license) et que le mappage `bin` pointe vers [`openclaw.mjs`](https://github.com/openclaw/openclaw/blob/main/openclaw.mjs) pour `openclaw`.
* [ ] Si les dépendances ont changé, exécute `pnpm install` pour que `pnpm-lock.yaml` soit à jour.

2. **Build &amp; artefacts**

* [ ] Si les entrées A2UI ont changé, exécute `pnpm canvas:a2ui:bundle` et commite tout fichier [`src/canvas-host/a2ui/a2ui.bundle.js`](https://github.com/openclaw/openclaw/blob/main/src/canvas-host/a2ui/a2ui.bundle.js) mis à jour.
* [ ] `pnpm run build` (régénère `dist/`).
* [ ] Vérifie que la clé `files` du paquet npm inclut tous les dossiers `dist/*` requis (notamment `dist/node-host/**` et `dist/acp/**` pour le nœud headless + la CLI ACP).
* [ ] Confirme que `dist/build-info.json` existe et inclut le hash `commit` attendu (la bannière CLI l’utilise pour les installations npm).
* [ ] Optionnel : `npm pack --pack-destination /tmp` après le build ; inspecte le contenu de l’archive tar et garde‑la sous la main pour la publication GitHub (ne la commite **pas**).

3. **Changelog &amp; docs**

* [ ] Mets à jour `CHANGELOG.md` avec les points clés visibles par l’utilisateur (crée le fichier s’il n’existe pas) ; garde les entrées strictement décroissantes par version.
* [ ] Assure‑toi que les exemples/options du README correspondent au comportement actuel de la CLI (en particulier les nouvelles commandes ou options).

4. **Validation**

* [ ] `pnpm lint`
* [ ] `pnpm test` (ou `pnpm test:coverage` si tu as besoin d’un rapport de couverture)
* [ ] `pnpm run build` (dernier sanity check après les tests)
* [ ] `pnpm release:check` (vérifie le contenu du pack npm)
* [ ] `OPENCLAW_INSTALL_SMOKE_SKIP_NONROOT=1 pnpm test:install:smoke` (smoke test d’installation Docker, voie rapide ; requis avant la release)
  * Si la release npm immédiatement précédente est connue comme défectueuse, définis `OPENCLAW_INSTALL_SMOKE_PREVIOUS=<last-good-version>` ou `OPENCLAW_INSTALL_SMOKE_SKIP_PREVIOUS=1` pour l’étape de préinstallation.
* [ ] (Optionnel) Smoke test complet de l’installateur (ajoute la couverture non‑root + CLI) : `pnpm test:install:smoke`
* [ ] (Optionnel) E2E de l’installateur (Docker, exécute `curl -fsSL https://openclaw.bot/install.sh | bash`, effectue la procédure d’onboarding, puis lance des appels d’outils réels) :
  * `pnpm test:install:e2e:openai` (nécessite `OPENAI_API_KEY`)
  * `pnpm test:install:e2e:anthropic` (nécessite `ANTHROPIC_API_KEY`)
  * `pnpm test:install:e2e` (nécessite les deux clés ; exécute les deux fournisseurs)
* [ ] (Optionnel) Vérifie ponctuellement le Gateway web si tes changements affectent les chemins d’envoi/réception.

5. **Application macOS (Sparkle)**

* [ ] Générer et signer l’app macOS, puis la compresser en ZIP pour la distribution.
* [ ] Générer l’appcast Sparkle (notes HTML via [`scripts/make_appcast.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/make_appcast.sh)) et mettre à jour `appcast.xml`.
* [ ] Conserver le ZIP de l’app (et le ZIP dSYM facultatif) prêt à être joint à la release GitHub.
* [ ] Suivre [macOS release](/fr/platforms/mac/release) pour les commandes exactes et les variables d’environnement requises.
  * `APP_BUILD` doit être numérique et strictement croissant (pas de `-beta`) pour que Sparkle compare correctement les versions.
  * En cas de notarisation, utiliser le profil de trousseau `openclaw-notary` créé à partir des variables d’environnement de l’API App Store Connect (voir [macOS release](/fr/platforms/mac/release)).

6. **Publication (npm)**

* [ ] S’assurer que l’état de Git est propre ; effectuer les commits et les push nécessaires.
* [ ] Lancer `npm login` (vérifier la 2FA) si nécessaire.
* [ ] Lancer `npm publish --access public` (utiliser `--tag beta` pour les préversions).
* [ ] Vérifier le registre npm : `npm view openclaw version`, `npm view openclaw dist-tags`, et `npx -y openclaw@X.Y.Z --version` (ou `--help`).

<div id="troubleshooting-notes-from-200-beta2-release">
  ### Dépannage (notes de la version 2.0.0-beta2)
</div>

* **`npm pack/publish` se bloque ou produit une archive tar énorme** : le bundle de l’app macOS dans `dist/OpenClaw.app` (et les zips de release) est embarqué dans le package. Corrige ce problème en définissant explicitement les contenus à publier via `files` dans `package.json` (inclure les sous-répertoires de dist, la doc, les compétences ; exclure les bundles d’app). Vérifie avec `npm pack --dry-run` que `dist/OpenClaw.app` n’est pas listé.
* **Boucle d’authentification web npm pour les dist-tags** : utilise l’authentification héritée pour obtenir une invite OTP :
  * `NPM_CONFIG_AUTH_TYPE=legacy npm dist-tag add openclaw@X.Y.Z latest`
* **La vérification `npx` échoue avec `ECOMPROMISED: Lock compromised`** : réessaie avec un cache neuf :
  * `NPM_CONFIG_CACHE=/tmp/npm-cache-$(date +%s) npx -y openclaw@X.Y.Z --version`
* **Le tag doit être repointé après un correctif tardif** : mets le tag à jour de force et pousse-le, puis vérifie que les artefacts de la release GitHub correspondent toujours :
  * `git tag -f vX.Y.Z && git push -f origin vX.Y.Z`

7. **Release GitHub + appcast**

* [ ] Taguer et pousser : `git tag vX.Y.Z && git push origin vX.Y.Z` (ou `git push --tags`).
* [ ] Créer/actualiser la release GitHub pour `vX.Y.Z` avec **le titre `openclaw X.Y.Z`** (pas seulement le tag) ; le corps doit inclure la section de changelog **complète** pour cette version (Points forts + Changements + Corrections), en ligne (aucun lien brut), et **ne doit pas répéter le titre à l’intérieur du corps**.
* [ ] Joindre les artefacts : tarball `npm pack` (optionnel), `OpenClaw-X.Y.Z.zip` et `OpenClaw-X.Y.Z.dSYM.zip` (si généré).
* [ ] Commiter le `appcast.xml` mis à jour et le pousser (Sparkle consomme le flux depuis main).
* [ ] Depuis un répertoire temporaire propre (sans `package.json`), exécuter `npx -y openclaw@X.Y.Z send --help` pour confirmer que l’installation et les points d’entrée CLI fonctionnent.
* [ ] Annoncer/partager les notes de version.

<div id="plugin-publish-scope-npm">
  ## Portée de publication des plugins (npm)
</div>

Nous ne publions que les **plugins npm existants** sous la portée `@openclaw/*`. Les plugins
intégrés qui ne sont pas sur npm restent **uniquement dans l’arborescence sur disque** (toujours livrés dans
`extensions/**`).

Procédure pour établir la liste :

1. Exécuter `npm search @openclaw --json` et récupérer les noms des packages.
2. Comparer avec les noms dans `extensions/*/package.json`.
3. Ne publier que **l’intersection** (déjà présents sur npm).

Liste actuelle des plugins npm (à mettre à jour si nécessaire) :

* @openclaw/bluebubbles
* @openclaw/diagnostics-otel
* @openclaw/discord
* @openclaw/lobster
* @openclaw/matrix
* @openclaw/msteams
* @openclaw/nextcloud-talk
* @openclaw/nostr
* @openclaw/voice-call
* @openclaw/zalo
* @openclaw/zalouser

Les notes de version doivent également mentionner les **nouveaux plugins intégrés optionnels** qui ne sont **pas
activés par défaut** (exemple : `tlon`).