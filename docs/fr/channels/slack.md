---
title: Slack
summary: "Configuration de Slack en mode socket ou webhook HTTP"
read_when: "Configuration ou débogage du mode socket/HTTP de Slack"
---

<div id="slack">
  # Slack
</div>

<div id="socket-mode-default">
  ## Mode Socket (par défaut)
</div>

<div id="quick-setup-beginner">
  ### Configuration rapide (débutant)
</div>

1. Créez une application Slack et activez le **Socket Mode**.
2. Créez un **App Token** (`xapp-...`) et un **Bot Token** (`xoxb-...`).
3. Configurez les jetons pour OpenClaw et démarrez le Gateway.

Configuration minimale :

```json5
{
  channels: {
    slack: {
      enabled: true,
      appToken: "xapp-...",
      botToken: "xoxb-..."
    }
  }
}
```

<div id="setup">
  ### Configuration
</div>

1. Créez une application Slack (**From scratch**) sur https://api.slack.com/apps.
2. **Socket Mode** → activez l’option. Puis allez dans **Basic Information** → **App-Level Tokens** → **Generate Token and Scopes** avec la portée `connections:write`. Copiez le **App Token** (`xapp-...`).
3. **OAuth &amp; Permissions** → ajoutez les autorisations de jeton bot (utilisez le manifeste ci-dessous). Cliquez sur **Install to Workspace**. Copiez le **Bot User OAuth Token** (`xoxb-...`).
4. Facultatif : **OAuth &amp; Permissions** → ajoutez des **User Token Scopes** (voir la liste en lecture seule ci-dessous). Réinstallez l’application et copiez le **User OAuth Token** (`xoxp-...`).
5. **Event Subscriptions** → activez les événements et abonnez-vous à :
   * `message.*` (inclut les éditions/suppressions/diffusions de fils de discussion)
   * `app_mention`
   * `reaction_added`, `reaction_removed`
   * `member_joined_channel`, `member_left_channel`
   * `channel_rename`
   * `pin_added`, `pin_removed`
6. Invitez le bot dans les canaux que vous voulez qu’il puisse lire.
7. Slash Commands → créez `/openclaw` si vous utilisez `channels.slack.slashCommand`. Si vous activez les commandes natives, ajoutez une commande slash par commande intégrée (mêmes noms que `/help`). Par défaut, le mode natif est désactivé pour Slack, sauf si vous définissez `channels.slack.commands.native: true` (la valeur globale de `commands.native` est `"auto"`, ce qui laisse Slack désactivé).
8. App Home → activez l’onglet **Messages Tab** pour que les utilisateurs puissent envoyer des messages privés (DM) au bot.

Utilisez le manifeste ci-dessous pour que les portées et les événements restent synchronisés.

Prise en charge multi‑comptes : utilisez `channels.slack.accounts` avec des jetons par compte et un `name` optionnel. Voir [`gateway/configuration`](/fr/gateway/configuration#telegramaccounts--discordaccounts--slackaccounts--signalaccounts--imessageaccounts) pour le modèle commun.

<div id="openclaw-config-minimal">
  ### Configuration d&#39;OpenClaw (minimale)
</div>

Définissez les jetons via des variables d&#39;environnement (recommandé) :

* `SLACK_APP_TOKEN=xapp-...`
* `SLACK_BOT_TOKEN=xoxb-...`

Ou dans la configuration :

```json5
{
  channels: {
    slack: {
      enabled: true,
      appToken: "xapp-...",
      botToken: "xoxb-..."
    }
  }
}
```

<div id="user-token-optional">
  ### Jeton utilisateur (facultatif)
</div>

OpenClaw peut utiliser un jeton utilisateur Slack (`xoxp-...`) pour les opérations de lecture (historique,
éléments épinglés, réactions, emoji, informations sur les membres). Par défaut, cela reste en lecture seule : les lectures
privilégient le jeton utilisateur lorsqu’il est présent, et les écritures utilisent toujours le jeton de bot, sauf si
vous activez explicitement l’option inverse. Même avec `userTokenReadOnly: false`, le jeton de bot reste
prioritaire pour les écritures lorsqu’il est disponible.

Les jetons utilisateur sont configurés dans le fichier de configuration (aucune prise en charge des variables d’environnement). Pour
plusieurs comptes, définissez `channels.slack.accounts.<id>.userToken`.

Exemple avec des jetons de bot + d’app + d’utilisateur :

```json5
{
  channels: {
    slack: {
      enabled: true,
      appToken: "xapp-...",
      botToken: "xoxb-...",
      userToken: "xoxp-..."
    }
  }
}
```

Exemple avec userTokenReadOnly défini explicitement (autoriser les écritures du jeton utilisateur) :

```json5
{
  channels: {
    slack: {
      enabled: true,
      appToken: "xapp-...",
      botToken: "xoxb-...",
      userToken: "xoxp-...",
      userTokenReadOnly: false
    }
  }
}
```

<div id="token-usage">
  #### Utilisation des jetons
</div>

* Les opérations de lecture (historique, liste des réactions, liste des éléments épinglés, liste des émojis, informations sur les membres, recherche) privilégient le jeton utilisateur lorsqu’il est configuré, sinon le jeton de bot.
* Les opérations d’écriture (envoi/modification/suppression de messages, ajout/suppression de réactions, épinglage/désépinglage, téléversement de fichiers) utilisent le jeton de bot par défaut. Si `userTokenReadOnly: false` et qu’aucun jeton de bot n’est disponible, OpenClaw utilise alors le jeton utilisateur.

<div id="history-context">
  ### Contexte de l&#39;historique
</div>

* `channels.slack.historyLimit` (ou `channels.slack.accounts.*.historyLimit`) définit le nombre de messages récents du canal/groupe qui sont intégrés dans le prompt.
* À défaut, la valeur de `messages.groupChat.historyLimit` est utilisée. Définissez `0` pour désactiver (50 par défaut).

<div id="http-mode-events-api">
  ## Mode HTTP (Events API)
</div>

Utilisez le mode webhook HTTP lorsque votre Gateway est accessible depuis Slack via HTTPS (cas typique pour les déploiements sur serveur).
Le mode HTTP utilise Events API + Interactivity + Slash Commands avec une URL de requête partagée.

<div id="setup">
  ### Configuration
</div>

1. Créez une application Slack et **désactivez le Socket Mode** (facultatif si vous n&#39;utilisez que HTTP).
2. Dans **Basic Information**, copiez le **Signing Secret**.
3. Dans **OAuth &amp; Permissions**, installez l&#39;application et copiez le **Bot User OAuth Token** (`xoxb-...`).
4. Dans **Event Subscriptions**, activez les événements et définissez la **Request URL** sur le chemin de webhook de votre Gateway (par défaut `/slack/events`).
5. Dans **Interactivity &amp; Shortcuts**, activez l&#39;option et définissez la même **Request URL**.
6. Dans **Slash Commands**, définissez la même **Request URL** pour vos commandes.

Exemple de Request URL :
`https://gateway-host/slack/events`

### Configuration d’OpenClaw (minimale)

```json5
{
  channels: {
    slack: {
      enabled: true,
      mode: "http",
      botToken: "xoxb-...",
      signingSecret: "your-signing-secret",
      webhookPath: "/slack/events"
    }
  }
}
```

Mode HTTP multicompte : définissez `channels.slack.accounts.<id>.mode = "http"` et fournissez un
`webhookPath` unique par compte pour que chaque application Slack puisse pointer vers sa propre URL.

<div id="manifest-optional">
  ### Manifest (facultatif)
</div>

Utilisez ce manifest d&#39;application Slack pour créer rapidement l&#39;application (ajustez le nom/la commande si vous le souhaitez). Incluez les portées utilisateur si vous prévoyez de configurer un jeton utilisateur.

```json
{
  "display_information": {
    "name": "OpenClaw",
    "description": "Slack connector for OpenClaw"
  },
  "features": {
    "bot_user": {
      "display_name": "OpenClaw",
      "always_online": false
    },
    "app_home": {
      "messages_tab_enabled": true,
      "messages_tab_read_only_enabled": false
    },
    "slash_commands": [
      {
        "command": "/openclaw",
        "description": "Send a message to OpenClaw",
        "should_escape": false
      }
    ]
  },
  "oauth_config": {
    "scopes": {
      "bot": [
        "chat:write",
        "channels:history",
        "channels:read",
        "groups:history",
        "groups:read",
        "groups:write",
        "im:history",
        "im:read",
        "im:write",
        "mpim:history",
        "mpim:read",
        "mpim:write",
        "users:read",
        "app_mentions:read",
        "reactions:read",
        "reactions:write",
        "pins:read",
        "pins:write",
        "emoji:read",
        "commands",
        "files:read",
        "files:write"
      ],
      "user": [
        "channels:history",
        "channels:read",
        "groups:history",
        "groups:read",
        "im:history",
        "im:read",
        "mpim:history",
        "mpim:read",
        "users:read",
        "reactions:read",
        "pins:read",
        "emoji:read",
        "search:read"
      ]
    }
  },
  "settings": {
    "socket_mode_enabled": true,
    "event_subscriptions": {
      "bot_events": [
        "app_mention",
        "message.channels",
        "message.groups",
        "message.im",
        "message.mpim",
        "reaction_added",
        "reaction_removed",
        "member_joined_channel",
        "member_left_channel",
        "channel_rename",
        "pin_added",
        "pin_removed"
      ]
    }
  }
}
```

Si vous activez les commandes natives, ajoutez une entrée `slash_commands` pour chaque commande que vous voulez exposer (correspondant à la liste `/help`). Vous pouvez surcharger ce comportement avec `channels.slack.commands.native`.

<div id="scopes-current-vs-optional">
  ## Portées (actuelles et facultatives)
</div>

L&#39;API Conversations de Slack utilise des portées par type : vous n&#39;avez besoin que des portées
pour les types de conversation que vous manipulez effectivement (channels, groups, im, mpim). Pour une présentation générale, voir
https://docs.slack.dev/apis/web-api/using-the-conversations-api/.

<div id="bot-token-scopes-required">
  ### Scopes de jeton de bot (obligatoires)
</div>

* `chat:write` (envoyer, mettre à jour et supprimer des messages via `chat.postMessage`)
  https://docs.slack.dev/reference/methods/chat.postMessage
* `im:write` (ouvrir des DMs via `conversations.open` pour les DMs utilisateur)
  https://docs.slack.dev/reference/methods/conversations.open
* `channels:history`, `groups:history`, `im:history`, `mpim:history`
  https://docs.slack.dev/reference/methods/conversations.history
* `channels:read`, `groups:read`, `im:read`, `mpim:read`
  https://docs.slack.dev/reference/methods/conversations.info
* `users:read` (recherche d’utilisateurs)
  https://docs.slack.dev/reference/methods/users.info
* `reactions:read`, `reactions:write` (`reactions.get` / `reactions.add`)
  https://docs.slack.dev/reference/methods/reactions.get
  https://docs.slack.dev/reference/methods/reactions.add
* `pins:read`, `pins:write` (`pins.list` / `pins.add` / `pins.remove`)
  https://docs.slack.dev/reference/scopes/pins.read
  https://docs.slack.dev/reference/scopes/pins.write
* `emoji:read` (`emoji.list`)
  https://docs.slack.dev/reference/scopes/emoji.read
* `files:write` (téléversements via `files.uploadV2`)
  https://docs.slack.dev/messaging/working-with-files/#upload

<div id="user-token-scopes-optional-read-only-by-default">
  ### Portées du jeton utilisateur (optionnelles, en lecture seule par défaut)
</div>

Ajoutez-les sous **User Token Scopes** si vous configurez `channels.slack.userToken`.

* `channels:history`, `groups:history`, `im:history`, `mpim:history`
* `channels:read`, `groups:read`, `im:read`, `mpim:read`
* `users:read`
* `reactions:read`
* `pins:read`
* `emoji:read`
* `search:read`

<div id="not-needed-today-but-likely-future">
  ### Pas nécessaire aujourd’hui (mais probablement à l’avenir)
</div>

* `mpim:write` (uniquement si nous ajoutons l’ouverture de MP de groupe / le démarrage de MP via `conversations.open`)
* `groups:write` (uniquement si nous ajoutons la gestion des canaux privés : création/renommage/invitation/archivage)
* `chat:write.public` (uniquement si nous voulons publier dans des canaux où le bot n’est pas présent)
  https://docs.slack.dev/reference/scopes/chat.write.public
* `users:read.email` (uniquement si nous avons besoin des champs d’adresse e‑mail depuis `users.info`)
  https://docs.slack.dev/changelog/2017-04-narrowing-email-access
* `files:read` (uniquement si nous commençons à lister/lire les métadonnées de fichiers)

<div id="config">
  ## Config
</div>

Slack utilise uniquement Socket Mode (sans serveur de webhook HTTP). Fournissez les deux jetons :

```json
{
  "slack": {
    "enabled": true,
    "botToken": "xoxb-...",
    "appToken": "xapp-...",
    "groupPolicy": "allowlist",
    "dm": {
      "enabled": true,
      "policy": "pairing",
      "allowFrom": ["U123", "U456", "*"],
      "groupEnabled": false,
      "groupChannels": ["G123"],
      "replyToMode": "all"
    },
    "channels": {
      "C123": { "allow": true, "requireMention": true },
      "#general": {
        "allow": true,
        "requireMention": true,
        "users": ["U123"],
        "skills": ["search", "docs"],
        "systemPrompt": "Keep answers short."
      }
    },
    "reactionNotifications": "own",
    "reactionAllowlist": ["U123"],
    "replyToMode": "off",
    "actions": {
      "reactions": true,
      "messages": true,
      "pins": true,
      "memberInfo": true,
      "emojiList": true
    },
    "slashCommand": {
      "enabled": true,
      "name": "openclaw",
      "sessionPrefix": "slack:slash",
      "ephemeral": true
    },
    "textChunkLimit": 4000,
    "mediaMaxMb": 20
  }
}
```

Les jetons peuvent également être fournis via des variables d&#39;environnement :

* `SLACK_BOT_TOKEN`
* `SLACK_APP_TOKEN`

Les réactions d&#39;accusé de réception sont contrôlées globalement via `messages.ackReaction` +
`messages.ackReactionScope`. Utilisez `messages.removeAckAfterReply` pour supprimer la
réaction d&#39;accusé de réception après que le bot a répondu.

<div id="limits">
  ## Limites
</div>

* Le texte sortant est découpé selon `channels.slack.textChunkLimit` (4000 par défaut).
* Découpage optionnel par retour à la ligne : définissez `channels.slack.chunkMode="newline"` pour scinder sur les lignes vides (délimitations de paragraphe) avant le découpage par longueur.
* Les envois de médias sont plafonnés par `channels.slack.mediaMaxMb` (20 par défaut).

<div id="reply-threading">
  ## Fil de réponses
</div>

Par défaut, OpenClaw répond dans le canal principal. Utilisez `channels.slack.replyToMode` pour contrôler la mise en fil de discussion automatique :

| Mode | Comportement |
| --- | --- |
| `off` | **Par défaut.** Répond dans le canal principal. Ne répond en fil que si le message déclencheur se trouvait déjà dans un fil. |
| `first` | La première réponse va dans un fil (sous le message déclencheur), les réponses suivantes vont dans le canal principal. Utile pour garder le contexte visible tout en évitant d’encombrer les fils. |
| `all` | Toutes les réponses vont dans un fil. Permet de garder les conversations regroupées dans un fil mais peut réduire la visibilité. |

Le mode s’applique à la fois aux réponses automatiques et aux appels d’outils d’agent (`slack sendMessage`).

<div id="per-chat-type-threading">
  ### Gestion des fils par type de conversation
</div>

Vous pouvez configurer un comportement de gestion des fils différent pour chaque type de conversation en définissant `channels.slack.replyToModeByChatType` :

```json5
{
  channels: {
    slack: {
      replyToMode: "off",        // default for channels
      replyToModeByChatType: {
        direct: "all",           // DMs always thread
        group: "first"           // les DM de groupe/MPIM mettent la première réponse en fil
      },
    }
  }
}
```

Types de discussion pris en charge :

* `direct` : MP 1:1 (Slack `im`)
* `group` : MP de groupe / MPIM (Slack `mpim`)
* `channel` : canaux standard (public/privé)

Ordre de priorité :

1. `replyToModeByChatType.<chatType>`
2. `replyToMode`
3. Valeur par défaut du fournisseur (`off`)

L’ancienne clé `channels.slack.dm.replyToMode` est toujours acceptée comme solution de repli pour `direct` lorsqu’aucune surcharge spécifique au type de discussion n’est définie.

Exemples :

Fils de discussion uniquement pour les MP :

```json5
{
  channels: {
    slack: {
      replyToMode: "off",
      replyToModeByChatType: { direct: "all" }
    }
  }
}
```

Regroupez les fils de discussion dans les messages privés de groupe, mais laissez les canaux à la racine :

```json5
{
  channels: {
    slack: {
      replyToMode: "off",
      replyToModeByChatType: { group: "first" }
    }
  }
}
```

Activez les fils dans les canaux, gardez les MP à la racine :

```json5
{
  channels: {
    slack: {
      replyToMode: "first",
      replyToModeByChatType: { direct: "off", group: "off" }
    }
  }
}
```

<div id="manual-threading-tags">
  ### Étiquettes de fil de discussion manuelles
</div>

Pour un contrôle plus fin, utilisez ces étiquettes dans les réponses de l’agent :

* `[[reply_to_current]]` — répondre au message déclencheur (démarrer/continuer le fil de discussion).
* `[[reply_to:<id>]]` — répondre à un ID de message spécifique.

<div id="sessions-routing">
  ## Sessions + routage
</div>

* Les DM partagent la session `main` (comme sur WhatsApp/Telegram).
* Les canaux sont associés à des sessions `agent:<agentId>:slack:channel:<channelId>`.
* Les slash commands utilisent des sessions `agent:<agentId>:slack:slash:<userId>` (préfixe configurable via `channels.slack.slashCommand.sessionPrefix`).
* Si Slack ne fournit pas `channel_type`, OpenClaw le déduit à partir du préfixe de l’ID de canal (`D`, `C`, `G`) et utilise par défaut `channel` pour conserver des clés de session stables.
* L’enregistrement natif des commandes utilise `commands.native` (valeur globale par défaut `"auto"` → Slack désactivé) et peut être surchargé au niveau de chaque espace de travail avec `channels.slack.commands.native`. Les commandes textuelles nécessitent des messages `/...` isolés et peuvent être désactivées avec `commands.text: false`. Les slash commands Slack sont gérées dans l’application Slack et ne sont pas supprimées automatiquement. Utilisez `commands.useAccessGroups: false` pour contourner les vérifications de groupes d’accès pour les commandes.
* Liste complète des commandes + configuration : [Slash commands](/fr/tools/slash-commands)

<div id="dm-security-pairing">
  ## Sécurité des MP (appairage)
</div>

* Valeur par défaut : `channels.slack.dm.policy="pairing"` — les expéditeurs inconnus de MP reçoivent un code d’appairage (expire après 1 heure).
* Approbation via : `openclaw pairing approve slack <code>`.
* Pour autoriser n’importe qui : définissez `channels.slack.dm.policy="open"` (le jeton de stratégie `open` autorise l’acceptation de messages sans restriction de la part de n’importe quel utilisateur) et `channels.slack.dm.allowFrom=["*"]`.
* `channels.slack.dm.allowFrom` accepte des ID d’utilisateur, des pseudos @ ou des adresses e‑mail (résolus au démarrage lorsque les jetons d’accès le permettent). L’assistant de configuration accepte les noms d’utilisateur et les résout en ID durant la configuration lorsque les jetons d’accès le permettent.

<div id="group-policy">
  ## Stratégie de groupe
</div>

* `channels.slack.groupPolicy` contrôle la gestion des canaux (`open|disabled|allowlist`).
* `allowlist` exige que les canaux soient listés dans `channels.slack.channels`.
* Si vous définissez uniquement `SLACK_BOT_TOKEN`/`SLACK_APP_TOKEN` et ne créez jamais de section `channels.slack`,
  le runtime définit par défaut `groupPolicy` sur `open` (paramètre autorisant l’acceptation de messages sans restriction depuis n’importe quel utilisateur). Ajoutez `channels.slack.groupPolicy`,
  `channels.defaults.groupPolicy` ou une liste d’autorisation de canaux pour verrouiller la configuration.
* L’assistant de configuration accepte les noms de `#channel` et les résout en ID lorsque possible
  (publics + privés) ; si plusieurs correspondances existent, il privilégie le canal actif.
* Au démarrage, OpenClaw résout les noms de canal/utilisateur dans les listes d’autorisation en ID (quand les jetons le permettent)
  et consigne la correspondance dans les logs ; les entrées non résolues sont conservées telles que saisies.
* Pour n’autoriser **aucun canal**, définissez `channels.slack.groupPolicy: "disabled"` (ou gardez une liste d’autorisation vide).

Options de canal (`channels.slack.channels.<id>` ou `channels.slack.channels.<name>`) :

* `allow` : autoriser/refuser le canal lorsque `groupPolicy="allowlist"`.
* `requireMention` : contrôle d’accès par mention pour le canal.
* `tools` : surcharges optionnelles de stratégie d’outils par canal (`allow`/`deny`/`alsoAllow`).
* `toolsBySender` : surcharges optionnelles de stratégie d’outils par expéditeur dans le canal (les clés sont des ID d’expéditeur/@handles/adresses e-mail ; le caractère générique `"*"` est pris en charge).
* `allowBots` : autoriser les messages rédigés par des bots dans ce canal (par défaut : false).
* `users` : liste d’autorisation d’utilisateurs optionnelle, par canal.
* `skills` : filtre de compétences (omission = toutes les compétences, vide = aucune).
* `systemPrompt` : prompt système supplémentaire pour le canal (combiné avec le sujet/l’objectif).
* `enabled` : définissez sur `false` pour désactiver le canal.

<div id="delivery-targets">
  ## Cibles de diffusion
</div>

Utilisez-les avec les envois cron/CLI :

* `user:&lt;id&gt;` pour les messages directs
* `channel:&lt;id&gt;` pour les canaux

<div id="tool-actions">
  ## Actions d&#39;outils
</div>

Les actions d&#39;outils Slack peuvent être restreintes avec `channels.slack.actions.*` :

| Groupe d&#39;actions | Valeur par défaut | Remarques |
| --- | --- | --- |
| reactions | activé | Réagir + lister les réactions |
| messages | activé | Lecture/Envoyer/modification/suppression |
| pins | activé | Épingler/désépingler/lister |
| memberInfo | activé | Informations sur les membres |
| emojiList | activé | Liste d&#39;émojis personnalisés |

<div id="security-notes">
  ## Notes de sécurité
</div>

* Les écritures utilisent par défaut le bot token afin que les actions modifiant l’état restent limitées aux autorisations et à l’identité du bot de l’application.
* Définir `userTokenReadOnly: false` permet au user token d’être utilisé pour des opérations d’écriture lorsqu’aucun bot token n’est disponible, ce qui signifie que les actions s’exécutent avec les droits de l’utilisateur ayant procédé à l’installation. Traitez le user token comme un jeton hautement privilégié et gardez les garde-fous sur les actions ainsi que les listes d’autorisation très restrictifs.
* Si vous activez les écritures avec un user token, assurez-vous que le user token inclut les portées d’écriture attendues (`chat:write`, `reactions:write`, `pins:write`, `files:write`), sinon ces opérations échoueront.

<div id="notes">
  ## Notes
</div>

* Le filtrage par mention est contrôlé via `channels.slack.channels` (définissez `requireMention` à `true`) ; `agents.list[].groupChat.mentionPatterns` (ou `messages.groupChat.mentionPatterns`) comptent également comme des mentions.
* Surcharge multi-agents : définissez des motifs spécifiques à chaque agent dans `agents.list[].groupChat.mentionPatterns`.
* Les notifications de réactions respectent `channels.slack.reactionNotifications` (utilisez `reactionAllowlist` avec le mode `allowlist`).
* Les messages émis par des bots sont ignorés par défaut ; activez-les via `channels.slack.allowBots` ou `channels.slack.channels.<id>.allowBots`.
* Avertissement : si vous autorisez les réponses à d&#39;autres bots (`channels.slack.allowBots=true` ou `channels.slack.channels.<id>.allowBots=true`), empêchez les boucles de réponses entre bots avec `requireMention`, des listes d’autorisation `channels.slack.channels.<id>.users` et/ou des garde-fous explicites dans `AGENTS.md` et `SOUL.md`.
* Pour l’outil Slack, les règles de suppression des réactions sont décrites dans [/tools/reactions](/fr/tools/reactions).
* Les pièces jointes sont téléchargées dans le media store lorsqu&#39;elles y sont autorisées et qu&#39;elles respectent la limite de taille.