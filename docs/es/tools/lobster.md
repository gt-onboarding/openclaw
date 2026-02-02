---
title: Lobster
summary: "Entorno de ejecución de flujos de trabajo tipados para OpenClaw con puntos de aprobación reanudables."
description: Entorno de ejecución de flujos de trabajo tipados para OpenClaw — pipelines componibles con puntos de aprobación.
read_when:
  - Quieres flujos de trabajo deterministas de varios pasos con aprobaciones explícitas
  - Necesitas reanudar un flujo de trabajo sin volver a ejecutar los pasos anteriores
---

<div id="lobster">
  # Lobster
</div>

Lobster es un shell para flujos de trabajo que permite a OpenClaw ejecutar secuencias de herramientas de varios pasos como una única operación determinista, con puntos de control de aprobación explícitos.

<div id="hook">
  ## Gancho
</div>

Tu asistente puede crear las herramientas que lo gestionan. Pide un flujo de trabajo y, 30 minutos después, tendrás una CLI y pipelines que se ejecutan como una única llamada. Lobster es la pieza que faltaba: pipelines deterministas, aprobaciones explícitas y estado reanudable.

<div id="why">
  ## Por qué
</div>

Hoy en día, los flujos de trabajo complejos requieren muchas llamadas sucesivas a herramientas. Cada llamada consume tokens, y el LLM tiene que orquestar cada paso. Lobster mueve esa orquestación a un runtime tipado:

* **Una llamada en lugar de muchas**: OpenClaw ejecuta una única llamada a la herramienta Lobster y obtiene un resultado estructurado.
* **Aprobaciones integradas**: Los efectos secundarios (enviar correo, publicar comentario) detienen el flujo de trabajo hasta que se aprueben explícitamente.
* **Reanudable**: Los flujos de trabajo detenidos devuelven un token; puedes aprobar y reanudar sin volver a ejecutar todo.

<div id="why-a-dsl-instead-of-plain-programs">
  ## ¿Por qué un DSL en lugar de programas convencionales?
</div>

Lobster es deliberadamente pequeño. El objetivo no es &quot;un lenguaje nuevo&quot;, sino una especificación de pipeline predecible y amigable para la IA, con aprobaciones de primera clase y tokens de reanudación.

* **La aprobación/reanudación viene integrada**: Un programa normal puede solicitar la intervención de una persona, pero no puede *pausarse y reanudarse* con un token duradero sin que tú mismo tengas que inventar ese runtime.
* **Determinismo + auditabilidad**: Los pipelines son datos, así que es fácil registrarlos, compararlos, reejecutarlos y revisarlos.
* **Superficie limitada para la IA**: Una gramática mínima + encadenamiento con JSON reduce las rutas de código “creativas” y hace que la validación sea realista.
* **Política de seguridad incorporada**: Los timeouts, límites de salida, comprobaciones de sandbox y listas de permitidos los aplica el runtime, no cada script.
* **Sigue siendo programable**: Cada paso puede invocar cualquier CLI o script. Si quieres JS/TS, genera archivos `.lobster` desde código.

<div id="how-it-works">
  ## Cómo funciona
</div>

OpenClaw ejecuta la CLI local `lobster` en **modo de herramienta** y analiza un envoltorio JSON desde stdout.
Si el pipeline se detiene a la espera de aprobación, la herramienta devuelve un `resumeToken` para que puedas continuar más tarde.

<div id="pattern-small-cli-json-pipes-approvals">
  ## Patrón: CLI pequeña + canalizaciones JSON + aprobaciones
</div>

Crea comandos pequeños que hablen JSON y luego encadénalos en una sola llamada a Lobster. (Nombres de comando de ejemplo a continuación; sustitúyelos por los tuyos propios).

```bash
inbox list --json
inbox categorize --json
inbox apply --json
```

```json
{
  "action": "run",
  "pipeline": "exec --json --shell 'inbox list --json' | exec --stdin json --shell 'inbox categorize --json' | exec --stdin json --shell 'inbox apply --json' | approve --preview-from-stdin --limit 5 --prompt 'Apply changes?'",
  "timeoutMs": 30000
}
```

Si el pipeline solicita aprobación, continúa con el token:

```json
{
  "action": "resume",
  "token": "<resumeToken>",
  "approve": true
}
```

La IA dispara el flujo de trabajo; Lobster ejecuta los pasos. Los puntos de aprobación mantienen los efectos secundarios explícitos y auditables.

Ejemplo: mapear elementos de entrada en llamadas a herramientas:

```bash
gog.gmail.search --query 'newer_than:1d' \
  | openclaw.invoke --tool message --action send --each --item-key message --args-json '{"provider":"telegram","to":"..."}'
```

<div id="json-only-llm-steps-llm-task">
  ## Pasos de LLM solo JSON (llm-task)
</div>

Para flujos de trabajo que necesitan un **paso de LLM estructurado**, habilita la herramienta de complemento opcional
`llm-task` y utilízala desde Lobster. Esto mantiene el flujo de trabajo
determinista y, al mismo tiempo, te permite clasificar/resumir/redactar con un modelo.

Habilita la herramienta:

```json
{
  "plugins": {
    "entries": {
      "llm-task": { "enabled": true }
    }
  },
  "agents": {
    "list": [
      {
        "id": "main",
        "tools": { "allow": ["llm-task"] }
      }
    ]
  }
}
```

Úsalo en un pipeline:

```lobster
openclaw.invoke --tool llm-task --action json --args-json '{
  "prompt": "Dado el correo electrónico de entrada, devuelve intención y borrador.",
  "input": { "subject": "Hello", "body": "Can you help?" },
  "schema": {
    "type": "object",
    "properties": {
      "intent": { "type": "string" },
      "draft": { "type": "string" }
    },
    "required": ["intent", "draft"],
    "additionalProperties": false
  }
}'
```

Consulta [LLM Task](/es/tools/llm-task) para obtener más detalles y opciones de configuración.

<div id="workflow-files-lobster">
  ## Archivos de flujo de trabajo (.lobster)
</div>

Lobster puede ejecutar archivos de flujo de trabajo YAML/JSON con los campos `name`, `args`, `steps`, `env`, `condition` y `approval`. En las llamadas de herramientas de OpenClaw, configura `pipeline` con la ruta del archivo.

```yaml
name: inbox-triage
args:
  tag:
    default: "family"
steps:
  - id: collect
    command: inbox list --json
  - id: categorize
    command: inbox categorize --json
    stdin: $collect.stdout
  - id: approve
    command: inbox apply --approve
    stdin: $categorize.stdout
    approval: required
  - id: execute
    command: inbox apply --execute
    stdin: $categorize.stdout
    condition: $approve.approved
```

Notas:

* `stdin: $step.stdout` y `stdin: $step.json` transmiten la salida de un paso anterior.
* `condition` (o `when`) puede condicionar la ejecución de pasos en función de `$step.approved`.

<div id="install-lobster">
  ## Instalar Lobster
</div>

Instala la CLI de Lobster en el **mismo host** que ejecuta el OpenClaw Gateway (consulta el [repositorio de Lobster](https://github.com/openclaw/lobster)) y asegúrate de que `lobster` esté en el `PATH`.
Si quieres usar una ubicación personalizada del binario, pasa un `lobsterPath` **absoluto** en la llamada a la herramienta.

<div id="enable-the-tool">
  ## Habilitar la herramienta
</div>

Lobster es una herramienta de complemento **opcional** (no está habilitada de forma predeterminada).

Recomendado (aditivo y seguro):

```json
{
  "tools": {
    "alsoAllow": ["lobster"]
  }
}
```

O por agente:

```json
{
  "agents": {
    "list": [
      {
        "id": "main",
        "tools": {
          "alsoAllow": ["lobster"]
        }
      }
    ]
  }
}
```

Evita usar `tools.allow: ["lobster"]` a menos que tengas la intención de ejecutar en un modo de lista de permitidos restrictiva.

Nota: las listas de permitidos son opcionales y se activan explícitamente para complementos opcionales. Si tu lista de permitidos solo menciona
herramientas de complementos (como `lobster`), OpenClaw mantiene las herramientas principales habilitadas. Para restringir las herramientas principales,
incluye también en la lista de permitidos las herramientas principales o los grupos que quieras.

<div id="example-email-triage">
  ## Ejemplo: Clasificación de correos electrónicos
</div>

Sin Lobster:

```
User: "Check my email and draft replies"
→ openclaw calls gmail.list
→ LLM summarizes
→ User: "draft replies to #2 and #5"
→ LLM drafts
→ User: "send #2"
→ openclaw calls gmail.send
(repeat daily, no memory of what was triaged)
```

Con Lobster:

```json
{
  "action": "run",
  "pipeline": "email.triage --limit 20",
  "timeoutMs": 30000
}
```

Devuelve un contenedor JSON (truncado):

```json
{
  "ok": true,
  "status": "needs_approval",
  "output": [{ "summary": "5 need replies, 2 need action" }],
  "requiresApproval": {
    "type": "approval_request",
    "prompt": "¿Enviar 2 borradores de respuesta?",
    "items": [],
    "resumeToken": "..."
  }
}
```

Si el usuario aprueba → reanudar:

```json
{
  "action": "resume",
  "token": "<resumeToken>",
  "approve": true
}
```

Un único flujo de trabajo. Determinista. Seguro.

<div id="tool-parameters">
  ## Parámetros de herramienta
</div>

<div id="run">
  ### `run`
</div>

Ejecuta un pipeline en modo de herramienta.

```json
{
  "action": "run",
  "pipeline": "gog.gmail.search --query 'newer_than:1d' | email.triage",
  "cwd": "/path/to/workspace",
  "timeoutMs": 30000,
  "maxStdoutBytes": 512000
}
```

Ejecutar un archivo de flujo de trabajo con argumentos:

```json
{
  "action": "run",
  "pipeline": "/path/to/inbox-triage.lobster",
  "argsJson": "{\"tag\":\"family\"}"
}
```

<div id="resume">
  ### `resume`
</div>

Reanuda un flujo de trabajo detenido tras la aprobación.

```json
{
  "action": "resume",
  "token": "<resumeToken>",
  "approve": true
}
```

<div id="optional-inputs">
  ### Entradas opcionales
</div>

* `lobsterPath`: Ruta absoluta del binario de Lobster (déjalo en blanco para usar `PATH`).
* `cwd`: Directorio de trabajo para el pipeline (por defecto, el directorio de trabajo del proceso actual).
* `timeoutMs`: Finaliza el subproceso si excede esta duración (predeterminado: 20000).
* `maxStdoutBytes`: Finaliza el subproceso si stdout excede este tamaño (predeterminado: 512000).
* `argsJson`: Cadena JSON pasada a `lobster run --args-json` (solo para archivos de flujo de trabajo).

<div id="output-envelope">
  ## Envoltorio de salida
</div>

Lobster devuelve un envoltorio JSON con uno de tres estados posibles:

* `ok` → finalizado correctamente
* `needs_approval` → en pausa; se requiere `requiresApproval.resumeToken` para reanudar
* `cancelled` → denegado explícitamente o cancelado

La herramienta expone el envoltorio tanto en `content` (JSON legible) como en `details` (objeto sin procesar).

<div id="approvals">
  ## Aprobaciones
</div>

Si `requiresApproval` está presente, inspecciona el prompt y toma una decisión:

* `approve: true` → reanudar y continuar con los efectos secundarios
* `approve: false` → cancelar y finalizar el flujo de trabajo

Usa `approve --preview-from-stdin --limit N` para adjuntar una vista previa JSON a las solicitudes de aprobación sin necesidad de usar combinaciones personalizadas con jq/heredoc. Los tokens de reanudación ahora son compactos: Lobster almacena el estado de reanudación del flujo de trabajo en su directorio de estado y devuelve una pequeña clave de token.

<div id="openprose">
  ## OpenProse
</div>

OpenProse combina bien con Lobster: utiliza `/prose` para orquestar la preparación de múltiples agentes y luego ejecuta un pipeline de Lobster para aprobaciones deterministas. Si un programa Prose necesita Lobster, permite la herramienta `lobster` para subagentes mediante `tools.subagents.tools`. Consulta [OpenProse](/es/prose).

<div id="safety">
  ## Seguridad
</div>

* **Solo subproceso local** — sin llamadas de red desde el propio complemento.
* **Sin secretos** — Lobster no gestiona OAuth; invoca herramientas de OpenClaw que sí lo hacen.
* **Consciente del sandbox** — se desactiva cuando el contexto de la herramienta se ejecuta en sandbox.
* **Endurecido** — `lobsterPath` debe ser absoluto si se especifica; se aplican tiempos de espera y límites de salida estrictos.

<div id="troubleshooting">
  ## Solución de problemas
</div>

* **`lobster subprocess timed out`** → aumenta `timeoutMs`, o divide una pipeline larga.
* **`lobster output exceeded maxStdoutBytes`** → incrementa `maxStdoutBytes` o reduce el tamaño de la salida.
* **`lobster returned invalid JSON`** → asegúrate de que la pipeline se ejecute en modo herramienta y que solo imprima JSON.
* **`lobster failed (code …)`** → ejecuta la misma pipeline en un terminal para inspeccionar stderr.

<div id="learn-more">
  ## Más información
</div>

* [Complementos](/es/plugin)
* [Creación de herramientas para complementos](/es/plugins/agent-tools)

<div id="case-study-community-workflows">
  ## Estudio de caso: flujos de trabajo de la comunidad
</div>

Un ejemplo público: un CLI de “segundo cerebro” + pipelines de Lobster que gestionan tres bóvedas de Markdown (personal, de pareja, compartida). El CLI emite JSON para estadísticas, listados de la bandeja de entrada y análisis de elementos obsoletos; Lobster encadena esos comandos en flujos de trabajo como `weekly-review`, `inbox-triage`, `memory-consolidation` y `shared-task-sync`, cada uno con puntos de aprobación. La IA se encarga del criterio (clasificación) cuando está disponible y recurre a reglas deterministas cuando no lo está.

* Hilo: https://x.com/plattenschieber/status/2014508656335770033
* Repo: https://github.com/bloomedai/brain-cli