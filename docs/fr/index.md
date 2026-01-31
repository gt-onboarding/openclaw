---
title: Index
summary: "Top-level overview of OpenClaw, features, and purpose"
read_when:
  - Introducing OpenClaw to newcomers
---

<div id="openclaw">
  # OpenClaw 🦞
</div>

> _"EXFOLIATE! EXFOLIATE!"_ — A space lobster, probably

<p align="center">
    <picture>
        <source media="(prefers-color-scheme: light)" srcset="https://raw.githubusercontent.com/openclaw/openclaw/main/docs/assets/openclaw-logo-text-dark.png">
        <img src="https://raw.githubusercontent.com/openclaw/openclaw/main/docs/assets/openclaw-logo-text.png" alt="OpenClaw" width="500">
    </picture>
</p>

<p align='center'>
  <strong>
    Any OS + WhatsApp/Telegram/Discord/iMessage gateway for AI agents (Pi).
  </strong>
  <br />
  Plugins add Mattermost and more. Send a message, get an agent response — from
  your pocket.
</p>

<p align='center'>
  <a href='https://github.com/openclaw/openclaw'>GitHub</a> ·
  <a href='https://github.com/openclaw/openclaw/releases'>Releases</a> ·
  <a href='/'>Docs</a> ·<a href='/start/openclaw'>OpenClaw assistant setup</a>
</p>

OpenClaw fait le pont entre WhatsApp (via WhatsApp Web / Baileys), Telegram (Bot API / grammY), Discord (Bot API / channels.discord.js) et iMessage (imsg CLI) vers des agents de codage comme [Pi](https://github.com/badlogic/pi-mono). Les plugins ajoutent Mattermost (Bot API + WebSocket) et plus encore.
OpenClaw alimente également l'assistant OpenClaw.


<div id="start-here">
  ## Pour commencer
</div>

- **Nouvelle installation depuis zéro :** [Prise en main](/start/getting-started)
- **Configuration guidée (recommandée) :** [Assistant](/start/wizard) (`openclaw onboard`)
- **Ouvrir le tableau de bord (Gateway en local) :** http://127.0.0.1:18789/ (ou http://localhost:18789/)

Si la Gateway tourne sur le même ordinateur, ce lien ouvre immédiatement le Control UI dans le navigateur. Si cela ne fonctionne pas, démarrez d’abord la Gateway : `openclaw gateway`.



<div id="dashboard-browser-control-ui">
  ## Tableau de bord (Control UI dans le navigateur)
</div>

Le tableau de bord est la Control UI accessible via le navigateur pour le chat, la configuration, les nœuds, les sessions, etc.
Par défaut en local : http://127.0.0.1:18789/
Accès à distance : [Surfaces Web](/web) et [Tailscale](/gateway/tailscale)

<p align="center">
  <img src="whatsapp-openclaw.jpg" alt="OpenClaw" width="420" />
</p>



<div id="how-it-works">
  ## Fonctionnement
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

La plupart des opérations transitent par le **Gateway** (`openclaw gateway`), un processus unique et persistant qui gère les connexions de canaux et le plan de contrôle WebSocket.


<div id="network-model">
  ## Modèle réseau
</div>

- **Un Gateway par hôte (recommandé)** : c’est le seul processus autorisé à détenir la session WhatsApp Web. Si vous avez besoin d’un bot de secours ou d’une isolation stricte, exécutez plusieurs gateways avec des profils et des ports isolés ; voir [Multiple gateways](/gateway/multiple-gateways).
- **Loopback en priorité** : le WS du Gateway utilise par défaut `ws://127.0.0.1:18789`.
  - L’assistant génère désormais par défaut un jeton de gateway (même pour le loopback).
  - Pour un accès via Tailnet, exécutez `openclaw gateway --bind tailnet --token ...` (un jeton est requis pour les liaisons non-loopback).
- **Nœuds** : se connectent au WebSocket du Gateway (LAN/tailnet/SSH selon les besoins) ; le pont TCP hérité est obsolète/supprimé.
- **Hôte Canvas** : serveur de fichiers HTTP sur `canvasHost.port` (par défaut `18793`), servant `/__openclaw__/canvas/` pour les WebViews des nœuds ; voir [Gateway configuration](/gateway/configuration) (`canvasHost`).
- **Utilisation à distance** : tunnel SSH ou tailnet/VPN ; voir [Accès à distance](/gateway/remote) et [Découverte](/gateway/discovery).



<div id="features-high-level">
  ## Fonctionnalités (vue d’ensemble)
</div>

- 📱 **Intégration WhatsApp** — Utilise Baileys pour le protocole WhatsApp Web
- ✈️ **Bot Telegram** — MP + groupes via grammY
- 🎮 **Bot Discord** — MP + salons de serveurs via channels.discord.js
- 🧩 **Bot Mattermost (plugin)** — Jeton de bot + événements WebSocket
- 💬 **iMessage** — Intégration locale de la CLI `imsg` (macOS)
- 🤖 **Passerelle d’agent** — Pi (mode RPC) avec streaming des outils
- ⏱️ **Streaming + découpage** — Détails sur le streaming par blocs + le streaming des brouillons Telegram ([/concepts/streaming](/concepts/streaming))
- 🧠 **Routage multi-agents** — Route les comptes de fournisseurs/peers vers des agents isolés (espace de travail + sessions par agent)
- 🔐 **Authentification par abonnement** — Anthropic (Claude Pro/Max) + OpenAI (ChatGPT/Codex) via OAuth
- 💬 **Sessions** — Les conversations directes sont regroupées dans `main` (par défaut) ; les groupes sont isolés
- 👥 **Prise en charge des discussions de groupe** — Basée sur les mentions par défaut ; le propriétaire peut basculer `/activation always|mention`
- 📎 **Prise en charge des médias** — Envoi et réception d’images, d’audio et de documents
- 🎤 **Notes vocales** — Hook de transcription optionnel
- 🖥️ **WebChat + application macOS** — UI locale + compagnon dans la barre de menus pour les opérations et le réveil vocal
- 📱 **Nœud iOS** — S’apparie en tant que nœud et expose une surface Canvas
- 📱 **Nœud Android** — S’apparie en tant que nœud et expose Canvas + Chat + Camera

Remarque : les chemins hérités Claude/Codex/Gemini/Opencode ont été supprimés ; Pi est le seul chemin d’agent de programmation.



<div id="quick-start">
  ## Démarrage rapide
</div>

Prérequis d’exécution : **Node ≥ 22**.



```bash
# Recommandé : installation globale (npm/pnpm)
npm install -g openclaw@latest
# ou : pnpm add -g openclaw@latest
```


# Initialiser + installer le service (service utilisateur launchd/systemd)
openclaw onboard --install-daemon



# Associer WhatsApp Web (affiche un QR code)
openclaw channels login



# Gateway s’exécute en tant que service après l’étape d’onboarding ; un lancement manuel reste toujours possible :

openclaw gateway --port 18789

````

Basculer entre les installations npm et git par la suite est simple : installez l'autre variante et exécutez `openclaw doctor` pour mettre à jour le point d'entrée du service Gateway.

Depuis les sources (développement) :

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm ui:build # installe automatiquement les dépendances UI à la première exécution
pnpm build
openclaw onboard --install-daemon
````

Si vous n’avez pas encore d’installation globale, exécutez l’étape d’onboarding avec `pnpm openclaw ...` depuis le dépôt.

Démarrage rapide multi‑instances (facultatif) :

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json \
OPENCLAW_STATE_DIR=~/.openclaw-a \
openclaw gateway --port 19001
```

Envoyez un message de test (nécessite que Gateway soit en cours d’exécution) :

```bash
openclaw message send --target +15555550123 --message "Hello from OpenClaw"
```


<div id="credits">
  ## Configuration (optionnelle)
</div>

La configuration se trouve dans `~/.openclaw/openclaw.json`.

* Si vous **ne faites rien**, OpenClaw utilise le binaire Pi fourni en mode RPC avec des sessions par expéditeur.
* Si vous voulez restreindre l’accès, commencez avec `channels.whatsapp.allowFrom` et (pour les groupes) les règles de mentions.

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


<div id="core-contributors">
  ## Docs
</div>

- Commencez ici :
  - [Docs hubs (toutes les pages liées)](/start/hubs)
  - [Help](/help) ← *correctifs courants + dépannage*
  - [Configuration](/gateway/configuration)
  - [Exemples de configuration](/gateway/configuration-examples)
  - [Commandes slash](/tools/slash-commands)
  - [Routage multi‑agents](/concepts/multi-agent)
  - [Mise à jour / rollback](/install/updating)
  - [Appairage (DM + nœuds)](/start/pairing)
  - [Mode Nix](/install/nix)
  - [Configuration de l’assistant OpenClaw](/start/openclaw)
  - [Compétences](/tools/skills)
  - [Configuration des compétences](/tools/skills-config)
  - [Modèles d’espace de travail](/reference/templates/AGENTS)
  - [Adaptateurs RPC](/reference/rpc)
  - [Runbook du Gateway](/gateway)
  - [Nœuds (iOS/Android)](/nodes)
  - [Surfaces web (Control UI)](/web)
  - [Découverte + transports](/gateway/discovery)
  - [Accès distant](/gateway/remote)
- Fournisseurs et UX :
  - [WebChat](/web/webchat)
  - [Control UI (navigateur)](/web/control-ui)
  - [Telegram](/channels/telegram)
  - [Discord](/channels/discord)
  - [Mattermost (plugin)](/channels/mattermost)
  - [iMessage](/channels/imessage)
  - [Groupes](/concepts/groups)
  - [Messages de groupe WhatsApp](/concepts/group-messages)
  - [Médias : images](/nodes/images)
  - [Médias : audio](/nodes/audio)
- Applications compagnons :
  - [Application macOS](/platforms/macos)
  - [Application iOS](/platforms/ios)
  - [Application Android](/platforms/android)
  - [Windows (WSL2)](/platforms/windows)
  - [Application Linux](/platforms/linux)
- Exploitation et sécurité :
  - [Sessions](/concepts/session)
  - [Tâches Cron](/automation/cron-jobs)
  - [Webhooks](/automation/webhook)
  - [Hooks Gmail (Pub/Sub)](/automation/gmail-pubsub)
  - [Sécurité](/gateway/security)
  - [Dépannage](/gateway/troubleshooting)



<div id="license">
  ## Le nom
</div>

**OpenClaw = CLAW + TARDIS** — parce que chaque homard spatial a besoin d’une machine à voyager dans le temps et l’espace.

---

*« Nous ne faisons que jouer avec nos propres prompts. »* — une IA, probablement défoncée aux tokens



## Crédits

- **Peter Steinberger** ([@steipete](https://twitter.com/steipete)) — Créateur, charmeur de homards
- **Mario Zechner** ([@badlogicc](https://twitter.com/badlogicgames)) — Créateur du Pi, spécialiste en tests d'intrusion
- **Clawd** — Le homard de l'espace qui a exigé un meilleur nom



## Contributeurs principaux

- **Maxim Vovshin** (@Hyaxia, 36747317+Hyaxia@users.noreply.github.com) — Skill Blogwatcher
- **Nacho Iacovino** (@nachoiacovino, nacho.iacovino@gmail.com) — Analyse des données de localisation (Telegram et WhatsApp)



## Licence

MIT — Libre comme un homard dans l’océan 🦞

---

*« On ne fait tous que jouer avec nos propres prompts. »* — Une IA, probablement gavée de tokens
