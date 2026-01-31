---
title: Control Ui
summary: "UI de control basada en el navegador para el Gateway (chat, nodos, configuración)"
read_when:
  - Quieres operar el Gateway desde un navegador
  - Quieres acceso a Tailnet sin túneles SSH
---

<div id="control-ui-browser">
  # Control UI (navegador)
</div>

La Control UI es una pequeña aplicación de una sola página basada en **Vite + Lit** servida por el Gateway:

* por defecto: `http://<host>:18789/`
* prefijo opcional: configura `gateway.controlUi.basePath` (por ejemplo, `/openclaw`)

Se comunica **directamente con el WebSocket del Gateway** a través del mismo puerto.

<div id="quick-open-local">
  ## Apertura rápida (local)
</div>

Si el Gateway se está ejecutando en el mismo equipo, abre:

* http://127.0.0.1:18789/ (o http://localhost:18789/)

Si la página no se carga, inicia primero el Gateway: `openclaw gateway`.

La autenticación se proporciona durante el handshake de WebSocket mediante:

* `connect.params.auth.token`
* `connect.params.auth.password`
  El panel de configuración del dashboard te permite almacenar un token; las contraseñas no se guardan.
  El asistente de incorporación genera un token del Gateway de forma predeterminada, así que pégalo aquí en la primera conexión.

<div id="what-it-can-do-today">
  ## Lo que puede hacer (hoy)
</div>

* Chatear con el modelo a través de Gateway WS (`chat.history`, `chat.send`, `chat.abort`, `chat.inject`)
* Transmitir llamadas de herramientas + tarjetas con salida de herramientas en tiempo real en Chat (eventos de agente)
* Canales: estado de WhatsApp/Telegram/Discord/Slack + canales de complemento (Mattermost, etc.) + inicio de sesión por QR + configuración por canal (`channels.status`, `web.login.*`, `config.patch`)
* Instancias: lista de presencia + actualización (`system-presence`)
* Sesiones: lista + anulaciones por sesión de razonamiento/detallado (`sessions.list`, `sessions.patch`)
* Tareas cron: listar/agregar/ejecutar/habilitar/deshabilitar + historial de ejecuciones (`cron.*`)
* Habilidades: estado, habilitar/deshabilitar, instalación, actualizaciones de claves de API (`skills.*`)
* Nodos: lista + capacidades (`node.list`)
* Aprobaciones de ejecución: editar listas de permitidos de Gateway o nodo + consultar la política para `exec host=gateway/node` (`exec.approvals.*`)
* Configuración: ver/editar `~/.openclaw/openclaw.json` (`config.get`, `config.set`)
* Configuración: aplicar + reiniciar con validación (`config.apply`) y activar la última sesión activa
* Las escrituras de configuración incluyen una protección de hash base para evitar sobrescribir ediciones simultáneas
* Esquema de configuración + renderizado de formularios (`config.schema`, incluidos esquemas de complementos + canales); el editor JSON sin procesar sigue disponible
* Depuración: instantáneas de estado/salud/modelos + registro de eventos + llamadas RPC manuales (`status`, `health`, `models.list`)
* Registros: seguimiento en vivo de los registros de archivos del Gateway con filtrado/exportación (`logs.tail`)
* Actualización: ejecutar una actualización de paquete/git + reinicio (`update.run`) con un informe de reinicio

<div id="chat-behavior">
  ## Comportamiento del chat
</div>

* `chat.send` es **no bloqueante**: envía un acuse inmediato con `{ runId, status: "started" }` y la respuesta se transmite mediante eventos `chat`.
* Reenviar con la misma `idempotencyKey` devuelve `{ status: "in_flight" }` mientras se está ejecutando y `{ status: "ok" }` tras la finalización.
* `chat.inject` agrega una nota del asistente a la transcripción de la sesión y emite un evento `chat` para actualizaciones solo para la UI (sin ejecución de agente, sin entrega al canal).
* Detener:
  * Haz clic en **Stop** (llama a `chat.abort`)
  * Escribe `/stop` (o `stop|esc|abort|wait|exit|interrupt`) para abortar fuera de banda
  * `chat.abort` admite `{ sessionKey }` (sin `runId`) para abortar todas las ejecuciones activas de esa sesión

<div id="tailnet-access-recommended">
  ## Acceso a Tailnet (recomendado)
</div>

<div id="integrated-tailscale-serve-preferred">
  ### Tailscale Serve integrado (recomendado)
</div>

Mantén el Gateway en loopback y deja que Tailscale Serve actúe como proxy HTTPS:

```bash
openclaw gateway --tailscale serve
```

Abre:

* `https://<magicdns>/` (o tu `gateway.controlUi.basePath` configurado)

De forma predeterminada, las solicitudes de Serve pueden autenticarse mediante los encabezados de identidad de Tailscale
(`tailscale-user-login`) cuando `gateway.auth.allowTailscale` está en `true`. OpenClaw
verifica la identidad resolviendo la dirección `x-forwarded-for` con
`tailscale whois` y haciéndola coincidir con el encabezado, y solo las acepta cuando la
solicitud llega a loopback con los encabezados `x-forwarded-*` de Tailscale. Configura
`gateway.auth.allowTailscale: false` (o fuerza `gateway.auth.mode: "password"`)
si quieres exigir un token/contraseña incluso para el tráfico de Serve.

<div id="bind-to-tailnet-token">
  ### Vincular a la tailnet + token
</div>

```bash
openclaw gateway --bind tailnet --token "$(openssl rand -hex 32)"
```

Luego, abre:

* `http://<tailscale-ip>:18789/` (o el `gateway.controlUi.basePath` que hayas configurado)

Pega el token en la configuración de la UI (que se envía como `connect.params.auth.token`).

<div id="insecure-http">
  ## HTTP inseguro
</div>

Si abres el panel de control mediante HTTP sin cifrar (`http://<lan-ip>` o `http://<tailscale-ip>`),
el navegador se ejecuta en un **contexto no seguro** y bloquea WebCrypto. De forma predeterminada,
OpenClaw **bloquea** las conexiones de la Control UI sin identidad de dispositivo.

**Corrección recomendada:** usa HTTPS (Tailscale Serve) o abre la UI localmente:

* `https://<magicdns>/` (Serve)
* `http://127.0.0.1:18789/` (en el host del Gateway)

**Ejemplo de degradación (solo token mediante HTTP):**

```json5
{
  gateway: {
    controlUi: { allowInsecureAuth: true },
    bind: "tailnet",
    auth: { mode: "token", token: "replace-me" }
  }
}
```

Esto desactiva la identidad del dispositivo y el emparejamiento para el Control UI (incluso en HTTPS). Úsalo solo si confías en la red.

Consulta [Tailscale](/es/gateway/tailscale) para obtener instrucciones de configuración de HTTPS.

<div id="building-the-ui">
  ## Compilación de la UI
</div>

El Gateway sirve archivos estáticos desde `dist/control-ui`. Genera esos archivos con:

```bash
pnpm ui:build # instala automáticamente las dependencias de UI en la primera ejecución
```

Base absoluta opcional (cuando necesites URLs fijas para los recursos):

```bash
OPENCLAW_CONTROL_UI_BASE_PATH=/openclaw/ pnpm ui:build
```

Para el desarrollo local (servidor de desarrollo independiente):

```bash
pnpm ui:dev # instala automáticamente las dependencias de UI en la primera ejecución
```

Luego configura la UI para que apunte a la URL WS de tu Gateway (por ejemplo, `ws://127.0.0.1:18789`).

<div id="debuggingtesting-dev-server-remote-gateway">
  ## Depuración/pruebas: servidor de desarrollo + Gateway remoto
</div>

La Control UI se sirve como archivos estáticos; el destino WebSocket es configurable y puede ser
distinto del origen HTTP. Esto es útil cuando quieres tener el servidor de desarrollo de Vite
en tu máquina local pero el Gateway se ejecuta en otro lugar.

1. Inicia el servidor de desarrollo de la UI: `pnpm ui:dev`
2. Abre una URL como:

```text
http://localhost:5173/?gatewayUrl=ws://<gateway-host>:18789
```

Autenticación opcional de una sola vez (si es necesaria):

```text
http://localhost:5173/?gatewayUrl=wss://<gateway-host>:18789&token=<gateway-token>
```

Notas:

* `gatewayUrl` se almacena en localStorage tras la carga y se elimina de la URL.
* `token` se almacena en localStorage; `password` se mantiene solo en memoria.
* Usa `wss://` cuando el Gateway está detrás de TLS (Tailscale Serve, proxy HTTPS, etc.).

Detalles de la configuración de acceso remoto: [Acceso remoto](/es/gateway/remote).
