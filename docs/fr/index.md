---
title: Index
summary: "Vue d’ensemble générale d’OpenClaw, de ses fonctionnalités et de sa finalité"
read_when:
  - Présentation d’OpenClaw aux nouveaux utilisateurs
---

<div id="openclaw">
  # OpenClaw 🦞
</div>

> *« EXFOLIEZ ! EXFOLIEZ ! »* — Un homard de l’espace, probablement

<p align="center">
  <picture>
    <source media="(prefers-color-scheme: light)" srcset="https://raw.githubusercontent.com/openclaw/openclaw/main/docs/assets/openclaw-logo-text-dark.png" />

    <img src="https://raw.githubusercontent.com/openclaw/openclaw/main/docs/assets/openclaw-logo-text.png" alt="OpenClaw" width="500" />
  </picture>
</p>

<p align="center">
  <strong>Passerelle pour agents IA (Pi) fonctionnant sur n’importe quel OS avec WhatsApp/Telegram/Discord/iMessage.</strong><br />
  Les plugins ajoutent Mattermost et plus encore.
  Envoyez un message, recevez la réponse d’un agent — directement depuis votre poche.
</p>

<p align="center">
  <a href="https://github.com/openclaw/openclaw">GitHub</a> ·
  <a href="https://github.com/openclaw/openclaw/releases">Versions</a> ·
  <a href="/fr/">Documentation</a> ·
  <a href="/fr/start/openclaw">Configuration de l’assistant OpenClaw</a>
</p>

OpenClaw fait le lien entre WhatsApp (via WhatsApp Web / Baileys), Telegram (Bot API / grammY), Discord (Bot API / channels.discord.js) et iMessage (imsg CLI), et des agents dédiés au code comme [Pi](https://github.com/badlogic/pi-mono). Les plugins ajoutent Mattermost (Bot API + WebSocket) et d’autres fonctionnalités.
OpenClaw alimente également l’assistant OpenClaw.

<div id="start-here">
  ## Commencer ici
</div>

* **Nouvelle installation à partir de zéro :** [Pour bien débuter](/fr/start/getting-started)
* **Configuration guidée (recommandée) :** [Assistant](/fr/start/wizard) (`openclaw onboard`)
* **Ouvrir le tableau de bord (Gateway local) :** http://127.0.0.1:18789/ (ou http://localhost:18789/)

Si le Gateway s’exécute sur le même ordinateur, ce lien ouvre immédiatement le Control UI dans votre navigateur. Si cela ne fonctionne pas, démarrez d’abord le Gateway : `openclaw gateway`.

<div id="dashboard-browser-control-ui">
  ## Dashboard (Control UI dans le navigateur)
</div>

Le dashboard est l’interface Control UI dans le navigateur pour le chat, la configuration, les nœuds, les sessions et plus encore.
Adresse locale par défaut : http://127.0.0.1:18789/
Accès distant : [Surfaces web](/fr/web) et [Tailscale](/fr/gateway/tailscale)

<p align="center">
  <img src="/whatsapp-openclaw.jpg" alt="OpenClaw" width="420" />
</p>

<div id="how-it-works">
  ## Comment ça fonctionne
</div>

```
WhatsApp / Telegram / Discord / iMessage (+ plugins)
        │
        ▼
  ┌───────────────────────────┐
  │          Gateway          │  ws://127.0.0.1:18789 (loopback-only)
  │     (source unique)       │
  │                           │  http://<gateway-host>:18793
  │                           │    /__openclaw__/canvas/ (Canvas host)
  └───────────┬───────────────┘
              │
              ├─ Pi agent (RPC)
              ├─ CLI (openclaw …)
              ├─ Chat UI (SwiftUI)
              ├─ macOS app (OpenClaw.app)
              ├─ iOS node via Gateway WS + pairing
              └─ Android node via Gateway WS + pairing
```

La plupart des opérations passent par le **Gateway** (`openclaw gateway`), un processus unique conçu pour tourner en continu, qui gère les connexions de canaux ainsi que le plan de contrôle WebSocket.

<div id="network-model">
  ## Modèle réseau
</div>

* **Un Gateway par hôte (recommandé)** : c’est le seul processus autorisé à détenir la session WhatsApp Web. Si vous avez besoin d’un bot de secours ou d’une isolation stricte, exécutez plusieurs gateways avec des profils et des ports isolés ; voir [Plusieurs gateways](/fr/gateway/multiple-gateways).
* **Loopback en priorité** : le WS du Gateway utilise par défaut `ws://127.0.0.1:18789`.
  * L’assistant génère désormais un jeton de gateway par défaut (même pour le loopback).
  * Pour un accès via Tailnet, exécutez `openclaw gateway --bind tailnet --token ...` (un jeton est requis pour les liaisons non-loopback).
* **Nœuds** : se connectent au WebSocket du Gateway (LAN/tailnet/SSH selon les besoins) ; l’ancien pont TCP est déprécié/supprimé.
* **Hôte Canvas** : serveur de fichiers HTTP sur `canvasHost.port` (par défaut `18793`), servant `/__openclaw__/canvas/` pour les WebViews des nœuds ; voir [Configuration du Gateway](/fr/gateway/configuration) (`canvasHost`).
* **Utilisation à distance** : tunnel SSH ou tailnet/VPN ; voir [Accès distant](/fr/gateway/remote) et [Découverte](/fr/gateway/discovery).

<div id="features-high-level">
  ## Fonctionnalités (vue d’ensemble)
</div>

* 📱 **Intégration WhatsApp** — Utilise Baileys pour le protocole WhatsApp Web
* ✈️ **Bot Telegram** — MP (DM) + groupes via grammY
* 🎮 **Bot Discord** — MP (DM) + salons de guildes via channels.discord.js
* 🧩 **Bot Mattermost (plugin)** — Jeton de bot + événements WebSocket
* 💬 **iMessage** — Intégration locale de la CLI imsg (macOS)
* 🤖 **Passerelle d’Agent** — Pi (mode RPC) avec streaming d’outils
* ⏱️ **Streaming + découpage en blocs** — Streaming par blocs + détails sur le streaming de brouillons Telegram ([/concepts/streaming](/fr/concepts/streaming))
* 🧠 **Routage multi-agents** — Achemine les comptes/peers de fournisseurs vers des agents isolés (espace de travail + sessions par agent)
* 🔐 **Authentification par abonnement** — Anthropic (Claude Pro/Max) + OpenAI (ChatGPT/Codex) via OAuth
* 💬 **Sessions** — Les discussions directes se replient dans `main` (par défaut) ; les groupes sont isolés
* 👥 **Prise en charge des discussions de groupe** — Basée sur la mention par défaut ; le propriétaire peut basculer `/activation always|mention`
* 📎 **Prise en charge des médias** — Envoi et réception d’images, d’audio et de documents
* 🎤 **Messages vocaux** — Hook de transcription optionnel
* 🖥️ **WebChat + app macOS** — UI locale + compagnon dans la barre de menus pour les opérations et le réveil vocal
* 📱 **Nœud iOS** — S’apparie comme nœud et expose une surface Canvas
* 📱 **Nœud Android** — S’apparie comme nœud et expose Canvas + Chat + Camera

Remarque : les chemins hérités Claude/Codex/Gemini/Opencode ont été supprimés ; Pi est le seul chemin d’agent de programmation.

<div id="quick-start">
  ## Démarrage rapide
</div>

Exigence d’exécution : **Node ≥ 22**.

```bash
# Recommended: global install (npm/pnpm)
npm install -g openclaw@latest
# or: pnpm add -g openclaw@latest

# Onboard + install the service (launchd/systemd user service)
openclaw onboard --install-daemon

# Pair WhatsApp Web (shows QR)
openclaw channels login

# Le Gateway s'exécute via le service après l'intégration ; l'exécution manuelle reste possible :
openclaw gateway --port 18789
```

Passer plus tard d’une installation via npm à une installation depuis git (ou inversement) est simple : installez l’autre variante puis exécutez `openclaw doctor` pour mettre à jour le point d’entrée du service Gateway.

Depuis les sources (développement) :

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm ui:build # installe automatiquement les dépendances de l'UI à la première exécution
pnpm build
openclaw onboard --install-daemon
```

Si vous n’avez pas encore d’installation globale, exécutez l’étape d’onboarding via `pnpm openclaw ...` depuis le dépôt.

Démarrage rapide multi-instance (facultatif) :

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json \
OPENCLAW_STATE_DIR=~/.openclaw-a \
openclaw gateway --port 19001
```

Envoyez un message de test (nécessite le Gateway en cours d’exécution) :

```bash
openclaw message send --target +15555550123 --message "Hello from OpenClaw"
```

<div id="configuration-optional">
  ## Configuration (optionnelle)
</div>

La configuration se trouve dans `~/.openclaw/openclaw.json`.

* Si vous **ne faites rien**, OpenClaw utilise le binaire Pi intégré en mode RPC avec des sessions par expéditeur.
* Si vous voulez restreindre l’accès, commencez par `channels.whatsapp.allowFrom` et, pour les groupes, définissez des règles de mentions.

Exemple :

```json5
{
  channels: {
    whatsapp: {
      allowFrom: ["+15555550123"],
      groups: { "*": { requireMention: true } }
    }
  },
  messages: { groupChat: { mentionPatterns: ["@openclaw"] } }
}
```

<div id="docs">
  ## Documentation
</div>

* Commencer ici :
  * [Docs hubs (toutes les pages référencées)](/fr/start/hubs)
  * [Aide](/fr/help) ← *corrections courantes + dépannage*
  * [Configuration](/fr/gateway/configuration)
  * [Exemples de configuration](/fr/gateway/configuration-examples)
  * [Commandes slash](/fr/tools/slash-commands)
  * [Routage multi-agents](/fr/concepts/multi-agent)
  * [Mise à jour / retour en arrière](/fr/install/updating)
  * [Appairage (DM + nœuds)](/fr/start/pairing)
  * [Mode Nix](/fr/install/nix)
  * [Configuration de l&#39;assistant OpenClaw](/fr/start/openclaw)
  * [Compétences](/fr/tools/skills)
  * [Configuration des compétences](/fr/tools/skills-config)
  * [Modèles d&#39;espaces de travail](/fr/reference/templates/AGENTS)
  * [Adaptateurs RPC](/fr/reference/rpc)
  * [Runbook du Gateway](/fr/gateway)
  * [Nœuds (iOS/Android)](/fr/nodes)
  * [Interfaces web (Control UI)](/fr/web)
  * [Découverte + transports](/fr/gateway/discovery)
  * [Accès à distance](/fr/gateway/remote)
* Fournisseurs et UX :
  * [WebChat](/fr/web/webchat)
  * [Control UI (navigateur)](/fr/web/control-ui)
  * [Telegram](/fr/channels/telegram)
  * [Discord](/fr/channels/discord)
  * [Mattermost (plugin)](/fr/channels/mattermost)
  * [iMessage](/fr/channels/imessage)
  * [Groupes](/fr/concepts/groups)
  * [Messages de groupe WhatsApp](/fr/concepts/group-messages)
  * [Médias : images](/fr/nodes/images)
  * [Médias : audio](/fr/nodes/audio)
* Applications compagnons :
  * [Application macOS](/fr/platforms/macos)
  * [Application iOS](/fr/platforms/ios)
  * [Application Android](/fr/platforms/android)
  * [Windows (WSL2)](/fr/platforms/windows)
  * [Application Linux](/fr/platforms/linux)
* Exploitation et sécurité :
  * [Sessions](/fr/concepts/session)
  * [Tâches cron](/fr/automation/cron-jobs)
  * [Webhooks](/fr/automation/webhook)
  * [Hooks Gmail (Pub/Sub)](/fr/automation/gmail-pubsub)
  * [Sécurité](/fr/gateway/security)
  * [Dépannage](/fr/gateway/troubleshooting)

<div id="the-name">
  ## Le nom
</div>

**OpenClaw = CLAW + TARDIS** — parce que chaque homard de l’espace a besoin d’une machine à voyager dans le temps et dans l’espace.

***

*« On fait tous que jouer avec nos propres prompts. »* — une IA, probablement défoncée aux tokens

<div id="credits">
  ## Crédits
</div>

* **Peter Steinberger** ([@steipete](https://twitter.com/steipete)) — Créateur, charmeur de homards
* **Mario Zechner** ([@badlogicc](https://twitter.com/badlogicgames)) — Créateur de Pi, spécialiste des tests d&#39;intrusion
* **Clawd** — Le homard de l&#39;espace qui a exigé un meilleur nom

<div id="core-contributors">
  ## Contributeurs principaux
</div>

* **Maxim Vovshin** (@Hyaxia, 36747317+Hyaxia@users.noreply.github.com) — skill Blogwatcher
* **Nacho Iacovino** (@nachoiacovino, nacho.iacovino@gmail.com) — analyse de localisation (Telegram + WhatsApp)

<div id="license">
  ## Licence
</div>

MIT — Libre comme un homard dans l’océan 🦞

***

*« On fait tous juste jouer avec nos propres prompts. »* — Une IA, probablement défoncée aux tokens