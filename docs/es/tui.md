---
title: TUI
summary: "Interfaz de terminal (TUI): conéctate al Gateway desde cualquier equipo"
read_when:
  - Quieres un recorrido para principiantes por la TUI
  - Necesitas la lista completa de funcionalidades, comandos y atajos de teclado de la TUI
---

<div id="tui-terminal-ui">
  # TUI (interfaz de usuario de terminal)
</div>

<div id="quick-start">
  ## Inicio rápido
</div>

1. Inicia el Gateway.

```bash
openclaw gateway
```

2. Abre la TUI.

```bash
openclaw tui
```

3. Escribe un mensaje y pulsa la tecla Enter.

Gateway remoto:

```bash
openclaw tui --url ws://<host>:<port> --token <gateway-token>
```

Usa `--password` si tu Gateway utiliza autenticación con contraseña.

<div id="what-you-see">
  ## Lo que ves
</div>

* Encabezado: URL de conexión, agente actual, sesión actual.
* Historial de chat: mensajes del usuario, respuestas del asistente, avisos del sistema, tarjetas de herramientas.
* Línea de estado: estado de conexión/ejecución (conectando, ejecutando, transmitiendo, inactivo, error).
* Pie de página: estado de conexión + agente + sesión + modelo + think/verbose/reasoning + recuento de tokens + deliver.
* Entrada: editor de texto con autocompletado.

<div id="mental-model-agents-sessions">
  ## Modelo mental: agentes + sesiones
</div>

* Los agentes son *slugs* únicos (p. ej., `main`, `research`). El Gateway expone la lista.
* Las sesiones pertenecen al agente actual.
* Las claves de sesión se almacenan como `agent:<agentId>:<sessionKey>`.
  * Si escribes `/session main`, la TUI lo expande a `agent:<currentAgent>:main`.
  * Si escribes `/session agent:other:main`, cambias explícitamente a la sesión de ese agente.
* Ámbito de la sesión:
  * `per-sender` (predeterminado): cada agente tiene muchas sesiones.
  * `global`: la TUI siempre usa la sesión `global` (el selector puede estar vacío).
* El agente y la sesión actuales siempre son visibles en el pie de página.

<div id="sending-delivery">
  ## Envío y entrega
</div>

* Los mensajes se envían al Gateway; la entrega a proveedores está desactivada por defecto.
* Activa la entrega:
  * `/deliver on`
  * o el panel de configuración (Settings)
  * o inicia con `openclaw tui --deliver`

<div id="pickers-overlays">
  ## Selectores + superposiciones
</div>

* Selector de modelo: lista los modelos disponibles y establece la anulación de la sesión.
* Selector de agente: elige un agente diferente.
* Selector de sesión: muestra solo las sesiones del agente actual.
* Configuración: activa o desactiva entrega, expansión de la salida de herramientas y visibilidad del razonamiento.

<div id="keyboard-shortcuts">
  ## Atajos de teclado
</div>

* Enter: enviar mensaje
* Esc: abortar ejecución activa
* Ctrl+C: borrar entrada (pulsa dos veces para salir)
* Ctrl+D: salir
* Ctrl+L: selector de modelo
* Ctrl+G: selector de agente
* Ctrl+P: selector de sesión
* Ctrl+O: alternar expansión de la salida de la herramienta
* Ctrl+T: alternar visibilidad del razonamiento (recarga el historial)

<div id="slash-commands">
  ## Comandos slash
</div>

Básicos:

* `/help`
* `/status`
* `/agent <id>` (o `/agents`)
* `/session <key>` (o `/sessions`)
* `/model <provider/model>` (o `/models`)

Controles de sesión:

* `/think <off|minimal|low|medium|high>`
* `/verbose <on|full|off>`
* `/reasoning <on|off|stream>`
* `/usage <off|tokens|full>`
* `/elevated <on|off|ask|full>` (alias: `/elev`)
* `/activation <mention|always>`
* `/deliver <on|off>`

Ciclo de vida de la sesión:

* `/new` o `/reset` (restablece la sesión)
* `/abort` (aborta la ejecución activa)
* `/settings`
* `/exit`

Otros comandos slash del Gateway (por ejemplo, `/context`) se reenvían al Gateway y se muestran como salida del sistema. Consulta [Comandos slash](/es/tools/slash-commands).

<div id="local-shell-commands">
  ## Comandos de shell locales
</div>

* Antepone una línea con `!` para ejecutar un comando de shell local en el host de la TUI.
* La TUI te pedirá una vez por sesión permitir la ejecución local; si rechazas, `!` permanece deshabilitado durante la sesión.
* Los comandos se ejecutan en un shell nuevo y no interactivo en el directorio de trabajo de la TUI (sin `cd` ni entorno persistente).
* Un `!` aislado se envía como un mensaje normal; los espacios iniciales no activan la ejecución local.

<div id="tool-output">
  ## Salida de herramientas
</div>

* Las invocaciones de herramientas se muestran como tarjetas con argumentos y resultados.
* Ctrl+O alterna entre vistas plegadas y desplegadas.
* Mientras se ejecutan las herramientas, las actualizaciones parciales se transmiten en streaming dentro de la misma tarjeta.

<div id="history-streaming">
  ## Historial + streaming
</div>

* Al conectarse, la TUI carga el historial más reciente (por defecto 200 mensajes).
* Las respuestas en streaming se actualizan en el mismo sitio hasta que se finalizan.
* La TUI también escucha eventos de herramientas del agente para mostrar tarjetas de herramientas más completas.

<div id="connection-details">
  ## Detalles de conexión
</div>

* La TUI se registra en el Gateway como `mode: "tui"`.
* Las reconexiones muestran un mensaje del sistema; los huecos de eventos se muestran en el log.

<div id="options">
  ## Opciones
</div>

* `--url <url>`: URL WS del Gateway (por defecto usa la configuración o `ws://127.0.0.1:<port>`)
* `--token <token>`: Token del Gateway (si es necesario)
* `--password <password>`: Contraseña del Gateway (si es necesario)
* `--session <key>`: Clave de sesión (predeterminado: `main`, o `global` cuando scope es `global`)
* `--deliver`: Entregar las respuestas del asistente al proveedor (desactivado por defecto)
* `--thinking <level>`: Sobrescribe el nivel de thinking para send
* `--timeout-ms <ms>`: Tiempo de espera del agente en ms (por defecto: `agents.defaults.timeoutSeconds`)

<div id="troubleshooting">
  ## Solución de problemas
</div>

Sin respuesta después de enviar un mensaje:

* Ejecuta `/status` en la TUI para confirmar que el Gateway está conectado y si está inactivo u ocupado.
* Revisa los registros del Gateway: `openclaw logs --follow`.
* Confirma que el agente puede ejecutarse: `openclaw status` y `openclaw models status`.
* Si esperas mensajes en un canal de chat, habilita la entrega de mensajes (`/deliver on` o `--deliver`).
* `--history-limit &lt;n&gt;`: Número de entradas de historial que se cargarán (valor predeterminado: 200)

<div id="troubleshooting">
  ## Solución de problemas
</div>

* `disconnected`: asegúrate de que el Gateway esté en ejecución y de que tus `--url/--token/--password` sean correctos.
* No hay agentes en el selector: comprueba `openclaw agents list` y tu configuración de enrutamiento.
* Selector de sesión vacío: es posible que estés en un ámbito global o que aún no tengas sesiones.