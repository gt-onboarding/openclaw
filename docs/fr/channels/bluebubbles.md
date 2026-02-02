---
title: Bluebubbles
summary: "iMessage via un serveur BlueBubbles sur macOS (envoi/réception via REST, indication de saisie, réactions, appairage, actions avancées)."
read_when:
  - Configurer le canal BlueBubbles
  - Dépanner l'appairage du webhook
  - Configurer iMessage sur macOS
---

<div id="bluebubbles-macos-rest">
  # BlueBubbles (macOS REST)
</div>

Statut : plugin intégré qui communique avec le serveur BlueBubbles sur macOS via HTTP. **Recommandé pour l’intégration avec iMessage** en raison de son API plus riche et de sa configuration plus simple par rapport à l’ancien canal imsg.

<div id="overview">
  ## Vue d’ensemble
</div>

* S’exécute sur macOS via l’application d’assistance BlueBubbles ([bluebubbles.app](https://bluebubbles.app)).
* Recommandé/testé : macOS Sequoia (15). macOS Tahoe (26) fonctionne ; l’édition est actuellement défaillante sur Tahoe et les mises à jour d’icône de groupe peuvent signaler une réussite sans se synchroniser.
* OpenClaw communique avec lui via son API REST (`GET /api/v1/ping`, `POST /message/text`, `POST /chat/:id/*`).
* Les messages entrants arrivent via des webhooks ; les réponses sortantes, indicateurs de saisie, accusés de lecture et tapbacks sont effectués par appels REST.
* Les pièces jointes et stickers sont traités comme médias entrants (et exposés à l’agent lorsque possible).
* L’appairage et la liste d’autorisation fonctionnent de la même manière que pour les autres canaux (`/start/pairing` etc.) avec `channels.bluebubbles.allowFrom` + codes d’appairage.
* Les réactions sont exposées comme événements système, exactement comme sur Slack/Telegram, afin que les agents puissent les « mentionner » avant de répondre.
* Fonctionnalités avancées : édition, annulation d’envoi, fils de réponses, effets de message, gestion de groupe.

<div id="quick-start">
  ## Démarrage rapide
</div>

1. Installez le serveur BlueBubbles sur votre Mac (suivez les instructions sur [bluebubbles.app/install](https://bluebubbles.app/install)).
2. Dans la configuration BlueBubbles, activez l&#39;API web et définissez un mot de passe.
3. Exécutez `openclaw onboard` et sélectionnez BlueBubbles, ou configurez manuellement :
   ```json5
   {
     channels: {
       bluebubbles: {
         enabled: true,
         serverUrl: "http://192.168.1.100:1234",
         password: "example-password",
         webhookPath: "/bluebubbles-webhook"
       }
     }
   }
   ```
4. Dirigez les webhooks BlueBubbles vers votre Gateway (exemple : `https://your-gateway-host:3000/bluebubbles-webhook?password=<password>`).
5. Démarrez le Gateway ; il enregistrera le gestionnaire de webhook et lancera l&#39;appairage.

<div id="onboarding">
  ## Prise en main
</div>

BlueBubbles est disponible dans l&#39;assistant de configuration interactif :

```
openclaw onboard
```

L’assistant vous demande :

* **URL du serveur** (obligatoire) : adresse du serveur BlueBubbles (par exemple `http://192.168.1.100:1234`)
* **Mot de passe** (obligatoire) : mot de passe API dans les paramètres de BlueBubbles Server
* **Chemin du webhook** (facultatif) : valeur par défaut : `/bluebubbles-webhook`
* **Politique de DM** : appairage, liste d’autorisation, open (paramètre autorisant l’acceptation de messages sans restriction de la part de n’importe quel utilisateur) ou désactivée
* **Liste d’autorisation** : numéros de téléphone, e-mails ou cibles de discussion

Vous pouvez aussi ajouter BlueBubbles via la CLI :

```
openclaw channels add bluebubbles --http-url http://192.168.1.100:1234 --password <password>
```

<div id="access-control-dms-groups">
  ## Contrôle d’accès (DMs + groupes)
</div>

DMs :

* Par défaut : `channels.bluebubbles.dmPolicy = "pairing"`.
* Les expéditeurs inconnus reçoivent un code d’appairage ; les messages sont ignorés jusqu’à approbation (les codes expirent après 1 heure).
* Approuver via :
  * `openclaw pairing list bluebubbles`
  * `openclaw pairing approve bluebubbles <CODE>`
* L’appairage est le mécanisme d’échange de jetons par défaut. Détails : [Pairing](/fr/start/pairing)

Groupes :

* `channels.bluebubbles.groupPolicy = open | allowlist | disabled` (par défaut : `allowlist`).
* `channels.bluebubbles.groupAllowFrom` contrôle qui peut déclencher des actions dans les groupes lorsque `allowlist` est activé.

<div id="mention-gating-groups">
  ### Restriction par mention (groupes)
</div>

BlueBubbles prend en charge la restriction par mention pour les discussions de groupe, en reproduisant le comportement d’iMessage/WhatsApp :

* Utilise `agents.list[].groupChat.mentionPatterns` (ou `messages.groupChat.mentionPatterns`) pour détecter les mentions.
* Lorsque `requireMention` est activé pour un groupe, l’agent ne répond que lorsqu’il est mentionné.
* Les commandes de contrôle envoyées par des expéditeurs autorisés contournent la restriction par mention.

Configuration par groupe :

```json5
{
  channels: {
    bluebubbles: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15555550123"],
      groups: {
        "*": { requireMention: true },  // default for all groups
        "iMessage;-;chat123": { requireMention: false }  // remplacement pour un groupe spécifique
      }
    }
  }
}
```

<div id="command-gating">
  ### Contrôle d’accès aux commandes
</div>

* Les commandes de contrôle (par exemple `/config`, `/model`) nécessitent une autorisation.
* Utilise `allowFrom` et `groupAllowFrom` pour déterminer l’autorisation d’exécuter les commandes.
* Les expéditeurs autorisés peuvent exécuter des commandes de contrôle même sans être mentionnés dans les groupes.

<div id="typing-read-receipts">
  ## Saisie + accusés de lecture
</div>

* **Indicateurs de saisie** : envoyés automatiquement avant et pendant la génération de la réponse.
* **Accusés de lecture** : contrôlés par `channels.bluebubbles.sendReadReceipts` (valeur par défaut : `true`).
* **Indicateurs de saisie** : OpenClaw envoie des événements de début de saisie ; BlueBubbles efface automatiquement l’indicateur de saisie lors de l’envoi ou à l’expiration du délai (l’arrêt manuel via DELETE est peu fiable).

```json5
{
  channels: {
    bluebubbles: {
      sendReadReceipts: false  // désactiver les accusés de réception
    }
  }
}
```

<div id="advanced-actions">
  ## Actions avancées
</div>

BlueBubbles prend en charge des actions avancées sur les messages lorsqu’elles sont activées dans la configuration :

```json5
{
  channels: {
    bluebubbles: {
      actions: {
        reactions: true,       // tapbacks (default: true)
        edit: true,            // edit sent messages (macOS 13+, broken on macOS 26 Tahoe)
        unsend: true,          // unsend messages (macOS 13+)
        reply: true,           // reply threading by message GUID
        sendWithEffect: true,  // message effects (slam, loud, etc.)
        renameGroup: true,     // rename group chats
        setGroupIcon: true,    // définir l'icône/photo de discussion de groupe (instable sur macOS 26 Tahoe)
        addParticipant: true,  // add participants to groups
        removeParticipant: true, // remove participants from groups
        leaveGroup: true,      // leave group chats
        sendAttachment: true   // send attachments/media
      }
    }
  }
}
```

Actions disponibles :

* **react** : Ajouter ou supprimer des réactions Tapback (`messageId`, `emoji`, `remove`)
* **edit** : Modifier un message envoyé (`messageId`, `text`)
* **unsend** : Annuler l&#39;envoi d&#39;un message (`messageId`)
* **reply** : Répondre à un message spécifique (`messageId`, `text`, `to`)
* **sendWithEffect** : Envoyer avec un effet iMessage (`text`, `to`, `effectId`)
* **renameGroup** : Renommer une discussion de groupe (`chatGuid`, `displayName`)
* **setGroupIcon** : Définir l&#39;icône/photo d&#39;une discussion de groupe (`chatGuid`, `media`) — instable sur macOS 26 Tahoe (l&#39;API peut indiquer une réussite mais l&#39;icône ne se synchronise pas).
* **addParticipant** : Ajouter quelqu&#39;un à un groupe (`chatGuid`, `address`)
* **removeParticipant** : Retirer quelqu&#39;un d&#39;un groupe (`chatGuid`, `address`)
* **leaveGroup** : Quitter une discussion de groupe (`chatGuid`)
* **sendAttachment** : Envoyer des médias/fichiers (`to`, `buffer`, `filename`, `asVoice`)
  * Mémos vocaux : définissez `asVoice: true` avec de l&#39;audio **MP3** ou **CAF** pour l&#39;envoyer comme message vocal iMessage. BlueBubbles convertit MP3 → CAF lors de l&#39;envoi de mémos vocaux.

<div id="message-ids-short-vs-full">
  ### ID de message (courts vs complets)
</div>

OpenClaw peut exposer des ID de message *courts* (par ex. `1`, `2`) pour économiser des tokens.

* `MessageSid` / `ReplyToId` peuvent être des ID courts.
* `MessageSidFull` / `ReplyToIdFull` contiennent les ID complets du fournisseur.
* Les ID courts sont en mémoire ; ils peuvent expirer lors d’un redémarrage ou d’une éviction du cache.
* Les actions acceptent des `messageId` courts ou complets, mais les ID courts provoqueront une erreur s’ils ne sont plus disponibles.

Utilisez des ID complets pour les automatisations durables et le stockage :

* Templates : `{{MessageSidFull}}`, `{{ReplyToIdFull}}`
* Contexte : `MessageSidFull` / `ReplyToIdFull` dans les payloads entrants

Voir [Configuration](/fr/gateway/configuration) pour les variables de template.

<div id="block-streaming">
  ## Diffusion par blocs
</div>

Contrôlez si les réponses sont envoyées dans un seul message ou diffusées en continu par blocs :

```json5
{
  channels: {
    bluebubbles: {
      blockStreaming: true  // active le streaming par blocs (comportement par défaut)
    }
  }
}
```

<div id="media-limits">
  ## Médias et limites
</div>

* Les pièces jointes entrantes sont téléchargées et stockées dans le cache de médias.
* Limite de taille des médias via `channels.bluebubbles.mediaMaxMb` (par défaut : 8 Mo).
* Le texte sortant est segmenté selon `channels.bluebubbles.textChunkLimit` (par défaut : 4000 caractères).

<div id="configuration-reference">
  ## Référence de configuration
</div>

Configuration complète : [Configuration](/fr/gateway/configuration)

Options du fournisseur :

* `channels.bluebubbles.enabled` : activer/désactiver le canal.
* `channels.bluebubbles.serverUrl` : URL de base de l’API REST BlueBubbles.
* `channels.bluebubbles.password` : mot de passe de l’API.
* `channels.bluebubbles.webhookPath` : chemin du point de terminaison webhook (par défaut : `/bluebubbles-webhook`).
* `channels.bluebubbles.dmPolicy` : `pairing | allowlist | open | disabled` (par défaut : `pairing`).
* `channels.bluebubbles.allowFrom` : liste d’autorisation pour les DM (handles, e‑mails, numéros E.164, `chat_id:*`, `chat_guid:*`).
* `channels.bluebubbles.groupPolicy` : `open | allowlist | disabled` (par défaut : `allowlist`).
* `channels.bluebubbles.groupAllowFrom` : liste d’autorisation des expéditeurs de groupes.
* `channels.bluebubbles.groups` : configuration par groupe (`requireMention`, etc.).
* `channels.bluebubbles.sendReadReceipts` : envoyer les accusés de lecture (par défaut : `true`).
* `channels.bluebubbles.blockStreaming` : activer le streaming par blocs (par défaut : `true`).
* `channels.bluebubbles.textChunkLimit` : taille maximale des blocs sortants en caractères (par défaut : 4000).
* `channels.bluebubbles.chunkMode` : `length` (valeur par défaut) ne segmente qu’en cas de dépassement de `textChunkLimit` ; `newline` segmente sur les lignes vides (délimitations de paragraphe) avant le découpage par longueur.
* `channels.bluebubbles.mediaMaxMb` : limite des médias entrants en Mo (par défaut : 8).
* `channels.bluebubbles.historyLimit` : nombre maximal de messages de groupe pour le contexte (0 désactive).
* `channels.bluebubbles.dmHistoryLimit` : limite d’historique des DM.
* `channels.bluebubbles.actions` : activer/désactiver des actions spécifiques.
* `channels.bluebubbles.accounts` : configuration multi-compte.

Options globales associées :

* `agents.list[].groupChat.mentionPatterns` (ou `messages.groupChat.mentionPatterns`).
* `messages.responsePrefix`.

<div id="addressing-delivery-targets">
  ## Ciblage des adresses / cibles de livraison
</div>

Privilégiez `chat_guid` pour un routage stable :

* `chat_guid:iMessage;-;+15555550123` (préféré pour les groupes)
* `chat_id:123`
* `chat_identifier:...`
* Identifiants directs : `+15555550123`, `user@example.com`
  * Si un identifiant direct n’a pas déjà une conversation de DM existante, OpenClaw en créera une via `POST /api/v1/chat/new`. Cela nécessite que la BlueBubbles Private API soit activée.

<div id="security">
  ## Sécurité
</div>

* Les requêtes webhook sont authentifiées en comparant les paramètres de requête ou en-têtes `guid`/`password` à `channels.bluebubbles.password`. Les requêtes provenant de `localhost` sont également acceptées.
* Gardez le mot de passe de l’API et l’endpoint webhook secrets (traitez-les comme des identifiants).
* La confiance accordée à localhost signifie qu’un reverse proxy sur la même machine peut, sans le vouloir, contourner le mot de passe. Si vous placez le Gateway derrière un proxy, imposez une authentification au niveau du proxy et configurez `gateway.trustedProxies`. Voir [Sécurité du Gateway](/fr/gateway/security#reverse-proxy-configuration).
* Activez HTTPS et des règles de pare-feu sur le serveur BlueBubbles si vous l’exposez en dehors de votre réseau local (LAN).

<div id="troubleshooting">
  ## Dépannage
</div>

* Si les événements typing/read cessent de fonctionner, vérifie les journaux de webhook de BlueBubbles et confirme que le chemin du Gateway correspond à `channels.bluebubbles.webhookPath`.
* Les codes d&#39;appairage expirent au bout d&#39;une heure ; utilise `openclaw pairing list bluebubbles` et `openclaw pairing approve bluebubbles <code>`.
* Les réactions nécessitent l&#39;API privée BlueBubbles (`POST /api/v1/message/react`) ; assure-toi que la version du serveur l’expose.
* La modification/l’annulation d’envoi nécessite macOS 13+ et une version compatible du serveur BlueBubbles. Sous macOS 26 (Tahoe), la modification est actuellement défaillante en raison de changements dans l’API privée.
* Les mises à jour de l’icône de groupe peuvent être instables sous macOS 26 (Tahoe) : l’API peut retourner un succès mais la nouvelle icône ne se synchronise pas.
* OpenClaw masque automatiquement les actions connues comme défaillantes selon la version de macOS du serveur BlueBubbles. Si la modification apparaît encore sous macOS 26 (Tahoe), désactive-la manuellement avec `channels.bluebubbles.actions.edit=false`.
* Pour les informations d’état/de santé : `openclaw status --all` ou `openclaw status --deep`.

Pour une vue d’ensemble des flux de travail des canaux, consulte la page [Channels](/fr/channels) et le guide [Plugins](/fr/plugins).