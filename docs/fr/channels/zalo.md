---
title: Zalo
summary: "Statut de prise en charge des bots Zalo, fonctionnalités et configuration"
read_when:
  - Lorsque vous travaillez sur des fonctionnalités ou des webhooks Zalo
---

<div id="zalo-bot-api">
  # Zalo (Bot API)
</div>

Statut : expérimental. Messages privés uniquement ; les groupes seront bientôt pris en charge, conformément à la documentation Zalo.

<div id="plugin-required">
  ## Plugin requis
</div>

Zalo est fourni sous forme de plugin et n&#39;est pas intégré à l&#39;installation de base.

* Installez-le via la CLI : `openclaw plugins install @openclaw/zalo`
* Ou sélectionnez **Zalo** pendant l&#39;onboarding et confirmez l&#39;invite d&#39;installation
* Détails : [Plugins](/fr/plugin)

<div id="quick-setup-beginner">
  ## Configuration rapide (débutant)
</div>

1. Installez le plugin Zalo :
   * Depuis un checkout du dépôt : `openclaw plugins install ./extensions/zalo`
   * Depuis npm (si publié) : `openclaw plugins install @openclaw/zalo`
   * Ou sélectionnez **Zalo** lors de l’onboarding et confirmez la demande d’installation
2. Définissez le token :
   * Variable d’environnement : `ZALO_BOT_TOKEN=...`
   * Ou configuration : `channels.zalo.botToken: "..."`.
3. Redémarrez le Gateway (ou terminez l’onboarding).
4. L’accès en DM (message privé) se fait par appairage par défaut ; approuvez le code d’appairage lors du premier contact.

Configuration minimale :

```json5
{
  channels: {
    zalo: {
      enabled: true,
      botToken: "12345689:abc-xyz",
      dmPolicy: "appairage"
    }
  }
}
```

<div id="what-it-is">
  ## Ce que c’est
</div>

Zalo est une application de messagerie principalement utilisée au Vietnam ; son Bot API permet au Gateway d’exécuter un bot pour des conversations 1:1.
Elle est bien adaptée pour le support ou les notifications lorsque vous voulez un routage déterministe vers Zalo.

* Un canal Zalo Bot API géré par le Gateway.
* Routage déterministe : les réponses retournent vers Zalo ; le modèle ne choisit jamais les canaux.
* Les messages privés (DM) partagent la session principale de l’agent.
* Les groupes ne sont pas encore pris en charge (la documentation Zalo indique « coming soon »).

<div id="setup-fast-path">
  ## Configuration rapide
</div>

<div id="1-create-a-bot-token-zalo-bot-platform">
  ### 1) Créer un jeton de bot (Zalo Bot Platform)
</div>

1. Allez sur **https://bot.zaloplatforms.com** et connectez-vous.
2. Créez un nouveau bot et configurez ses paramètres.
3. Copiez le jeton du bot (format : `12345689:abc-xyz`).

<div id="2-configure-the-token-env-or-config">
  ### 2) Configurer le jeton (env ou config)
</div>

Exemple :

```json5
{
  channels: {
    zalo: {
      enabled: true,
      botToken: "12345689:abc-xyz",
      dmPolicy: "pairing"
    }
  }
}
```

Option d&#39;environnement : `ZALO_BOT_TOKEN=...` (fonctionne uniquement pour le compte par défaut).

Prise en charge de plusieurs comptes : utilisez `channels.zalo.accounts` avec un jeton par compte et un `name` facultatif.

3. Redémarrez Gateway. Zalo démarre lorsqu&#39;un jeton est résolu (env ou config).
4. L&#39;accès en DM utilise par défaut l&#39;appairage. Approuvez le code lors du premier contact avec le bot.

<div id="how-it-works-behavior">
  ## Fonctionnement (comportement)
</div>

* Les messages entrants sont normalisés dans l’enveloppe de canal partagée avec des espaces réservés aux médias.
* Les réponses sont toujours renvoyées vers le même chat Zalo.
* Le long-polling est activé par défaut ; le mode webhook est disponible via `channels.zalo.webhookUrl`.

<div id="limits">
  ## Limites
</div>

* Le texte sortant est découpé en segments de 2000 caractères (limite de l&#39;API Zalo).
* Les téléchargements et envois de médias sont plafonnés par `channels.zalo.mediaMaxMb` (5 par défaut).
* Le streaming est désactivé par défaut, car la limite de 2000 caractères le rend peu utile.

<div id="access-control-dms">
  ## Contrôle d’accès (messages privés)
</div>

<div id="dm-access">
  ### Accès DM
</div>

* Par défaut : `channels.zalo.dmPolicy = "pairing"`. Les expéditeurs inconnus reçoivent un code d’appairage ; les messages sont ignorés jusqu’à approbation (les codes expirent après 1 heure).
* Approuvez via :
  * `openclaw pairing list zalo`
  * `openclaw pairing approve zalo <CODE>`
* L’appairage est l’échange de jetons par défaut. Détails : [Pairing](/fr/start/pairing)
* `channels.zalo.allowFrom` accepte des identifiants numériques d’utilisateurs (aucune recherche par nom d’utilisateur disponible).

<div id="long-polling-vs-webhook">
  ## Long-polling vs webhook
</div>

* Par défaut : long-polling (aucune URL publique requise).
* Mode webhook : définissez `channels.zalo.webhookUrl` et `channels.zalo.webhookSecret`.
  * Le secret du webhook doit comporter entre 8 et 256 caractères.
  * L’URL du webhook doit utiliser HTTPS.
  * Zalo envoie des événements avec l’en-tête `X-Bot-Api-Secret-Token` pour vérification.
  * Le service HTTP de Gateway gère les requêtes webhook sur `channels.zalo.webhookPath` (par défaut, le chemin de l’URL du webhook).

**Remarque :** `getUpdates` (polling) et le webhook sont mutuellement exclusifs selon la documentation de l’API Zalo.

<div id="supported-message-types">
  ## Types de messages pris en charge
</div>

* **Messages texte** : prise en charge complète avec découpage en segments de 2000 caractères.
* **Messages image** : téléchargement et traitement des images reçues ; envoi d’images via `sendPhoto`.
* **Stickers** : enregistrés dans les journaux mais pas entièrement traités (aucune réponse de l’agent).
* **Types non pris en charge** : enregistrés dans les journaux (par exemple, messages provenant d’utilisateurs protégés).

<div id="capabilities">
  ## Fonctionnalités
</div>

| Fonctionnalité | Statut |
|----------------|--------|
| Messages privés | ✅ Pris en charge |
| Groupes | ❌ Bientôt disponible (d&#39;après la doc Zalo) |
| Médias (images) | ✅ Pris en charge |
| Réactions | ❌ Non pris en charge |
| Fils de discussion | ❌ Non pris en charge |
| Sondages | ❌ Non pris en charge |
| Commandes natives | ❌ Non pris en charge |
| Streaming | ⚠️ Bloqué (limite à 2 000 caractères) |

<div id="delivery-targets-clicron">
  ## Cibles de diffusion (CLI/cron)
</div>

* Utilisez un ID de conversation comme cible.
* Exemple : `openclaw message send --channel zalo --target 123456789 --message "hi"`.

<div id="troubleshooting">
  ## Dépannage
</div>

**Le bot ne répond pas :**

* Vérifie que le jeton est valide : `openclaw channels status --probe`
* Vérifie que l&#39;expéditeur est autorisé (appairage ou allowFrom)
* Vérifie les journaux du Gateway : `openclaw logs --follow`

**Le webhook ne reçoit pas d&#39;événements :**

* Vérifie que l&#39;URL du webhook utilise HTTPS
* Vérifie que le jeton secret contient entre 8 et 256 caractères
* Confirme que le point de terminaison HTTP du Gateway est joignable sur le chemin configuré
* Vérifie qu&#39;aucun polling `getUpdates` n&#39;est en cours d&#39;exécution (les deux sont mutuellement exclusifs)

<div id="configuration-reference-zalo">
  ## Référence de configuration (Zalo)
</div>

Configuration complète : [Configuration](/fr/gateway/configuration)

Options du fournisseur :

* `channels.zalo.enabled` : activer/désactiver le démarrage du canal.
* `channels.zalo.botToken` : jeton du bot depuis la Zalo Bot Platform.
* `channels.zalo.tokenFile` : lire le jeton depuis un chemin de fichier.
* `channels.zalo.dmPolicy` : `pairing | allowlist | open | disabled` (par défaut : appairage).
* `channels.zalo.allowFrom` : liste d’autorisation des DM (ID d’utilisateurs). `open` nécessite `"*"`. L’assistant en ligne de commande demandera des ID numériques.
* `channels.zalo.mediaMaxMb` : limite de média entrant/sortant (Mo, par défaut 5).
* `channels.zalo.webhookUrl` : activer le mode webhook (HTTPS requis).
* `channels.zalo.webhookSecret` : secret du webhook (8–256 caractères).
* `channels.zalo.webhookPath` : chemin du webhook sur le serveur HTTP du Gateway.
* `channels.zalo.proxy` : URL du proxy pour les requêtes API.

Options multi-compte :

* `channels.zalo.accounts.<id>.botToken` : jeton par compte.
* `channels.zalo.accounts.<id>.tokenFile` : fichier de jeton par compte.
* `channels.zalo.accounts.<id>.name` : nom d’affichage.
* `channels.zalo.accounts.<id>.enabled` : activer/désactiver le compte.
* `channels.zalo.accounts.<id>.dmPolicy` : stratégie de DM par compte.
* `channels.zalo.accounts.<id>.allowFrom` : liste d’autorisation par compte.
* `channels.zalo.accounts.<id>.webhookUrl` : URL de webhook par compte.
* `channels.zalo.accounts.<id>.webhookSecret` : secret de webhook par compte.
* `channels.zalo.accounts.<id>.webhookPath` : chemin de webhook par compte.
* `channels.zalo.accounts.<id>.proxy` : URL de proxy par compte.