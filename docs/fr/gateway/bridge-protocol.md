---
title: Protocole Bridge
summary: "Protocole Bridge (nœuds hérités) : JSONL sur TCP, appairage, RPC à portée limitée"
read_when:
  - Développement ou débogage de clients de nœud (mode nœud iOS/Android/macOS)
  - Analyse des échecs d'appairage ou d'authentification Bridge
  - Audit de la surface de nœud exposée par le Gateway
---

<div id="bridge-protocol-legacy-node-transport">
  # Protocole Bridge (ancien transport de nœud)
</div>

Le protocole Bridge est un ancien transport de nœud (TCP JSONL). Les nouveaux clients de nœud
doivent utiliser à la place le protocole WebSocket unifié du Gateway.

Si vous développez un operator ou un client de nœud, utilisez le
[protocole Gateway](/fr/gateway/protocol).

**Remarque :** les versions actuelles d’OpenClaw n’incluent plus l’écouteur TCP du bridge ; ce document est conservé à titre de référence historique.
Les clés de configuration héritées `bridge.*` ne font plus partie du schéma de configuration.

<div id="why-we-have-both">
  ## Pourquoi nous avons les deux
</div>

* **Périmètre de sécurité** : le bridge expose une petite liste d’autorisation au lieu de
  toute la surface de l’API du Gateway.
* **Appairage + identité de nœud** : l’admission des nœuds est gérée par le Gateway et liée
  à un jeton par nœud.
* **UX de découverte** : les nœuds peuvent découvrir des Gateways via Bonjour sur le LAN, ou se connecter
  directement via un tailnet.
* **WS loopback** : le plan de contrôle WS complet reste local, sauf s’il est tunnelisé via SSH.

<div id="transport">
  ## Transport
</div>

* TCP, un objet JSON par ligne (JSONL).
* TLS facultatif (lorsque `bridge.tls.enabled` est true).
* Le port d&#39;écoute par défaut historique était `18790` (les versions actuelles ne démarrent plus de bridge TCP).

Lorsque TLS est activé, les enregistrements TXT de découverte incluent `bridgeTls=1` ainsi que
`bridgeTlsSha256`, afin que les nœuds puissent effectuer le pinning du certificat.

<div id="handshake-pairing">
  ## Handshake + appairage
</div>

1. Le client envoie `hello` avec les métadonnées du nœud et un jeton (si déjà appairé).
2. Si le client n’est pas appairé, le Gateway répond `error` (`NOT_PAIRED`/`UNAUTHORIZED`).
3. Le client envoie `pair-request`.
4. Le Gateway attend l’approbation, puis envoie `pair-ok` et `hello-ok`.

`hello-ok` renvoie `serverName` et peut inclure `canvasHostUrl`.

<div id="frames">
  ## Trames
</div>

Client → Gateway :

* `req` / `res` : RPC du Gateway avec portée définie (chat, sessions, config, health, voicewake, skills.bins)
* `event` : signaux du nœud (transcription vocale, requête d’agent, abonnement au chat, cycle de vie d’exécution)

Gateway → Client :

* `invoke` / `invoke-res` : commandes du nœud (`canvas.*`, `camera.*`, `screen.record`,
  `location.get`, `sms.send`)
* `event` : mises à jour de chat pour les sessions auxquelles le client est abonné
* `ping` / `pong` : maintien de la connexion

L’ancienne logique d’application de la liste d’autorisation se trouvait dans `src/gateway/server-bridge.ts` (supprimée).

<div id="exec-lifecycle-events">
  ## Événements de cycle de vie d’exécution
</div>

Les nœuds peuvent émettre des événements `exec.finished` ou `exec.denied` pour signaler l’activité `system.run`.
Ceux‑ci sont associés à des événements système dans le Gateway. (Les anciens nœuds peuvent encore émettre `exec.started`.)

Champs du payload (tous optionnels sauf indication contraire) :

* `sessionKey` (obligatoire) : session d’agent qui doit recevoir l’événement système.
* `runId` : identifiant d’exécution unique pour le regroupement.
* `command` : chaîne de commande brute ou formatée.
* `exitCode`, `timedOut`, `success`, `output` : détails d’achèvement (uniquement pour `exec.finished`).
* `reason` : raison du refus (uniquement pour `exec.denied`).

<div id="tailnet-usage">
  ## Utilisation d’un tailnet
</div>

* Associer le bridge à une adresse IP du tailnet : `bridge.bind: "tailnet"` dans
  `~/.openclaw/openclaw.json`.
* Les clients se connectent via le nom MagicDNS ou l’adresse IP du tailnet.
* Bonjour **ne** traverse **pas** les réseaux ; utilisez un hôte/port manuel ou le DNS‑SD global (wide-area)
  si nécessaire.

<div id="versioning">
  ## Gestion des versions
</div>

Bridge est actuellement en **v1 implicite** (sans négociation min/max). On part du principe que la rétrocompatibilité est assurée ; ajoute un champ de version du protocole Bridge avant tout changement cassant.