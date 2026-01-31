---
title: Backends de CLI
summary: "Backends de CLI: mecanismo de respaldo solo texto mediante CLIs de IA locales"
read_when:
  - Quieres un mecanismo de respaldo confiable cuando fallen los proveedores de API
  - Estás ejecutando Claude Code CLI u otras CLIs de IA locales y quieres reutilizarlas
  - Necesitas un flujo solo de texto, sin herramientas, que aun así admita sesiones e imágenes
---

<div id="cli-backends-fallback-runtime">
  # Backends de CLI (runtime de respaldo)
</div>

OpenClaw puede ejecutar **CLI de IA locales** como **mecanismo de respaldo solo de texto** cuando los proveedores de API están caídos,
sujetos a límites de uso o se comportan de forma anómala. Esto es deliberadamente conservador:

- **Las herramientas están deshabilitadas** (no hay llamadas a herramientas).
- **Texto de entrada → texto de salida** (fiable).
- **Se admiten sesiones** (así las interacciones posteriores se mantienen coherentes).
- **Las imágenes se pueden pasar** si la CLI acepta rutas de imágenes.

Esto está pensado como una **red de seguridad** más que como vía principal. Úsalo cuando
quieras respuestas de texto “que siempre funcionan” sin depender de APIs externas.

<div id="beginner-friendly-quick-start">
  ## Inicio rápido para principiantes
</div>

Puedes usar Claude Code CLI **sin necesidad de configuración** (OpenClaw incluye una configuración predeterminada integrada):

```bash
openclaw agent --message "hi" --model claude-cli/opus-4.5
```

Codex CLI también funciona de inmediato sin necesidad de configuración adicional:

```bash
openclaw agent --message "hi" --model codex-cli/gpt-5.2-codex
```

Si tu Gateway se ejecuta con launchd/systemd y PATH es muy limitado, añade únicamente la
ruta del comando:

```json5
{
  agents: {
    defaults: {
      cliBackends: {
        "claude-cli": {
          command: "/opt/homebrew/bin/claude"
        }
      }
    }
  }
}
```

Eso es todo. No necesitas claves ni configuración de autenticación adicional aparte de la propia CLI.


<div id="using-it-as-a-fallback">
  ## Usarlo como mecanismo de respaldo
</div>

Agrega un backend de CLI a tu lista de respaldo para que solo se ejecute cuando fallen los modelos principales:

```json5
{
  agents: {
    defaults: {
      model: {
        primary: "anthropic/claude-opus-4-5",
        fallbacks: [
          "claude-cli/opus-4.5"
        ]
      },
      models: {
        "anthropic/claude-opus-4-5": { alias: "Opus" },
        "claude-cli/opus-4.5": {}
      }
    }
  }
}
```

Notas:

* Si usas `agents.defaults.models` (lista de permitidos), debes incluir `claude-cli/...`.
* Si el proveedor principal falla (autenticación, límites de tasa, timeouts), OpenClaw
  intentará usar el backend de la CLI como siguiente opción.


<div id="configuration-overview">
  ## Descripción general de la configuración
</div>

Todos los backends de la CLI se encuentran en:

```
agents.defaults.cliBackends
```

Cada entrada se identifica mediante un **ID de proveedor** (por ejemplo, `claude-cli`, `my-cli`).
El ID de proveedor se convierte en el lado izquierdo de la referencia de tu modelo:

```
<provider>/<model>
```


<div id="example-configuration">
  ### Ejemplo de configuración
</div>

```json5
{
  agents: {
    defaults: {
      cliBackends: {
        "claude-cli": {
          command: "/opt/homebrew/bin/claude"
        },
        "my-cli": {
          command: "my-cli",
          args: ["--json"],
          output: "json",
          input: "arg",
          modelArg: "--model",
          modelAliases: {
            "claude-opus-4-5": "opus",
            "claude-sonnet-4-5": "sonnet"
          },
          sessionArg: "--session",
          sessionMode: "existing",
          sessionIdFields: ["session_id", "conversation_id"],
          systemPromptArg: "--system",
          systemPromptWhen: "first",
          imageArg: "--image",
          imageMode: "repeat",
          serialize: true
        }
      }
    }
  }
}
```


<div id="how-it-works">
  ## Cómo funciona
</div>

1) **Selecciona un backend** basándose en el prefijo del proveedor (`claude-cli/...`).
2) **Construye un prompt de sistema** usando el mismo prompt de OpenClaw + el contexto del espacio de trabajo.
3) **Ejecuta la CLI** con un ID de sesión (si es compatible) para que el historial se mantenga coherente.
4) **Analiza la salida** (JSON o texto plano) y devuelve el texto final.
5) **Guarda los ID de sesión** por backend, de modo que los mensajes posteriores reutilicen la misma sesión de CLI.

<div id="sessions">
  ## Sesiones
</div>

- Si la CLI admite sesiones, configura `sessionArg` (p. ej. `--session-id`) o
  `sessionArgs` (marcador de posición `{sessionId}`) cuando sea necesario insertar el ID
  en varios flags.
- Si la CLI usa un **subcomando de reanudación** con flags diferentes, configura
  `resumeArgs` (reemplaza `args` al reanudar) y, opcionalmente, `resumeOutput`
  (para reanudaciones que no sean JSON).
- `sessionMode`:
  - `always`: enviar siempre un ID de sesión (nuevo UUID si no hay uno almacenado).
  - `existing`: enviar un ID de sesión solo si ya se había almacenado uno.
  - `none`: no enviar nunca un ID de sesión.

<div id="images-pass-through">
  ## Imágenes (paso directo)
</div>

Si tu CLI admite rutas de imagen, configura `imageArg`:

```json5
imageArg: "--image",
imageMode: "repeat"
```

OpenClaw escribirá imágenes en base64 en archivos temporales. Si `imageArg` está definido, esas rutas se pasan como argumentos de la CLI. Si `imageArg` falta, OpenClaw añade las rutas de archivo al prompt (inyección de ruta), lo cual es suficiente para las CLIs que cargan automáticamente archivos locales desde rutas en texto plano (comportamiento de Claude Code CLI).


<div id="inputs-outputs">
  ## Entradas / salidas
</div>

- `output: "json"` (predeterminado) intenta analizar JSON y extraer texto + ID de sesión.
- `output: "jsonl"` analiza flujos JSONL (CLI de Codex `--json`) y extrae el
  último mensaje del agente más `thread_id` cuando está presente.
- `output: "text"` trata stdout como la respuesta final.

Modos de entrada:

- `input: "arg"` (predeterminado) pasa el prompt como el último argumento de la CLI.
- `input: "stdin"` envía el prompt a través de stdin.
- Si el prompt es muy largo y `maxPromptArgChars` está establecido, se usa stdin.

<div id="defaults-built-in">
  ## Valores predeterminados (integrados)
</div>

OpenClaw proporciona una configuración predeterminada para `claude-cli`:

- `command: "claude"`
- `args: ["-p", "--output-format", "json", "--dangerously-skip-permissions"]`
- `resumeArgs: ["-p", "--output-format", "json", "--dangerously-skip-permissions", "--resume", "{sessionId}"]`
- `modelArg: "--model"`
- `systemPromptArg: "--append-system-prompt"`
- `sessionArg: "--session-id"`
- `systemPromptWhen: "first"`
- `sessionMode: "always"`

OpenClaw también proporciona una configuración predeterminada para `codex-cli`:

- `command: "codex"`
- `args: ["exec","--json","--color","never","--sandbox","read-only","--skip-git-repo-check"]`
- `resumeArgs: ["exec","resume","{sessionId}","--color","never","--sandbox","read-only","--skip-git-repo-check"]`
- `output: "jsonl"`
- `resumeOutput: "text"`
- `modelArg: "--model"`
- `imageArg: "--image"`
- `sessionMode: "existing"`

Anula esta configuración solo si es necesario (caso habitual: ruta absoluta de `command`).

<div id="limitations">
  ## Limitaciones
</div>

- **Sin herramientas de OpenClaw** (el backend de la CLI nunca recibe llamadas a herramientas). Algunas CLI
  aún pueden ejecutar su propio conjunto de herramientas de agente.
- **Sin streaming** (la salida de la CLI se recopila y luego se devuelve).
- **Las salidas estructuradas** dependen del formato JSON de la CLI.
- **Las sesiones de Codex CLI** se reanudan mediante salida de texto (sin JSONL), que es menos
  estructurada que la ejecución inicial con `--json`. Las sesiones de OpenClaw siguen funcionando
  con normalidad.

<div id="troubleshooting">
  ## Solución de problemas
</div>

- **CLI no encontrada**: establece `command` en una ruta absoluta.
- **Nombre de modelo incorrecto**: utiliza `modelAliases` para mapear `provider/model` → modelo de la CLI.
- **Sin continuidad de sesión**: asegúrate de que `sessionArg` esté configurado y que `sessionMode` no sea
  `none` (actualmente Codex CLI no puede reanudarse con salida JSON).
- **Imágenes ignoradas**: establece `imageArg` (y verifica que la CLI admita rutas de archivos).