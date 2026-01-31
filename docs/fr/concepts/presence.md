---
title: Présence
summary: "Comment les entrées de présence OpenClaw sont produites, fusionnées et affichées"
read_when:
  - Débogage de l’onglet Instances
  - Analyse de lignes d’instances en double ou obsolètes
  - Modification des balises de connexion WS du Gateway ou des événements système
---

<div id="presence">
  # Présence
</div>

La « présence » OpenClaw est une vue légère, en mode *best effort*, de :

- le **Gateway** lui‑même, et
- les **clients connectés au Gateway** (app Mac, WebChat, CLI, etc.)

La présence est principalement utilisée pour afficher l’onglet **Instances** de l’application macOS et pour
offrir à l’opérateur une visibilité rapide.

<div id="presence-fields-what-shows-up">
  ## Champs de présence (ce qui apparaît)
</div>

Les entrées de présence sont des objets structurés avec des champs comme :

- `instanceId` (optionnel mais fortement recommandé) : identité stable du client (généralement `connect.client.instanceId`)
- `host` : nom d’hôte compréhensible pour un humain
- `ip` : adresse IP fournie au mieux (« best-effort »)
- `version` : chaîne de version du client
- `deviceFamily` / `modelIdentifier` : indications matérielles
- `mode` : `ui`, `webchat`, `cli`, `backend`, `probe`, `test`, `node`, ...
- `lastInputSeconds` : « secondes écoulées depuis la dernière saisie utilisateur » (si disponible)
- `reason` : `self`, `connect`, `node-connected`, `periodic`, ...
- `ts` : horodatage de dernière mise à jour (ms depuis l’époque Unix)

<div id="producers-where-presence-comes-from">
  ## Producteurs (origine de la présence)
</div>

Les entrées de présence proviennent de plusieurs sources et sont ensuite **fusionnées**.

<div id="1-gateway-self-entry">
  ### 1) Entrée « self » du Gateway
</div>

Le Gateway initialise toujours une entrée « self » au démarrage afin que les UI affichent l’hôte du Gateway
même avant que des clients ne s’y connectent.

<div id="2-websocket-connect">
  ### 2) Connexion WebSocket
</div>

Chaque client WS commence par une requête `connect`. Si le handshake réussit, le Gateway crée ou met à jour (upsert) une entrée de présence pour cette connexion.

<div id="why-oneoff-cli-commands-dont-show-up">
  #### Pourquoi les commandes CLI ponctuelles n’apparaissent pas
</div>

La CLI se connecte souvent pour des commandes ponctuelles et de courte durée. Pour éviter d’encombrer la liste des instances, `client.mode === "cli"` n’est **pas** enregistré comme entrée de présence.

<div id="3-system-event-beacons">
  ### 3) Balises `system-event`
</div>

Les clients peuvent émettre des balises périodiques plus détaillées via la méthode `system-event`. L'application macOS s’en sert pour signaler le nom d’hôte, l’adresse IP et `lastInputSeconds`.

<div id="4-node-connects-role-node">
  ### 4) Connexion du nœud (rôle : node)
</div>

Lorsqu’un nœud se connecte au Gateway via le WebSocket avec `role: node`, le Gateway
effectue un upsert d’une entrée de présence pour ce nœud (même logique que pour les autres clients WS).

<div id="merge-dedupe-rules-why-instanceid-matters">
  ## Règles de fusion + déduplication (pourquoi `instanceId` est important)
</div>

Les entrées de présence sont stockées dans une seule map en mémoire :

- Les entrées sont indexées par une **clé de présence**.
- La meilleure clé est un `instanceId` stable (provenant de `connect.client.instanceId`) qui survit aux redémarrages.
- Les clés sont insensibles à la casse.

Si un client se reconnecte sans `instanceId` stable, il peut apparaître comme une ligne
**dupliquée**.

<div id="ttl-and-bounded-size">
  ## TTL et taille bornée
</div>

La présence est intentionnellement éphémère :

- **TTL :** les entrées de plus de 5 minutes sont supprimées
- **Nombre maximal d’entrées :** 200 (les plus anciennes sont supprimées en premier)

Cela permet de garder la liste à jour et d’éviter une croissance non bornée de l’utilisation de la mémoire.

<div id="remotetunnel-caveat-loopback-ips">
  ## Mise en garde concernant les connexions distantes/tunnel (IP de loopback)
</div>

Lorsqu’un client se connecte via un tunnel SSH ou un transfert de port local, Gateway peut
voir l’adresse distante comme `127.0.0.1`. Pour éviter d’écraser une adresse IP correcte
fournie par le client, les adresses distantes en loopback sont ignorées.

<div id="consumers">
  ## Clients
</div>

<div id="macos-instances-tab">
  ### Onglet Instances macOS
</div>

L’application macOS affiche le résultat de `system-presence` et ajoute un indicateur d’état (Actif/Inactif/Obsolète) en fonction de l’ancienneté de la dernière mise à jour.

<div id="debugging-tips">
  ## Conseils de débogage
</div>

- Pour voir la liste brute, exécutez `system-presence` sur le Gateway.
- Si vous voyez des doublons :
  - vérifiez que les clients envoient un `client.instanceId` stable dans le handshake
  - vérifiez que les balises périodiques utilisent le même `instanceId`
  - vérifiez si l’entrée dérivée de la connexion n’a pas d’`instanceId` (les doublons sont alors prévus)