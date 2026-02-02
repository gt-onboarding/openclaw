---
title: Clawhub
summary: "Guide ClawHub : registre public de compétences + workflows CLI"
read_when:
  - Présentation de ClawHub à de nouveaux utilisateurs
  - Installation, recherche ou publication de compétences
  - Explication des options CLI de ClawHub et du comportement de synchronisation
---

<div id="clawhub">
  # ClawHub
</div>

ClawHub est le **registre public de compétences pour OpenClaw**. C’est un service gratuit : toutes les compétences sont publiques, ouvertes et visibles par tout le monde pour le partage et la réutilisation. Une compétence n’est qu’un dossier contenant un fichier `SKILL.md` (ainsi que des fichiers texte annexes). Vous pouvez parcourir les compétences dans l’application web ou utiliser la CLI pour rechercher, installer, mettre à jour et publier des compétences.

Site : [clawhub.com](https://clawhub.com)

<div id="who-this-is-for-beginner-friendly">
  ## À qui s’adresse ce guide (accessible aux débutants)
</div>

Si vous voulez ajouter de nouvelles capacités à votre agent OpenClaw, ClawHub est le moyen le plus simple de trouver et installer des compétences. Vous n’avez pas besoin de connaître le fonctionnement interne. Vous pouvez :

* Rechercher des compétences en langage naturel.
* Installer une compétence dans votre espace de travail.
* Mettre à jour des compétences plus tard avec une seule commande.
* Sauvegarder vos propres compétences en les publiant.

<div id="quick-start-non-technical">
  ## Démarrage rapide (non technique)
</div>

1. Installez la CLI (voir la section suivante).
2. Recherchez ce dont vous avez besoin :
   * `clawhub search "calendar"`
3. Installez une skill :
   * `clawhub install <skill-slug>`
4. Démarrez une nouvelle session OpenClaw afin que la nouvelle skill soit prise en compte.

<div id="install-the-cli">
  ## Installez la CLI
</div>

Choisissez une option :

```bash
npm i -g clawhub
```

```bash
pnpm add -g clawhub
```

<div id="how-it-fits-into-openclaw">
  ## Intégration dans OpenClaw
</div>

Par défaut, la CLI installe les compétences dans `./skills` sous votre répertoire de travail actuel. Si un espace de travail OpenClaw est configuré, `clawhub` se rabat sur cet espace de travail, sauf si vous redéfinissez `--workdir` (ou `CLAWHUB_WORKDIR`). OpenClaw charge les compétences de l’espace de travail depuis `<workspace>/skills` et les prendra en compte lors de la **prochaine** session. Si vous utilisez déjà `~/.openclaw/skills` ou des compétences intégrées, les compétences de l’espace de travail sont prioritaires.

Pour plus de détails sur la façon dont les compétences sont chargées, partagées et soumises à des restrictions d’accès, voir
[Compétences](/fr/tools/skills).

<div id="what-the-service-provides-features">
  ## Ce que le service propose (fonctionnalités)
</div>

* **Consultation publique** des compétences et de leur contenu `SKILL.md`.
* **Recherche** propulsée par des embeddings (recherche vectorielle), et pas seulement par des mots-clés.
* **Gestion de versions** avec semver, journaux de modifications et tags (y compris `latest`).
* **Téléchargements** de chaque version sous forme d’archive ZIP.
* **Étoiles et commentaires** pour les retours de la communauté.
* **Hooks de modération** pour les validations et les audits.
* **API adaptée à la CLI** pour l’automatisation et les scripts.

<div id="cli-commands-and-parameters">
  ## Commandes et paramètres CLI
</div>

Options globales (s&#39;appliquent à toutes les commandes) :

* `--workdir <dir>`: Répertoire de travail (par défaut : répertoire courant ; se replie sur l&#39;espace de travail OpenClaw).
* `--dir <dir>`: Répertoire des compétences, relatif à workdir (par défaut : `skills`).
* `--site <url>`: URL de base du site (connexion via navigateur).
* `--registry <url>`: URL de base de l&#39;API du registre.
* `--no-input`: Désactiver les invites (mode non interactif).
* `-V, --cli-version`: Afficher la version de la CLI.

Authentification :

* `clawhub login` (flux via navigateur) ou `clawhub login --token <token>`
* `clawhub logout`
* `clawhub whoami`

Options :

* `--token <token>`: Coller un jeton d&#39;API.
* `--label <label>`: Libellé stocké pour les jetons de connexion via navigateur (par défaut : `CLI token`).
* `--no-browser`: Ne pas ouvrir de navigateur (nécessite `--token`).

Recherche :

* `clawhub search "query"`
* `--limit <n>`: Nombre maximal de résultats.

Installation :

* `clawhub install <slug>`
* `--version <version>`: Installer une version spécifique.
* `--force`: Écraser si le dossier existe déjà.

Mise à jour :

* `clawhub update <slug>`
* `clawhub update --all`
* `--version <version>`: Mettre à jour vers une version spécifique (un seul slug).
* `--force`: Écraser lorsque les fichiers locaux ne correspondent à aucune version publiée.

Liste :

* `clawhub list` (lit `.clawhub/lock.json`)

Publication :

* `clawhub publish <path>`
* `--slug <slug>`: Slug de la compétence.
* `--name <name>`: Nom d&#39;affichage.
* `--version <version>`: Version SemVer.
* `--changelog <text>`: Texte du changelog (peut être vide).
* `--tags <tags>`: Tags séparés par des virgules (par défaut : `latest`).

Suppression/rétablissement (propriétaire/admin uniquement) :

* `clawhub delete <slug> --yes`
* `clawhub undelete <slug> --yes`

Sync (analyse les compétences locales + publie les nouvelles/mises à jour) :

* `clawhub sync`
* `--root <dir...>`: Racines d&#39;analyse supplémentaires.
* `--all`: Tout envoyer sans invites.
* `--dry-run`: Afficher ce qui serait envoyé.
* `--bump <type>`: `patch|minor|major` pour les mises à jour (par défaut : `patch`).
* `--changelog <text>`: Changelog pour les mises à jour non interactives.
* `--tags <tags>`: Tags séparés par des virgules (par défaut : `latest`).
* `--concurrency <n>`: Vérifications du registre (par défaut : 4).

<div id="common-workflows-for-agents">
  ## Flux de travail courants pour les agents
</div>

<div id="search-for-skills">
  ### Rechercher des compétences
</div>

```bash
clawhub search "postgres backups"
```

<div id="download-new-skills">
  ### Télécharger de nouvelles compétences
</div>

```bash
clawhub install my-skill-pack
```

<div id="update-installed-skills">
  ### Mettre à jour vos compétences installées
</div>

```bash
clawhub update --all
```

<div id="back-up-your-skills-publish-or-sync">
  ### Sauvegardez vos compétences (publier ou synchroniser)
</div>

Pour un seul dossier de compétence :

```bash
clawhub publish ./my-skill --slug my-skill --name "My Skill" --version 1.0.0 --tags latest
```

Pour analyser et sauvegarder plusieurs compétences à la fois :

```bash
clawhub sync --all
```

<div id="advanced-details-technical">
  ## Détails techniques avancés
</div>

<div id="versioning-and-tags">
  ### Versionnage et tags
</div>

* Chaque publication crée une nouvelle `SkillVersion` **semver**.
* Les tags (comme `latest`) pointent vers une version ; déplacer un tag permet de revenir à une version précédente.
* Un journal des modifications est associé à chaque version et peut être vide lors de la synchronisation ou de la publication de mises à jour.

<div id="local-changes-vs-registry-versions">
  ### Modifications locales et versions du registre
</div>

Les mises à jour comparent le contenu local des skills aux versions du registre à l&#39;aide d&#39;un hachage de contenu. Si les fichiers locaux ne correspondent à aucune version publiée, la CLI demande confirmation avant d&#39;écraser les fichiers (ou nécessite `--force` dans les exécutions non interactives).

<div id="sync-scanning-and-fallback-roots">
  ### Analyse de synchronisation et racines de repli
</div>

`clawhub sync` analyse d&#39;abord votre répertoire de travail courant. Si aucune compétence n&#39;est trouvée, il se replie sur des emplacements historiques connus (par exemple `~/openclaw/skills` et `~/.openclaw/skills`). Ce mécanisme est conçu pour retrouver d&#39;anciennes installations de compétences sans options supplémentaires en ligne de commande.

<div id="storage-and-lockfile">
  ### Stockage et fichier de verrouillage
</div>

* Les compétences installées sont enregistrées dans `.clawhub/lock.json` de votre répertoire de travail.
* Les jetons d’authentification sont stockés dans le fichier de configuration de la CLI ClawHub (modifiable via `CLAWHUB_CONFIG_PATH`).

<div id="telemetry-install-counts">
  ### Télémétrie (nombre d’installations)
</div>

Lorsque vous exécutez `clawhub sync` alors que vous êtes connecté, la CLI envoie un instantané minimal pour comptabiliser les installations. Vous pouvez entièrement désactiver ce comportement :

```bash
export CLAWHUB_DISABLE_TELEMETRY=1
```

<div id="environment-variables">
  ## Variables d&#39;environnement
</div>

* `CLAWHUB_SITE`: Redéfinit l&#39;URL du site.
* `CLAWHUB_REGISTRY`: Redéfinit l&#39;URL de l&#39;api du registre.
* `CLAWHUB_CONFIG_PATH`: Redéfinit l&#39;emplacement où la CLI stocke le jeton et la configuration.
* `CLAWHUB_WORKDIR`: Redéfinit le répertoire de travail par défaut.
* `CLAWHUB_DISABLE_TELEMETRY=1`: Désactive la télémétrie pour `sync`.