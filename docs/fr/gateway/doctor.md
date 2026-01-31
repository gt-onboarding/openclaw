---
title: Doctor
summary: "Commande Doctor : vérifications d'intégrité, migrations de configuration et étapes de réparation"
read_when:
  - Ajout ou modification de migrations de Doctor
  - Introduction de modifications de configuration rompant la compatibilité
---

<div id="doctor">
  # Doctor
</div>

`openclaw doctor` est l’outil de réparation et de migration pour OpenClaw. Il corrige la configuration et l’état obsolètes, vérifie l’intégrité du système et fournit des instructions de réparation concrètes.

<div id="quick-start">
  ## Démarrage rapide
</div>

```bash
openclaw doctor
```

<div id="headless-automation">
  ### Mode sans interface / automatisation
</div>

```bash
openclaw doctor --yes
```

Accepter les valeurs par défaut sans demande de confirmation (y compris, le cas échéant, les étapes de redémarrage, de réparation de service ou de sandbox).

```bash
openclaw doctor --repair
```

Applique automatiquement les correctifs recommandés (correctifs et redémarrages lorsqu’ils sont sans risque).

```bash
openclaw doctor --repair --force
```

Appliquer également des corrections agressives (remplace les configurations personnalisées de Supervisor).

```bash
openclaw doctor --non-interactive
```

S’exécute sans interaction et applique uniquement des migrations sûres (normalisation de la configuration + déplacement de l’état sur disque). Ignore les actions de redémarrage/service/sandbox qui nécessitent une confirmation humaine.
Les anciennes migrations d’état sont exécutées automatiquement lorsqu’elles sont détectées.

```bash
openclaw doctor --deep
```

Analyser les services système à la recherche d’installations supplémentaires du Gateway (launchd/systemd/schtasks).

Si vous souhaitez examiner les modifications avant de les appliquer, ouvrez d’abord le fichier de configuration :

```bash
cat ~/.openclaw/openclaw.json
```

<div id="what-it-does-summary">
  ## Ce qu’il fait (résumé)
</div>

* Mise à jour préalable facultative pour les installations git (mode interactif uniquement).
* Vérification de fraîcheur du protocole UI (reconstruit le Control UI lorsque le schéma de protocole est plus récent).
* Vérification d’intégrité (« health check ») + invite au redémarrage.
* Résumé de l’état des compétences (éligibles/manquantes/bloquées).
* Normalisation de la configuration pour les valeurs héritées (legacy).
* Avertissements de surdéfinition (override) du fournisseur OpenCode Zen (`models.providers.opencode`).
* Migration de l’état historique sur disque (sessions/répertoire d’agent/auth WhatsApp).
* Vérifications d’intégrité et de permissions de l’état (sessions, transcriptions, répertoire d’état).
* Vérifications des permissions des fichiers de configuration (`chmod 600`) lors de l’exécution locale.
* Intégrité de l’authentification des modèles : vérifie l’expiration OAuth, peut rafraîchir les jetons proches de l’expiration et signale les états de cooldown/désactivation des profils d’authentification.
* Détection d’un répertoire d’espace de travail supplémentaire (`~/openclaw`).
* Réparation de l’image de sandbox lorsque le sandboxing est activé.
* Migration des services hérités et détection d’instances supplémentaires de Gateway.
* Vérifications du runtime du Gateway (service installé mais non démarré ; étiquette launchd en cache).
* Avertissements sur l’état des canaux (sondés depuis le Gateway en cours d’exécution).
* Audit de la configuration du superviseur (launchd/systemd/schtasks) avec réparation optionnelle.
* Vérifications des bonnes pratiques du runtime du Gateway (Node vs Bun, chemins de gestionnaires de versions).
* Diagnostics de collision de port du Gateway (par défaut `18789`).
* Avertissements de sécurité pour les politiques de DM `open`.
* Avertissements d’authentification du Gateway lorsqu’aucun `gateway.auth.token` n’est défini (mode local ; propose la génération de jeton).
* Vérification du mode « linger » systemd sur Linux.
* Vérifications pour les installations depuis les sources (incohérence d’espace de travail pnpm, ressources UI manquantes, binaire tsx manquant).
* Écrit la configuration mise à jour + les métadonnées de l’assistant de configuration (wizard).

<div id="detailed-behavior-and-rationale">
  ## Fonctionnement détaillé et justification
</div>

<div id="0-optional-update-git-installs">
  ### 0) Mise à jour facultative (installations Git)
</div>

S’il s’agit d’un checkout Git et que `doctor` s’exécute de manière interactive, il propose de
mettre à jour (fetch/rebase/build) avant d’exécuter `doctor`.

<div id="1-config-normalization">
  ### 1) Normalisation de la configuration
</div>

Si la configuration contient d’anciens formats de valeurs (par exemple `messages.ackReaction`
sans surcharge spécifique à un canal), doctor les normalise pour les faire correspondre au schéma actuel.

<div id="2-legacy-config-key-migrations">
  ### 2) Migrations des clés de configuration héritées
</div>

Lorsque la configuration contient des clés obsolètes, les autres commandes refusent de s&#39;exécuter et vous demandent
d&#39;exécuter `openclaw doctor`.

Doctor va :

* Expliquer quelles clés héritées ont été trouvées.
* Afficher la migration qu&#39;il a appliquée.
* Réécrire `~/.openclaw/openclaw.json` avec le schéma mis à jour.

Le Gateway exécute également automatiquement les migrations de doctor au démarrage lorsqu&#39;il détecte un
format de configuration hérité, de sorte que les configurations obsolètes sont réparées sans intervention manuelle.

Migrations actuelles :

* `routing.allowFrom` → `channels.whatsapp.allowFrom`
* `routing.groupChat.requireMention` → `channels.whatsapp/telegram/imessage.groups."*".requireMention`
* `routing.groupChat.historyLimit` → `messages.groupChat.historyLimit`
* `routing.groupChat.mentionPatterns` → `messages.groupChat.mentionPatterns`
* `routing.queue` → `messages.queue`
* `routing.bindings` → `bindings` de niveau supérieur
* `routing.agents`/`routing.defaultAgentId` → `agents.list` + `agents.list[].default`
* `routing.agentToAgent` → `tools.agentToAgent`
* `routing.transcribeAudio` → `tools.media.audio.models`
* `bindings[].match.accountID` → `bindings[].match.accountId`
* `identity` → `agents.list[].identity`
* `agent.*` → `agents.defaults` + `tools.*` (tools/elevated/exec/sandbox/subagents)
* `agent.model`/`allowedModels`/`modelAliases`/`modelFallbacks`/`imageModelFallbacks`
  → `agents.defaults.models` + `agents.defaults.model.primary/fallbacks` + `agents.defaults.imageModel.primary/fallbacks`

<div id="2b-opencode-zen-provider-overrides">
  ### 2b) Redéfinitions du fournisseur OpenCode Zen
</div>

Si vous avez ajouté `models.providers.opencode` (ou `opencode-zen`) manuellement, cela
remplace le catalogue OpenCode Zen intégré de `@mariozechner/pi-ai`. Cela peut
contraindre tous les modèles à utiliser une seule API ou ramener les coûts à zéro. Doctor vous avertit afin que vous puissiez
supprimer cette redéfinition et rétablir le routage API et les coûts par modèle.

<div id="3-legacy-state-migrations-disk-layout">
  ### 3) Migrations d&#39;état héritées (organisation sur disque)
</div>

Doctor peut migrer d&#39;anciens agencements sur disque vers la structure actuelle :

* Stockage des sessions + transcriptions :
  * de `~/.openclaw/sessions/` vers `~/.openclaw/agents/<agentId>/sessions/`
* Répertoire d&#39;agent :
  * de `~/.openclaw/agent/` vers `~/.openclaw/agents/<agentId>/agent/`
* État d&#39;authentification WhatsApp (Baileys) :
  * depuis l&#39;ancien répertoire `~/.openclaw/credentials/*.json` (sauf `oauth.json`)
  * vers `~/.openclaw/credentials/whatsapp/<accountId>/...` (ID de compte par défaut : `default`)

Ces migrations sont effectuées selon le principe du best-effort et sont idempotentes ; Doctor émet des avertissements
lorsqu&#39;il laisse d&#39;anciens dossiers en place comme sauvegardes. Le Gateway et la CLI migrent également
automatiquement les anciennes sessions et le répertoire d&#39;agent au démarrage, de sorte que l&#39;historique, l&#39;authentification et les modèles se retrouvent dans le chemin propre à chaque agent sans exécution manuelle de Doctor. L&#39;authentification WhatsApp n&#39;est volontairement migrée que via `openclaw doctor`.

<div id="4-state-integrity-checks-session-persistence-routing-and-safety">
  ### 4) Vérifications d&#39;intégrité de l&#39;état (persistance des sessions, routage et sécurité)
</div>

Le répertoire d&#39;état est le centre névralgique opérationnel. S&#39;il disparaît, vous perdez
les sessions, les identifiants, les journaux et la configuration (sauf si vous avez des sauvegardes ailleurs).

Contrôles effectués par Doctor :

* **State dir missing** : avertit d&#39;une perte d&#39;état catastrophique, propose de recréer
  le répertoire et vous rappelle qu&#39;il ne peut pas récupérer les données manquantes.
* **State dir permissions** : vérifie que le répertoire est inscriptible ; propose de réparer les
  permissions (et émet une astuce `chown` lorsqu&#39;une incohérence propriétaire/groupe est détectée).
* **Session dirs missing** : `sessions/` et le répertoire de stockage des sessions sont
  requis pour conserver l&#39;historique et éviter les plantages `ENOENT`.
* **Transcript mismatch** : avertit lorsque des entrées de session récentes ont des
  fichiers de transcription manquants.
* **Main session “1-line JSONL”** : signale lorsque la transcription principale ne contient
  qu&#39;une seule ligne (l&#39;historique ne s&#39;accumule pas).
* **Multiple state dirs** : avertit lorsque plusieurs dossiers `~/.openclaw` existent dans
  différents répertoires personnels ou lorsque `OPENCLAW_STATE_DIR` pointe ailleurs (l&#39;historique
  peut se fragmenter entre les installations).
* **Remote mode reminder** : si `gateway.mode=remote`, Doctor vous rappelle de l&#39;exécuter
  sur l&#39;hôte distant (c&#39;est là que réside l&#39;état).
* **Config file permissions** : avertit si `~/.openclaw/openclaw.json` est lisible par le
  groupe/tout le monde et propose de restreindre à `600`.

<div id="5-model-auth-health-oauth-expiry">
  ### 5) État de l’authentification des modèles (expiration OAuth)
</div>

Doctor inspecte les profils OAuth dans le stockage d’authentification, avertit lorsque les jetons
sont sur le point d’expirer ou sont expirés, et peut les actualiser lorsque c’est possible en toute sécurité. Si le profil
Anthropic Claude Code est obsolète, il suggère d’exécuter `claude setup-token` (ou de coller un setup-token).
Les invites d’actualisation n’apparaissent que lors d’une exécution interactive (TTY) ; `--non-interactive`
ignore les tentatives d’actualisation.

Doctor signale également les profils d’authentification temporairement inutilisables en raison :

* de courts délais d’attente (cooldowns) liés aux limitations de débit/dépassements de délai/échecs d’authentification
* de désactivations plus longues (échecs de facturation/crédit)

<div id="6-hooks-model-validation">
  ### 6) Validation du modèle des hooks
</div>

Si `hooks.gmail.model` est défini, doctor valide la référence de modèle par rapport
au catalogue et à la liste d’autorisation et émet un avertissement lorsqu’elle ne peut pas être résolue ou qu’elle est interdite.

<div id="7-sandbox-image-repair">
  ### 7) Réparation d’image de sandbox
</div>

Lorsque le sandboxing est activé, `doctor` vérifie les images Docker et propose de créer l’image ou de revenir à d’anciens noms (legacy) si l’image actuelle est absente.

<div id="8-gateway-service-migrations-and-cleanup-hints">
  ### 8) Migrations de services Gateway et conseils de nettoyage
</div>

Doctor détecte les anciens services Gateway (launchd/systemd/schtasks) et
propose de les supprimer puis d’installer le service OpenClaw en utilisant le port
Gateway actuel. Il peut également rechercher des services supplémentaires
de type Gateway et afficher des conseils de nettoyage.
Les services Gateway OpenClaw nommés par profil sont considérés comme des
services de première classe et ne sont pas signalés comme « supplémentaires ».

<div id="9-security-warnings">
  ### 9) Avertissements de sécurité
</div>

Doctor émet des avertissements lorsqu’un fournisseur est configuré en open pour les DMs (acceptation de messages sans restriction depuis n’importe quel utilisateur) sans liste d’autorisation, ou lorsqu’une politique est configurée de manière dangereuse.

<div id="10-systemd-linger-linux">
  ### 10) systemd linger (Linux)
</div>

Si vous exécutez le service en tant que service utilisateur systemd, doctor vérifie que le mode de persistance (linger) est activé afin que Gateway reste actif après la déconnexion.

<div id="11-skills-status">
  ### 11) Statut des compétences
</div>

Doctor affiche un bref récapitulatif des compétences éligibles/manquantes/bloquées pour l’espace de travail actuel.

<div id="12-gateway-auth-checks-local-token">
  ### 12) Vérifications d&#39;authentification du Gateway (jeton local)
</div>

Doctor affiche un avertissement lorsque `gateway.auth` est absent sur un Gateway local et propose de générer un jeton. Utilisez `openclaw doctor --generate-gateway-token` pour forcer la création du jeton dans les flux d&#39;automatisation.

<div id="13-gateway-health-check-restart">
  ### 13) Contrôle d’intégrité du Gateway + redémarrage
</div>

Doctor exécute un contrôle d’intégrité et propose de redémarrer le Gateway lorsqu’il semble ne pas fonctionner correctement.

<div id="14-channel-status-warnings">
  ### 14) Avertissements sur l&#39;état des canaux
</div>

Si le Gateway est en bon état de fonctionnement, `doctor` exécute une vérification de l&#39;état des canaux et signale des avertissements avec des correctifs suggérés.

<div id="15-supervisor-config-audit-repair">
  ### 15) Audit et réparation de la configuration du superviseur
</div>

`openclaw doctor` vérifie la configuration du superviseur installée (launchd/systemd/schtasks) pour
détecter les valeurs par défaut manquantes ou obsolètes (par exemple, les dépendances systemd `network-online` et
le délai de redémarrage). Lorsqu’il détecte une incohérence, il recommande une mise à jour et peut
réécrire le fichier de service ou la tâche avec les valeurs par défaut actuelles.

Remarques :

* `openclaw doctor` demande une confirmation avant de réécrire la configuration du superviseur.
* `openclaw doctor --yes` accepte automatiquement les propositions de réparation par défaut.
* `openclaw doctor --repair` applique les correctifs recommandés sans confirmation.
* `openclaw doctor --repair --force` écrase les configurations de superviseur personnalisées.
* Vous pouvez toujours forcer une réécriture complète via `openclaw gateway install --force`.

<div id="16-gateway-runtime-port-diagnostics">
  ### 16) Diagnostic du runtime du Gateway et des ports
</div>

Doctor analyse le runtime du service (PID, dernier statut de sortie) et émet un avertissement lorsque
le service est installé mais n’est pas effectivement en cours d’exécution. Il vérifie également les collisions de ports
sur le port du Gateway (par défaut `18789`) et signale les causes probables (Gateway déjà
en cours d’exécution, tunnel SSH).

<div id="17-gateway-runtime-best-practices">
  ### 17) Bonnes pratiques d’exécution du Gateway
</div>

Doctor émet un avertissement lorsque le service Gateway fonctionne avec Bun ou via un chemin Node géré par un gestionnaire de versions
(`nvm`, `fnm`, `volta`, `asdf`, etc.). Les canaux WhatsApp + Telegram nécessitent Node,
et les chemins gérés par un gestionnaire de versions peuvent cesser de fonctionner après des mises à jour, car le service ne charge pas vos scripts d’initialisation de shell. Doctor propose de migrer vers une installation système de Node lorsqu’elle est disponible (Homebrew/apt/choco).

<div id="18-config-write-wizard-metadata">
  ### 18) Écriture de la configuration + métadonnées de l’assistant
</div>

Doctor enregistre toute modification de configuration et appose des métadonnées de l’assistant pour consigner l’exécution de Doctor.

<div id="19-workspace-tips-backup-memory-system">
  ### 19) Conseils relatifs à l’espace de travail (sauvegarde + système de mémoire)
</div>

Doctor propose un système de mémoire de l’espace de travail lorsqu’il est absent et affiche un conseil de sauvegarde
si l’espace de travail n’est pas déjà versionné avec Git.

Voir [/concepts/agent-workspace](/fr/concepts/agent-workspace) pour un guide complet de la
structure de l’espace de travail et de la sauvegarde via Git (GitHub ou GitLab privés recommandés).