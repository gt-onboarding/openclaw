---
title: Gateway
summary: "CLI del Gateway de OpenClaw (`openclaw gateway`) — ejecutar, consultar y descubrir instancias de Gateway"
read_when:
  - Ejecutar el Gateway desde la CLI (entorno de desarrollo o servidores)
  - Depurar la autenticación del Gateway, los modos de enlace (bind) y la conectividad
  - Descubrir instancias de Gateway mediante Bonjour (LAN + tailnet)
---

<div id="gateway-cli">
  # CLI del Gateway
</div>

El Gateway es el servidor WebSocket de OpenClaw (canales, nodos, sesiones, hooks).

Los subcomandos de esta página se ejecutan con `openclaw gateway …`.

Documentación relacionada:

* [/gateway/bonjour](/es/gateway/bonjour)
* [/gateway/discovery](/es/gateway/discovery)
* [/gateway/configuration](/es/gateway/configuration)

<div id="run-the-gateway">
  ## Ejecuta el Gateway
</div>

Ejecuta un proceso local de Gateway:

```bash
openclaw gateway
```

Alias de primer plano:

```bash
openclaw gateway run
```

Notas:

* De forma predeterminada, el Gateway no se inicia a menos que `gateway.mode=local` esté configurado en `~/.openclaw/openclaw.json`. Usa `--allow-unconfigured` para ejecuciones ad‑hoc/de desarrollo.
* Escuchar en interfaces más allá de loopback sin autenticación está bloqueado (medida de seguridad).
* `SIGUSR1` desencadena un reinicio en proceso cuando está autorizado (habilita `commands.restart` o usa la herramienta/configuración del Gateway para aplicar/actualizar la configuración).
* Los manejadores de `SIGINT`/`SIGTERM` detienen el proceso del Gateway, pero no restauran ningún estado personalizado del terminal. Si envuelves la CLI con una TUI o entrada en modo raw, restaura el terminal antes de salir.

<div id="options">
  ### Opciones
</div>

* `--port <port>`: puerto WS (el valor predeterminado viene de la configuración/entorno; normalmente `18789`).
* `--bind <loopback|lan|tailnet|auto|custom>`: modo de enlace (bind) del listener.
* `--auth <token|password>`: sobrescribe el modo de autenticación.
* `--token <token>`: sobrescribe el token (también establece `OPENCLAW_GATEWAY_TOKEN` para el proceso).
* `--password <password>`: sobrescribe la contraseña (también establece `OPENCLAW_GATEWAY_PASSWORD` para el proceso).
* `--tailscale <off|serve|funnel>`: expone el Gateway a través de Tailscale.
* `--tailscale-reset-on-exit`: restablece la configuración de serve/funnel de Tailscale al cerrar.
* `--allow-unconfigured`: permite iniciar el Gateway sin `gateway.mode=local` en la configuración.
* `--dev`: crea una configuración de desarrollo + espacio de trabajo si falta (omite BOOTSTRAP.md).
* `--reset`: restablece la configuración de desarrollo + credenciales + sesiones + espacio de trabajo (requiere `--dev`).
* `--force`: finaliza cualquier listener existente en el puerto seleccionado antes de iniciar.
* `--verbose`: registros detallados.
* `--claude-cli-logs`: muestra solo los registros de claude-cli en la consola (y habilita su stdout/stderr).
* `--ws-log <auto|full|compact>`: estilo de registro de WS (valor predeterminado `auto`).
* `--compact`: alias de `--ws-log compact`.
* `--raw-stream`: registra eventos de flujo sin procesar del modelo en JSONL.
* `--raw-stream-path <path>`: ruta del JSONL de flujo sin procesar.

<div id="query-a-running-gateway">
  ## Consultar un Gateway en funcionamiento
</div>

Todos los comandos de consulta usan WebSocket RPC.

Modos de salida:

* Predeterminado: legible para humanos (con color en TTY).
* `--json`: JSON legible por máquina (sin estilos/spinner).
* `--no-color` (o `NO_COLOR=1`): desactiva ANSI manteniendo el formato legible para humanos.

Opciones compartidas (cuando estén disponibles):

* `--url <url>`: URL WebSocket del Gateway.
* `--token <token>`: token del Gateway.
* `--password <password>`: contraseña del Gateway.
* `--timeout <ms>`: límite de tiempo/presupuesto (varía según el comando).
* `--expect-final`: espera una respuesta «final» (llamadas de agentes).

<div id="gateway-health">
  ### `gateway health`
</div>

```bash
openclaw gateway health --url ws://127.0.0.1:18789
```

<div id="gateway-status">
  ### `gateway status`
</div>

`gateway status` muestra el servicio del Gateway (launchd/systemd/schtasks) más un sondeo RPC opcional.

```bash
openclaw gateway status
openclaw gateway status --json
```

Options:

* `--url <url>`: sobrescribe la URL de sondeo.
* `--token <token>`: autenticación con token para el sondeo.
* `--password <password>`: autenticación con contraseña para el sondeo.
* `--timeout <ms>`: tiempo de espera del sondeo (por defecto `10000`).
* `--no-probe`: omite el sondeo RPC (vista solo de servicios).
* `--deep`: analiza también los servicios a nivel de sistema.

<div id="gateway-probe">
  ### `gateway probe`
</div>

`gateway probe` es el comando de “depurar todo”. Siempre sondea:

* tu Gateway remoto configurado (si lo hay), y
* localhost (loopback) **aunque haya un remoto configurado**.

Si hay múltiples Gateways accesibles, los muestra todos. Se admiten múltiples Gateways cuando usas perfiles/puertos aislados (por ejemplo, un bot de rescate), pero en la mayoría de las instalaciones solo se ejecuta un único Gateway.

```bash
openclaw gateway probe
openclaw gateway probe --json
```

<div id="remote-over-ssh-mac-app-parity">
  #### Remoto por SSH (equivalencia con la app de Mac)
</div>

El modo “Remote over SSH” de la app de macOS usa un reenvío de puertos local para que el Gateway remoto (que puede estar vinculado solo a la interfaz de loopback) sea accesible en `ws://127.0.0.1:<port>`.

Equivalente en la CLI:

```bash
openclaw gateway probe --ssh user@gateway-host
```

Opciones:

* `--ssh <target>`: `user@host` o `user@host:port` (el puerto por defecto es `22`).
* `--ssh-identity <path>`: archivo de identidad.
* `--ssh-auto`: selecciona el primer host Gateway descubierto como objetivo SSH (solo LAN/WAB).

Configuración (opcional, se usa como valores predeterminados):

* `gateway.remote.sshTarget`
* `gateway.remote.sshIdentity`

<div id="gateway-call-method">
  ### `gateway call <method>`
</div>

Utilidad auxiliar RPC de bajo nivel.

```bash
openclaw gateway call status
openclaw gateway call logs.tail --params '{"sinceMs": 60000}'
```

<div id="manage-the-gateway-service">
  ## Gestionar el servicio del Gateway
</div>

```bash
openclaw gateway install
openclaw gateway start
openclaw gateway stop
openclaw gateway restart
openclaw gateway uninstall
```

Notas:

* `gateway install` admite `--port`, `--runtime`, `--token`, `--force`, `--json`.
* Los comandos de ciclo de vida aceptan `--json` para automatización mediante scripts.

<div id="discover-gateways-bonjour">
  ## Descubrir gateways (Bonjour)
</div>

`gateway discover` escanea en busca de balizas de Gateway (`_openclaw-gw._tcp`).

* Multicast DNS-SD: `local.`
* Unicast DNS-SD (Wide-Area Bonjour): elige un dominio (por ejemplo: `openclaw.internal.`) y configura split DNS + un servidor DNS; consulta [/gateway/bonjour](/es/gateway/bonjour)

Solo los gateways con el descubrimiento Bonjour habilitado (valor predeterminado) anuncian la baliza.

Los registros de descubrimiento de área amplia incluyen (TXT):

* `role` (indicación del rol del gateway)
* `transport` (indicación del transporte, p. ej. `gateway`)
* `gatewayPort` (puerto WebSocket, normalmente `18789`)
* `sshPort` (puerto SSH; es `22` de forma predeterminada si no está presente)
* `tailnetDns` (nombre de host MagicDNS, cuando esté disponible)
* `gatewayTls` / `gatewayTlsSha256` (TLS habilitado + huella digital del certificado)
* `cliPath` (indicación opcional para instalaciones remotas)

<div id="gateway-discover">
  ### `gateway discover`
</div>

```bash
openclaw gateway discover
```

Opciones:

* `--timeout <ms>`: tiempo de espera por comando (browse/resolve); valor predeterminado: `2000`.
* `--json`: salida en formato legible por máquinas (también desactiva los estilos/spinner).

Ejemplos:

```bash
openclaw gateway discover --timeout 4000
openclaw gateway discover --json | jq '.beacons[].wsUrl'
```
