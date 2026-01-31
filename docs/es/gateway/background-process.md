---
title: Proceso en segundo plano
summary: "Ejecución de Background Exec y gestión de procesos"
read_when:
  - Agregar o modificar el comportamiento de Background Exec
  - Depurar tareas exec de ejecución prolongada
---

<div id="background-exec-process-tool">
  # Exec en segundo plano y herramienta de procesos
</div>

OpenClaw ejecuta comandos de shell mediante la herramienta `exec` y mantiene en memoria las tareas de ejecución prolongada. La herramienta `process` gestiona esas sesiones en segundo plano.

<div id="exec-tool">
  ## herramienta exec
</div>

Parámetros clave:

- `command` (obligatorio)
- `yieldMs` (valor predeterminado 10000): pasa a segundo plano automáticamente tras este retraso
- `background` (bool): pasa a segundo plano inmediatamente
- `timeout` (segundos, valor predeterminado 1800): finaliza el proceso después de este tiempo de espera
- `elevated` (bool): se ejecuta en el host si el modo elevado está habilitado/permitido
- ¿Necesitas un TTY real? Establece `pty: true`.
- `workdir`, `env`

Comportamiento:

- Las ejecuciones en primer plano devuelven la salida directamente.
- Cuando pasa a segundo plano (explícitamente o por timeout), la herramienta devuelve `status: "running"` + `sessionId` y un pequeño tramo final de la salida.
- La salida se mantiene en memoria hasta que la sesión se consulta o se limpia.
- Si la herramienta `process` no está permitida, `exec` se ejecuta de forma síncrona e ignora `yieldMs`/`background`.

<div id="child-process-bridging">
  ## Puente para procesos hijo
</div>

Al lanzar procesos hijo de larga duración fuera de las herramientas de exec/process (por ejemplo, reinicios de la CLI o helpers del Gateway), adjunta el helper de puente de procesos hijo para que las señales de terminación se reenvíen y los listeners se desregistren al salir o en caso de error. Esto evita procesos huérfanos en systemd y mantiene un comportamiento de apagado coherente en todas las plataformas.

Anulaciones de entorno:

- `PI_BASH_YIELD_MS`: intervalo de cesión predeterminado (ms)
- `PI_BASH_MAX_OUTPUT_CHARS`: límite de salida en memoria (caracteres)
- `OPENCLAW_BASH_PENDING_MAX_OUTPUT_CHARS`: límite pendiente de stdout/stderr por flujo (caracteres)
- `PI_BASH_JOB_TTL_MS`: TTL para sesiones finalizadas (ms, limitado a 1m–3h)

Configuración (preferida):

- `tools.exec.backgroundMs` (predeterminado 10000)
- `tools.exec.timeoutSec` (predeterminado 1800)
- `tools.exec.cleanupMs` (predeterminado 1800000)
 - `tools.exec.notifyOnExit` (predeterminado true): encola un evento de sistema y solicita un latido cuando termina un exec en segundo plano.

<div id="process-tool">
  ## herramienta process
</div>

Acciones:

- `list`: sesiones en ejecución y finalizadas
- `poll`: extrae nueva salida de una sesión (también informa el código de salida)
- `log`: lee la salida agregada (admite `offset` + `limit`)
- `write`: envía stdin (`data`, `eof` opcional)
- `kill`: termina una sesión en segundo plano
- `clear`: elimina una sesión finalizada de la memoria
- `remove`: mata si está en ejecución; de lo contrario, limpia si ya terminó

Notas:

- Solo las sesiones en segundo plano se listan y se conservan en memoria.
- Las sesiones se pierden al reiniciar el proceso (no hay persistencia en disco).
- Los logs de la sesión solo se guardan en el historial del chat si ejecutas `process poll/log` y se registra el resultado de la herramienta.
- `process` tiene ámbito por agente; solo ve sesiones iniciadas por ese agente.
- `process list` incluye un `name` derivado (verbo del comando + destino) para inspecciones rápidas.
- `process log` usa `offset`/`limit` basados en líneas (omite `offset` para obtener las últimas N líneas).

<div id="examples">
  ## Ejemplos
</div>

Ejecuta una tarea larga y comprueba su estado más tarde:

```json
{"tool": "exec", "command": "sleep 5 && echo done", "yieldMs": 1000}
```

```json
{"tool": "process", "action": "poll", "sessionId": "<id>"}
```

Iniciar inmediatamente en segundo plano:

```json
{"tool": "exec", "command": "npm run build", "background": true}
```

Enviar la entrada estándar (stdin):

```json
{"tool": "process", "action": "write", "sessionId": "<id>", "data": "y\n"}
```
