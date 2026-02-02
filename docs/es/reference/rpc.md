---
title: Rpc
summary: "Adaptadores RPC para CLIs externas (signal-cli, imsg) y patrones de Gateway"
read_when:
  - Agregar o modificar integraciones externas de CLI
  - Depurar adaptadores RPC (signal-cli, imsg)
---

<div id="rpc-adapters">
  # Adaptadores RPC
</div>

OpenClaw integra CLIs externas a través de JSON-RPC. Actualmente se emplean dos patrones.

<div id="pattern-a-http-daemon-signal-cli">
  ## Patrón A: demonio HTTP (signal-cli)
</div>

* `signal-cli` se ejecuta como un demonio con JSON-RPC sobre HTTP.
* El flujo de eventos es SSE (`/api/v1/events`).
* Comprobación de estado: `/api/v1/check`.
* OpenClaw controla el ciclo de vida cuando `channels.signal.autoStart=true`.

Consulta [Signal](/es/channels/signal) para la configuración y los endpoints.

<div id="pattern-b-stdio-child-process-imsg">
  ## Patrón B: proceso hijo stdio (imsg)
</div>

* OpenClaw lanza `imsg rpc` como proceso hijo.
* JSON-RPC usa delimitación por líneas en stdin/stdout (un objeto JSON por línea).
* No se requiere puerto TCP ni demonio.

Métodos principales utilizados:

* `watch.subscribe` → notificaciones (`method: "message"`)
* `watch.unsubscribe`
* `send`
* `chats.list` (sondeo/diagnóstico)

Consulta [iMessage](/es/channels/imessage) para la configuración y el direccionamiento (se prefiere `chat_id`).

<div id="adapter-guidelines">
  ## Directrices para adaptadores
</div>

* Gateway controla el proceso (inicio/detención ligados al ciclo de vida del proveedor).
* Mantén los clientes RPC robustos: tiempos de espera, reinicio al salir.
* Prefiere identificadores estables (p. ej., `chat_id`) en lugar de cadenas de texto visibles.