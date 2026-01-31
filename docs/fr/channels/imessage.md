---
title: iMessage
summary: "Prise en charge d’iMessage via imsg (JSON-RPC sur stdio), configuration et routage par chat_id"
read_when:
  - Configuration de la prise en charge d’iMessage
  - Débogage de l’envoi et de la réception iMessage
---

<div id="imessage-imsg">
  # iMessage (imsg)
</div>

Statut : intégration CLI externe. Gateway lance `imsg rpc` (JSON-RPC sur stdio).

<div id="quick-setup-beginner">
  ## Configuration rapide (débutant)
</div>

1. Assurez-vous que Messages est ouvert et connecté sur ce Mac.
2. Installez `imsg` :
   * `brew install steipete/tap/imsg`
3. Configurez OpenClaw avec `channels.imessage.cliPath` et `channels.imessage.dbPath`.
4. Démarrez Gateway et approuvez toutes les boîtes de dialogue macOS (Automatisation + Accès complet au disque).

Configuration minimale :

```json5
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "/usr/local/bin/imsg",
      dbPath: "/Users/<vous>/Library/Messages/chat.db"
    }
  }
}
```

<div id="what-it-is">
  ## De quoi il s&#39;agit
</div>

* Canal iMessage reposant sur `imsg` sous macOS.
* Routage déterministe : les réponses retournent toujours vers iMessage.
* Les messages privés (DM) partagent la session principale de l&#39;agent ; les groupes sont isolés (`agent:<agentId>:imessage:group:<chat_id>`).
* Si une conversation à plusieurs participants arrive avec `is_group=false`, vous pouvez tout de même l&#39;isoler par `chat_id` via `channels.imessage.groups` (voir « Fils de type groupe » ci-dessous).

<div id="config-writes">
  ## Modifications de configuration
</div>

Par défaut, iMessage est autorisé à écrire des mises à jour de configuration déclenchées par `/config set|unset` (nécessite `commands.config: true`).

Pour désactiver :

```json5
{
  channels: { imessage: { configWrites: false } }
}
```

<div id="requirements">
  ## Prérequis
</div>

* macOS avec Messages configuré et connecté.
* Accès complet au disque pour OpenClaw + `imsg` (accès à la base de données Messages).
* Autorisation d’automatisation pour l’envoi.
* `channels.imessage.cliPath` peut pointer vers n’importe quelle commande qui fait office de proxy pour stdin/stdout (par exemple, un script wrapper qui se connecte en SSH à un autre Mac et exécute `imsg rpc`).

<div id="setup-fast-path">
  ## Configuration (procédure rapide)
</div>

1. Vérifiez que Messages est connecté sur ce Mac.
2. Configurez iMessage puis démarrez Gateway.

<div id="dedicated-bot-macos-user-for-isolated-identity">
  ### Utilisateur macOS dédié pour le bot (pour isoler l’identité)
</div>

Si vous voulez que le bot envoie depuis une **identité iMessage distincte** (et garder vos Messages personnels séparés), utilisez un identifiant Apple dédié et un utilisateur macOS dédié.

1. Créez un identifiant Apple dédié (exemple : `my-cool-bot@icloud.com`).
   * Apple peut exiger un numéro de téléphone pour la vérification / l’authentification à deux facteurs (2FA).
2. Créez un utilisateur macOS (exemple : `openclawhome`) et connectez-vous à cette session.
3. Ouvrez Messages dans cette session utilisateur macOS et connectez-vous à iMessage en utilisant l’identifiant Apple du bot.
4. Activez la connexion à distance (Réglages Système → Général → Partage → Connexion à distance).
5. Installez `imsg` :
   * `brew install steipete/tap/imsg`
6. Configurez SSH de façon à ce que `ssh <bot-macos-user>@localhost true` fonctionne sans mot de passe.
7. Pointez `channels.imessage.accounts.bot.cliPath` vers un wrapper SSH qui exécute `imsg` en tant qu’utilisateur du bot.

Lors de la première exécution, l’envoi/la réception peut nécessiter des autorisations d’interface graphique (Automatisation + Accès complet au disque) dans *la session macOS du bot*. Si `imsg rpc` semble bloqué ou se termine, connectez-vous à cette session (le Partage d’écran est utile), exécutez une fois `imsg chats --limit 1` / `imsg send ...`, approuvez les invites, puis réessayez.

Exemple de wrapper (`chmod +x`). Remplacez `<bot-macos-user>` par votre nom d’utilisateur macOS réel :

```bash
#!/usr/bin/env bash
set -euo pipefail

# Exécutez d'abord une session SSH interactive pour accepter les clés d'hôte :
#   ssh <bot-macos-user>@localhost true
exec /usr/bin/ssh -o BatchMode=yes -o ConnectTimeout=5 -T <bot-macos-user>@localhost \
  "/usr/local/bin/imsg" "$@"
```

Exemple de config :

```json5
{
  channels: {
    imessage: {
      enabled: true,
      accounts: {
        bot: {
          name: "Bot",
          enabled: true,
          cliPath: "/path/to/imsg-bot",
          dbPath: "/Users/<bot-macos-user>/Library/Messages/chat.db"
        }
      }
    }
  }
}
```

Pour les configurations à compte unique, utilisez les options de niveau supérieur (`channels.imessage.cliPath`, `channels.imessage.dbPath`) plutôt que la map `accounts`.

<div id="remotessh-variant-optional">
  ### Variante distante/SSH (facultative)
</div>

Si vous voulez utiliser iMessage sur un autre Mac, définissez `channels.imessage.cliPath` sur un wrapper qui exécute `imsg` sur l’hôte macOS distant via SSH. OpenClaw n’a besoin que des entrées/sorties standard (stdio).

Exemple de wrapper :

```bash
#!/usr/bin/env bash
exec ssh -T gateway-host imsg "$@"
```

**Pièces jointes distantes :** Lorsque `cliPath` pointe vers un hôte distant via SSH, les chemins de pièces jointes dans la base de données Messages font référence à des fichiers situés sur la machine distante. OpenClaw peut les récupérer automatiquement via SCP en configurant `channels.imessage.remoteHost` :

```json5
{
  channels: {
    imessage: {
      cliPath: "~/imsg-ssh",                     // Wrapper SSH vers Mac distant
      remoteHost: "user@gateway-host",           // for SCP file transfer
      includeAttachments: true
    }
  }
}
```

Si `remoteHost` n&#39;est pas défini, OpenClaw tente de le déterminer automatiquement à partir de la commande SSH utilisée dans votre script wrapper. Une configuration explicite est recommandée pour plus de fiabilité.

<div id="remote-mac-via-tailscale-example">
  #### Mac à distance via Tailscale (exemple)
</div>

Si le Gateway s’exécute sur un hôte/VM Linux mais qu’iMessage doit tourner sur un Mac, Tailscale est le pont le plus simple : le Gateway communique avec le Mac sur le tailnet, exécute `imsg` via SSH et rapatrie les pièces jointes via SCP.

Architecture :

```
┌──────────────────────────────┐          SSH (imsg rpc)          ┌──────────────────────────┐
│ Hôte passerelle (Linux/VM)   │──────────────────────────────────▶│ Mac avec Messages + imsg │
│ - openclaw gateway           │          SCP (attachments)        │ - Messages signed in     │
│ - channels.imessage.cliPath  │◀──────────────────────────────────│ - Remote Login enabled   │
└──────────────────────────────┘                                   └──────────────────────────┘
              ▲
              │ Tailscale tailnet (hostname or 100.x.y.z)
              ▼
        user@gateway-host
```

Exemple concret de configuration (nom d’hôte Tailscale) :

```json5
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "~/.openclaw/scripts/imsg-ssh",
      remoteHost: "bot@mac-mini.tailnet-1234.ts.net",
      includeAttachments: true,
      dbPath: "/Users/bot/Library/Messages/chat.db"
    }
  }
}
```

Exemple de script wrapper (`~/.openclaw/scripts/imsg-ssh`) :

```bash
#!/usr/bin/env bash
exec ssh -T bot@mac-mini.tailnet-1234.ts.net imsg "$@"
```

Notes :

* Assurez-vous que le Mac est connecté à Messages et que la connexion à distance (« Remote Login ») est activée.
* Utilisez des clés SSH afin que `ssh bot@mac-mini.tailnet-1234.ts.net` fonctionne sans demande interactive.
* `remoteHost` doit correspondre à la cible SSH pour que `scp` puisse récupérer les pièces jointes.

Prise en charge de plusieurs comptes : utilisez `channels.imessage.accounts` avec une configuration par compte et un `name` optionnel. Voir [`gateway/configuration`](/fr/gateway/configuration#telegramaccounts--discordaccounts--slackaccounts--signalaccounts--imessageaccounts) pour le schéma commun. Ne validez pas `~/.openclaw/openclaw.json` (il contient souvent des jetons).

<div id="access-control-dms-groups">
  ## Contrôle d’accès (DM + groupes)
</div>

DM :

* Par défaut : `channels.imessage.dmPolicy = "pairing"`.
* Les expéditeurs inconnus reçoivent un code d’appairage ; les messages sont ignorés jusqu’à approbation (les codes expirent après 1 heure).
* Validez via :
  * `openclaw pairing list imessage`
  * `openclaw pairing approve imessage <CODE>`
* L’appairage est l’échange de jetons par défaut pour les DM iMessage. Détails : [Appairage](/fr/start/pairing)

Groupes :

* `channels.imessage.groupPolicy = open | allowlist | disabled`.
* `channels.imessage.groupAllowFrom` contrôle qui peut déclencher l’agent dans les groupes lorsque `allowlist` est défini.
* Le filtrage par mention utilise `agents.list[].groupChat.mentionPatterns` (ou `messages.groupChat.mentionPatterns`) car iMessage n’a pas de métadonnées de mention natives.
* Surcharge multi‑agent : définissez des modèles spécifiques à chaque agent dans `agents.list[].groupChat.mentionPatterns`.

<div id="how-it-works-behavior">
  ## Fonctionnement (comportement)
</div>

* `imsg` diffuse en continu les événements de message ; le Gateway les normalise dans l’enveloppe de canal commune.
* Les réponses sont toujours routées vers le même identifiant de discussion ou handle.

<div id="group-ish-threads-is_groupfalse">
  ## Fils « type groupe » (`is_group=false`)
</div>

Certains fils iMessage peuvent avoir plusieurs participants mais tout de même arriver avec `is_group=false` selon la façon dont Messages stocke l’identifiant du chat.

Si vous configurez explicitement un `chat_id` sous `channels.imessage.groups`, OpenClaw traite ce fil comme un « groupe » pour :

* l’isolation des sessions (clé de session distincte `agent:<agentId>:imessage:group:<chat_id>`)
* le comportement de liste d’autorisation et de filtrage par mention pour les groupes

Exemple :

```json5
{
  channels: {
    imessage: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15555550123"],
      groups: {
        "42": { "requireMention": false }
      }
    }
  }
}
```

C’est utile lorsque vous voulez une personnalité ou un modèle isolé pour un fil de discussion spécifique (voir [Routage multi‑agents](/fr/concepts/multi-agent)). Pour l’isolation du système de fichiers, voir [Sandboxing](/fr/gateway/sandboxing).

<div id="media-limits">
  ## Médias et limites
</div>

* Prise en compte facultative des pièces jointes via `channels.imessage.includeAttachments`.
* Limite de taille des médias via `channels.imessage.mediaMaxMb`.

<div id="limits">
  ## Limites
</div>

* Le texte sortant est découpé en fragments selon `channels.imessage.textChunkLimit` (valeur par défaut : 4000).
* Fragmentation facultative par sauts de ligne : définissez `channels.imessage.chunkMode="newline"` pour découper sur les lignes vides (frontières de paragraphe) avant le découpage par longueur.
* Les envois de médias sont plafonnés par `channels.imessage.mediaMaxMb` (valeur par défaut : 16).

<div id="addressing-delivery-targets">
  ## Adressage / cibles de livraison
</div>

Privilégiez `chat_id` pour un routage stable :

* `chat_id:123` (recommandé)
* `chat_guid:...`
* `chat_identifier:...`
* identifiants directs : `imessage:+1555` / `sms:+1555` / `user@example.com`

Lister les chats :

```
imsg chats --limit 20
```

<div id="configuration-reference-imessage">
  ## Référence de configuration (iMessage)
</div>

Configuration complète : [Configuration](/fr/gateway/configuration)

Options du fournisseur :

* `channels.imessage.enabled` : activer/désactiver le démarrage du canal.
* `channels.imessage.cliPath` : chemin vers `imsg`.
* `channels.imessage.dbPath` : chemin de la base de données Messages.
* `channels.imessage.remoteHost` : hôte SSH pour le transfert de pièces jointes via SCP lorsque `cliPath` pointe vers un Mac distant (par exemple, `user@gateway-host`). Détecté automatiquement à partir du wrapper SSH si non défini.
* `channels.imessage.service` : `imessage | sms | auto`.
* `channels.imessage.region` : région SMS.
* `channels.imessage.dmPolicy` : `pairing | allowlist | open | disabled` (par défaut : pairing).
* `channels.imessage.allowFrom` : liste d’autorisation pour les messages directs (handles, e‑mails, numéros E.164 ou `chat_id:*`). `open` nécessite `"*"`. iMessage n’a pas de noms d’utilisateur ; utilisez des handles ou des cibles de discussion.
* `channels.imessage.groupPolicy` : `open | allowlist | disabled` (par défaut : allowlist).
* `channels.imessage.groupAllowFrom` : liste d’autorisation des expéditeurs pour les groupes.
* `channels.imessage.historyLimit` / `channels.imessage.accounts.*.historyLimit` : nombre maximal de messages de groupe à inclure comme contexte (0 pour désactiver).
* `channels.imessage.dmHistoryLimit` : limite d’historique des messages directs en tours utilisateur. Surcharges par utilisateur : `channels.imessage.dms["<handle>"].historyLimit`.
* `channels.imessage.groups` : valeurs par défaut par groupe + liste d’autorisation (utilisez `"*"` pour les valeurs par défaut globales).
* `channels.imessage.includeAttachments` : ingérer les pièces jointes dans le contexte.
* `channels.imessage.mediaMaxMb` : limite de taille des médias entrants/sortants (Mo).
* `channels.imessage.textChunkLimit` : taille des blocs sortants (caractères).
* `channels.imessage.chunkMode` : `length` (par défaut) ou `newline` pour découper sur les lignes vides (limites de paragraphe) avant le découpage par longueur.

Options globales associées :

* `agents.list[].groupChat.mentionPatterns` (ou `messages.groupChat.mentionPatterns`).
* `messages.responsePrefix`.