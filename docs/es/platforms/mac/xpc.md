---
title: Xpc
summary: "Arquitectura de IPC de macOS para la app de OpenClaw, el transporte de nodo del Gateway y PeekabooBridge"
read_when:
  - Al editar contratos IPC o el IPC de la app de la barra de menús
---

<div id="openclaw-macos-ipc-architecture">
  # Arquitectura IPC de OpenClaw en macOS
</div>

**Modelo actual:** un socket Unix local conecta el **servicio host del nodo** con la **app de macOS** para aprobaciones de ejecución y `system.run`. Existe una CLI de depuración `openclaw-mac` para comprobaciones de descubrimiento y conexión; las acciones de los agentes siguen pasando a través del WebSocket del Gateway y `node.invoke`. La automatización de la UI utiliza PeekabooBridge.

<div id="goals">
  ## Objetivos
</div>

* Una única instancia de la app gráfica que centralice todo el trabajo relacionado con TCC (notificaciones, grabación de pantalla, micrófono, voz, AppleScript).
* Una superficie mínima para la automatización: comandos de Gateway + nodo, más PeekabooBridge para automatización de UI.
* Permisos predecibles: siempre el mismo ID de bundle firmado, lanzado por launchd, de modo que los permisos otorgados por TCC persistan.

<div id="how-it-works">
  ## Cómo funciona
</div>

<div id="gateway-node-transport">
  ### Transporte entre Gateway y nodo
</div>

* La aplicación ejecuta el Gateway (modo local) y se conecta a él como nodo.
* Las acciones del agente se realizan mediante `node.invoke` (p. ej., `system.run`, `system.notify`, `canvas.*`).

<div id="node-service-app-ipc">
  ### Servicio de nodo + IPC de la app
</div>

* Un servicio de nodo sin interfaz gráfica (headless) se conecta al WebSocket del Gateway.
* Las solicitudes `system.run` se reenvían a la app de macOS a través de un socket Unix local.
* La app realiza el exec en el contexto de la UI, solicita confirmación si es necesario y devuelve la salida.

Diagrama (SCI):

```
Agent -> Gateway -> Node Service (WS)
                      |  IPC (UDS + token + HMAC + TTL)
                      v
                  Mac App (UI + TCC + system.run)
```

<div id="peekaboobridge-ui-automation">
  ### PeekabooBridge (automatización de la UI)
</div>

* La automatización de la UI utiliza un socket UNIX independiente llamado `bridge.sock` y el protocolo JSON de PeekabooBridge.
* Orden de preferencia de host (del lado del cliente): Peekaboo.app → Claude.app → OpenClaw.app → ejecución local.
* Seguridad: los hosts del bridge requieren un TeamID permitido; el mecanismo de escape solo para DEBUG con el mismo UID está protegido por `PEEKABOO_ALLOW_UNSIGNED_SOCKET_CLIENTS=1` (convención de Peekaboo).
* Consulta: [Uso de PeekabooBridge](/es/platforms/mac/peekaboo) para más detalles.

<div id="operational-flows">
  ## Flujos operativos
</div>

* Reinicio/recompilación: `SIGN_IDENTITY="Apple Development: <Developer Name> (<TEAMID>)" scripts/restart-mac.sh`
  * Termina las instancias existentes
  * Compilación Swift + empaquetado
  * Escribe/prepara/inicia el LaunchAgent
* Instancia única: la app se cierra inmediatamente si ya se está ejecutando otra instancia con el mismo bundle ID.

<div id="hardening-notes">
  ## Notas de hardening
</div>

* Es preferible exigir una coincidencia de TeamID para todas las superficies privilegiadas.
* PeekabooBridge: `PEEKABOO_ALLOW_UNSIGNED_SOCKET_CLIENTS=1` (solo DEBUG) puede permitir clientes con el mismo UID para desarrollo local.
* Toda la comunicación permanece local; no se exponen sockets de red.
* Los avisos TCC se originan únicamente desde el paquete de la app GUI; mantén el ID de paquete firmado estable entre compilaciones.
* Hardening de IPC: modo de socket `0600`, token, comprobaciones de UID del par, desafío/respuesta HMAC, TTL breve.