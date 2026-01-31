---
title: Parcours d’intégration
summary: "Flux d’intégration au premier lancement d’OpenClaw (application macOS)"
read_when:
  - Concevoir l’assistant d’intégration pour macOS
  - Implémenter la configuration de l’authentification ou de l’identité
---

<div id="onboarding-macos-app">
  # Onboarding (application macOS)
</div>

Ce document décrit le parcours d&#39;onboarding **au premier lancement** actuel. L&#39;objectif est d&#39;offrir une expérience fluide de « jour 0 » : choisir où le Gateway s&#39;exécute, configurer l&#39;authentification, lancer l&#39;assistant de configuration et laisser l&#39;agent s&#39;initialiser tout seul.

<div id="page-order-current">
  ## Ordre des pages (actuel)
</div>

1. Bienvenue + avertissement de sécurité
2. **Sélection du Gateway** (Local / À distance / Configurer plus tard)
3. **Auth (Anthropic OAuth)** — uniquement en local
4. **Assistant de configuration** (piloté par le Gateway)
5. **Autorisations** (demandes d’autorisation TCC)
6. **CLI** (facultatif)
7. **Discussion d’onboarding** (session dédiée)
8. Prêt

<div id="1-local-vs-remote">
  ## 1) Local vs distant
</div>

Où le **Gateway** s’exécute‑t‑il ?

* **Local (ce Mac) :** le processus d’onboarding peut exécuter les flux OAuth et enregistrer les identifiants
  en local.
* **Distant (via SSH/Tailnet) :** le processus d’onboarding **n’**exécute **pas** OAuth en local ;
  les identifiants doivent déjà exister sur l’hôte du Gateway.
* **Configurer plus tard :** ignorer cette étape et laisser l’app non configurée.

Astuce pour l’authentification du Gateway :

* L’assistant génère désormais un **token** même pour le loopback, donc les clients WS locaux doivent s’authentifier.
* Si vous désactivez l’authentification, n’importe quel processus local peut se connecter ; n’utilisez cela que sur des machines entièrement fiables.
* Utilisez un **token** pour un accès depuis plusieurs machines ou pour des liaisons non loopback.

<div id="2-local-only-auth-anthropic-oauth">
  ## 2) Authentification locale uniquement (Anthropic OAuth)
</div>

L’application macOS prend en charge Anthropic OAuth (Claude Pro/Max). Déroulement :

* Ouvre le navigateur pour OAuth (PKCE)
* Demande à l’utilisateur de coller la valeur `code#state`
* Écrit les identifiants dans `~/.openclaw/credentials/oauth.json`

Les autres fournisseurs (OpenAI, API personnalisées) sont configurés via des variables d’environnement
ou des fichiers de configuration pour l’instant.

<div id="3-setup-wizard-gatewaydriven">
  ## 3) Assistant de configuration (piloté par le Gateway)
</div>

L’application peut exécuter le même assistant de configuration que la CLI. Cela permet de garder l’onboarding synchronisé avec le comportement du Gateway et d’éviter de dupliquer la logique dans SwiftUI.

<div id="4-permissions">
  ## 4) Autorisations
</div>

La procédure d’onboarding requiert les autorisations TCC nécessaires pour :

* Notifications
* Accessibilité
* Enregistrement de l’écran
* Microphone / reconnaissance vocale
* Automatisation (AppleScript)

<div id="5-cli-optional">
  ## 5) CLI (facultatif)
</div>

L’app peut installer la CLI globale `openclaw` via npm/pnpm afin que les workflows dans le terminal et les tâches launchd soient prêts à l’emploi.

<div id="6-onboarding-chat-dedicated-session">
  ## 6) Conversation de prise en main (session dédiée)
</div>

Après la configuration, l’application ouvre une session dédiée de conversation de prise en main afin que l’agent puisse
se présenter et vous guider pour la suite. Cela permet de garder cette aide au premier lancement séparée
de vos conversations habituelles.

<div id="agent-bootstrap-ritual">
  ## Rituel de bootstrap de l’agent
</div>

Lors de la première exécution d’un agent, OpenClaw initialise un espace de travail (par défaut `~/.openclaw/workspace`) :

* Préremplit (`seeds`) `AGENTS.md`, `BOOTSTRAP.md`, `IDENTITY.md`, `USER.md`
* Exécute un court rituel de questions-réponses (une question à la fois)
* Écrit l’identité et les préférences dans `IDENTITY.md`, `USER.md`, `SOUL.md`
* Supprime `BOOTSTRAP.md` une fois terminé afin qu’il ne s’exécute qu’une seule fois

<div id="optional-gmail-hooks-manual">
  ## Facultatif : hooks Gmail (configuration manuelle)
</div>

La configuration Pub/Sub de Gmail se fait actuellement manuellement. Utilisez :

```bash
openclaw webhooks gmail setup --account you@gmail.com
```

Pour plus de détails, voir [/automation/gmail-pubsub](/fr/automation/gmail-pubsub).

<div id="remote-mode-notes">
  ## Notes sur le mode distant
</div>

Lorsque le Gateway s&#39;exécute sur une autre machine, les identifiants et les fichiers d&#39;espace de travail résident
**sur cet hôte**. Si vous avez besoin d&#39;OAuth en mode distant, créez :

* `~/.openclaw/credentials/oauth.json`
* `~/.openclaw/agents/<agentId>/agent/auth-profiles.json`

sur l&#39;hôte du Gateway.