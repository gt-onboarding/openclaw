---
title: Prosa
summary: "OpenProse: flujos de trabajo .prose, comandos de barra diagonal (/) y estado en OpenClaw"
read_when:
  - Quieres ejecutar o escribir flujos de trabajo .prose
  - Quieres habilitar el complemento OpenProse
  - Necesitas entender el almacenamiento de estado
---

<div id="openprose">
  # OpenProse
</div>

OpenProse es un formato de flujo de trabajo portátil, centrado en Markdown, para orquestar sesiones de IA. En OpenClaw se distribuye como un complemento que instala un paquete de habilidades de OpenProse más un comando slash `/prose`. Los programas viven en archivos `.prose` y pueden lanzar múltiples subagentes con un control de flujo explícito.

Sitio web oficial: https://www.prose.md

<div id="what-it-can-do">
  ## Lo que puede hacer
</div>

* Investigación y síntesis multiagente con paralelismo explícito.
* Flujos de trabajo repetibles y seguros para aprobaciones (revisión de código, triaje de incidentes, pipelines de contenido).
* Programas `.prose` reutilizables que puedes ejecutar en runtimes de agentes compatibles.

<div id="install-enable">
  ## Instalar y habilitar
</div>

Los complementos integrados están deshabilitados de forma predeterminada. Activa OpenProse:

```bash
openclaw plugins enable open-prose
```

Reinicia el Gateway después de activar el complemento.

Dev/local checkout: `openclaw plugins install ./extensions/open-prose`

Documentación relacionada: [Complementos](/es/plugin), [Manifiesto de complementos](/es/plugins/manifest), [Habilidades](/es/tools/skills).

<div id="slash-command">
  ## Comando de barra inclinada
</div>

OpenProse registra `/prose` como un comando de habilidad invocable por el usuario. Se dirige a las instrucciones de la máquina virtual (VM) de OpenProse y utiliza herramientas de OpenClaw internamente.

Comandos comunes:

```
/prose help
/prose run <file.prose>
/prose run <handle/slug>
/prose run <https://example.com/file.prose>
/prose compile <file.prose>
/prose examples
/prose update
```

<div id="example-a-simple-prose-file">
  ## Ejemplo: archivo `.prose` simple
</div>

```prose
# Research + synthesis with two agents running in parallel.

input topic: "What should we research?"

agent researcher:
  model: sonnet
  prompt: "You research thoroughly and cite sources."

agent writer:
  model: opus
  prompt: "You write a concise summary."

parallel:
  findings = session: researcher
    prompt: "Research {topic}."
  draft = session: writer
    prompt: "Summarize {topic}."

session "Merge the findings + draft into a final answer."
context: { findings, draft }
```

<div id="file-locations">
  ## Ubicaciones de archivos
</div>

OpenProse almacena el estado en `.prose/` dentro de tu espacio de trabajo:

```
.prose/
├── .env
├── runs/
│   └── {YYYYMMDD}-{HHMMSS}-{random}/
│       ├── program.prose
│       ├── state.md
│       ├── bindings/
│       └── agents/
└── agents/
```

Los agentes persistentes de usuario se encuentran en:

```
~/.prose/agents/
```

<div id="state-modes">
  ## Modos de estado
</div>

OpenProse admite varios backends de estado:

* **filesystem** (predeterminado): `.prose/runs/...`
* **in-context**: transitorio, para programas pequeños
* **sqlite** (experimental): requiere el binario `sqlite3`
* **postgres** (experimental): requiere `psql` y una cadena de conexión

Notas:

* sqlite/postgres son opcionales y experimentales.
* Las credenciales de postgres se registran en los logs de los subagentes; utiliza una base de datos dedicada con el menor privilegio posible.

<div id="remote-programs">
  ## Programas remotos
</div>

`/prose run <handle/slug>` se resuelve en `https://p.prose.md/<handle>/<slug>`.
Las URL directas se recuperan tal cual. Esto utiliza la herramienta `web_fetch` (o `exec` para POST).

<div id="openclaw-runtime-mapping">
  ## Correspondencia del entorno de ejecución de OpenClaw
</div>

Los programas de OpenProse se corresponden con primitivas de OpenClaw:

| Concepto de OpenProse | Herramienta de OpenClaw |
| --- | --- |
| Iniciar sesión / herramienta Task | `sessions_spawn` |
| Lectura/escritura de archivos | `read` / `write` |
| Web fetch | `web_fetch` |

Si la lista de permitidos de tus herramientas bloquea estas herramientas, los programas de OpenProse fallarán. Consulta [configuración de habilidades](/es/tools/skills-config).

<div id="security-approvals">
  ## Seguridad y aprobaciones
</div>

Trata los archivos `.prose` como código. Revísalos antes de ejecutarlos. Usa listas de permitidos de herramientas de OpenClaw y compuertas de aprobación para controlar los efectos secundarios.

Para flujos de trabajo deterministas y con aprobaciones, consulta [Lobster](/es/tools/lobster).