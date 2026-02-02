---
title: Protokoll
summary: "Gateway-WebSocket-Protokoll: Handshake, Frames, Versionierung"
read_when:
  - Implementierung oder Aktualisierung von Gateway-WS-Clients
  - Debugging von Protokollabweichungen oder Verbindungsfehlern
  - Regenerierung von Protokollschemata und -modellen
---

<div id="gateway-protocol-websocket">
  # Gateway-Protokoll (WebSocket)
</div>

Das Gateway-WS-Protokoll ist die **einzige Control-Plane- und Knoten-Transportschicht**
für OpenClaw. Alle Clients (CLI, Web-UI, macOS-App, iOS/Android-Knoten, headless-Knoten)
verbinden sich über WebSocket und geben beim Handshake ihre **role** + **scope** an.

<div id="transport">
  ## Transport
</div>

- WebSocket, Textframes mit JSON-Payloads.
- Der erste Frame **muss** eine `connect`-Anfrage sein.

<div id="handshake-connect">
  ## Handshake (Verbindungsaufbau)
</div>

Gateway → Client (Pre-Connect-Challenge):

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

Wenn ein Device-Token ausgestellt wird, enthält `hello-ok` zusätzlich:

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
  ### Knotenbeispiel
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

Methoden mit Seiteneffekten erfordern **Idempotenzschlüssel** (siehe Schema).

<div id="roles-scopes">
  ## Rollen + Scopes
</div>

<div id="roles">
  ### Rollen
</div>

- `operator` = Control-Plane-Client (CLI/UI/Automatisierung).
- `node` = Fähigkeitsknoten (camera/screen/canvas/system.run).

<div id="scopes-operator">
  ### Scopes (Operator)
</div>

Häufig verwendete Scopes:

- `operator.read`
- `operator.write`
- `operator.admin`
- `operator.approvals`
- `operator.pairing`

<div id="capscommandspermissions-node">
  ### Caps/commands/permissions (Knoten)
</div>

Knoten deklarieren ihre Fähigkeiten (Claims) beim Verbindungsaufbau:

- `caps`: übergeordnete Fähigkeitskategorien.
- `commands`: Allowlist der Befehle, die aufgerufen werden dürfen.
- `permissions`: feingranulare Schalter (z. B. `screen.record`, `camera.capture`).

Das Gateway behandelt diese als **Claims** und erzwingt serverseitige Allowlists.

<div id="presence">
  ## Präsenz
</div>

- `system-presence` gibt Einträge zurück, die nach Geräteidentität (als Schlüssel) organisiert sind.
- Präsenz-Einträge enthalten `deviceId`, `roles` und `scopes`, sodass UIs eine einzelne Zeile pro Gerät anzeigen können,
  selbst wenn ein Gerät sowohl als **operator** als auch als **node** verbunden ist.

<div id="node-helper-methods">
  ### Knoten-Hilfsmethoden
</div>

- Knoten können `skills.bins` aufrufen, um die aktuelle Liste der ausführbaren Skill-Binaries
  für Auto-Allow-Checks abzurufen.

<div id="exec-approvals">
  ## Exec-Genehmigungen
</div>

- Wenn eine Exec-Anfrage eine Genehmigung benötigt, broadcastet das Gateway `exec.approval.requested`.
- Operator-Clients erteilen die Genehmigung, indem sie `exec.approval.resolve` aufrufen (erfordert den Scope `operator.approvals`).

<div id="versioning">
  ## Versionierung
</div>

- `PROTOCOL_VERSION` befindet sich in `src/gateway/protocol/schema.ts`.
- Clients senden `minProtocol` + `maxProtocol`; der Server lehnt ab, wenn sie nicht zueinander passen.
- Schemata und Modelle werden aus TypeBox-Definitionen generiert:
  - `pnpm protocol:gen`
  - `pnpm protocol:gen:swift`
  - `pnpm protocol:check`

<div id="auth">
  ## Auth
</div>

- Wenn `OPENCLAW_GATEWAY_TOKEN` (oder `--token`) gesetzt ist, muss
  `connect.params.auth.token` übereinstimmen, sonst wird der Socket geschlossen.
- Nach der kopplung stellt das Gateway ein **Geräte-Token** aus, das auf die
  Verbindungsrolle + Scopes beschränkt ist. Es wird in
  `hello-ok.auth.deviceToken` zurückgegeben und sollte vom Client für
  zukünftige Verbindungen dauerhaft gespeichert werden.
- Geräte-Tokens können über `device.token.rotate` und `device.token.revoke`
  rotiert bzw. widerrufen werden (erfordert `operator.pairing` Scope).

<div id="device-identity-pairing">
  ## Geräteidentität + Kopplung
</div>

- Knoten sollten eine stabile Geräteidentität (`device.id`) angeben, die aus einem
  Schlüsselpaar-Fingerabdruck abgeleitet ist.
- Gateways geben pro Gerät und Rolle Token aus.
- Kopplungsbestätigungen sind für neue Geräte-IDs erforderlich, es sei denn, lokale
  automatische Genehmigung ist aktiviert.
- **Lokale** Verbindungen umfassen Loopback und die eigene Tailnet-Adresse des Gateway-Hosts
  (sodass Tailnet-Bindings auf demselben Host weiterhin automatisch genehmigt werden können).
- Alle WS-Clients müssen während `connect` eine `device`-Identität angeben (Operator + Knoten).
  Die Control UI kann sie **nur** weglassen, wenn `gateway.controlUi.allowInsecureAuth` aktiviert ist
  (oder `gateway.controlUi.dangerouslyDisableDeviceAuth` für Notfallzwecke).
- Nicht-lokale Verbindungen müssen die vom Server bereitgestellte `connect.challenge`-Nonce signieren.

<div id="tls-pinning">
  ## TLS + Pinning
</div>

- TLS wird für WS-Verbindungen unterstützt.
- Clients können optional den Fingerabdruck des Gateway-Zertifikats pinnen (siehe `gateway.tls`-
  Konfiguration sowie `gateway.remote.tlsFingerprint` oder CLI `--tls-fingerprint`).

<div id="scope">
  ## Geltungsbereich
</div>

Dieses Protokoll stellt die **vollständige Gateway-API** zur Verfügung (Status, Kanäle, Modelle, Chat,
Agent, Sitzungen, Knoten, Genehmigungen usw.). Die exakte API-Oberfläche ist durch die
TypeBox-Schemata in `src/gateway/protocol/schema.ts` definiert.