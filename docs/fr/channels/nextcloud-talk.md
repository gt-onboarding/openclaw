---
title: Nextcloud Talk
summary: "Statut du support, fonctionnalités et configuration de Nextcloud Talk"
read_when:
  - Travail sur les fonctionnalités du canal Nextcloud Talk
---

<div id="nextcloud-talk-plugin">
  # Nextcloud Talk (plugin)
</div>

Statut : pris en charge via un plugin (bot webhook). Les messages directs, les salons de discussion, les réactions et les messages Markdown sont pris en charge.

<div id="plugin-required">
  ## Plugin requis
</div>

Nextcloud Talk est fourni sous forme de plugin et n’est pas inclus dans l’installation principale.

Installez-le via la CLI (registre npm) :

```bash
openclaw plugins install @openclaw/nextcloud-talk
```

Copie locale (lors de l’exécution depuis un dépôt Git) :

```bash
openclaw plugins install ./extensions/nextcloud-talk
```

Si vous choisissez Nextcloud Talk lors de la configuration/l’onboarding et qu’un checkout Git est détecté,
OpenClaw proposera automatiquement le chemin d’installation local.

Détails : [Plugins](/fr/plugin)

<div id="quick-setup-beginner">
  ## Configuration rapide (débutant)
</div>

1. Installez le plugin Nextcloud Talk.
2. Sur votre serveur Nextcloud, créez un bot :
   ```bash
   ./occ talk:bot:install "OpenClaw" "<shared-secret>" "<webhook-url>" --feature reaction
   ```
3. Activez le bot dans les paramètres du salon cible.
4. Configurez OpenClaw :
   * Config : `channels.nextcloud-talk.baseUrl` + `channels.nextcloud-talk.botSecret`
   * Ou variable d’environnement : `NEXTCLOUD_TALK_BOT_SECRET` (compte par défaut uniquement)
5. Redémarrez le Gateway (ou terminez l’onboarding).

Configuration minimale :

```json5
{
  channels: {
    "nextcloud-talk": {
      enabled: true,
      baseUrl: "https://cloud.example.com",
      botSecret: "shared-secret",
      dmPolicy: "pairing"
    }
  }
}
```

<div id="notes">
  ## Remarques
</div>

* Les bots ne peuvent pas initier de messages privés. L’utilisateur doit d’abord envoyer un message au bot.
* L’URL du webhook doit être accessible depuis le Gateway ; définissez `webhookPublicUrl` si le Gateway est derrière un proxy.
* Les téléversements de contenus média ne sont pas pris en charge par l’API du bot ; les contenus média sont envoyés sous forme d’URL.
* Le payload du webhook ne distingue pas les messages privés des salons ; définissez `apiUser` + `apiPassword` pour activer les recherches de type de salon (sinon les messages privés sont traités comme des salons).

<div id="access-control-dms">
  ## Contrôle d’accès (DMs/messages privés)
</div>

* Valeur par défaut : `channels.nextcloud-talk.dmPolicy = "pairing"`. Les expéditeurs inconnus reçoivent un code d’appairage.
* Approuver via :
  * `openclaw pairing list nextcloud-talk`
  * `openclaw pairing approve nextcloud-talk <CODE>`
* Messages privés publics : `channels.nextcloud-talk.dmPolicy="open"` ainsi que `channels.nextcloud-talk.allowFrom=["*"]`. Le mode `open` permet d’accepter des messages de tout utilisateur sans restriction.

<div id="rooms-groups">
  ## Salons (groupes)
</div>

* Par défaut : `channels.nextcloud-talk.groupPolicy = "allowlist"` (restreint aux mentions).
* Définissez les salons sur liste d’autorisation avec `channels.nextcloud-talk.rooms` :

```json5
{
  channels: {
    "nextcloud-talk": {
      rooms: {
        "room-token": { requireMention: true }
      }
    }
  }
}
```

* Pour n’autoriser aucun salon de discussion, laissez la liste d’autorisation vide ou définissez `channels.nextcloud-talk.groupPolicy="disabled"`.

<div id="capabilities">
  ## Fonctionnalités
</div>

| Fonctionnalité | Statut |
|---------|--------|
| Messages directs | Pris en charge |
| Salons | Pris en charge |
| Fils de discussion | Non pris en charge |
| Médias | Uniquement via URL |
| Réactions | Pris en charge |
| Commandes natives | Non pris en charge |

<div id="configuration-reference-nextcloud-talk">
  ## Référence de configuration (Nextcloud Talk)
</div>

Configuration complète : [Configuration](/fr/gateway/configuration)

Options du fournisseur :

* `channels.nextcloud-talk.enabled` : activer/désactiver le démarrage du canal.
* `channels.nextcloud-talk.baseUrl` : URL de l’instance Nextcloud.
* `channels.nextcloud-talk.botSecret` : secret partagé du bot.
* `channels.nextcloud-talk.botSecretFile` : chemin du fichier contenant le secret.
* `channels.nextcloud-talk.apiUser` : utilisateur d’API pour la recherche de salons (détection des DM).
* `channels.nextcloud-talk.apiPassword` : mot de passe d’API/d’application pour la recherche de salons.
* `channels.nextcloud-talk.apiPasswordFile` : chemin du fichier de mot de passe d’API.
* `channels.nextcloud-talk.webhookPort` : port d’écoute du webhook (par défaut : 8788).
* `channels.nextcloud-talk.webhookHost` : hôte du webhook (par défaut : 0.0.0.0).
* `channels.nextcloud-talk.webhookPath` : chemin du webhook (par défaut : /nextcloud-talk-webhook).
* `channels.nextcloud-talk.webhookPublicUrl` : URL de webhook accessible depuis l’extérieur.
* `channels.nextcloud-talk.dmPolicy` : `pairing | allowlist | open | disabled`.
* `channels.nextcloud-talk.allowFrom` : liste d’autorisation des DM (ID utilisateur). `open` nécessite `"*"`.
* `channels.nextcloud-talk.groupPolicy` : `allowlist | open | disabled`.
* `channels.nextcloud-talk.groupAllowFrom` : liste d’autorisation des groupes (ID utilisateur).
* `channels.nextcloud-talk.rooms` : paramètres et liste d’autorisation par salon.
* `channels.nextcloud-talk.historyLimit` : limite d’historique des groupes (0 pour désactiver).
* `channels.nextcloud-talk.dmHistoryLimit` : limite d’historique des DM (0 pour désactiver).
* `channels.nextcloud-talk.dms` : remplacements spécifiques par DM (`historyLimit`).
* `channels.nextcloud-talk.textChunkLimit` : taille des blocs de texte sortants (caractères).
* `channels.nextcloud-talk.chunkMode` : `length` (par défaut) ou `newline` pour découper sur les lignes vides (limites de paragraphe) avant le fractionnement par longueur.
* `channels.nextcloud-talk.blockStreaming` : désactiver le streaming par blocs pour ce canal.
* `channels.nextcloud-talk.blockStreamingCoalesce` : réglage de la coalescence du streaming par blocs.
* `channels.nextcloud-talk.mediaMaxMb` : limite des contenus média entrants (Mo).