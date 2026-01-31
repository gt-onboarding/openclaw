---
title: Webchat
summary: "Hébergement statique WebChat en loopback et utilisation du WS du Gateway pour l'UI de chat"
read_when:
  - Débogage ou configuration de l'accès à WebChat
---

<div id="webchat-gateway-websocket-ui">
  # WebChat (Gateway WebSocket UI)
</div>

Statut : l’UI de chat SwiftUI pour macOS/iOS communique directement avec le WebSocket du Gateway.

<div id="what-it-is">
  ## De quoi s’agit-il
</div>

* Une UI de chat native pour Gateway (sans navigateur intégré ni serveur statique local).
* Utilise les mêmes sessions et règles de routage que les autres canaux.
* Routage déterministe : les réponses reviennent toujours dans WebChat.

<div id="quick-start">
  ## Démarrage rapide
</div>

1. Démarrez le Gateway.
2. Ouvrez la WebChat UI (application macOS/iOS) ou l&#39;onglet de discussion du Control UI.
3. Assurez-vous que l&#39;authentification du Gateway est configurée (obligatoire par défaut, même sur l&#39;interface loopback locale).

<div id="how-it-works-behavior">
  ## Fonctionnement (comportement)
</div>

* L&#39;UI se connecte au WebSocket du Gateway et utilise `chat.history`, `chat.send` et `chat.inject`.
* `chat.inject` ajoute une note de l&#39;assistant à la fin du journal de conversation et la diffuse à l&#39;UI (aucune exécution d&#39;agent).
* L&#39;historique est toujours récupéré auprès du Gateway (aucune surveillance de fichiers locaux).
* Si le Gateway est injoignable, WebChat est en lecture seule.

<div id="remote-use">
  ## Utilisation à distance
</div>

* Le mode distant fait passer le WebSocket du Gateway via un tunnel SSH/Tailscale.
* Vous n’avez pas besoin d’exécuter un serveur WebChat distinct.

<div id="configuration-reference-webchat">
  ## Référence de configuration (WebChat)
</div>

Configuration complète : [Configuration](/fr/gateway/configuration)

Options du canal :

* Aucun bloc `webchat.*` dédié. WebChat utilise le point de terminaison du Gateway + les paramètres d’authentification ci-dessous.

Options globales associées :

* `gateway.port`, `gateway.bind` : hôte/port WebSocket.
* `gateway.auth.mode`, `gateway.auth.token`, `gateway.auth.password` : authentification WebSocket.
* `gateway.remote.url`, `gateway.remote.token`, `gateway.remote.password` : cible du Gateway distant.
* `session.*` : paramètres de stockage des sessions et valeurs par défaut des clés principales.