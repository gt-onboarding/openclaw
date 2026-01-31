---
title: Gateway
summary: "Guía operativa del servicio Gateway, su ciclo de vida y operaciones"
read_when:
  - Al ejecutar o depurar el proceso Gateway
---

<div id="gateway-service-runbook">
  # Runbook del servicio Gateway
</div>

Última actualización: 2025-12-09

<div id="what-it-is">
  ## Qué es
</div>

* El proceso en ejecución permanente que posee la única conexión Baileys/Telegram y el plano de control/eventos.
* Sustituye al comando heredado `gateway`. Punto de entrada de la CLI: `openclaw gateway`.
* Se ejecuta hasta que se detiene; finaliza con un código de salida distinto de cero ante errores fatales para que el supervisor lo reinicie.

<div id="how-to-run-local">
  ## Cómo ejecutar en local
</div>

```bash
openclaw gateway --port 18789
# for full debug/trace logs in stdio:
openclaw gateway --port 18789 --verbose
# si el puerto está ocupado, terminar los listeners y luego iniciar:
openclaw gateway --force
# dev loop (auto-reload on TS changes):
pnpm gateway:watch
```

* La recarga dinámica de configuración supervisa `~/.openclaw/openclaw.json` (o `OPENCLAW_CONFIG_PATH`).
  * Modo predeterminado: `gateway.reload.mode="hybrid"` (aplica en caliente los cambios seguros y reinicia ante cambios críticos).
  * La recarga dinámica usa un reinicio en el mismo proceso mediante **SIGUSR1** cuando es necesario.
  * Desactívala con `gateway.reload.mode="off"`.
* Vincula el plano de control WebSocket a `127.0.0.1:<port>` (predeterminado 18789).
* El mismo puerto también sirve HTTP (Control UI, hooks, A2UI). Multiplexación de puerto único.
  * OpenAI Chat Completions (HTTP): [`/v1/chat/completions`](/es/gateway/openai-http-api).
  * OpenResponses (HTTP): [`/v1/responses`](/es/gateway/openresponses-http-api).
  * Tools Invoke (HTTP): [`/tools/invoke`](/es/gateway/tools-invoke-http-api).
* Inicia un servidor de archivos Canvas de forma predeterminada en `canvasHost.port` (predeterminado `18793`), sirviendo `http://<gateway-host>:18793/__openclaw__/canvas/` desde `~/.openclaw/workspace/canvas`. Desactívalo con `canvasHost.enabled=false` o `OPENCLAW_SKIP_CANVAS_HOST=1`.
* Registra en stdout; usa launchd/systemd para mantenerlo en ejecución y rotar los logs.
* Pasa `--verbose` para reflejar el registro de depuración (handshakes, req/res, eventos) desde el archivo de log hacia stdio cuando estés solucionando problemas.
* `--force` usa `lsof` para encontrar procesos que escuchan en el puerto elegido, envía SIGTERM, registra qué terminó y luego inicia el Gateway (falla rápidamente si falta `lsof`).
* Si lo ejecutas bajo un supervisor (launchd/systemd/modo de proceso hijo de la app de macOS), una parada/reinicio normalmente envía **SIGTERM**; compilaciones más antiguas pueden mostrar esto como código de salida `pnpm` `ELIFECYCLE` **143** (SIGTERM), que es un apagado normal, no un fallo.
* **SIGUSR1** desencadena un reinicio en el mismo proceso cuando está autorizado (aplicación/actualización de herramienta/configuración del Gateway, o habilita `commands.restart` para reinicios manuales).
* La autenticación del Gateway es obligatoria de forma predeterminada: establece `gateway.auth.token` (o `OPENCLAW_GATEWAY_TOKEN`) o `gateway.auth.password`. Los clientes deben enviar `connect.params.auth.token/password` a menos que usen identidad de Tailscale Serve.
* El asistente ahora genera un token de forma predeterminada, incluso en loopback.
* Precedencia de puertos: `--port` &gt; `OPENCLAW_GATEWAY_PORT` &gt; `gateway.port` &gt; valor predeterminado `18789`.

<div id="remote-access">
  ## Acceso remoto
</div>

* Es preferible usar Tailscale/VPN; en caso contrario, utiliza un túnel SSH:
  ```bash
  ssh -N -L 18789:127.0.0.1:18789 user@host
  ```
* Luego, los clientes se conectan a `ws://127.0.0.1:18789` a través del túnel.
* Si se configura un token, los clientes deben incluirlo en `connect.params.auth.token` incluso cuando se conecten a través del túnel.

<div id="multiple-gateways-same-host">
  ## Múltiples Gateways (mismo host)
</div>

Suele ser innecesario: un solo Gateway puede servir múltiples canales de mensajería y agentes. Usa múltiples Gateways solo para redundancia o aislamiento estricto (p. ej., bot de rescate).

Es compatible siempre que aísles el estado y la configuración y uses puertos únicos. Guía completa: [Múltiples Gateways](/es/gateway/multiple-gateways).

Los nombres de servicio tienen en cuenta el perfil:

* macOS: `bot.molt.&lt;profile&gt;` (el legado `com.openclaw.*` puede seguir existiendo)
* Linux: `openclaw-gateway-&lt;profile&gt;.service`
* Windows: `OpenClaw Gateway (&lt;profile&gt;)`

Los metadatos de instalación están incluidos en la configuración del servicio:

* `OPENCLAW_SERVICE_MARKER=openclaw`
* `OPENCLAW_SERVICE_KIND=gateway`
* `OPENCLAW_SERVICE_VERSION=&lt;version&gt;`

Patrón Rescue-Bot: mantén un segundo Gateway aislado con su propio perfil, directorio de estado, espacio de trabajo y un espaciado de puertos base independiente. Guía completa: [Guía de Rescue-bot](/es/gateway/multiple-gateways#rescue-bot-guide).

<div id="dev-profile-dev">
  ### Perfil de desarrollo (`--dev`)
</div>

Vía rápida: ejecuta una instancia de desarrollo totalmente aislada (config/estado/espacio de trabajo) sin afectar tu configuración principal.

```bash
openclaw --dev setup
openclaw --dev gateway --allow-unconfigured
# luego apunta a la instancia dev:
openclaw --dev status
openclaw --dev health
```

Valores predeterminados (se pueden sobrescribir mediante variables de entorno/flags de CLI/config):

* `OPENCLAW_STATE_DIR=~/.openclaw-dev`
* `OPENCLAW_CONFIG_PATH=~/.openclaw-dev/openclaw.json`
* `OPENCLAW_GATEWAY_PORT=19001` (Gateway WS + HTTP)
* puerto del servicio de control del navegador = `19003` (derivado: `gateway.port+2`, solo loopback)
* `canvasHost.port=19005` (derivado: `gateway.port+4`)
* `agents.defaults.workspace` pasa a ser `~/.openclaw/workspace-dev` de forma predeterminada cuando ejecutas `setup`/`onboard` con `--dev`.

Puertos derivados (reglas generales):

* Puerto base = `gateway.port` (o `OPENCLAW_GATEWAY_PORT` / `--port`)
* puerto del servicio de control del navegador = base + 2 (solo loopback)
* `canvasHost.port = base + 4` (o `OPENCLAW_CANVAS_HOST_PORT` / anulación en la config)
* Los puertos CDP del perfil del navegador se asignan automáticamente desde `browser.controlPort + 9 .. + 108` (se persisten por perfil).

Lista de comprobación para cada instancia:

* `gateway.port` único
* `OPENCLAW_CONFIG_PATH` único
* `OPENCLAW_STATE_DIR` único
* `agents.defaults.workspace` único
* números de WhatsApp separados (si usas WA)

Instalación del servicio por perfil:

```bash
openclaw --profile main gateway install
openclaw --profile rescue gateway install
```

Ejemplo:

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json OPENCLAW_STATE_DIR=~/.openclaw-a openclaw gateway --port 19001
OPENCLAW_CONFIG_PATH=~/.openclaw/b.json OPENCLAW_STATE_DIR=~/.openclaw-b openclaw gateway --port 19002
```

<div id="protocol-operator-view">
  ## Protocolo (vista del operador)
</div>

* Documentación completa: [Protocolo de Gateway](/es/gateway/protocol) y [Protocolo de Bridge (heredado)](/es/gateway/bridge-protocol).
* Primer frame obligatorio desde el cliente: `req {type:"req", id, method:"connect", params:{minProtocol,maxProtocol,client:{id,displayName?,version,platform,deviceFamily?,modelIdentifier?,mode,instanceId?}, caps, auth?, locale?, userAgent? } }`.
* Gateway responde con `res {type:"res", id, ok:true, payload:hello-ok }` (u `ok:false` con un error, luego cierra).
* Tras el handshake:
  * Requests: `{type:"req", id, method, params}` → `{type:"res", id, ok, payload|error}`
  * Events: `{type:"event", event, payload, seq?, stateVersion?}`
* Entradas de presencia estructuradas: `{host, ip, version, platform?, deviceFamily?, modelIdentifier?, mode, lastInputSeconds?, ts, reason?, tags?[], instanceId? }` (para clientes WS, `instanceId` proviene de `connect.client.instanceId`).
* Las respuestas de `agent` son de dos etapas: primero un `res` de ack `{runId,status:"accepted"}`, luego un `res` final `{runId,status:"ok"|"error",summary}` después de que finaliza la ejecución; la salida en streaming llega como `event:"agent"`.

<div id="methods-initial-set">
  ## Métodos (conjunto inicial)
</div>

* `health` — instantánea completa del estado (misma estructura que `openclaw health --json`).
* `status` — resumen breve.
* `system-presence` — lista de presencia actual.
* `system-event` — publicar una nota de presencia/sistema (estructurada).
* `send` — enviar un mensaje mediante el/los canal(es) activo(s).
* `agent` — ejecutar un turno de un agente (transmite eventos de vuelta por la misma conexión).
* `node.list` — listar nodos emparejados y conectados actualmente (incluye `caps`, `deviceFamily`, `modelIdentifier`, `paired`, `connected` y los `commands` anunciados).
* `node.describe` — describir un nodo (capacidades y comandos `node.invoke` compatibles; funciona para nodos emparejados y para nodos no emparejados conectados actualmente).
* `node.invoke` — invocar un comando en un nodo (p. ej., `canvas.*`, `camera.*`).
* `node.pair.*` — ciclo de vida de emparejamiento (`request`, `list`, `approve`, `reject`, `verify`).

Consulta también: [Presence](/es/concepts/presence) para ver cómo se genera y se deduplica la presencia y por qué un `client.instanceId` estable es importante.

<div id="events">
  ## Eventos
</div>

* `agent` — eventos de herramienta/salida por streaming de la ejecución del agente (marcados con número de secuencia).
* `presence` — actualizaciones de presencia (deltas con stateVersion) enviadas a todos los clientes conectados.
* `tick` — señal periódica de keepalive/no-op para confirmar que sigue activo.
* `shutdown` — el Gateway se está apagando; la carga útil incluye `reason` y un `restartExpectedMs` opcional. Los clientes deben reconectarse.

<div id="webchat-integration">
  ## Integración de WebChat
</div>

* WebChat es una UI nativa de SwiftUI que se comunica directamente con el WebSocket del Gateway para historial, envíos, cancelaciones y eventos.
* El uso remoto pasa por el mismo túnel SSH/Tailscale; si se configura un token del Gateway, el cliente lo incluye durante `connect`.
* La app de macOS se conecta mediante un único WS (conexión compartida); carga el estado de presencia a partir del snapshot inicial y escucha eventos de `presence` para actualizar la UI.

<div id="typing-and-validation">
  ## Tipado y validación
</div>

* El servidor valida cada frame entrante con AJV contra el JSON Schema generado a partir de las definiciones de protocolo.
* Los clientes (TS/Swift) consumen los tipos generados (TS directamente; Swift a través del generador del repositorio).
* Las definiciones de protocolo son la fuente de verdad única; vuelve a generar los esquemas/modelos con:
  * `pnpm protocol:gen`
  * `pnpm protocol:gen:swift`

<div id="connection-snapshot">
  ## Instantánea de conexión
</div>

* `hello-ok` incluye una `snapshot` con `presence`, `health`, `stateVersion` y `uptimeMs` más `policy {maxPayload,maxBufferedBytes,tickIntervalMs}` para que los clientes puedan renderizar de inmediato sin solicitudes adicionales.
* `health`/`system-presence` siguen disponibles para refresco manual, pero no son necesarios en el momento de la conexión.

<div id="error-codes-reserror-shape">
  ## Códigos de error (estructura de res.error)
</div>

* Los errores usan `{ code, message, details?, retryable?, retryAfterMs? }`.
* Códigos estándar:
  * `NOT_LINKED` — WhatsApp no está autenticado.
  * `AGENT_TIMEOUT` — el agente no respondió dentro del plazo configurado.
  * `INVALID_REQUEST` — la validación de esquema/parámetros falló.
  * `UNAVAILABLE` — el Gateway se está apagando o alguna dependencia no está disponible.

<div id="keepalive-behavior">
  ## Comportamiento de keepalive
</div>

* Los eventos `tick` (o ping/pong WS) se emiten periódicamente para que los clientes sepan que el Gateway sigue activo incluso cuando no hay tráfico.
* Las confirmaciones de send/del agente siguen siendo respuestas separadas; no sobrecargues los ticks para operaciones de send.

<div id="replay-gaps">
  ## Repetición / huecos
</div>

* Los eventos no se repiten. Los clientes detectan huecos en la secuencia (`seq`) y deben volver a sincronizar (`health` + `system-presence`) antes de continuar. Los clientes WebChat y macOS ahora se actualizan automáticamente al detectar un hueco.

<div id="supervision-macos-example">
  ## Supervisión (ejemplo en macOS)
</div>

* Usa launchd para mantener el servicio activo:
  * Program: ruta a `openclaw`
  * Arguments: `gateway`
  * KeepAlive: true
  * StandardOut/Err: rutas de archivos o `syslog`
* En caso de fallo, launchd reinicia; una mala configuración fatal debería seguir terminando para que el operador lo note.
* Los LaunchAgents son por usuario y requieren una sesión iniciada; para despliegues headless usa un LaunchDaemon personalizado (no se incluye).
  * `openclaw gateway install` escribe `~/Library/LaunchAgents/bot.molt.gateway.plist`
    (o `bot.molt.<profile>.plist`; los `com.openclaw.*` heredados se eliminan).
  * `openclaw doctor` audita la configuración del LaunchAgent y puede actualizarla a los valores predeterminados actuales.

<div id="gateway-service-management-cli">
  ## Gestión del servicio de Gateway (CLI)
</div>

Usa la CLI de Gateway para instalar/iniciar/detener/reiniciar/consultar el estado:

```bash
openclaw gateway status
openclaw gateway install
openclaw gateway stop
openclaw gateway restart
openclaw logs --follow
```

Notas:

* `gateway status` sondea el RPC del Gateway de forma predeterminada usando el puerto/configuración resueltos del servicio (puedes sobrescribirlo con `--url`).
* `gateway status --deep` añade análisis a nivel de sistema (LaunchDaemons/unidades del sistema).
* `gateway status --no-probe` omite el sondeo RPC (útil cuando la red está caída).
* `gateway status --json` es estable para scripts.
* `gateway status` informa el **runtime del supervisor** (launchd/systemd en ejecución) por separado de la **disponibilidad del RPC** (conexión WS + RPC de estado).
* `gateway status` imprime la ruta de configuración + el destino del sondeo para evitar la confusión entre “localhost vs bind en LAN” y desajustes de perfil.
* `gateway status` incluye la última línea de error del Gateway cuando el servicio parece estar en ejecución pero el puerto está cerrado.
* `logs` hace tail del archivo de log del Gateway vía RPC (no necesitas `tail`/`grep` manual).
* Si se detectan otros servicios similares al Gateway, la CLI muestra una advertencia a menos que sean servicios de perfil de OpenClaw.
  Aun así, recomendamos **un gateway por máquina** para la mayoría de las configuraciones; usa perfiles/puertos aislados para redundancia o un bot de rescate. Consulta [Multiple gateways](/es/gateway/multiple-gateways).
  * Limpieza: `openclaw gateway uninstall` (servicio actual) y `openclaw doctor` (migraciones heredadas).
* `gateway install` no realiza ninguna acción cuando el servicio ya está instalado; usa `openclaw gateway install --force` para reinstalar (cambios de perfil/entorno/ruta).

App de macOS incluida:

* OpenClaw.app puede incluir un relay del Gateway basado en Node e instalar un LaunchAgent por usuario etiquetado
  `bot.molt.gateway` (o `bot.molt.<profile>`; las etiquetas heredadas `com.openclaw.*` siguen pudiéndose descargar (unload) correctamente).
* Para detenerlo limpiamente, usa `openclaw gateway stop` (o `launchctl bootout gui/$UID/bot.molt.gateway`).
* Para reiniciarlo, usa `openclaw gateway restart` (o `launchctl kickstart -k gui/$UID/bot.molt.gateway`).
  * `launchctl` solo funciona si el LaunchAgent está instalado; de lo contrario usa primero `openclaw gateway install`.
  * Sustituye la etiqueta por `bot.molt.<profile>` cuando ejecutes un perfil con nombre.

<div id="supervision-systemd-user-unit">
  ## Supervisión (unidad de usuario de systemd)
</div>

OpenClaw instala de forma predeterminada un **servicio de usuario de systemd** en Linux/WSL2. Recomendamos servicios de usuario para máquinas de un solo usuario (entorno más simple, configuración por usuario). Usa un **servicio del sistema** para servidores multiusuario o siempre activos (no se requiere `lingering`, supervisión compartida).

`openclaw gateway install` escribe la unidad de usuario. `openclaw doctor` audita la unidad y puede actualizarla para que se alinee con los valores predeterminados recomendados actualmente.

Crea `~/.config/systemd/user/openclaw-gateway[-<profile>].service`:

```
[Unit]
Description=OpenClaw Gateway (profile: <profile>, v<version>)
After=network-online.target
Wants=network-online.target

[Service]
ExecStart=/usr/local/bin/openclaw gateway --port 18789
Restart=always
RestartSec=5
Environment=OPENCLAW_GATEWAY_TOKEN=
WorkingDirectory=/home/youruser

[Install]
WantedBy=default.target
```

Habilita el lingering (necesario para que el servicio de usuario siga activo tras el cierre de sesión o durante la inactividad):

```
sudo loginctl enable-linger youruser
```

El proceso de onboarding ejecuta esto en Linux/WSL2 (puede solicitar sudo; escribe en `/var/lib/systemd/linger`).
Luego habilita el servicio:

```
systemctl --user enable --now openclaw-gateway[-<profile>].service
```

**Alternativa (servicio de sistema)**: para servidores siempre encendidos o multiusuario,
puedes instalar una unidad **system** de systemd en lugar de una unidad de usuario (no se necesita *lingering*).
Crea `/etc/systemd/system/openclaw-gateway[-<profile>].service` (copia la unidad anterior,
cambia `WantedBy=multi-user.target`, establece `User=` + `WorkingDirectory=`) y luego:

```
sudo systemctl daemon-reload
sudo systemctl enable --now openclaw-gateway[-<profile>].service
```

<div id="windows-wsl2">
  ## Windows (WSL2)
</div>

Las instalaciones en Windows deben utilizar **WSL2** y seguir la sección anterior sobre systemd en Linux.

<div id="operational-checks">
  ## Comprobaciones operativas
</div>

* Liveness: abre un WS y envía `req:connect` → espera recibir una `res` con `payload.type="hello-ok"` (con snapshot).
* Readiness: llama a `health` → espera `ok: true` y un canal vinculado en `linkChannel` (cuando corresponda).
* Depuración: suscríbete a los eventos `tick` y `presence`; asegúrate de que `status` muestre la antigüedad del enlace o de la autenticación; las entradas de presencia deben mostrar el host del Gateway y los clientes conectados.

<div id="safety-guarantees">
  ## Garantías de seguridad
</div>

* Supón un Gateway por host de forma predeterminada; si ejecutas varios perfiles, aísla los puertos y el estado, y asegúrate de apuntar a la instancia correcta.
* No se recurre a conexiones directas con Baileys; si el Gateway está caído, los envíos fallan de inmediato.
* Los primeros frames que no sean de conexión o el JSON malformado se rechazan y se cierra el socket.
* Apagado ordenado: emite el evento `shutdown` antes de cerrar; los clientes deben manejar el cierre y la reconexión.

<div id="cli-helpers">
  ## Utilidades de la CLI
</div>

* `openclaw gateway health|status` — consulta el estado/salud a través del WS del Gateway.
* `openclaw message send --target <num> --message "hi" [--media ...]` — envía a través del Gateway (idempotente para WhatsApp).
* `openclaw agent --message "hi" --to <num>` — ejecuta un turno de agente (espera el resultado final de forma predeterminada).
* `openclaw gateway call <method> --params '{"k":"v"}'` — invocador directo de métodos para depuración.
* `openclaw gateway stop|restart` — detiene/reinicia el servicio Gateway supervisado (launchd/systemd).
* Los subcomandos de ayuda del Gateway suponen que hay un Gateway en ejecución en `--url`; ya no inician uno automáticamente.

<div id="migration-guidance">
  ## Guía de migración
</div>

* Retira los usos de `openclaw gateway` y del puerto de control TCP obsoleto.
* Actualiza los clientes para que hablen el protocolo WS con conexión obligatoria y presencia estructurada.