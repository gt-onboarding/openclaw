---
title: Android
summary: "Application Android (nœud) : runbook de connexion + Canvas/Chat/Camera"
read_when:
  - Appairage ou reconnexion du nœud Android
  - Débogage de la découverte du Gateway ou de l’authentification depuis Android
  - Vérification de la cohérence de l’historique de conversation entre les différents clients
---

<div id="android-app-node">
  # Application Android (nœud)
</div>

<div id="support-snapshot">
  ## Résumé de prise en charge
</div>

* Rôle : application de nœud compagnon (Android n’héberge pas le Gateway).
* Gateway requis : oui (exécutez-le sur macOS, Linux ou Windows via WSL2).
* Installation : [Bien démarrer](/fr/start/getting-started) + [Appairage](/fr/gateway/pairing).
* Gateway : [Runbook](/fr/gateway) + [Configuration](/fr/gateway/configuration).
  * Protocoles : [Protocole du Gateway](/fr/gateway/protocol) (nœuds + plan de contrôle).

<div id="system-control">
  ## Contrôle système
</div>

Le contrôle système (launchd/systemd) réside sur l’hôte du Gateway. Voir [Gateway](/fr/gateway).

<div id="connection-runbook">
  ## Runbook de connexion
</div>

Application nœud Android ⇄ (mDNS/NSD + WebSocket) ⇄ **Gateway**

Android se connecte directement au Gateway via WebSocket (par défaut `ws://<host>:18789`) et utilise l&#39;appairage géré par le Gateway.

<div id="prerequisites">
  ### Prérequis
</div>

* Vous pouvez exécuter le Gateway sur la machine « master ».
* L’appareil/émulateur Android peut accéder au WebSocket du Gateway :
  * Même LAN avec mDNS/NSD, **ou**
  * Même tailnet Tailscale utilisant Wide-Area Bonjour / DNS-SD unicast (voir ci-dessous), **ou**
  * Hôte/port du Gateway saisi manuellement (solution de repli)
* Vous pouvez exécuter la CLI (`openclaw`) sur la machine du Gateway (ou via SSH).

<div id="1-start-the-gateway">
  ### 1) Démarrer le Gateway
</div>

```bash
openclaw gateway --port 18789 --verbose
```

Vérifiez dans les logs que vous voyez quelque chose comme :

* `listening on ws://0.0.0.0:18789`

Pour les configurations limitées au tailnet (recommandé pour Vienne ⇄ Londres), liez le Gateway à l’adresse IP du tailnet :

* Définissez `gateway.bind: "tailnet"` dans `~/.openclaw/openclaw.json` sur l’hôte du Gateway.
* Redémarrez le Gateway / l’app macOS de barre de menus.

<div id="2-verify-discovery-optional">
  ### 2) Vérifier la découverte (facultatif)
</div>

Depuis la machine Gateway :

```bash
dns-sd -B _openclaw-gw._tcp local.
```

Pour davantage de notes de débogage : [Bonjour](/fr/gateway/bonjour).

<div id="tailnet-vienna-london-discovery-via-unicast-dns-sd">
  #### Découverte Tailnet (Vienne ⇄ Londres) via DNS-SD unicast
</div>

La découverte NSD/mDNS sous Android ne traverse pas les réseaux. Si votre nœud Android et le Gateway se trouvent sur des réseaux différents mais sont connectés via Tailscale, utilisez plutôt Wide-Area Bonjour / DNS-SD unicast :

1. Configurez une zone DNS-SD (par exemple `openclaw.internal.`) sur l’hôte du Gateway et publiez des enregistrements `_openclaw-gw._tcp`.
2. Configurez le split DNS de Tailscale pour le domaine choisi en le faisant pointer vers ce serveur DNS.

Détails et exemple de configuration CoreDNS : [Bonjour](/fr/gateway/bonjour).

<div id="3-connect-from-android">
  ### 3) Se connecter depuis Android
</div>

Dans l&#39;application Android :

* L&#39;application maintient sa connexion au Gateway via un **service au premier plan** (notification persistante).
* Ouvrez **Settings**.
* Sous **Discovered Gateways**, sélectionnez votre Gateway et appuyez sur **Connect**.
* Si mDNS est bloqué, utilisez **Advanced → Manual Gateway** (hôte + port), puis **Connect (Manual)**.

Après le premier appairage réussi, Android se reconnecte automatiquement au lancement :

* au point de terminaison manuel (s&#39;il est activé), sinon
* au dernier Gateway découvert (en mode best-effort).

<div id="4-approve-pairing-cli">
  ### 4) Approuver l&#39;appairage (CLI)
</div>

Sur la machine qui héberge le Gateway :

```bash
openclaw nodes pending
openclaw nodes approve <requestId>
```

Détails de l’appairage : [Appairage au Gateway](/fr/gateway/pairing).

<div id="5-verify-the-node-is-connected">
  ### 5) Vérifiez que le nœud est connecté
</div>

* Via le statut des nœuds :
  ```bash
  openclaw nodes status
  ```
* Via Gateway :
  ```bash
  openclaw gateway call node.list --params "{}"
  ```

<div id="6-chat-history">
  ### 6) Chat + historique
</div>

Le volet Chat du nœud Android utilise la **clé de session principale** (`main`) du Gateway, donc l’historique et les réponses sont partagés avec WebChat et les autres clients :

* Historique : `chat.history`
* Envoyer : `chat.send`
* Mises à jour push (best-effort) : `chat.subscribe` → `event:"chat"`

<div id="7-canvas-camera">
  ### 7) Canvas + appareil photo
</div>

<div id="gateway-canvas-host-recommended-for-web-content">
  #### Hôte Canvas du Gateway (recommandé pour le contenu Web)
</div>

Si vous voulez que le nœud affiche du HTML/CSS/JS réel que l’agent peut modifier sur le disque, faites pointer le nœud vers l’hôte Canvas du Gateway.

Remarque : les nœuds utilisent l’hôte Canvas autonome sur `canvasHost.port` (par défaut `18793`).

1. Créez `~/.openclaw/workspace/canvas/index.html` sur l’hôte du Gateway.

2. Depuis le nœud, accédez-y (LAN) :

```bash
openclaw nodes invoke --node "<Android Node>" --command canvas.navigate --params '{"url":"http://<gateway-hostname>.local:18793/__openclaw__/canvas/"}'
```

Tailnet (facultatif) : si les deux appareils sont sur Tailscale, utilisez un nom MagicDNS ou une IP de Tailnet au lieu de `.local`, par exemple : `http://<gateway-magicdns>:18793/__openclaw__/canvas/`.

Ce serveur injecte un client de rechargement automatique dans le HTML et recharge à chaque modification de fichier.
L&#39;hôte A2UI est accessible à l&#39;adresse `http://<gateway-host>:18793/__openclaw__/a2ui/`.

Commandes Canvas (premier plan uniquement) :

* `canvas.eval`, `canvas.snapshot`, `canvas.navigate` (utilisez `{"url":""}` ou `{"url":"/"}` pour revenir au gabarit par défaut). `canvas.snapshot` renvoie `{ format, base64 }` (`format="jpeg"` par défaut).
* A2UI : `canvas.a2ui.push`, `canvas.a2ui.reset` (`canvas.a2ui.pushJSONL`, alias obsolète)

Commandes caméra (premier plan uniquement ; soumises aux autorisations) :

* `camera.snap` (jpg)
* `camera.clip` (mp4)

Voir [nœud Camera](/fr/nodes/camera) pour les paramètres et les utilitaires CLI.
