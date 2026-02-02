---
title: Mise à jour
summary: "Mettre à jour OpenClaw en toute sécurité (installation globale ou depuis les sources) et stratégie de retour arrière"
read_when:
  - Mise à jour d’OpenClaw
  - Un dysfonctionnement survient après une mise à jour
---

<div id="updating">
  # Mise à jour
</div>

OpenClaw évolue rapidement (en phase pré‑« 1.0 »). Traitez les mises à jour comme un déploiement d’infrastructure : mettre à jour → exécuter des vérifications → redémarrer (ou utiliser `openclaw update`, qui redémarre) → vérifier.

<div id="recommended-re-run-the-website-installer-upgrade-in-place">
  ## Recommandé : relancer le programme d’installation depuis le site web (mise à niveau de l’installation existante)
</div>

La méthode de mise à jour **préférée** consiste à relancer le programme d’installation depuis le site web. Celui‑ci détecte les installations existantes, met à niveau l’installation déjà en place et exécute `openclaw doctor` si nécessaire.

```bash
curl -fsSL https://openclaw.bot/install.sh | bash
```

Notes :

* Ajoutez `--no-onboard` si vous ne voulez pas que l’assistant de configuration initiale se relance.
* Pour les **installations depuis les sources**, utilisez :
  ```bash
  curl -fsSL https://openclaw.bot/install.sh | bash -s -- --install-method git --no-onboard
  ```
  Le programme d’installation exécutera `git pull --rebase` **uniquement** si le dépôt est propre.
* Pour les **installations globales**, le script utilise `npm install -g openclaw@latest` en interne.
* Note de compatibilité : `openclaw` reste disponible en tant que shim de compatibilité.

<div id="before-you-update">
  ## Avant la mise à jour
</div>

* Sachez comment vous avez installé : **globalement** (npm/pnpm) vs **depuis les sources** (`git clone`).
* Sachez comment votre Gateway s’exécute : **au premier plan dans un terminal** vs **en tant que service supervisé** (launchd/systemd).
* Faites un instantané de vos personnalisations :
  * Config : `~/.openclaw/openclaw.json`
  * Identifiants : `~/.openclaw/credentials/`
  * Espace de travail : `~/.openclaw/workspace`

<div id="update-global-install">
  ## Mise à jour (installation globale)
</div>

Installation globale (choisissez une méthode) :

```bash
npm i -g openclaw@latest
```

```bash
pnpm add -g openclaw@latest
```

Nous ne **recommandons pas** Bun comme runtime pour le Gateway (bogues WhatsApp/Telegram).

Pour changer de canal de mise à jour (installations git + npm) :

```bash
openclaw update --channel beta
openclaw update --channel dev
openclaw update --channel stable
```

Utilisez `--tag <dist-tag|version>` pour installer ponctuellement un tag ou une version spécifique.

Voir [Canaux de développement](/fr/install/development-channels) pour la signification des canaux et les notes de version.

Remarque : pour les installations via npm, Gateway enregistre dans les logs une indication de mise à jour au démarrage (il vérifie le tag du canal actuel). Désactivez ce comportement via `update.checkOnStart: false`.

Ensuite :

```bash
openclaw doctor
openclaw gateway restart
openclaw health
```

Notes :

* Si votre Gateway s’exécute en tant que service, `openclaw gateway restart` est préférable au fait de tuer des processus (PID).
* Si vous devez rester sur une version spécifique, consultez « Rollback / pinning » ci-dessous.

<div id="update-openclaw-update">
  ## Mise à jour (`openclaw update`)
</div>

Pour les **installations à partir des sources** (checkout Git), privilégiez :

```bash
openclaw update
```

Cette commande exécute un processus de mise à jour relativement sûr :

* Nécessite un arbre de travail git propre.
* Bascule sur le canal sélectionné (tag ou branche).
* Effectue un fetch puis un rebase par rapport à l’upstream configuré (canal de développement).
* Installe les dépendances, effectue la build, génère le Control UI, puis exécute `openclaw doctor`.
* Redémarre le service Gateway par défaut (utilisez `--no-restart` pour éviter le redémarrage).

Si vous avez installé via **npm/pnpm** (sans métadonnées git), `openclaw update` tentera de se mettre à jour via votre gestionnaire de paquets. S’il ne peut pas détecter l’installation, utilisez plutôt « Update (global install) ».

<div id="update-control-ui-rpc">
  ## Mise à jour (Control UI / RPC)
</div>

La Control UI propose **Update &amp; Restart** (RPC : `update.run`). Cette action :

1. Exécute le même flux de mise à jour du code source que `openclaw update` (git checkout uniquement).
2. Écrit un fichier sentinelle de redémarrage avec un rapport structuré (dernières lignes de stdout/stderr).
3. Redémarre le Gateway et notifie la dernière session active avec le rapport.

Si le rebase échoue, le Gateway s’arrête et redémarre sans appliquer la mise à jour.

<div id="update-from-source">
  ## Mise à jour (depuis les sources)
</div>

À partir du dépôt cloné :

Méthode recommandée :

```bash
openclaw update
```

Manuel (plus ou moins équivalent) :

```bash
git pull
pnpm install
pnpm build
pnpm ui:build # installe automatiquement les dépendances de l'UI à la première exécution
openclaw doctor
openclaw health
```

Notes :

* `pnpm build` est important lorsque vous exécutez le binaire packagé `openclaw` ([`openclaw.mjs`](https://github.com/openclaw/openclaw/blob/main/openclaw.mjs)) ou que vous utilisez Node pour exécuter `dist/`.
* Si vous l’exécutez depuis un checkout du dépôt sans installation globale, utilisez `pnpm openclaw ...` pour les commandes CLI.
* Si vous l’exécutez directement depuis le code TypeScript (`pnpm openclaw ...`), une recompilation est généralement inutile, mais **les migrations de configuration s’appliquent toujours** → exécutez `openclaw doctor`.
* Passer d’une installation globale à une installation via git (et inversement) est simple : installez l’autre variante, puis exécutez `openclaw doctor` afin que le point d’entrée du service Gateway soit réécrit vers l’installation actuelle.

<div id="always-run-openclaw-doctor">
  ## Toujours exécuter : `openclaw doctor`
</div>

Doctor est la commande de « mise à jour sans risque ». Elle est délibérément ennuyeuse : réparer + migrer + avertir.

Remarque : si vous êtes sur une **installation depuis les sources** (git checkout), `openclaw doctor` proposera d’exécuter d’abord `openclaw update`.

Exemples de tâches effectuées :

* Migrer les clés de configuration obsolètes et les anciens emplacements de fichiers de configuration.
* Auditer les stratégies de DM et avertir en cas de paramètres « open » risqués (le jeton de stratégie open autorise l’acceptation de messages par n’importe quel utilisateur sans restriction).
* Vérifier l’état de santé du Gateway et proposer un redémarrage.
* Détecter et migrer les anciens services Gateway (launchd/systemd ; schtasks hérités) vers les services OpenClaw actuels.
* Sous Linux, vérifier que le mode « systemd user lingering » est activé (afin que le Gateway survive à la déconnexion).

Détails : [Doctor](/fr/gateway/doctor)

<div id="start-stop-restart-the-gateway">
  ## Démarrer / arrêter / redémarrer le Gateway
</div>

CLI (fonctionne quel que soit le système d’exploitation) :

```bash
openclaw gateway status
openclaw gateway stop
openclaw gateway restart
openclaw gateway --port 18789
openclaw logs --follow
```

Si le service est supervisé :

* macOS launchd (LaunchAgent fourni avec l’app) : `launchctl kickstart -k gui/$UID/bot.molt.gateway` (utilisez `bot.molt.<profile>` ; l’ancien préfixe `com.openclaw.*` fonctionne toujours)
* Service utilisateur systemd sous Linux : `systemctl --user restart openclaw-gateway[-<profile>].service`
* Windows (WSL2) : `systemctl --user restart openclaw-gateway[-<profile>].service`
  * `launchctl`/`systemctl` ne fonctionnent que si le service est installé ; sinon exécutez `openclaw gateway install`.

Runbook + noms exacts de services : [Gateway runbook](/fr/gateway)

<div id="rollback-pinning-when-something-breaks">
  ## Retour arrière / verrouillage de version (en cas de problème)
</div>

<div id="pin-global-install">
  ### Épingler (installation globale)
</div>

Installe une version dont tu sais qu&#39;elle fonctionne (remplace `<version>` par la dernière version fonctionnelle) :

```bash
npm i -g openclaw@<version>
```

```bash
pnpm add -g openclaw@<version>
```

Astuce : pour afficher la version actuellement publiée, exécutez `npm view openclaw version`.

Ensuite, redémarrez puis relancez doctor :

```bash
openclaw doctor
openclaw gateway restart
```

<div id="pin-source-by-date">
  ### Figer la source par date
</div>

Choisissez un commit à une date donnée (par exemple : « état de la branche main au 2026-01-01 ») :

```bash
git fetch origin
git checkout "$(git rev-list -n 1 --before=\"2026-01-01\" origin/main)"
```

Puis réinstallez les dépendances et redémarrez :

```bash
pnpm install
pnpm build
openclaw gateway restart
```

Si vous souhaitez revenir plus tard à la version la plus récente :

```bash
git checkout main
git pull
```

<div id="if-youre-stuck">
  ## Si vous êtes bloqué
</div>

* Relancez `openclaw doctor` et examinez attentivement sa sortie (elle indique souvent comment corriger le problème).
* Consultez : [Dépannage](/fr/gateway/troubleshooting)
* Posez votre question sur Discord : https://channels.discord.gg/clawd