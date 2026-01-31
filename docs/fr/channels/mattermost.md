---
title: Mattermost
summary: "Configuration du bot Mattermost et d'OpenClaw"
read_when:
  - Configuration de Mattermost
  - Débogage du routage dans Mattermost
---

<div id="mattermost-plugin">
  # Mattermost (plugin)
</div>

Statut : pris en charge via un plugin (jeton de bot + événements WebSocket). Les canaux, les groupes et les messages privés (DM) sont pris en charge.
Mattermost est une plateforme de messagerie d’équipe auto‑hébergée ; consultez le site officiel
[mattermost.com](https://mattermost.com) pour les détails sur le produit et les téléchargements.

<div id="plugin-required">
  ## Plugin requis
</div>

Mattermost est distribué sous forme de plugin et n’est pas inclus dans l’installation de base.

Installez‑le via la CLI depuis le registre npm :

```bash
openclaw plugins install @openclaw/mattermost
```

Copie locale (lorsque vous exécutez depuis un dépôt Git) :

```bash
openclaw plugins install ./extensions/mattermost
```

Si vous choisissez Mattermost pendant la configuration/onboarding et qu’un checkout Git est détecté,
OpenClaw proposera automatiquement le chemin d’installation local.

Détails : [Plugins](/fr/plugin)

<div id="quick-setup">
  ## Configuration rapide
</div>

1. Installez le plugin Mattermost.
2. Créez un compte bot Mattermost et copiez le **jeton du bot**.
3. Copiez l’**URL de base** Mattermost (par exemple `https://chat.example.com`).
4. Configurez OpenClaw et démarrez le Gateway.

Configuration minimale :

```json5
{
  channels: {
    mattermost: {
      enabled: true,
      botToken: "mm-token",
      baseUrl: "https://chat.example.com",
      dmPolicy: "pairing"
    }
  }
}
```

<div id="environment-variables-default-account">
  ## Variables d&#39;environnement (compte par défaut)
</div>

Définissez-les sur l&#39;hôte du Gateway si vous préférez utiliser des variables d&#39;environnement :

* `MATTERMOST_BOT_TOKEN=...`
* `MATTERMOST_URL=https://chat.example.com`

Les variables d&#39;environnement s&#39;appliquent uniquement au compte **par défaut** (`default`). Les autres comptes doivent utiliser des valeurs de configuration.

<div id="chat-modes">
  ## Modes de chat
</div>

Mattermost répond automatiquement aux DMs. Le comportement dans les canaux est contrôlé par `chatmode` :

* `oncall` (par défaut) : répondre uniquement lorsqu’il est @mentionné dans les canaux.
* `onmessage` : répondre à chaque message dans un canal.
* `onchar` : répondre lorsqu’un message commence par un préfixe déclencheur.

Exemple de configuration :

```json5
{
  channels: {
    mattermost: {
      chatmode: "onchar",
      oncharPrefixes: [">", "!"]
    }
  }
}
```

Remarques :

* `onchar` continue de répondre aux @mentions explicites.
* `channels.mattermost.requireMention` reste pris en compte pour les configurations héritées, mais `chatmode` est à privilégier.

<div id="access-control-dms">
  ## Contrôle d&#39;accès (DM)
</div>

* Par défaut : `channels.mattermost.dmPolicy = "pairing"` (les expéditeurs inconnus reçoivent un code d&#39;appairage).
* Approuver via :
  * `openclaw pairing list mattermost`
  * `openclaw pairing approve mattermost <CODE>`
* DM publics : `channels.mattermost.dmPolicy="open"` plus `channels.mattermost.allowFrom=["*"]` (le mode `open` autorise l&#39;acceptation de messages sans restriction de la part de n&#39;importe quel utilisateur).

<div id="channels-groups">
  ## Canaux (groupes)
</div>

* Par défaut : `channels.mattermost.groupPolicy = "allowlist"` (restreint aux mentions).
* Autorisez des expéditeurs via `channels.mattermost.groupAllowFrom` (IDs d’utilisateurs ou `@username`).
* Canaux ouverts : `channels.mattermost.groupPolicy="open"` (restreint aux mentions ; la valeur de stratégie open autorise l’acceptation de messages sans restriction de n’importe quel utilisateur).

<div id="targets-for-outbound-delivery">
  ## Cibles pour l’envoi sortant
</div>

Utilisez ces formats de destination avec `openclaw message send` ou cron/webhooks :

* `channel:<id>` pour un canal
* `user:<id>` pour un message direct (DM)
* `@username` pour un message direct (DM, résolu via l’API Mattermost)

Les ID seuls sont traités comme des canaux.

<div id="multi-account">
  ## Plusieurs comptes
</div>

Mattermost prend en charge plusieurs comptes via `channels.mattermost.accounts` :

```json5
{
  channels: {
    mattermost: {
      accounts: {
        default: { name: "Primary", botToken: "mm-token", baseUrl: "https://chat.example.com" },
        alerts: { name: "Alerts", botToken: "mm-token-2", baseUrl: "https://alerts.example.com" }
      }
    }
  }
}
```

<div id="troubleshooting">
  ## Dépannage
</div>

* Aucune réponse dans les canaux : vérifiez que le bot est présent dans le canal et mentionnez-le (oncall), utilisez un préfixe de déclenchement (onchar) ou définissez `chatmode: "onmessage"`.
* Erreurs d’authentification : vérifiez le jeton du bot, l’URL de base et si le compte est bien activé.
* Problèmes avec plusieurs comptes : les variables d’environnement ne s’appliquent qu’au compte `default`.