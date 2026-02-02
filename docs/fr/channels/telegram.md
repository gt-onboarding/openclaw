---
title: Telegram
summary: "Statut de la prise en charge des bots Telegram, fonctionnalités et configuration"
read_when:
  - Lorsque vous travaillez sur les fonctionnalités Telegram ou les webhooks
---

<div id="telegram-bot-api">
  # Telegram (Bot API)
</div>

Statut : prêt pour la mise en production pour les messages privés (DM) avec le bot et les groupes via grammY. Long polling par défaut ; webhook en option.

<div id="quick-setup-beginner">
  ## Configuration rapide (débutant)
</div>

1. Crée un bot avec **@BotFather** et copie le jeton.
2. Configure le jeton :
   * Env : `TELEGRAM_BOT_TOKEN=...`
   * Ou config : `channels.telegram.botToken: "..."`
   * Si les deux sont définis, la config est prioritaire (la variable d’environnement ne sert de repli que pour le compte par défaut).
3. Lance le Gateway.
4. L’accès en DM utilise par défaut l’appairage ; approuve le code d’appairage lors du premier contact.

Configuration minimale :

```json5
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "123:abc",
      dmPolicy: "pairing"
    }
  }
}
```

<div id="what-it-is">
  ## Ce que c&#39;est
</div>

* Un canal Telegram Bot API géré par Gateway.
* Routage déterministe : les réponses sont renvoyées vers Telegram ; le modèle ne choisit jamais les canaux.
* Les messages privés partagent la session principale de l&#39;agent ; les groupes restent isolés (`agent:<agentId>:telegram:group:<chatId>`).

<div id="setup-fast-path">
  ## Configuration (mode rapide)
</div>

<div id="1-create-a-bot-token-botfather">
  ### 1) Créer un jeton de bot (BotFather)
</div>

1. Ouvrez Telegram et discutez avec **@BotFather**.
2. Exécutez `/newbot`, puis suivez les instructions (nom + nom d’utilisateur se terminant par `bot`).
3. Copiez le jeton et conservez-le en lieu sûr.

Paramètres optionnels de BotFather :

* `/setjoingroups` — autoriser/refuser l’ajout du bot à des groupes.
* `/setprivacy` — contrôler si le bot voit tous les messages de groupe.

<div id="2-configure-the-token-env-or-config">
  ### 2) Configurer le token (env ou config)
</div>

Exemple :

```json5
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "123:abc",
      dmPolicy: "pairing",
      groups: { "*": { requireMention: true } }
    }
  }
}
```

Option d’environnement : `TELEGRAM_BOT_TOKEN=...` (fonctionne pour le compte par défaut).
Si l’option d’environnement et la configuration sont toutes deux définies, la configuration est prioritaire.

Prise en charge de plusieurs comptes : utilisez `channels.telegram.accounts` avec des jetons par compte et un `name` facultatif. Voir [`gateway/configuration`](/fr/gateway/configuration#telegramaccounts--discordaccounts--slackaccounts--signalaccounts--imessageaccounts) pour le modèle partagé.

3. Démarrez le Gateway. Telegram démarre lorsqu’un jeton est résolu (configuration d’abord, puis repli sur l’option d’environnement).
4. L’accès en DM est, par défaut, soumis à l’appairage. Approuvez le code lorsque le bot est contacté pour la première fois.
5. Pour les groupes : ajoutez le bot, définissez le comportement de confidentialité/d’administration (ci-dessous), puis définissez `channels.telegram.groups` pour contrôler le filtrage par mention + les listes d’autorisation.

<div id="token-privacy-permissions-telegram-side">
  ## Token + confidentialité + autorisations (côté Telegram)
</div>

<div id="token-creation-botfather">
  ### Création du token (BotFather)
</div>

* `/newbot` crée le bot et renvoie le token (gardez-le secret).
* Si un token est divulgué, révoquez-le/générez-en un nouveau via @BotFather et mettez à jour votre configuration.

<div id="group-message-visibility-privacy-mode">
  ### Visibilité des messages de groupe (mode de confidentialité)
</div>

Les bots Telegram utilisent par défaut le **mode de confidentialité**, qui limite les messages de groupe qu&#39;ils reçoivent.
Si votre bot doit voir *tous* les messages de groupe, vous avez deux options :

* Désactiver le mode de confidentialité avec `/setprivacy` **ou**
* Ajouter le bot comme **administrateur** du groupe (les bots administrateurs reçoivent tous les messages).

**Remarque :** lorsque vous modifiez le mode de confidentialité, Telegram exige de retirer puis de ré‑ajouter le bot
dans chaque groupe pour que le changement prenne effet.

<div id="group-permissions-admin-rights">
  ### Autorisations de groupe (droits d’administrateur)
</div>

Le statut d’administrateur est défini dans le groupe (UI Telegram). Les bots administrateurs reçoivent toujours tous les messages du groupe, donc attribuez le statut d’administrateur si vous avez besoin d’une visibilité complète.

<div id="how-it-works-behavior">
  ## Fonctionnement (comportement)
</div>

* Les messages entrants sont normalisés dans l’enveloppe de canal partagée, avec le contexte de réponse et des espaces réservés pour les médias.
* Les réponses dans un groupe nécessitent par défaut une mention (mention @ native ou `agents.list[].groupChat.mentionPatterns` / `messages.groupChat.mentionPatterns`).
* Surcharge multi‑agent : définissez des motifs spécifiques par agent dans `agents.list[].groupChat.mentionPatterns`.
* Les réponses sont toujours renvoyées vers le même chat Telegram.
* Le long polling utilise grammY runner avec un séquencement par chat ; le parallélisme global est limité par `agents.defaults.maxConcurrent`.
* Telegram Bot API ne prend pas en charge les accusés de lecture ; il n’existe pas d’option `sendReadReceipts`.

<div id="formatting-telegram-html">
  ## Mise en forme (HTML Telegram)
</div>

* Le texte sortant vers Telegram utilise `parse_mode: "HTML"` (le sous-ensemble de balises pris en charge par Telegram).
* L’entrée de type Markdown est rendue sous forme de **HTML compatible avec Telegram** (gras/italique/barré/code/liens) ; les éléments de bloc sont aplatis en texte avec sauts de ligne/puces.
* Le HTML brut provenant des modèles est échappé pour éviter les erreurs d’analyse par Telegram.
* Si Telegram rejette le contenu HTML, OpenClaw renvoie le même message en texte brut.

<div id="commands-native-custom">
  ## Commandes (intégrées + personnalisées)
</div>

OpenClaw enregistre des commandes intégrées (comme `/status`, `/reset`, `/model`) dans le menu du bot Telegram au démarrage.
Vous pouvez ajouter des commandes personnalisées au menu via la configuration :

```json5
{
  channels: {
    telegram: {
      customCommands: [
        { command: "backup", description: "Git backup" },
        { command: "generate", description: "Create an image" }
      ]
    }
  }
}
```

<div id="troubleshooting">
  ## Dépannage
</div>

* `setMyCommands failed` dans les logs signifie généralement que le HTTPS/DNS sortant vers `api.telegram.org` est bloqué.
* Si vous voyez des erreurs `sendMessage` ou `sendChatAction`, vérifiez le routage IPv6 et le DNS.

Pour plus d’aide : [Dépannage des canaux](/fr/channels/troubleshooting).

Notes :

* Les commandes personnalisées sont **uniquement des entrées de menu** ; OpenClaw ne les exécute pas, sauf si vous les traitez ailleurs.
* Les noms de commande sont normalisés (`/` initial supprimé, conversion en minuscules) et doivent correspondre à `a-z`, `0-9`, `_` (1–32 caractères).
* Les commandes personnalisées **ne peuvent pas remplacer les commandes natives**. Les conflits sont ignorés et consignés dans les logs.
* Si `commands.native` est désactivé, seules les commandes personnalisées sont enregistrées (ou effacées s’il n’y en a aucune).

<div id="limits">
  ## Limites
</div>

* Le texte sortant est découpé en blocs selon `channels.telegram.textChunkLimit` (4000 par défaut).
* Découpage optionnel par saut de ligne : définissez `channels.telegram.chunkMode="newline"` pour découper sur les lignes vides (limites de paragraphe) avant le découpage par longueur.
* Les téléchargements et envois de médias sont plafonnés par `channels.telegram.mediaMaxMb` (5 par défaut).
* Les requêtes Telegram Bot API expirent après `channels.telegram.timeoutSeconds` (500 par défaut via grammY). Réglez une valeur plus basse pour éviter les blocages prolongés.
* Le contexte d’historique de groupe utilise `channels.telegram.historyLimit` (ou `channels.telegram.accounts.*.historyLimit`), avec repli sur `messages.groupChat.historyLimit`. Définissez `0` pour désactiver (50 par défaut).
* L’historique des messages privés (DM) peut être limité avec `channels.telegram.dmHistoryLimit` (tours côté utilisateur). Surcharges par utilisateur : `channels.telegram.dms["<user_id>"].historyLimit`.

<div id="group-activation-modes">
  ## Modes d’activation en groupe
</div>

Par défaut, le bot ne répond qu’aux mentions dans les groupes (`@botname` ou des motifs définis dans `agents.list[].groupChat.mentionPatterns`). Pour modifier ce comportement :

<div id="via-config-recommended">
  ### Par la configuration (recommandé)
</div>

```json5
{
  channels: {
    telegram: {
      groups: {
        "-1001234567890": { requireMention: false }  // toujours répondre dans ce groupe
      }
    }
  }
}
```

**Important :** le paramètre `channels.telegram.groups` crée une **liste d’autorisation** : seuls les groupes référencés (ou `"*"`) seront acceptés.
Les sujets de forum héritent de la configuration de leur groupe parent (`allowFrom`, `requireMention`, `skills`, `prompts`), sauf si vous ajoutez des remplacements spécifiques par sujet sous `channels.telegram.groups.<groupId>.topics.<topicId>`.

Pour autoriser tous les groupes avec always-respond :

```json5
{
  channels: {
    telegram: {
      groups: {
        "*": { requireMention: false }  // tous les groupes, toujours répondre
      }
    }
  }
}
```

Pour conserver le mode « mention-only » pour tous les groupes (comportement par défaut) :

```json5
{
  channels: {
    telegram: {
      groups: {
        "*": { requireMention: true }  // ou omettre complètement groups
      }
    }
  }
}
```

<div id="via-command-session-level">
  ### Par commande (au niveau de la session)
</div>

Envoie dans le groupe :

* `/activation always` - répondre à tous les messages
* `/activation mention` - nécessite une mention (par défaut)

**Remarque :** Les commandes ne mettent à jour que l’état de la session. Pour un comportement persistant entre les redémarrages, utilisez la configuration.

<div id="getting-the-group-chat-id">
  ### Récupérer l’ID de la discussion de groupe
</div>

Transférez n’importe quel message du groupe à `@userinfobot` ou `@getidsbot` sur Telegram pour voir l’ID du chat (un nombre négatif comme `-1001234567890`).

**Astuce :** Pour récupérer votre propre ID utilisateur, envoyez un message privé au bot et il répondra avec votre ID utilisateur (message d’appairage), ou utilisez `/whoami` une fois les commandes activées.

**Note de confidentialité :** `@userinfobot` est un bot tiers. Si vous préférez, ajoutez le bot au groupe, envoyez un message, puis utilisez `openclaw logs --follow` pour lire `chat.id`, ou utilisez la méthode `getUpdates` de la Bot API.

<div id="config-writes">
  ## Écritures de configuration
</div>

Par défaut, Telegram est autorisé à écrire des mises à jour de la configuration déclenchées par des événements du canal ou `/config set|unset`.

Cela se produit lorsque :

* Un groupe est converti en supergroupe et que Telegram émet `migrate_to_chat_id` (l’ID de chat change). OpenClaw peut migrer automatiquement `channels.telegram.groups`.
* Vous exécutez `/config set` ou `/config unset` dans un chat Telegram (nécessite `commands.config: true`).

Pour désactiver :

```json5
{
  channels: { telegram: { configWrites: false } }
}
```

<div id="topics-forum-supergroups">
  ## Sujets (supergroupes de forum)
</div>

Les sujets de forum Telegram incluent un `message_thread_id` par message. OpenClaw :

* Ajoute `:topic:<threadId>` à la clé de session du groupe Telegram afin que chaque sujet soit isolé.
* Envoie les indicateurs de saisie et les réponses avec `message_thread_id` pour que les réponses restent dans le sujet.
* Le sujet général (ID de fil `1`) est particulier : les envois de messages omettent `message_thread_id` (Telegram le rejette), mais les indicateurs de saisie le contiennent toujours.
* Expose `MessageThreadId` + `IsForum` dans le contexte de template pour le routage/le rendu des templates.
* Une configuration spécifique à un sujet est disponible sous `channels.telegram.groups.<chatId>.topics.<threadId>` (compétences, listes d’autorisation, réponse automatique, prompts système, désactivation).
* Les configurations de sujet héritent des paramètres du groupe (requireMention, listes d’autorisation, compétences, prompts, activation) sauf si elles sont remplacées au niveau du sujet.

Les conversations privées peuvent inclure `message_thread_id` dans certains cas particuliers. OpenClaw conserve la clé de session du DM inchangée, mais utilise tout de même l’ID de fil pour les réponses/le streaming des brouillons lorsqu’il est présent.

<div id="inline-buttons">
  ## Boutons intégrés
</div>

Telegram prend en charge les claviers intégrés (inline keyboards) avec des boutons de rappel.

```json5
{
  "channels": {
    "telegram": {
      "capabilities": {
        "inlineButtons": "allowlist"
      }
    }
  }
}
```

Pour une configuration par compte :

```json5
{
  "channels": {
    "telegram": {
      "accounts": {
        "main": {
          "capabilities": {
            "inlineButtons": "allowlist"
          }
        }
      }
    }
  }
}
```

Portées :

* `off` — boutons inline désactivés
* `dm` — uniquement les DMs (cibles de groupe bloquées)
* `group` — uniquement les groupes (cibles de DM bloquées)
* `all` — DMs + groupes
* `allowlist` — DMs + groupes, mais uniquement les expéditeurs autorisés par `allowFrom`/`groupAllowFrom` (mêmes règles que pour les commandes de contrôle)

Par défaut : `allowlist`.
Ancien comportement : `capabilities: ["inlineButtons"]` = `inlineButtons: "all"`.

<div id="sending-buttons">
  ### Envoi de boutons
</div>

Utilisez l’outil de messages avec le paramètre `buttons` :

```json5
{
  "action": "send",
  "channel": "telegram",
  "to": "123456789",
  "message": "Choose an option:",
  "buttons": [
    [
      {"text": "Yes", "callback_data": "yes"},
      {"text": "No", "callback_data": "no"}
    ],
    [
      {"text": "Cancel", "callback_data": "cancel"}
    ]
  ]
}
```

Lorsqu’un utilisateur clique sur un bouton, les données de callback sont renvoyées à l’agent sous forme de message au format :
`callback_data: value`

<div id="configuration-options">
  ### Options de configuration
</div>

Les capacités Telegram peuvent être configurées à deux niveaux (sous forme d’objet comme illustré ci-dessus ; les anciens tableaux de chaînes de caractères restent pris en charge) :

* `channels.telegram.capabilities` : configuration globale par défaut des capacités, appliquée à tous les comptes Telegram sauf si elle est surchargée.
* `channels.telegram.accounts.<account>.capabilities` : capacités propres à chaque compte, qui remplacent la configuration globale pour ce compte spécifique.

Utilisez la configuration globale lorsque tous les bots/comptes Telegram doivent se comporter de la même manière. Utilisez la configuration par compte lorsque différents bots nécessitent des comportements différents (par exemple, un compte ne gère que les DMs tandis qu’un autre est autorisé dans les groupes).

<div id="access-control-dms-groups">
  ## Contrôle des accès (DM et groupes)
</div>

<div id="dm-access">
  ### Accès aux DM
</div>

* Par défaut : `channels.telegram.dmPolicy = "pairing"`. Les expéditeurs inconnus reçoivent un code d’appairage ; les messages sont ignorés tant qu’ils n’ont pas été approuvés (les codes expirent après 1 heure).
* Approbation via :
  * `openclaw pairing list telegram`
  * `openclaw pairing approve telegram <CODE>`
* L’appairage est le mécanisme d’échange de jetons utilisé par défaut pour les DM Telegram. Détails : [Pairing](/fr/start/pairing)
* `channels.telegram.allowFrom` accepte des ID utilisateur numériques (recommandé) ou des entrées `@username`. Ce **n’est pas** le nom d’utilisateur du bot ; utilisez l’ID de l’expéditeur humain. L’assistant de configuration accepte `@username` et le convertit en ID numérique lorsque c’est possible.

<div id="finding-your-telegram-user-id">
  #### Trouver votre identifiant utilisateur Telegram
</div>

Plus sûr (sans bot tiers) :

1. Démarrez le Gateway et envoyez un DM à votre bot.
2. Exécutez `openclaw logs --follow` et recherchez `from.id`.

Autre méthode (Bot API officielle) :

1. Envoyez un DM à votre bot.
2. Récupérez les mises à jour avec le jeton de votre bot et lisez `message.from.id` :
   ```bash
   curl "https://api.telegram.org/bot<bot_token>/getUpdates"
   ```

Solution tierce (moins privée) :

* Envoyez un DM à `@userinfobot` ou `@getidsbot` et utilisez l’identifiant utilisateur retourné.

<div id="group-access">
  ### Accès aux groupes
</div>

Deux contrôles indépendants :

**1. Quels groupes sont autorisés** (liste d’autorisation de groupes via `channels.telegram.groups`) :

* Aucune configuration `groups` = tous les groupes sont autorisés
* Avec une configuration `groups` = seuls les groupes répertoriés ou `"*"` sont autorisés
* Exemple : `"groups": { "-1001234567890": {}, "*": {} }` autorise tous les groupes

**2. Quels expéditeurs sont autorisés** (filtrage des expéditeurs via `channels.telegram.groupPolicy`) :

* `"open"` = tous les expéditeurs dans les groupes autorisés peuvent envoyer des messages (paramètre permettant d’accepter sans restriction les messages de n’importe quel utilisateur)
* `"allowlist"` = seuls les expéditeurs dans `channels.telegram.groupAllowFrom` peuvent envoyer des messages
* `"disabled"` = aucun message de groupe n’est accepté\
  La valeur par défaut est `groupPolicy: "allowlist"` (bloqué tant que vous n’ajoutez pas `groupAllowFrom`).

La plupart des utilisateurs utilisent : `groupPolicy: "allowlist"` + `groupAllowFrom` + groupes spécifiques répertoriés dans `channels.telegram.groups`

<div id="long-polling-vs-webhook">
  ## Long-polling vs webhook
</div>

* Par défaut : long-polling (aucune URL publique requise).
* Mode webhook : définissez `channels.telegram.webhookUrl` (facultativement `channels.telegram.webhookSecret` + `channels.telegram.webhookPath`).
  * L&#39;écouteur local écoute sur `0.0.0.0:8787` et expose `POST /telegram-webhook` par défaut.
  * Si votre URL publique est différente, utilisez un reverse proxy et faites pointer `channels.telegram.webhookUrl` vers le point de terminaison public.

<div id="reply-threading">
  ## Fil de discussion des réponses
</div>

Telegram prend en charge les réponses facultatives en fil de discussion via des balises :

* `[[reply_to_current]]` -- répondre au message ayant déclenché l’action.
* `[[reply_to:<id>]]` -- répondre à un message spécifique par son identifiant.

Contrôlé par `channels.telegram.replyToMode` :

* `first` (valeur par défaut), `all`, `off`.

<div id="audio-messages-voice-vs-file">
  ## Messages audio (voix vs fichier)
</div>

Telegram distingue les **notes vocales** (bulle ronde) des **fichiers audio** (carte avec métadonnées).
OpenClaw utilise par défaut les fichiers audio pour des raisons de rétrocompatibilité.

Pour forcer une bulle de note vocale dans les réponses des agents, incluez cette balise n’importe où dans la réponse :

* `[[audio_as_voice]]` — envoyer l’audio comme note vocale au lieu d’un fichier.

La balise est supprimée du texte envoyé. Les autres canaux ignorent cette balise.

Pour les envois via l’outil de messagerie, définissez `asVoice: true` avec une URL `media` audio compatible voix
(`message` est facultatif lorsque `media` est présent) :

```json5
{
  "action": "send",
  "channel": "telegram",
  "to": "123456789",
  "media": "https://example.com/voice.ogg",
  "asVoice": true
}
```

<div id="stickers">
  ## Stickers
</div>

OpenClaw prend en charge la réception et l’envoi de stickers Telegram, avec un mécanisme de mise en cache intelligent.

<div id="receiving-stickers">
  ### Réception des stickers
</div>

Lorsqu’un utilisateur envoie un sticker, OpenClaw le traite en fonction de son type :

* **Stickers statiques (WEBP)** : téléchargés et analysés par la vision. Le sticker apparaît sous forme d’espace réservé `<media:sticker>` dans le contenu du message.
* **Stickers animés (TGS)** : ignorés (le format Lottie n’est pas pris en charge pour le traitement).
* **Stickers vidéo (WEBM)** : ignorés (le format vidéo n’est pas pris en charge pour le traitement).

Champ de contexte de template disponible lors de la réception de stickers :

* `Sticker` — objet avec :
  * `emoji` — emoji associé au sticker
  * `setName` — nom du pack de stickers
  * `fileId` — ID de fichier Telegram (pour renvoyer le même sticker)
  * `fileUniqueId` — ID stable pour la recherche dans le cache
  * `cachedDescription` — description générée par la vision, mise en cache lorsqu’elle est disponible

<div id="sticker-cache">
  ### Cache des stickers
</div>

Les stickers sont traités via les capacités de vision de l’IA pour générer des descriptions. Comme les mêmes stickers sont souvent renvoyés, OpenClaw met ces descriptions en cache afin d’éviter des appels API redondants.

**Fonctionnement :**

1. **Première rencontre :** L’image du sticker est envoyée à l’IA pour une analyse visuelle. L’IA génère une description (par exemple : « A cartoon cat waving enthusiastically »).
2. **Stockage en cache :** La description est enregistrée avec l’ID de fichier du sticker, son emoji et le nom de son set.
3. **Rencontres suivantes :** Lorsque le même sticker est à nouveau vu, la description mise en cache est utilisée directement. L’image n’est pas renvoyée à l’IA.

**Emplacement du cache :** `~/.openclaw/telegram/sticker-cache.json`

**Format d’une entrée de cache :**

```json
{
  "fileId": "CAACAgIAAxkBAAI...",
  "fileUniqueId": "AgADBAADb6cxG2Y",
  "emoji": "👋",
  "setName": "CoolCats",
  "description": "Un chat de dessin animé qui fait un signe de la main avec enthousiasme",
  "cachedAt": "2026-01-15T10:30:00.000Z"
}
```

**Avantages :**

* Réduit les coûts d’API en évitant les appels répétés au modèle de vision pour un même sticker
* Temps de réponse plus courts pour les stickers mis en cache (aucun délai lié au traitement de vision)
* Permet la recherche de stickers à partir des descriptions mises en cache

Le cache est alimenté automatiquement à mesure que les stickers sont reçus. Aucune gestion manuelle du cache n’est requise.

<div id="sending-stickers">
  ### Envoi de stickers
</div>

L&#39;agent peut envoyer des stickers et en rechercher à l&#39;aide des actions `sticker` et `sticker-search`. Celles-ci sont désactivées par défaut et doivent être activées dans la configuration :

```json5
{
  channels: {
    telegram: {
      actions: {
        sticker: true
      }
    }
  }
}
```

**Envoyer un sticker :**

```json5
{
  "action": "sticker",
  "channel": "telegram",
  "to": "123456789",
  "fileId": "CAACAgIAAxkBAAI..."
}
```

Paramètres :

* `fileId` (obligatoire) — l’ID de fichier Telegram de l’autocollant. Obtenez-le à partir de `Sticker.fileId` lors de la réception d’un autocollant, ou à partir d’un résultat de recherche `sticker-search`.
* `replyTo` (facultatif) — ID du message auquel répondre.
* `threadId` (facultatif) — ID du fil de discussion pour les sujets de forum.

**Rechercher des autocollants :**

L’agent peut rechercher des autocollants mis en cache par description, emoji ou nom de pack :

```json5
{
  "action": "sticker-search",
  "channel": "telegram",
  "query": "cat waving",
  "limit": 5
}
```

Renvoie les stickers correspondants à partir du cache :

```json5
{
  "ok": true,
  "count": 2,
  "stickers": [
    {
      "fileId": "CAACAgIAAxkBAAI...",
      "emoji": "👋",
      "description": "Un chat de dessin animé faisant un signe de la main avec enthousiasme",
      "setName": "CoolCats"
    }
  ]
}
```

La recherche utilise une correspondance floue sur le texte de description, les caractères emoji et les noms de sets.

**Exemple avec threading :**

```json5
{
  "action": "sticker",
  "channel": "telegram",
  "to": "-1001234567890",
  "fileId": "CAACAgIAAxkBAAI...",
  "replyTo": 42,
  "threadId": 123
}
```

<div id="streaming-drafts">
  ## Diffusion en continu (brouillons)
</div>

Telegram peut diffuser en continu des **bulles de brouillon** pendant que l&#39;agent génère une réponse.
OpenClaw utilise l&#39;API Bot `sendMessageDraft` (pas de vrais messages), puis envoie
la réponse finale comme un message normal.

Prérequis (Telegram Bot API 9.3+) :

* **Chats privés avec sujets activés** (mode sujet de forum pour le bot).
* Les messages entrants doivent inclure `message_thread_id` (fil de sujet privé).
* La diffusion en continu est ignorée pour les groupes/supergroupes/canaux.

Configuration :

* `channels.telegram.streamMode: "off" | "partial" | "block"` (par défaut : `partial`)
  * `partial` : met à jour la bulle de brouillon avec le dernier texte diffusé.
  * `block` : met à jour la bulle de brouillon par blocs plus larges (par paquets).
  * `off` : désactive la diffusion de brouillons.
* Optionnel (uniquement pour `streamMode: "block"`) :
  * `channels.telegram.draftChunk: { minChars?, maxChars?, breakPreference? }`
    * valeurs par défaut : `minChars: 200`, `maxChars: 800`, `breakPreference: "paragraph"` (limité par `channels.telegram.textChunkLimit`).

Remarque : la diffusion de brouillons est distincte de la **diffusion par blocs** (messages de canal).
La diffusion par blocs est désactivée par défaut et nécessite `channels.telegram.blockStreaming: true`
si vous voulez des messages Telegram envoyés plus tôt au lieu de mises à jour de brouillons.

Flux de raisonnement (Telegram uniquement) :

* `/reasoning stream` diffuse le raisonnement dans la bulle de brouillon pendant que la réponse est
  générée, puis envoie la réponse finale sans le raisonnement.
* Si `channels.telegram.streamMode` est `off`, le flux de raisonnement est désactivé.
  Plus de contexte : [Streaming + chunking](/fr/concepts/streaming).

<div id="retry-policy">
  ## Stratégie de nouvelle tentative
</div>

Les appels sortants à l&#39;API Telegram sont réessayés en cas d&#39;erreurs réseau transitoires/429 avec backoff exponentiel et jitter. Configurez cela via `channels.telegram.retry`. Consultez [Stratégie de nouvelle tentative](/fr/concepts/retry).

<div id="agent-tool-messages-reactions">
  ## Outil d’Agent (messages + réactions)
</div>

* Outil : `telegram` avec l’action `sendMessage` (`to`, `content`, `mediaUrl` facultatif, `replyToMessageId`, `messageThreadId`).
* Outil : `telegram` avec l’action `react` (`chatId`, `messageId`, `emoji`).
* Outil : `telegram` avec l’action `deleteMessage` (`chatId`, `messageId`).
* Sémantique de suppression des réactions : voir [/tools/reactions](/fr/tools/reactions).
* Activation des outils : `channels.telegram.actions.reactions`, `channels.telegram.actions.sendMessage`, `channels.telegram.actions.deleteMessage` (par défaut : activé), et `channels.telegram.actions.sticker` (par défaut : désactivé).

<div id="reaction-notifications">
  ## Notifications de réactions
</div>

**Fonctionnement des réactions :**
Les réactions Telegram arrivent sous forme de **événements `message_reaction` distincts**, et non comme des propriétés dans les payloads de message. Lorsqu’un utilisateur ajoute une réaction, OpenClaw :

1. Reçoit la mise à jour `message_reaction` depuis l’API Telegram
2. La convertit en **événement système** au format : `"Telegram reaction added: {emoji} by {user} on msg {id}"`
3. Met en file d’attente l’événement système en utilisant **la même clé de session** que les messages normaux
4. Lorsque le prochain message arrive dans cette conversation, les événements système sont vidés de la file et ajoutés en tête du contexte de l’agent

L’agent voit les réactions comme des **notifications système** dans l’historique de conversation, et non comme des métadonnées de message.

**Configuration :**

* `channels.telegram.reactionNotifications` : contrôle quelles réactions déclenchent des notifications
  * `"off"` — ignorer toutes les réactions
  * `"own"` — notifier lorsque des utilisateurs réagissent aux messages du bot (best-effort en mémoire) (valeur par défaut)
  * `"all"` — notifier pour toutes les réactions

* `channels.telegram.reactionLevel` : contrôle la capacité de réaction de l’agent
  * `"off"` — l’agent ne peut pas réagir aux messages
  * `"ack"` — le bot envoie des réactions d’accusé de réception (👀 pendant le traitement) (valeur par défaut)
  * `"minimal"` — l’agent peut réagir avec parcimonie (recommandation : 1 toutes les 5 à 10 échanges)
  * `"extensive"` — l’agent peut réagir librement lorsque c’est approprié

**Groupes de forum :** Les réactions dans les groupes de forum incluent `message_thread_id` et utilisent des clés de session de la forme `agent:main:telegram:group:{chatId}:topic:{threadId}`. Cela garantit que les réactions et les messages d’un même sujet restent regroupés.

**Exemple de configuration :**

```json5
{
  channels: {
    telegram: {
      reactionNotifications: "all",  // See all reactions
      reactionLevel: "minimal"        // L'agent peut réagir avec modération
    }
  }
}
```

**Exigences :**

* Les bots Telegram doivent demander explicitement `message_reaction` dans `allowed_updates` (configuré automatiquement par OpenClaw)
* En mode webhook, les réactions sont incluses dans le champ `allowed_updates` du webhook
* En mode polling, les réactions sont incluses dans le champ `allowed_updates` de `getUpdates`

<div id="delivery-targets-clicron">
  ## Cibles d’envoi (CLI/cron)
</div>

* Utilisez un ID de chat (`123456789`) ou un nom d’utilisateur (`@name`) comme cible.
* Exemple : `openclaw message send --channel telegram --target 123456789 --message "hi"`.

<div id="troubleshooting">
  ## Dépannage
</div>

**Le bot ne répond pas aux messages sans mention dans un groupe :**

* Si vous avez défini `channels.telegram.groups.*.requireMention=false`, le **mode de confidentialité** de l’API Bot de Telegram doit être désactivé.
  * BotFather : `/setprivacy` → **Disable** (puis retirez et ré‑ajoutez le bot au groupe)
* `openclaw channels status` affiche un avertissement lorsque la configuration s’attend à recevoir des messages de groupe sans mention.
* `openclaw channels status --probe` peut en plus vérifier l’appartenance pour des IDs de groupe numériques explicites (il ne peut pas auditer les règles génériques `"*"`).
* Test rapide : `/activation always` (valable uniquement pour la session ; utilisez la configuration pour rendre le comportement persistant)

**Le bot ne voit pas du tout les messages de groupe :**

* Si `channels.telegram.groups` est défini, le groupe doit y être listé ou utiliser `"*"`
* Vérifiez les paramètres de confidentialité dans @BotFather → &quot;Group Privacy&quot; doit être **OFF**
* Vérifiez que le bot est bien membre du groupe (et pas seulement admin sans accès en lecture)
* Vérifiez les journaux du Gateway : `openclaw logs --follow` (recherchez &quot;skipping group message&quot;)

**Le bot répond aux mentions mais pas à `/activation always` :**

* La commande `/activation` met à jour l’état de la session mais ne le persiste pas dans la configuration
* Pour un comportement persistant, ajoutez le groupe à `channels.telegram.groups` avec `requireMention: false`

**Des commandes comme `/status` ne fonctionnent pas :**

* Assurez‑vous que votre ID utilisateur Telegram est autorisé (via appairage ou `channels.telegram.allowFrom`)
* Les commandes nécessitent une autorisation même dans les groupes avec `groupPolicy: "open"`

**Le long-polling s’arrête immédiatement sur Node 22+ (souvent avec des proxys/un fetch personnalisé) :**

* Node 22+ est plus strict concernant les instances d’`AbortSignal` ; des signaux externes peuvent interrompre les appels `fetch` immédiatement.
* Mettez à niveau vers une version d’OpenClaw qui normalise les signaux d’annulation, ou exécutez le Gateway sur Node 20 en attendant de pouvoir mettre à niveau.

**Le bot démarre puis cesse silencieusement de répondre (ou les journaux affichent `HttpError: Network request ... failed`) :**

* Certains hôtes résolvent `api.telegram.org` d’abord en IPv6. Si votre serveur n’a pas de trafic sortant IPv6 fonctionnel, grammY peut rester bloqué sur des requêtes uniquement IPv6.
* Corrigez en activant la connectivité IPv6 sortante **ou** en forçant la résolution IPv4 pour `api.telegram.org` (par exemple, en ajoutant une entrée `/etc/hosts` utilisant l’enregistrement A IPv4, ou en préférant IPv4 dans votre pile DNS système), puis redémarrez le Gateway.
* Vérification rapide : `dig +short api.telegram.org A` et `dig +short api.telegram.org AAAA` pour confirmer ce que renvoie le DNS.

<div id="configuration-reference-telegram">
  ## Référence de configuration (Telegram)
</div>

Configuration complète : [Configuration](/fr/gateway/configuration)

Options du fournisseur :

* `channels.telegram.enabled` : activer/désactiver le démarrage du canal.
* `channels.telegram.botToken` : jeton de bot (BotFather).
* `channels.telegram.tokenFile` : lire le jeton à partir d’un chemin de fichier.
* `channels.telegram.dmPolicy` : `pairing | allowlist | open | disabled` (valeur par défaut : pairing).
* `channels.telegram.allowFrom` : liste d’autorisation de DM (ID/noms d’utilisateur). `open` nécessite `"*"`.
* `channels.telegram.groupPolicy` : `open | allowlist | disabled` (valeur par défaut : allowlist).
* `channels.telegram.groupAllowFrom` : liste d’autorisation des expéditeurs de groupes (ID/noms d’utilisateur).
* `channels.telegram.groups` : valeurs par défaut par groupe + liste d’autorisation (utilisez `"*"` pour des valeurs par défaut globales).
  * `channels.telegram.groups.<id>.requireMention` : valeur par défaut de filtrage par mention.
  * `channels.telegram.groups.<id>.skills` : filtre de compétences (omis = toutes les compétences, vide = aucune).
  * `channels.telegram.groups.<id>.allowFrom` : surcharge de la liste d’autorisation des expéditeurs pour ce groupe.
  * `channels.telegram.groups.<id>.systemPrompt` : invite système supplémentaire pour le groupe.
  * `channels.telegram.groups.<id>.enabled` : désactiver le groupe lorsque `false`.
  * `channels.telegram.groups.<id>.topics.<threadId>.*` : surcharges par sujet (mêmes champs que le groupe).
  * `channels.telegram.groups.<id>.topics.<threadId>.requireMention` : surcharge du filtrage par mention pour ce sujet.
* `channels.telegram.capabilities.inlineButtons` : `off | dm | group | all | allowlist` (valeur par défaut : allowlist).
* `channels.telegram.accounts.<account>.capabilities.inlineButtons` : surcharge par compte.
* `channels.telegram.replyToMode` : `off | first | all` (valeur par défaut : `first`).
* `channels.telegram.textChunkLimit` : taille des blocs sortants (caractères).
* `channels.telegram.chunkMode` : `length` (par défaut) ou `newline` pour couper sur les lignes vides (frontières de paragraphe) avant le découpage par longueur.
* `channels.telegram.linkPreview` : activer/désactiver l’aperçu des liens pour les messages sortants (valeur par défaut : true).
* `channels.telegram.streamMode` : `off | partial | block` (diffusion de réponses brouillon).
* `channels.telegram.mediaMaxMb` : limite de taille des médias entrants/sortants (Mo).
* `channels.telegram.retry` : stratégie de nouvelle tentative pour les appels à l’API Telegram sortants (attempts, minDelayMs, maxDelayMs, jitter).
* `channels.telegram.network.autoSelectFamily` : remplacer la valeur Node autoSelectFamily (true=activer, false=désactiver). Par défaut désactivé sur Node 22 pour éviter les timeouts Happy Eyeballs.
* `channels.telegram.proxy` : URL du proxy pour les appels Bot API (SOCKS/HTTP).
* `channels.telegram.webhookUrl` : activer le mode webhook.
* `channels.telegram.webhookSecret` : secret webhook (optionnel).
* `channels.telegram.webhookPath` : chemin webhook local (par défaut `/telegram-webhook`).
* `channels.telegram.actions.reactions` : filtrer les réactions d’outils Telegram.
* `channels.telegram.actions.sendMessage` : filtrer l’envoi de messages des outils Telegram.
* `channels.telegram.actions.deleteMessage` : filtrer la suppression de messages des outils Telegram.
* `channels.telegram.actions.sticker` : filtrer les actions de stickers Telegram — envoi et recherche (valeur par défaut : false).
* `channels.telegram.reactionNotifications` : `off | own | all` — détermine quelles réactions déclenchent des événements système (valeur par défaut : `own` lorsqu’elle n’est pas définie).
* `channels.telegram.reactionLevel` : `off | ack | minimal | extensive` — détermine la capacité de réaction de l’agent (valeur par défaut : `minimal` lorsqu’elle n’est pas définie).

Options globales associées :

* `agents.list[].groupChat.mentionPatterns` (motifs de filtrage par mention).
* `messages.groupChat.mentionPatterns` (solution de repli globale).
* `commands.native` (par défaut `"auto"` → activé pour Telegram/Discord, désactivé pour Slack), `commands.text`, `commands.useAccessGroups` (comportement des commandes). Remplacer par `channels.telegram.commands.native`.
* `messages.responsePrefix`, `messages.ackReaction`, `messages.ackReactionScope`, `messages.removeAckAfterReply`.