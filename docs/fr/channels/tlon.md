---
title: Tlon
summary: "État de la prise en charge, fonctionnalités et configuration de Tlon/Urbit"
read_when:
  - Vous travaillez sur les fonctionnalités du canal Tlon/Urbit
---

<div id="tlon-plugin">
  # Tlon (plugin)
</div>

Tlon est une messagerie décentralisée basée sur Urbit. OpenClaw se connecte à votre vaisseau Urbit et peut
répondre aux DMs et aux messages des discussions de groupe. Les réponses dans un groupe requièrent par défaut une mention @ et peuvent
être davantage restreintes via des listes d’autorisation.

Statut : pris en charge via un plugin. DMs, mentions de groupe, réponses dans les fils de discussion et solution de repli en texte seul pour les médias
(URL ajoutée à la légende). Les réactions, sondages et envois de médias natifs ne sont pas pris en charge.

<div id="plugin-required">
  ## Plugin requis
</div>

Tlon est fourni en tant que plugin et n&#39;est pas inclus dans l&#39;installation de base.

Installez-le via la CLI (registre npm) :

```bash
openclaw plugins install @openclaw/tlon
```

Copie de travail locale (lors de l’exécution depuis un dépôt Git) :

```bash
openclaw plugins install ./extensions/tlon
```

Détails : [Plugins](/fr/plugin)

<div id="setup">
  ## Configuration
</div>

1. Installez le plugin Tlon.
2. Récupérez l’URL de votre ship et votre code de connexion.
3. Configurez `channels.tlon`.
4. Redémarrez le Gateway.
5. Envoyez un message privé (DM) au bot ou mentionnez‑le dans un canal de groupe.

Configuration minimale (compte unique) :

```json5
{
  channels: {
    tlon: {
      enabled: true,
      ship: "~sampel-palnet",
      url: "https://your-ship-host",
      code: "lidlut-tabwed-pillex-ridrup"
    }
  }
}
```

<div id="group-channels">
  ## Canaux de groupe
</div>

La découverte automatique est activée par défaut. Vous pouvez également épingler des canaux manuellement :

```json5
{
  channels: {
    tlon: {
      groupChannels: [
        "chat/~host-ship/general",
        "chat/~host-ship/support"
      ]
    }
  }
}
```

Désactivez la détection automatique :

```json5
{
  channels: {
    tlon: {
      autoDiscoverChannels: false
    }
  }
}
```

<div id="access-control">
  ## Contrôle d’accès
</div>

Liste d’autorisation DM (vide = tout autoriser) :

```json5
{
  channels: {
    tlon: {
      dmAllowlist: ["~zod", "~nec"]
    }
  }
}
```

Autorisations de groupe (restreintes par défaut) :

```json5
{
  channels: {
    tlon: {
      defaultAuthorizedShips: ["~zod"],
      authorization: {
        channelRules: {
          "chat/~host-ship/general": {
            mode: "restricted",
            allowedShips: ["~zod", "~nec"]
          },
          "chat/~host-ship/announcements": {
            mode: "open"
          }
        }
      }
    }
  }
}
```

<div id="delivery-targets-clicron">
  ## Cibles de distribution (CLI/cron)
</div>

Utilisez-les avec `openclaw message send` ou pour une distribution via cron :

* DM : `~sampel-palnet` ou `dm/~sampel-palnet`
* Groupe : `chat/~host-ship/channel` ou `group:~host-ship/channel`

<div id="notes">
  ## Remarques
</div>

* Les réponses dans les groupes nécessitent une mention (par exemple `~your-bot-ship`) pour répondre.
* Réponses dans les fils de discussion : si le message entrant est dans un fil, OpenClaw répond dans ce fil.
* Médias : `sendMedia` se rabat sur un texte + une URL (pas de téléversement natif).