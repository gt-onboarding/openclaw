---
title: macOS
summary: "Aplicación complementaria de OpenClaw para macOS (barra de menús + broker del Gateway)"
read_when:
  - Al implementar funcionalidades de la app para macOS
  - Al modificar el ciclo de vida del Gateway o la interconexión de nodos en macOS
---

<div id="openclaw-macos-companion-menu-bar-gateway-broker">
  # OpenClaw macOS Companion (barra de menús + intermediario del Gateway)
</div>

La app de macOS es la **aplicación compañera de la barra de menús** para OpenClaw. Gestiona los permisos,
administra y se conecta al Gateway localmente (con launchd o de forma manual) y expone las
capacidades de macOS al agente como un nodo.

<div id="what-it-does">
  ## Qué hace
</div>

* Muestra notificaciones nativas y el estado en la barra de menús.
* Gestiona los diálogos de permisos TCC (Notificaciones, Accesibilidad, Grabación de pantalla, Micrófono,
  Reconocimiento de voz, Automatización/AppleScript).
* Ejecuta o se conecta al Gateway (local o remoto).
* Expone herramientas exclusivas de macOS (Canvas, Camera, Grabación de pantalla, `system.run`).
* Inicia el servicio de host de nodo local en modo **remote** (launchd) y lo detiene en modo **local**.
* Opcionalmente actúa como host de **PeekabooBridge** para automatización de la UI.
* Instala la CLI global (`openclaw`) mediante npm/pnpm a petición (bun no se recomienda para el entorno de ejecución del Gateway).

<div id="local-vs-remote-mode">
  ## Modo local vs remoto
</div>

* **Local** (predeterminado): la aplicación se conecta a un Gateway local en ejecución si está disponible;
  de lo contrario, habilita el servicio launchd mediante `openclaw gateway install`.
* **Remoto**: la aplicación se conecta a un Gateway a través de SSH/Tailscale y nunca inicia
  un proceso local.
  La aplicación inicia el **servicio de host del nodo** local para que el Gateway remoto pueda conectarse a este Mac.
  La aplicación no inicia el Gateway como proceso hijo.

<div id="launchd-control">
  ## Control de Launchd
</div>

La app administra un LaunchAgent por usuario etiquetado como `bot.molt.gateway`
(o `bot.molt.<profile>` cuando se usa `--profile`/`OPENCLAW_PROFILE`; los antiguos `com.openclaw.*` se siguen desactivando).

```bash
launchctl kickstart -k gui/$UID/bot.molt.gateway
launchctl bootout gui/$UID/bot.molt.gateway
```

Reemplaza la etiqueta por `bot.molt.<profile>` al ejecutar un perfil con nombre.

Si el LaunchAgent no está instalado, actívalo desde la aplicación o ejecuta
`openclaw gateway install`.

<div id="node-capabilities-mac">
  ## Capacidades del nodo (mac)
</div>

La app de macOS se presenta como un nodo. Comandos comunes:

* Canvas: `canvas.present`, `canvas.navigate`, `canvas.eval`, `canvas.snapshot`, `canvas.a2ui.*`
* Camera: `camera.snap`, `camera.clip`
* Screen: `screen.record`
* System: `system.run`, `system.notify`

El nodo expone un mapa de `permissions` para que los agentes puedan decidir qué está permitido.

Servicio de nodo + IPC de la app:

* Cuando el servicio de nodo en modo headless (sin interfaz) se está ejecutando (modo remoto), se conecta al WS del Gateway como un nodo.
* `system.run` se ejecuta en la app de macOS (contexto de UI/TCC) a través de un socket Unix local; las solicitudes y la salida permanecen dentro de la app.

Diagrama (SCI):

```
Gateway -> Node Service (WS)
                 |  IPC (UDS + token + HMAC + TTL)
                 v
             App Mac (UI + TCC + system.run)
```

<div id="exec-approvals-systemrun">
  ## Aprobaciones de ejecución (system.run)
</div>

`system.run` está controlado por las **Aprobaciones de ejecución** en la app de macOS (Settings → Exec approvals).
La seguridad + la solicitud + la lista de permitidos se almacenan localmente en el Mac en:

```
~/.openclaw/exec-approvals.json
```

Ejemplo:

```json
{
  "version": 1,
  "defaults": {
    "security": "deny",
    "ask": "on-miss"
  },
  "agents": {
    "main": {
      "security": "allowlist",
      "ask": "on-miss",
      "allowlist": [
        { "pattern": "/opt/homebrew/bin/rg" }
      ]
    }
  }
}
```

Notas:

* Las entradas de `allowlist` son patrones glob para rutas de binarios resueltas.
* Al seleccionar «Permitir siempre» en el diálogo se añade ese comando a la lista de permitidos.
* Las anulaciones del entorno de `system.run` se filtran (se eliminan `PATH`, `DYLD_*`, `LD_*`, `NODE_OPTIONS`, `PYTHON*`, `PERL*`, `RUBYOPT`) y luego se combinan con el entorno de la aplicación.

<div id="deep-links">
  ## Enlaces profundos
</div>

La aplicación registra el esquema de URL `openclaw://` para realizar acciones locales.

<div id="openclawagent">
  ### `openclaw://agent`
</div>

Inicia una solicitud `agent` al Gateway.

```bash
open 'openclaw://agent?message=Hello%20from%20deep%20link'
```

Parámetros de consulta:

* `message` (obligatorio)
* `sessionKey` (opcional)
* `thinking` (opcional)
* `deliver` / `to` / `channel` (opcional)
* `timeoutSeconds` (opcional)
* `key` (opcional, clave de modo desatendido)

Seguridad:

* Sin `key`, la aplicación solicita confirmación.
* Con una `key` válida, la ejecución se realiza en modo desatendido (previsto para automatizaciones personales).

<div id="onboarding-flow-typical">
  ## Flujo de incorporación (típico)
</div>

1. Instala y abre **OpenClaw.app**.
2. Completa la lista de verificación de permisos (solicitudes TCC).
3. Asegúrate de que el modo **Local** esté activado y que el Gateway se esté ejecutando.
4. Instala la CLI si quieres acceso desde la línea de comandos.

<div id="build-dev-workflow-native">
  ## Flujo de compilación y desarrollo (nativo)
</div>

* `cd apps/macos && swift build`
* `swift run OpenClaw` (o Xcode)
* Empaqueta la app: `scripts/package-mac-app.sh`

<div id="debug-gateway-connectivity-macos-cli">
  ## Depurar la conectividad con el Gateway (CLI de macOS)
</div>

Usa la CLI de depuración para probar el mismo handshake y la misma lógica de descubrimiento WebSocket del Gateway que utiliza la aplicación de macOS, sin tener que iniciarla.

```bash
cd apps/macos
swift run openclaw-mac connect --json
swift run openclaw-mac discover --timeout 3000 --json
```

Opciones de conexión:

* `--url <ws://host:port>`: sobrescribe la configuración
* `--mode <local|remote>`: se obtiene desde la configuración (valor predeterminado: configuración o local)
* `--probe`: fuerza un nuevo sondeo de estado
* `--timeout <ms>`: tiempo de espera de la solicitud (valor predeterminado: `15000`)
* `--json`: salida estructurada para hacer diff

Opciones de descubrimiento:

* `--include-local`: incluye Gateways que de otro modo se filtrarían como “locales”
* `--timeout <ms>`: ventana global de descubrimiento (valor predeterminado: `2000`)
* `--json`: salida estructurada para hacer diff

Consejo: compáralo con `openclaw gateway discover --json` para ver si la
canalización de descubrimiento de la app de macOS (NWBrowser + mecanismo de reserva DNS‑SD de tailnet) difiere del
descubrimiento basado en `dns-sd` del CLI de Node.

<div id="remote-connection-plumbing-ssh-tunnels">
  ## Infraestructura de conexión remota (túneles SSH)
</div>

Cuando la aplicación de macOS se ejecuta en modo **Remote**, establece un túnel SSH para que los componentes de la UI local puedan comunicarse con un Gateway remoto como si estuviera en localhost.

<div id="control-tunnel-gateway-websocket-port">
  ### Túnel de control (puerto WebSocket del Gateway)
</div>

* **Propósito:** comprobaciones de estado, estado, Web Chat, configuración y otras llamadas del plano de control.
* **Puerto local:** el puerto del Gateway (predeterminado `18789`), siempre estable.
* **Puerto remoto:** el mismo puerto del Gateway en el host remoto.
* **Comportamiento:** sin puerto local aleatorio; la app reutiliza un túnel existente en buen estado
  o lo reinicia si es necesario.
* **Forma SSH:** `ssh -N -L <local>:127.0.0.1:<remote>` con BatchMode +
  ExitOnForwardFailure + opciones de keepalive.
* **Informe de IP:** el túnel SSH usa loopback, por lo que el Gateway verá la IP del nodo
  como `127.0.0.1`. Usa el transporte **Direct (ws/wss)** si quieres que aparezca la IP real
  del cliente (consulta [acceso remoto en macOS](/es/platforms/mac/remote)).

Para los pasos de configuración, consulta [acceso remoto en macOS](/es/platforms/mac/remote). Para detalles del protocolo, consulta [protocolo del Gateway](/es/gateway/protocol).

<div id="related-docs">
  ## Documentación relacionada
</div>

* [Runbook del Gateway](/es/gateway)
* [Gateway (macOS)](/es/platforms/mac/bundled-gateway)
* [Permisos de macOS](/es/platforms/mac/permissions)
* [Canvas](/es/platforms/mac/canvas)