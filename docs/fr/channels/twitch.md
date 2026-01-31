---
title: Twitch
summary: "Configuration et installation du bot de chat Twitch"
read_when:
  - Configuration de l'intégration du chat Twitch avec OpenClaw
---

<div id="twitch-plugin">
  # Twitch (plugin)
</div>

Prise en charge du chat Twitch via une connexion IRC. OpenClaw se connecte en tant qu’utilisateur Twitch (compte de bot) pour recevoir et envoyer des messages dans les canaux.

<div id="plugin-required">
  ## Plugin requis
</div>

Twitch est fourni sous forme de plugin et n&#39;est pas inclus avec l&#39;installation de base.

Installez-le via la CLI (registre npm) :

```bash
openclaw plugins install @openclaw/twitch
```

Version locale (lorsque vous exécutez depuis un dépôt Git) :

```bash
openclaw plugins install ./extensions/twitch
```

Pour plus de détails : [Plugins](/fr/plugin)

<div id="quick-setup-beginner">
  ## Configuration rapide (débutant)
</div>

1. Créez un compte Twitch dédié pour le bot (ou utilisez un compte existant).
2. Générez les identifiants : [Twitch Token Generator](https://twitchtokengenerator.com/)
   * Sélectionnez **Bot Token**
   * Vérifiez que les scopes `chat:read` et `chat:write` sont bien sélectionnés
   * Copiez le **Client ID** et l’**Access Token**
3. Trouvez votre ID utilisateur Twitch : https://www.streamweasels.com/tools/convert-twitch-username-to-user-id/
4. Configurez le token :
   * Env : `OPENCLAW_TWITCH_ACCESS_TOKEN=...` (compte par défaut uniquement)
   * Ou configuration : `channels.twitch.accessToken`
   * Si les deux sont définis, la configuration est prioritaire (la variable d’environnement n’est utilisée qu’en repli pour le compte par défaut).
5. Démarrez Gateway.

**⚠️ Important :** ajoutez un contrôle d’accès (`allowFrom` ou `allowedRoles`) pour empêcher des utilisateurs non autorisés de déclencher le bot. `requireMention` est à `true` par défaut.

Configuration minimale :

```json5
{
  channels: {
    twitch: {
      enabled: true,
      username: "openclaw",              // Bot's Twitch account
      accessToken: "oauth:abc123...",    // OAuth Access Token (or use OPENCLAW_TWITCH_ACCESS_TOKEN env var)
      clientId: "xyz789...",             // Client ID from Token Generator
      channel: "vevisk",                 // Which Twitch channel's chat to join (required)
      allowFrom: ["123456789"]           // (recommandé) Uniquement votre ID utilisateur Twitch - obtenez-le sur https://www.streamweasels.com/tools/convert-twitch-username-to-user-id/
    }
  }
}
```

<div id="what-it-is">
  ## Ce que c’est
</div>

* Une chaîne Twitch appartenant au Gateway.
* Routage déterministe : les réponses sont toujours renvoyées vers Twitch.
* Chaque compte est associé à une clé de session isolée `agent:<agentId>:twitch:<accountName>`.
* `username` est le compte du bot (celui qui s’authentifie), `channel` est le salon de discussion à rejoindre.

<div id="setup-detailed">
  ## Configuration détaillée
</div>

<div id="generate-credentials">
  ### Générer les identifiants
</div>

Utilisez [Twitch Token Generator](https://twitchtokengenerator.com/) :

* Sélectionnez **Bot Token**
* Vérifiez que les `scopes` `chat:read` et `chat:write` sont sélectionnés
* Copiez le **Client ID** et le **Access Token**

Aucun enregistrement manuel d’application n’est requis. Les jetons expirent après quelques heures.

<div id="configure-the-bot">
  ### Configurer le bot
</div>

**Variable d’environnement (compte par défaut uniquement) :**

```bash
OPENCLAW_TWITCH_ACCESS_TOKEN=oauth:abc123...
```

**Ou via la configuration :**

```json5
{
  channels: {
    twitch: {
      enabled: true,
      username: "openclaw",
      accessToken: "oauth:abc123...",
      clientId: "xyz789...",
      channel: "vevisk"
    }
  }
}
```

Si `env` et `config` sont tous deux définis, `config` est prioritaire.

<div id="access-control-recommended">
  ### Contrôle d’accès (recommandé)
</div>

```json5
{
  channels: {
    twitch: {
      allowFrom: ["123456789"],       // (recommandé) Votre ID utilisateur Twitch uniquement
      allowedRoles: ["moderator"]     // Or restrict to roles
    }
  }
}
```

**Rôles disponibles :** `"moderator"`, `"owner"`, `"vip"`, `"subscriber"`, `"all"`.

**Pourquoi utiliser des ID utilisateur ?** Les noms d’utilisateur peuvent changer, ce qui permet l’usurpation d’identité. Les ID utilisateur sont permanents.

Trouvez votre ID utilisateur Twitch : https://www.streamweasels.com/tools/convert-twitch-username-%20to-user-id/ (Convertissez votre nom d’utilisateur Twitch en ID)

<div id="token-refresh-optional">
  ## Actualisation du jeton (facultatif)
</div>

Les jetons provenant de [Twitch Token Generator](https://twitchtokengenerator.com/) ne peuvent pas être actualisés automatiquement : renouvelez-les lorsqu’ils expirent.

Pour l’actualisation automatique des jetons, créez votre propre application Twitch dans la [console développeur Twitch](https://dev.twitch.tv/console) et ajoutez-la à votre configuration :

```json5
{
  channels: {
    twitch: {
      clientSecret: "your_client_secret",
      refreshToken: "your_refresh_token"
    }
  }
}
```

Le bot actualise automatiquement les jetons d’accès avant leur expiration et enregistre les événements de rafraîchissement dans les journaux.

<div id="multi-account-support">
  ## Prise en charge de plusieurs comptes
</div>

Utilisez `channels.twitch.accounts` avec des jetons spécifiques à chaque compte. Voir [`gateway/configuration`](/fr/gateway/configuration) pour le modèle partagé.

Exemple (un compte bot dans deux chaînes) :

```json5
{
  channels: {
    twitch: {
      accounts: {
        channel1: {
          username: "openclaw",
          accessToken: "oauth:abc123...",
          clientId: "xyz789...",
          channel: "vevisk"
        },
        channel2: {
          username: "openclaw",
          accessToken: "oauth:def456...",
          clientId: "uvw012...",
          channel: "secondchannel"
        }
      }
    }
  }
}
```

**Remarque :** chaque compte doit disposer de son propre jeton (un jeton par canal).

<div id="access-control">
  ## Contrôle d’accès
</div>

<div id="role-based-restrictions">
  ### Restrictions par rôle
</div>

```json5
{
  channels: {
    twitch: {
      accounts: {
        default: {
          allowedRoles: ["moderator", "vip"]
        }
      }
    }
  }
}
```

<div id="allowlist-by-user-id-most-secure">
  ### Liste d’autorisation par identifiant utilisateur (le plus sûr)
</div>

```json5
{
  channels: {
    twitch: {
      accounts: {
        default: {
          allowFrom: ["123456789", "987654321"]
        }
      }
    }
  }
}
```

<div id="combined-allowlist-roles">
  ### Liste d’autorisation combinée + rôles
</div>

Les utilisateurs figurant dans `allowFrom` ne sont pas soumis aux contrôles de rôle :

```json5
{
  channels: {
    twitch: {
      accounts: {
        default: {
          allowFrom: ["123456789"],
          allowedRoles: ["moderator"]
        }
      }
    }
  }
}
```

<div id="disable-mention-requirement">
  ### Désactiver l&#39;obligation de @mention
</div>

Par défaut, `requireMention` est `true`. Pour la désactiver et répondre à tous les messages :

```json5
{
  channels: {
    twitch: {
      accounts: {
        default: {
          requireMention: false
        }
      }
    }
  }
}
```

<div id="troubleshooting">
  ## Dépannage
</div>

Commencez par exécuter les commandes de diagnostic :

```bash
openclaw doctor
openclaw channels status --probe
```

<div id="bot-doesnt-respond-to-messages">
  ### Le bot ne répond pas aux messages
</div>

**Vérifiez le contrôle d&#39;accès :** Définissez temporairement `allowedRoles: ["all"]` pour tester.

**Vérifiez que le bot est dans le canal :** Le bot doit être présent dans le canal spécifié dans `channel`.

<div id="token-issues">
  ### Problèmes de jeton
</div>

**&quot;Failed to connect&quot; ou erreurs d’authentification :**

* Vérifiez que `accessToken` correspond bien à la valeur du jeton d’accès OAuth (généralement préfixée par `oauth:`)
* Vérifiez que le jeton dispose des scopes `chat:read` et `chat:write`
* Si vous utilisez le renouvellement de jeton, vérifiez que `clientSecret` et `refreshToken` sont correctement configurés

<div id="token-refresh-not-working">
  ### Le rafraîchissement du token ne fonctionne pas
</div>

**Vérifiez les logs pour les événements de rafraîchissement :**

```
Using env token source for mybot
Access token refreshed for user 123456 (expires in 14400s)
```

Si vous voyez le message « token refresh disabled (no refresh token) » :

* Assurez-vous que `clientSecret` est fourni
* Assurez-vous que `refreshToken` est fourni

<div id="config">
  ## Configuration
</div>

**Configuration du compte :**

* `username` - Nom d’utilisateur du bot
* `accessToken` - Jeton d’accès OAuth avec `chat:read` et `chat:write`
* `clientId` - ID client Twitch (depuis le générateur de jetons ou votre application)
* `channel` - Canal à rejoindre (obligatoire)
* `enabled` - Activer ce compte (par défaut : `true`)
* `clientSecret` - Facultatif : pour le rafraîchissement automatique du jeton
* `refreshToken` - Facultatif : pour le rafraîchissement automatique du jeton
* `expiresIn` - Expiration du jeton en secondes
* `obtainmentTimestamp` - Horodatage d’obtention du jeton
* `allowFrom` - Liste d’autorisation d’identifiants utilisateur
* `allowedRoles` - Contrôle d’accès basé sur les rôles (`"moderator" | "owner" | "vip" | "subscriber" | "all"`)
* `requireMention` - Nécessite une @mention (par défaut : `true`)

**Options du fournisseur :**

* `channels.twitch.enabled` - Activer/désactiver le démarrage du canal
* `channels.twitch.username` - Nom d’utilisateur du bot (configuration simplifiée pour compte unique)
* `channels.twitch.accessToken` - Jeton d’accès OAuth (configuration simplifiée pour compte unique)
* `channels.twitch.clientId` - ID client Twitch (configuration simplifiée pour compte unique)
* `channels.twitch.channel` - Canal à rejoindre (configuration simplifiée pour compte unique)
* `channels.twitch.accounts.<accountName>` - Configuration multi-comptes (tous les champs de compte ci‑dessus)

Exemple complet :

```json5
{
  channels: {
    twitch: {
      enabled: true,
      username: "openclaw",
      accessToken: "oauth:abc123...",
      clientId: "xyz789...",
      channel: "vevisk",
      clientSecret: "secret123...",
      refreshToken: "refresh456...",
      allowFrom: ["123456789"],
      allowedRoles: ["moderator", "vip"],
      accounts: {
        default: {
          username: "mybot",
          accessToken: "oauth:abc123...",
          clientId: "xyz789...",
          channel: "your_channel",
          enabled: true,
          clientSecret: "secret123...",
          refreshToken: "refresh456...",
          expiresIn: 14400,
          obtainmentTimestamp: 1706092800000,
          allowFrom: ["123456789", "987654321"],
          allowedRoles: ["moderator"]
        }
      }
    }
  }
}
```

<div id="tool-actions">
  ## Actions de l&#39;outil
</div>

L&#39;agent peut appeler `twitch` avec l&#39;action :

* `send` - Envoyer un message sur une chaîne

Exemple :

```json5
{
  "action": "twitch",
  "params": {
    "message": "Hello Twitch!",
    "to": "#mychannel"
  }
}
```

<div id="safety-ops">
  ## Sécurité &amp; exploitation
</div>

* **Traite les jetons comme des mots de passe** - Ne les enregistre jamais dans Git
* **Utilise le rafraîchissement automatique des jetons** pour les bots qui tournent en continu
* **Utilise des listes d’autorisation d’identifiants utilisateur** plutôt que des noms d’utilisateur pour le contrôle d’accès
* **Surveille les journaux** pour les événements de rafraîchissement de jetons et l’état de la connexion
* **Limite au maximum la portée des jetons** - Ne demande que `chat:read` et `chat:write`
* **En cas de blocage** : redémarre le Gateway après avoir confirmé qu’aucun autre processus ne détient la session

<div id="limits">
  ## Limites
</div>

* **500 caractères** par message (découpés automatiquement aux limites de mots)
* Le Markdown est supprimé avant le découpage
* Aucune limitation de débit (utilise les limites de débit intégrées de Twitch)