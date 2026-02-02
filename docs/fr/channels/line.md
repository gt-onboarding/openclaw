---
title: LINE
summary: "Mise en place, configuration et utilisation du plugin LINE Messaging API"
read_when:
  - Vous voulez connecter OpenClaw à LINE
  - Vous devez configurer le webhook LINE et les identifiants
  - Vous voulez des options de messagerie spécifiques à LINE
---

<div id="line-plugin">
  # LINE (plugin)
</div>

LINE se connecte à OpenClaw via la LINE Messaging API. Le plugin s’exécute en tant que
récepteur de webhook sur le Gateway et utilise votre channel access token et votre channel secret pour
l’authentification.

Statut : pris en charge via plugin. Les messages directs, les conversations de groupe, les médias, les positions géographiques, les messages Flex,
les messages de modèles (template messages) et les réponses rapides sont pris en charge. Les réactions et les fils de discussion ne sont pas pris en charge.

<div id="plugin-required">
  ## Plugin requis
</div>

Installez le plugin LINE :

```bash
openclaw plugins install @openclaw/line
```

Copie de travail locale (lorsque vous exécutez depuis un dépôt Git) :

```bash
openclaw plugins install ./extensions/line
```


<div id="setup">
  ## Configuration
</div>

1. Crée un compte LINE Developers et ouvre la Console :
   https://developers.line.biz/console/
2. Crée (ou sélectionne) un fournisseur et ajoute un canal **Messaging API**.
3. Copie le **Channel access token** et le **Channel secret** dans les paramètres du canal.
4. Active **Use webhook** dans les paramètres de la Messaging API.
5. Configure l’URL du webhook vers l’endpoint de ton Gateway (HTTPS requis) :

```
https://gateway-host/line/webhook
```

Le Gateway répond à la vérification du webhook de LINE (GET) et aux événements entrants (POST).
Si vous avez besoin d’un chemin personnalisé, définissez `channels.line.webhookPath` ou
`channels.line.accounts.&lt;id&gt;.webhookPath` et mettez ensuite l’URL à jour en conséquence.


<div id="configure">
  ## Configurer
</div>

Configuration minimale :

```json5
{
  channels: {
    line: {
      enabled: true,
      channelAccessToken: "LINE_CHANNEL_ACCESS_TOKEN",
      channelSecret: "LINE_CHANNEL_SECRET",
      dmPolicy: "pairing"
    }
  }
}
```

Variables d&#39;environnement (uniquement pour le compte par défaut) :

* `LINE_CHANNEL_ACCESS_TOKEN`
* `LINE_CHANNEL_SECRET`

Fichiers de jeton et de secret :

```json5
{
  channels: {
    line: {
      tokenFile: "/path/to/line-token.txt",
      secretFile: "/path/to/line-secret.txt"
    }
  }
}
```

Plusieurs comptes :

```json5
{
  channels: {
    line: {
      accounts: {
        marketing: {
          channelAccessToken: "...",
          channelSecret: "...",
          webhookPath: "/line/marketing"
        }
      }
    }
  }
}
```


<div id="access-control">
  ## Contrôle d’accès
</div>

Par défaut, les messages privés passent par un appairage. Les expéditeurs inconnus reçoivent un code d’appairage et leurs messages sont ignorés jusqu’à ce qu’ils soient approuvés.

```bash
openclaw pairing list line
openclaw pairing approve line <CODE>
```

Listes d’autorisation et politiques : (la valeur de politique `open` autorise l’acceptation de messages sans restriction de la part de n’importe quel utilisateur)

* `channels.line.dmPolicy`: `pairing | allowlist | open | disabled`
* `channels.line.allowFrom`: ID utilisateur LINE présents dans la liste d’autorisation pour les messages privés (MP)
* `channels.line.groupPolicy`: `allowlist | open | disabled`
* `channels.line.groupAllowFrom`: ID utilisateur LINE présents dans la liste d’autorisation pour les groupes
* Remplacements spécifiques à chaque groupe : `channels.line.groups.<groupId>.allowFrom`

Les identifiants LINE sont sensibles à la casse. Exemples d’identifiants valides :

* Utilisateur : `U` + 32 caractères hexadécimaux
* Groupe : `C` + 32 caractères hexadécimaux
* Salon : `R` + 32 caractères hexadécimaux


<div id="message-behavior">
  ## Comportement des messages
</div>

- Le texte est découpé en blocs de 5000 caractères.
- Le formatage Markdown est supprimé ; les blocs de code et les tableaux sont convertis en cartes Flex lorsque c’est possible.
- Les réponses en streaming sont mises en mémoire tampon ; LINE reçoit des blocs complets avec une animation de chargement pendant que l’agent travaille.
- Les téléchargements de médias sont plafonnés par `channels.line.mediaMaxMb` (10 par défaut).

<div id="channel-data-rich-messages">
  ## Données de canal (messages enrichis)
</div>

Utilisez `channelData.line` pour envoyer des réponses rapides, des emplacements, des cartes Flex ou des messages de modèle.

```json5
{
  text: "Here you go",
  channelData: {
    line: {
      quickReplies: ["Status", "Help"],
      location: {
        title: "Office",
        address: "123 Main St",
        latitude: 35.681236,
        longitude: 139.767125
      },
      flexMessage: {
        altText: "Status card",
        contents: { /* Flex payload */ }
      },
      templateMessage: {
        type: "confirm",
        text: "Proceed?",
        confirmLabel: "Yes",
        confirmData: "yes",
        cancelLabel: "No",
        cancelData: "no"
      }
    }
  }
}
```

Le plugin LINE propose également une commande `/card` pour des préréglages de messages Flex :

```
/card info "Welcome" "Thanks for joining!"
```


<div id="troubleshooting">
  ## Dépannage
</div>

- **Échec de la vérification du webhook :** vérifiez que l’URL du webhook est en HTTPS et que
  `channelSecret` correspond à celui de la console LINE.
- **Aucun événement entrant :** assurez-vous que le chemin du webhook correspond à `channels.line.webhookPath`
  et que le Gateway est accessible depuis LINE.
- **Erreurs de téléchargement de médias :** augmentez `channels.line.mediaMaxMb` si la taille des médias dépasse
  la limite par défaut.