---
title: Appairage
summary: "Aperçu de l’appairage : approuver qui peut vous envoyer des MP et quels nœuds peuvent se connecter"
read_when:
  - Configuration du contrôle d’accès aux MP
  - Appairage d’un nouveau nœud iOS/Android
  - Analyse de la posture de sécurité d’OpenClaw
---

<div id="pairing">
  # Appairage
</div>

L’« appairage » correspond à l’étape d’**approbation explicite par le propriétaire** dans OpenClaw.
Elle est utilisée à deux endroits :

1. **Appairage DM** (qui est autorisé à parler au bot)
2. **Appairage de nœud** (quels appareils/nœuds sont autorisés à rejoindre le réseau du Gateway)

Contexte de sécurité : [Sécurité](/fr/gateway/security)

<div id="1-dm-pairing-inbound-chat-access">
  ## 1) Appairage par DM (accès au chat entrant)
</div>

Lorsqu&#39;un canal est configuré avec la politique de DM `pairing`, les expéditeurs inconnus reçoivent un code court et leur message **n&#39;est pas traité** tant que vous ne l&#39;avez pas approuvé.

Les politiques de DM par défaut sont documentées ici : [Sécurité](/fr/gateway/security)

Codes d&#39;appairage :

* 8 caractères, majuscules, sans caractères ambigus (`0O1I`).
* **Expirent au bout d&#39;une heure**. Le bot n&#39;envoie le message d&#39;appairage que lorsqu&#39;une nouvelle requête est créée (environ une fois par heure et par expéditeur).
* Les demandes d&#39;appairage DM en attente sont limitées à **3 par canal** par défaut ; les demandes supplémentaires sont ignorées jusqu&#39;à ce que l&#39;une d&#39;entre elles expire ou soit approuvée.

<div id="approve-a-sender">
  ### Autoriser un expéditeur
</div>

```bash
openclaw pairing list telegram
openclaw pairing approve telegram <CODE>
```

Canaux pris en charge : `telegram`, `whatsapp`, `signal`, `imessage`, `discord`, `slack`.

### Où se trouve l’état

Stocké sous `~/.openclaw/credentials/` :

* Requêtes en attente : `<channel>-pairing.json`
* Stockage de la liste d’autorisation approuvée : `<channel>-allowFrom.json`

Considérez-les comme sensibles (ils conditionnent l’accès à votre assistant).

<div id="2-node-device-pairing-iosandroidmacosheadless-nodes">
  ## 2) Appairage des appareils nœuds (nœuds iOS/Android/macOS/sans interface)
</div>

Les nœuds se connectent au Gateway en tant qu’**appareils** avec `role: node`. Le Gateway
crée une demande d’appairage de l’appareil qui doit être approuvée.

<div id="approve-a-node-device">
  ### Approuver un nœud
</div>

```bash
openclaw devices list
openclaw devices approve <requestId>
openclaw devices reject <requestId>
```

<div id="where-the-state-lives">
  ### Emplacement de l&#39;état
</div>

Enregistré sous `~/.openclaw/devices/` :

* `pending.json` (à courte durée de vie ; les requêtes en attente expirent)
* `paired.json` (appareils couplés + jetons)

<div id="notes">
  ### Notes
</div>

* L’API héritée `node.pair.*` (CLI : `openclaw nodes pending/approve`) correspond à un registre d’appairage séparé, géré par le Gateway. Les nœuds WS nécessitent toujours un appairage de périphérique.

<div id="related-docs">
  ## Documentation associée
</div>

* Modèle de sécurité et injection de prompt : [Security](/fr/gateway/security)
* Mise à jour en toute sécurité (lancer doctor) : [Updating](/fr/install/updating)
* Configurations des canaux :
  * Telegram : [Telegram](/fr/channels/telegram)
  * WhatsApp : [WhatsApp](/fr/channels/whatsapp)
  * Signal : [Signal](/fr/channels/signal)
  * iMessage : [iMessage](/fr/channels/imessage)
  * Discord : [Discord](/fr/channels/discord)
  * Slack : [Slack](/fr/channels/slack)