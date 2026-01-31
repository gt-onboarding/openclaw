---
title: Msteams
summary: "État de prise en charge du bot Microsoft Teams, fonctionnalités et configuration"
read_when:
  - Lorsque vous travaillez sur les fonctionnalités du canal MS Teams
---

<div id="microsoft-teams-plugin">
  # Microsoft Teams (plugin)
</div>

> « Vous qui entrez, abandonnez tout espoir. »

Mise à jour : 2026-01-21

Statut : le texte et les pièces jointes dans les messages privés (DM) sont pris en charge ; l’envoi de fichiers dans les canaux/groupes nécessite `sharePointSiteId` + des autorisations Graph (voir [Envoi de fichiers dans les discussions de groupe](#sending-files-in-group-chats)). Les sondages sont envoyés via des Adaptive Cards.

<div id="plugin-required">
  ## Plugin nécessaire
</div>

Microsoft Teams est fourni sous forme de plugin et n&#39;est pas inclus dans l&#39;installation de base.

**Changement incompatible (2026.1.15)\u00A0:** MS Teams a été retiré du cœur. Si vous l&#39;utilisez, vous devez installer le plugin.

Explication\u00A0: cela allège les installations de base et permet de mettre à jour les dépendances MS Teams de façon indépendante.

Installation via la CLI (registre npm)\u00A0:

```bash
openclaw plugins install @openclaw/msteams
```

Copie locale (lors de l&#39;exécution depuis un dépôt Git) :

```bash
openclaw plugins install ./extensions/msteams
```

Si vous sélectionnez Teams lors de la configuration / de l’onboarding et qu’un checkout Git est détecté,
OpenClaw proposera automatiquement le chemin d’installation local.

Détails : [Plugins](/fr/plugin)

<div id="quick-setup-beginner">
  ## Configuration rapide (débutant)
</div>

1. Installez le plugin Microsoft Teams.
2. Créez un **Azure Bot** (ID d&#39;application + secret client + ID de locataire).
3. Configurez OpenClaw avec ces identifiants.
4. Exposez `/api/messages` (port 3978 par défaut) via une URL publique ou un tunnel.
5. Installez le package d&#39;application Teams et démarrez le Gateway.

Configuration minimale :

```json5
{
  channels: {
    msteams: {
      enabled: true,
      appId: "<APP_ID>",
      appPassword: "<APP_PASSWORD>",
      tenantId: "<TENANT_ID>",
      webhook: { port: 3978, path: "/api/messages" }
    }
  }
}
```

Remarque : les discussions de groupe sont bloquées par défaut (`channels.msteams.groupPolicy: "allowlist"`). Pour autoriser les réponses dans les discussions de groupe, définissez `channels.msteams.groupAllowFrom` (ou utilisez `groupPolicy: "open"` pour autoriser tout membre, sous réserve d’une @mention obligatoire).

<div id="goals">
  ## Objectifs
</div>

* Parler à OpenClaw via des messages privés (DM) Teams, des discussions de groupe ou des canaux.
* Maintenir un routage déterministe : les réponses retournent toujours sur le canal d’où elles proviennent.
* Adopter par défaut un comportement de canal sécurisé (mentions requises sauf configuration contraire).

<div id="config-writes">
  ## Écritures dans la configuration
</div>

Par défaut, Microsoft Teams est autorisé à effectuer des mises à jour de configuration déclenchées par `/config set|unset` (nécessite `commands.config: true`).

Pour le désactiver :

```json5
{
  channels: { msteams: { configWrites: false } }
}
```

<div id="access-control-dms-groups">
  ## Contrôle d’accès (MP + groupes)
</div>

**Accès en MP**

* Par défaut : `channels.msteams.dmPolicy = "pairing"`. Les expéditeurs inconnus sont ignorés jusqu’à approbation.
* `channels.msteams.allowFrom` accepte des ID d’objet AAD, des UPN ou des noms d’affichage. L’assistant résout les noms en ID via Microsoft Graph lorsque les informations d’identification le permettent.

**Accès en groupe**

* Par défaut : `channels.msteams.groupPolicy = "allowlist"` (bloqué sauf si vous ajoutez `groupAllowFrom`). Utilisez `channels.defaults.groupPolicy` pour remplacer la valeur par défaut lorsqu’elle n’est pas définie.
* `channels.msteams.groupAllowFrom` contrôle quels expéditeurs peuvent déclencher en discussions/canaux de groupe (se replie sur `channels.msteams.allowFrom`).
* Définissez `groupPolicy: "open"` pour autoriser n’importe quel membre (la valeur de stratégie `open` permet l’acceptation des messages sans restriction depuis n’importe quel membre, tout en restant limitée par la nécessité d’une mention par défaut).
* Pour n’autoriser **aucun canal**, définissez `channels.msteams.groupPolicy: "disabled"`.

Exemple :

```json5
{
  channels: {
    msteams: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["user@org.com"]
    }
  }
}
```

**Teams + liste d’autorisation de canaux**

* Limitez la portée des réponses de groupe/canal en listant les équipes et les canaux sous `channels.msteams.teams`.
* Les clés peuvent être des ID ou des noms d’équipe ; les clés de canal peuvent être des ID de conversation ou des noms.
* Lorsque `groupPolicy="allowlist"` et qu’une liste d’autorisation d’équipes est présente, seules les équipes/canaux listés sont acceptés (restreint par mention).
* L’assistant de configuration accepte des entrées `Team/Channel` et les enregistre pour vous.
* Au démarrage, OpenClaw résout les noms d’équipe/canal et de liste d’autorisation d’utilisateurs en ID (lorsque les autorisations Graph le permettent)
  et consigne la correspondance dans les journaux ; les entrées non résolues sont conservées telles que saisies.

Exemple :

```json5
{
  channels: {
    msteams: {
      groupPolicy: "allowlist",
      teams: {
        "My Team": {
          channels: {
            "General": { requireMention: true }
          }
        }
      }
    }
  }
}
```

<div id="how-it-works">
  ## Comment ça fonctionne
</div>

1. Installe le plugin Microsoft Teams.
2. Crée un **bot Azure** (ID d’application + secret + ID de tenant).
3. Crée un **package d’application Teams** qui référence le bot et inclut les autorisations RSC ci‑dessous.
4. Charge/installe l’application Teams dans une équipe (ou dans une portée personnelle pour les messages privés).
5. Configure `msteams` dans `~/.openclaw/openclaw.json` (ou via des variables d’environnement) et démarre le Gateway OpenClaw.
6. Le Gateway OpenClaw écoute le trafic webhook du Bot Framework sur `/api/messages` par défaut.

<div id="azure-bot-setup-prerequisites">
  ## Configuration du bot Azure (prérequis)
</div>

Avant de configurer OpenClaw, vous devez créer une ressource de bot Azure.

<div id="step-1-create-azure-bot">
  ### Étape 1 : Créer un bot Azure
</div>

1. Accédez à [Create Azure Bot](https://portal.azure.com/#create/Microsoft.AzureBot)
2. Renseignez l’onglet **Basics** :

   | Champ | Valeur |
   |-------|--------|
   | **Bot handle** | Le nom de votre bot, par exemple `openclaw-msteams` (doit être unique) |
   | **Subscription** | Sélectionnez votre abonnement Azure |
   | **Resource group** | Créez-en un nouveau ou utilisez un existant |
   | **Pricing tier** | **Free** pour le développement et les tests |
   | **Type of App** | **Single Tenant** (recommandé — voir la note ci-dessous) |
   | **Creation type** | **Create new Microsoft App ID** |

> **Avis d’obsolescence :** la création de nouveaux bots multi‑locataires n’est plus prise en charge après le 31 juillet 2025. Utilisez **Single Tenant** pour les nouveaux bots.

3. Cliquez sur **Review + create** → **Create** (attendez environ 1 à 2 minutes)

<div id="step-2-get-credentials">
  ### Étape 2 : Récupérer les identifiants
</div>

1. Accédez à votre ressource Azure Bot → **Configuration**
2. Copiez **Microsoft App ID** → c’est votre `appId`
3. Cliquez sur **Manage Password** → accédez à l’App Registration
4. Dans **Certificates &amp; secrets** → **New client secret** → copiez la **Value** → c’est votre `appPassword`
5. Allez dans **Overview** → copiez **Directory (tenant) ID** → c’est votre `tenantId`

<div id="step-3-configure-messaging-endpoint">
  ### Étape 3 : Configurer le point de terminaison de messagerie
</div>

1. Dans Azure Bot → **Configuration**
2. Définissez **Messaging endpoint** sur votre URL de webhook :
   * Production : `https://your-domain.com/api/messages`
   * Développement local : utilisez un tunnel (voir [Développement local](#local-development-tunneling) ci-dessous)

<div id="step-4-enable-teams-channel">
  ### Étape 4 : Activer le canal Teams
</div>

1. Dans Azure Bot → **Channels**
2. Cliquez sur **Microsoft Teams** → Configure → Save
3. Acceptez les conditions d’utilisation

<div id="local-development-tunneling">
  ## Développement local (via un tunnel)
</div>

Teams ne peut pas accéder à `localhost`. Utilisez un tunnel pour le développement local :

**Option A : ngrok**

```bash
ngrok http 3978
# Copiez l'URL https, par ex. https://abc123.ngrok.io
# Définissez le point de terminaison de messagerie sur : https://abc123.ngrok.io/api/messages
```

**Option B : Tailscale Funnel**

```bash
tailscale funnel 3978
# Utilisez votre URL Tailscale Funnel comme point de terminaison de messagerie
```

<div id="teams-developer-portal-alternative">
  ## Teams Developer Portal (alternative)
</div>

Au lieu de créer manuellement un fichier manifeste au format ZIP, vous pouvez utiliser le [Teams Developer Portal](https://dev.teams.microsoft.com/apps) :

1. Cliquez sur **+ New app**
2. Renseignez les informations de base (nom, description, informations du développeur)
3. Allez dans **App features** → **Bot**
4. Sélectionnez **Enter a bot ID manually** et collez votre Azure Bot App ID
5. Vérifiez les portées : **Personal**, **Team**, **Group Chat**
6. Cliquez sur **Distribute** → **Download app package**
7. Dans Teams : **Apps** → **Manage your apps** → **Upload a custom app** → sélectionnez le ZIP

Cette méthode est souvent plus simple que de modifier manuellement les manifestes JSON.

<div id="testing-the-bot">
  ## Tester le bot
</div>

**Option A : Azure Web Chat (vérifier d&#39;abord le webhook)**

1. Dans Azure Portal → votre ressource Azure Bot → **Test in Web Chat**
2. Envoyez un message : vous devriez voir une réponse
3. Cela confirme que votre endpoint de webhook fonctionne avant la configuration de Teams

**Option B : Teams (après l&#39;installation de l&#39;application)**

1. Installez l&#39;application Teams (sideload ou catalogue de l&#39;organisation)
2. Trouvez le bot dans Teams et envoyez-lui un message privé (DM)
3. Vérifiez les logs du Gateway pour l&#39;activité entrante

<div id="setup-minimal-text-only">
  ## Configuration (texte minimal uniquement)
</div>

1. **Installer le plugin Microsoft Teams**
   * Depuis npm : `openclaw plugins install @openclaw/msteams`
   * Depuis un checkout local : `openclaw plugins install ./extensions/msteams`

2. **Enregistrement du bot**
   * Créez un Azure Bot (voir ci‑dessus) et notez :
     * App ID
     * Client secret (mot de passe d’application)
     * Tenant ID (single‑tenant)

3. **Manifeste d’application Teams**
   * Inclure une entrée `bot` avec `botId = <App ID>`.
   * Scopes : `personal`, `team`, `groupChat`.
   * `supportsFiles: true` (requis pour la gestion des fichiers pour la portée `personal`).
   * Ajouter les permissions RSC (ci‑dessous).
   * Créer des icônes : `outline.png` (32x32) et `color.png` (192x192).
   * Compresser ces trois fichiers ensemble : `manifest.json`, `outline.png`, `color.png`.

4. **Configurer OpenClaw**

   ```json
   {
     "msteams": {
       "enabled": true,
       "appId": "<APP_ID>",
       "appPassword": "<APP_PASSWORD>",
       "tenantId": "<TENANT_ID>",
       "webhook": { "port": 3978, "path": "/api/messages" }
     }
   }
   ```

   Vous pouvez aussi utiliser des variables d’environnement au lieu des clés de configuration :

   * `MSTEAMS_APP_ID`
   * `MSTEAMS_APP_PASSWORD`
   * `MSTEAMS_TENANT_ID`

5. **Point de terminaison du bot**
   * Configurez l’Azure Bot Messaging Endpoint sur :
     * `https://<host>:3978/api/messages` (ou le chemin/port de votre choix).

6. **Exécuter le Gateway**
   * Le canal Teams démarre automatiquement lorsque le plugin est installé et que la configuration `msteams` existe avec les identifiants.

<div id="history-context">
  ## Contexte de l’historique
</div>

* `channels.msteams.historyLimit` détermine combien de messages récents du canal/groupe sont inclus dans le prompt.
* À défaut, la valeur de `messages.groupChat.historyLimit` est utilisée. Définissez `0` pour désactiver (50 par défaut).
* L’historique des DM peut être limité avec `channels.msteams.dmHistoryLimit` (tours utilisateur). Remplacements par utilisateur : `channels.msteams.dms["&lt;user_id&gt;"].historyLimit`.

<div id="current-teams-rsc-permissions-manifest">
  ## Autorisations RSC Teams actuelles (manifeste)
</div>

Voici les **autorisations resourceSpecific existantes** dans notre manifeste d’application Teams. Elles ne s’appliquent qu’à l’intérieur de l’équipe/chat où l’application est installée.

**Pour les canaux (portée d’équipe) :**

* `ChannelMessage.Read.Group` (Application) - recevoir tous les messages de canal sans @mention
* `ChannelMessage.Send.Group` (Application)
* `Member.Read.Group` (Application)
* `Owner.Read.Group` (Application)
* `ChannelSettings.Read.Group` (Application)
* `TeamMember.Read.Group` (Application)
* `TeamSettings.Read.Group` (Application)

**Pour les discussions de groupe :**

* `ChatMessage.Read.Chat` (Application) - recevoir tous les messages de discussion de groupe sans @mention

<div id="example-teams-manifest-redacted">
  ## Exemple de manifeste Teams (expurgé)
</div>

Exemple minimal valide avec les champs requis. Remplacez les ID et les URL.

```json
{
  "$schema": "https://developer.microsoft.com/en-us/json-schemas/teams/v1.23/MicrosoftTeams.schema.json",
  "manifestVersion": "1.23",
  "version": "1.0.0",
  "id": "00000000-0000-0000-0000-000000000000",
  "name": { "short": "OpenClaw" },
  "developer": {
    "name": "Your Org",
    "websiteUrl": "https://example.com",
    "privacyUrl": "https://example.com/privacy",
    "termsOfUseUrl": "https://example.com/terms"
  },
  "description": { "short": "OpenClaw in Teams", "full": "OpenClaw in Teams" },
  "icons": { "outline": "outline.png", "color": "color.png" },
  "accentColor": "#5B6DEF",
  "bots": [
    {
      "botId": "11111111-1111-1111-1111-111111111111",
      "scopes": ["personal", "team", "groupChat"],
      "isNotificationOnly": false,
      "supportsCalling": false,
      "supportsVideo": false,
      "supportsFiles": true
    }
  ],
  "webApplicationInfo": {
    "id": "11111111-1111-1111-1111-111111111111"
  },
  "authorization": {
    "permissions": {
      "resourceSpecific": [
        { "name": "ChannelMessage.Read.Group", "type": "Application" },
        { "name": "ChannelMessage.Send.Group", "type": "Application" },
        { "name": "Member.Read.Group", "type": "Application" },
        { "name": "Owner.Read.Group", "type": "Application" },
        { "name": "ChannelSettings.Read.Group", "type": "Application" },
        { "name": "TeamMember.Read.Group", "type": "Application" },
        { "name": "TeamSettings.Read.Group", "type": "Application" },
        { "name": "ChatMessage.Read.Chat", "type": "Application" }
      ]
    }
  }
}
```

<div id="manifest-caveats-must-have-fields">
  ### Avertissements concernant le manifeste (champs obligatoires)
</div>

* `bots[].botId` **doit impérativement** correspondre à l’ID d’application Azure Bot.
* `webApplicationInfo.id` **doit impérativement** correspondre à l’ID d’application Azure Bot.
* `bots[].scopes` doit inclure les surfaces que vous prévoyez d’utiliser (`personal`, `team`, `groupChat`).
* `bots[].supportsFiles: true` est requis pour la gestion des fichiers dans la portée personnelle.
* `authorization.permissions.resourceSpecific` doit inclure les autorisations `read`/`send` sur les canaux si vous souhaitez du trafic de canal.

<div id="updating-an-existing-app">
  ### Mettre à jour une application existante
</div>

Pour mettre à jour une application Teams déjà installée (par exemple pour ajouter des autorisations RSC) :

1. Mettez à jour votre `manifest.json` avec les nouveaux paramètres
2. **Incrémentez le champ `version`** (par exemple `1.0.0` → `1.1.0`)
3. **Recréez l’archive zip** du manifeste avec les icônes (`manifest.json`, `outline.png`, `color.png`)
4. Chargez le nouveau fichier zip :
   * **Option A (Teams Admin Center) :** Teams Admin Center → Teams apps → Manage apps → trouvez votre application → Upload new version
   * **Option B (sideload) :** Dans Teams → Apps → Manage your apps → Upload a custom app
5. **Pour les canaux d’équipe :** réinstallez l’application dans chaque équipe pour que les nouvelles autorisations prennent effet
6. **Quittez complètement puis relancez Teams** (ne vous contentez pas de fermer la fenêtre) afin de vider les métadonnées d’application mises en cache

<div id="capabilities-rsc-only-vs-graph">
  ## Fonctionnalités : RSC uniquement vs Graph
</div>

<div id="with-teams-rsc-only-app-installed-no-graph-api-permissions">
  ### Avec **Teams RSC uniquement** (application installée, sans autorisations Graph API)
</div>

Fonctionne :

* Lecture du contenu **texte** des messages de canal.
* Envoi du contenu **texte** des messages de canal.
* Réception des pièces jointes de fichiers **personnels (DM)**.

Ne fonctionne PAS :

* **Contenu d’images ou de fichiers** de canal/groupe (la charge utile ne contient qu’un extrait HTML).
* Téléchargement des pièces jointes stockées dans SharePoint/OneDrive.
* Lecture de l’historique des messages (au-delà de l’événement webhook en temps réel).

<div id="with-teams-rsc-microsoft-graph-application-permissions">
  ### Avec **Teams RSC + autorisations d’application Microsoft Graph**
</div>

Permet :

* De télécharger les contenus hébergés (images collées dans les messages).
* De télécharger les pièces jointes de fichiers stockés dans SharePoint/OneDrive.
* De lire l’historique des messages de canal/de discussion via Microsoft Graph.

<div id="rsc-vs-graph-api">
  ### RSC vs Graph API
</div>

| Fonctionnalité | Permissions RSC | Graph API |
|----------------|-----------------|-----------|
| **Messages en temps réel** | Oui (via webhook) | Non (interrogation périodique uniquement) |
| **Messages historiques** | Non | Oui (peut interroger l’historique) |
| **Complexité de configuration** | Manifeste d’application uniquement | Nécessite un consentement administrateur + un flux de jetons |
| **Fonctionne hors ligne** | Non (doit être en cours d’exécution) | Oui (peut interroger à tout moment) |

**Conclusion :** RSC est conçu pour l’écoute en temps réel ; Graph API est conçu pour l’accès à l’historique. Pour rattraper les messages manqués pendant que vous étiez hors ligne, vous avez besoin de Graph API avec `ChannelMessage.Read.All` (nécessite un consentement administrateur).

<div id="graph-enabled-media-history-required-for-channels">
  ## Médias et historique via Graph (obligatoire pour les canaux)
</div>

Si vous avez besoin d’images/fichiers dans les **canaux** ou que vous voulez récupérer l’**historique des messages**, vous devez activer les autorisations Microsoft Graph et accorder le consentement d’administrateur.

1. Dans l’**App Registration** Entra ID (Azure AD), ajoutez les **Application permissions** Microsoft Graph :
   * `ChannelMessage.Read.All` (pièces jointes de canaux + historique)
   * `Chat.Read.All` ou `ChatMessage.Read.All` (conversations de groupe)
2. **Accordez le consentement d’administrateur** pour le locataire.
3. Incrémentez la **version du manifeste** de l’application Teams, téléversez-la de nouveau et **réinstallez l’application dans Teams**.
4. **Quittez complètement puis relancez Teams** pour vider les métadonnées d’application mises en cache.

<div id="known-limitations">
  ## Limites connues
</div>

<div id="webhook-timeouts">
  ### Dépassements de délai des webhooks
</div>

Teams envoie les messages via un webhook HTTP. Si le traitement prend trop de temps (par exemple, en cas de réponses LLM lentes), vous pouvez rencontrer :

* Des dépassements de délai au niveau du Gateway
* Teams qui renvoie le message (ce qui peut entraîner des doublons)
* Des réponses perdues

OpenClaw gère cela en répondant rapidement et en envoyant les réponses de manière proactive, mais des réponses très lentes peuvent malgré tout provoquer des problèmes.

<div id="formatting">
  ### Mise en forme
</div>

Le Markdown de Teams est plus limité que celui de Slack ou Discord :

* La mise en forme de base fonctionne : **gras**, *italique*, `code`, liens
* Le Markdown complexe (tableaux, listes imbriquées) peut ne pas s’afficher correctement
* Les Adaptive Cards sont prises en charge pour les sondages et l’envoi de cartes personnalisées (voir ci-dessous)

<div id="configuration">
  ## Configuration
</div>

Paramètres principaux (voir `/gateway/configuration` pour les modèles de configuration partagés entre canaux) :

* `channels.msteams.enabled` : activer/désactiver le canal.
* `channels.msteams.appId`, `channels.msteams.appPassword`, `channels.msteams.tenantId` : identifiants du bot.
* `channels.msteams.webhook.port` (valeur par défaut : `3978`)
* `channels.msteams.webhook.path` (valeur par défaut : `/api/messages`)
* `channels.msteams.dmPolicy` : `pairing | allowlist | open | disabled` (valeur par défaut : `pairing`)
* `channels.msteams.allowFrom` : liste d’autorisation pour les messages privés (DM) (ID d’objets AAD, UPN ou noms d’affichage). L’assistant de configuration résout les noms en ID pendant la configuration lorsque l’accès Graph est disponible.
* `channels.msteams.textChunkLimit` : taille des blocs de texte sortants.
* `channels.msteams.chunkMode` : `length` (par défaut) ou `newline` pour découper sur les lignes vides (limites de paragraphes) avant le découpage par longueur.
* `channels.msteams.mediaAllowHosts` : liste d’autorisation pour les hôtes des pièces jointes entrantes (par défaut, les domaines Microsoft/Teams).
* `channels.msteams.requireMention` : exiger une @mention dans les canaux/groupes (true par défaut).
* `channels.msteams.replyStyle` : `thread | top-level` (voir [Style de réponse](#reply-style-threads-vs-posts)).
* `channels.msteams.teams.<teamId>.replyStyle` : surdéfinition par équipe.
* `channels.msteams.teams.<teamId>.requireMention` : surdéfinition par équipe.
* `channels.msteams.teams.<teamId>.tools` : surdéfinitions par défaut des stratégies d’outils par équipe (`allow`/`deny`/`alsoAllow`) utilisées lorsqu’il manque une surdéfinition au niveau du canal.
* `channels.msteams.teams.<teamId>.toolsBySender` : surdéfinitions par défaut des stratégies d’outils par équipe et par expéditeur (prise en charge du caractère générique `"*"`).
* `channels.msteams.teams.<teamId>.channels.<conversationId>.replyStyle` : surdéfinition par canal.
* `channels.msteams.teams.<teamId>.channels.<conversationId>.requireMention` : surdéfinition par canal.
* `channels.msteams.teams.<teamId>.channels.<conversationId>.tools` : surdéfinitions de la stratégie d’outils par canal (`allow`/`deny`/`alsoAllow`).
* `channels.msteams.teams.<teamId>.channels.<conversationId>.toolsBySender` : surdéfinitions de la stratégie d’outils par canal et par expéditeur (prise en charge du caractère générique `"*"`).
* `channels.msteams.sharePointSiteId` : ID du site SharePoint pour les téléversements de fichiers dans les discussions de groupe/canaux (voir [Envoi de fichiers dans les discussions de groupe](#sending-files-in-group-chats)).

<div id="routing-sessions">
  ## Routage et sessions
</div>

* Les clés de session suivent le format standard d’agent (voir [/concepts/session](/fr/concepts/session)) :
  * Les messages directs partagent la session principale (`agent:<agentId>:<mainKey>`).
  * Les messages de canal ou de groupe utilisent l’identifiant de conversation :
    * `agent:<agentId>:msteams:channel:<conversationId>`
    * `agent:<agentId>:msteams:group:<conversationId>`

<div id="reply-style-threads-vs-posts">
  ## Style de réponse : fils de discussion vs publications
</div>

Teams a récemment introduit deux styles d’UI de canal reposant sur le même modèle de données sous-jacent :

| Style                               | Description                                                                        | `replyStyle` recommandé |
| ----------------------------------- | ---------------------------------------------------------------------------------- | ----------------------- |
| **Publications** (classique)        | Les messages apparaissent sous forme de cartes avec des réponses en fil en dessous | `thread` (par défaut)   |
| **Fils de discussion** (type Slack) | Les messages s’enchaînent linéairement, plus comme dans Slack                      | `top-level`             |

**Problème :** l’API Teams n’expose pas le style d’UI utilisé par un canal. Si vous utilisez le mauvais `replyStyle` :

* `thread` dans un canal en style fils de discussion → les réponses apparaissent imbriquées de façon étrange
* `top-level` dans un canal en style publications → les réponses apparaissent comme des publications de premier niveau séparées au lieu d’être dans le fil

**Solution :** configurez `replyStyle` par canal, en fonction de la manière dont le canal est configuré :

```json
{
  "msteams": {
    "replyStyle": "thread",
    "teams": {
      "19:abc...@thread.tacv2": {
        "channels": {
          "19:xyz...@thread.tacv2": {
            "replyStyle": "top-level"
          }
        }
      }
    }
  }
}
```

<div id="attachments-images">
  ## Pièces jointes et images
</div>

**Limitations actuelles :**

* **DMs :** les images et pièces jointes fonctionnent via les API de fichiers du bot Teams.
* **Canaux/groupes :** les pièces jointes sont stockées dans l’espace de stockage M365 (SharePoint/OneDrive). La charge utile du webhook inclut uniquement un stub HTML, pas les octets réels du fichier. **Des autorisations Graph API sont requises** pour télécharger les pièces jointes des canaux.

Sans autorisations Graph, les messages de canal contenant des images seront reçus comme texte uniquement (le contenu de l’image n’est pas accessible au bot).
Par défaut, OpenClaw ne télécharge les médias qu’à partir des noms d’hôte Microsoft/Teams. Vous pouvez remplacer ce comportement avec `channels.msteams.mediaAllowHosts` (utilisez `["*"]` pour autoriser n’importe quel hôte).

<div id="sending-files-in-group-chats">
  ## Envoi de fichiers dans les conversations de groupe
</div>

Les bots peuvent envoyer des fichiers dans des conversations privées (DM) en utilisant le flux FileConsentCard (intégré). Cependant, **l’envoi de fichiers dans les conversations de groupe/canaux** nécessite une configuration supplémentaire :

| Contexte | Comment les fichiers sont envoyés | Configuration nécessaire |
|---------|------------------------------------|--------------------------|
| **DMs** | FileConsentCard → l’utilisateur accepte → le bot met le fichier en ligne | Fonctionne par défaut |
| **Conversations de groupe/canaux** | Téléverser vers SharePoint → partager le lien | Nécessite `sharePointSiteId` + des autorisations Graph |
| **Images (tout contexte)** | Encodées en Base64 et intégrées | Fonctionne par défaut |

<div id="why-group-chats-need-sharepoint">
  ### Pourquoi les conversations de groupe ont besoin de SharePoint
</div>

Les bots n&#39;ont pas d&#39;espace OneDrive personnel (le point de terminaison Graph API `/me/drive` ne fonctionne pas pour les identités d&#39;application). Pour envoyer des fichiers dans les conversations de groupe ou les canaux, le bot les charge sur un **site SharePoint** et crée un lien de partage.

<div id="setup">
  ### Configuration
</div>

1. **Ajoutez les autorisations Graph API** dans Entra ID (Azure AD) → App Registration :
   * `Sites.ReadWrite.All` (Application) - téléverser des fichiers vers SharePoint
   * `Chat.Read.All` (Application) - facultatif, active les liens de partage par utilisateur

2. **Accordez le consentement administrateur** pour le tenant.

3. **Récupérez l’ID de votre site SharePoint :**
   ```bash
   # Via Graph Explorer ou curl avec un jeton valide :
   curl -H "Authorization: Bearer $TOKEN" \
     "https://graph.microsoft.com/v1.0/sites/{hostname}:/{site-path}"

   # Exemple : pour un site à l’adresse "contoso.sharepoint.com/sites/BotFiles"
   curl -H "Authorization: Bearer $TOKEN" \
     "https://graph.microsoft.com/v1.0/sites/contoso.sharepoint.com:/sites/BotFiles"

   # La réponse contient notamment : "id": "contoso.sharepoint.com,guid1,guid2"
   ```

4. **Configurez OpenClaw :**
   ```json5
   {
     channels: {
       msteams: {
         // ... autres paramètres de configuration ...
         sharePointSiteId: "contoso.sharepoint.com,guid1,guid2"
       }
     }
   }
   ```

<div id="sharing-behavior">
  ### Comportement de partage
</div>

| Permission | Comportement de partage |
|------------|-------------------------|
| `Sites.ReadWrite.All` only | Lien de partage à l’échelle de l’organisation (toute personne de l’organisation peut y accéder) |
| `Sites.ReadWrite.All` + `Chat.Read.All` | Lien de partage individuel (seuls les membres du chat peuvent y accéder) |

Le partage individuel est plus sécurisé, car seuls les participants au chat peuvent accéder au fichier. Si l’autorisation `Chat.Read.All` est absente, le bot repasse à un partage à l’échelle de l’organisation.

<div id="fallback-behavior">
  ### Comportement de repli
</div>

| Scénario | Résultat |
|----------|--------|
| Conversation de groupe + fichier + `sharePointSiteId` configuré | Chargement sur SharePoint, envoi d’un lien de partage |
| Conversation de groupe + fichier + pas de `sharePointSiteId` | Tentative de chargement vers OneDrive (peut échouer), envoi du texte seul |
| Conversation personnelle + fichier | Workflow FileConsentCard (fonctionne sans SharePoint) |
| Tout contexte + image | Encodée en Base64 en ligne (fonctionne sans SharePoint) |

<div id="files-stored-location">
  ### Emplacement de stockage des fichiers
</div>

Les fichiers téléversés sont stockés dans le dossier `/OpenClawShared/` de la bibliothèque de documents par défaut du site SharePoint configuré.

<div id="polls-adaptive-cards">
  ## Sondages (Adaptive Cards)
</div>

OpenClaw envoie les sondages Teams sous forme d’Adaptive Cards (il n’existe pas d’API de sondage native pour Teams).

* CLI : `openclaw message poll --channel msteams --target conversation:<id> ...`
* Les votes sont enregistrés par le Gateway OpenClaw dans `~/.openclaw/msteams-polls.json`.
* Le Gateway doit rester en ligne pour que les votes soient enregistrés.
* Les sondages ne publient pas encore automatiquement de synthèse des résultats (consultez le fichier de stockage si nécessaire).

<div id="adaptive-cards-arbitrary">
  ## Cartes adaptatives (arbitraires)
</div>

Envoyez n&#39;importe quel JSON de carte adaptative à des utilisateurs ou dans des conversations Teams en utilisant l&#39;outil `message` ou la CLI.

Le paramètre `card` accepte un objet JSON de carte adaptative. Lorsque `card` est renseigné, le texte du message est facultatif.

**Outil d’agent :**

```json
{
  "action": "send",
  "channel": "msteams",
  "target": "user:<id>",
  "card": {
    "type": "AdaptiveCard",
    "version": "1.5",
    "body": [{"type": "TextBlock", "text": "Hello!"}]
  }
}
```

**CLI :**

```bash
openclaw message send --channel msteams \
  --target "conversation:19:abc...@thread.tacv2" \
  --card '{"type":"AdaptiveCard","version":"1.5","body":[{"type":"TextBlock","text":"Hello!"}]}'
```

Consultez la [documentation Adaptive Cards](https://adaptivecards.io/) pour le schéma des cartes et des exemples. Pour plus de détails sur les formats cibles, consultez la section [Formats cibles](#target-formats) ci-dessous.

<div id="target-formats">
  ## Formats des cibles
</div>

Les cibles MSTeams utilisent des préfixes pour distinguer les utilisateurs des conversations :

| Type de cible         | Format                           | Exemple                                             |
| --------------------- | -------------------------------- | --------------------------------------------------- |
| Utilisateur (par ID)  | `user:<aad-object-id>`           | `user:40a1a0ed-4ff2-4164-a219-55518990c197`         |
| Utilisateur (par nom) | `user:<display-name>`            | `user:John Smith` (nécessite Graph API)             |
| Groupe/canal          | `conversation:<conversation-id>` | `conversation:19:abc123...@thread.tacv2`            |
| Groupe/canal (brut)   | `<conversation-id>`              | `19:abc123...@thread.tacv2` (si contient `@thread`) |

**Exemples CLI :**

```bash
# Send to a user by ID
openclaw message send --channel msteams --target "user:40a1a0ed-..." --message "Hello"

# Envoyer à un utilisateur par nom d'affichage (déclenche une recherche via l'API Graph)
openclaw message send --channel msteams --target "user:John Smith" --message "Hello"

# Send to a group chat or channel
openclaw message send --channel msteams --target "conversation:19:abc...@thread.tacv2" --message "Hello"

# Send an Adaptive Card to a conversation
openclaw message send --channel msteams --target "conversation:19:abc...@thread.tacv2" \
  --card '{"type":"AdaptiveCard","version":"1.5","body":[{"type":"TextBlock","text":"Hello"}]}'
```

**Exemples d’outils d’Agent :**

```json
{
  "action": "send",
  "channel": "msteams",
  "target": "user:John Smith",
  "message": "Hello!"
}
```

```json
{
  "action": "send",
  "channel": "msteams",
  "target": "conversation:19:abc...@thread.tacv2",
  "card": {"type": "AdaptiveCard", "version": "1.5", "body": [{"type": "TextBlock", "text": "Hello"}]}
}
```

Remarque : sans le préfixe `user:`, les noms sont résolus par défaut comme groupe/équipe. Utilisez toujours `user:` lorsque vous ciblez des personnes par leur nom d’affichage.

<div id="proactive-messaging">
  ## Messages proactifs
</div>

* Les messages proactifs ne sont possibles **qu’après** une première interaction de l’utilisateur, car c’est à ce moment-là que nous stockons les références de conversation.
* Voir `/gateway/configuration` pour `dmPolicy` et le filtrage par liste d’autorisation.

<div id="team-and-channel-ids-common-gotcha">
  ## Identifiants d’équipe et de canal (piège courant)
</div>

Le paramètre de requête `groupId` dans les URL Teams n’est **PAS** l’identifiant d’équipe utilisé pour la configuration. Extrayez plutôt les identifiants à partir du chemin de l’URL :

**URL d’équipe :**

```
https://teams.microsoft.com/l/team/19%3ABk4j...%40thread.tacv2/conversations?groupId=...
                                    └────────────────────────────┘
                                    Team ID (décodez cette URL)
```

**URL du canal :**

```
https://teams.microsoft.com/l/channel/19%3A15bc...%40thread.tacv2/ChannelName?groupId=...
                                      └─────────────────────────┘
                                      Channel ID (URL-decode this)
```

**Pour la config :**

* Team ID = segment de chemin d’URL après `/team/` (décodé depuis l’URL, par exemple `19:Bk4j...@thread.tacv2`)
* Channel ID = segment de chemin d’URL après `/channel/` (décodé depuis l’URL)
* **Ignorez** le paramètre de requête `groupId`

<div id="private-channels">
  ## Canaux privés
</div>

Les bots ne sont que partiellement pris en charge dans les canaux privés :

| Fonctionnalité | Canaux standard | Canaux privés |
|----------------|-----------------|---------------|
| Installation du bot | Oui | Limitée |
| Messages en temps réel (webhook) | Oui | Peut ne pas fonctionner |
| Autorisations RSC | Oui | Peut se comporter différemment |
| @mentions | Oui | Si le bot est accessible |
| Historique via Graph API | Oui | Oui (avec autorisations) |

**Solutions de contournement si les canaux privés ne fonctionnent pas :**

1. Utiliser des canaux standard pour les interactions avec le bot
2. Utiliser les messages privés (DM) – les utilisateurs peuvent toujours envoyer un message directement au bot
3. Utiliser Graph API pour l&#39;accès à l&#39;historique (nécessite `ChannelMessage.Read.All`)

<div id="troubleshooting">
  ## Résolution des problèmes
</div>

<div id="common-issues">
  ### Problèmes courants
</div>

* **Les images ne s’affichent pas dans les canaux :** autorisations Graph ou consentement administrateur manquants. Réinstallez l’application Teams, puis quittez et rouvrez complètement Teams.
* **Aucune réponse dans le canal :** les mentions sont requises par défaut ; définissez `channels.msteams.requireMention=false` ou configurez ce paramètre par équipe/canal.
* **Incohérence de version (Teams affiche encore l’ancien manifeste) :** supprimez puis réajoutez l’application, puis quittez complètement Teams pour forcer l’actualisation.
* **401 Unauthorized depuis le webhook :** comportement attendu lors d’un test manuel sans Azure JWT – cela signifie que le point de terminaison (endpoint) est joignable mais que l’authentification a échoué. Utilisez Azure Web Chat pour tester correctement.

<div id="manifest-upload-errors">
  ### Erreurs lors du téléversement du manifeste
</div>

* **&quot;Icon file cannot be empty&quot; :** Le manifeste référence des fichiers d’icône de taille 0 octet. Créez des icônes PNG valides (32x32 pour `outline.png`, 192x192 pour `color.png`).
* **&quot;webApplicationInfo.Id already in use&quot; :** L’application est encore installée dans une autre équipe ou conversation. Repérez-la et désinstallez-la d’abord, ou attendez 5 à 10 minutes pour la propagation.
* **&quot;Something went wrong&quot; lors du téléversement :** Téléversez-la plutôt via https://admin.teams.microsoft.com, ouvrez les outils de développement du navigateur (F12) → onglet Network, et vérifiez le corps de la réponse pour trouver l’erreur réelle.
* **Échec du sideload :** Essayez &quot;Upload an app to your org&#39;s app catalog&quot; plutôt que &quot;Upload a custom app&quot; ; cette opération contourne souvent les restrictions de sideload.

<div id="rsc-permissions-not-working">
  ### Les autorisations RSC ne fonctionnent pas
</div>

1. Vérifiez que `webApplicationInfo.id` correspond exactement à l’App ID de votre bot
2. Chargez à nouveau l’application et réinstallez-la dans l’équipe ou la discussion
3. Vérifiez si l’administrateur de votre organisation a bloqué les autorisations RSC
4. Confirmez que vous utilisez la bonne portée : `ChannelMessage.Read.Group` pour les équipes, `ChatMessage.Read.Chat` pour les discussions de groupe

<div id="references">
  ## Références
</div>

* [Create Azure Bot](https://learn.microsoft.com/en-us/azure/bot-service/bot-service-quickstart-registration) - guide de configuration d’un bot Azure
* [Teams Developer Portal](https://dev.teams.microsoft.com/apps) - créer et gérer des applications Teams
* [Teams app manifest schema](https://learn.microsoft.com/en-us/microsoftteams/platform/resources/schema/manifest-schema)
* [Receive channel messages with RSC](https://learn.microsoft.com/en-us/microsoftteams/platform/bots/how-to/conversations/channel-messages-with-rsc)
* [RSC permissions reference](https://learn.microsoft.com/en-us/microsoftteams/platform/graph-api/rsc/resource-specific-consent)
* [Teams bot file handling](https://learn.microsoft.com/en-us/microsoftteams/platform/bots/how-to/bots-filesv4) (pour les canaux/groupes, nécessite Graph)
* [Proactive messaging](https://learn.microsoft.com/en-us/microsoftteams/platform/bots/how-to/conversations/send-proactive-messages)