---
title: Signal
summary: "Prise en charge de Signal via signal-cli (JSON-RPC + SSE), configuration et modèle de numérotation"
read_when:
  - Mise en place de la prise en charge de Signal
  - Débogage de l’envoi et de la réception avec Signal
---

<div id="signal-signal-cli">
  # Signal (signal-cli)
</div>

Statut : intégration CLI externe. Gateway communique avec `signal-cli` via JSON-RPC HTTP et SSE.

<div id="quick-setup-beginner">
  ## Configuration rapide (débutant)
</div>

1. Utilisez un **numéro Signal distinct** pour le bot (recommandé).
2. Installez `signal-cli` (Java requis).
3. Associez l&#39;appareil du bot et démarrez le démon :
   * `signal-cli link -n "OpenClaw"`
4. Configurez OpenClaw et démarrez le Gateway.

Configuration minimale :

```json5
{
  channels: {
    signal: {
      enabled: true,
      account: "+15551234567",
      cliPath: "signal-cli",
      dmPolicy: "pairing",
      allowFrom: ["+15557654321"]
    }
  }
}
```

<div id="what-it-is">
  ## Ce que c’est
</div>

* Canal Signal via `signal-cli` (et non la bibliothèque intégrée libsignal).
* Routage déterministe : les réponses vont toujours à Signal.
* Les messages privés partagent la session principale de l’agent ; les groupes sont isolés (`agent:<agentId>:signal:group:<groupId>`).

<div id="config-writes">
  ## Écritures de configuration
</div>

Par défaut, Signal est autorisé à écrire des mises à jour de configuration déclenchées par `/config set|unset` (requiert `commands.config: true`).

Pour le désactiver :

```json5
{
  channels: { signal: { configWrites: false } }
}
```

<div id="the-number-model-important">
  ## Le modèle de numéro (important)
</div>

* Le Gateway se connecte à un **appareil Signal** (le compte `signal-cli`).
* Si vous exécutez le bot sur **votre compte Signal personnel**, il ignorera vos propres messages (protection anti-boucle).
* Pour le cas « j’envoie un message au bot et il répond », utilisez un **numéro de bot distinct**.

<div id="setup-fast-path">
  ## Configuration (parcours rapide)
</div>

1. Installez `signal-cli` (Java requis).
2. Associez un compte bot :
   * `signal-cli link -n "OpenClaw"` puis scannez le code QR dans Signal.
3. Configurez Signal et démarrez le service Gateway.

Exemple :

```json5
{
  channels: {
    signal: {
      enabled: true,
      account: "+15551234567",
      cliPath: "signal-cli",
      dmPolicy: "appairage",
      allowFrom: ["+15557654321"]
    }
  }
}
```

Prise en charge multi-compte : utilisez `channels.signal.accounts` avec une configuration propre à chaque compte et un `name` facultatif. Voir [`gateway/configuration`](/fr/gateway/configuration#telegramaccounts--discordaccounts--slackaccounts--signalaccounts--imessageaccounts) pour le schéma commun.

<div id="external-daemon-mode-httpurl">
  ## Mode démon externe (httpUrl)
</div>

Si vous voulez gérer `signal-cli` vous‑même (démarrages à froid lents de la JVM, initialisation de conteneurs ou CPU partagés), exécutez le démon séparément et pointez OpenClaw vers celui‑ci :

```json5
{
  channels: {
    signal: {
      httpUrl: "http://127.0.0.1:8080",
      autoStart: false
    }
  }
}
```

Cela contourne le lancement automatique et l’attente de démarrage à l’intérieur d’OpenClaw. Pour les démarrages lents avec lancement automatique, configurez `channels.signal.startupTimeoutMs`.

<div id="access-control-dms-groups">
  ## Contrôle d’accès (DM + groupes)
</div>

DM :

* Par défaut : `channels.signal.dmPolicy = "pairing"`.
* Les expéditeurs inconnus reçoivent un code d’appairage ; les messages sont ignorés jusqu’à approbation (les codes expirent au bout d’une heure).
* Approuver via :
  * `openclaw pairing list signal`
  * `openclaw pairing approve signal <CODE>`
* L’appairage est le mécanisme d’échange de jetons par défaut pour les DM Signal. Détails : [Pairing](/fr/start/pairing)
* Les expéditeurs identifiés uniquement par un UUID (provenant de `sourceUuid`) sont stockés sous la forme `uuid:<id>` dans `channels.signal.allowFrom`.

Groupes :

* `channels.signal.groupPolicy = open (autorise la réception de messages de n’importe quel utilisateur) | allowlist | disabled`.
* `channels.signal.groupAllowFrom` contrôle qui est autorisé à déclencher dans les groupes lorsque `allowlist` est activé.

<div id="how-it-works-behavior">
  ## Fonctionnement (comportement)
</div>

* `signal-cli` s’exécute en tant que démon ; le Gateway lit les événements via SSE.
* Les messages entrants sont normalisés dans une enveloppe de canal partagée.
* Les réponses sont toujours routées vers le même numéro ou le même groupe.

<div id="media-limits">
  ## Médias + limites
</div>

* Le texte sortant est découpé selon `channels.signal.textChunkLimit` (4000 par défaut).
* Fragmentation optionnelle par saut de ligne : définissez `channels.signal.chunkMode="newline"` pour découper aux lignes vides (frontières de paragraphe) avant la fragmentation par longueur.
* Pièces jointes prises en charge (base64 récupéré depuis `signal-cli`).
* Limite de média par défaut : `channels.signal.mediaMaxMb` (8 par défaut).
* Utilisez `channels.signal.ignoreAttachments` pour ignorer le téléchargement des médias.
* Le contexte d’historique de groupe utilise `channels.signal.historyLimit` (ou `channels.signal.accounts.*.historyLimit`), à défaut `messages.groupChat.historyLimit`. Définissez `0` pour désactiver (50 par défaut).

<div id="typing-read-receipts">
  ## Saisie + accusés de lecture
</div>

* **Indicateurs de saisie** : OpenClaw envoie des signaux de saisie via `signal-cli sendTyping` et les actualise tant qu&#39;une réponse est en cours de génération.
* **Accusés de lecture** : lorsque `channels.signal.sendReadReceipts` est à true, OpenClaw relaie les accusés de lecture pour les messages privés (DM) autorisés.
* signal-cli n&#39;expose pas les accusés de lecture pour les groupes.

<div id="reactions-message-tool">
  ## Réactions (outil de message)
</div>

* Utilisez `message action=react` avec `channel=signal`.
* Cibles : expéditeur E.164 ou UUID (utilisez `uuid:<id>` à partir de la sortie d’appairage ; l’UUID nu fonctionne aussi).
* `messageId` est l’horodatage Signal du message auquel vous réagissez.
* Les réactions de groupe nécessitent `targetAuthor` ou `targetAuthorUuid`.

Exemples :

```
message action=react channel=signal target=uuid:123e4567-e89b-12d3-a456-426614174000 messageId=1737630212345 emoji=🔥
message action=react channel=signal target=+15551234567 messageId=1737630212345 emoji=🔥 remove=true
message action=react channel=signal target=signal:group:<groupId> targetAuthor=uuid:<sender-uuid> messageId=1737630212345 emoji=✅
```

Config :

* `channels.signal.actions.reactions` : activer/désactiver les actions de réactions (valeur par défaut : true).
* `channels.signal.reactionLevel` : `off | ack | minimal | extensive`.
  * `off`/`ack` désactivent les réactions de l’agent (l’outil de messagerie `react` renverra une erreur).
  * `minimal`/`extensive` activent les réactions de l’agent et définissent le niveau de guidage.
* Surcharges par compte : `channels.signal.accounts.<id>.actions.reactions`, `channels.signal.accounts.<id>.reactionLevel`.

<div id="delivery-targets-clicron">
  ## Cibles de livraison (CLI/cron)
</div>

* Messages directs (DM) : `signal:+15551234567` (ou simple E.164).
* DM par UUID : `uuid:<id>` (ou UUID seul).
* Groupes : `signal:group:<groupId>`.
* Noms d’utilisateur : `username:<name>` (si pris en charge par votre compte Signal).

<div id="configuration-reference-signal">
  ## Référence de configuration (Signal)
</div>

Configuration complète : [Configuration](/fr/gateway/configuration)

Options du fournisseur :

* `channels.signal.enabled` : activer/désactiver le démarrage du canal.
* `channels.signal.account` : numéro E.164 pour le compte du bot.
* `channels.signal.cliPath` : chemin vers `signal-cli`.
* `channels.signal.httpUrl` : URL complète du démon (remplace host/port).
* `channels.signal.httpHost`, `channels.signal.httpPort` : adresse/port d’écoute du démon (par défaut 127.0.0.1:8080).
* `channels.signal.autoStart` : lancement automatique du démon (par défaut true si `httpUrl` n’est pas défini).
* `channels.signal.startupTimeoutMs` : délai d’attente au démarrage en ms (plafond 120000).
* `channels.signal.receiveMode` : `on-start | manual`.
* `channels.signal.ignoreAttachments` : ignorer les téléchargements de pièces jointes.
* `channels.signal.ignoreStories` : ignorer les stories provenant du démon.
* `channels.signal.sendReadReceipts` : transmettre les accusés de lecture.
* `channels.signal.dmPolicy` : `pairing | allowlist | open | disabled` (par défaut : appairage).
* `channels.signal.allowFrom` : liste d’autorisation pour les messages privés (E.164 ou `uuid:<id>`). `open` nécessite `"*"`. Signal n’a pas de noms d’utilisateur ; utilisez des identifiants téléphone/UUID.
* `channels.signal.groupPolicy` : `open | allowlist | disabled` (par défaut : liste d’autorisation).
* `channels.signal.groupAllowFrom` : liste d’autorisation des expéditeurs en groupe.
* `channels.signal.historyLimit` : nombre maximal de messages de groupe à inclure comme contexte (0 désactive).
* `channels.signal.dmHistoryLimit` : limite d’historique des messages privés en tours utilisateur. Surcharges par utilisateur : `channels.signal.dms["<phone_or_uuid>"].historyLimit`.
* `channels.signal.textChunkLimit` : taille des blocs sortants (caractères).
* `channels.signal.chunkMode` : `length` (par défaut) ou `newline` pour découper sur les lignes vides (limites de paragraphe) avant le découpage par longueur.
* `channels.signal.mediaMaxMb` : limite de taille des médias entrants/sortants (Mo).

Options globales associées :

* `agents.list[].groupChat.mentionPatterns` (Signal ne prend pas en charge les mentions natives).
* `messages.groupChat.mentionPatterns` (solution de repli globale).
* `messages.responsePrefix`.