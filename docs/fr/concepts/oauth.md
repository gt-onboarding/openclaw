---
title: OAuth
summary: "OAuth dans OpenClaw : échange de jetons, stockage et schémas multi-comptes"
read_when:
  - Vous souhaitez comprendre de bout en bout OAuth dans OpenClaw
  - Vous rencontrez des problèmes d'invalidation de jetons ou de déconnexion
  - Vous souhaitez des flux d'authentification setup-token ou OAuth
  - Vous souhaitez gérer plusieurs comptes ou le routage par profil
---

<div id="oauth">
  # OAuth
</div>

OpenClaw prend en charge l’« authentification par abonnement » via OAuth pour les fournisseurs qui la proposent (notamment **OpenAI Codex (ChatGPT OAuth)**). Pour les abonnements Anthropic, utilisez le flux **setup-token**. Cette page explique :

* comment fonctionne l’**échange de jetons** OAuth (PKCE)
* où les jetons sont **stockés** (et pourquoi)
* comment gérer **plusieurs comptes** (profils + remplacements par session)

OpenClaw prend aussi en charge des **plugins de fournisseur** qui incluent leurs propres flux OAuth ou à clé API. Exécutez-les via :

```bash
openclaw models auth login --provider <id>
```

<div id="the-token-sink-why-it-exists">
  ## Le puits de jetons (pourquoi il existe)
</div>

Les fournisseurs OAuth créent généralement un **nouveau jeton d’actualisation** lors des flux de connexion ou de rafraîchissement. Certains fournisseurs (ou clients OAuth) peuvent invalider d’anciens jetons d’actualisation lorsqu’un nouveau est émis pour le même utilisateur/application.

Symptôme concret :

* vous vous connectez via OpenClaw *et* via Claude Code / Codex CLI → l’un des deux se retrouve aléatoirement « déconnecté » plus tard

Pour limiter ce problème, OpenClaw traite `auth-profiles.json` comme un **puits de jetons** :

* le runtime lit les identifiants depuis **un seul endroit**
* nous pouvons conserver plusieurs profils et les router de manière déterministe

<div id="storage-where-tokens-live">
  ## Stockage (emplacement des jetons)
</div>

Les secrets sont stockés **par agent** :

* Profils d’authentification (OAuth + clés API) : `~/.openclaw/agents/<agentId>/agent/auth-profiles.json`
* Cache d’exécution (géré automatiquement ; ne pas modifier) : `~/.openclaw/agents/<agentId>/agent/auth.json`

Fichier hérité réservé à l’import (toujours pris en charge, mais ce n’est plus le stockage principal) :

* `~/.openclaw/credentials/oauth.json` (importé dans `auth-profiles.json` lors de la première utilisation)

Tout ce qui précède respecte également `$OPENCLAW_STATE_DIR` (remplacement du répertoire d’état). Référence complète : [/gateway/configuration](/fr/gateway/configuration#auth-storage-oauth--api-keys)

<div id="anthropic-setup-token-subscription-auth">
  ## Jeton de configuration Anthropic (authentification par abonnement)
</div>

Exécutez la commande `claude setup-token` sur n’importe quelle machine, puis collez le jeton dans OpenClaw :

```bash
openclaw models auth setup-token --provider anthropic
```

Si vous avez généré le jeton ailleurs, collez-le ici manuellement :

```bash
openclaw models auth paste-token --provider anthropic
```

Vérifier :

```bash
openclaw models status
```

<div id="oauth-exchange-how-login-works">
  ## Échange OAuth (fonctionnement de la connexion)
</div>

Les flux de connexion interactifs d’OpenClaw sont implémentés dans `@mariozechner/pi-ai` et intégrés aux assistants et aux commandes.

<div id="anthropic-claude-promax-setup-token">
  ### Configuration du setup-token Anthropic (Claude Pro/Max)
</div>

Schéma du flux :

1. exécutez `claude setup-token`
2. collez le jeton dans OpenClaw
3. enregistrez-le en tant que profil d&#39;authentification par jeton (sans rafraîchissement)

Le chemin de l&#39;assistant est `openclaw onboard` → option d&#39;authentification `setup-token` (Anthropic).

<div id="openai-codex-chatgpt-oauth">
  ### OpenAI Codex (ChatGPT OAuth)
</div>

Schéma du flux (PKCE) :

1. générer un vérificateur/défi PKCE + un `state` aléatoire
2. ouvrir `https://auth.openai.com/oauth/authorize?...`
3. essayer de capturer le callback sur `http://127.0.0.1:1455/auth/callback`
4. si le callback ne peut pas être capturé (ou si vous êtes à distance/en mode headless), collez l’URL/le code de redirection
5. échanger le code sur `https://auth.openai.com/oauth/token`
6. extraire `accountId` du jeton d’accès et stocker `{ access, refresh, expires, accountId }`

Le chemin de l’assistant interactif est `openclaw onboard` → option d’authentification `openai-codex`.

<div id="refresh-expiry">
  ## Rafraîchissement + expiration
</div>

Les profils stockent un horodatage `expires`.

À l&#39;exécution :

* si `expires` est dans le futur → utiliser le jeton d&#39;accès stocké
* si expiré → rafraîchir (sous verrou de fichier) et écraser les informations d&#39;identification stockées

Le processus de rafraîchissement est automatique ; vous n&#39;avez généralement pas besoin de gérer les jetons manuellement.

<div id="multiple-accounts-profiles-routing">
  ## Plusieurs comptes (profils) + routage
</div>

Deux schémas :

<div id="1-preferred-separate-agents">
  ### 1) Recommandé : agents séparés
</div>

Si vous voulez que « personnel » et « travail » ne se croisent jamais, utilisez des agents isolés (sessions + identifiants + espace de travail séparés) :

```bash
openclaw agents add work
openclaw agents add personal
```

Puis configurez l’authentification par agent (assistant de configuration) et redirigez les conversations vers le bon agent.

<div id="2-advanced-multiple-profiles-in-one-agent">
  ### 2) Avancé : plusieurs profils dans un même agent
</div>

`auth-profiles.json` prend en charge plusieurs ID de profil pour le même fournisseur.

Choisissez quel profil est utilisé :

* globalement via l’ordre de configuration (`auth.order`)
* pour chaque session via `/model ...@<profileId>`

Exemple (priorité à la session) :

* `/model Opus@anthropic:work`

Comment lister les ID de profil disponibles :

* `openclaw channels list --json` (affiche `auth[]`)

Documentation associée :

* [/concepts/model-failover](/fr/concepts/model-failover) (règles de rotation + cooldown)
* [/tools/slash-commands](/fr/tools/slash-commands) (surface de commande)