---
title: Discord
summary: "Statut de prise en charge du bot Discord, capacités et configuration"
read_when:
  - Travailler sur les fonctionnalités du canal Discord
---

<div id="discord-bot-api">
  # Discord (Bot API)
</div>

Statut : prêt pour les messages privés (DM) et les salons textuels de serveurs via le Gateway officiel des bots Discord.

<div id="quick-setup-beginner">
  ## Configuration rapide (débutant)
</div>

1. Créez un bot Discord et copiez le jeton du bot.
2. Dans les paramètres de l’application Discord, activez **Message Content Intent** (et **Server Members Intent** si vous prévoyez d’utiliser des listes d’autorisation ou des recherches par nom).
3. Configurez le jeton pour OpenClaw :
   * Env : `DISCORD_BOT_TOKEN=...`
   * Ou config : `channels.discord.token: "..."`
   * Si les deux sont définis, la configuration a la priorité (la variable d’environnement ne sert de solution de repli que pour le compte par défaut).
4. Invitez le bot sur votre serveur avec les autorisations d’envoi de messages (créez un serveur privé si vous ne voulez que des DMs).
5. Démarrez le Gateway.
6. L’accès en DM utilise l’appairage par défaut ; approuvez le code d’appairage lors du premier contact.

Configuration minimale :

```json5
{
  channels: {
    discord: {
      enabled: true,
      token: "VOTRE_JETON_BOT"
    }
  }
}
```

<div id="goals">
  ## Objectifs
</div>

* Parler avec OpenClaw via des DMs Discord ou des salons de serveur (guild).
* Les conversations directes sont regroupées dans la session principale de l’agent (par défaut `agent:main:main`) ; les salons de serveur restent isolés en `agent:<agentId>:discord:channel:<channelId>` (les noms affichés utilisent `discord:<guildSlug>#<channelSlug>`).
* Les DMs de groupe sont ignorés par défaut ; active-les via `channels.discord.dm.groupEnabled` et, éventuellement, restreins-les avec `channels.discord.dm.groupChannels`.
* Assurer un routage déterministe : les réponses sont toujours renvoyées vers le salon par lequel elles sont arrivées.

<div id="how-it-works">
  ## Fonctionnement
</div>

1. Créez une application Discord → Bot, activez les intents dont vous avez besoin (MP/DM + messages de serveur (guild) + contenu des messages), puis récupérez le jeton du bot.
2. Invitez le bot sur votre serveur avec les permissions nécessaires pour lire/envoyer des messages là où vous souhaitez l’utiliser.
3. Configurez OpenClaw avec `channels.discord.token` (ou `DISCORD_BOT_TOKEN` en secours).
4. Démarrez le Gateway ; il lance automatiquement le canal Discord lorsqu’un jeton est disponible (config en premier, variable d’environnement en secours) et que `channels.discord.enabled` n’est pas `false`.
   * Si vous préférez les variables d’environnement, définissez `DISCORD_BOT_TOKEN` (un bloc de configuration est facultatif).
5. Conversations directes : utilisez `user:<id>` (ou une mention `<@id>`) pour l’envoi ; tous les échanges arrivent dans la session partagée `main`. Les IDs purement numériques sont ambigus et rejetés.
6. Canaux de serveur (guild) : utilisez `channel:<channelId>` pour l’envoi. Les mentions sont exigées par défaut et peuvent être configurées par serveur (guild) ou par canal.
7. Conversations directes : sécurisées par défaut via `channels.discord.dm.policy` (valeur par défaut : `"pairing"`). Les expéditeurs inconnus reçoivent un code d’appairage (qui expire au bout d’une heure) ; approuvez via `openclaw pairing approve discord <code>`.
   * Pour conserver l’ancien comportement « ouvert à tout le monde » : définissez `channels.discord.dm.policy="open"` (le paramètre `open` autorise l’acceptation de messages sans restriction depuis n’importe quel utilisateur) et `channels.discord.dm.allowFrom=["*"]`.
   * Pour une liste d’autorisation stricte : définissez `channels.discord.dm.policy="allowlist"` et référencez les expéditeurs dans `channels.discord.dm.allowFrom`.
   * Pour ignorer tous les MP : définissez `channels.discord.dm.enabled=false` ou `channels.discord.dm.policy="disabled"`.
8. Les MP de groupe sont ignorés par défaut ; activez-les via `channels.discord.dm.groupEnabled` et restreignez-les éventuellement avec `channels.discord.dm.groupChannels`.
9. Règles optionnelles par serveur (guild) : définissez `channels.discord.guilds`, indexé par id de serveur (recommandé) ou par slug, avec des règles par canal.
10. Commandes natives optionnelles : `commands.native` vaut par défaut `"auto"` (activé pour Discord/Telegram, désactivé pour Slack). Remplacez avec `channels.discord.commands.native: true|false|"auto"` ; `false` efface les commandes précédemment enregistrées. Les commandes texte sont contrôlées par `commands.text` et doivent être envoyées sous forme de messages `/...` autonomes. Utilisez `commands.useAccessGroups: false` pour contourner les vérifications des groupes d’accès pour les commandes.
    * Liste complète des commandes + configuration : [Slash commands](/fr/tools/slash-commands)
11. Historique de contexte de serveur (guild) optionnel : définissez `channels.discord.historyLimit` (par défaut 20, avec repli vers `messages.groupChat.historyLimit`) pour inclure les N derniers messages du serveur comme contexte lors de la réponse à une mention. Mettez `0` pour désactiver.
12. Réactions : l’agent peut déclencher des réactions via l’outil `discord` (contrôlé par `channels.discord.actions.*`).
    * Sémantique de suppression de réaction : voir [/tools/reactions](/fr/tools/reactions).
    * L’outil `discord` n’est exposé que lorsque le canal courant est Discord.
13. Les commandes natives utilisent des clés de session isolées (`agent:<agentId>:discord:slash:<userId>`) plutôt que la session partagée `main`.

Note : La résolution nom → id utilise la recherche de membres de serveur (guild) et nécessite l’intent « Server Members » ; si le bot ne peut pas rechercher de membres, utilisez des ids ou des mentions `<@id>`.
Note : Les slugs sont en minuscules, avec les espaces remplacés par `-`. Les noms de canaux sont « slugifiés » sans le `#` initial.
Note : Les lignes de contexte de serveur (guild) `[from:]` incluent `author.tag` + `id` pour faciliter les réponses prêtes à ping.

<div id="config-writes">
  ## Écritures de config
</div>

Par défaut, Discord est autorisé à appliquer des mises à jour de configuration déclenchées par `/config set|unset` (nécessite `commands.config: true`).

Désactivez avec :

```json5
{
  channels: { discord: { configWrites: false } }
}
```

<div id="how-to-create-your-own-bot">
  ## Comment créer votre propre bot
</div>

Voici la configuration à réaliser dans le « Discord Developer Portal » pour faire tourner OpenClaw dans un salon de serveur (guild) tel que `#help`.

<div id="1-create-the-discord-app-bot-user">
  ### 1) Créer l’application Discord et l’utilisateur bot
</div>

1. Portail développeur Discord → **Applications** → **New Application**
2. Dans votre application :
   * **Bot** → **Add Bot**
   * Copiez le **Bot Token** (c’est la valeur à renseigner dans `DISCORD_BOT_TOKEN`)

<div id="2-enable-the-gateway-intents-openclaw-needs">
  ### 2) Activer les intents du Gateway dont OpenClaw a besoin
</div>

Discord bloque les « intents privilégiés » tant que vous ne les avez pas explicitement activés.

Dans **Bot** → **Privileged Gateway Intents**, activez :

* **Message Content Intent** (requis pour lire le texte des messages dans la plupart des serveurs ; sans cela, vous verrez « Used disallowed intents » ou le bot se connectera mais ne réagira pas aux messages)
* **Server Members Intent** (recommandé ; requis pour certaines recherches de membres/utilisateurs et pour faire correspondre les membres à la liste d’autorisation dans les serveurs)

Vous n’avez généralement **pas** besoin de **Presence Intent**.

<div id="3-generate-an-invite-url-oauth2-url-generator">
  ### 3) Générer une URL d’invitation (générateur d’URL OAuth2)
</div>

Dans votre application : **OAuth2** → **URL Generator**

**Scopes**

* ✅ `bot`
* ✅ `applications.commands` (requis pour les commandes natives)

**Permissions du bot** (base minimale)

* ✅ Voir les salons
* ✅ Envoyer des messages
* ✅ Lire l’historique des messages
* ✅ Intégrer des liens
* ✅ Joindre des fichiers
* ✅ Ajouter des réactions (facultatif mais recommandé)
* ✅ Utiliser des émojis/autocollants externes (facultatif ; uniquement si vous souhaitez les utiliser)

Évitez **Administrator**, sauf si vous êtes en train de déboguer et faites entièrement confiance au bot.

Copiez l’URL générée, ouvrez-la, choisissez votre serveur et installez le bot.

<div id="4-get-the-ids-guilduserchannel">
  ### 4) Récupérer les ID (guild/user/channel)
</div>

Discord utilise des ID numériques partout ; la configuration d’OpenClaw préfère utiliser les ID.

1. Discord (desktop/web) → **User Settings** → **Advanced** → activer **Developer Mode**
2. Cliquez avec le bouton droit :
   * Nom du serveur → **Copy Server ID** (guild id)
   * Canal (par ex. `#help`) → **Copy Channel ID**
   * Votre compte → **Copy User ID**

<div id="5-configure-openclaw">
  ### 5) Configurer OpenClaw
</div>

<div id="token">
  #### Jeton
</div>

Configurez le jeton du bot via une variable d&#39;environnement (recommandé sur les serveurs) :

* `DISCORD_BOT_TOKEN=...`

Ou via la configuration :

```json5
{
  channels: {
    discord: {
      enabled: true,
      token: "VOTRE_JETON_BOT"
    }
  }
}
```

Prise en charge de plusieurs comptes : utilisez `channels.discord.accounts` avec des jetons par compte et un `name` facultatif. Voir [`gateway/configuration`](/fr/gateway/configuration#telegramaccounts--discordaccounts--slackaccounts--signalaccounts--imessageaccounts) pour le schéma commun.

<div id="allowlist-channel-routing">
  #### Liste d’autorisation + routage des canaux
</div>

Exemple « serveur unique, moi seul autorisé, seul #help autorisé » :

```json5
{
  channels: {
    discord: {
      enabled: true,
      dm: { enabled: false },
      guilds: {
        "YOUR_GUILD_ID": {
          users: ["YOUR_USER_ID"],
          requireMention: true,
          channels: {
            help: { allow: true, requireMention: true }
          }
        }
      },
      retry: {
        attempts: 3,
        minDelayMs: 500,
        maxDelayMs: 30000,
        jitter: 0.1
      }
    }
  }
}
```

Notes :

* `requireMention: true` signifie que le bot ne répond que lorsqu’il est mentionné (recommandé pour les canaux partagés).
* `agents.list[].groupChat.mentionPatterns` (ou `messages.groupChat.mentionPatterns`) comptent également comme des mentions pour les messages de serveur (guild).
* Remplacement multi-agent : définissez des motifs spécifiques à chaque agent sur `agents.list[].groupChat.mentionPatterns`.
* Si `channels` est présent, tout canal non répertorié est refusé par défaut.
* Utilisez une entrée de canal `"*"` pour appliquer les valeurs par défaut à tous les canaux ; des entrées de canal explicites remplacent ce caractère générique.
* Les fils de discussion héritent de la configuration du canal parent (liste d’autorisation, `requireMention`, compétences, prompts, etc.) sauf si vous ajoutez explicitement l’identifiant du canal du fil.
* Les messages émis par des bots sont ignorés par défaut ; définissez `channels.discord.allowBots=true` pour les autoriser (vos propres messages restent filtrés).
* Avertissement : si vous autorisez les réponses à d’autres bots (`channels.discord.allowBots=true`), empêchez les boucles de réponses entre bots avec `requireMention`, les listes d’autorisation `channels.discord.guilds.*.channels.<id>.users` et/ou des garde-fous explicites dans `AGENTS.md` et `SOUL.md`.

<div id="6-verify-it-works">
  ### 6) Vérifiez que tout fonctionne
</div>

1. Démarrez le Gateway.
2. Dans le salon de votre serveur, envoyez : `@Krill hello` (ou quel que soit le nom de votre bot).
3. Si rien ne se passe, consultez la section **Dépannage** ci-dessous.

<div id="troubleshooting">
  ### Dépannage
</div>

* D’abord, exécute `openclaw doctor` et `openclaw channels status --probe` (avertissements exploitables + audits rapides).
* **« Used disallowed intents »** : active **Message Content Intent** (et probablement **Server Members Intent**) dans le Developer Portal, puis redémarre le Gateway.
* **Le bot se connecte mais ne répond jamais dans un salon de guilde** :
  * **Message Content Intent** manquant, ou
  * Le bot n’a pas les autorisations dans le salon (View/Send/Read History), ou
  * Ta config exige des mentions et tu ne l’as pas mentionné, ou
  * La liste d’autorisation de ta guilde ou de ton salon refuse le salon/l’utilisateur.
* **`requireMention: false` mais toujours aucune réponse** :
* `channels.discord.groupPolicy` vaut par défaut **allowlist** ; définis-le sur `"open"` (permet l’acceptation de messages sans restriction depuis n’importe quel utilisateur) ou ajoute une entrée de guilde sous `channels.discord.guilds` (liste éventuellement des salons sous `channels.discord.guilds.<id>.channels` pour restreindre).
  * Si tu ne définis que `DISCORD_BOT_TOKEN` et ne crées jamais de section `channels.discord`, le runtime
    définit `groupPolicy` par défaut à `open` (autorise l’acceptation de messages sans restriction depuis n’importe quel utilisateur). Ajoute `channels.discord.groupPolicy`,
    `channels.defaults.groupPolicy` ou une liste d’autorisation de guilde/salon pour le verrouiller.
* `requireMention` doit se trouver sous `channels.discord.guilds` (ou un salon spécifique). `channels.discord.requireMention` au niveau supérieur est ignoré.
* Les **vérifications des autorisations** (`channels status --probe`) ne contrôlent que les ID numériques de salon. Si tu utilises des slugs/noms comme clés `channels.discord.guilds.*.channels`, l’audit ne peut pas vérifier les autorisations.
* **Les DM ne fonctionnent pas** : `channels.discord.dm.enabled=false`, `channels.discord.dm.policy="disabled"`, ou tu n’as pas encore été approuvé (`channels.discord.dm.policy="pairing"`).

<div id="capabilities-limits">
  ## Capacités et limites
</div>

* Messages privés (DM) et salons texte de serveurs (les fils sont traités comme des salons distincts ; la voix n’est pas prise en charge).
* Indicateurs de saisie envoyés en mode « best effort » ; le découpage des messages utilise `channels.discord.textChunkLimit` (2000 par défaut) et scinde les réponses très longues par nombre de lignes (`channels.discord.maxLinesPerMessage`, 17 par défaut).
* Découpage optionnel par saut de ligne : définissez `channels.discord.chunkMode="newline"` pour découper sur les lignes vides (frontières de paragraphes) avant le découpage par longueur.
* Téléversement de fichiers pris en charge jusqu’à la valeur configurée de `channels.discord.mediaMaxMb` (8 Mo par défaut).
* Réponses dans les serveurs limitées aux mentions par défaut afin d’éviter les bots bruyants.
* Contexte de réponse injecté lorsqu’un message fait référence à un autre message (contenu cité + identifiants).
* Les fils de réponses natifs sont **désactivés par défaut** ; activez-les avec `channels.discord.replyToMode` et les tags de réponse.

<div id="retry-policy">
  ## Stratégie de réessai
</div>

Les appels sortants à l’API Discord sont retentés en cas de limitation de débit (429), en utilisant le `retry_after` de Discord lorsqu’il est disponible, avec backoff exponentiel et jitter. Configurez via `channels.discord.retry`. Voir [Stratégie de réessai](/fr/concepts/retry).

<div id="config">
  ## Configuration
</div>

```json5
{
  channels: {
    discord: {
      enabled: true,
      token: "abc.123",
      groupPolicy: "allowlist",
      guilds: {
        "*": {
          channels: {
            general: { allow: true }
          }
        }
      },
      mediaMaxMb: 8,
      actions: {
        reactions: true,
        stickers: true,
        emojiUploads: true,
        stickerUploads: true,
        polls: true,
        permissions: true,
        messages: true,
        threads: true,
        pins: true,
        search: true,
        memberInfo: true,
        roleInfo: true,
        roles: false,
        channelInfo: true,
        channels: true,
        voiceStatus: true,
        events: true,
        moderation: false
      },
      replyToMode: "off",
      dm: {
        enabled: true,
        policy: "pairing", // appairage | liste d'autorisation | open | désactivé
        allowFrom: ["123456789012345678", "steipete"],
        groupEnabled: false,
        groupChannels: ["openclaw-dm"]
      },
      guilds: {
        "*": { requireMention: true },
        "123456789012345678": {
          slug: "friends-of-openclaw",
          requireMention: false,
          reactionNotifications: "own",
          users: ["987654321098765432", "steipete"],
          channels: {
            general: { allow: true },
            help: {
              allow: true,
              requireMention: true,
              users: ["987654321098765432"],
              skills: ["search", "docs"],
              systemPrompt: "Keep answers short."
            }
          }
        }
      }
    }
  }
}
```

Les réactions d&#39;accusé de réception (ack) sont contrôlées globalement par `messages.ackReaction` +
`messages.ackReactionScope`. Utilisez `messages.removeAckAfterReply` pour supprimer la
réaction d&#39;accusé de réception après la réponse du bot.

* `dm.enabled` : mettre `false` pour ignorer tous les DMs (par défaut `true`).
* `dm.policy` : contrôle d’accès aux DMs (`pairing` recommandé). `"open"` requiert `dm.allowFrom=["*"]`.
* `dm.allowFrom` : liste d’autorisation de DM (ids ou noms d’utilisateurs). Utilisée par `dm.policy="allowlist"` et pour la validation de `dm.policy="open"`. L’assistant interactif accepte les noms d’utilisateur et les convertit en ids lorsque le bot peut rechercher des membres.
* `dm.groupEnabled` : activer les DMs de groupe (par défaut `false`).
* `dm.groupChannels` : liste d’autorisation facultative pour les ids ou slugs de canaux de DM de groupe.
* `groupPolicy` : contrôle la gestion des canaux de serveur (`open|disabled|allowlist`) ; `allowlist` requiert des listes d’autorisation de canaux.
* `guilds` : règles par serveur indexées par id de serveur (préféré) ou slug.
* `guilds."*"` : paramètres par serveur par défaut appliqués lorsqu’aucune entrée explicite n’existe.
* `guilds.<id>.slug` : slug convivial facultatif utilisé pour les noms affichés.
* `guilds.<id>.users` : liste d’autorisation facultative par serveur pour les utilisateurs (ids ou noms).
* `guilds.<id>.tools` : surcharges facultatives de la politique d’outils par serveur (`allow`/`deny`/`alsoAllow`) utilisées lorsque la surcharge au niveau du canal est absente.
* `guilds.<id>.toolsBySender` : surcharges facultatives de la politique d’outils par émetteur au niveau du serveur (s’appliquent lorsque la surcharge au niveau du canal est absente ; joker `"*"` pris en charge).
* `guilds.<id>.channels.<channel>.allow` : autoriser/refuser le canal lorsque `groupPolicy="allowlist"`.
* `guilds.<id>.channels.<channel>.requireMention` : filtrage par mention pour le canal.
* `guilds.<id>.channels.<channel>.tools` : surcharges facultatives de la politique d’outils par canal (`allow`/`deny`/`alsoAllow`).
* `guilds.<id>.channels.<channel>.toolsBySender` : surcharges facultatives de la politique d’outils par émetteur dans le canal (joker `"*"` pris en charge).
* `guilds.<id>.channels.<channel>.users` : liste d’autorisation facultative par canal pour les utilisateurs.
* `guilds.<id>.channels.<channel>.skills` : filtre de compétences (omis = toutes les compétences, vide = aucune).
* `guilds.<id>.channels.<channel>.systemPrompt` : invite système supplémentaire pour le canal (combinée avec le sujet du canal).
* `guilds.<id>.channels.<channel>.enabled` : mettre `false` pour désactiver le canal.
* `guilds.<id>.channels` : règles de canal (les clés sont des slugs ou ids de canal).
* `guilds.<id>.requireMention` : exigence de mention par serveur (surchargeable par canal).
* `guilds.<id>.reactionNotifications` : mode d’événement système pour les réactions (`off`, `own`, `all`, `allowlist`).
* `textChunkLimit` : taille des blocs de texte sortants (caractères). Par défaut : 2000.
* `chunkMode` : `length` (par défaut) segmente uniquement lorsqu’on dépasse `textChunkLimit` ; `newline` segmente sur les lignes vides (limites de paragraphe) avant le découpage par longueur.
* `maxLinesPerMessage` : limite souple du nombre de lignes par message. Par défaut : 17.
* `mediaMaxMb` : limite la taille des médias entrants enregistrés sur disque.
* `historyLimit` : nombre de messages récents du serveur à inclure comme contexte lors de la réponse à une mention (par défaut 20 ; repli sur `messages.groupChat.historyLimit` ; `0` désactive).
* `dmHistoryLimit` : limite d’historique des DMs en tours utilisateur. Surcharges par utilisateur : `dms["<user_id>"].historyLimit`.
* `retry` : politique de nouvelle tentative pour les appels sortants à l’API Discord (attempts, minDelayMs, maxDelayMs, jitter).
* `actions` : garde-fous d’outils par action ; omettre pour tout autoriser (mettre `false` pour désactiver).
  * `reactions` (couvre les réactions + la lecture des réactions)
  * `stickers`, `emojiUploads`, `stickerUploads`, `polls`, `permissions`, `messages`, `threads`, `pins`, `search`
  * `memberInfo`, `roleInfo`, `channelInfo`, `voiceStatus`, `events`
  * `channels` (création/édition/suppression de canaux + catégories + permissions)
  * `roles` (ajout/suppression de rôles, par défaut `false`)
  * `moderation` (timeout/kick/ban, par défaut `false`)

Les notifications de réactions utilisent `guilds.<id>.reactionNotifications` :

* `off` : aucun événement de réaction.
* `own` : réactions sur les messages du bot lui‑même (valeur par défaut).
* `all` : toutes les réactions sur tous les messages.
* `allowlist` : réactions de `guilds.<id>.users` sur tous les messages (liste vide = désactivé).

<div id="tool-action-defaults">
  ### Valeurs par défaut des actions d’outils
</div>

| Groupe d’actions | Par défaut | Remarques |
| --- | --- | --- |
| reactions | activé | Réagir + lister les réactions + emojiList |
| stickers | activé | Envoyer des stickers |
| emojiUploads | activé | Importer des émojis |
| stickerUploads | activé | Importer des stickers |
| polls | activé | Créer des sondages |
| permissions | activé | Instantané des permissions du canal |
| messages | activé | Lire/envoyer/modifier/supprimer |
| threads | activé | Créer/lister/répondre |
| pins | activé | Épingler/désépingler/lister |
| search | activé | Recherche de messages (fonctionnalité en préversion) |
| memberInfo | activé | Informations sur les membres |
| roleInfo | activé | Liste des rôles |
| channelInfo | activé | Informations sur les canaux + liste |
| channels | activé | Gestion des canaux/catégories |
| voiceStatus | activé | Consultation de l’état vocal |
| events | activé | Lister/créer des évènements planifiés |
| roles | désactivé | Ajout/suppression de rôles |
| moderation | désactivé | Timeout/expulsion/bannissement |

* `replyToMode` : `off` (par défaut), `first` ou `all`. Ne s’applique que lorsque le modèle inclut une balise de réponse.

<div id="reply-tags">
  ## Balises de réponse
</div>

Pour demander une réponse dans un fil, le modèle peut inclure une balise dans sa sortie :

* `[[reply_to_current]]` — répondre au message Discord à l’origine de la requête.
* `[[reply_to:<id>]]` — répondre à un ID de message spécifique issu du contexte/de l’historique.
  Les IDs de messages courants sont ajoutés aux prompts sous la forme `[message_id: …]` ; les entrées d’historique incluent déjà des IDs.

Le comportement est contrôlé par `channels.discord.replyToMode` :

* `off` : ignorer les balises.
* `first` : seul le premier bloc/pièce jointe sortant(e) est une réponse.
* `all` : chaque bloc/pièce jointe sortant(e) est une réponse.

Notes sur l’appariement avec la liste d’autorisation :

* `allowFrom`/`users`/`groupChannels` acceptent des IDs, des noms, des tags ou des mentions comme `<@id>`.
* Les préfixes comme `discord:`/`user:` (utilisateurs) et `channel:` (MP de groupe) sont pris en charge.
* Utilisez `*` pour autoriser n’importe quel expéditeur/canal.
* Quand `guilds.<id>.channels` est présent, les canaux non listés sont refusés par défaut.
* Quand `guilds.<id>.channels` est omis, tous les canaux de la guilde présente dans la liste d’autorisation sont autorisés.
* Pour n’autoriser **aucun canal**, définissez `channels.discord.groupPolicy: "disabled"` (ou conservez une liste d’autorisation vide).
* L’assistant de configuration accepte les noms de `Guild/Channel` (publics + privés) et les résout en IDs lorsque c’est possible.
* Au démarrage, OpenClaw résout les noms de canaux/utilisateurs dans les listes d’autorisation en IDs (quand le bot peut rechercher les membres)
  et consigne la table de correspondance dans les logs ; les entrées non résolues sont conservées telles que saisies.

Notes sur les commandes natives :

* Les commandes enregistrées reflètent les commandes de chat d’OpenClaw.
* Les commandes natives respectent les mêmes listes d’autorisation que les MP/messages de guilde (`channels.discord.dm.allowFrom`, `channels.discord.guilds`, règles par canal).
* Les commandes slash peuvent rester visibles dans l’UI Discord pour des utilisateurs qui ne figurent pas dans la liste d’autorisation ; OpenClaw applique les listes d’autorisation lors de l’exécution et répond « non autorisé ».

<div id="tool-actions">
  ## Actions des outils
</div>

L’agent peut appeler `discord` avec des actions comme :

* `react` / `reactions` (ajouter ou lister des réactions)
* `sticker`, `poll`, `permissions`
* `readMessages`, `sendMessage`, `editMessage`, `deleteMessage`
* Les payloads des outils de lecture/recherche/épinglage incluent des `timestampMs` normalisés (ms depuis l’époque UTC) et `timestampUtc`, en plus du `timestamp` Discord brut.
* `threadCreate`, `threadList`, `threadReply`
* `pinMessage`, `unpinMessage`, `listPins`
* `searchMessages`, `memberInfo`, `roleInfo`, `roleAdd`, `roleRemove`, `emojiList`
* `channelInfo`, `channelList`, `voiceStatus`, `eventList`, `eventCreate`
* `timeout`, `kick`, `ban`

Les identifiants de messages Discord sont exposés dans le contexte injecté (`[discord message id: …]` et lignes d’historique) afin que l’agent puisse les cibler.
Les emojis peuvent être en Unicode (par exemple : `✅`) ou utiliser une syntaxe d’emoji personnalisée comme `<:party_blob:1234567890>`.

<div id="safety-ops">
  ## Sécurité &amp; opérations
</div>

* Traitez le jeton du bot comme un mot de passe ; privilégiez la variable d&#39;environnement `DISCORD_BOT_TOKEN` sur les hôtes supervisés ou restreignez les autorisations du fichier de configuration.
* N&#39;accordez au bot que les autorisations dont il a réellement besoin (généralement *Read/Send Messages*).
* Si le bot est bloqué ou soumis à une limitation de débit, redémarrez Gateway (`openclaw gateway --force`) après avoir vérifié qu&#39;aucun autre processus n&#39;utilise la session Discord.