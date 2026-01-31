---
title: TypeBox
summary: "TypeBox-Schemata als zentrale Referenzquelle für das Gateway-Protokoll"
read_when:
  - Beim Aktualisieren von Protokollschemata oder bei der Codegenerierung
---

<div id="typebox-as-protocol-source-of-truth">
  # TypeBox als Protokoll-Single-Source-of-Truth
</div>

Zuletzt aktualisiert: 2026-01-10

TypeBox ist eine TypeScript-first-Schema-Bibliothek. Wir verwenden sie, um das **Gateway
WebSocket-Protokoll** (Handshake, Request/Response, Server-Events) zu definieren. Diese Schemas
steuern die **Laufzeitvalidierung**, den **JSON-Schema-Export** und die **Swift-Codegenerierung** für
die macOS-App. Eine einzige Single Source of Truth; alles andere wird generiert.

Wenn du Kontext zum Protokoll auf höherer Ebene suchst, starte mit
[Gateway-Architektur](/de/concepts/architecture).

<div id="mental-model-30-seconds">
  ## Mentales Modell (30 Sekunden)
</div>

Jede Gateway-WS-Nachricht ist einer von drei Frame-Typen:

* **Request**: `{ type: "req", id, method, params }`
* **Response**: `{ type: "res", id, ok, payload | error }`
* **Event**: `{ type: "event", event, payload, seq?, stateVersion? }`

Der erste Frame **muss** ein `connect`-Request sein. Danach können Clients
Methoden aufrufen (z. B. `health`, `send`, `chat.send`) und Events abonnieren (z. B.
`presence`, `tick`, `agent`).

Verbindungsablauf (minimal):

```
Client                    Gateway
  |---- req:connect -------->|
  |<---- res:hello-ok --------|
  |<---- event:tick ----------|
  |---- req:health ---------->|
  |<---- res:health ----------|
```

Gängige Methoden + Events:

| Kategorie | Beispiele                                                 | Hinweise                                  |
| --------- | --------------------------------------------------------- | ----------------------------------------- |
| Kern      | `connect`, `health`, `status`                             | `connect` muss zuerst kommen              |
| Messaging | `send`, `poll`, `agent`, `agent.wait`                     | Nebenwirkungen benötigen `idempotencyKey` |
| Chat      | `chat.history`, `chat.send`, `chat.abort`, `chat.inject`  | WebChat verwendet diese                   |
| Sitzungen | `sessions.list`, `sessions.patch`, `sessions.delete`      | Sitzungsverwaltung                        |
| Knoten    | `node.list`, `node.invoke`, `node.pair.*`                 | Gateway-WS- und Knoten-Aktionen           |
| Events    | `tick`, `presence`, `agent`, `chat`, `health`, `shutdown` | Server-Push                               |

Die verbindliche Liste befindet sich in `src/gateway/server.ts` (`METHODS`, `EVENTS`).

<div id="where-the-schemas-live">
  ## Wo die Schemas liegen
</div>

* Quelle: `src/gateway/protocol/schema.ts`
* Laufzeit-Validatoren (AJV): `src/gateway/protocol/index.ts`
* Server-Handshake + Methoden-Dispatch: `src/gateway/server.ts`
* Node-Client: `src/gateway/client.ts`
* Generiertes JSON-Schema: `dist/protocol.schema.json`
* Generierte Swift-Modelle: `apps/macos/Sources/OpenClawProtocol/GatewayModels.swift`

<div id="current-pipeline">
  ## Aktuelle Pipeline
</div>

* `pnpm protocol:gen`
  * schreibt JSON Schema (Draft‑07) in `dist/protocol.schema.json`
* `pnpm protocol:gen:swift`
  * generiert Swift-Gateway-Modelle
* `pnpm protocol:check`
  * führt beide Generatoren aus und überprüft, dass die Ausgabe committed wurde

<div id="how-the-schemas-are-used-at-runtime">
  ## Wie die Schemas zur Laufzeit verwendet werden
</div>

* **Serverseitig**: Jeder eingehende Frame wird mit AJV validiert. Der Handshake akzeptiert nur eine `connect`-Anfrage, deren Parameter `ConnectParams` entsprechen.
* **Clientseitig**: Der JS-Client validiert Ereignis- und Antwort-Frames, bevor er sie verwendet.
* **Methoden-Schnittstelle**: Das Gateway kündigt die unterstützten `methods` und `events` in `hello-ok` an.

<div id="example-frames">
  ## Beispiel-Frames
</div>

Connect (erste Nachricht):

```json
{
  "type": "req",
  "id": "c1",
  "method": "connect",
  "params": {
    "minProtocol": 2,
    "maxProtocol": 2,
    "client": {
      "id": "openclaw-macos",
      "displayName": "macos",
      "version": "1.0.0",
      "platform": "macos 15.1",
      "mode": "ui",
      "instanceId": "A1B2"
    }
  }
}
```

„Hello-ok“-Antwort:

```json
{
  "type": "res",
  "id": "c1",
  "ok": true,
  "payload": {
    "type": "hello-ok",
    "protocol": 2,
    "server": { "version": "dev", "connId": "ws-1" },
    "features": { "methods": ["health"], "events": ["tick"] },
    "snapshot": { "presence": [], "health": {}, "stateVersion": { "presence": 0, "health": 0 }, "uptimeMs": 0 },
    "policy": { "maxPayload": 1048576, "maxBufferedBytes": 1048576, "tickIntervalMs": 30000 }
  }
}
```

Anfrage + Antwort:

```json
{ "type": "req", "id": "r1", "method": "health" }
```

```json
{ "type": "res", "id": "r1", "ok": true, "payload": { "ok": true } }
```

Event:

```json
{ "type": "event", "event": "tick", "payload": { "ts": 1730000000 }, "seq": 12 }
```

<div id="minimal-client-nodejs">
  ## Minimaler Client (Node.js)
</div>

Kleinstmöglicher sinnvoller Ablauf: connect + health.

```ts
import { WebSocket } from "ws";

const ws = new WebSocket("ws://127.0.0.1:18789");

ws.on("open", () => {
  ws.send(JSON.stringify({
    type: "req",
    id: "c1",
    method: "connect",
    params: {
      minProtocol: 3,
      maxProtocol: 3,
      client: {
        id: "cli",
        displayName: "example",
        version: "dev",
        platform: "node",
        mode: "cli"
      }
    }
  }));
});

ws.on("message", (data) => {
  const msg = JSON.parse(String(data));
  if (msg.type === "res" && msg.id === "c1" && msg.ok) {
    ws.send(JSON.stringify({ type: "req", id: "h1", method: "health" }));
  }
  if (msg.type === "res" && msg.id === "h1") {
    console.log("health:", msg.payload);
    ws.close();
  }
});
```

<div id="worked-example-add-a-method-endtoend">
  ## Durchgearbeitetes Beispiel: eine Methode end‑to‑end hinzufügen
</div>

Beispiel: Füge eine neue `system.echo`‑Request hinzu, die `{ ok: true, text }` zurückgibt.

1. **Schema (maßgebliche Quelle)**

Füge Folgendes in `src/gateway/protocol/schema.ts` ein:

```ts
export const SystemEchoParamsSchema = Type.Object(
  { text: NonEmptyString },
  { additionalProperties: false },
);

export const SystemEchoResultSchema = Type.Object(
  { ok: Type.Boolean(), text: NonEmptyString },
  { additionalProperties: false },
);
```

Füge beides zu `ProtocolSchemas` hinzu und exportiere die entsprechenden Typen:

```ts
  SystemEchoParams: SystemEchoParamsSchema,
  SystemEchoResult: SystemEchoResultSchema,
```

```ts
export type SystemEchoParams = Static<typeof SystemEchoParamsSchema>;
export type SystemEchoResult = Static<typeof SystemEchoResultSchema>;
```

2. **Validierung**

In `src/gateway/protocol/index.ts`, exportiere einen AJV-Validator:

```ts
export const validateSystemEchoParams =
  ajv.compile<SystemEchoParams>(SystemEchoParamsSchema);
```

3. **Server-Verhalten**

Fügen Sie einen Handler in `src/gateway/server-methods/system.ts` hinzu:

```ts
export const systemHandlers: GatewayRequestHandlers = {
  "system.echo": ({ params, respond }) => {
    const text = String(params.text ?? "");
    respond(true, { ok: true, text });
  },
};
```

Registriere es in `src/gateway/server-methods.ts` (dort werden `systemHandlers` bereits zusammengeführt) und füge anschließend `"system.echo"` zu `METHODS` in `src/gateway/server.ts` hinzu.

4. **Neu generieren**

```bash
pnpm protocol:check
```

5. **Tests + Dokumentation**

Füge einen Servertest in `src/gateway/server.*.test.ts` hinzu und dokumentiere die Methode in der Dokumentation.

<div id="swift-codegen-behavior">
  ## Swift-Codegenerierungsverhalten
</div>

Der Swift-Generator erzeugt:

* `GatewayFrame`-Enum mit den Fällen `req`, `res`, `event` und `unknown`
* Stark typisierte Payload-Structs/-Enums
* `ErrorCode`-Werte und `GATEWAY_PROTOCOL_VERSION`

Unbekannte Frame-Typen werden für die Vorwärtskompatibilität als rohe Payloads beibehalten.

<div id="versioning-compatibility">
  ## Versionierung + Kompatibilität
</div>

* `PROTOCOL_VERSION` befindet sich in `src/gateway/protocol/schema.ts`.
* Clients senden `minProtocol` + `maxProtocol`; der Server lehnt Verbindungen mit inkompatiblen Versionen ab.
* Die Swift-Modelle behalten unbekannte Frame-Typen bei, um die Kompatibilität mit älteren Clients zu erhalten.

<div id="schema-patterns-and-conventions">
  ## Schema-Patterns und Konventionen
</div>

* Die meisten Objekte verwenden `additionalProperties: false` für strikt validierte Payloads.
* `NonEmptyString` ist der Standard für IDs sowie Methoden- und Ereignisnamen.
* Das Top-Level-`GatewayFrame` verwendet einen **Discriminator** für `type`.
* Methoden mit Nebeneffekten erfordern üblicherweise einen `idempotencyKey` in den Parametern
  (Beispiel: `send`, `poll`, `agent`, `chat.send`).

<div id="live-schema-json">
  ## Live-JSON-Schema
</div>

Das generierte JSON-Schema befindet sich im Repository unter `dist/protocol.schema.json`. Die
veröffentlichte Rohdatei ist in der Regel verfügbar unter:

* https://raw.githubusercontent.com/openclaw/openclaw/main/dist/protocol.schema.json

<div id="when-you-change-schemas">
  ## Wenn du Schemata änderst
</div>

1. Aktualisiere die TypeBox-Schemata.
2. Führe `pnpm protocol:check` aus.
3. Committe das regenerierte Schema und die Swift-Modelle.