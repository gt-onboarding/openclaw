---
title: Tareas cron
summary: "Tareas cron y activaciones para el planificador del Gateway"
read_when:
  - Programar trabajos en segundo plano o activaciones
  - Conectar automatizaciones que deban ejecutarse con o junto con el latido
  - Decidir entre latido y cron para tareas programadas
---

<div id="cron-jobs-gateway-scheduler">
  # Trabajos cron (planificador del Gateway)
</div>

> **¿Cron vs latido?** Consulta [Cron vs Heartbeat](/es/automation/cron-vs-heartbeat) para obtener orientación sobre cuándo usar cada uno.

Cron es el planificador integrado del Gateway. Mantiene las tareas, despierta al agente en
el momento adecuado y, opcionalmente, puede devolver la salida a un chat.

Si quieres *«ejecuta esto todas las mañanas»* o *«avisa al agente en 20 minutos»*,
cron es el mecanismo.

<div id="tldr">
  ## TL;DR
</div>

* Cron se ejecuta **dentro del Gateway** (no dentro del modelo).
* Las tareas persisten en `~/.openclaw/cron/`, por lo que los reinicios no pierden las programaciones.
* Dos estilos de ejecución:
  * **Sesión principal**: pone en cola un evento del sistema y luego se ejecuta en el próximo latido.
  * **Aislado**: ejecuta el turno de un agente dedicado en `cron:<jobId>` y, opcionalmente, entrega la salida.
* Las activaciones son de primera clase: una tarea puede solicitar «despertar ahora» en lugar de «próximo latido».

<div id="beginner-friendly-overview">
  ## Descripción básica para principiantes
</div>

Piensa en un cron job como: **cuándo** ejecutar + **qué** hacer.

1. **Elige una programación**
   * Recordatorio de una sola vez → `schedule.kind = "at"` (CLI: `--at`)
   * Trabajo repetitivo → `schedule.kind = "every"` o `schedule.kind = "cron"`
   * Si tu marca de tiempo ISO omite una zona horaria, se interpreta como **UTC**.

2. **Elige dónde se ejecuta**
   * `sessionTarget: "main"` → se ejecuta durante el siguiente latido con el contexto principal.
   * `sessionTarget: "isolated"` → ejecuta una interacción dedicada del agente en `cron:<jobId>`.

3. **Elige el payload**
   * Sesión principal → `payload.kind = "systemEvent"`
   * Sesión aislada → `payload.kind = "agentTurn"`

Opcional: `deleteAfterRun: true` elimina del almacenamiento los trabajos de una sola vez que hayan finalizado correctamente.

<div id="concepts">
  ## Conceptos
</div>

<div id="jobs">
  ### Jobs
</div>

Un cron job es un registro almacenado con:

* una **planificación** (cuándo debe ejecutarse),
* una **carga útil** (qué debe hacer),
* una **entrega** opcional (dónde se debe enviar la salida).
* una **asignación de agente** opcional (`agentId`): ejecuta el job bajo un agente específico; si
  falta o es desconocido, el Gateway recurre al agente predeterminado.

Los jobs se identifican mediante un `jobId` estable (usado por las APIs de CLI/Gateway).
En las llamadas a herramientas de agente, `jobId` es canónico; el `id` heredado se acepta por compatibilidad.
Los jobs pueden, opcionalmente, eliminarse automáticamente después de una ejecución única y satisfactoria mediante `deleteAfterRun: true`.

<div id="schedules">
  ### Programaciones
</div>

Cron admite tres tipos de programación:

* `at`: marca de tiempo para una única ejecución (ms desde el epoch). Gateway acepta ISO 8601 y la convierte a UTC.
* `every`: intervalo fijo (ms).
* `cron`: expresión cron de 5 campos con zona horaria IANA opcional.

Las expresiones cron usan `croner`. Si se omite una zona horaria, se usa la zona horaria local del host de Gateway.

<div id="main-vs-isolated-execution">
  ### Ejecución principal frente a aislada
</div>

<div id="main-session-jobs-system-events">
  #### Trabajos principales de sesión (eventos del sistema)
</div>

Los trabajos principales ponen en cola un evento del sistema y, opcionalmente, despiertan el ejecutor de latido.
Deben usar `payload.kind = "systemEvent"`.

* `wakeMode: "next-heartbeat"` (predeterminado): el evento espera al siguiente latido programado.
* `wakeMode: "now"`: el evento dispara una ejecución de latido inmediata.

Esta opción es la más adecuada cuando necesitas el prompt de latido normal + el contexto de la sesión principal.
Consulta [Latido](/es/gateway/heartbeat).

<div id="isolated-jobs-dedicated-cron-sessions">
  #### Trabajos aislados (sesiones de cron dedicadas)
</div>

Los trabajos aislados ejecutan un turno de agente dedicado en la sesión `cron:&lt;jobId&gt;`.

Comportamientos clave:

* El prompt se prefija con `[cron:&lt;jobId&gt; &lt;job name&gt;]` para trazabilidad.
* Cada ejecución inicia un **nuevo ID de sesión** (sin arrastrar conversación previa).
* Se publica un resumen en la sesión principal (prefijo `Cron`, configurable).
* `wakeMode: "now"` desencadena un latido inmediato después de publicar el resumen.
* Si `payload.deliver: true`, la salida se entrega a un canal; de lo contrario, permanece interna.

Utiliza trabajos aislados para tareas ruidosas, frecuentes o &quot;tareas en segundo plano&quot; que no deberían saturar
tu historial principal de chat.

<div id="payload-shapes-what-runs">
  ### Formas del payload (qué se ejecuta)
</div>

Se admiten dos tipos de payload:

* `systemEvent`: solo para la sesión principal, enrutado a través del prompt de latido.
* `agentTurn`: solo para sesión aislada, ejecuta un turno dedicado de agente.

Campos comunes de `agentTurn`:

* `message`: prompt de texto obligatorio.
* `model` / `thinking`: sobrescrituras opcionales (ver más abajo).
* `timeoutSeconds`: sobrescritura opcional del tiempo de espera.
* `deliver`: `true` para enviar la salida a un destino de canal.
* `channel`: `last` o un canal específico.
* `to`: destino específico del canal (teléfono/chat/id de canal).
* `bestEffortDeliver`: evita que la tarea falle si falla la entrega.

Opciones de aislamiento (solo para `session=isolated`):

* `postToMainPrefix` (CLI: `--post-prefix`): prefijo para el evento de sistema en la sesión principal.
* `postToMainMode`: `summary` (predeterminado) o `full`.
* `postToMainMaxChars`: número máximo de caracteres cuando `postToMainMode=full` (predeterminado 8000).

<div id="model-and-thinking-overrides">
  ### Sobrescrituras de modelo y nivel de razonamiento
</div>

Los jobs aislados (`agentTurn`) pueden sobrescribir el modelo y el nivel de razonamiento:

* `model`: Cadena de proveedor/modelo (por ejemplo, `anthropic/claude-sonnet-4-20250514`) o alias (por ejemplo, `opus`)
* `thinking`: Nivel de razonamiento (`off`, `minimal`, `low`, `medium`, `high`, `xhigh`; solo modelos GPT-5.2 + Codex)

Nota: También puedes configurar `model` en jobs de la sesión principal, pero esto cambia el modelo compartido de la sesión principal. Recomendamos usar sobrescrituras de modelo solo para jobs aislados para evitar cambios de contexto inesperados.

Prioridad de resolución:

1. Sobrescritura en el payload del job (máxima prioridad)
2. Valores predeterminados específicos del hook (por ejemplo, `hooks.gmail.model`)
3. Valor predeterminado de la configuración del Agente

<div id="delivery-channel-target">
  ### Entrega (canal + destino)
</div>

Los trabajos aislados pueden enviar salida a un canal. La carga útil del trabajo puede especificar:

* `channel`: `whatsapp` / `telegram` / `discord` / `slack` / `mattermost` (complemento) / `signal` / `imessage` / `last`
* `to`: destino/receptor específico del canal

Si se omiten `channel` o `to`, cron puede recurrir a la “última ruta” de la sesión principal
(la última ubicación donde respondió el agente).

Notas sobre la entrega:

* Si `to` está definido, cron entrega automáticamente la salida final del agente incluso si se omite `deliver`.
* Usa `deliver: true` cuando quieras entrega usando la última ruta sin un `to` explícito.
* Usa `deliver: false` para mantener la salida interna incluso si hay un `to` presente.

Recordatorios sobre el formato del destino:

* Los destinos de Slack/Discord/Mattermost (complemento) deben usar prefijos explícitos (por ejemplo, `channel:<id>`, `user:<id>`) para evitar ambigüedades.
* Los temas de Telegram deben usar el formato `:topic:` (ver más abajo).

<div id="telegram-delivery-targets-topics-forum-threads">
  #### Destinos de entrega en Telegram (temas / hilos de foro)
</div>

Telegram admite temas de foro mediante `message_thread_id`. Para entregas programadas con cron, puedes codificar
el tema/hilo en el campo `to`:

* `-1001234567890` (solo id del chat)
* `-1001234567890:topic:123` (preferido: marcador de tema explícito)
* `-1001234567890:123` (forma abreviada: sufijo numérico)

También se aceptan destinos con prefijo como `telegram:...` / `telegram:group:...`:

* `telegram:group:-1001234567890:topic:123`

<div id="storage-history">
  ## Almacenamiento e historial
</div>

* Almacén de trabajos: `~/.openclaw/cron/jobs.json` (JSON gestionado por el Gateway).
* Historial de ejecuciones: `~/.openclaw/cron/runs/<jobId>.jsonl` (JSONL, purgado automáticamente).
* Ruta de almacén personalizada: `cron.store` en la configuración.

<div id="configuration">
  ## Configuración
</div>

```json5
{
  cron: {
    enabled: true, // por defecto true
    store: "~/.openclaw/cron/jobs.json",
    maxConcurrentRuns: 1 // por defecto 1
  }
}
```

Deshabilita cron por completo:

* `cron.enabled: false` (config)
* `OPENCLAW_SKIP_CRON=1` (env)

<div id="cli-quickstart">
  ## Inicio rápido de la CLI
</div>

Recordatorio de ejecución única (UTC ISO, se elimina automáticamente tras completarse correctamente):

```bash
openclaw cron add \
  --name "Send reminder" \
  --at "2026-01-12T18:00:00Z" \
  --session main \
  --system-event "Reminder: submit expense report." \
  --wake now \
  --delete-after-run
```

Recordatorio único (sesión principal, activar de inmediato):

```bash
openclaw cron add \
  --name "Calendar check" \
  --at "20m" \
  --session main \
  --system-event "Next heartbeat: check calendar." \
  --wake now
```

Tarea recurrente aislada (envío a WhatsApp):

```bash
openclaw cron add \
  --name "Morning status" \
  --cron "0 7 * * *" \
  --tz "America/Los_Angeles" \
  --session isolated \
  --message "Summarize inbox + calendar for today." \
  --deliver \
  --channel whatsapp \
  --to "+15551234567"
```

Tarea aislada recurrente (enviar a un tema de Telegram):

```bash
openclaw cron add \
  --name "Nightly summary (topic)" \
  --cron "0 22 * * *" \
  --tz "America/Los_Angeles" \
  --session isolated \
  --message "Summarize today; send to the nightly topic." \
  --deliver \
  --channel telegram \
  --to "-1001234567890:topic:123"
```

Tarea aislada con anulación de modelo y razonamiento:

````bash
openclaw cron add \
  --name "Deep analysis" \
  --cron "0 6 * * 1" \
  --tz "America/Los_Angeles" \
  --session isolated \
  --message "Weekly deep analysis of project progress." \
  --model "opus" \
  --thinking high \
  --deliver \
  --channel whatsapp \
  --to "+15551234567"

Agent selection (multi-agent setups):
```bash
# Fijar un trabajo al agente "ops" (recurre al predeterminado si ese agente no existe)
openclaw cron add --name "Ops sweep" --cron "0 6 * * *" --session isolated --message "Check ops queue" --agent ops

# Switch or clear the agent on an existing job
openclaw cron edit <jobId> --agent ops
openclaw cron edit <jobId> --clear-agent
````

````

Ejecución manual (depuración):
```bash
openclaw cron run <jobId> --force
````

Editar una tarea programada existente (actualizar campos con PATCH):

```bash
openclaw cron edit <jobId> \
  --message "Updated prompt" \
  --model "opus" \
  --thinking low
```

Historial de ejecuciones:

```bash
openclaw cron runs --id <jobId> --limit 50
```

Evento de sistema inmediato sin crear una tarea programada:

```bash
openclaw system event --mode now --text "Next heartbeat: check battery."
```

<div id="gateway-api-surface">
  ## Superficie de la API del Gateway
</div>

* `cron.list`, `cron.status`, `cron.add`, `cron.update`, `cron.remove`
* `cron.run` (forzado o cuando corresponde), `cron.runs`
  Para eventos inmediatos del sistema sin un job, usa [`openclaw system event`](/es/cli/system).

<div id="troubleshooting">
  ## Solución de problemas
</div>

<div id="nothing-runs">
  ### «No se ejecuta nada»
</div>

* Comprueba que cron esté habilitado: `cron.enabled` y `OPENCLAW_SKIP_CRON`.
* Comprueba que el Gateway se esté ejecutando de forma continua (cron se ejecuta dentro del proceso del Gateway).
* Para tareas programadas de `cron`: confirma la zona horaria (`--tz`) y compárala con la zona horaria del host.

<div id="telegram-delivers-to-the-wrong-place">
  ### Telegram entrega en el lugar equivocado
</div>

* Para temas de foro, usa `-100…:topic:<id>` para que sea explícito y no haya ambigüedad.
* Si ves prefijos `telegram:...` en los registros o en los objetivos de “última ruta” almacenados, es normal;
  el cron los acepta y sigue interpretando correctamente los ID de tema.