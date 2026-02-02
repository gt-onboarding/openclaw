---
title: WhatsApp
summary: "Intégration de WhatsApp (canal web) : connexion, boîte de réception, réponses, médias et exploitation"
read_when:
  - Lorsque vous travaillez sur le comportement du canal WhatsApp/web ou le routage de la boîte de réception
---

<div id="whatsapp-web-channel">
  # WhatsApp (canal web)
</div>

Statut : WhatsApp Web via Baileys uniquement. Gateway gère les sessions.

<div id="quick-setup-beginner">
  ## Configuration rapide (débutant)
</div>

1. Utilisez un **numéro de téléphone distinct** si possible (recommandé).
2. Configurez WhatsApp dans `~/.openclaw/openclaw.json`.
3. Exécutez `openclaw channels login` pour scanner le code QR (Appareils connectés).
4. Démarrez le Gateway.

Configuration minimale :

```json5
{
  channels: {
    whatsapp: {
      dmPolicy: "allowlist",
      allowFrom: ["+15551234567"]
    }
  }
}
```

<div id="goals">
  ## Objectifs
</div>

* Plusieurs comptes WhatsApp (multi-compte) dans un seul processus Gateway.
* Routage déterministe : les réponses sont renvoyées vers WhatsApp, sans routage par modèle.
* Le modèle voit suffisamment de contexte pour comprendre les réponses citées.

<div id="config-writes">
  ## Écritures dans la configuration
</div>

Par défaut, WhatsApp est autorisé à écrire des mises à jour de configuration déclenchées par `/config set|unset` (nécessite `commands.config: true`).

Pour désactiver :

```json5
{
  channels: { whatsapp: { configWrites: false } }
}
```

<div id="architecture-who-owns-what">
  ## Architecture (qui gère quoi)
</div>

* **Gateway** gère le socket Baileys et la boucle de réception.
* La **CLI / application macOS** communiquent avec Gateway ; aucune utilisation directe de Baileys.
* Un **écouteur actif** est requis pour les envois sortants ; sinon, l&#39;envoi échoue immédiatement.

<div id="getting-a-phone-number-two-modes">
  ## Obtenir un numéro de téléphone (deux modes)
</div>

WhatsApp requiert un véritable numéro de téléphone mobile pour la vérification. Les numéros VoIP et virtuels sont généralement bloqués. Il existe deux méthodes prises en charge pour exécuter OpenClaw sur WhatsApp :

<div id="dedicated-number-recommended">
  ### Numéro dédié (recommandé)
</div>

Utilise un **numéro de téléphone distinct** pour OpenClaw. Meilleure UX, routage propre, pas de bizarreries de conversations avec toi‑même. Configuration idéale : **ancien téléphone Android de secours + eSIM**. Laisse‑le connecté au Wi‑Fi et branché sur secteur, puis associe‑le via un QR code.

**WhatsApp Business :** Tu peux utiliser WhatsApp Business sur le même appareil avec un numéro différent. Idéal pour garder ton WhatsApp personnel séparé — installe WhatsApp Business et enregistre le numéro dédié à OpenClaw dessus.

**Exemple de configuration (numéro dédié, liste d’autorisation pour un seul utilisateur) :**

```json5
{
  channels: {
    whatsapp: {
      dmPolicy: "allowlist",
      allowFrom: ["+15551234567"]
    }
  }
}
```

**Mode d’appairage (facultatif) :**
Si vous préférez l’appairage à une liste d’autorisation, définissez `channels.whatsapp.dmPolicy` sur `pairing`. Les expéditeurs inconnus reçoivent un code d’appairage ; approuvez avec :
`openclaw pairing approve whatsapp <code>`

<div id="personal-number-fallback">
  ### Numéro personnel (repli)
</div>

Solution de repli rapide : exécutez OpenClaw sur **votre propre numéro**. Envoyez-vous un message (fonction WhatsApp « Message yourself ») pour les tests afin d’éviter de spammer vos contacts. Attendez-vous à devoir lire les codes de vérification sur votre téléphone principal pendant la configuration et vos tests. **Vous devez activer le mode self-chat.**
Lorsque l’assistant de configuration demande votre numéro WhatsApp personnel, saisissez le numéro du téléphone depuis lequel vous enverrez les messages (le propriétaire/expéditeur), et non le numéro de l’assistant.

**Exemple de configuration (numéro personnel, self-chat) :**

```json
{
  "whatsapp": {
    "selfChatMode": true,
    "dmPolicy": "liste d'autorisation",
    "allowFrom": ["+15551234567"]
  }
}
```

Les réponses en auto-chat utilisent par défaut `[{identity.name}]` lorsqu&#39;il est défini (sinon `[openclaw]`)
si `messages.responsePrefix` n&#39;est pas renseigné. Définissez-le explicitement pour personnaliser ou désactiver
le préfixe (utilisez `""` pour le supprimer).

<div id="number-sourcing-tips">
  ### Conseils pour l&#39;obtention d&#39;un numéro
</div>

* **eSIM locale** auprès de l&#39;opérateur mobile de votre pays (le plus fiable)
  * Autriche : [hot.at](https://www.hot.at)
  * Royaume-Uni : [giffgaff](https://www.giffgaff.com) — SIM gratuite, sans contrat
* **Carte SIM prépayée** — peu coûteuse, n&#39;a besoin que de recevoir un SMS pour la vérification

**À éviter :** TextNow, Google Voice, la plupart des services de « SMS gratuits » — WhatsApp les bloque de manière agressive.

**Conseil :** Le numéro n&#39;a besoin de recevoir qu&#39;un seul SMS de vérification. Ensuite, les sessions WhatsApp Web restent actives via `creds.json`.

<div id="why-not-twilio">
  ## Pourquoi pas Twilio ?
</div>

* Les premières versions d’OpenClaw prenaient en charge l’intégration WhatsApp Business de Twilio.
* Les numéros WhatsApp Business sont mal adaptés à un assistant personnel.
* Meta impose une fenêtre de réponse de 24 heures ; si vous n’avez pas répondu au cours des 24 dernières heures, le numéro professionnel ne peut pas initier de nouveaux messages.
* Une utilisation à fort volume ou très « bavarde » déclenche des blocages agressifs, car les comptes professionnels ne sont pas conçus pour envoyer des dizaines de messages d’assistant personnel.
* Résultat : délivrabilité peu fiable et blocages fréquents, la prise en charge a donc été supprimée.

<div id="login-credentials">
  ## Connexion + identifiants
</div>

* Commande de connexion : `openclaw channels login` (QR via Appareils liés).
* Connexion multi‑compte : `openclaw channels login --account <id>` (`<id>` = `accountId`).
* Compte par défaut (quand `--account` est omis) : `default` si présent, sinon le premier identifiant de compte configuré (selon l’ordre de tri).
* Identifiants stockés dans `~/.openclaw/credentials/whatsapp/<accountId>/creds.json`.
* Copie de sauvegarde dans `creds.json.bak` (restaurée en cas de corruption).
* Compatibilité avec les anciennes versions : les anciennes installations stockaient les fichiers Baileys directement dans `~/.openclaw/credentials/`.
* Déconnexion : `openclaw channels logout` (ou `--account <id>`) supprime l’état d’authentification WhatsApp (mais conserve le fichier partagé `oauth.json`).
* Socket déconnectée ⇒ erreur invitant à relier le compte.

<div id="inbound-flow-dm-group">
  ## Flux entrant (DM + groupe)
</div>

* Les événements WhatsApp proviennent de `messages.upsert` (Baileys).
* Les écouteurs de la boîte de réception sont détachés à l&#39;arrêt afin d&#39;éviter l&#39;accumulation de gestionnaires d&#39;événements lors des tests/redémarrages.
* Les chats de statut/diffusion sont ignorés.
* Les chats directs utilisent E.164 ; les groupes utilisent un JID de groupe.
* **Politique de DM** : `channels.whatsapp.dmPolicy` contrôle l&#39;accès aux chats directs (valeur par défaut : `pairing`).
  * Appairage : les expéditeurs inconnus reçoivent un code d&#39;appairage (à approuver via `openclaw pairing approve whatsapp <code>` ; les codes expirent après 1 heure).
  * open : nécessite que `channels.whatsapp.allowFrom` inclue `"*"` (le réglage `open` autorise l&#39;acceptation de messages sans restriction depuis n&#39;importe quel utilisateur).
  * Votre numéro WhatsApp associé est implicitement approuvé, donc les messages que vous vous envoyez contournent les vérifications `channels.whatsapp.dmPolicy` et `channels.whatsapp.allowFrom`.

<div id="personal-number-mode-fallback">
  ### Mode numéro personnel (repli)
</div>

Si vous exécutez OpenClaw sur votre **numéro WhatsApp personnel**, activez `channels.whatsapp.selfChatMode` (voir l’exemple ci-dessus).

Comportement :

* Les DMs sortants ne déclenchent jamais de réponses d’appairage (évite de spammer vos contacts).
* Les expéditeurs inconnus des messages entrants suivent toujours `channels.whatsapp.dmPolicy`.
* Le mode self-chat (allowFrom inclut votre numéro) évite les accusés de lecture automatiques et ignore les JID de mention.
* Des accusés de lecture sont envoyés pour les DMs qui ne sont pas en mode self-chat.

<div id="read-receipts">
  ## Accusés de lecture
</div>

Par défaut, le Gateway marque les messages WhatsApp entrants comme lus (double coche bleue) lorsqu&#39;ils sont acceptés.

Désactiver globalement :

```json5
{
  channels: { whatsapp: { sendReadReceipts: false } }
}
```

Désactiver pour chaque compte :

```json5
{
  channels: {
    whatsapp: {
      accounts: {
        personal: { sendReadReceipts: false }
      }
    }
  }
}
```

Remarques :

* Le mode conversation automatique ignore toujours les accusés de lecture.

<div id="whatsapp-faq-sending-messages-pairing">
  ## FAQ WhatsApp : envoi de messages + appairage
</div>

**Est-ce qu’OpenClaw va envoyer des messages à des contacts aléatoires lorsque je relie WhatsApp ?**\
Non. La stratégie de DM par défaut est **appairage**, donc les expéditeurs inconnus ne reçoivent qu’un code d’appairage et leur message **n’est pas traité**. OpenClaw ne répond qu’aux conversations qu’il reçoit, ou aux envois que vous déclenchez explicitement (agent/CLI).

**Comment fonctionne l’appairage sur WhatsApp ?**\
L’appairage est un filtre de DM pour les expéditeurs inconnus :

* Le premier DM d’un nouvel expéditeur renvoie un code court (le message n’est pas traité).
* Approuvez avec : `openclaw pairing approve whatsapp <code>` (listez avec `openclaw pairing list whatsapp`).
* Les codes expirent après 1 heure ; les demandes en attente sont limitées à 3 par canal.

**Plusieurs personnes peuvent‑elles utiliser différentes instances OpenClaw sur un même numéro WhatsApp ?**\
Oui, en routant chaque expéditeur vers un agent différent via `bindings` (pair `kind: "dm"`, expéditeur E.164 comme `+15551234567`). Les réponses proviennent toujours du **même compte WhatsApp**, et les conversations directes sont regroupées dans la session principale de chaque agent, donc utilisez **un agent par personne**. Le contrôle d’accès aux DM (`dmPolicy`/`allowFrom`) est global par compte WhatsApp. Voir [Routage multi‑agents](/fr/concepts/multi-agent).

**Pourquoi demandez‑vous mon numéro de téléphone dans l’assistant ?**\
L’assistant l’utilise pour définir votre **liste d’autorisation/propriétaire** afin que vos propres DM soient autorisés. Il n’est pas utilisé pour l’envoi automatique. Si vous utilisez votre numéro WhatsApp personnel, utilisez ce même numéro et activez `channels.whatsapp.selfChatMode`.

<div id="message-normalization-what-the-model-sees">
  ## Normalisation des messages (ce que voit le modèle)
</div>

* `Body` est le corps du message actuel incluant l’enveloppe.
* Le contexte de réponse citée est **toujours ajouté** :
  ```
  [Replying to +1555 id:ABC123]
  <quoted text or <media:...>>
  [/Replying]
  ```
* Les métadonnées de réponse sont également définies :
  * `ReplyToId` = stanzaId
  * `ReplyToBody` = corps cité ou espace réservé pour le média
  * `ReplyToSender` = E.164 lorsqu’il est connu
* Les messages entrants ne contenant que des médias utilisent des espaces réservés :
  * `<media:image|video|audio|document|sticker>`

<div id="groups">
  ## Groupes
</div>

* Les groupes correspondent à des sessions `agent:<agentId>:whatsapp:group:<jid>`.
* Politique de groupe : `channels.whatsapp.groupPolicy = open|disabled|allowlist` (par défaut `allowlist`).
* Modes d’activation :
  * `mention` (par défaut) : nécessite une @mention ou une correspondance avec une expression régulière.
  * `always` : déclenchement systématique.
* `/activation mention|always` est réservé au propriétaire et doit être envoyé en tant que message autonome.
* Propriétaire = `channels.whatsapp.allowFrom` (ou propre E.164 si non défini).
* **Injection d’historique** (uniquement pour les messages en attente) :
  * Les messages récents *non traités* (50 par défaut) sont insérés sous :
    `[Chat messages since your last reply - for context]` (les messages déjà présents dans la session ne sont pas réinjectés)
  * Le message actuel sous :
    `[Current message - respond to this]`
  * Suffixe d’expéditeur ajouté : `[from: Name (+E164)]`
* Les métadonnées de groupe sont mises en cache pendant 5 min (sujet + participants).

<div id="reply-delivery-threading">
  ## Acheminement des réponses (fils de discussion)
</div>

* WhatsApp Web envoie des messages standard (sans fil de réponses citées dans la version actuelle du Gateway).
* Les balises de réponse sont ignorées sur ce canal.

<div id="acknowledgment-reactions-auto-react-on-receipt">
  ## Réactions d&#39;accusé de réception (réaction automatique à la réception)
</div>

WhatsApp peut automatiquement envoyer une réaction emoji aux messages entrants dès leur réception, avant que le bot ne génère une réponse. Cela fournit un retour instantané aux utilisateurs, indiquant que leur message a bien été reçu.

**Configuration :**

```json
{
  "whatsapp": {
    "ackReaction": {
      "emoji": "👀",
      "direct": true,
      "group": "mentions"
    }
  }
}
```

**Options :**

* `emoji` (string) : Emoji à utiliser pour l’accusé de réception (p. ex. &quot;👀&quot;, &quot;✅&quot;, &quot;📨&quot;). Vide ou omis = fonctionnalité désactivée.
* `direct` (boolean, default: `true`) : Envoyer des réactions dans les conversations directes/DM.
* `group` (string, default: `"mentions"`) : Comportement en groupe :
  * `"always"` : Réagir à tous les messages de groupe (même sans @mention)
  * `"mentions"` : Réagir uniquement lorsque le bot est @mentionné
  * `"never"` : Ne jamais réagir dans les groupes

**Remplacement par compte :**

```json
{
  "whatsapp": {
    "accounts": {
      "work": {
        "ackReaction": {
          "emoji": "✅",
          "direct": false,
          "group": "always"
        }
      }
    }
  }
}
```

**Notes sur le comportement :**

* Les réactions sont envoyées **immédiatement** à la réception du message, avant les indicateurs de saisie ou les réponses du bot.
* Dans les groupes avec `requireMention: false` (activation : toujours), `group: "mentions"` réagit à tous les messages (pas seulement aux @mentions).
* Fire-and-forget : les échecs de réaction sont consignés dans les logs mais n&#39;empêchent pas le bot de répondre.
* Le JID du participant est automatiquement inclus pour les réactions de groupe.
* WhatsApp ignore `messages.ackReaction` ; utilisez `channels.whatsapp.ackReaction` à la place.

<div id="agent-tool-reactions">
  ## Outil d’agent (réactions)
</div>

* Outil : `whatsapp` avec l’action `react` (`chatJid`, `messageId`, `emoji`, facultatif `remove`).
* Facultatif : `participant` (expéditeur dans un groupe), `fromMe` (réaction à votre propre message), `accountId` (multi‑compte).
* Sémantique de retrait des réactions : voir [/tools/reactions](/fr/tools/reactions).
* Contrôle d’accès à l’outil : `channels.whatsapp.actions.reactions` (par défaut : activé).

<div id="limits">
  ## Limites
</div>

* Le texte sortant est fragmenté selon `channels.whatsapp.textChunkLimit` (4000 par défaut).
* Fragmentation optionnelle par sauts de ligne : définissez `channels.whatsapp.chunkMode="newline"` pour découper aux lignes vides (limites de paragraphe) avant la fragmentation par longueur.
* Les enregistrements de médias entrants sont plafonnés par `channels.whatsapp.mediaMaxMb` (50 Mo par défaut).
* Les éléments multimédia sortants sont plafonnés par `agents.defaults.mediaMaxMb` (5 Mo par défaut).

<div id="outbound-send-text-media">
  ## Envoi sortant (texte + média)
</div>

* Utilise un écouteur web actif ; erreur si le Gateway n’est pas en cours d’exécution.
* Découpage du texte : 4k max par message (configurable via `channels.whatsapp.textChunkLimit`, optionnel `channels.whatsapp.chunkMode`).
* Médias :
  * Image/vidéo/audio/document pris en charge.
  * Audio envoyé en PTT ; `audio/ogg` =&gt; `audio/ogg; codecs=opus`.
  * Légende uniquement sur le premier élément média.
  * La récupération des médias prend en charge les URL HTTP(S) et les chemins locaux.
  * GIF animés : WhatsApp attend un MP4 avec `gifPlayback: true` pour une lecture en boucle intégrée.
    * CLI : `openclaw message send --media <mp4> --gif-playback`
    * Gateway : les paramètres de `send` incluent `gifPlayback: true`

<div id="voice-notes-ptt-audio">
  ## Messages vocaux (audio PTT)
</div>

WhatsApp envoie l&#39;audio sous forme de **messages vocaux** (bulle PTT).

* Résultats optimaux : OGG/Opus. OpenClaw réécrit `audio/ogg` en `audio/ogg; codecs=opus`.
* `[[audio_as_voice]]` est ignoré pour WhatsApp (l&#39;audio est déjà envoyé comme message vocal).

<div id="media-limits-optimization">
  ## Limites des médias + optimisation
</div>

* Limite sortante par défaut : 5 Mo (par élément média).
* Remplacement : `agents.defaults.mediaMaxMb`.
* Les images sont automatiquement optimisées en JPEG sous la limite (redimensionnement + ajustement de la qualité).
* Média trop volumineux ⇒ erreur ; la réponse média est remplacée par un avertissement textuel.

<div id="heartbeats">
  ## Signaux de vie
</div>

* **Signal de vie du Gateway** journalise l&#39;état de santé de la connexion (`web.heartbeatSeconds`, 60 s par défaut).
* **Signal de vie de l&#39;agent** peut être configuré par agent (`agents.list[].heartbeat`) ou globalement
  via `agents.defaults.heartbeat` (valeur de repli lorsqu&#39;aucune entrée par agent n&#39;est définie).
  * Utilise l&#39;invite de signal de vie configurée (par défaut : `Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.`) ainsi que le comportement de saut associé à `HEARTBEAT_OK`.
  * L’envoi se fait par défaut vers le dernier canal utilisé (ou la cible configurée).

<div id="reconnect-behavior">
  ## Comportement de reconnexion
</div>

* Stratégie de backoff (attente exponentielle) : `web.reconnect` :
  * `initialMs`, `maxMs`, `factor`, `jitter`, `maxAttempts`.
* Si `maxAttempts` est atteint, le monitoring web s’arrête (mode dégradé).
* Déconnexion =&gt; arrêt et nécessite un nouvel appairage.

<div id="config-quick-map">
  ## Vue d’ensemble rapide de la configuration
</div>

* `channels.whatsapp.dmPolicy` (politique de DM : pairing/liste d’autorisation/open/disabled).
* `channels.whatsapp.selfChatMode` (configuration sur le même téléphone ; le bot utilise votre numéro WhatsApp personnel).
* `channels.whatsapp.allowFrom` (liste d’autorisation de DM). WhatsApp utilise des numéros de téléphone au format E.164 (pas de noms d’utilisateur).
* `channels.whatsapp.mediaMaxMb` (limite de sauvegarde des médias entrants).
* `channels.whatsapp.ackReaction` (réaction automatique à la réception d’un message : `{emoji, direct, group}`).
* `channels.whatsapp.accounts.<accountId>.*` (paramètres par compte + `authDir` facultatif).
* `channels.whatsapp.accounts.<accountId>.mediaMaxMb` (plafond de médias entrants par compte).
* `channels.whatsapp.accounts.<accountId>.ackReaction` (surcharge de la réaction d’accusé de réception par compte).
* `channels.whatsapp.groupAllowFrom` (liste d’autorisation des expéditeurs de groupes).
* `channels.whatsapp.groupPolicy` (politique de groupe).
* `channels.whatsapp.historyLimit` / `channels.whatsapp.accounts.<accountId>.historyLimit` (contexte d’historique de groupe ; `0` désactive).
* `channels.whatsapp.dmHistoryLimit` (limite d’historique de DM en tours utilisateur). Surcharges par utilisateur : `channels.whatsapp.dms["<phone>"].historyLimit`.
* `channels.whatsapp.groups` (liste d’autorisation de groupes + valeurs par défaut de filtrage par mention ; utilisez `"*"` pour tout autoriser)
* `channels.whatsapp.actions.reactions` (contrôle des réactions d’outils WhatsApp).
* `agents.list[].groupChat.mentionPatterns` (ou `messages.groupChat.mentionPatterns`)
* `messages.groupChat.historyLimit`
* `channels.whatsapp.messagePrefix` (préfixe entrant ; par compte : `channels.whatsapp.accounts.<accountId>.messagePrefix` ; obsolète : `messages.messagePrefix`)
* `messages.responsePrefix` (préfixe sortant)
* `agents.defaults.mediaMaxMb`
* `agents.defaults.heartbeat.every`
* `agents.defaults.heartbeat.model` (surcharge facultative)
* `agents.defaults.heartbeat.target`
* `agents.defaults.heartbeat.to`
* `agents.defaults.heartbeat.session`
* `agents.list[].heartbeat.*` (surcharges par agent)
* `session.*` (portée, délai d’inactivité, stockage, mainKey)
* `web.enabled` (désactive le démarrage du canal lorsqu’il est défini sur false)
* `web.heartbeatSeconds`
* `web.reconnect.*`

<div id="logs-troubleshooting">
  ## Journaux + dépannage
</div>

* Sous-systèmes : `whatsapp/inbound`, `whatsapp/outbound`, `web-heartbeat`, `web-reconnect`.
* Fichier journal : `/tmp/openclaw/openclaw-YYYY-MM-DD.log` (configurable).
* Guide de dépannage : [Dépannage du Gateway](/fr/gateway/troubleshooting).

<div id="troubleshooting-quick">
  ## Dépannage (rapide)
</div>

**Non lié / Connexion via QR requise**

* Symptôme : `channels status` affiche `linked: false` ou affiche l’avertissement « Not linked ».
* Correctif : exécute `openclaw channels login` sur l’hôte du Gateway et scanne le QR (WhatsApp → Settings → Linked Devices).

**Lié mais déconnecté / boucle de reconnexion**

* Symptôme : `channels status` affiche `running, disconnected` ou affiche l’avertissement « Linked but disconnected ».
* Correctif : lance `openclaw doctor` (ou redémarre le Gateway). Si le problème persiste, relie via `channels login` et inspecte `openclaw logs --follow`.

**Runtime Bun**

* Bun est **fortement déconseillé**. WhatsApp (Baileys) et Telegram sont instables avec Bun.
  Exécute le Gateway avec **Node**. (Voir la note sur le runtime dans Getting Started.)