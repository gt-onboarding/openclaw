---
title: Configuration
summary: "Guide de configuration : gardez votre configuration OpenClaw personnalisée tout en restant à jour"
read_when:
  - Configuration d’une nouvelle machine
  - Vous voulez ce qu’il y a de plus récent sans perturber votre configuration personnelle
---

<div id="setup">
  # Mise en route
</div>

Dernière mise à jour : 2026-01-01

<div id="tldr">
  ## TL;DR
</div>

* **La personnalisation se fait en dehors du dépôt :** `~/.openclaw/workspace` (espace de travail) + `~/.openclaw/openclaw.json` (config).
* **Workflow stable :** installez l’app macOS ; laissez-la exécuter le Gateway fourni.
* **Workflow cutting edge :** exécutez vous‑même le Gateway via `pnpm gateway:watch`, puis laissez l’app macOS se connecter en mode Local.

<div id="prereqs-from-source">
  ## Prérequis (d’après la source)
</div>

* Node `>=22`
* `pnpm`
* Docker (facultatif ; uniquement pour l’installation en conteneur/e2e — voir [Docker](/fr/install/docker))

<div id="tailoring-strategy-so-updates-dont-hurt">
  ## Stratégie de personnalisation (pour des mises à jour sans douleur)
</div>

Si vous voulez du « 100 % sur mesure » *et* des mises à jour simples, conservez vos personnalisations dans :

* **Config :** `~/.openclaw/openclaw.json` (style JSON/JSON5)
* **Espace de travail :** `~/.openclaw/workspace` (compétences, prompts, données de mémoire ; transformez-le en dépôt Git privé)

Effectuez le bootstrap une seule fois :

```bash
openclaw setup
```

Depuis ce dépôt, utilisez le point d&#39;entrée CLI local :

```bash
openclaw setup
```

Si vous n’avez pas encore d’installation globale, exécutez la commande `pnpm openclaw setup`.

<div id="stable-workflow-macos-app-first">
  ## Flux de travail stable (app macOS en premier)
</div>

1. Installez et lancez **OpenClaw.app** (barre de menus).
2. Terminez la check-list d&#39;onboarding/autorisations (boîtes de dialogue TCC).
3. Assurez-vous que Gateway est **Local** et en cours d&#39;exécution (l&#39;app le gère).
4. Connectez les interfaces (exemple : WhatsApp) :

```bash
openclaw channels login
```

5. Vérification rapide :

```bash
openclaw health
```

Si la procédure d’onboarding n’est pas disponible dans votre version :

* Exécutez `openclaw setup`, puis `openclaw channels login`, puis lancez manuellement le Gateway (`openclaw gateway`).

<div id="bleeding-edge-workflow-gateway-in-a-terminal">
  ## Flux de travail de pointe (Gateway dans un terminal)
</div>

Objectif : travailler sur le Gateway en TypeScript, obtenir le rechargement à chaud, garder l&#39;UI de l&#39;app macOS connectée.

<div id="0-optional-run-the-macos-app-from-source-too">
  ### 0) (Optionnel) Lancer aussi l’app macOS depuis les sources
</div>

Si vous voulez également la version de développement la plus récente de l’app macOS :

```bash
./scripts/restart-mac.sh
```

<div id="1-start-the-dev-gateway">
  ### 1) Démarrer le Gateway de développement
</div>

```bash
pnpm install
pnpm gateway:watch
```

`gateway:watch` démarre le Gateway en mode watch et le recharge à chaque modification des fichiers TypeScript.

<div id="2-point-the-macos-app-at-your-running-gateway">
  ### 2) Pointez l’app macOS vers votre Gateway en cours d’exécution
</div>

Dans **OpenClaw.app** :

* Mode de connexion : **Local**
  L’app se connectera à la Gateway en cours d’exécution sur le port configuré.

<div id="3-verify">
  ### 3) Vérifier
</div>

* Le statut du Gateway dans l’application doit afficher **« Using existing gateway … »**
* Ou via la CLI :

```bash
openclaw health
```

<div id="common-footguns">
  ### Pièges courants
</div>

* **Mauvais port :** le WS du Gateway utilise par défaut `ws://127.0.0.1:18789` ; conservez l’application et la CLI sur le même port.
* **Où est stocké l’état :**
  * Identifiants : `~/.openclaw/credentials/`
  * Sessions : `~/.openclaw/agents/<agentId>/sessions/`
  * Journaux : `/tmp/openclaw/`

<div id="credential-storage-map">
  ## Cartographie du stockage des identifiants
</div>

Utilisez ceci pour le débogage de l’authentification ou pour décider ce qu’il faut sauvegarder :

* **WhatsApp** : `~/.openclaw/credentials/whatsapp/<accountId>/creds.json`
* **Jeton de bot Telegram** : config/env ou `channels.telegram.tokenFile`
* **Jeton de bot Discord** : config/env (fichier de jeton pas encore pris en charge)
* **Jetons Slack** : config/env (`channels.slack.*`)
* **Listes d’autorisation d’appairage** : `~/.openclaw/credentials/<channel>-allowFrom.json`
* **Profils d’authentification de modèle** : `~/.openclaw/agents/<agentId>/agent/auth-profiles.json`
* **Import d’OAuth hérité** : `~/.openclaw/credentials/oauth.json`

Plus de détails : [Sécurité](/fr/gateway/security#credential-storage-map).

<div id="updating-without-wrecking-your-setup">
  ## Mise à jour (sans casser votre configuration)
</div>

* Considérez `~/.openclaw/workspace` et `~/.openclaw/` comme « vos fichiers » ; ne mettez pas de prompts/configurations personnelles dans le dépôt `openclaw`.
* Mise à jour des sources : `git pull` + `pnpm install` (lorsque le fichier de verrouillage (lockfile) a changé) + continuez à utiliser `pnpm gateway:watch`.

<div id="linux-systemd-user-service">
  ## Linux (service utilisateur systemd)
</div>

Les installations Linux utilisent un service **utilisateur** systemd. Par défaut, systemd arrête les services utilisateur à la déconnexion ou en cas d’inactivité, ce qui stoppe Gateway. La procédure d’onboarding tente d’activer le mode *lingering* pour vous (peut nécessiter sudo). S’il est encore désactivé, exécutez :

```bash
sudo loginctl enable-linger $USER
```

Pour des serveurs toujours en fonctionnement ou multi-utilisateurs, envisagez un service **système** plutôt qu’un service utilisateur (sans avoir besoin d’activer le mode *linger*). Voir le [runbook Gateway](/fr/gateway) pour les notes sur systemd.

<div id="related-docs">
  ## Documentation connexe
</div>

* [Gateway runbook](/fr/gateway) (flags, supervision, ports)
* [Configuration de Gateway](/fr/gateway/configuration) (schéma de configuration + exemples)
* [Discord](/fr/channels/discord) et [Telegram](/fr/channels/telegram) (tags de réponse + paramètres replyToMode)
* [Configuration de l&#39;assistant OpenClaw](/fr/start/openclaw)
* [Application macOS](/fr/platforms/macos) (cycle de vie de Gateway)