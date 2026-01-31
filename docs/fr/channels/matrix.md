---
title: Matrix
summary: "Statut de la prise en charge de Matrix, fonctionnalités et configuration"
read_when:
  - Travail sur les fonctionnalités du canal Matrix
---

<div id="matrix-plugin">
  # Matrix (plugin)
</div>

Matrix est un protocole de messagerie ouvert et décentralisé. OpenClaw se connecte en tant qu’**utilisateur** Matrix
sur n’importe quel homeserver, donc vous devez disposer d’un compte Matrix pour le bot. Une fois connecté, vous pouvez envoyer des messages privés
au bot directement ou l’inviter dans des salons (des « groupes » Matrix). Beeper est aussi un client valide,
mais il nécessite que l’E2EE soit activé.

Statut : pris en charge via un plugin (@vector-im/matrix-bot-sdk). Messages directs, salons, fils de discussion, médias, réactions,
sondages (send + poll-start en tant que texte), localisation et E2EE (avec prise en charge de la couche crypto).

<div id="plugin-required">
  ## Plugin requis
</div>

Matrix est fourni en tant que plugin et n’est pas inclus dans l’installation de base.

Installez-le via la CLI (registre npm) :

```bash
openclaw plugins install @openclaw/matrix
```

Copie locale (lors de l’exécution depuis un dépôt Git) :

```bash
openclaw plugins install ./extensions/matrix
```

Si vous choisissez Matrix pendant la configuration/l’onboarding et qu’un checkout Git est détecté,
OpenClaw proposera automatiquement le chemin d’installation local.

Détails : [Plugins](/fr/plugin)

<div id="setup">
  ## Configuration
</div>

1. Installez le plugin Matrix :
   * Depuis npm : `openclaw plugins install @openclaw/matrix`
   * Depuis un dépôt local : `openclaw plugins install ./extensions/matrix`
2. Créez un compte Matrix sur un homeserver :
   * Consultez les options d’hébergement sur [https://matrix.org/ecosystem/hosting/](https://matrix.org/ecosystem/hosting/)
   * Ou hébergez-le vous-même.
3. Récupérez un jeton d’accès pour le compte bot :

   * Utilisez l’API de connexion Matrix avec `curl` sur votre homeserver :

   ```bash
   curl --request POST \
     --url https://matrix.example.org/_matrix/client/v3/login \
     --header 'Content-Type: application/json' \
     --data '{
     "type": "m.login.password",
     "identifier": {
       "type": "m.id.user",
       "user": "your-user-name"
     },
     "password": "your-password"
   }'
   ```

   * Remplacez `matrix.example.org` par l’URL de votre homeserver.
   * Ou définissez `channels.matrix.userId` + `channels.matrix.password` : OpenClaw appelle le même
     endpoint de connexion, stocke le jeton d’accès dans `~/.openclaw/credentials/matrix/credentials.json`,
     et le réutilise au prochain démarrage.
4. Configurez les identifiants :
   * Variables d’environnement : `MATRIX_HOMESERVER`, `MATRIX_ACCESS_TOKEN` (ou `MATRIX_USER_ID` + `MATRIX_PASSWORD`)
   * Ou configuration : `channels.matrix.*`
   * Si les deux sont définis, la configuration est prioritaire.
   * Avec un jeton d’accès : l’ID utilisateur est récupéré automatiquement via `/whoami`.
   * Lorsqu’il est défini, `channels.matrix.userId` doit être l’ID Matrix complet (exemple : `@bot:example.org`).
5. Redémarrez le Gateway (ou terminez l’onboarding).
6. Démarrez un DM avec le bot ou invitez-le dans un salon depuis n’importe quel client Matrix
   (Element, Beeper, etc. ; voir https://matrix.org/ecosystem/clients/). Beeper exige l’E2EE :
   définissez donc `channels.matrix.encryption: true` et vérifiez l’appareil.

Configuration minimale (jeton d’accès, ID utilisateur récupéré automatiquement) :

```json5
{
  channels: {
    matrix: {
      enabled: true,
      homeserver: "https://matrix.example.org",
      accessToken: "syt_***",
      dm: { policy: "appairage" }
    }
  }
}
```

Configuration E2EE (chiffrement de bout en bout activé) :

```json5
{
  channels: {
    matrix: {
      enabled: true,
      homeserver: "https://matrix.example.org",
      accessToken: "syt_***",
      encryption: true,
      dm: { policy: "pairing" }
    }
  }
}
```

<div id="encryption-e2ee">
  ## Chiffrement (E2EE)
</div>

Le chiffrement de bout en bout est **pris en charge** via le SDK crypto Rust.

Activez-le avec `channels.matrix.encryption: true` :

* Si le module crypto se charge, les salons chiffrés sont déchiffrés automatiquement.
* Les médias sortants sont chiffrés lors de l’envoi vers des salons chiffrés.
* Lors de la première connexion, OpenClaw demande la vérification de l’appareil à partir de vos autres sessions.
* Vérifiez l’appareil dans un autre client Matrix (Element, etc.) pour activer le partage de clés.
* Si le module crypto ne peut pas être chargé, l’E2EE est désactivé et les salons chiffrés ne seront pas déchiffrés ; OpenClaw enregistre un avertissement dans les logs.
* Si vous voyez des erreurs de module crypto manquant (par exemple, `@matrix-org/matrix-sdk-crypto-nodejs-*`), autorisez les scripts de build pour `@matrix-org/matrix-sdk-crypto-nodejs` et exécutez
  `pnpm rebuild @matrix-org/matrix-sdk-crypto-nodejs` ou récupérez le binaire avec
  `node node_modules/@matrix-org/matrix-sdk-crypto-nodejs/download-lib.js`.

L’état crypto est stocké par compte + jeton d’accès dans
`~/.openclaw/matrix/accounts/<account>/<homeserver>__<user>/<token-hash>/crypto/`
(base de données SQLite). L’état de synchronisation est stocké à côté dans `bot-storage.json`.
Si le jeton d’accès (appareil) change, un nouvel espace de stockage est créé et le bot doit être
de nouveau vérifié pour les salons chiffrés.

**Vérification de l’appareil :**
Lorsque l’E2EE est activé, le bot demande la vérification à partir de vos autres sessions au démarrage.
Ouvrez Element (ou un autre client) et approuvez la demande de vérification pour établir la confiance.
Une fois vérifié, le bot peut déchiffrer les messages dans les salons chiffrés.

<div id="routing-model">
  ## Modèle de routage
</div>

* Les réponses sont toujours renvoyées vers Matrix.
* Les DMs partagent la session principale de l&#39;agent ; les salons correspondent à des sessions de groupe.

<div id="access-control-dms">
  ## Contrôle d&#39;accès (DM)
</div>

* Par défaut : `channels.matrix.dm.policy = "pairing"`. Les expéditeurs inconnus reçoivent un code d&#39;appairage.
* Approuver via :
  * `openclaw pairing list matrix`
  * `openclaw pairing approve matrix <CODE>`
* DM publics : `channels.matrix.dm.policy="open"` plus `channels.matrix.dm.allowFrom=["*"]`.
* `channels.matrix.dm.allowFrom` accepte des identifiants utilisateur ou des noms d&#39;affichage. L&#39;assistant fait correspondre les noms d&#39;affichage aux identifiants utilisateur lorsque la recherche dans l&#39;annuaire est disponible.

<div id="rooms-groups">
  ## Salons (groupes)
</div>

* Valeur par défaut : `channels.matrix.groupPolicy = "allowlist"` (accès conditionné à une mention). Utilisez `channels.defaults.groupPolicy` pour remplacer la valeur par défaut lorsqu’elle n’est pas définie.
* Ajoutez des salons à la liste d’autorisation avec `channels.matrix.groups` (ID de salon, alias ou noms) :

```json5
{
  channels: {
    matrix: {
      groupPolicy: "allowlist",
      groups: {
        "!roomId:example.org": { allow: true },
        "#alias:example.org": { allow: true }
      },
      groupAllowFrom: ["@owner:example.org"]
    }
  }
}
```

* `requireMention: false` active la réponse automatique dans cette room.
* `groups."*"` permet de définir des valeurs par défaut pour le filtrage par mention dans l’ensemble des rooms.
* `groupAllowFrom` restreint quels expéditeurs peuvent déclencher le bot dans les rooms (optionnel).
* Des listes d’autorisation `users` par room peuvent restreindre davantage les expéditeurs à l’intérieur d’une room spécifique.
* L’assistant de configuration demande des listes d’autorisation de rooms (ID de rooms, alias ou noms) et résout les noms lorsque possible.
* Au démarrage, OpenClaw résout les noms de rooms/utilisateurs dans les listes d’autorisation en ID et consigne la correspondance dans les logs ; les entrées non résolues sont conservées telles que saisies.
* Les invitations à des rooms sont acceptées automatiquement par défaut ; contrôlez ce comportement avec `channels.matrix.autoJoin` et `channels.matrix.autoJoinAllowlist`.
* Pour n’**autoriser aucune room**, définissez `channels.matrix.groupPolicy: "disabled"` (ou conservez une liste d’autorisation vide).
* Clé héritée : `channels.matrix.rooms` (même structure que `groups`).

<div id="threads">
  ## Fils de discussion
</div>

* Les réponses organisées en fils de discussion sont prises en charge.
* `channels.matrix.threadReplies` détermine si les réponses restent dans des fils de discussion :
  * `off`, `inbound` (valeur par défaut), `always`
* `channels.matrix.replyToMode` contrôle les métadonnées de réponse lorsque vous ne répondez pas dans un fil de discussion :
  * `off` (valeur par défaut), `first`, `all`

<div id="capabilities">
  ## Fonctionnalités
</div>

| Fonctionnalité | Statut |
|----------------|--------|
| Messages directs | ✅ Pris en charge |
| Salons | ✅ Pris en charge |
| Fils de discussion | ✅ Pris en charge |
| Médias | ✅ Pris en charge |
| E2EE | ✅ Pris en charge (module crypto requis) |
| Réactions | ✅ Pris en charge (envoi/read via des outils) |
| Sondages | ✅ Envoi pris en charge ; les débuts de sondages entrants sont convertis en texte (réponses/fins ignorées) |
| Localisation | ✅ Pris en charge (URI geo ; altitude ignorée) |
| Commandes natives | ✅ Pris en charge |

<div id="configuration-reference-matrix">
  ## Référence de configuration (Matrix)
</div>

Configuration complète : [Configuration](/fr/gateway/configuration)

Options du fournisseur :

* `channels.matrix.enabled` : activer/désactiver le démarrage du canal.
* `channels.matrix.homeserver` : URL du homeserver.
* `channels.matrix.userId` : ID utilisateur Matrix (facultatif avec jeton d’accès).
* `channels.matrix.accessToken` : jeton d’accès.
* `channels.matrix.password` : mot de passe pour la connexion (jeton stocké).
* `channels.matrix.deviceName` : nom d’affichage de l’appareil.
* `channels.matrix.encryption` : activer l’E2EE (par défaut : false).
* `channels.matrix.initialSyncLimit` : limite de synchronisation initiale.
* `channels.matrix.threadReplies` : `off | inbound | always` (par défaut : inbound).
* `channels.matrix.textChunkLimit` : taille des blocs de texte sortants (caractères).
* `channels.matrix.chunkMode` : `length` (par défaut) ou `newline` pour couper sur les lignes vides (limites de paragraphe) avant le découpage par longueur.
* `channels.matrix.dm.policy` : `pairing | allowlist | open | disabled` (par défaut : pairing).
* `channels.matrix.dm.allowFrom` : liste d’autorisation pour les DMs (IDs utilisateur ou noms d’affichage). `open` nécessite `"*"`. L’assistant convertit les noms en IDs quand c’est possible.
* `channels.matrix.groupPolicy` : `allowlist | open | disabled` (par défaut : allowlist).
* `channels.matrix.groupAllowFrom` : émetteurs autorisés (allowlist) pour les messages de groupe.
* `channels.matrix.allowlistOnly` : imposer les règles de liste d’autorisation pour les DMs + salons.
* `channels.matrix.groups` : liste d’autorisation de groupes + mappage des paramètres par salon.
* `channels.matrix.rooms` : ancienne configuration de liste d’autorisation de groupes.
* `channels.matrix.replyToMode` : mode de réponse pour les fils/étiquettes.
* `channels.matrix.mediaMaxMb` : limite de média entrant/sortant (Mo).
* `channels.matrix.autoJoin` : gestion des invitations (`always | allowlist | off`, par défaut : always).
* `channels.matrix.autoJoinAllowlist` : IDs/alias de salons autorisés pour l’adhésion automatique.
* `channels.matrix.actions` : filtrage des outils par action (réactions/messages/épingles/memberInfo/channelInfo).