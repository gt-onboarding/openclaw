---
title: iOS
summary: "Application iOS du nœud : connexion au Gateway, appairage, canvas et dépannage"
read_when:
  - Appairage ou reconnexion du nœud iOS
  - Exécution de l'application iOS à partir des sources
  - Débogage de la découverte du Gateway ou des commandes canvas
---

<div id="ios-app-node">
  # App iOS (nœud)
</div>

Disponibilité : aperçu interne. L’app iOS n’est pas encore distribuée publiquement.

<div id="what-it-does">
  ## Ce qu’elle fait
</div>

* Se connecte au Gateway via WebSocket (LAN ou tailnet).
* Expose les capacités du nœud : Canvas, capture d’écran, capture de la caméra, localisation, mode Conversation, réveil vocal.
* Reçoit des commandes `node.invoke` et signale les événements d’état du nœud.

<div id="requirements">
  ## Prérequis
</div>

* Gateway en cours d’exécution sur un autre appareil (macOS, Linux ou Windows via WSL2).
* Accès réseau :
  * Même LAN via Bonjour, **ou**
  * Tailnet via DNS-SD en unicast (domaine d’exemple : `openclaw.internal.`), **ou**
  * Configuration manuelle de l’hôte et du port (solution de repli).

<div id="quick-start-pair-connect">
  ## Démarrage rapide (appairage et connexion)
</div>

1. Démarrez le Gateway :

```bash
openclaw gateway --port 18789
```

2. Dans l’app iOS, ouvrez Réglages et choisissez un Gateway détecté (ou activez « Manual Host » et saisissez l’hôte/le port).

3. Approuvez la demande d’appairage sur l’hôte Gateway :

```bash
openclaw nodes pending
openclaw nodes approve <requestId>
```

4. Vérifiez la connexion :

```bash
openclaw nodes status
openclaw gateway call node.list --params "{}"
```

<div id="discovery-paths">
  ## Méthodes de découverte
</div>

<div id="bonjour-lan">
  ### Bonjour (LAN)
</div>

Le Gateway annonce `_openclaw-gw._tcp` sur `local.`. L’app iOS les répertorie automatiquement.

<div id="tailnet-cross-network">
  ### Tailnet (inter-réseaux)
</div>

Si mDNS est bloqué, utilisez une zone DNS-SD unicast (choisissez un domaine ; exemple : `openclaw.internal.`) et le split DNS de Tailscale.
Consultez [Bonjour](/fr/gateway/bonjour) pour un exemple avec CoreDNS.

<div id="manual-hostport">
  ### Hôte/port manuel
</div>

Dans **Settings**, activez **Manual Host** et saisissez l’hôte du Gateway et le port (par défaut `18789`).

<div id="canvas-a2ui">
  ## Canvas + A2UI
</div>

Le nœud iOS rend un canvas WKWebView. Utilisez `node.invoke` pour le piloter :

```bash
openclaw nodes invoke --node "iOS Node" --command canvas.navigate --params '{"url":"http://<gateway-host>:18793/__openclaw__/canvas/"}'
```

Notes :

* L’hôte canvas du Gateway expose `/__openclaw__/canvas/` et `/__openclaw__/a2ui/`.
* Le nœud iOS navigue automatiquement vers A2UI à la connexion lorsqu’une URL d’hôte canvas est annoncée.
* Revenez à la structure intégrée par défaut avec `canvas.navigate` et `{"url":""}`.

<div id="canvas-eval-snapshot">
  ### Évaluation du canvas / capture instantanée
</div>

```bash
openclaw nodes invoke --node "iOS Node" --command canvas.eval --params '{"javaScript":"(() => { const {ctx} = window.__openclaw; ctx.clearRect(0,0,innerWidth,innerHeight); ctx.lineWidth=6; ctx.strokeStyle=\"#ff2d55\"; ctx.beginPath(); ctx.moveTo(40,40); ctx.lineTo(innerWidth-40, innerHeight-40); ctx.stroke(); return \"ok\"; })()"}'
```

```bash
openclaw nodes invoke --node "iOS Node" --command canvas.snapshot --params '{"maxWidth":900,"format":"jpeg"}'
```

<div id="voice-wake-talk-mode">
  ## Réveil vocal + mode conversation
</div>

* Le réveil vocal et le mode conversation sont disponibles dans Paramètres.
* iOS peut suspendre l’audio en arrière-plan ; considérez les fonctionnalités vocales comme fournies sans garantie lorsqu’une l’application n’est pas active.

<div id="common-errors">
  ## Erreurs courantes
</div>

* `NODE_BACKGROUND_UNAVAILABLE` : mettez l’app iOS au premier plan (les commandes `canvas`/`camera`/`screen` l’exigent).
* `A2UI_HOST_NOT_CONFIGURED` : le Gateway n’a pas annoncé d’URL d’hôte canvas ; vérifiez `canvasHost` dans la [configuration du Gateway](/fr/gateway/configuration).
* La demande d’appairage n’apparaît jamais : exécutez `openclaw nodes pending` et approuvez manuellement.
* La reconnexion échoue après réinstallation : le jeton d’appairage dans le Trousseau a été effacé ; appairez à nouveau le nœud.

<div id="related-docs">
  ## Documentation associée
</div>

* [Appairage](/fr/gateway/pairing)
* [Découverte](/fr/gateway/discovery)
* [Bonjour](/fr/gateway/bonjour)