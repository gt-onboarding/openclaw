---
title: Cron vs latido
summary: "Guía para elegir entre latido y trabajos cron para la automatización"
read_when:
  - Decidir cómo programar tareas recurrentes
  - Configurar monitorización o notificaciones en segundo plano
  - Optimizar el uso de tokens para comprobaciones periódicas
---

<div id="cron-vs-heartbeat-when-to-use-each">
  # Cron vs latido: cuándo usar cada uno
</div>

Tanto los latidos como los trabajos cron te permiten ejecutar tareas de forma programada. Esta guía te ayuda a elegir el mecanismo adecuado para tu caso de uso.

<div id="quick-decision-guide">
  ## Guía rápida de decisión
</div>

| Caso de uso | Recomendado | Por qué |
|-------------|-------------|--------|
| Revisar la bandeja de entrada cada 30 min | Heartbeat | Se agrupa con otras comprobaciones y tiene en cuenta el contexto |
| Enviar informe diario a las 9 en punto | Cron (aislado) | Se necesita una hora exacta |
| Supervisar el calendario en busca de eventos próximos | Heartbeat | Encaja de forma natural con el seguimiento periódico |
| Ejecutar un análisis profundo semanal | Cron (aislado) | Tarea independiente, puede usar un modelo diferente |
| Recuérdame en 20 minutos | Cron (principal, `--at`) | Ejecución única con temporización precisa |
| Comprobación en segundo plano del estado del proyecto | Heartbeat | Se apoya en el ciclo existente |

<div id="heartbeat-periodic-awareness">
  ## Latido: Conciencia periódica
</div>

Los latidos se ejecutan en la **sesión principal** a un intervalo regular (predeterminado: 30 min). Están diseñados para que el agente compruebe el estado de las cosas y haga aflorar cualquier aspecto importante.

<div id="when-to-use-heartbeat">
  ### Cuándo usar el latido
</div>

* **Múltiples comprobaciones periódicas**: En lugar de 5 tareas cron independientes que comprueben la bandeja de entrada, el calendario, el tiempo, las notificaciones y el estado de los proyectos, un solo latido puede agrupar todo esto.
* **Decisiones sensibles al contexto**: El agente tiene todo el contexto de la sesión principal, por lo que puede tomar decisiones inteligentes sobre qué es urgente y qué puede esperar.
* **Continuidad conversacional**: Las ejecuciones del latido comparten la misma sesión, así que el agente recuerda conversaciones recientes y puede hacer seguimiento de forma natural.
* **Supervisión con poca sobrecarga**: Un solo latido reemplaza muchas pequeñas tareas de sondeo.

<div id="heartbeat-advantages">
  ### Ventajas del latido
</div>

* **Agrupa múltiples comprobaciones**: Un turno de agente puede revisar la bandeja de entrada, el calendario y las notificaciones a la vez.
* **Reduce las llamadas a la API**: Un solo latido es más barato que 5 tareas cron aisladas.
* **Con conocimiento del contexto**: El agente sabe en qué has estado trabajando y puede priorizar en consecuencia.
* **Supresión inteligente**: Si no hay nada que requiera atención, el agente responde `HEARTBEAT_OK` y no se entrega ningún mensaje.
* **Temporización natural**: Se desvía ligeramente en función de la carga de la cola, lo cual es adecuado para la mayoría de los casos de monitoreo.

<div id="heartbeat-example-heartbeatmd-checklist">
  ### Ejemplo de latido: lista de comprobación de HEARTBEAT.md
</div>

```md
# Lista de verificación de latido

- Revisar correo electrónico para mensajes urgentes
- Revisar calendario para eventos en las próximas 2 horas
- Si finalizó una tarea en segundo plano, resumir resultados
- Si está inactivo por más de 8 horas, enviar una breve verificación
```

El agente lee esta configuración en cada latido y procesa todos los elementos en una sola interacción.

<div id="configuring-heartbeat">
  ### Configuración de latido
</div>

```json5
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m",        // interval
        target: "last",      // where to deliver alerts
        activeHours: { start: "08:00", end: "22:00" }  // opcional
      }
    }
  }
}
```

Consulta [Heartbeat](/es/gateway/heartbeat) para obtener la configuración completa.

<div id="cron-precise-scheduling">
  ## Cron: Programación precisa
</div>

Los trabajos de cron se ejecutan en **momentos exactos** y pueden ejecutarse en sesiones aisladas sin afectar al contexto principal.

<div id="when-to-use-cron">
  ### Cuándo usar cron
</div>

* **Se requiere una hora exacta**: &quot;Envía esto a las 9:00 AM todos los lunes&quot; (no &quot;en algún momento alrededor de las 9&quot;).
* **Tareas independientes**: Tareas que no necesitan contexto conversacional.
* **Modelo/razonamiento diferente**: Análisis intensivo que justifica un modelo más potente.
* **Recordatorios puntuales**: &quot;Recuérdame en 20 minutos&quot; con `--at`.
* **Tareas ruidosas/frecuentes**: Tareas que saturarían el historial de la sesión principal.
* **Disparadores externos**: Tareas que deben ejecutarse independientemente de si el agente está activo.

<div id="cron-advantages">
  ### Ventajas de Cron
</div>

* **Programación exacta**: expresiones cron de 5 campos con soporte de zona horaria.
* **Aislamiento de sesión**: se ejecuta en `cron:<jobId>` sin ensuciar el historial principal.
* **Anulación de modelo**: usa un modelo más barato o más potente por tarea.
* **Control de entrega**: puede entregar directamente a un canal; aun así, publica un resumen en la sesión principal de forma predeterminada (configurable).
* **Sin necesidad de contexto de agente**: se ejecuta incluso si la sesión principal está inactiva o compactada.
* **Soporte de ejecución única**: `--at` para marcas de tiempo futuras precisas.

<div id="cron-example-daily-morning-briefing">
  ### Ejemplo de Cron: resumen matutino diario
</div>

```bash
openclaw cron add \
  --name "Morning briefing" \
  --cron "0 7 * * *" \
  --tz "America/New_York" \
  --session isolated \
  --message "Generate today's briefing: weather, calendar, top emails, news summary." \
  --model opus \
  --deliver \
  --channel whatsapp \
  --to "+15551234567"
```

Esto se ejecuta exactamente a las 7:00 a. m., hora de Nueva York, usa Opus para mayor calidad y se entrega directamente por WhatsApp.

<div id="cron-example-one-shot-reminder">
  ### Ejemplo de cron: recordatorio de una sola ejecución
</div>

```bash
openclaw cron add \
  --name "Meeting reminder" \
  --at "20m" \
  --session main \
  --system-event "Reminder: standup meeting starts in 10 minutes." \
  --wake now \
  --delete-after-run
```

Consulta la página [trabajos de Cron](/es/automation/cron-jobs) para ver la referencia completa de la CLI.

<div id="decision-flowchart">
  ## Diagrama de flujo de decisiones
</div>

```
Does the task need to run at an EXACT time?
  YES -> Use cron
  NO  -> Continue...

Does the task need isolation from main session?
  YES -> Use cron (isolated)
  NO  -> Continue...

Can this task be batched with other periodic checks?
  YES -> Use heartbeat (add to HEARTBEAT.md)
  NO  -> Use cron

Is this a one-shot reminder?
  YES -> Use cron with --at
  NO  -> Continue...

Does it need a different model or thinking level?
  YES -> Use cron (isolated) with --model/--thinking
  NO  -> Use heartbeat
```

<div id="combining-both">
  ## Combinar ambos
</div>

La configuración más eficiente utiliza **ambos**:

1. **Latido** se encarga de la supervisión rutinaria (bandeja de entrada, calendario, notificaciones) en un único ciclo por lotes cada 30 minutos.
2. **Cron** gestiona programaciones precisas (informes diarios, revisiones semanales) y recordatorios puntuales.

<div id="example-efficient-automation-setup">
  ### Ejemplo: configuración eficiente de automatización
</div>

**HEARTBEAT.md** (revisado cada 30 min):

```md
# Heartbeat checklist
- Scan inbox for urgent emails
- Check calendar for events in next 2h
- Review any pending tasks
- Light check-in if quiet for 8+ hours
```

**Trabajos de cron** (programación precisa):

```bash
# Daily morning briefing at 7am
openclaw cron add --name "Morning brief" --cron "0 7 * * *" --session isolated --message "..." --deliver

# Revisión semanal de proyecto los lunes a las 9am
openclaw cron add --name "Weekly review" --cron "0 9 * * 1" --session isolated --message "..." --model opus

# One-shot reminder
openclaw cron add --name "Call back" --at "2h" --session main --system-event "Call back the client" --wake now
```

<div id="lobster-deterministic-workflows-with-approvals">
  ## Lobster: Flujos de trabajo deterministas con aprobaciones
</div>

Lobster es el entorno de ejecución de flujos de trabajo para **pipelines de herramientas de varios pasos** que necesitan una ejecución determinista y aprobaciones explícitas.
Úsalo cuando la tarea sea más que un único turno de un agente y quieras un flujo de trabajo que puedas reanudar con puntos de control humanos.

<div id="when-lobster-fits">
  ### Cuándo usar Lobster
</div>

* **Automatización en varios pasos**: Necesitas un pipeline fijo de llamadas a herramientas, no un prompt aislado.
* **Puntos de aprobación**: Las acciones con efectos secundarios deben pausarse hasta que las apruebes y luego continuar.
* **Ejecuciones reanudables**: Continúa un flujo de trabajo en pausa sin volver a ejecutar los pasos anteriores.

<div id="how-it-pairs-with-heartbeat-and-cron">
  ### Cómo se combina con latido y cron
</div>

* **Latido/cron** deciden *cuándo* ocurre una ejecución.
* **Lobster** define *qué pasos* ocurren una vez que la ejecución comienza.

Para flujos de trabajo programados, usa cron o el latido para activar un turno de agente que invoque a Lobster.
Para flujos de trabajo ad hoc, invoca a Lobster directamente.

<div id="operational-notes-from-the-code">
  ### Notas operativas (en el código)
</div>

* Lobster se ejecuta como un **subproceso local** (`lobster` CLI) en modo de herramienta y devuelve una **envoltura JSON**.
* Si la herramienta devuelve `needs_approval`, reanudas con un `resumeToken` y el indicador `approve`.
* La herramienta es un **complemento opcional**; habilítala adicionalmente mediante `tools.alsoAllow: ["lobster"]` (recomendado).
* Si pasas `lobsterPath`, debe ser una **ruta absoluta**.

Consulta [Lobster](/es/tools/lobster) para obtener información completa sobre su uso y ejemplos.

<div id="main-session-vs-isolated-session">
  ## Sesión principal vs sesión aislada
</div>

Tanto el latido como cron pueden interactuar con la sesión principal, pero de forma diferente:

| | Latido | Cron (principal) | Cron (aislado) |
|---|---|---|---|
| Sesión | Principal | Principal (vía evento del sistema) | `cron:<jobId>` |
| Historial | Compartido | Compartido | Nuevo en cada ejecución |
| Contexto | Completo | Completo | Ninguno (empieza limpio) |
| Modelo | Modelo de la sesión principal | Modelo de la sesión principal | Se puede anular |
| Salida | Se entrega si no es `HEARTBEAT_OK` | Prompt de latido + evento | Resumen publicado en la sesión principal |

<div id="when-to-use-main-session-cron">
  ### Cuándo usar cron con la sesión principal
</div>

Usa `--session main` con `--system-event` cuando quieras:

* Que el recordatorio/evento aparezca en el contexto de la sesión principal
* Que el agente lo gestione durante el siguiente latido con todo el contexto
* No realizar una ejecución aislada independiente

```bash
openclaw cron add \
  --name "Check project" \
  --every "4h" \
  --session main \
  --system-event "Time for a project health check" \
  --wake now
```

<div id="when-to-use-isolated-cron">
  ### Cuándo usar cron aislado
</div>

Usa `--session isolated` cuando quieras:

* Un estado limpio sin contexto previo
* Un modelo diferente o configuraciones de razonamiento distintas
* Salida entregada directamente a un canal (el resumen sigue publicándose en el canal principal de forma predeterminada)
* Historial que no llene de ruido la sesión principal

```bash
openclaw cron add \
  --name "Deep analysis" \
  --cron "0 6 * * 0" \
  --session isolated \
  --message "Weekly codebase analysis..." \
  --model opus \
  --thinking high \
  --deliver
```

<div id="cost-considerations">
  ## Consideraciones de costos
</div>

| Mecanismo | Perfil de costos |
|-----------|------------------|
| Latido | Un turno cada N minutos; escala con el tamaño de HEARTBEAT.md |
| Cron (principal) | Agrega un evento al siguiente latido (sin turno aislado) |
| Cron (aislado) | Turno completo del agente por tarea; puede usar un modelo más barato |

**Consejos**:

* Mantén `HEARTBEAT.md` pequeño para minimizar la sobrecarga de tokens.
* Agrupa comprobaciones similares en el latido en lugar de múltiples trabajos de cron.
* Usa `target: "none"` en el latido si solo quieres procesamiento interno.
* Usa un cron aislado con un modelo más barato para tareas rutinarias.

<div id="related">
  ## Relacionado
</div>

* [Latido](/es/gateway/heartbeat) - configuración completa del latido
* [Trabajos cron](/es/automation/cron-jobs) - referencia completa de la CLI y la API de cron
* [System](/es/cli/system) - eventos del sistema y controles del latido