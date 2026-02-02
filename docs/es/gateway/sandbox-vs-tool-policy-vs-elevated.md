---
title: Sandbox vs política de herramientas vs ejecución elevada
summary: "Por qué una herramienta está bloqueada: entorno de ejecución de sandbox, política de permitir/denegar herramientas y controles de ejecución elevada"
read_when: "Te topas con la 'sandbox jail' o ves que se rechaza una herramienta/ejecución elevada y quieres saber exactamente qué clave de configuración cambiar."
status: active
---

<div id="sandbox-vs-tool-policy-vs-elevated">
  # Sandbox vs Tool Policy vs Elevated
</div>

OpenClaw tiene tres controles relacionados (pero diferentes):

1. **Sandbox** (`agents.defaults.sandbox.*` / `agents.list[].sandbox.*`) decide **dónde se ejecutan las herramientas** (Docker vs host).
2. **Tool policy** (`tools.*`, `tools.sandbox.tools.*`, `agents.list[].tools.*`) decide **qué herramientas están disponibles/permitidas**.
3. **Elevated** (`tools.elevated.*`, `agents.list[].tools.elevated.*`) es una **vía de escape solo de ejecución** para ejecutarse en el host cuando estás en un entorno sandbox.

<div id="quick-debug">
  ## Depuración rápida
</div>

Usa el inspector para ver qué está haciendo *realmente* OpenClaw:

```bash
openclaw sandbox explain
openclaw sandbox explain --session agent:main:main
openclaw sandbox explain --agent work
openclaw sandbox explain --json
```

Muestra:

* modo efectivo de sandbox / ámbito / acceso al espacio de trabajo
* si la sesión está actualmente en sandbox (main vs non-main)
* permisos efectivos de herramientas en la sandbox (permitir/denegar) y si proceden de agent/global/default
* compuertas elevadas y rutas de claves de corrección (fix-it)

<div id="sandbox-where-tools-run">
  ## Sandbox: dónde se ejecutan las herramientas
</div>

La ejecución en sandbox se controla mediante `agents.defaults.sandbox.mode`:

* `"off"`: todo se ejecuta en el host.
* `"non-main"`: solo las sesiones no principales se ejecutan en sandbox (fuente común de “sorpresas” en grupos/canales).
* `"all"`: todo se ejecuta en sandbox.

Consulta [Sandboxing](/es/gateway/sandboxing) para ver la matriz completa (ámbito, montajes del espacio de trabajo, imágenes).

<div id="bind-mounts-security-quick-check">
  ### Montajes bind (comprobación rápida de seguridad)
</div>

* `docker.binds` *atraviesa* el sistema de archivos del sandbox: todo lo que montes será visible dentro del contenedor con el modo que configures (`:ro` o `:rw`).
* El modo predeterminado es lectura-escritura si omites el modo; prioriza `:ro` para código fuente/secretos.
* `scope: "shared"` ignora los montajes bind por agente (solo se aplican los montajes bind globales).
* Vincular `/var/run/docker.sock` entrega efectivamente el control del host al sandbox; hazlo solo de forma deliberada.
* El acceso al espacio de trabajo (`workspaceAccess: "ro"`/`"rw"`) es independiente de los modos de bind.

<div id="tool-policy-which-tools-existare-callable">
  ## Política de herramientas: qué herramientas existen/son invocables
</div>

Importan dos capas:

* **Perfil de herramienta**: `tools.profile` y `agents.list[].tools.profile` (lista de permitidos base)
* **Perfil de herramienta del proveedor**: `tools.byProvider[provider].profile` y `agents.list[].tools.byProvider[provider].profile`
* **Política de herramientas global/por agente**: `tools.allow`/`tools.deny` y `agents.list[].tools.allow`/`agents.list[].tools.deny`
* **Política de herramientas del proveedor**: `tools.byProvider[provider].allow/deny` y `agents.list[].tools.byProvider[provider].allow/deny`
* **Política de herramientas del sandbox** (solo aplica cuando la sesión está en sandbox): `tools.sandbox.tools.allow`/`tools.sandbox.tools.deny` y `agents.list[].tools.sandbox.tools.*`

Reglas generales:

* `deny` siempre gana.
* Si `allow` no está vacío, todo lo demás se considera bloqueado.
* La política de herramientas es el límite estricto: `/exec` no puede anular una herramienta `exec` denegada.
* `/exec` solo cambia los valores predeterminados de la sesión para remitentes autorizados; no concede acceso a herramientas.

Las claves de herramientas del proveedor aceptan tanto `provider` (p. ej., `google-antigravity`) como `provider/model` (p. ej., `openai/gpt-5.2`).

<div id="tool-groups-shorthands">
  ### Grupos de herramientas (formas abreviadas)
</div>

Las políticas de herramientas (globales, de agente, de sandbox) admiten entradas de tipo `group:*` que se expanden en varias herramientas:

```json5
{
  tools: {
    sandbox: {
      tools: {
        allow: ["group:runtime", "group:fs", "group:sessions", "group:memory"]
      }
    }
  }
}
```

Grupos disponibles:

* `group:runtime`: `exec`, `bash`, `process`
* `group:fs`: `read`, `write`, `edit`, `apply_patch`
* `group:sessions`: `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status`
* `group:memory`: `memory_search`, `memory_get`
* `group:ui`: `browser`, `canvas`
* `group:automation`: `cron`, `gateway`
* `group:messaging`: `message`
* `group:nodes`: `nodes`
* `group:openclaw`: todas las herramientas integradas de OpenClaw (excluye complementos de proveedores)

<div id="elevated-exec-only-run-on-host">
  ## Elevated: solo exec, “ejecutar en el host”
</div>

Elevated **no** otorga herramientas adicionales; solo afecta a `exec`.

* Si estás en sandbox, `/elevated on` (o `exec` con `elevated: true`) se ejecuta en el host (las aprobaciones pueden seguir siendo necesarias).
* Usa `/elevated full` para omitir aprobaciones de exec para la sesión.
* Si ya estás ejecutando en modo directo, elevated es efectivamente un no-op (sigue estando sujeto a controles).
* Elevated **no** tiene ámbito de skill y **no** anula las listas de permitidos/prohibidos de herramientas.
* `/exec` es independiente de elevated. Solo ajusta los valores predeterminados de exec por sesión para remitentes autorizados.

Controles:

* Habilitación: `tools.elevated.enabled` (y opcionalmente `agents.list[].tools.elevated.enabled`)
* Listas de permitidos de remitentes: `tools.elevated.allowFrom.<provider>` (y opcionalmente `agents.list[].tools.elevated.allowFrom.<provider>`)

Consulta [Modo Elevated](/es/tools/elevated).

<div id="common-sandbox-jail-fixes">
  ## Soluciones habituales al aislamiento de la sandbox
</div>

<div id="tool-x-blocked-by-sandbox-tool-policy">
  ### “Herramienta X bloqueada por la política de herramientas de la sandbox”
</div>

Opciones de corrección (elige una):

* Desactiva la sandbox: `agents.defaults.sandbox.mode=off` (o por agente `agents.list[].sandbox.mode=off`)
* Permite la herramienta dentro de la sandbox:
  * elimínala de `tools.sandbox.tools.deny` (o por agente `agents.list[].tools.sandbox.tools.deny`)
  * o añádela a `tools.sandbox.tools.allow` (o por agente en allow)

<div id="i-thought-this-was-main-why-is-it-sandboxed">
  ### «Pensaba que esto era main, ¿por qué está en sandbox?»
</div>

En el modo `"non-main"`, las claves de grupo/canal *no* son main. Usa la clave de la sesión main (que se muestra con `sandbox explain`) o cambia el modo a `"off"`.