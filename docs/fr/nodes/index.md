---
title: Nœuds
summary: "Nœuds : appairage, fonctionnalités, permissions et assistants CLI pour canvas/camera/screen/system"
read_when:
  - Appairage de nœuds iOS/Android avec le Gateway
  - Utiliser le canvas/la caméra du nœud comme contexte pour l’agent
  - Ajouter de nouvelles commandes de nœud ou des assistants CLI
---

<div id="nodes">
  # Nœuds
</div>

Un **nœud** est un appareil compagnon (macOS/iOS/Android/sans interface) qui se connecte au **WebSocket** du Gateway (même port que les opérateurs) avec `role: "node"` et expose une interface de commande (par ex. `canvas.*`, `camera.*`, `system.*`) via `node.invoke`. Détails du protocole : [Protocole Gateway](/fr/gateway/protocol).

Transport hérité : [Protocole Bridge](/fr/gateway/bridge-protocol) (TCP JSONL ; obsolète/supprimé pour les nœuds actuels).

macOS peut également fonctionner en **mode nœud** : l’application de barre de menus se connecte au serveur WS du Gateway et expose ses commandes locales canvas/camera en tant que nœud (ainsi, `openclaw nodes …` fonctionne avec ce Mac).

Remarques :

* Les nœuds sont des **périphériques**, pas des Gateway. Ils n’exécutent pas le service Gateway.
* Les messages Telegram/WhatsApp/etc. arrivent sur le **Gateway**, pas sur les nœuds.

<div id="pairing-status">
  ## Appairage + statut
</div>

**Les nœuds WS utilisent l’appairage de périphériques.** Les nœuds présentent une identité de périphérique lors de `connect` ; le Gateway
crée une demande d’appairage de périphérique pour `role: node`. Approuvez-la via la CLI des périphériques (ou l’UI).

CLI rapide :

```bash
openclaw devices list
openclaw devices approve <requestId>
openclaw devices reject <requestId>
openclaw nodes status
openclaw nodes describe --node <idOrNameOrIp>
```

Remarques :

* `nodes status` marque un nœud comme **appairé** lorsque son rôle d’appairage d’appareil inclut `node`.
* `node.pair.*` (CLI : `openclaw nodes pending/approve/reject`) est un espace de stockage d’appairage de nœuds distinct, géré par le Gateway ; il ne conditionne **pas** le handshake WS `connect`.

<div id="remote-node-host-systemrun">
  ## Hôte de nœud distant (system.run)
</div>

Utilisez un **hôte de nœud** lorsque votre Gateway s’exécute sur une machine et que vous voulez que les commandes
s’exécutent sur une autre. Le modèle communique toujours avec le **Gateway** ; la Gateway
redirige les appels `exec` vers l’**hôte de nœud** lorsque `host=node` est sélectionné.

<div id="what-runs-where">
  ### Ce qui s’exécute où
</div>

* **Hôte Gateway** : reçoit les messages, exécute le modèle, achemine les appels d’outils.
* **Hôte nœud** : exécute `system.run`/`system.which` sur la machine du nœud.
* **Approbations** : sont appliquées sur l’hôte nœud via `~/.openclaw/exec-approvals.json`.

<div id="start-a-node-host-foreground">
  ### Démarrer un hôte de nœud (en avant-plan)
</div>

Sur la machine du nœud :

```bash
openclaw node run --host <gateway-host> --port 18789 --display-name "Build Node"
```

<div id="start-a-node-host-service">
  ### Démarrer l’hôte de nœud (service)
</div>

```bash
openclaw node install --host <gateway-host> --port 18789 --display-name "Build Node"
openclaw node restart
```

<div id="pair-name">
  ### Appairer + nommer
</div>

Sur l’hôte du Gateway :

```bash
openclaw nodes pending
openclaw nodes approve <requestId>
openclaw nodes list
```

Options d’attribution de noms :

* `--display-name` avec `openclaw node run` / `openclaw node install` (enregistré dans `~/.openclaw/node.json` sur le nœud).
* `openclaw nodes rename --node <id|name|ip> --name "Build Node"` (surcharge côté Gateway).

<div id="allowlist-the-commands">
  ### Ajouter les commandes à la liste d’autorisation
</div>

Les autorisations Exec sont **propres à chaque hôte de nœud**. Ajoutez des entrées à la liste d’autorisation depuis le Gateway :

```bash
openclaw approvals allowlist add --node <id|name|ip> "/usr/bin/uname"
openclaw approvals allowlist add --node <id|name|ip> "/usr/bin/sw_vers"
```

Les autorisations sont stockées sur l’hôte du nœud dans `~/.openclaw/exec-approvals.json`.

<div id="point-exec-at-the-node">
  ### Pointer exec vers le nœud
</div>

Configurez les valeurs par défaut (configuration du Gateway) :

```bash
openclaw config set tools.exec.host node
openclaw config set tools.exec.security allowlist
openclaw config set tools.exec.node "<id-or-name>"
```

Ou par session :

```
/exec host=node security=allowlist node=<id-or-name>
```

Une fois ce paramètre défini, tout appel `exec` avec `host=node` s’exécute sur l’hôte du nœud (sous réserve de la liste d’autorisation et des approbations du nœud).

Voir aussi :

* [CLI de l’hôte du nœud](/fr/cli/node)
* [Outil exec](/fr/tools/exec)
* [Approbations exec](/fr/tools/exec-approvals)

<div id="invoking-commands">
  ## Appels de commandes
</div>

Bas niveau (RPC brut) :

```bash
openclaw nodes invoke --node <idOrNameOrIp> --command canvas.eval --params '{"javaScript":"location.href"}'
```

Des fonctions utilitaires de plus haut niveau sont disponibles pour les workflows courants consistant à « fournir à l’agent une pièce jointe MEDIA ».

<div id="screenshots-canvas-snapshots">
  ## Captures d’écran (instantanés du Canvas)
</div>

Si le node affiche le Canvas (WebView), `canvas.snapshot` renvoie `{ format, base64 }`.

Aide CLI (enregistre dans un fichier temporaire et affiche `MEDIA:<path>`) :

```bash
openclaw nodes canvas snapshot --node <idOrNameOrIp> --format png
openclaw nodes canvas snapshot --node <idOrNameOrIp> --format jpg --max-width 1200 --quality 0.9
```

<div id="canvas-controls">
  ### Commandes du canevas
</div>

```bash
openclaw nodes canvas present --node <idOrNameOrIp> --target https://example.com
openclaw nodes canvas hide --node <idOrNameOrIp>
openclaw nodes canvas navigate https://example.com --node <idOrNameOrIp>
openclaw nodes canvas eval --node <idOrNameOrIp> --js "document.title"
```

Notes :

* `canvas present` accepte des URL ou des chemins de fichiers locaux (`--target`), ainsi que des options facultatives `--x/--y/--width/--height` pour le positionnement.
* `canvas eval` accepte du JavaScript en ligne (`--js`) ou un argument positionnel.

<div id="a2ui-canvas">
  ### A2UI (Canvas)
</div>

```bash
openclaw nodes canvas a2ui push --node <idOrNameOrIp> --text "Bonjour"
openclaw nodes canvas a2ui push --node <idOrNameOrIp> --jsonl ./payload.jsonl
openclaw nodes canvas a2ui reset --node <idOrNameOrIp>
```

Remarques :

* Seul le format JSONL A2UI v0.8 est pris en charge (v0.9/createSurface est rejeté).

<div id="photos-videos-node-camera">
  ## Photos + vidéos (caméra du nœud)
</div>

Photos (`jpg`) :

```bash
openclaw nodes camera list --node <idOrNameOrIp>
openclaw nodes camera snap --node <idOrNameOrIp>            # par défaut : les deux orientations (2 lignes MEDIA)
openclaw nodes camera snap --node <idOrNameOrIp> --facing front
```

Enregistrements vidéo (`mp4`) :

```bash
openclaw nodes camera clip --node <idOrNameOrIp> --duration 10s
openclaw nodes camera clip --node <idOrNameOrIp> --duration 3000 --no-audio
```

Notes :

* Le nœud doit être **au premier plan** pour `canvas.*` et `camera.*` (les appels en arrière-plan renvoient `NODE_BACKGROUND_UNAVAILABLE`).
* La durée du clip est limitée (actuellement `<= 60s`) afin d’éviter des charges utiles base64 trop volumineuses.
* Android demandera les autorisations `CAMERA`/`RECORD_AUDIO` lorsque c’est possible ; si elles sont refusées, ces opérations échouent avec `*_PERMISSION_REQUIRED`.

<div id="screen-recordings-nodes">
  ## Enregistrements d&#39;écran (nœuds)
</div>

Les nœuds exposent `screen.record` (format mp4). Exemple :

```bash
openclaw nodes screen record --node <idOrNameOrIp> --duration 10s --fps 10
openclaw nodes screen record --node <idOrNameOrIp> --duration 10s --fps 10 --no-audio
```

Remarques :

* `screen.record` nécessite que l’application du nœud soit au premier plan.
* Android affichera l’invite système de capture d’écran avant l’enregistrement.
* Les enregistrements d’écran sont limités à `<= 60s`.
* `--no-audio` désactive la capture du microphone (pris en charge sur iOS/Android ; macOS utilise le système pour la capture audio).
* Utilisez `--screen <index>` pour sélectionner un écran quand plusieurs écrans sont disponibles.

<div id="location-nodes">
  ## Localisation (nœuds)
</div>

Les nœuds exposent `location.get` lorsque la fonctionnalité de localisation est activée dans les paramètres.

Aide de la CLI :

```bash
openclaw nodes location get --node <idOrNameOrIp>
openclaw nodes location get --node <idOrNameOrIp> --accuracy precise --max-age 15000 --location-timeout 10000
```

Remarques :

* La localisation est **désactivée par défaut**.
* « Toujours » nécessite une autorisation système ; la récupération en arrière-plan est effectuée sans garantie de résultat.
* La réponse inclut lat/lon, la précision (en mètres) et l’horodatage.

<div id="sms-android-nodes">
  ## SMS (nœuds Android)
</div>

Les nœuds Android peuvent exposer `sms.send` lorsque l’utilisateur accorde l’autorisation d’envoi de **SMS** et que l’appareil prend en charge la fonctionnalité téléphonique.

Appel de bas niveau :

```bash
openclaw nodes invoke --node <idOrNameOrIp> --command sms.send --params '{"to":"+15555550123","message":"Hello from OpenClaw"}'
```

Notes :

* La demande d’autorisation doit être acceptée sur l’appareil Android avant que la fonctionnalité ne soit annoncée.
* Les appareils uniquement Wi‑Fi dépourvus de téléphonie n’annonceront pas `sms.send`.

<div id="system-commands-node-host-mac-node">
  ## Commandes système (hôte du nœud / nœud macOS)
</div>

Le nœud macOS expose `system.run`, `system.notify` et `system.execApprovals.get/set`.
L&#39;hôte de nœud sans interface (headless) expose `system.run`, `system.which` et `system.execApprovals.get/set`.

Exemples :

```bash
openclaw nodes run --node <idOrNameOrIp> -- echo "Hello from mac node"
openclaw nodes notify --node <idOrNameOrIp> --title "Ping" --body "Gateway ready"
```

Remarques :

* `system.run` renvoie stdout/stderr/code de sortie dans la charge utile.
* `system.notify` respecte l’état d’autorisation des notifications dans l’app macOS.
* `system.run` prend en charge `--cwd`, `--env KEY=VAL`, `--command-timeout` et `--needs-screen-recording`.
* `system.notify` prend en charge `--priority <passive|active|timeSensitive>` et `--delivery <system|overlay|auto>`.
* Les nœuds macOS ignorent les surcharges de `PATH` ; les hôtes de nœud sans interface n’acceptent `PATH` que lorsqu’il vient préfixer le PATH de l’hôte de nœud.
* En mode nœud macOS, `system.run` est soumis aux approbations d’exécution dans l’app macOS (Settings → Exec approvals).
  Ask/allowlist/full se comportent de la même manière que sur l’hôte de nœud sans interface ; les demandes refusées renvoient `SYSTEM_RUN_DENIED`.
* Sur un hôte de nœud sans interface, `system.run` est soumis aux approbations d’exécution (`~/.openclaw/exec-approvals.json`).

<div id="exec-node-binding">
  ## Association du nœud exec
</div>

Lorsque plusieurs nœuds sont disponibles, vous pouvez associer `exec` à un nœud spécifique.
Cela définit le nœud par défaut pour `exec host=node` (et peut être surchargé pour chaque agent).

Valeur par défaut globale :

```bash
openclaw config set tools.exec.node "node-id-or-name"
```

Surcharge par agent :

```bash
openclaw config get agents.list
openclaw config set agents.list[0].tools.exec.node "node-id-or-name"
```

Laisser non défini pour autoriser n’importe quel nœud :

```bash
openclaw config unset tools.exec.node
openclaw config unset agents.list[0].tools.exec.node
```

<div id="permissions-map">
  ## Mappage des autorisations
</div>

Les nœuds peuvent inclure un mappage `permissions` dans `node.list` / `node.describe`, indexé par nom d’autorisation (par exemple `screenRecording`, `accessibility`) avec des valeurs booléennes (`true` = accordée).

<div id="headless-node-host-cross-platform">
  ## Hôte de nœud sans interface (headless) (multi‑plateforme)
</div>

OpenClaw peut exécuter un **hôte de nœud sans interface (headless)** (sans UI) qui se connecte au WS du Gateway
et expose `system.run` / `system.which`. C’est utile sur Linux/Windows
ou pour exécuter un nœud minimal sur le même hôte qu’un serveur.

Lancez‑le :

```bash
openclaw node run --host <gateway-host> --port 18789
```

Notes :

* L&#39;appairage reste requis (Gateway affichera une demande d’approbation du nœud).
* L’hôte du nœud stocke son ID de nœud, son jeton, son nom d’affichage et ses informations de connexion au Gateway dans `~/.openclaw/node.json`.
* Les autorisations d’exécution sont appliquées localement via `~/.openclaw/exec-approvals.json`
  (voir [Exec approvals](/fr/tools/exec-approvals)).
* Sur macOS, l’hôte de nœud sans interface privilégie l’hôte d’exécution de l’application compagnon lorsqu’il est joignable et revient à une exécution locale si l’application est indisponible. Définissez `OPENCLAW_NODE_EXEC_HOST=app` pour exiger l’application, ou `OPENCLAW_NODE_EXEC_FALLBACK=0` pour désactiver le repli.
* Ajoutez `--tls` / `--tls-fingerprint` lorsque le WS (WebSocket) du Gateway utilise TLS.

<div id="mac-node-mode">
  ## Mode nœud Mac
</div>

* L’application de barre de menus macOS se connecte au serveur WS du Gateway en tant que nœud (ce qui permet à `openclaw nodes …` de fonctionner sur ce Mac).
* En mode distant, l’app ouvre un tunnel SSH vers le port du Gateway et se connecte à `localhost`.