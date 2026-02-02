---
title: Migration
summary: "Déplacer (migrer) une installation OpenClaw d'une machine à une autre"
read_when:
  - Vous déplacez OpenClaw vers un nouvel ordinateur portable ou serveur
  - Vous voulez conserver les sessions, l'authentification et les connexions de canaux (WhatsApp, etc.)
---

<div id="migrating-openclaw-to-a-new-machine">
  # Migration d’OpenClaw vers une nouvelle machine
</div>

Ce guide explique comment migrer le Gateway OpenClaw d’une machine à une autre **sans refaire la procédure d’onboarding**.

Sur le principe, la migration est simple :

* Copiez le **répertoire d’état** (`$OPENCLAW_STATE_DIR`, valeur par défaut : `~/.openclaw/`) — il inclut la configuration, l’authentification, les sessions et l’état des canaux.
* Copiez votre **espace de travail** (`~/.openclaw/workspace/` par défaut) — il inclut vos fichiers d’agent (mémoire, prompts, etc.).

Mais il existe des pièges classiques liés aux **profils**, aux **droits d’accès** et aux **copies partielles**.

<div id="before-you-start-what-you-are-migrating">
  ## Avant de commencer (ce que vous allez migrer)
</div>

<div id="1-identify-your-state-directory">
  ### 1) Identifiez votre répertoire d’état
</div>

La plupart des installations utilisent la valeur par défaut :

* **Répertoire d’état :** `~/.openclaw/`

Mais il peut être différent si vous utilisez :

* `--profile <name>` (devient souvent `~/.openclaw-<profile>/`)
* `OPENCLAW_STATE_DIR=/some/path`

Si vous n’êtes pas sûr, exécutez la commande suivante sur l’ancienne machine :

```bash
openclaw status
```

Recherchez toute occurrence de `OPENCLAW_STATE_DIR` / profile dans la sortie. Si vous exécutez plusieurs instances du Gateway, répétez l’opération pour chaque profil.

<div id="2-identify-your-workspace">
  ### 2) Identifiez votre espace de travail
</div>

Emplacements par défaut les plus courants :

* `~/.openclaw/workspace/` (espace de travail recommandé)
* un dossier personnalisé que vous avez créé

Votre espace de travail est l&#39;endroit où se trouvent des fichiers comme `MEMORY.md`, `USER.md` et `memory/*.md`.

<div id="3-understand-what-you-will-preserve">
  ### 3) Comprendre ce que vous allez conserver
</div>

Si vous copiez **à la fois** le répertoire d’état et l’espace de travail, vous conservez :

* La configuration du Gateway (`openclaw.json`)
* Les profils d’authentification / clés API / jetons OAuth
* L’historique des sessions + l’état de l’agent
* L’état des canaux (par exemple connexion/session WhatsApp)
* Vos fichiers d’espace de travail (mémoire, notes de compétences, etc.)

Si vous copiez **uniquement** l’espace de travail (par exemple via Git), vous ne conservez **pas** :

* les sessions
* les identifiants
* les connexions aux canaux

Ces éléments se trouvent sous `$OPENCLAW_STATE_DIR`.

<div id="migration-steps-recommended">
  ## Étapes de migration (recommandées)
</div>

<div id="step-0-make-a-backup-old-machine">
  ### Étape 0 — Faire une sauvegarde (ancienne machine)
</div>

Sur l’**ancienne** machine, arrêtez d’abord Gateway pour que les fichiers ne soient pas modifiés pendant la copie :

```bash
openclaw gateway stop
```

(Facultatif mais recommandé) archivez le répertoire d’état et l’espace de travail :

```bash
# Ajustez les chemins si vous utilisez un profil ou des emplacements personnalisés
cd ~
tar -czf openclaw-state.tgz .openclaw

tar -czf openclaw-workspace.tgz .openclaw/workspace
```

Si vous avez plusieurs profils/répertoires d’état (par exemple `~/.openclaw-main`, `~/.openclaw-work`), archivez chacun.

<div id="step-1-install-openclaw-on-the-new-machine">
  ### Étape 1 — Installer OpenClaw sur la nouvelle machine
</div>

Sur la **nouvelle** machine, installe la CLI (et Node.js si nécessaire) :

* Voir : [Installation](/fr/install)

À ce stade, ce n’est pas grave si la procédure d’onboarding crée un nouveau `~/.openclaw/` — tu le remplaceras à l’étape suivante.

<div id="step-2-copy-the-state-dir-workspace-to-the-new-machine">
  ### Étape 2 — Copier le répertoire d’état + l’espace de travail vers la nouvelle machine
</div>

Copiez **les deux** :

* `$OPENCLAW_STATE_DIR` (par défaut `~/.openclaw/`)
* votre espace de travail (par défaut `~/.openclaw/workspace/`)

Approches courantes :

* utiliser `scp` pour transférer les archives tar puis les extraire
* utiliser `rsync -a` via SSH
* utiliser un disque externe

Après la copie, vérifiez :

* que les répertoires cachés ont été inclus (par exemple `.openclaw/`)
* que la propriété des fichiers est correcte pour l’utilisateur qui exécute le service Gateway

<div id="step-3-run-doctor-migrations-service-repair">
  ### Étape 3 — Exécuter Doctor (migrations + réparation du service)
</div>

Sur la **nouvelle** machine :

```bash
openclaw doctor
```

`doctor` est la commande « sûre et sans surprise ». Elle répare les services, applique les migrations de configuration et avertit en cas d’incohérences.

Ensuite :

```bash
openclaw gateway restart
openclaw status
```

<div id="common-footguns-and-how-to-avoid-them">
  ## Pièges classiques (et comment les éviter)
</div>

<div id="footgun-profile-state-dir-mismatch">
  ### Piège : incompatibilité entre profil et state-dir
</div>

Si vous faisiez tourner l’ancien Gateway avec un profil (ou `OPENCLAW_STATE_DIR`), et que le nouveau Gateway en utilise un différent, vous verrez des symptômes comme :

* des modifications de configuration qui ne prennent pas effet
* des canaux manquants ou déconnectés
* un historique de session vide

Correctif : exécutez le Gateway/service en utilisant le **même** profil/state-dir que celui que vous avez migré, puis relancez :

```bash
openclaw doctor
```

<div id="footgun-copying-only-openclawjson">
  ### Piège classique : copier uniquement `openclaw.json`
</div>

`openclaw.json` ne suffit pas. De nombreux fournisseurs stockent l’état sous :

* `$OPENCLAW_STATE_DIR/credentials/`
* `$OPENCLAW_STATE_DIR/agents/<agentId>/...`

Migrez toujours l’intégralité du dossier `$OPENCLAW_STATE_DIR`.

<div id="footgun-permissions-ownership">
  ### Footgun: autorisations / propriété
</div>

Si vous avez copié des fichiers en tant que root ou changé d’utilisateur, le Gateway risque de ne pas pouvoir lire les identifiants/sessions.

Solution : assurez-vous que le répertoire d’état et l’espace de travail appartiennent à l’utilisateur qui exécute le Gateway.

<div id="footgun-migrating-between-remotelocal-modes">
  ### Piège à éviter : migration entre modes distant/local
</div>

* Si votre UI (WebUI/TUI) pointe vers un Gateway **distant**, c’est l’hôte distant qui détient le stockage des sessions et l’espace de travail.
* La migration de votre ordinateur portable ne déplacera pas l’état du Gateway distant.

Si vous êtes en mode distant, migrez l’**hôte du Gateway**.

<div id="footgun-secrets-in-backups">
  ### Gros piège : secrets dans les sauvegardes
</div>

`$OPENCLAW_STATE_DIR` contient des secrets (clés API, jetons OAuth, identifiants WhatsApp). Traitez les sauvegardes comme des secrets de production :

* conservez-les chiffrées
* évitez de les partager par des canaux non sécurisés
* renouvelez les clés si vous soupçonnez une exposition

<div id="verification-checklist">
  ## Liste de vérification
</div>

Sur la nouvelle machine, vérifiez :

* `openclaw status` indique que le Gateway OpenClaw est en cours d’exécution
* Vos canaux sont toujours connectés (par exemple, WhatsApp ne demande pas un nouvel appairage)
* Le tableau de bord s’ouvre et affiche les sessions existantes
* Les fichiers de votre espace de travail (memory, configs) sont présents

<div id="related">
  ## Ressources associées
</div>

* [Doctor](/fr/gateway/doctor)
* [Dépannage du Gateway](/fr/gateway/troubleshooting)
* [Où OpenClaw stocke-t-il ses données ?](/fr/help/faq#where-does-openclaw-store-its-data)