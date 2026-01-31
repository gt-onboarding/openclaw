---
title: Subagentes
summary: "Subagentes: iniciando ejecuciones de agentes aisladas que informan de los resultados al chat que las solicitó"
read_when:
  - Quieres ejecutar trabajo en segundo plano o en paralelo mediante el agente
  - Estás cambiando sessions_spawn o la política de herramientas de subagentes
---

<div id="sub-agents">
  # Sub-agentes
</div>

Los sub-agentes son ejecuciones de agente en segundo plano generadas a partir de una ejecución de agente existente. Se ejecutan en su propia sesión (`agent:<agentId>:subagent:<uuid>`) y, cuando terminan, **anuncian** su resultado de vuelta al canal de chat que realizó la solicitud.

<div id="slash-command">
  ## Comando slash
</div>

Usa `/subagents` para inspeccionar o controlar ejecuciones de subagentes para la **sesión actual**:

- `/subagents list`
- `/subagents stop <id|#|all>`
- `/subagents log <id|#> [limit] [tools]`
- `/subagents info <id|#>`
- `/subagents send <id|#> <message>`

`/subagents info` muestra los metadatos de la ejecución (estado, marcas de tiempo, id de sesión, ruta de la transcripción, limpieza).

Objetivos principales:

- Paralelizar el trabajo de “investigación / tarea larga / herramienta lenta” sin bloquear la ejecución principal.
- Mantener los subagentes aislados por defecto (separación de sesión + sandbox opcional).
- Mantener la superficie de herramientas difícil de usar incorrectamente: los subagentes **no** obtienen las herramientas de la sesión por defecto.
- Evitar el fan-out anidado: los subagentes no pueden crear otros subagentes.

Nota sobre costos: cada subagente tiene su **propio** contexto y uso de tokens. Para tareas pesadas o repetitivas, configura un modelo más barato para los subagentes y mantén tu agente principal en un modelo de mayor calidad.
Puedes configurar esto mediante `agents.defaults.subagents.model` o con configuraciones específicas por agente.

<div id="tool">
  ## Tool
</div>

Usa `sessions_spawn`:

- Inicia una ejecución de subagente (`deliver: false`, lane global: `subagent`)
- Luego ejecuta un paso de anuncio y publica la respuesta de anuncio en el canal de chat del solicitante
- Modelo predeterminado: hereda del proceso que lo invoca a menos que configures `agents.defaults.subagents.model` (o por agente `agents.list[].subagents.model`); un `sessions_spawn.model` explícito sigue teniendo prioridad.

Parámetros de la herramienta:

- `task` (obligatorio)
- `label?` (opcional)
- `agentId?` (opcional; crea el subagente bajo otro id de agente si está permitido)
- `model?` (opcional; anula el modelo del subagente; los valores no válidos se omiten y el subagente se ejecuta con el modelo predeterminado, dejando una advertencia en el resultado de la herramienta)
- `thinking?` (opcional; anula el nivel de razonamiento para la ejecución del subagente)
- `runTimeoutSeconds?` (predeterminado `0`; cuando se establece, la ejecución del subagente se aborta después de N segundos)
- `cleanup?` (`delete|keep`, predeterminado `keep`)

Lista de permitidos:

- `agents.list[].subagents.allowAgents`: lista de ids de agente a los que se puede dirigir mediante `agentId` (`["*"]` para permitir cualquiera). Predeterminado: solo el agente solicitante.

Descubrimiento:

- Usa `agents_list` para ver qué ids de agente están actualmente permitidos para `sessions_spawn`.

Archivado automático:

- Las sesiones de subagente se archivan automáticamente después de `agents.defaults.subagents.archiveAfterMinutes` (predeterminado: 60).
- El archivado usa `sessions.delete` y renombra la transcripción a `*.deleted.<timestamp>` (misma carpeta).
- `cleanup: "delete"` archiva inmediatamente después del anuncio (aun conserva la transcripción mediante el renombrado).
- El archivado automático es de mejor esfuerzo; los temporizadores pendientes se pierden si el Gateway se reinicia.
- `runTimeoutSeconds` **no** archiva automáticamente; solo detiene la ejecución. La sesión permanece hasta el archivado automático.

<div id="authentication">
  ## Autenticación
</div>

La autenticación de subagentes se resuelve por **ID de agente**, no por tipo de sesión:

- La clave de sesión del subagente es `agent:<agentId>:subagent:<uuid>`.
- El almacén de autenticación se carga desde el `agentDir` de ese agente.
- Los perfiles de autenticación del agente principal se combinan como **respaldo**; los perfiles del agente tienen prioridad sobre los del agente principal en caso de conflicto.

Nota: la combinación es aditiva, por lo que los perfiles del agente principal siempre están disponibles como respaldo. La autenticación completamente aislada por agente aún no está soportada.

<div id="announce">
  ## Anuncio
</div>

Los subagentes informan mediante un paso de anuncio:

- El paso de anuncio se ejecuta dentro de la sesión del subagente (no en la sesión del solicitante).
- Si el subagente responde exactamente `ANNOUNCE_SKIP`, no se publica nada.
- De lo contrario, la respuesta de anuncio se publica en el canal de chat del solicitante mediante una llamada `agent` de seguimiento (`deliver=true`).
- Las respuestas de anuncio preservan el enrutamiento de hilo/tema cuando está disponible (hilos de Slack, temas de Telegram, hilos de Matrix).
- Los mensajes de anuncio se normalizan a una plantilla estable:
  - `Status:` derivado del resultado de la ejecución (`success`, `error`, `timeout` o `unknown`).
  - `Result:` el contenido resumen del paso de anuncio (o `(not available)` si falta).
  - `Notes:` detalles de error y otro contexto útil.
- `Status` no se infiere de la salida del modelo; proviene de señales de resultado en tiempo de ejecución.

Las cargas útiles de anuncio incluyen una línea de estadísticas al final (incluso cuando están envueltas):

- Tiempo de ejecución (por ejemplo, `runtime 5m12s`)
- Uso de tokens (entrada/salida/total)
- Coste estimado cuando se ha configurado el precio del modelo (`models.providers.*.models[].cost`)
- `sessionKey`, `sessionId` y ruta de la transcripción (para que el agente principal pueda obtener el historial mediante `sessions_history` o inspeccionar el archivo en disco)

<div id="tool-policy-sub-agent-tools">
  ## Política de herramientas (herramientas de subagentes)
</div>

De forma predeterminada, los subagentes tienen acceso a **todas las herramientas excepto las herramientas de sesión**:

* `sessions_list`
* `sessions_history`
* `sessions_send`
* `sessions_spawn`

Anula esto mediante la configuración:

```json5
{
  agents: {
    defaults: {
      subagents: {
        maxConcurrent: 1
      }
    }
  },
  tools: {
    subagents: {
      tools: {
        // deny tiene prioridad
        deny: ["gateway", "cron"],
        // si se establece allow, se convierte en solo-permitir (deny sigue teniendo prioridad)
        // allow: ["read", "exec", "process"]
      }
    }
  }
}
```


<div id="concurrency">
  ## Concurrencia
</div>

Los subagentes usan una cola dedicada dentro del proceso:

- Nombre de la cola: `subagent`
- Concurrencia: `agents.defaults.subagents.maxConcurrent` (valor por defecto `8`)

<div id="stopping">
  ## Detener
</div>

- Enviar `/stop` en el chat del solicitante cancela la sesión del solicitante y detiene cualquier ejecución de subagente activa iniciada a partir de ella.

<div id="limitations">
  ## Limitaciones
</div>

- El anuncio del sub-agente es de tipo **best-effort**. Si el Gateway se reinicia, se pierde el trabajo pendiente de «announce back».
- Los sub-agentes siguen compartiendo los mismos recursos del proceso del Gateway; considera `maxConcurrent` una válvula de seguridad.
- `sessions_spawn` es siempre no bloqueante: devuelve `{ status: "accepted", runId, childSessionKey }` inmediatamente.
- El contexto del sub-agente solo inyecta `AGENTS.md` + `TOOLS.md` (sin `SOUL.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md` ni `BOOTSTRAP.md`).