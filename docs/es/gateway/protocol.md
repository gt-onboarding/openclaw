---
title: Protocolo
summary: "Protocolo WebSocket del Gateway: handshake, frames, versionado"
read_when:
  - Implementar o actualizar clientes WS del Gateway
  - Depurar incompatibilidades de protocolo o fallos de conexión
  - Regenerar el esquema y los modelos del protocolo
---

<div id="gateway-protocol-websocket">
  # Protocolo del Gateway (WebSocket)
</div>

El protocolo WS del Gateway es el **único plano de control + transporte de nodos**
para OpenClaw. Todos los clientes (CLI, UI web, app de macOS, nodos de iOS/Android,
nodos en modo headless) se conectan mediante WebSocket y declaran su **rol** + **ámbito**
durante el handshake inicial.

<div id="transport">
  ## Transporte
</div>

- WebSocket, frames de texto con contenido JSON.
- El primer frame **debe** ser una solicitud `connect`.

<div id="handshake-connect">
  ## Handshake (conexión)
</div>

Gateway → cliente (desafío de preconexión):

```json
{
  "type": "event",
  "event": "connect.challenge",
  "payload": { "nonce": "…", "ts": 1737264000000 }
}
```

Cliente → Gateway:

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

Gateway → cliente:

```json
{
  "type": "res",
  "id": "…",
  "ok": true,
  "payload": { "type": "hello-ok", "protocol": 3, "policy": { "tickIntervalMs": 15000 } }
}
```

Cuando se emite un token de dispositivo, `hello-ok` también incluye lo siguiente:

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
  ### Ejemplo de nodo
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
  ## Estructura
</div>

- **Solicitud**: `{type:"req", id, method, params}`  
- **Respuesta**: `{type:"res", id, ok, payload|error}`  
- **Evento**: `{type:"event", event, payload, seq?, stateVersion?}`

Los métodos que producen efectos secundarios requieren **claves de idempotencia** (consulta el esquema).

<div id="roles-scopes">
  ## Roles y ámbitos
</div>

<div id="roles">
  ### Roles
</div>

- `operator` = cliente del plano de control (CLI/UI/automatización).
- `node` = host de capacidades (cámara/pantalla/canvas/system.run).

<div id="scopes-operator">
  ### Ámbitos (operador)
</div>

Ámbitos comunes:

- `operator.read`
- `operator.write`
- `operator.admin`
- `operator.approvals`
- `operator.pairing`

<div id="capscommandspermissions-node">
  ### Capacidades/comandos/permisos (nodo)
</div>

Los nodos declaran sus capacidades cuando se conectan:

- `caps`: categorías de capacidades de alto nivel.
- `commands`: lista de permitidos de comandos que se pueden invocar.
- `permissions`: controles granulares (p. ej., `screen.record`, `camera.capture`).

El Gateway trata estos como **declaraciones** y aplica listas de permitidos en el lado del servidor.

<div id="presence">
  ## Presencia
</div>

- `system-presence` devuelve entradas con clave por identidad de dispositivo.
- Las entradas de presencia incluyen `deviceId`, `roles` y `scopes` para que las UIs puedan mostrar una sola fila por dispositivo,
  incluso cuando se conecte tanto como **operator** como **nodo**.

<div id="node-helper-methods">
  ### Métodos auxiliares de nodos
</div>

- Los nodos pueden invocar `skills.bins` para obtener la lista actual de ejecutables de habilidades
  para las comprobaciones de autoaprobación.

<div id="exec-approvals">
  ## Aprobaciones de exec
</div>

- Cuando una solicitud de exec necesita aprobación, el Gateway emite `exec.approval.requested`.
- Los clientes de operador lo resuelven llamando a `exec.approval.resolve` (requiere el ámbito `operator.approvals`).

<div id="versioning">
  ## Versionado
</div>

- `PROTOCOL_VERSION` se encuentra en `src/gateway/protocol/schema.ts`.
- Los clientes envían `minProtocol` + `maxProtocol`; el servidor rechaza si no coinciden.
- Los esquemas y modelos se generan a partir de definiciones de TypeBox:
  - `pnpm protocol:gen`
  - `pnpm protocol:gen:swift`
  - `pnpm protocol:check`

<div id="auth">
  ## Autenticación
</div>

- Si `OPENCLAW_GATEWAY_TOKEN` (o `--token`) está configurado, `connect.params.auth.token`
  debe coincidir; de lo contrario, se cerrará el socket.
- Después del emparejamiento, el Gateway emite un **token de dispositivo** con un ámbito definido por el rol de la conexión y sus ámbitos. Se devuelve en `hello-ok.auth.deviceToken` y el cliente debe
  almacenarlo de forma persistente para conexiones futuras.
- Los tokens de dispositivo se pueden rotar o revocar mediante `device.token.rotate` y
  `device.token.revoke` (requiere el ámbito `operator.pairing`).

<div id="device-identity-pairing">
  ## Identidad del dispositivo + emparejamiento
</div>

- Los nodos deben incluir una identidad de dispositivo estable (`device.id`) derivada de una
  huella digital de un par de claves.
- Los Gateway emiten tokens por dispositivo + rol.
- Se requieren aprobaciones de emparejamiento para nuevos IDs de dispositivo, a menos que
  la aprobación automática local esté habilitada.
- Las conexiones **locales** incluyen loopback y la propia dirección tailnet del host del Gateway
  (de modo que los enlaces tailnet en el mismo host todavía puedan aprobarse automáticamente).
- Todos los clientes WS deben incluir la identidad `device` durante `connect` (operador + nodo).
  Control UI puede omitirla **solo** cuando `gateway.controlUi.allowInsecureAuth` está habilitado
  (o `gateway.controlUi.dangerouslyDisableDeviceAuth` para uso de emergencia *break-glass*).
- Las conexiones no locales deben firmar el nonce `connect.challenge` proporcionado por el servidor.

<div id="tls-pinning">
  ## TLS + pinning
</div>

- TLS se admite para conexiones WS.
- Los clientes pueden, opcionalmente, fijar la huella digital del certificado del Gateway (consulta la configuración `gateway.tls`
  junto con `gateway.remote.tlsFingerprint` o la opción de la CLI `--tls-fingerprint`).

<div id="scope">
  ## Ámbito
</div>

Este protocolo expone la **API completa del Gateway** (estado, canales, modelos, chat,
agente, sesiones, nodos, aprobaciones, etc.). La interfaz exacta está definida por los
esquemas TypeBox en `src/gateway/protocol/schema.ts`.