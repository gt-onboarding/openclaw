---
title: Protocollo
summary: "Protocollo WebSocket del Gateway: handshake, frame, gestione delle versioni"
read_when:
  - Implementare o aggiornare client WS per il Gateway
  - Eseguire il debug di incompatibilità del protocollo o errori di connessione
  - Rigenerare gli schemi/modelli del protocollo
---

<div id="gateway-protocol-websocket">
  # Protocollo del Gateway (WebSocket)
</div>

Il protocollo WS del Gateway è l’**unico piano di controllo e canale di trasporto per i nodi** in
OpenClaw. Tutti i client (CLI, UI web, app macOS, nodi iOS/Android, nodi headless)
si connettono tramite WebSocket e dichiarano il proprio **role** + **scope** durante l’handshake iniziale.

<div id="transport">
  ## Trasporto
</div>

- WebSocket, frame testuali con payload JSON.
- Il primo frame **deve** essere una richiesta `connect`.

<div id="handshake-connect">
  ## Handshake (connessione)
</div>

Gateway → Client (challenge di pre-connessione):

```json
{
  "type": "event",
  "event": "connect.challenge",
  "payload": { "nonce": "…", "ts": 1737264000000 }
}
```

Client → Gateway:

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

Gateway → Client:

```json
{
  "type": "res",
  "id": "…",
  "ok": true,
  "payload": { "type": "hello-ok", "protocol": 3, "policy": { "tickIntervalMs": 15000 } }
}
```

Quando viene emesso un token per un dispositivo, `hello-ok` contiene anche:

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
  ### Esempio di nodo
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
  ## Framing
</div>

- **Request**: `{type:"req", id, method, params}`  
- **Response**: `{type:"res", id, ok, payload|error}`  
- **Event**: `{type:"event", event, payload, seq?, stateVersion?}`

I metodi che hanno effetti collaterali richiedono **chiavi di idempotenza** (vedi schema).

<div id="roles-scopes">
  ## Ruoli e scope
</div>

<div id="roles">
  ### Ruoli
</div>

- `operator` = client del control plane (CLI/UI/automazione).
- `node` = host delle funzionalità (camera/screen/canvas/system.run).

<div id="scopes-operator">
  ### Scope (operatore)
</div>

Scope comuni dell'operatore:

- `operator.read`
- `operator.write`
- `operator.admin`
- `operator.approvals`
- `operator.pairing`

<div id="capscommandspermissions-node">
  ### Capacità/comandi/permessi (nodo)
</div>

I nodi dichiarano le proprie capacità al momento della connessione:

- `caps`: categorie di capacità di alto livello.
- `commands`: lista di autorizzati dei comandi invocabili.
- `permissions`: flag granulari (ad es. `screen.record`, `camera.capture`).

Il Gateway interpreta questi elementi come **dichiarazioni** e applica liste di autorizzati lato server.

<div id="presence">
  ## Presenza
</div>

- `system-presence` restituisce voci indicizzate per identità del dispositivo.
- Le voci di presenza includono `deviceId`, `roles` e `scopes` in modo che le UI possano visualizzare una singola riga per dispositivo
  anche quando il dispositivo si connette sia come **operator** sia come **nodo**.

<div id="node-helper-methods">
  ### Metodi di supporto per i nodi
</div>

- I nodi possono invocare `skills.bins` per recuperare l'elenco corrente degli eseguibili delle abilità
  per eseguire i controlli di auto-allow.

<div id="exec-approvals">
  ## Approvazioni exec
</div>

- Quando una richiesta di exec richiede approvazione, il Gateway trasmette `exec.approval.requested`.
- I client dell’operatore risolvono chiamando `exec.approval.resolve` (richiede lo scope `operator.approvals`).

<div id="versioning">
  ## Versioning
</div>

- `PROTOCOL_VERSION` si trova in `src/gateway/protocol/schema.ts`.
- I client inviano `minProtocol` + `maxProtocol`; il server rifiuta le versioni non compatibili.
- Gli schemi e i modelli vengono generati dalle definizioni TypeBox:
  - `pnpm protocol:gen`
  - `pnpm protocol:gen:swift`
  - `pnpm protocol:check`

<div id="auth">
  ## Autenticazione
</div>

- Se `OPENCLAW_GATEWAY_TOKEN` (o `--token`) è impostato, `connect.params.auth.token`
  deve coincidere, altrimenti il socket viene chiuso.
- Dopo l'abbinamento, il Gateway rilascia un **device token** limitato in base al ruolo
  della connessione e agli scope. Viene restituito in `hello-ok.auth.deviceToken` e
  deve essere salvato dal client per le connessioni future.
- I device token possono essere ruotati o revocati tramite `device.token.rotate` e
  `device.token.revoke` (richiede lo scope `operator.pairing`).

<div id="device-identity-pairing">
  ## Identità del dispositivo + abbinamento
</div>

- I nodi devono includere un’identità di dispositivo stabile (`device.id`) derivata
  dall’impronta digitale di una coppia di chiavi.
- I Gateway emettono token per dispositivo + ruolo.
- Le approvazioni di abbinamento sono richieste per i nuovi ID dispositivo, a meno che
  l’auto-approvazione locale non sia abilitata.
- Le connessioni **locali** includono il loopback e l’indirizzo tailnet dell’host del Gateway stesso
  (quindi i binding tailnet sullo stesso host possono comunque auto-approvarsi).
- Tutti i client WS devono includere l’identità `device` durante `connect` (operatore + nodo).
  Il Control UI può ometterla **solo** quando `gateway.controlUi.allowInsecureAuth` è abilitato
  (oppure `gateway.controlUi.dangerouslyDisableDeviceAuth` per uso di emergenza).
- Le connessioni non locali devono firmare il nonce `connect.challenge` fornito dal server.

<div id="tls-pinning">
  ## TLS + pinning
</div>

- TLS è supportato per le connessioni WS.
- I client possono opzionalmente effettuare il pinning dell'impronta digitale del certificato del Gateway (vedi configurazione `gateway.tls` e `gateway.remote.tlsFingerprint` o l'opzione CLI `--tls-fingerprint`).

<div id="scope">
  ## Ambito
</div>

Questo protocollo espone le **API complete del Gateway** (stato, canali, modelli, chat,
agenti, sessioni, nodi, approvazioni, ecc.). L'esatta superficie esposta è definita dagli
schemi TypeBox in `src/gateway/protocol/schema.ts`.