---
title: Protocolo Bridge
summary: "Protocolo Bridge (nodos heredados): TCP JSONL, emparejamiento, RPC con ámbito"
read_when:
  - Al crear o depurar clientes de nodo (modo nodo en iOS/Android/macOS)
  - Al investigar fallos de emparejamiento o de autenticación del Bridge
  - Al auditar la superficie de nodo expuesta por el Gateway
---

<div id="bridge-protocol-legacy-node-transport">
  # Protocolo Bridge (transporte de nodo obsoleto)
</div>

El protocolo Bridge es un transporte de nodo **obsoleto** (TCP JSONL). Los nuevos clientes de nodo
deben usar en su lugar el protocolo WebSocket unificado del Gateway.

Si vas a implementar un operador o un cliente de nodo, usa el
[protocolo del Gateway](/es/gateway/protocol).

**Nota:** Las compilaciones actuales de OpenClaw ya no incluyen el listener TCP bridge; este documento se conserva como referencia histórica.
Las claves de configuración obsoletas `bridge.*` ya no forman parte del esquema de configuración.

<div id="why-we-have-both">
  ## Por qué tenemos ambos
</div>

* **Límite de seguridad**: el bridge expone una pequeña lista de permitidos en lugar de
  toda la superficie de la API del Gateway.
* **Emparejamiento + identidad del nodo**: la admisión de nodos la gestiona el Gateway y está vinculada
  a un token específico por nodo.
* **UX de descubrimiento**: los nodos pueden descubrir Gateways mediante Bonjour en la LAN, o conectarse
  directamente a través de una tailnet.
* **WS de loopback**: todo el plano de control WS permanece local a menos que se exponga a través de un túnel SSH.

<div id="transport">
  ## Transporte
</div>

* TCP, un objeto JSON por línea (JSONL).
* TLS opcional (cuando `bridge.tls.enabled` es `true`).
* El puerto de escucha predeterminado heredado era `18790` (las versiones actuales no inician un bridge TCP).

Cuando TLS está habilitado, los registros TXT de descubrimiento incluyen `bridgeTls=1` más
`bridgeTlsSha256` para que los nodos puedan hacer pinning del certificado.

<div id="handshake-pairing">
  ## Handshake + emparejamiento
</div>

1. El cliente envía `hello` con metadatos del nodo y token (si ya está emparejado).
2. Si no está emparejado, el Gateway responde con `error` (`NOT_PAIRED`/`UNAUTHORIZED`).
3. El cliente envía `pair-request`.
4. El Gateway espera la aprobación y luego envía `pair-ok` y `hello-ok`.

`hello-ok` devuelve `serverName` y puede incluir `canvasHostUrl`.

<div id="frames">
  ## Tramas
</div>

Cliente → Gateway:

* `req` / `res`: RPC del Gateway con ámbito definido (chat, sesiones, config, health, voicewake, skills.bins)
* `event`: señales del nodo (transcripción de voz, solicitud de agente, suscripción al chat, ciclo de vida de exec)

Gateway → Cliente:

* `invoke` / `invoke-res`: comandos del nodo (`canvas.*`, `camera.*`, `screen.record`,
  `location.get`, `sms.send`)
* `event`: actualizaciones de chat para sesiones suscritas
* `ping` / `pong`: señal de keepalive

La lógica de aplicación de la lista de permitidos heredada estaba en `src/gateway/server-bridge.ts` (eliminada).

<div id="exec-lifecycle-events">
  ## Eventos del ciclo de vida de exec
</div>

Los nodos pueden emitir eventos `exec.finished` o `exec.denied` para exponer la actividad de system.run.
Estos se mapean a eventos del sistema en el Gateway. (Los nodos heredados aún pueden emitir `exec.started`).

Campos del payload (todos opcionales salvo que se indique):

* `sessionKey` (obligatorio): sesión de agente que recibirá el evento del sistema.
* `runId`: id de ejecución único para agrupar.
* `command`: cadena de comando sin procesar o formateada.
* `exitCode`, `timedOut`, `success`, `output`: detalles de finalización (solo para finished).
* `reason`: motivo de denegación (solo para denied).

<div id="tailnet-usage">
  ## Uso de tailnet
</div>

* Vincula el bridge a una IP de tailnet: `bridge.bind: "tailnet"` en
  `~/.openclaw/openclaw.json`.
* Los clientes se conectan mediante el nombre de MagicDNS o la IP de tailnet.
* Bonjour **no** atraviesa redes; usa host/puerto manual o DNS‑SD de área amplia
  cuando sea necesario.

<div id="versioning">
  ## Versionado
</div>

Bridge es actualmente **v1 implícita** (sin negociación de versión mínima/máxima). Se espera compatibilidad con versiones anteriores; añade un campo de versión del protocolo de bridge antes de introducir cualquier cambio incompatible.