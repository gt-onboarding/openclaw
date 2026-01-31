---
title: Comandos slash
summary: "Comandos slash: de texto vs nativos, configuración y comandos disponibles"
read_when:
  - Cuando uses o configures comandos de chat
  - Cuando depures el enrutamiento de comandos o sus permisos
---

<div id="slash-commands">
  # Comandos slash
</div>

Los comandos los gestiona el Gateway. La mayoría de los comandos deben enviarse como un mensaje **independiente** que comience con `/`.
El comando de chat bash solo en el host usa `! <cmd>` (con `/bash <cmd>` como alias).

Hay dos sistemas relacionados:

* **Comandos**: mensajes independientes `/...`.
* **Directivas**: `/think`, `/verbose`, `/reasoning`, `/elevated`, `/exec`, `/model`, `/queue`.
  * Las directivas se eliminan del mensaje antes de que el modelo lo vea.
  * En mensajes de chat normales (no solo directivas), se tratan como “sugerencias en línea” y **no** mantienen la configuración de la sesión.
  * En mensajes que solo contienen directivas (el mensaje contiene únicamente directivas), estas persisten en la sesión y responden con un acuse de recibo.
  * Las directivas solo se aplican para **remitentes autorizados** (lista de permitidos de canal/emparejamiento más `commands.useAccessGroups`).
    Los remitentes no autorizados ven las directivas tratadas como texto plano.

También hay algunos **atajos en línea** (solo remitentes en lista de permitidos/autorizados): `/help`, `/commands`, `/status`, `/whoami` (`/id`).
Se ejecutan inmediatamente, se eliminan antes de que el modelo vea el mensaje y el texto restante continúa por el flujo normal.

<div id="config">
  ## Configuración
</div>

```json5
{
  commands: {
    native: "auto",
    nativeSkills: "auto",
    text: true,
    bash: false,
    bashForegroundMs: 2000,
    config: false,
    debug: false,
    restart: false,
    useAccessGroups: true
  }
}
```

* `commands.text` (valor predeterminado `true`) habilita el análisis de `/...` en mensajes de chat.
  * En plataformas sin comandos nativos (WhatsApp/WebChat/Signal/iMessage/Google Chat/MS Teams), los comandos de texto siguen funcionando incluso si configuras este valor en `false`.
* `commands.native` (valor predeterminado `"auto"`) registra comandos nativos.
  * Auto: activado para Discord/Telegram; desactivado para Slack (hasta que añadas comandos de barra diagonal); se ignora para proveedores sin compatibilidad nativa.
  * Configura `channels.discord.commands.native`, `channels.telegram.commands.native` o `channels.slack.commands.native` para anularlo por proveedor (bool o `"auto"`).
  * `false` borra los comandos previamente registrados en Discord/Telegram al inicio. Los comandos de Slack se gestionan en la app de Slack y no se eliminan automáticamente.
* `commands.nativeSkills` (valor predeterminado `"auto"`) registra comandos de **habilidad** de forma nativa cuando hay compatibilidad.
  * Auto: activado para Discord/Telegram; desactivado para Slack (Slack requiere crear un comando de barra diagonal por habilidad).
  * Configura `channels.discord.commands.nativeSkills`, `channels.telegram.commands.nativeSkills` o `channels.slack.commands.nativeSkills` para anularlo por proveedor (bool o `"auto"`).
* `commands.bash` (valor predeterminado `false`) habilita `! <cmd>` para ejecutar comandos de shell del host (`/bash <cmd>` es un alias; requiere la lista de permitidos `tools.elevated`).
* `commands.bashForegroundMs` (valor predeterminado `2000`) controla cuánto tiempo espera bash antes de cambiar al modo en segundo plano (`0` lo pasa a segundo plano inmediatamente).
* `commands.config` (valor predeterminado `false`) habilita `/config` (lee/escribe `openclaw.json`).
* `commands.debug` (valor predeterminado `false`) habilita `/debug` (anulaciones solo en tiempo de ejecución).
* `commands.useAccessGroups` (valor predeterminado `true`) aplica la lista de permitidos y las políticas para los comandos.

<div id="command-list">
  ## Lista de comandos
</div>

Texto + nativo (cuando está habilitado):

* `/help`
* `/commands`
* `/skill <name> [input]` (ejecuta una skill por nombre)
* `/status` (muestra el estado actual; incluye uso/cuota del proveedor para el proveedor de modelo actual cuando esté disponible)
* `/allowlist` (listar/agregar/eliminar entradas de la lista de permitidos)
* `/approve <id> allow-once|allow-always|deny` (resuelve solicitudes de aprobación de ejecución)
* `/context [list|detail|json]` (explica el “context”; `detail` muestra el tamaño por archivo + por herramienta + por skill + prompt de sistema)
* `/whoami` (muestra tu id de remitente; alias: `/id`)
* `/subagents list|stop|log|info|send` (inspecciona, detiene, registra o envía mensajes a ejecuciones de subagentes para la sesión actual)
* `/config show|get|set|unset` (persiste la configuración en disco, solo para el propietario; requiere `commands.config: true`)
* `/debug show|set|unset|reset` (overrides en tiempo de ejecución, solo para el propietario; requiere `commands.debug: true`)
* `/usage off|tokens|full|cost` (pie de uso por respuesta o resumen local de coste)
* `/tts off|always|inbound|tagged|status|provider|limit|summary|audio` (controla TTS; ver [/tts](/es/tts))
  * Discord: el comando nativo es `/voice` (Discord reserva `/tts`); el comando de texto `/tts` sigue funcionando.
* `/stop`
* `/restart`
* `/dock-telegram` (alias: `/dock_telegram`) (cambia las respuestas a Telegram)
* `/dock-discord` (alias: `/dock_discord`) (cambia las respuestas a Discord)
* `/dock-slack` (alias: `/dock_slack`) (cambia las respuestas a Slack)
* `/activation mention|always` (solo grupos)
* `/send on|off|inherit` (solo para el propietario)
* `/reset` o `/new [model]` (sugerencia de modelo opcional; el resto se pasa tal cual)
* `/think <off|minimal|low|medium|high|xhigh>` (opciones dinámicas según modelo/proveedor; alias: `/thinking`, `/t`)
* `/verbose on|full|off` (alias: `/v`)
* `/reasoning on|off|stream` (alias: `/reason`; cuando está activado, envía un mensaje separado con el prefijo `Reasoning:`; `stream` = solo borrador de Telegram)
* `/elevated on|off|ask|full` (alias: `/elev`; `full` omite aprobaciones de ejecución)
* `/exec host=<sandbox|gateway|node> security=<deny|allowlist|full> ask=<off|on-miss|always> node=<id>` (envía `/exec` para mostrar la configuración actual)
* `/model <name>` (alias: `/models`; o `/<alias>` desde `agents.defaults.models.*.alias`)
* `/queue <mode>` (más opciones como `debounce:2s cap:25 drop:summarize`; envía `/queue` para ver la configuración actual)
* `/bash <command>` (solo host; alias de `! <command>`; requiere `commands.bash: true` + listas de permitidos de `tools.elevated`)

Solo texto:

* `/compact [instructions]` (ver [/concepts/compaction](/es/concepts/compaction))
* `! <command>` (solo host; uno a la vez; usa `!poll` + `!stop` para trabajos de larga duración)
* `!poll` (consulta la salida/estado; acepta `sessionId` opcional; `/bash poll` también funciona)
* `!stop` (detiene la tarea de bash en ejecución; acepta `sessionId` opcional; `/bash stop` también funciona)

Notas:

* Los comandos aceptan un `:` opcional entre el comando y los argumentos (por ejemplo, `/think: high`, `/send: on`, `/help:`).
* `/new <model>` acepta un alias de modelo, `provider/model` o un nombre de proveedor (coincidencia difusa); si no hay coincidencia, el texto se trata como cuerpo del mensaje.
* Para un desglose completo del uso por proveedor, usa `openclaw status --usage`.
* `/allowlist add|remove` requiere `commands.config=true` y respeta `configWrites` del canal.
* `/usage` controla el pie de uso por respuesta; `/usage cost` imprime un resumen local de costos a partir de los registros de sesión de OpenClaw.
* `/restart` está deshabilitado de forma predeterminada; establece `commands.restart: true` para habilitarlo.
* `/verbose` está pensado para depuración y visibilidad adicional; mantenlo **apagado** en el uso normal.
* `/reasoning` (y `/verbose`) son arriesgados en entornos de grupo: pueden revelar razonamientos internos o salida de herramientas que no pretendías exponer. Es preferible dejarlos desactivados, especialmente en chats de grupo.
* **Ruta rápida:** los mensajes que solo contienen comandos de remitentes en la lista de permitidos se manejan de inmediato (se saltan la cola y el modelo).
* **Control por mención en grupos:** los mensajes que solo contienen comandos de remitentes en la lista de permitidos omiten los requisitos de mención.
* **Atajos en línea (solo remitentes en la lista de permitidos):** ciertos comandos también funcionan cuando se incrustan en un mensaje normal y se eliminan antes de que el modelo vea el texto restante.
  * Ejemplo: `hey /status` genera una respuesta de estado, y el texto restante continúa por el flujo normal.
* Actualmente: `/help`, `/commands`, `/status`, `/whoami` (`/id`).
* Los mensajes no autorizados que solo contienen comandos se ignoran sin mostrar ningún aviso, y los tokens `/...` en línea se tratan como texto sin formato.
* **Comandos de habilidades:** las habilidades `user-invocable` se exponen como comandos slash. Los nombres se normalizan a `a-z0-9_` (máx. 32 caracteres); las colisiones reciben sufijos numéricos (por ejemplo, `_2`).
  * `/skill <name> [input]` ejecuta una habilidad por nombre (útil cuando los límites de comandos nativos impiden comandos por habilidad).
  * De forma predeterminada, los comandos de habilidades se reenvían al modelo como una solicitud normal.
  * Las habilidades pueden declarar opcionalmente `command-dispatch: tool` para enrutar el comando directamente a una herramienta (determinista, sin modelo).
  * Ejemplo: `/prose` (complemento OpenProse): consulta [OpenProse](/es/prose).
* **Argumentos de comandos nativos:** Discord usa autocompletado para opciones dinámicas (y menús de botones cuando omites argumentos obligatorios). Telegram y Slack muestran un menú de botones cuando un comando admite opciones y omites el argumento.

<div id="usage-surfaces-what-shows-where">
  ## Superficies de uso (qué se muestra y dónde)
</div>

* El **uso/cuota del proveedor** (ejemplo: “Claude 80% restante”) aparece en `/status` para el proveedor de modelo actual cuando el seguimiento de uso está habilitado.
* Los **tokens/coste por respuesta** se controlan con `/usage off|tokens|full` (se añade a las respuestas normales).
* `/model status` trata sobre **modelos/autenticación/endpoints**, no sobre uso.

<div id="model-selection-model">
  ## Selección de modelo (`/model`)
</div>

`/model` se implementa como una directiva.

Ejemplos:

```
/model
/model list
/model 3
/model openai/gpt-5.2
/model opus@anthropic:default
/model status
```

Notas:

* `/model` y `/model list` muestran un selector numerado y compacto (familia de modelo + proveedores disponibles).
* `/model <#>` selecciona una entrada de ese selector (y prefiere el proveedor actual cuando es posible).
* `/model status` muestra la vista detallada, incluida la URL de endpoint del proveedor configurado (`baseUrl`) y el modo de api (`api`) cuando está disponible.

<div id="debug-overrides">
  ## Anulaciones de depuración
</div>

`/debug` te permite definir anulaciones de configuración **solo en tiempo de ejecución** (en memoria, no en disco). Solo disponible para el propietario. Desactivado de forma predeterminada; actívalo con `commands.debug: true`.

Ejemplos:

```
/debug show
/debug set messages.responsePrefix="[openclaw]"
/debug set channels.whatsapp.allowFrom=["+1555","+4477"]
/debug unset messages.responsePrefix
/debug reset
```

Notas:

* Las anulaciones se aplican de inmediato a las nuevas lecturas de configuración, pero **no** se guardan en `openclaw.json`.
* Usa `/debug reset` para borrar todas las anulaciones y volver a la configuración en disco.

<div id="config-updates">
  ## Actualizaciones de configuración
</div>

`/config` escribe en tu configuración en disco (`openclaw.json`). Solo para el propietario. Está deshabilitado de forma predeterminada; actívalo con `commands.config: true`.

Ejemplos:

```
/config show
/config show messages.responsePrefix
/config get messages.responsePrefix
/config set messages.responsePrefix="[openclaw]"
/config unset messages.responsePrefix
```

Notas:

* La configuración se valida antes de guardarse; los cambios no válidos se rechazan.
* Las actualizaciones de `/config` persisten entre reinicios.

<div id="surface-notes">
  ## Notas generales
</div>

* Los **comandos de texto** se ejecutan en la sesión de chat normal (los mensajes directos (DMs) comparten `main`, los grupos tienen su propia sesión).
* Los **comandos nativos** usan sesiones aisladas:
  * Discord: `agent:<agentId>:discord:slash:<userId>`
  * Slack: `agent:<agentId>:slack:slash:<userId>` (prefijo configurable mediante `channels.slack.slashCommand.sessionPrefix`)
  * Telegram: `telegram:slash:<userId>` (apunta a la sesión del chat mediante `CommandTargetSessionKey`)
* **`/stop`** se dirige a la sesión de chat activa para poder abortar la ejecución actual.
* **Slack:** `channels.slack.slashCommand` se sigue admitiendo para un único comando al estilo `/openclaw`. Si habilitas `commands.native`, debes crear un comando slash de Slack por cada comando integrado (mismos nombres que `/help`). Los menús de argumentos de comandos para Slack se entregan como botones efímeros de Block Kit.