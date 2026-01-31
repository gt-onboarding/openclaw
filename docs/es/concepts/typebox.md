---
title: Typebox
summary: "Esquemas de TypeBox como única fuente de verdad del protocolo del Gateway"
read_when:
  - Al actualizar los esquemas de protocolo o el codegen
---

<div id="typebox-as-protocol-source-of-truth">
  # TypeBox como fuente de verdad del protocolo
</div>

Última actualización: 2026-01-10

TypeBox es una biblioteca de esquemas pensada específicamente para TypeScript. La usamos para definir el **protocolo WebSocket del Gateway** (handshake, request/response, eventos del servidor). Esos esquemas se usan para la **validación en tiempo de ejecución**, la **exportación a JSON Schema** y la **generación de código Swift** para la aplicación de macOS. Una única fuente de verdad; todo lo demás se genera.

Si quieres una visión más general del protocolo, empieza por
[Arquitectura de Gateway](/es/concepts/architecture).

<div id="mental-model-30-seconds">
  ## Modelo mental (30 segundos)
</div>

Cada mensaje WS del Gateway es uno de tres tipos de frames:

* **Request**: `{ type: "req", id, method, params }`
* **Response**: `{ type: "res", id, ok, payload | error }`
* **Event**: `{ type: "event", event, payload, seq?, stateVersion? }`

El primer frame **debe** ser una solicitud `connect`. Después de eso, los clientes pueden invocar
métodos (p. ej. `health`, `send`, `chat.send`) y suscribirse a eventos (p. ej.
`presence`, `tick`, `agent`).

Flujo de conexión (mínimo):

```
Client                    Gateway
  |---- req:connect -------->|
  |<---- res:hello-ok --------|
  |<---- event:tick ----------|
  |---- req:health ---------->|
  |<---- res:health ----------|
```

Métodos y eventos comunes:

| Category  | Examples                                                  | Notes                                                              |
| --------- | --------------------------------------------------------- | ------------------------------------------------------------------ |
| Core      | `connect`, `health`, `status`                             | `connect` must be first                                            |
| Messaging | `send`, `poll`, `agent`, `agent.wait`                     | las operaciones con efectos secundarios requieren `idempotencyKey` |
| Chat      | `chat.history`, `chat.send`, `chat.abort`, `chat.inject`  | WebChat usa estos                                                  |
| Sessions  | `sessions.list`, `sessions.patch`, `sessions.delete`      | administración de sesiones                                         |
| Nodes     | `node.list`, `node.invoke`, `node.pair.*`                 | WS del Gateway y acciones de nodo                                  |
| Events    | `tick`, `presence`, `agent`, `chat`, `health`, `shutdown` | envío desde el servidor                                            |

La lista oficial se encuentra en `src/gateway/server.ts` (`METHODS`, `EVENTS`).

<div id="where-the-schemas-live">
  ## Dónde se encuentran los esquemas
</div>

* Código fuente: `src/gateway/protocol/schema.ts`
* Validadores en tiempo de ejecución (AJV): `src/gateway/protocol/index.ts`
* Handshake del servidor + despacho de métodos: `src/gateway/server.ts`
* Cliente de nodo: `src/gateway/client.ts`
* JSON Schema generado: `dist/protocol.schema.json`
* Modelos Swift generados: `apps/macos/Sources/OpenClawProtocol/GatewayModels.swift`

<div id="current-pipeline">
  ## Flujo actual
</div>

* `pnpm protocol:gen`
  * escribe el esquema JSON (draft‑07) en `dist/protocol.schema.json`
* `pnpm protocol:gen:swift`
  * genera modelos de Gateway en Swift
* `pnpm protocol:check`
  * ejecuta ambos generadores y verifica que la salida se haya confirmado (commit) en el repositorio

<div id="how-the-schemas-are-used-at-runtime">
  ## Cómo se usan los esquemas en tiempo de ejecución
</div>

* **En el servidor**: cada frame entrante se valida con AJV. El handshake solo
  acepta una solicitud de `connect` cuyos parámetros coincidan con `ConnectParams`.
* **En el cliente**: el cliente JS valida los frames de eventos y respuestas antes de
  usarlos.
* **Interfaz de métodos**: el Gateway anuncia los `methods` y `events` que admite en `hello-ok`.

<div id="example-frames">
  ## Ejemplos de frames
</div>

Connect (primer mensaje):

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

Respuesta hello-ok:

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

Petición + respuesta:

```json
{ "type": "req", "id": "r1", "method": "health" }
```

```json
{ "type": "res", "id": "r1", "ok": true, "payload": { "ok": true } }
```

Evento:

```json
{ "type": "event", "event": "tick", "payload": { "ts": 1730000000 }, "seq": 12 }
```

<div id="minimal-client-nodejs">
  ## Cliente mínimo (Node.js)
</div>

Flujo mínimo útil: conectar + comprobación de estado.

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
  ## Ejemplo práctico: añadir un método de extremo a extremo
</div>

Ejemplo: añade una nueva solicitud `system.echo` que devuelve `{ ok: true, text }`.

1. **Esquema (fuente de la verdad)**

Añade lo siguiente en `src/gateway/protocol/schema.ts`:

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

Añade ambos a `ProtocolSchemas` y exporta sus tipos:

```ts
  SystemEchoParams: SystemEchoParamsSchema,
  SystemEchoResult: SystemEchoResultSchema,
```

```ts
export type SystemEchoParams = Static<typeof SystemEchoParamsSchema>;
export type SystemEchoResult = Static<typeof SystemEchoResultSchema>;
```

2. **Validación**

En `src/gateway/protocol/index.ts`, exporta un validador de AJV:

```ts
export const validateSystemEchoParams =
  ajv.compile<SystemEchoParams>(SystemEchoParamsSchema);
```

3. **Comportamiento del servidor**

Añade un controlador en `src/gateway/server-methods/system.ts`:

```ts
export const systemHandlers: GatewayRequestHandlers = {
  "system.echo": ({ params, respond }) => {
    const text = String(params.text ?? "");
    respond(true, { ok: true, text });
  },
};
```

Regístralo en `src/gateway/server-methods.ts` (ya integra `systemHandlers`),
luego agrega `"system.echo"` a `METHODS` en `src/gateway/server.ts`.

4. **Regenerar**

```bash
pnpm protocol:check
```

5. **Pruebas + documentación**

Añade una prueba de servidor en `src/gateway/server.*.test.ts` y documenta el método.

<div id="swift-codegen-behavior">
  ## Comportamiento de la generación de código Swift
</div>

El generador de Swift emite:

* Enum `GatewayFrame` con los casos `req`, `res`, `event` y `unknown`
* Structs/enums fuertemente tipados para los payloads
* Valores `ErrorCode` y `GATEWAY_PROTOCOL_VERSION`

Los tipos de frame desconocidos se conservan como payloads sin procesar para mantener la compatibilidad con versiones futuras.

<div id="versioning-compatibility">
  ## Versionado + compatibilidad
</div>

* `PROTOCOL_VERSION` se define en `src/gateway/protocol/schema.ts`.
* Los clientes envían `minProtocol` + `maxProtocol`; el servidor rechaza las incompatibilidades.
* Los modelos de Swift conservan tipos de frames desconocidos para evitar que los clientes antiguos dejen de funcionar.

<div id="schema-patterns-and-conventions">
  ## Patrones y convenciones de esquemas
</div>

* La mayoría de los objetos usan `additionalProperties: false` para cargas útiles estrictas.
* `NonEmptyString` es el valor predeterminado para IDs y nombres de métodos/eventos.
* El `GatewayFrame` de nivel superior usa un **discriminador** en `type`.
* Los métodos con efectos secundarios suelen requerir una `idempotencyKey` en los parámetros
  (ejemplo: `send`, `poll`, `agent`, `chat.send`).

<div id="live-schema-json">
  ## Esquema JSON en vivo
</div>

El esquema JSON generado se encuentra en el repositorio en `dist/protocol.schema.json`. El
archivo raw publicado suele estar disponible en:

* https://raw.githubusercontent.com/openclaw/openclaw/main/dist/protocol.schema.json

<div id="when-you-change-schemas">
  ## Cuando cambies esquemas
</div>

1. Actualiza los esquemas de TypeBox.
2. Ejecuta `pnpm protocol:check`.
3. Haz commit del esquema regenerado y de los modelos Swift.