---
title: Latido
summary: "Mensajes de sondeo del latido y reglas de notificación"
read_when:
  - Ajustar la cadencia o los mensajes del latido
  - Decidir entre latido y cron para tareas programadas
---

<div id="heartbeat-gateway">
  # Heartbeat (Gateway)
</div>

> **¿Heartbeat vs Cron?** Consulta [Cron vs Heartbeat](/es/automation/cron-vs-heartbeat) para saber cuándo usar cada uno.

Latido ejecuta **turnos periódicos del agente** en la sesión principal para que el modelo pueda sacar a la luz cualquier cosa que requiera atención sin llenarte de spam.

<div id="quick-start-beginner">
  ## Inicio rápido (principiante)
</div>

1. Deja los latidos activados (el valor predeterminado es `30m`, o `1h` para Anthropic OAuth/setup-token) o define tu propia cadencia.
2. Crea una pequeña checklist `HEARTBEAT.md` en el espacio de trabajo del agente (opcional pero recomendado).
3. Decide adónde deben ir los mensajes de latido (`target: "last"` es el valor predeterminado).
4. Opcional: habilita el envío del razonamiento del latido para mayor transparencia.
5. Opcional: restringe los latidos al horario de actividad (hora local).

Ejemplo de configuración:

```json5
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m",
        target: "last",
        // activeHours: { start: "08:00", end: "24:00" },
        // includeReasoning: true, // opcional: enviar mensaje `Reasoning:` separado también
      }
    }
  }
}
```

<div id="defaults">
  ## Valores predeterminados
</div>

* Intervalo: `30m` (o `1h` cuando Anthropic OAuth/setup-token es el modo de autenticación detectado). Configura `agents.defaults.heartbeat.every` o, por agente, `agents.list[].heartbeat.every`; usa `0m` para desactivarlo.
* Cuerpo del prompt (configurable mediante `agents.defaults.heartbeat.prompt`):
  `Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.`
* El prompt de latido se envía **literalmente** como mensaje del usuario. El prompt
  del sistema incluye una sección &quot;Heartbeat&quot; y la ejecución se marca internamente.
* Las horas activas (`heartbeat.activeHours`) se evalúan en la zona horaria configurada.
  Fuera de esa ventana, los latidos se omiten hasta el siguiente ciclo dentro de la ventana.

<div id="what-the-heartbeat-prompt-is-for">
  ## Para qué sirve el prompt de latido
</div>

El prompt predeterminado es intencionalmente amplio:

* **Tareas en segundo plano**: «Consider outstanding tasks» anima al agente a revisar
  seguimientos pendientes (bandeja de entrada, calendario, recordatorios, trabajo en cola) y resaltar cualquier asunto urgente.
* **Seguimiento humano**: «Checkup sometimes on your human during day time» impulsa un
  mensaje ocasional y ligero de «¿necesitas algo?», pero evita el spam nocturno
  usando tu zona horaria local configurada (consulta [/concepts/timezone](/es/concepts/timezone)).

Si quieres que un latido haga algo muy específico (por ejemplo, «check Gmail PubSub
stats» o «verify Gateway health»), define `agents.defaults.heartbeat.prompt` (o
`agents.list[].heartbeat.prompt`) con un cuerpo personalizado (enviado tal cual).

<div id="response-contract">
  ## Contrato de respuesta
</div>

* Si no hay nada que requiera atención, responde con **`HEARTBEAT_OK`**.
* Durante las ejecuciones del latido, OpenClaw trata `HEARTBEAT_OK` como un acuse de recibo cuando aparece
  al **inicio o al final** de la respuesta. El token se elimina y la respuesta se
  descarta si el contenido restante es **≤ `ackMaxChars`** (valor predeterminado: 300).
* Si `HEARTBEAT_OK` aparece en **medio** de una respuesta, no se trata
  de forma especial.
* Para alertas, **no** incluyas `HEARTBEAT_OK`; devuelve solo el texto de la alerta.

Fuera de los latidos, un `HEARTBEAT_OK` suelto al inicio/final de un mensaje se elimina
y se registra; un mensaje que sea solo `HEARTBEAT_OK` se descarta.

<div id="config">
  ## Configuración
</div>

```json5
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m",           // default: 30m (0m disables)
        model: "anthropic/claude-opus-4-5",
        includeReasoning: false, // predeterminado: false (entrega mensaje Reasoning: separado cuando esté disponible)
        target: "last",         // last | none | <channel id> (core or plugin, e.g. "bluebubbles")
        to: "+15551234567",     // optional channel-specific override
        prompt: "Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.",
        ackMaxChars: 300         // max chars allowed after HEARTBEAT_OK
      }
    }
  }
}
```

<div id="scope-and-precedence">
  ### Ámbito y precedencia
</div>

* `agents.defaults.heartbeat` define el comportamiento global del latido.
* `agents.list[].heartbeat` se aplica por encima; si algún agente tiene un bloque `heartbeat`, **solo esos agentes** ejecutan latidos.
* `channels.defaults.heartbeat` establece los valores predeterminados de visibilidad para todos los canales.
* `channels.<channel>.heartbeat` sobrescribe los valores predeterminados del canal.
* `channels.<channel>.accounts.<id>.heartbeat` (canales multicuenta) sobrescribe la configuración específica del canal.

<div id="per-agent-heartbeats">
  ### Latidos por agente
</div>

Si alguna entrada de `agents.list[]` incluye un bloque de `heartbeat`, **solo esos agentes**
ejecutan latidos. El bloque por agente se superpone a `agents.defaults.heartbeat`
(para que puedas definir valores predeterminados compartidos una sola vez y sobreescribirlos por agente).

Ejemplo: dos agentes, solo el segundo agente ejecuta latidos.

```json5
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m",
        target: "last"
      }
    },
    list: [
      { id: "main", default: true },
      {
        id: "ops",
        heartbeat: {
          every: "1h",
          target: "whatsapp",
          to: "+15551234567",
          prompt: "Lee HEARTBEAT.md si existe (contexto del espacio de trabajo). Síguelo estrictamente. No infiera ni repita tareas antiguas de conversaciones anteriores. Si nada requiere atención, responde HEARTBEAT_OK."
        }
      }
    ]
  }
}
```

<div id="field-notes">
  ### Notas de campo
</div>

* `every`: intervalo del latido (cadena de duración; unidad predeterminada = minutos).
* `model`: sobrescritura opcional del modelo para ejecuciones de latido (`provider/model`).
* `includeReasoning`: cuando está habilitado, también entrega el mensaje `Reasoning:` por separado cuando esté disponible (misma estructura que `/reasoning on`).
* `session`: clave de sesión opcional para ejecuciones de latido.
  * `main` (predeterminado): sesión principal del agente.
  * Clave de sesión explícita (cópiala desde `openclaw sessions --json` o desde la [CLI de sesiones](/es/cli/sessions)).
  * Formatos de clave de sesión: consulta [Sesiones](/es/concepts/session) y [Grupos](/es/concepts/groups).
* `target`:
  * `last` (predeterminado): entrega al último canal externo usado.
  * canal explícito: `whatsapp` / `telegram` / `discord` / `googlechat` / `slack` / `msteams` / `signal` / `imessage`.
  * `none`: ejecuta el latido pero **no lo entregues** externamente.
* `to`: sobrescritura opcional del destinatario (ID específico del canal, p. ej., E.164 para WhatsApp o un ID de chat de Telegram).
* `prompt`: sobrescribe el cuerpo del prompt predeterminado (no se fusiona).
* `ackMaxChars`: número máximo de caracteres permitidos después de `HEARTBEAT_OK` antes de la entrega.

<div id="delivery-behavior">
  ## Comportamiento de entrega
</div>

* Los latidos se ejecutan en la sesión principal del agente de forma predeterminada (`agent:<id>:<mainKey>`),
  o en `global` cuando `session.scope = "global"`. Establece `session` para anular esto y usar una
  sesión de canal específica (Discord/WhatsApp/etc.).
* `session` solo afecta el contexto de ejecución; la entrega se controla con `target` y `to`.
* Para entregar a un canal/destinatario específico, configura `target` + `to`. Con
  `target: "last"`, la entrega usa el último canal externo para esa sesión.
* Si la cola principal está ocupada, se omite el latido y se vuelve a intentar más tarde.
* Si `target` no se resuelve en ningún destino externo, la ejecución igualmente se realiza pero no
  se envía ningún mensaje saliente.
* Las respuestas generadas únicamente por el latido **no** mantienen viva la sesión; el último `updatedAt`
  se restaura para que el vencimiento por inactividad funcione con normalidad.

<div id="visibility-controls">
  ## Controles de visibilidad
</div>

De forma predeterminada, las confirmaciones `HEARTBEAT_OK` se suprimen mientras se
entrega el contenido de alerta. Puedes ajustar esto a nivel de canal o de cuenta:

```yaml
channels:
  defaults:
    heartbeat:
      showOk: false      # Ocultar HEARTBEAT_OK (predeterminado)
      showAlerts: true   # Mostrar mensajes de alerta (predeterminado)
      useIndicator: true # Emitir eventos de indicador (predeterminado)
  telegram:
    heartbeat:
      showOk: true       # Mostrar confirmaciones OK en Telegram
  whatsapp:
    accounts:
      work:
        heartbeat:
          showAlerts: false # Suprimir entrega de alertas para esta cuenta
```

Precedencia: por cuenta → por canal → valores predeterminados del canal → valores predeterminados internos.

<div id="what-each-flag-does">
  ### Qué hace cada flag
</div>

* `showOk`: envía un acuse de recibo `HEARTBEAT_OK` cuando el modelo devuelve una respuesta de solo OK.
* `showAlerts`: envía el contenido de la alerta cuando el modelo devuelve una respuesta que no es OK.
* `useIndicator`: emite eventos de indicador para las superficies de estado en la UI.

Si **las tres** son falsas, OpenClaw omite por completo el ciclo de latido (sin llamada al modelo).

<div id="per-channel-vs-per-account-examples">
  ### Ejemplos por canal frente a por cuenta
</div>

```yaml
channels:
  defaults:
    heartbeat:
      showOk: false
      showAlerts: true
      useIndicator: true
  slack:
    heartbeat:
      showOk: true # todas las cuentas de Slack
    accounts:
      ops:
        heartbeat:
          showAlerts: false # suprimir alertas únicamente para la cuenta ops
  telegram:
    heartbeat:
      showOk: true
```

<div id="common-patterns">
  ### Patrones comunes
</div>

| Objetivo | Configuración |
| --- | --- |
| Comportamiento por defecto (OK silenciosos, alertas activadas) | *(no se necesita configuración)* |
| Totalmente silencioso (sin mensajes, sin indicador) | `channels.defaults.heartbeat: { showOk: false, showAlerts: false, useIndicator: false }` |
| Solo indicador (sin mensajes) | `channels.defaults.heartbeat: { showOk: false, showAlerts: false, useIndicator: true }` |
| OK solo en un canal | `channels.telegram.heartbeat: { showOk: true }` |

<div id="heartbeatmd-optional">
  ## HEARTBEAT.md (opcional)
</div>

Si existe un archivo `HEARTBEAT.md` en el espacio de trabajo, la instrucción
predeterminada indica al agente que lo lea. Piensa en él como tu &quot;lista de
verificación de latido&quot;: pequeña, estable y segura de incluir cada 30 minutos.

Si `HEARTBEAT.md` existe pero está efectivamente vacío (solo líneas en blanco y
encabezados de markdown como `# Heading`), OpenClaw omite la ejecución del
latido para ahorrar llamadas a la API. Si el archivo no existe, el latido
sigue ejecutándose y el modelo decide qué hacer.

Mantenlo diminuto (lista de verificación o recordatorios breves) para evitar que
la instrucción se infle en exceso.

Ejemplo de `HEARTBEAT.md`:

```md
# Lista de verificación de latido

- Escaneo rápido: ¿hay algo urgente en las bandejas de entrada?
- Si es de día, realiza una verificación ligera si no hay nada más pendiente.
- Si una tarea está bloqueada, anota *qué falta* y pregúntale a Peter la próxima vez.
```

<div id="can-the-agent-update-heartbeatmd">
  ### ¿Puede el agente actualizar HEARTBEAT.md?
</div>

Sí, si se lo pides.

`HEARTBEAT.md` es solo un archivo normal en el espacio de trabajo del agente, así que puedes decirle al
agente (en un chat normal) algo como:

* «Actualiza `HEARTBEAT.md` para añadir una revisión diaria del calendario».
* «Reescribe `HEARTBEAT.md` para que sea más breve y se centre en los seguimientos de la bandeja de entrada».

Si quieres que esto ocurra de forma proactiva, también puedes incluir una línea explícita en
tu mensaje de latido como: «Si la lista de verificación se queda obsoleta, actualiza HEARTBEAT.md
con una mejor».

Nota de seguridad: no pongas secretos (claves api, números de teléfono, tokens privados) en
`HEARTBEAT.md`, ya que pasa a formar parte del contexto del prompt.

<div id="manual-wake-on-demand">
  ## Activación manual (a demanda)
</div>

Puedes encolar un evento del sistema y activar inmediatamente un latido con:

```bash
openclaw system event --text "Check for urgent follow-ups" --mode now
```

Si varios agentes tienen configurado `heartbeat`, una activación manual ejecuta de inmediato los latidos de cada uno de esos agentes.

Usa `--mode next-heartbeat` para esperar al siguiente ciclo programado.

<div id="reasoning-delivery-optional">
  ## Entrega de razonamiento (opcional)
</div>

De forma predeterminada, los latidos entregan solo la carga útil de la “respuesta” final.

Si quieres transparencia, habilita:

* `agents.defaults.heartbeat.includeReasoning: true`

Cuando está habilitado, los latidos también entregarán un mensaje separado con el prefijo
`Reasoning:` (mismo formato que `/reasoning on`). Esto puede ser útil cuando el agente
está gestionando múltiples sesiones/códices y quieres ver por qué decidió enviarte un ping,
pero también puede exponer más detalles internos de los que deseas. Es preferible mantenerlo
desactivado en chats de grupo.

<div id="cost-awareness">
  ## Conciencia de costos
</div>

Los latidos ejecutan turnos completos de los agentes. Los intervalos más cortos consumen más tokens. Mantén `HEARTBEAT.md` pequeño y considera un `model` más económico o `target: "none"` si solo necesitas actualizaciones del estado interno.