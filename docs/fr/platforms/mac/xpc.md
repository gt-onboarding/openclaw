---
title: Xpc
summary: "Architecture IPC macOS pour l’app OpenClaw, le transport du nœud Gateway et PeekabooBridge"
read_when:
  - Modification des contrats IPC ou de l’IPC de l’app de la barre de menus
---

<div id="openclaw-macos-ipc-architecture">
  # Architecture IPC d’OpenClaw sur macOS
</div>

**Modèle actuel :** un socket Unix local relie le **service hôte du nœud** à l’**application macOS** pour les approbations d’exécution + `system.run`. Un CLI de débogage `openclaw-mac` existe pour les vérifications de découverte/connexion ; les actions des agents passent toujours par le WebSocket du Gateway et `node.invoke`. L’automatisation de l’UI utilise PeekabooBridge.

<div id="goals">
  ## Objectifs
</div>

* Une seule instance d’application GUI qui gère toutes les interactions avec TCC (notifications, enregistrement d’écran, micro, synthèse vocale, AppleScript).
* Une surface minimale d’automatisation : commandes Gateway + nœud, plus PeekabooBridge pour l’automatisation de l’UI.
* Des autorisations prévisibles : toujours le même bundle ID signé, lancé par launchd, afin que les autorisations TCC soient préservées.

<div id="how-it-works">
  ## Principe de fonctionnement
</div>

<div id="gateway-node-transport">
  ### Transport entre le Gateway et le nœud
</div>

* L’application exécute le Gateway (mode local) et s’y connecte en tant que nœud.
* Les actions des agents sont effectuées via `node.invoke` (par exemple `system.run`, `system.notify`, `canvas.*`).

<div id="node-service-app-ipc">
  ### Service de nœud + IPC de l’application
</div>

* Un service hôte de nœud sans interface graphique (headless) se connecte au WebSocket du Gateway.
* Les requêtes `system.run` sont transmises à l’application macOS via un socket Unix local.
* L’application exécute la commande dans le contexte de l’UI, demande une confirmation si nécessaire, puis renvoie la sortie.

Diagramme (SCI) :

```
Agent -> Gateway -> Node Service (WS)
                      |  IPC (UDS + token + HMAC + TTL)
                      v
                  Mac App (UI + TCC + system.run)
```

<div id="peekaboobridge-ui-automation">
  ### PeekabooBridge (automatisation de l’UI)
</div>

* L’automatisation de l’UI utilise un socket UNIX distinct nommé `bridge.sock` et le protocole JSON PeekabooBridge.
* Ordre de préférence des hôtes (côté client) : Peekaboo.app → Claude.app → OpenClaw.app → exécution locale.
* Sécurité : les hôtes du bridge exigent un TeamID autorisé ; l’échappatoire DEBUG limitée au même UID est contrôlée par `PEEKABOO_ALLOW_UNSIGNED_SOCKET_CLIENTS=1` (convention Peekaboo).
* Voir : [Utilisation de PeekabooBridge](/fr/platforms/mac/peekaboo) pour plus de détails.

<div id="operational-flows">
  ## Flux opérationnels
</div>

* Redémarrage/rebuild : `SIGN_IDENTITY="Apple Development: <Developer Name> (<TEAMID>)" scripts/restart-mac.sh`
  * Arrête les instances existantes
  * Compilation Swift + empaquetage
  * Écrit/initialise/lance le LaunchAgent
* Instance unique : l’application se ferme immédiatement si une autre instance avec le même bundle ID est en cours d’exécution.

<div id="hardening-notes">
  ## Notes de durcissement
</div>

* Privilégiez l’exigence d’une correspondance de TeamID pour toutes les surfaces privilégiées.
* PeekabooBridge : `PEEKABOO_ALLOW_UNSIGNED_SOCKET_CLIENTS=1` (uniquement en DEBUG) peut autoriser des appelants avec le même UID pour le développement local.
* Toutes les communications restent strictement locales ; aucun socket réseau n’est exposé.
* Les boîtes de dialogue TCC proviennent uniquement du bundle de l’application GUI ; conservez l’ID de bundle signé stable d’une compilation à l’autre.
* Durcissement de l’IPC : mode de socket `0600`, jeton, vérifications du UID du pair, défi/réponse HMAC, TTL court.