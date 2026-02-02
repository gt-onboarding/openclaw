---
title: Nodos
summary: "Nodos: emparejamiento, capacidades, permisos y utilidades de la CLI para canvas/camera/screen/system"
read_when:
  - Emparejar nodos iOS/Android con un Gateway
  - Usar canvas/camera de un nodo como contexto de un agente
  - Añadir nuevos comandos de nodo o utilidades de la CLI
---

<div id="nodes">
  # Nodos
</div>

Un **nodo** es un dispositivo complementario (macOS/iOS/Android/sin interfaz gráfica) que se conecta al **WebSocket** del Gateway (mismo puerto que los operadores) con `role: "node"` y expone una interfaz de comandos (por ejemplo, `canvas.*`, `camera.*`, `system.*`) mediante `node.invoke`. Detalles del protocolo: [Gateway protocol](/es/gateway/protocol).

Transporte heredado: [Bridge protocol](/es/gateway/bridge-protocol) (TCP JSONL; obsoleto/eliminado para los nodos actuales).

macOS también puede ejecutarse en **modo nodo**: la app de la barra de menús se conecta al servidor WS del Gateway y expone sus comandos locales de canvas/cámara como un nodo (de modo que `openclaw nodes …` funciona con este Mac).

Notas:

* Los nodos son **periféricos**, no Gateways. No ejecutan el servicio del Gateway.
* Los mensajes de Telegram/WhatsApp/etc. llegan al **Gateway**, no a los nodos.

<div id="pairing-status">
  ## Emparejamiento + estado
</div>

**Los nodos WS utilizan emparejamiento de dispositivo.** Los nodos presentan una identidad de dispositivo durante `connect`; el Gateway
crea una solicitud de emparejamiento de dispositivo para `role: node`. Aprueba la solicitud desde la CLI de dispositivos (o la UI).

CLI rápida:

```bash
openclaw devices list
openclaw devices approve <requestId>
openclaw devices reject <requestId>
openclaw nodes status
openclaw nodes describe --node <idOrNameOrIp>
```

Notas:

* `nodes status` marca un nodo como **paired** cuando su rol de emparejamiento de dispositivo incluye `node`.
* `node.pair.*` (CLI: `openclaw nodes pending/approve/reject`) es un almacén de emparejamiento de nodos independiente, propiedad del Gateway; **no** restringe ni condiciona el handshake WS `connect`.

<div id="remote-node-host-systemrun">
  ## Host de nodo remoto (system.run)
</div>

Usa un **host de nodo** cuando tu Gateway se ejecuta en una máquina y quieres ejecutar los comandos en otra. El modelo sigue comunicándose con el **Gateway**; el Gateway reenvía las llamadas `exec` al **host de nodo** cuando se selecciona `host=node`.

<div id="what-runs-where">
  ### Qué se ejecuta dónde
</div>

* **Host del Gateway**: recibe mensajes, ejecuta el modelo, enruta las llamadas a herramientas.
* **Host del nodo**: ejecuta `system.run`/`system.which` en la máquina del nodo.
* **Aprobaciones**: se aplican en el host del nodo mediante `~/.openclaw/exec-approvals.json`.

<div id="start-a-node-host-foreground">
  ### Iniciar un host de nodo (en primer plano)
</div>

En la máquina del nodo:

```bash
openclaw node run --host <gateway-host> --port 18789 --display-name "Build Node"
```

<div id="start-a-node-host-service">
  ### Iniciar el host de nodo (servicio)
</div>

```bash
openclaw node install --host <gateway-host> --port 18789 --display-name "Build Node"
openclaw node restart
```

<div id="pair-name">
  ### Emparejar + nombrar
</div>

En el host del Gateway:

```bash
openclaw nodes pending
openclaw nodes approve <requestId>
openclaw nodes list
```

Opciones para el nombre:

* `--display-name` en `openclaw node run` / `openclaw node install` (se guarda en `~/.openclaw/node.json` en el nodo).
* `openclaw nodes rename --node <id|name|ip> --name "Build Node"` (sobrescritura desde el Gateway).

<div id="allowlist-the-commands">
  ### Añadir los comandos a la lista de permitidos
</div>

Las aprobaciones de exec son **por cada host de nodo**. Añade entradas a la lista de permitidos desde el Gateway:

```bash
openclaw approvals allowlist add --node <id|name|ip> "/usr/bin/uname"
openclaw approvals allowlist add --node <id|name|ip> "/usr/bin/sw_vers"
```

Las aprobaciones se almacenan en el host del nodo en `~/.openclaw/exec-approvals.json`.

<div id="point-exec-at-the-node">
  ### Configura exec para apuntar al nodo
</div>

Configura los valores predeterminados (configuración del Gateway):

```bash
openclaw config set tools.exec.host node
openclaw config set tools.exec.security allowlist
openclaw config set tools.exec.node "<id-or-name>"
```

O bien por sesión:

```
/exec host=node security=allowlist node=<id-or-name>
```

Una vez establecido, cualquier llamada a `exec` con `host=node` se ejecuta en el host del nodo (sujeto a la lista de permitidos y aprobaciones del nodo).

Relacionado:

* [CLI del host del nodo](/es/cli/node)
* [Herramienta `exec`](/es/tools/exec)
* [Aprobaciones de `exec`](/es/tools/exec-approvals)

<div id="invoking-commands">
  ## Invocación de comandos
</div>

A bajo nivel (RPC sin procesar):

```bash
openclaw nodes invoke --node <idOrNameOrIp> --command canvas.eval --params '{"javaScript":"location.href"}'
```

Existen ayudantes de nivel superior para los flujos de trabajo habituales de «proporcionar al agente un adjunto MEDIA».

<div id="screenshots-canvas-snapshots">
  ## Capturas de pantalla (instantáneas del lienzo)
</div>

Si el nodo está mostrando Canvas (WebView), `canvas.snapshot` devuelve `{ format, base64 }`.

Utilidad de la CLI (escribe en un archivo temporal e imprime `MEDIA:<path>`):

```bash
openclaw nodes canvas snapshot --node <idOrNameOrIp> --format png
openclaw nodes canvas snapshot --node <idOrNameOrIp> --format jpg --max-width 1200 --quality 0.9
```

<div id="canvas-controls">
  ### Controles del lienzo
</div>

```bash
openclaw nodes canvas present --node <idOrNameOrIp> --target https://example.com
openclaw nodes canvas hide --node <idOrNameOrIp>
openclaw nodes canvas navigate https://example.com --node <idOrNameOrIp>
openclaw nodes canvas eval --node <idOrNameOrIp> --js "document.title"
```

Notas:

* `canvas present` acepta URL o rutas de archivo locales (`--target`), además de `--x/--y/--width/--height` opcionales para el posicionamiento.
* `canvas eval` acepta JavaScript en línea (`--js`) o un argumento posicional.

<div id="a2ui-canvas">
  ### A2UI (Canvas)
</div>

```bash
openclaw nodes canvas a2ui push --node <idOrNameOrIp> --text "Hello"
openclaw nodes canvas a2ui push --node <idOrNameOrIp> --jsonl ./payload.jsonl
openclaw nodes canvas a2ui reset --node <idOrNameOrIp>
```

Notas:

* Solo es compatible A2UI v0.8 JSONL (v0.9/createSurface se rechaza).

<div id="photos-videos-node-camera">
  ## Fotos y vídeos (cámara del nodo)
</div>

Fotos (`jpg`):

```bash
openclaw nodes camera list --node <idOrNameOrIp>
openclaw nodes camera snap --node <idOrNameOrIp>            # por defecto: ambas orientaciones (2 líneas MEDIA)
openclaw nodes camera snap --node <idOrNameOrIp> --facing front
```

Clips de vídeo (`mp4`):

```bash
openclaw nodes camera clip --node <idOrNameOrIp> --duration 10s
openclaw nodes camera clip --node <idOrNameOrIp> --duration 3000 --no-audio
```

Notas:

* El nodo debe estar **en primer plano** para `canvas.*` y `camera.*` (las llamadas en segundo plano devuelven `NODE_BACKGROUND_UNAVAILABLE`).
* La duración del clip está limitada (actualmente `<= 60s`) para evitar cargas base64 demasiado grandes.
* Android solicitará permisos de `CAMERA`/`RECORD_AUDIO` cuando sea posible; si los permisos se deniegan, se producirá un error con `*_PERMISSION_REQUIRED`.

<div id="screen-recordings-nodes">
  ## Grabaciones de pantalla (nodos)
</div>

Los nodos exponen `screen.record` (MP4). Ejemplo:

```bash
openclaw nodes screen record --node <idOrNameOrIp> --duration 10s --fps 10
openclaw nodes screen record --node <idOrNameOrIp> --duration 10s --fps 10 --no-audio
```

Notas:

* `screen.record` requiere que la aplicación nodo esté en primer plano.
* Android mostrará el aviso del sistema para la captura de pantalla antes de comenzar a grabar.
* Las grabaciones de pantalla se limitan a un máximo de `<= 60s`.
* `--no-audio` desactiva la captura del micrófono (compatible con iOS/Android; macOS usa el audio de captura del sistema).
* Utiliza `--screen <index>` para seleccionar una pantalla cuando haya varias pantallas disponibles.

<div id="location-nodes">
  ## Ubicación (nodos)
</div>

Los nodos exponen `location.get` cuando Location está activado en la configuración.

Ayuda de la CLI:

```bash
openclaw nodes location get --node <idOrNameOrIp>
openclaw nodes location get --node <idOrNameOrIp> --accuracy precise --max-age 15000 --location-timeout 10000
```

Notas:

* La ubicación está **desactivada por defecto**.
* “Siempre” requiere permiso del sistema; la obtención en segundo plano se realiza sin garantías.
* La respuesta incluye lat/lon, precisión (metros) y marca de tiempo.

<div id="sms-android-nodes">
  ## SMS (nodos Android)
</div>

Los nodos Android pueden exponer `sms.send` cuando el usuario otorga el permiso **SMS** y el dispositivo es compatible con funciones de telefonía.

Invocación de bajo nivel:

```bash
openclaw nodes invoke --node <idOrNameOrIp> --command sms.send --params '{"to":"+15555550123","message":"Hello from OpenClaw"}'
```

Notas:

* La solicitud de permisos debe aceptarse en el dispositivo Android antes de que se anuncie la capacidad.
* Los dispositivos solo Wi‑Fi sin telefonía no anunciarán `sms.send`.

<div id="system-commands-node-host-mac-node">
  ## Comandos del sistema (host del nodo / nodo de Mac)
</div>

El nodo de macOS expone `system.run`, `system.notify` y `system.execApprovals.get/set`.
El host del nodo sin interfaz gráfica expone `system.run`, `system.which` y `system.execApprovals.get/set`.

Ejemplos:

```bash
openclaw nodes run --node <idOrNameOrIp> -- echo "Hello from mac node"
openclaw nodes notify --node <idOrNameOrIp> --title "Ping" --body "Gateway ready"
```

Notas:

* `system.run` devuelve stdout/stderr/código de salida en la carga útil.
* `system.notify` respeta el estado de permisos de notificación en la app de macOS.
* `system.run` admite `--cwd`, `--env KEY=VAL`, `--command-timeout` y `--needs-screen-recording`.
* `system.notify` admite `--priority <passive|active|timeSensitive>` y `--delivery <system|overlay|auto>`.
* Los nodos de macOS descartan las anulaciones de `PATH`; los hosts de nodo sin interfaz solo aceptan `PATH` cuando este antepone el PATH del host del nodo.
* En modo nodo de macOS, `system.run` está controlado por aprobaciones de ejecución en la app de macOS (Settings → Exec approvals).
  `ask`/`allowlist`/`full` se comportan igual que en el host de nodo sin interfaz; las solicitudes denegadas devuelven `SYSTEM_RUN_DENIED`.
* En el host de nodo sin interfaz, `system.run` está controlado por aprobaciones de ejecución (`~/.openclaw/exec-approvals.json`).

<div id="exec-node-binding">
  ## Asignación de nodo para exec
</div>

Cuando hay varios nodos disponibles, puedes asignar exec a un nodo específico.
Esto establece el nodo predeterminado para `exec host=node` (y se puede anular por agente).

Predeterminado global:

```bash
openclaw config set tools.exec.node "node-id-or-name"
```

Anulación por agente:

```bash
openclaw config get agents.list
openclaw config set agents.list[0].tools.exec.node "node-id-or-name"
```

Déjalo sin configurar para permitir cualquier nodo:

```bash
openclaw config unset tools.exec.node
openclaw config unset agents.list[0].tools.exec.node
```

<div id="permissions-map">
  ## Mapa de permisos
</div>

Los nodos pueden incluir un mapa `permissions` en `node.list` / `node.describe`, cuyas claves son nombres de permisos (por ejemplo, `screenRecording`, `accessibility`) con valores booleanos (`true` = concedido).

<div id="headless-node-host-cross-platform">
  ## Host de nodo headless (multiplataforma)
</div>

OpenClaw puede ejecutar un **host de nodo headless** (sin UI) que se conecta al WebSocket del Gateway
y expone `system.run` / `system.which`. Esto es útil en Linux o Windows,
o para ejecutar un nodo mínimo junto a un servidor.

Inícialo:

```bash
openclaw node run --host <gateway-host> --port 18789
```

Notas:

* El emparejamiento sigue siendo necesario (el Gateway mostrará un aviso de aprobación de nodo).
* El host del nodo almacena su ID de nodo, token, nombre para mostrar e información de conexión al Gateway en `~/.openclaw/node.json`.
* Las aprobaciones de ejecución se aplican localmente mediante `~/.openclaw/exec-approvals.json`
  (consulta [Exec approvals](/es/tools/exec-approvals)).
* En macOS, el host de nodo en modo headless prefiere el host de ejecución de la app compañera cuando es accesible y recurre
  a la ejecución local si la app no está disponible. Establece `OPENCLAW_NODE_EXEC_HOST=app` para exigir
  el uso de la app, o `OPENCLAW_NODE_EXEC_FALLBACK=0` para desactivar el fallback.
* Añade `--tls` / `--tls-fingerprint` cuando el WS del Gateway utilice TLS.

<div id="mac-node-mode">
  ## Modo de nodo en Mac
</div>

* La aplicación de la barra de menús de macOS se conecta al servidor WS del Gateway como nodo (para que `openclaw nodes …` funcione con este Mac).
* En modo remoto, la aplicación abre un túnel SSH para el puerto del Gateway y se conecta a `localhost`.