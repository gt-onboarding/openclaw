---
title: Groupes
summary: "Comportement des discussions de groupe sur les différentes plateformes (WhatsApp/Telegram/Discord/Slack/Signal/iMessage/Microsoft Teams)"
read_when:
  - Modification du comportement des discussions de groupe ou du filtrage des mentions
---

<div id="groups">
  # Groupes
</div>

OpenClaw traite les discussions de groupe de manière cohérente sur tous les canaux suivants : WhatsApp, Telegram, Discord, Slack, Signal, iMessage, Microsoft Teams.

<div id="beginner-intro-2-minutes">
  ## Introduction pour débutants (2 minutes)
</div>

OpenClaw « vit » sur vos propres comptes de messagerie. Il n’y a pas d’utilisateur bot WhatsApp séparé.
Si **vous** êtes dans un groupe, OpenClaw peut voir ce groupe et y répondre.

Comportement par défaut :

* Les groupes sont restreints (`groupPolicy: "allowlist"`).
* Les réponses nécessitent une mention, sauf si vous désactivez explicitement la restriction par mention.

Traduction : les expéditeurs figurant dans la liste d’autorisation peuvent déclencher OpenClaw en le mentionnant.

> TL;DR
>
> * **L’accès en DM** est contrôlé par `*.allowFrom`.
> * **L’accès aux groupes** est contrôlé par `*.groupPolicy` + les listes d’autorisation (`*.groups`, `*.groupAllowFrom`).
> * **Le déclenchement des réponses** est contrôlé par la restriction par mention (`requireMention`, `/activation`).

Flux rapide (ce qui se passe pour un message de groupe) :

```
groupPolicy? disabled -> drop
groupPolicy? allowlist -> group allowed? no -> drop
requireMention? yes -> mentioned? no -> store for context only
otherwise -> reply
```

![Flux des messages de groupe](/images/groups-flow.svg)

Si vous voulez…

| Objectif                                                     | Ce qu’il faut configurer                                   |
| ------------------------------------------------------------ | ---------------------------------------------------------- |
| Autoriser tous les groupes mais ne répondre qu’aux @mentions | `groups: { "*": { requireMention: true } }`                |
| Désactiver toutes les réponses dans les groupes              | `groupPolicy: "disabled"`                                  |
| Uniquement certains groupes                                  | `groups: { "<group-id>": { ... } }` (sans clé `"*"`)       |
| Vous seul pouvez déclencher dans les groupes                 | `groupPolicy: "allowlist"`, `groupAllowFrom: ["+1555..."]` |

<div id="session-keys">
  ## Clés de session
</div>

* Les sessions de groupe utilisent des clés de session `agent:<agentId>:<channel>:group:<id>` (les salons/canaux utilisent `agent:<agentId>:<channel>:channel:<id>`).
* Les sujets de forum Telegram ajoutent `:topic:<threadId>` à l’identifiant de groupe afin que chaque sujet ait sa propre session.
* Les conversations directes utilisent la session principale (ou une session par expéditeur si configuré).
* Les signaux de vie sont ignorés pour les sessions de groupe.

<div id="pattern-personal-dms-public-groups-single-agent">
  ## Modèle : DMs personnels + groupes publics (agent unique)
</div>

Oui — ce modèle fonctionne bien si votre trafic « personnel » est constitué de **DMs** et votre trafic « public » de **groupes**.

Pourquoi : en mode agent unique, les DMs arrivent généralement dans la clé de **session** principale (`agent:main:main`), tandis que les groupes utilisent toujours des clés de **session** non principales (`agent:main:<channel>:group:<id>`). Si vous activez la sandbox avec `mode: "non-main"`, ces sessions de groupe s’exécutent dans Docker tandis que votre session DM principale reste sur l’hôte.

Cela vous donne un seul « cerveau » d’agent (espace de travail + mémoire partagés), mais deux profils d’exécution :

* **DMs** : tous les outils (hôte)
* **Groupes** : sandbox + outils restreints (Docker)

> Si vous avez besoin d’espaces de travail/personas réellement séparés (les contextes « personnel » et « public » ne doivent jamais se mélanger), utilisez un deuxième agent + des bindings. Voir [Routage multi-agents](/fr/concepts/multi-agent).

Exemple (DMs sur l’hôte, groupes en sandbox + outils limités à la messagerie) :

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main", // groups/channels are non-main -> sandboxed
        scope: "session", // strongest isolation (one container per group/channel)
        workspaceAccess: "none"
      }
    }
  },
  tools: {
    sandbox: {
      tools: {
        // Si allow n'est pas vide, tout le reste est bloqué (deny l'emporte toujours).
        allow: ["group:messaging", "group:sessions"],
        deny: ["group:runtime", "group:fs", "group:ui", "nodes", "cron", "gateway"]
      }
    }
  }
}
```

Vous préférez « les groupes ne peuvent voir que le dossier X » plutôt que « aucun accès à l’hôte » ? Conservez `workspaceAccess: "none"` et montez uniquement les chemins figurant dans la liste d’autorisation dans la sandbox :

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main",
        scope: "session",
        workspaceAccess: "none",
        docker: {
          binds: [
            // cheminHôte:cheminConteneur:mode
            "~/FriendsShared:/data:ro"
          ]
        }
      }
    }
  }
}
```

Contenu associé :

* Clés de configuration et valeurs par défaut : [Configuration du Gateway](/fr/gateway/configuration#agentsdefaultssandbox)
* Débogage de la raison pour laquelle un outil est bloqué : [Sandbox vs Tool Policy vs Elevated](/fr/gateway/sandbox-vs-tool-policy-vs-elevated)
* Détails des montages bind : [Sandboxing](/fr/gateway/sandboxing#custom-bind-mounts)

<div id="display-labels">
  ## Libellés d’affichage
</div>

* Les libellés UI utilisent `displayName` lorsqu’il est disponible, au format `<channel>:<token>`.
* `#room` est réservé aux salons/canaux ; les conversations de groupe utilisent `g-<slug>` (minuscules, espaces → `-`, conserver `#@+._-`).

<div id="group-policy">
  ## Politique de groupe
</div>

Contrôlez la manière dont les messages de groupes/salons sont traités pour chaque canal :

```json5
{
  channels: {
    whatsapp: {
      groupPolicy: "disabled", // "open" (acceptation sans restriction) | "disabled" | "allowlist"
      groupAllowFrom: ["+15551234567"]
    },
    telegram: {
      groupPolicy: "disabled",
      groupAllowFrom: ["123456789", "@username"]
    },
    signal: {
      groupPolicy: "disabled",
      groupAllowFrom: ["+15551234567"]
    },
    imessage: {
      groupPolicy: "disabled",
      groupAllowFrom: ["chat_id:123"]
    },
    msteams: {
      groupPolicy: "disabled",
      groupAllowFrom: ["user@org.com"]
    },
    discord: {
      groupPolicy: "allowlist",
      guilds: {
        "GUILD_ID": { channels: { help: { allow: true } } }
      }
    },
    slack: {
      groupPolicy: "allowlist",
      channels: { "#general": { allow: true } }
    },
    matrix: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["@owner:example.org"],
      groups: {
        "!roomId:example.org": { allow: true },
        "#alias:example.org": { allow: true }
      }
    }
  }
}
```

| Policy        | Behavior                                                                                    |
| ------------- | ------------------------------------------------------------------------------------------- |
| `"open"`      | Les groupes contournent les listes d’autorisation ; le filtrage par mention reste appliqué. |
| `"disabled"`  | Bloque entièrement tous les messages de groupe.                                             |
| `"allowlist"` | N’autorise que les groupes/salons correspondant à la liste d’autorisation configurée.       |

Notes :

* `groupPolicy` est distinct du filtrage par mention (qui nécessite des @mentions).
* WhatsApp/Telegram/Signal/iMessage/Microsoft Teams : utilisez `groupAllowFrom` (à défaut : `allowFrom` explicite).
* Discord : la liste d’autorisation utilise `channels.discord.guilds.<id>.channels`.
* Slack : la liste d’autorisation utilise `channels.slack.channels`.
* Matrix : la liste d’autorisation utilise `channels.matrix.groups` (ID de salons, alias ou noms). Utilisez `channels.matrix.groupAllowFrom` pour restreindre les expéditeurs ; des listes d’autorisation `users` par salon sont également prises en charge.
* Les messages privés de groupe (Group DMs) sont contrôlés séparément (`channels.discord.dm.*`, `channels.slack.dm.*`).
* La liste d’autorisation Telegram peut correspondre à des ID utilisateur (`"123456789"`, `"telegram:123456789"`, `"tg:123456789"`) ou à des noms d’utilisateur (`"@alice"` ou `"alice"`) ; les préfixes ne sont pas sensibles à la casse.
* La valeur par défaut est `groupPolicy: "allowlist"` ; si votre liste d’autorisation de groupes est vide, les messages de groupe sont bloqués.

Modèle mental rapide (ordre d’évaluation pour les messages de groupe) :

1. `groupPolicy` (open/disabled/allowlist)
2. Listes d’autorisation de groupes (`*.groups`, `*.groupAllowFrom`, liste d’autorisation spécifique au canal)
3. Filtrage par mention (`requireMention`, `/activation`)

<div id="mention-gating-default">
  ## Filtrage par mention (par défaut)
</div>

Les messages de groupe exigent une mention, sauf si ce comportement est remplacé au niveau du groupe. Les valeurs par défaut sont définies par sous-système sous `*.groups."*"`.

Répondre à un message de bot compte comme une mention implicite (lorsque le canal prend en charge les métadonnées de réponse). Cela s’applique à Telegram, WhatsApp, Slack, Discord et Microsoft Teams.

```json5
{
  channels: {
    whatsapp: {
      groups: {
        "*": { requireMention: true },
        "123@g.us": { requireMention: false }
      }
    },
    telegram: {
      groups: {
        "*": { requireMention: true },
        "123456789": { requireMention: false }
      }
    },
    imessage: {
      groups: {
        "*": { requireMention: true },
        "123": { requireMention: false }
      }
    }
  },
  agents: {
    list: [
      {
        id: "main",
        groupChat: {
          mentionPatterns: ["@openclaw", "openclaw", "\\+15555550123"],
          historyLimit: 50
        }
      }
    ]
  }
}
```

Notes :

* Les `mentionPatterns` sont des regex insensibles à la casse.
* Les interfaces qui fournissent des mentions explicites passent toujours ; les motifs sont un mécanisme de repli.
* Surcharge par agent : `agents.list[].groupChat.mentionPatterns` (utile lorsque plusieurs agents partagent un groupe).
* Le filtrage par mention est appliqué uniquement lorsque la détection de mention est possible (mentions natives ou `mentionPatterns` configurés).
* Les paramètres par défaut Discord se trouvent dans `channels.discord.guilds."*"` (surchargés par serveur/canal).
* Le contexte d’historique de groupe est encapsulé de manière uniforme sur tous les canaux et est **pending-only** (messages ignorés en raison du filtrage par mention) ; utilise `messages.groupChat.historyLimit` pour la valeur globale par défaut et `channels.<channel>.historyLimit` (ou `channels.<channel>.accounts.*.historyLimit`) pour les surcharges. Définis `0` pour désactiver.

<div id="groupchannel-tool-restrictions-optional">
  ## Restrictions d’outils par groupe/canal (optionnel)
</div>

Certaines configurations de canal permettent de restreindre les outils disponibles **à l’intérieur d’un groupe/salon/canal spécifique**.

* `tools` : autoriser/interdire des outils pour l’ensemble du groupe.
* `toolsBySender` : surcharges par expéditeur au sein du groupe (les clés sont des identifiants d’expéditeur, des noms d’utilisateur, des adresses e‑mail ou des numéros de téléphone selon le canal). Utilisez `"*"` comme joker.

Ordre de résolution (la règle la plus spécifique l’emporte) :

1. correspondance `toolsBySender` du groupe/canal
2. `tools` du groupe/canal
3. correspondance `toolsBySender` par défaut (`"*"`)
4. `tools` par défaut (`"*"`)

Exemple (Telegram) :

```json5
{
  channels: {
    telegram: {
      groups: {
        "*": { tools: { deny: ["exec"] } },
        "-1001234567890": {
          tools: { deny: ["exec", "read", "write"] },
          toolsBySender: {
            "123456789": { alsoAllow: ["exec"] }
          }
        }
      }
    }
  }
}
```

Remarques :

* Les restrictions d’utilisation des outils par groupe/canal s’appliquent en complément de la stratégie d’outils globale/de l’agent (les interdictions priment toujours).
* Certains canaux utilisent une hiérarchie différente pour les salons/canaux (par ex. Discord `guilds.*.channels.*`, Slack `channels.*`, MS Teams `teams.*.channels.*`).

<div id="group-allowlists">
  ## Listes d’autorisation de groupes
</div>

Lorsque `channels.whatsapp.groups`, `channels.telegram.groups` ou `channels.imessage.groups` est configuré, les clés agissent comme une liste d’autorisation de groupes. Utilisez `"*"` pour autoriser tous les groupes tout en définissant le comportement de mention par défaut.

Cas d’usage courants (copier/coller) :

1. Désactiver toutes les réponses de groupe

```json5
{
  channels: { whatsapp: { groupPolicy: "disabled" } }
}
```

2. N’autoriser que certains groupes (WhatsApp)

```json5
{
  channels: {
    whatsapp: {
      groups: {
        "123@g.us": { requireMention: true },
        "456@g.us": { requireMention: false }
      }
    }
  }
}
```

3. Autoriser tous les groupes mais exiger une mention explicite

```json5
{
  channels: {
    whatsapp: {
      groups: { "*": { requireMention: true } }
    }
  }
}
```

4. Seul le propriétaire peut déclencher le bot dans les groupes (WhatsApp)

```json5
{
  channels: {
    whatsapp: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15551234567"],
      groups: { "*": { requireMention: true } }
    }
  }
}
```

<div id="activation-owner-only">
  ## Activation (propriétaire uniquement)
</div>

Les propriétaires de groupe peuvent activer ou désactiver la fonctionnalité par groupe :

* `/activation mention`
* `/activation always`

Le propriétaire est déterminé par `channels.whatsapp.allowFrom` (ou le numéro E.164 propre au bot lorsqu’il n’est pas défini). Envoyez la commande dans un message distinct. Les autres interfaces ignorent actuellement `/activation`.

<div id="context-fields">
  ## Champs de contexte
</div>

Les payloads entrants de groupe définissent :

* `ChatType=group`
* `GroupSubject` (si disponible)
* `GroupMembers` (si disponible)
* `WasMentioned` (résultat du filtrage par mention)
* Les sujets de forum Telegram incluent également `MessageThreadId` et `IsForum`.

La consigne système de l’agent inclut une introduction de groupe lors du premier échange d’une nouvelle session de groupe. Elle rappelle au modèle de répondre comme un humain, d’éviter les tableaux Markdown et de ne pas taper de séquences « \n » littérales.

<div id="imessage-specifics">
  ## Spécificités iMessage
</div>

* Privilégiez `chat_id:<id>` lors du routage ou de la définition de la liste d’autorisation.
* Listez les conversations : `imsg chats --limit 20`.
* Les réponses dans les groupes reviennent toujours vers le même `chat_id`.

<div id="whatsapp-specifics">
  ## Spécificités propres à WhatsApp
</div>

Consultez [Messages de groupe](/fr/concepts/group-messages) pour le comportement spécifique à WhatsApp (injection de l’historique, détails sur la gestion des mentions).