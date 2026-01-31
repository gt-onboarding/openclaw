---
title: Habilidades
summary: "Habilidades: gestionadas vs en espacio de trabajo, reglas de habilitación y carga, y conexión de config/env"
read_when:
  - Agregar o modificar habilidades
  - Cambiar las reglas de habilitación o carga de habilidades
---

<div id="skills-openclaw">
  # Habilidades (OpenClaw)
</div>

OpenClaw usa carpetas de habilidades **compatibles con [AgentSkills](https://agentskills.io)** para enseñar al agente a usar herramientas. Cada habilidad es un directorio que contiene un `SKILL.md` con frontmatter YAML e instrucciones. OpenClaw carga **habilidades integradas** más anulaciones locales opcionales y las filtra en tiempo de carga según el entorno, la configuración y la presencia de binarios.

<div id="locations-and-precedence">
  ## Ubicaciones y precedencia
</div>

Las habilidades se cargan desde **tres** lugares:

1. **Habilidades integradas**: incluidas con la instalación (paquete de npm o OpenClaw.app)
2. **Habilidades administradas/locales**: `~/.openclaw/skills`
3. **Habilidades del espacio de trabajo**: `<workspace>/skills`

Si hay un conflicto de nombres de habilidades, la precedencia es:

`<workspace>/skills` (más alta) → `~/.openclaw/skills` → habilidades integradas (más baja)

Además, puedes configurar carpetas adicionales de habilidades (con la precedencia más baja) mediante
`skills.load.extraDirs` en `~/.openclaw/openclaw.json`.

<div id="per-agent-vs-shared-skills">
  ## Habilidades por agente vs compartidas
</div>

En configuraciones **multiagente**, cada agente tiene su propio espacio de trabajo. Eso significa:

* Las **habilidades por agente** se encuentran en `<workspace>/skills` solo para ese agente.
* Las **habilidades compartidas** se encuentran en `~/.openclaw/skills` (gestionadas/locales) y son visibles
  para **todos los agentes** en la misma máquina.
* También se pueden añadir **carpetas compartidas** mediante `skills.load.extraDirs` (con la
  precedencia más baja) si deseas un paquete de habilidades común usado por varios agentes.

Si el mismo nombre de habilidad existe en más de un lugar, se aplica la precedencia habitual:
gana el espacio de trabajo, luego gestionadas/locales y después las integradas.

<div id="plugins-skills">
  ## Complementos + habilidades
</div>

Los complementos pueden incluir sus propias habilidades indicando directorios `skills` en
`openclaw.plugin.json` (rutas relativas a la raíz del complemento). Las habilidades del complemento se cargan
cuando el complemento está habilitado y participan en las reglas normales de precedencia de habilidades.
Puedes condicionarlas mediante `metadata.openclaw.requires.config` en la entrada de configuración del complemento.
Consulta [Complementos](/es/plugin) para el descubrimiento y la configuración, y [Herramientas](/es/tools) para la superficie de herramientas que esas habilidades exponen.

<div id="clawhub-install-sync">
  ## ClawHub (instalación + sincronización)
</div>

ClawHub es el registro público de habilidades para OpenClaw. Explóralo en
https://clawhub.com. Úsalo para descubrir, instalar, actualizar y hacer copias de seguridad de habilidades.
Guía completa: [ClawHub](/es/tools/clawhub).

Flujos habituales:

* Instalar una habilidad en tu espacio de trabajo:
  * `clawhub install <skill-slug>`
* Actualizar todas las habilidades instaladas:
  * `clawhub update --all`
* Sincronizar (escanear + publicar actualizaciones):
  * `clawhub sync --all`

De forma predeterminada, `clawhub` instala en `./skills` dentro de tu directorio
de trabajo actual (o, en su defecto, en el espacio de trabajo de OpenClaw configurado). OpenClaw lo reconocerá como `<workspace>/skills` en la siguiente sesión.

<div id="security-notes">
  ## Notas de seguridad
</div>

* Trata las habilidades de terceros como **código confiable**. Léelas antes de habilitarlas.
* Prefiere ejecuciones en sandbox para entradas no confiables y herramientas arriesgadas. Consulta [Sandboxing](/es/gateway/sandboxing).
* `skills.entries.*.env` y `skills.entries.*.apiKey` inyectan secretos en el proceso **host**
  de ese turno del agente (no en el sandbox). Mantén los secretos fuera de los prompts y de los registros.
* Para un modelo de amenazas más amplio y listas de verificación, consulta [Security](/es/gateway/security).

<div id="format-agentskills-pi-compatible">
  ## Formato (AgentSkills + compatible con Pi)
</div>

`SKILL.md` debe incluir como mínimo:

```markdown
---
name: nano-banana-pro
description: Generar o editar imágenes a través de Gemini 3 Pro Image
---
```

Notas:

* Seguimos la especificación AgentSkills para la estructura/intención.
* El analizador usado por el agente incrustado solo admite claves de frontmatter de **una sola línea**.
* `metadata` debe ser un **objeto JSON de una sola línea**.
* Usa `{baseDir}` en las instrucciones para hacer referencia a la ruta de la carpeta de la habilidad.
* Claves de frontmatter opcionales:
  * `homepage` — URL mostrada como “Website” en la UI de Skills de macOS (también compatible mediante `metadata.openclaw.homepage`).
  * `user-invocable` — `true|false` (predeterminado: `true`). Si es `true`, la habilidad se expone como un comando slash para el usuario.
  * `disable-model-invocation` — `true|false` (predeterminado: `false`). Si es `true`, la habilidad se excluye del prompt del modelo (sigue disponible mediante invocación del usuario).
  * `command-dispatch` — `tool` (opcional). Cuando se establece en `tool`, el comando slash omite el modelo y se despacha directamente a una herramienta.
  * `command-tool` — nombre de la herramienta que se invocará cuando `command-dispatch: tool` esté configurado.
  * `command-arg-mode` — `raw` (predeterminado). Para el despacho a herramientas, reenvía la cadena de argumentos sin procesar a la herramienta (sin análisis adicional por el núcleo).

    La herramienta se invoca con los parámetros:
    `{ command: "<raw args>", commandName: "<slash command>", skillName: "<skill name>" }`.

<div id="gating-load-time-filters">
  ## Gating (filtros en tiempo de carga)
</div>

OpenClaw **filtra las habilidades al cargarlas** usando `metadata` (JSON en una sola línea):

```markdown
---
name: nano-banana-pro
description: Generar o editar imágenes a través de Gemini 3 Pro Image
metadata: {"openclaw":{"requires":{"bins":["uv"],"env":["GEMINI_API_KEY"],"config":["browser.enabled"]},"primaryEnv":"GEMINI_API_KEY"}}
---
```

Campos bajo `metadata.openclaw`:

* `always: true` — incluye siempre la habilidad (ignora otras compuertas/condiciones).
* `emoji` — emoji opcional usado por la UI de Habilidades de macOS.
* `homepage` — URL opcional mostrada como “Website” en la UI de Habilidades de macOS.
* `os` — lista opcional de plataformas (`darwin`, `linux`, `win32`). Si se establece, la habilidad solo es elegible en esos sistemas operativos.
* `requires.bins` — lista; cada uno debe existir en el `PATH`.
* `requires.anyBins` — lista; al menos uno debe existir en el `PATH`.
* `requires.env` — lista; la variable de entorno debe existir **o** proporcionarse en la configuración.
* `requires.config` — lista de rutas de `openclaw.json` que deben ser verdaderas (truthy).
* `primaryEnv` — nombre de la variable de entorno asociada con `skills.entries.<name>.apiKey`.
* `install` — array opcional de especificaciones de instalador usadas por la UI de Habilidades de macOS (brew/node/go/uv/download).

Nota sobre sandboxing:

* `requires.bins` se comprueba en el **host** en el momento de carga de la habilidad.
* Si un agente está aislado en un sandbox, el binario también debe existir **dentro del contenedor**.
  Instálalo mediante `agents.defaults.sandbox.docker.setupCommand` (o una imagen personalizada).
  `setupCommand` se ejecuta una vez después de que se crea el contenedor.
  Las instalaciones de paquetes también requieren salida de red, un sistema de archivos raíz escribible y un usuario root en el sandbox.
  Ejemplo: la habilidad `summarize` (`skills/summarize/SKILL.md`) necesita el CLI `summarize`
  en el contenedor del sandbox para ejecutarse allí.

Ejemplo de instalador:

```markdown
---
name: gemini
description: Usa Gemini CLI para asistencia de codificación y búsquedas en Google.
metadata: {"openclaw":{"emoji":"♊️","requires":{"bins":["gemini"]},"install":[{"id":"brew","kind":"brew","formula":"gemini-cli","bins":["gemini"],"label":"Install Gemini CLI (brew)"}]}}
---
```

Notas:

* Si se listan varios instaladores, el Gateway elige una **única** opción preferida (brew cuando está disponible, de lo contrario Node).
* Si todos los instaladores son `download`, OpenClaw enumera cada entrada para que puedas ver los artefactos disponibles.
* Las especificaciones del instalador pueden incluir `os: ["darwin"|"linux"|"win32"]` para filtrar opciones por plataforma.
* Las instalaciones de Node respetan `skills.install.nodeManager` en `openclaw.json` (valor por defecto: npm; opciones: npm/pnpm/yarn/bun).
  Esto solo afecta a las **instalaciones de habilidades**; el runtime del Gateway debe seguir siendo Node
  (no se recomienda Bun para WhatsApp/Telegram).
* Instalaciones de Go: si `go` no está instalado y `brew` está disponible, el Gateway instala Go primero a través de Homebrew y establece `GOBIN` al `bin` de Homebrew cuando es posible.
* Instalaciones por descarga: `url` (obligatorio), `archive` (`tar.gz` | `tar.bz2` | `zip`), `extract` (por defecto: automático cuando se detecta un archivo comprimido), `stripComponents`, `targetDir` (por defecto: `~/.openclaw/tools/<skillKey>`).

Si no hay ningún `metadata.openclaw`, la habilidad siempre será elegible (a menos que
esté deshabilitada en la configuración o bloqueada por `skills.allowBundled` para habilidades integradas).

<div id="config-overrides-openclawopenclawjson">
  ## Anulaciones de configuración (`~/.openclaw/openclaw.json`)
</div>

Las habilidades incluidas/administradas se pueden activar o desactivar y configurarse con valores de entorno:

```json5
{
  skills: {
    entries: {
      "nano-banana-pro": {
        enabled: true,
        apiKey: "GEMINI_KEY_HERE",
        env: {
          GEMINI_API_KEY: "GEMINI_KEY_HERE"
        },
        config: {
          endpoint: "https://example.invalid",
          model: "nano-pro"
        }
      },
      peekaboo: { enabled: true },
      sag: { enabled: false }
    }
  }
}
```

Nota: si el nombre de la habilidad contiene guiones, coloca la clave entre comillas (JSON5 permite claves entre comillas).

Las claves de configuración coinciden con el **nombre de la habilidad** de forma predeterminada. Si una habilidad define
`metadata.openclaw.skillKey`, usa esa clave dentro de `skills.entries`.

Reglas:

* `enabled: false` desactiva la habilidad incluso si viene empaquetada/instalada.
* `env`: se inyecta **solo si** la variable aún no está definida en el proceso.
* `apiKey`: atajo para habilidades que declaran `metadata.openclaw.primaryEnv`.
* `config`: contenedor opcional para campos personalizados por habilidad; las claves personalizadas deben definirse aquí.
* `allowBundled`: lista de permitidos opcional solo para habilidades **empaquetadas**. Si se establece, solo
  las habilidades empaquetadas incluidas en la lista son válidas (las habilidades gestionadas/del espacio de trabajo no se ven afectadas).

<div id="environment-injection-per-agent-run">
  ## Inyección de entorno (por ejecución de agente)
</div>

Cuando comienza una ejecución de agente, OpenClaw:

1. Lee los metadatos de las habilidades.
2. Aplica cualquier `skills.entries.<key>.env` o `skills.entries.<key>.apiKey` a
   `process.env`.
3. Construye el system prompt con las habilidades **habilitadas**.
4. Restaura el entorno original después de que termina la ejecución.

Esto se **limita a la ejecución del agente**, no a un entorno global de shell.

<div id="session-snapshot-performance">
  ## Instantánea de sesión (rendimiento)
</div>

OpenClaw toma una instantánea de las habilidades elegibles **cuando comienza una sesión** y reutiliza esa lista para los turnos posteriores en la misma sesión. Los cambios en las habilidades o la configuración entran en vigor en la próxima sesión nueva.

Las habilidades también pueden actualizarse a mitad de sesión cuando el watcher de habilidades está habilitado o cuando aparece un nuevo nodo remoto elegible (ver más abajo). Piensa en esto como un **hot reload**: la lista actualizada se aplica en el siguiente turno del agente.

<div id="remote-macos-nodes-linux-gateway">
  ## Nodos remotos de macOS (Gateway en Linux)
</div>

Si el Gateway se está ejecutando en Linux pero hay un **nodo de macOS** conectado **con `system.run` permitido** (la seguridad de aprobaciones de ejecución no está configurada en `deny`), OpenClaw puede tratar las habilidades exclusivas de macOS como disponibles cuando los binarios requeridos están presentes en ese nodo. El agente debería ejecutar esas habilidades mediante la herramienta `nodes` (normalmente `nodes.run`).

Esto depende de que el nodo informe de los comandos que admite y de una comprobación de binarios mediante `system.run`. Si el nodo de macOS se desconecta posteriormente, las habilidades seguirán siendo visibles; las invocaciones pueden fallar hasta que el nodo se vuelva a conectar.

<div id="skills-watcher-auto-refresh">
  ## Monitor de habilidades (actualización automática)
</div>

De forma predeterminada, OpenClaw supervisa las carpetas de habilidades y actualiza la instantánea de habilidades cuando se modifican los archivos `SKILL.md`. Configura esto en `skills.load`:

```json5
{
  skills: {
    load: {
      watch: true,
      watchDebounceMs: 250
    }
  }
}
```

<div id="token-impact-skills-list">
  ## Impacto en los tokens (lista de habilidades)
</div>

Cuando las habilidades pueden usarse, OpenClaw inyecta una lista compacta en XML de las habilidades disponibles en el prompt del sistema (mediante `formatSkillsForPrompt` en `pi-coding-agent`). El coste es determinista:

* **Sobrecarga base (solo cuando hay ≥1 habilidad):** 195 caracteres.
* **Por habilidad:** 97 caracteres + la longitud de los valores de `<name>`, `<description>` y `<location>` con caracteres escapados en XML.

Fórmula (caracteres):

```
total = 195 + Σ (97 + len(name_escaped) + len(description_escaped) + len(location_escaped))
```

Notas:

* El escape de XML expande `& < > " '` en entidades (`&amp;`, `&lt;`, etc.), aumentando la longitud.
* El número de tokens varía según el tokenizador del modelo. Una estimación aproximada al estilo de OpenAI es de ~4 caracteres/token, por lo que **97 caracteres ≈ 24 tokens** por skill más la longitud real de tus campos.

<div id="managed-skills-lifecycle">
  ## Ciclo de vida de las habilidades gestionadas
</div>

OpenClaw incluye un conjunto base de habilidades como **habilidades empaquetadas** como parte de la
instalación (paquete de npm u OpenClaw.app). `~/.openclaw/skills` existe para realizar
anulaciones locales (por ejemplo, fijar la versión o aplicar un parche a una habilidad sin modificar la copia empaquetada). Las habilidades del espacio de trabajo pertenecen al usuario y prevalecen sobre las anteriores en caso de conflictos de nombre.

<div id="config-reference">
  ## Referencia de configuración
</div>

Consulta [Configuración de habilidades](/es/tools/skills-config) para ver el esquema completo de configuración.

<div id="looking-for-more-skills">
  ## ¿Buscas más habilidades?
</div>

Visita https://clawhub.com.

***