---
title: Canaux
summary: "Plateformes de messagerie compatibles avec OpenClaw"
read_when:
  - Vous souhaitez choisir un canal de chat pour OpenClaw
  - Vous avez besoin d’un aperçu rapide des plateformes de messagerie compatibles
---

<div id="chat-channels">
  # Canaux de discussion
</div>

OpenClaw peut communiquer avec vous sur n&#39;importe quelle application de messagerie que vous utilisez déjà. Chaque canal se connecte via le Gateway.
Le texte est pris en charge partout ; la prise en charge des contenus multimédias et des réactions dépend du canal.

<div id="supported-channels">
  ## Canaux pris en charge
</div>

* [WhatsApp](/fr/channels/whatsapp) — Le plus populaire ; utilise Baileys et nécessite un appairage via QR.
* [Telegram](/fr/channels/telegram) — Bot API via grammY ; prend en charge les groupes.
* [Discord](/fr/channels/discord) — Discord Bot API + Gateway ; prend en charge les serveurs, canaux et DMs.
* [Slack](/fr/channels/slack) — Bolt SDK ; applications d’espaces de travail.
* [Google Chat](/fr/channels/googlechat) — Application utilisant l’API Google Chat via webhook HTTP.
* [Mattermost](/fr/channels/mattermost) — Bot API + WebSocket ; canaux, groupes, DMs (plugin, installé séparément).
* [Signal](/fr/channels/signal) — signal-cli ; axé sur la confidentialité.
* [BlueBubbles](/fr/channels/bluebubbles) — **Recommandé pour iMessage** ; utilise l’API REST du serveur BlueBubbles pour macOS avec prise en charge complète des fonctionnalités (modification, annulation d’envoi, effets, réactions, gestion des groupes — la fonction de modification est actuellement défaillante sur macOS 26 Tahoe).
* [iMessage](/fr/channels/imessage) — macOS uniquement ; intégration native via imsg (hérité, envisagez BlueBubbles pour les nouvelles installations).
* [Microsoft Teams](/fr/channels/msteams) — Bot Framework ; prise en charge des environnements d’entreprise (plugin, installé séparément).
* [LINE](/fr/channels/line) — Bot LINE Messaging API (plugin, installé séparément).
* [Nextcloud Talk](/fr/channels/nextcloud-talk) — Chat auto-hébergé via Nextcloud Talk (plugin, installé séparément).
* [Matrix](/fr/channels/matrix) — Protocole Matrix (plugin, installé séparément).
* [Nostr](/fr/channels/nostr) — DMs décentralisés via NIP-04 (plugin, installé séparément).
* [Tlon](/fr/channels/tlon) — Messagerie basée sur Urbit (plugin, installé séparément).
* [Twitch](/fr/channels/twitch) — Chat Twitch via connexion IRC (plugin, installé séparément).
* [Zalo](/fr/channels/zalo) — Zalo Bot API ; messagerie populaire au Vietnam (plugin, installé séparément).
* [Zalo Personal](/fr/channels/zalouser) — Compte Zalo personnel via connexion par QR (plugin, installé séparément).
* [WebChat](/fr/web/webchat) — UI WebChat du Gateway sur WebSocket.

<div id="notes">
  ## Remarques
</div>

* Les canaux peuvent fonctionner simultanément ; configurez-en plusieurs et OpenClaw effectuera le routage par conversation.
* La configuration la plus rapide est généralement **Telegram** (simple jeton de bot). WhatsApp nécessite un appairage via code QR et
  stocke davantage d’état sur le disque.
* Le comportement en groupe varie selon le canal ; voir [Groupes](/fr/concepts/groups).
* L’appairage en DM et les listes d’autorisation sont appliqués pour des raisons de sécurité ; voir [Sécurité](/fr/gateway/security).
* Fonctionnement interne de Telegram : [notes grammY](/fr/channels/grammy).
* Dépannage : [Dépannage des canaux](/fr/channels/troubleshooting).
* Les fournisseurs de modèles sont documentés séparément ; voir [Fournisseurs de modèles](/fr/providers/models).