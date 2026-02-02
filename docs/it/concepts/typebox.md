---
title: Typebox
summary: "Schemi TypeBox come fonte unica di verità per il protocollo del Gateway"
read_when:
  - Durante l'aggiornamento degli schemi di protocollo o del codegen
---

<div id="typebox-as-protocol-source-of-truth">
  # TypeBox come fonte di verità del protocollo
</div>

Ultimo aggiornamento: 2026-01-10

TypeBox è una libreria di schemi TypeScript-first. La utilizziamo per definire il **protocollo WebSocket del Gateway** (handshake, request/response, eventi del server). Quegli schemi alimentano la **validazione in fase di esecuzione**, l&#39;**esportazione in JSON Schema** e la **generazione di codice Swift** per
l&#39;app macOS. Un&#39;unica fonte di verità; tutto il resto è generato.

Se vuoi una panoramica del protocollo a un livello più alto, inizia da
[Architettura del Gateway](/it/concepts/architecture).

<div id="mental-model-30-seconds">
  ## Modello mentale (30 secondi)
</div>

Ogni messaggio WS del Gateway è uno di tre tipi di frame:

* **Request**: `{ type: "req", id, method, params }`
* **Response**: `{ type: "res", id, ok, payload | error }`
* **Event**: `{ type: "event", event, payload, seq?, stateVersion? }`

Il primo frame **deve** essere una richiesta `connect`. Dopodiché, i client possono chiamare
metodi (ad es. `health`, `send`, `chat.send`) e iscriversi agli eventi (ad es.
`presence`, `tick`, `agent`).

Flusso di connessione (di base):

```
Client                    Gateway
  |---- req:connect -------->|
  |<---- res:hello-ok --------|
  |<---- event:tick ----------|
  |---- req:health ---------->|
  |<---- res:health ----------|
```

Metodi + eventi comuni:

| Categoria | Esempi                                                    | Note                                                              |
| --------- | --------------------------------------------------------- | ----------------------------------------------------------------- |
| Core      | `connect`, `health`, `status`                             | `connect` deve essere il primo                                    |
| Messaging | `send`, `poll`, `agent`, `agent.wait`                     | le operazioni con effetti collaterali richiedono `idempotencyKey` |
| Chat      | `chat.history`, `chat.send`, `chat.abort`, `chat.inject`  | WebChat utilizza questi                                           |
| Sessions  | `sessions.list`, `sessions.patch`, `sessions.delete`      | amministrazione delle sessioni                                    |
| Nodes     | `node.list`, `node.invoke`, `node.pair.*`                 | azioni WS del Gateway e del nodo                                  |
| Events    | `tick`, `presence`, `agent`, `chat`, `health`, `shutdown` | push del server                                                   |

L&#39;elenco di riferimento si trova in `src/gateway/server.ts` (`METHODS`, `EVENTS`).

<div id="where-the-schemas-live">
  ## Dove si trovano gli schemi
</div>

* Sorgente: `src/gateway/protocol/schema.ts`
* Validator di runtime (AJV): `src/gateway/protocol/index.ts`
* Handshake del server + dispatch dei metodi: `src/gateway/server.ts`
* Client del nodo: `src/gateway/client.ts`
* JSON Schema generato: `dist/protocol.schema.json`
* Modelli Swift generati: `apps/macos/Sources/OpenClawProtocol/GatewayModels.swift`

<div id="current-pipeline">
  ## Pipeline attuale
</div>

* `pnpm protocol:gen`
  * scrive lo schema JSON (draft‑07) in `dist/protocol.schema.json`
* `pnpm protocol:gen:swift`
  * genera i modelli Swift per il Gateway
* `pnpm protocol:check`
  * esegue entrambi i generatori e verifica che l&#39;output sia stato committato

<div id="how-the-schemas-are-used-at-runtime">
  ## Come vengono utilizzati gli schemi a runtime
</div>

* **Lato server**: ogni frame in ingresso viene validato con AJV. L&#39;handshake
  accetta solo una richiesta `connect` i cui parametri corrispondono a `ConnectParams`.
* **Lato client**: il client JS convalida i frame di evento e di risposta prima
  di utilizzarli.
* **Interfaccia dei metodi**: il Gateway espone i `methods` e gli `events`
  supportati in `hello-ok`.

<div id="example-frames">
  ## Esempi di frame
</div>

Connect (primo messaggio):

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

Risposta «hello-ok»:

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

Richiesta e risposta:

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
  ## Client minimo (Node.js)
</div>

Flusso utile minimo: connessione + health check.

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
  ## Esempio pratico: aggiungere un metodo end‑to‑end
</div>

Esempio: aggiungi una nuova richiesta `system.echo` che restituisce `{ ok: true, text }`.

1. **Schema (fonte unica di verità)**

Aggiungi a `src/gateway/protocol/schema.ts`:

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

Aggiungi entrambi a `ProtocolSchemas` ed esportane i tipi:

```ts
  SystemEchoParams: SystemEchoParamsSchema,
  SystemEchoResult: SystemEchoResultSchema,
```

```ts
export type SystemEchoParams = Static<typeof SystemEchoParamsSchema>;
export type SystemEchoResult = Static<typeof SystemEchoResultSchema>;
```

2. **Validazione**

In `src/gateway/protocol/index.ts`, esporta un validatore AJV:

```ts
export const validateSystemEchoParams =
  ajv.compile<SystemEchoParams>(SystemEchoParamsSchema);
```

3. **Comportamento del server**

Aggiungi un handler in `src/gateway/server-methods/system.ts`:

```ts
export const systemHandlers: GatewayRequestHandlers = {
  "system.echo": ({ params, respond }) => {
    const text = String(params.text ?? "");
    respond(true, { ok: true, text });
  },
};
```

Registralo in `src/gateway/server-methods.ts` (che già include `systemHandlers`),
poi aggiungi `"system.echo"` a `METHODS` in `src/gateway/server.ts`.

4. **Rigenera**

```bash
pnpm protocol:check
```

5. **Test + documentazione**

Aggiungi un test del server in `src/gateway/server.*.test.ts` e documenta il metodo nella documentazione.

<div id="swift-codegen-behavior">
  ## Comportamento della generazione del codice Swift
</div>

Il generatore Swift emette:

* enum `GatewayFrame` con i casi `req`, `res`, `event` e `unknown`
* struct/enum di payload fortemente tipizzati
* valori `ErrorCode` e `GATEWAY_PROTOCOL_VERSION`

I tipi di frame non riconosciuti vengono preservati come payload grezzi per garantire la compatibilità futura.

<div id="versioning-compatibility">
  ## Versioning e compatibilità
</div>

* `PROTOCOL_VERSION` si trova in `src/gateway/protocol/schema.ts`.
* I client inviano `minProtocol` + `maxProtocol`; il server rifiuta i casi non compatibili.
* I modelli Swift mantengono i tipi di frame sconosciuti per non interrompere la compatibilità con i client meno recenti.

<div id="schema-patterns-and-conventions">
  ## Pattern e convenzioni degli schemi
</div>

* La maggior parte degli oggetti usa `additionalProperties: false` per payload con vincoli rigidi.
* `NonEmptyString` è il valore predefinito per ID e nomi di metodi/eventi.
* Il `GatewayFrame` di primo livello usa un **discriminatore** sul campo `type`.
* I metodi con effetti collaterali richiedono in genere una `idempotencyKey` nei parametri
  (esempio: `send`, `poll`, `agent`, `chat.send`).

<div id="live-schema-json">
  ## Schema JSON live
</div>

Lo schema JSON generato si trova nel repository in `dist/protocol.schema.json`. Il
file grezzo pubblicato è normalmente disponibile all&#39;indirizzo:

* https://raw.githubusercontent.com/openclaw/openclaw/main/dist/protocol.schema.json

<div id="when-you-change-schemas">
  ## Quando modifichi gli schemi
</div>

1. Aggiorna gli schemi TypeBox.
2. Esegui `pnpm protocol:check`.
3. Fai il commit dello schema rigenerato e dei modelli Swift.