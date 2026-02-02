---
title: Presencia
summary: "Cómo se generan, combinan y muestran las entradas de presencia de OpenClaw"
read_when:
  - Solucionar problemas de la pestaña Instances
  - Investigar filas de instancias duplicadas o obsoletas
  - Cambiar las balizas de conexión WS del Gateway o de eventos del sistema
---

<div id="presence">
  # Presencia
</div>

La “presencia” de OpenClaw es una vista ligera, basada en el mejor esfuerzo, de:

- el propio **Gateway**, y
- los **clientes conectados al Gateway** (app de macOS, WebChat, CLI, etc.)

La presencia se usa principalmente para mostrar la pestaña **Instances** de la app de macOS y para
proporcionar al operador una visión rápida.

<div id="presence-fields-what-shows-up">
  ## Campos de presencia (lo que se muestra)
</div>

Las entradas de presencia son objetos estructurados con campos como:

- `instanceId` (opcional pero muy recomendable): identidad estable del cliente (normalmente `connect.client.instanceId`)
- `host`: nombre de host fácil de leer para humanos
- `ip`: dirección IP determinada por mejor esfuerzo
- `version`: cadena de versión del cliente
- `deviceFamily` / `modelIdentifier`: indicios sobre el hardware
- `mode`: `ui`, `webchat`, `cli`, `backend`, `probe`, `test`, `node`, ...
- `lastInputSeconds`: “segundos desde la última entrada del usuario” (si se conoce)
- `reason`: `self`, `connect`, `node-connected`, `periodic`, ...
- `ts`: marca de tiempo de la última actualización (ms desde la época Unix)

<div id="producers-where-presence-comes-from">
  ## Productores (orígenes de la presencia)
</div>

Las entradas de presencia se generan a partir de múltiples fuentes y se **combinan**.

<div id="1-gateway-self-entry">
  ### 1) Entrada propia del Gateway
</div>

El Gateway siempre inicializa una entrada de “self” al inicio para que las UIs muestren el host del Gateway incluso antes de que cualquier cliente se conecte.

<div id="2-websocket-connect">
  ### 2) Conexión WebSocket
</div>

Cada cliente WS comienza con una solicitud de `connect`. Tras un handshake exitoso, el Gateway crea o actualiza una entrada de presencia para esa conexión.

<div id="why-oneoff-cli-commands-dont-show-up">
  #### Por qué los comandos puntuales de la CLI no aparecen
</div>

La CLI suele conectarse para comandos breves y puntuales. Para evitar saturar la
lista de instancias, `client.mode === "cli"` **no** se convierte en una entrada de presencia.

<div id="3-system-event-beacons">
  ### 3) Balizas de `system-event`
</div>

Los clientes pueden enviar balizas periódicas más completas mediante el método `system-event`. La app de macOS usa esto para informar del nombre de host, la IP y `lastInputSeconds`.

<div id="4-node-connects-role-node">
  ### 4) Conexión del nodo (rol: node)
</div>

Cuando un nodo se conecta al WebSocket del Gateway con `role: node`, el Gateway
inserta o actualiza una entrada de presencia para ese nodo (el mismo flujo que con otros clientes WS).

<div id="merge-dedupe-rules-why-instanceid-matters">
  ## Reglas de combinación y desduplicación (por qué `instanceId` es importante)
</div>

Las entradas de presencia se almacenan en un único mapa en memoria:

- Las entradas se indexan por una **clave de presencia**.
- La mejor clave es un `instanceId` estable (de `connect.client.instanceId`) que persista a los reinicios.
- Las claves no distinguen entre mayúsculas y minúsculas.

Si un cliente se reconecta sin un `instanceId` estable, puede mostrarse como una fila
**duplicada**.

<div id="ttl-and-bounded-size">
  ## TTL y tamaño limitado
</div>

La presencia es intencionalmente efímera:

- **TTL:** las entradas con más de 5 minutos se eliminan
- **Entradas máximas:** 200 (se descartan primero las más antiguas)

Esto mantiene la lista actualizada y evita un crecimiento descontrolado de la memoria.

<div id="remotetunnel-caveat-loopback-ips">
  ## Advertencia sobre conexiones remotas/túneles (IPs de loopback)
</div>

Cuando un cliente se conecta mediante un túnel SSH o un reenvío de puertos local, el Gateway puede
ver la dirección remota como `127.0.0.1`. Para evitar sobrescribir una IP correcta reportada por el cliente,
se ignoran las direcciones remotas de loopback.

<div id="consumers">
  ## Consumidores
</div>

<div id="macos-instances-tab">
  ### Pestaña Instances en macOS
</div>

La aplicación de macOS muestra la salida de `system-presence` y aplica un pequeño
indicador de estado (Active/Idle/Stale) según la antigüedad de la última actualización.

<div id="debugging-tips">
  ## Consejos de depuración
</div>

- Para ver la lista sin procesar, llama a `system-presence` en el Gateway.
- Si ves duplicados:
  - confirma que los clientes envían un `client.instanceId` estable en el handshake
  - confirma que los beacons periódicos usan el mismo `instanceId`
  - comprueba si la entrada derivada de la conexión no tiene `instanceId` (en ese caso, es normal que haya duplicados)