---
title: Protocole
summary: "Protocole WebSocket du Gateway : handshake, trames, gestion des versions"
read_when:
  - Lors de l’implémentation ou de la mise à jour de clients WS du Gateway
  - Pour déboguer des incompatibilités de protocole ou des échecs de connexion
  - Pour régénérer les schémas/modèles de protocole
---

<div id="gateway-protocol-websocket">
  # Protocole Gateway (WebSocket)
</div>

Le protocole WS de Gateway est le **seul plan de contrôle et transport des nœuds**
pour OpenClaw. Tous les clients (CLI, UI web, application macOS, nœuds iOS/Android,
nœuds headless) se connectent via WebSocket et déclarent leur **rôle** + **scope**
au moment de l’établissement de la connexion.

<div id="transport">
  ## Transport
</div>

- WebSocket, trames texte avec contenu JSON.
- La première trame **doit impérativement** être une requête `connect`.

<div id="handshake-connect">
  ## Handshake (connexion)
</div>

Gateway → Client (challenge de pré-connexion) :

```json
{
  "type": "event",
  "event": "connect.challenge",
  "payload": { "nonce": "…", "ts": 1737264000000 }
}
```

Client → Gateway :

```json
{
  "type": "req",
  "id": "…",
  "method": "connect",
  "params": {
    "minProtocol": 3,
    "maxProtocol": 3,
    "client": {
      "id": "cli",
      "version": "1.2.3",
      "platform": "macos",
      "mode": "operator"
    },
    "role": "operator",
    "scopes": ["operator.read", "operator.write"],
    "caps": [],
    "commands": [],
    "permissions": {},
    "auth": { "token": "…" },
    "locale": "en-US",
    "userAgent": "openclaw-cli/1.2.3",
    "device": {
      "id": "device_fingerprint",
      "publicKey": "…",
      "signature": "…",
      "signedAt": 1737264000000,
      "nonce": "…"
    }
  }
}
```

Gateway → Client :

```json
{
  "type": "res",
  "id": "…",
  "ok": true,
  "payload": { "type": "hello-ok", "protocol": 3, "policy": { "tickIntervalMs": 15000 } }
}
```

Lorsqu’un jeton d’appareil est délivré, `hello-ok` inclut également :

```json
{
  "auth": {
    "deviceToken": "…",
    "role": "operator",
    "scopes": ["operator.read", "operator.write"]
  }
}
```


<div id="node-example">
  ### Exemple de nœud
</div>

```json
{
  "type": "req",
  "id": "…",
  "method": "connect",
  "params": {
    "minProtocol": 3,
    "maxProtocol": 3,
    "client": {
      "id": "ios-node",
      "version": "1.2.3",
      "platform": "ios",
      "mode": "node"
    },
    "role": "node",
    "scopes": [],
    "caps": ["camera", "canvas", "screen", "location", "voice"],
    "commands": ["camera.snap", "canvas.navigate", "screen.record", "location.get"],
    "permissions": { "camera.capture": true, "screen.record": false },
    "auth": { "token": "…" },
    "locale": "en-US",
    "userAgent": "openclaw-ios/1.2.3",
    "device": {
      "id": "device_fingerprint",
      "publicKey": "…",
      "signature": "…",
      "signedAt": 1737264000000,
      "nonce": "…"
    }
  }
}
```


<div id="framing">
  ## Trame
</div>

- **Request** : `{type:"req", id, method, params}`  
- **Response** : `{type:"res", id, ok, payload|error}`  
- **Event** : `{type:"event", event, payload, seq?, stateVersion?}`

Les méthodes ayant des effets de bord exigent des **clés d'idempotence** (voir le schéma).

<div id="roles-scopes">
  ## Rôles + portées
</div>

<div id="roles">
  ### Rôles
</div>

- `operator` = client du plan de contrôle (CLI/UI/automatisation).
- `node` = hôte de capacités (camera/screen/canvas/system.run).

<div id="scopes-operator">
  ### Portées (opérateur)
</div>

Portées usuelles :

- `operator.read`
- `operator.write`
- `operator.admin`
- `operator.approvals`
- `operator.pairing`

<div id="capscommandspermissions-node">
  ### Capacités/commandes/autorisations (nœud)
</div>

Les nœuds déclarent leurs capacités au moment de l’établissement de la connexion :

- `caps` : catégories de capacités de haut niveau.
- `commands` : liste d’autorisation des commandes pouvant être invoquées.
- `permissions` : paramètres granulaires (par ex. `screen.record`, `camera.capture`).

Le Gateway les traite comme des **déclarations** et applique des listes d’autorisation côté serveur.

<div id="presence">
  ## Présence
</div>

- `system-presence` renvoie des entrées dont la clé est l’identité de l’appareil.
- Les entrées de présence incluent `deviceId`, `roles` et `scopes`, afin que les UI puissent afficher une seule ligne par appareil,
  même lorsqu’un appareil se connecte à la fois comme **operator** et comme **node**.

<div id="node-helper-methods">
  ### Méthodes utilitaires des nœuds
</div>

- Les nœuds peuvent appeler `skills.bins` pour récupérer la liste actuelle des exécutables de compétences
  pour les contrôles d’autorisation automatique.

<div id="exec-approvals">
  ## Approbations d'exécution
</div>

- Lorsqu'une requête d'exécution nécessite une approbation, le Gateway émet `exec.approval.requested`.
- Les clients opérateur la résolvent en appelant `exec.approval.resolve` (nécessite la portée `operator.approvals`).

<div id="versioning">
  ## Versionnement
</div>

- `PROTOCOL_VERSION` est défini dans `src/gateway/protocol/schema.ts`.
- Les clients envoient `minProtocol` + `maxProtocol` ; le serveur rejette en cas de non-correspondance.
- Les schémas et modèles sont générés à partir des définitions TypeBox :
  - `pnpm protocol:gen`
  - `pnpm protocol:gen:swift`
  - `pnpm protocol:check`

<div id="auth">
  ## Authentification
</div>

- Si `OPENCLAW_GATEWAY_TOKEN` (ou `--token`) est défini, `connect.params.auth.token`
  doit correspondre, faute de quoi la socket est fermée.
- Après l’appairage, le Gateway émet un **device token** restreint au rôle de la
  connexion et aux portées associées. Il est renvoyé dans `hello-ok.auth.deviceToken` et
  doit être conservé par le client pour les connexions futures.
- Les device tokens peuvent être rotés ou révoqués via `device.token.rotate` et
  `device.token.revoke` (nécessite la portée `operator.pairing`).

<div id="device-identity-pairing">
  ## Identité de l’appareil + appairage
</div>

- Les nœuds doivent inclure une identité d’appareil stable (`device.id`), dérivée
  de l’empreinte d’une paire de clés.
- Les Gateways émettent des jetons par appareil + rôle.
- Des approbations d’appairage sont requises pour les nouveaux IDs d’appareil, sauf
  si l’auto-approbation locale est activée.
- Les connexions **locales** incluent le loopback et l’adresse tailnet propre à
  l’hôte Gateway (de sorte que les liaisons tailnet sur le même hôte puissent
  toujours s’auto-approuver).
- Tous les clients WS doivent inclure l’identité `device` lors de `connect`
  (opérateur + nœud). Control UI peut l’omettre **uniquement** lorsque
  `gateway.controlUi.allowInsecureAuth` est activé (ou
  `gateway.controlUi.dangerouslyDisableDeviceAuth` pour un usage d’urgence
  « break-glass »).
- Les connexions non locales doivent signer le nonce `connect.challenge` fourni
  par le serveur.

<div id="tls-pinning">
  ## TLS + épinglage
</div>

- TLS est pris en charge pour les connexions WS.
- Les clients ont la possibilité d’épingler l’empreinte du certificat du Gateway (voir la configuration `gateway.tls`
  ainsi que `gateway.remote.tlsFingerprint` ou l’option CLI `--tls-fingerprint`).

<div id="scope">
  ## Portée
</div>

Ce protocole expose l’**API complète du Gateway** (statut, canaux, modèles, chat,
agent, sessions, nœuds, approbations, etc.). La surface exacte est définie par les
schémas TypeBox dans `src/gateway/protocol/schema.ts`.