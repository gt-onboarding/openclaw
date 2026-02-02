---
title: Typebox
summary: "Les schémas TypeBox comme source de vérité unique pour le protocole Gateway"
read_when:
  - Lors de la mise à jour des schémas de protocole ou du codegen
---

<div id="typebox-as-protocol-source-of-truth">
  # TypeBox comme source de vérité du protocole
</div>

Dernière mise à jour : 2026-01-10

TypeBox est une bibliothèque de schémas pour TypeScript. Nous l’utilisons pour définir le **protocole WebSocket du Gateway** (handshake, requête/réponse, événements serveur). Ces schémas pilotent la **validation à l’exécution**, l’**export en JSON Schema** et la **génération de code Swift** pour l’application macOS. Une seule source de vérité ; tout le reste est généré.

Si vous voulez un contexte plus haut niveau sur le protocole, commencez par
[Gateway architecture](/fr/concepts/architecture).

<div id="mental-model-30-seconds">
  ## Modèle mental (30 secondes)
</div>

Chaque message WS de Gateway est l’une des trois trames suivantes :

* **Request** : `{ type: "req", id, method, params }`
* **Response** : `{ type: "res", id, ok, payload | error }`
* **Event** : `{ type: "event", event, payload, seq?, stateVersion? }`

La première trame **doit** être une requête `connect`. Ensuite, les clients peuvent appeler
des méthodes (par exemple `health`, `send`, `chat.send`) et s’abonner à des événements (par exemple
`presence`, `tick`, `agent`).

Flux de connexion (minimal) :

```
Client                    Gateway
  |---- req:connect -------->|
  |<---- res:hello-ok --------|
  |<---- event:tick ----------|
  |---- req:health ---------->|
  |<---- res:health ----------|
```

Méthodes et événements courants :

| Catégorie  | Exemples                                                  | Notes                                           |
| ---------- | --------------------------------------------------------- | ----------------------------------------------- |
| Core       | `connect`, `health`, `status`                             | `connect` doit être le premier                  |
| Messaging  | `send`, `poll`, `agent`, `agent.wait`                     | les effets de bord nécessitent `idempotencyKey` |
| Chat       | `chat.history`, `chat.send`, `chat.abort`, `chat.inject`  | WebChat les utilise                             |
| Sessions   | `sessions.list`, `sessions.patch`, `sessions.delete`      | administration des sessions                     |
| Nœuds      | `node.list`, `node.invoke`, `node.pair.*`                 | actions WS Gateway + nœud                       |
| Événements | `tick`, `presence`, `agent`, `chat`, `health`, `shutdown` | push serveur                                    |

La liste de référence se trouve dans `src/gateway/server.ts` (`METHODS`, `EVENTS`).

<div id="where-the-schemas-live">
  ## Emplacement des schémas
</div>

* Source : `src/gateway/protocol/schema.ts`
* Validateurs à l’exécution (AJV) : `src/gateway/protocol/index.ts`
* Handshake serveur + répartition des méthodes : `src/gateway/server.ts`
* Client de nœud : `src/gateway/client.ts`
* Schéma JSON généré : `dist/protocol.schema.json`
* Modèles Swift générés : `apps/macos/Sources/OpenClawProtocol/GatewayModels.swift`

<div id="current-pipeline">
  ## Pipeline actuel
</div>

* `pnpm protocol:gen`
  * écrit le schéma JSON (draft‑07) dans `dist/protocol.schema.json`
* `pnpm protocol:gen:swift`
  * génère les modèles Swift pour le Gateway
* `pnpm protocol:check`
  * exécute les deux générateurs et vérifie que la sortie est bien commise dans le dépôt

<div id="how-the-schemas-are-used-at-runtime">
  ## Comment les schémas sont utilisés à l&#39;exécution
</div>

* **Côté serveur** : chaque trame entrante est validée avec AJV. Le handshake
  n&#39;accepte qu&#39;une requête `connect` dont les paramètres correspondent à `ConnectParams`.
* **Côté client** : le client JavaScript valide les trames d&#39;événements et de réponses avant
  de les utiliser.
* **Surface des méthodes** : le Gateway annonce les `methods` et `events` pris en charge dans `hello-ok`.

<div id="example-frames">
  ## Exemples de trames
</div>

Connexion (premier message) :

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

Réponse « Hello-ok » :

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

Requête et réponse :

```json
{ "type": "req", "id": "r1", "method": "health" }
```

```json
{ "type": "res", "id": "r1", "ok": true, "payload": { "ok": true } }
```

Événement :

```json
{ "type": "event", "event": "tick", "payload": { "ts": 1730000000 }, "seq": 12 }
```

<div id="minimal-client-nodejs">
  ## Client minimal (Node.js)
</div>

Flux minimal utile : connexion + vérification d&#39;état.

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
  ## Exemple concret : ajouter une méthode de bout en bout
</div>

Exemple : ajouter une nouvelle requête `system.echo` qui renvoie `{ ok: true, text }`.

1. **Schéma (source de vérité)**

Ajoutez à `src/gateway/protocol/schema.ts` :

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

Ajoutez les deux à `ProtocolSchemas` et exportez les types :

```ts
  SystemEchoParams: SystemEchoParamsSchema,
  SystemEchoResult: SystemEchoResultSchema,
```

```ts
export type SystemEchoParams = Static<typeof SystemEchoParamsSchema>;
export type SystemEchoResult = Static<typeof SystemEchoResultSchema>;
```

2. **Validation**

Dans `src/gateway/protocol/index.ts`, exportez un validateur AJV :

```ts
export const validateSystemEchoParams =
  ajv.compile<SystemEchoParams>(SystemEchoParamsSchema);
```

3. **Comportement du serveur**

Ajoutez un gestionnaire dans `src/gateway/server-methods/system.ts` :

```ts
export const systemHandlers: GatewayRequestHandlers = {
  "system.echo": ({ params, respond }) => {
    const text = String(params.text ?? "");
    respond(true, { ok: true, text });
  },
};
```

Déclarez-le dans `src/gateway/server-methods.ts` (qui fusionne déjà `systemHandlers`),
puis ajoutez `"system.echo"` à `METHODS` dans `src/gateway/server.ts`.

4. **Régénérer**

```bash
pnpm protocol:check
```

5. **Tests + docs**

Ajoutez un test de serveur dans `src/gateway/server.*.test.ts` et documentez la méthode.

<div id="swift-codegen-behavior">
  ## Comportement du générateur Swift
</div>

Le générateur Swift émet :

* une enum `GatewayFrame` avec les cas `req`, `res`, `event` et `unknown`
* des structs/enums de charges utiles fortement typées
* les valeurs `ErrorCode` et `GATEWAY_PROTOCOL_VERSION`

Les types de trames inconnus sont conservés en tant que charges utiles brutes pour assurer la compatibilité ascendante.

<div id="versioning-compatibility">
  ## Gestion de version + compatibilité
</div>

* `PROTOCOL_VERSION` se trouve dans `src/gateway/protocol/schema.ts`.
* Les clients envoient `minProtocol` + `maxProtocol` ; le serveur rejette les écarts de version.
* Les modèles Swift conservent les types de trames inconnus afin de ne pas rompre la compatibilité avec les anciens clients.

<div id="schema-patterns-and-conventions">
  ## Modèles et conventions de schéma
</div>

* La plupart des objets utilisent `additionalProperties: false` pour des charges utiles strictes.
* `NonEmptyString` est la valeur par défaut pour les identifiants et les noms de méthodes/événements.
* Le `GatewayFrame` de niveau supérieur utilise un **discriminateur** basé sur `type`.
* Les méthodes avec effets de bord requièrent généralement une `idempotencyKey` dans les paramètres
  (exemple : `send`, `poll`, `agent`, `chat.send`).

<div id="live-schema-json">
  ## Schéma JSON actuel
</div>

Le schéma JSON généré se trouve dans le dépôt, sous `dist/protocol.schema.json`. Le
fichier brut publié est généralement disponible à l’adresse suivante :

* https://raw.githubusercontent.com/openclaw/openclaw/main/dist/protocol.schema.json

<div id="when-you-change-schemas">
  ## Lorsque vous modifiez les schémas
</div>

1. Mettez à jour les schémas TypeBox.
2. Exécutez `pnpm protocol:check`.
3. Validez le schéma régénéré et les modèles Swift.