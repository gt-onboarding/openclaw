---
title: Espacio de trabajo del agente
summary: "Espacio de trabajo del agente: ubicación, estructura y estrategia de copia de seguridad"
read_when:
  - Necesitas explicar el espacio de trabajo del agente o su estructura de archivos
  - Quieres realizar una copia de seguridad o migrar el espacio de trabajo de un agente
---

<div id="agent-workspace">
  # Espacio de trabajo del Agente
</div>

El espacio de trabajo es el hogar del agente. Es el único directorio de trabajo que se usa para
las herramientas de archivos y como contexto del espacio de trabajo. Manténlo privado y trátalo como memoria.

Esto es independiente de `~/.openclaw/`, que almacena configuración, credenciales y
sesiones.

**Importante:** el espacio de trabajo es el **cwd predeterminado**, no una sandbox estricta. Las herramientas
resuelven rutas relativas con respecto al espacio de trabajo, pero las rutas absolutas aún pueden acceder a
otras ubicaciones del host a menos que el sandboxing esté habilitado. Si necesitas aislamiento, usa
[`agents.defaults.sandbox`](/es/gateway/sandboxing) (y/o configuración de sandbox por agente).
Cuando el sandboxing está habilitado y `workspaceAccess` no es `"rw"`, las herramientas operan
dentro de un espacio de trabajo de sandbox bajo `~/.openclaw/sandboxes`, no en el espacio de trabajo de tu host.

<div id="default-location">
  ## Ubicación predeterminada
</div>

* Valor predeterminado: `~/.openclaw/workspace`
* Si `OPENCLAW_PROFILE` está definida y no es `"default"`, la ubicación predeterminada pasa a ser
  `~/.openclaw/workspace-<profile>`.
* Anula este valor en `~/.openclaw/openclaw.json`:

```json5
{
  agent: {
    workspace: "~/.openclaw/workspace"
  }
}
```

`openclaw onboard`, `openclaw configure` o `openclaw setup` crearán el
espacio de trabajo y generarán los archivos de arranque si faltan.

Si ya gestionas tú mismo los archivos del espacio de trabajo, puedes desactivar la
creación de archivos de arranque:

```json5
{ agent: { skipBootstrap: true } }
```

<div id="extra-workspace-folders">
  ## Carpetas adicionales de espacio de trabajo
</div>

Las instalaciones más antiguas pueden haber creado `~/openclaw`. Mantener múltiples
directorios de espacio de trabajo puede provocar problemas de autenticación o
inconsistencias de estado, porque solo un espacio de trabajo está activo a la vez.

**Recomendación:** mantén un único espacio de trabajo activo. Si ya no usas las
carpetas adicionales, archívalas o muévelas a la Papelera (por ejemplo
`trash ~/openclaw`). Si mantienes intencionalmente múltiples espacios de
trabajo, asegúrate de que `agents.defaults.workspace` apunte al activo.

`openclaw doctor` emite una advertencia cuando detecta directorios
adicionales de espacio de trabajo.

<div id="workspace-file-map-what-each-file-means">
  ## Mapa de archivos del espacio de trabajo (qué significa cada archivo)
</div>

Estos son los archivos estándar que OpenClaw espera dentro del espacio de trabajo:

* `AGENTS.md`
  * Instrucciones operativas para el agente y cómo debe usar la memoria.
  * Se carga al inicio de cada sesión.
  * Buen lugar para reglas, prioridades y detalles sobre “cómo comportarse”.

* `SOUL.md`
  * Personalidad, tono y límites.
  * Se carga en cada sesión.

* `USER.md`
  * Quién es el usuario y cómo debes dirigirte a él.
  * Se carga en cada sesión.

* `IDENTITY.md`
  * El nombre, la vibra y los emojis del agente.
  * Se crea/actualiza durante el ritual de bootstrap.

* `TOOLS.md`
  * Notas sobre tus herramientas locales y convenciones.
  * No controla la disponibilidad de herramientas; solo es una guía.

* `HEARTBEAT.md`
  * Lista de comprobación opcional y mínima para ejecuciones de latido.
  * Manténla corta para evitar quemar tokens.

* `BOOT.md`
  * Lista de comprobación de arranque opcional que se ejecuta al reiniciar el Gateway cuando los hooks internos están habilitados.
  * Manténla corta; usa la herramienta `message` para envíos salientes.

* `BOOTSTRAP.md`
  * Ritual de primera ejecución (solo una vez).
  * Solo se crea para un espacio de trabajo completamente nuevo.
  * Elimínalo después de que el ritual se complete.

* `memory/YYYY-MM-DD.md`
  * Registro diario de memoria (un archivo por día).
  * Se recomienda leer los de hoy y ayer al inicio de la sesión.

* `MEMORY.md` (opcional)
  * Memoria seleccionada a largo plazo.
  * Solo se carga en la sesión principal y privada (no en contextos compartidos/de grupo).

Consulta [Memory](/es/concepts/memory) para ver el flujo de trabajo y el vaciado automático de memoria.

* `skills/` (opcional)
  * Habilidades específicas del espacio de trabajo.
  * Anulan las habilidades gestionadas/incluidas cuando los nombres colisionan.

* `canvas/` (opcional)
  * Archivos de UI de canvas para pantallas de nodos (por ejemplo, `canvas/index.html`).

Si falta algún archivo de bootstrap, OpenClaw inyecta un marcador de “archivo ausente” en
la sesión y continúa. Los archivos de bootstrap grandes se truncan cuando se inyectan;
ajusta el límite con `agents.defaults.bootstrapMaxChars` (valor predeterminado: 20000).
`openclaw setup` puede recrear los archivos predeterminados que falten sin sobrescribir los
archivos existentes.

<div id="what-is-not-in-the-workspace">
  ## Lo que NO está en el espacio de trabajo
</div>

Estos se encuentran en `~/.openclaw/` y NO deben incluirse en el repositorio del espacio de trabajo:

* `~/.openclaw/openclaw.json` (configuración)
* `~/.openclaw/credentials/` (tokens OAuth, claves de API)
* `~/.openclaw/agents/<agentId>/sessions/` (transcripciones de sesión + metadatos)
* `~/.openclaw/skills/` (habilidades gestionadas)

Si necesitas migrar sesiones o configuración, cópialas por separado y mantenlas
fuera del control de versiones.

<div id="git-backup-recommended-private">
  ## Copia de seguridad con Git (recomendado, privado)
</div>

Trata el espacio de trabajo como memoria privada. Guárdalo en un repositorio Git **privado** para que tenga copia de seguridad y puedas recuperarlo.

Ejecuta estos pasos en la máquina donde se ejecuta Gateway (es decir, donde reside el espacio de trabajo).

<div id="1-initialize-the-repo">
  ### 1) Inicializar el repositorio
</div>

Si Git está instalado, los espacios de trabajo completamente nuevos se inicializan automáticamente. Si este
espacio de trabajo aún no es un repositorio, ejecuta:

```bash
cd ~/.openclaw/workspace
git init
git add AGENTS.md SOUL.md TOOLS.md IDENTITY.md USER.md HEARTBEAT.md memory/
git commit -m "Añadir espacio de trabajo del agente"
```

<div id="2-add-a-private-remote-beginner-friendly-options">
  ### 2) Añadir un remoto privado (opciones para principiantes)
</div>

Opción A: UI web de GitHub

1. Crea un nuevo repositorio **privado** en GitHub.
2. No lo inicialices con un README (evita conflictos de merge).
3. Copia la URL remota HTTPS.
4. Añade el remoto y haz push:

```bash
git branch -M main
git remote add origin <https-url>
git push -u origin main
```

Opción B: GitHub CLI (`gh`)

```bash
gh auth login
gh repo create openclaw-workspace --private --source . --remote origin --push
```

Opción C: UI web de GitLab

1. Crea un nuevo repositorio **privado** en GitLab.
2. No lo inicialices con un README (para evitar conflictos de merge).
3. Copia la URL remota HTTPS.
4. Añade el remoto y haz push:

```bash
git branch -M main
git remote add origin <https-url>
git push -u origin main
```

<div id="3-ongoing-updates">
  ### 3) Actualizaciones continuas
</div>

```bash
git status
git add .
git commit -m "Actualizar memoria"
git push
```

<div id="do-not-commit-secrets">
  ## No hagas commit de secretos
</div>

Incluso en un repositorio privado, evita almacenar secretos en el espacio de trabajo:

* Claves de API, tokens OAuth, contraseñas o credenciales privadas.
* Cualquier cosa bajo `~/.openclaw/`.
* Volcados sin procesar de chats o adjuntos sensibles.

Si debes almacenar referencias sensibles, usa marcadores de posición y conserva el
secreto real en otro lugar (administrador de contraseñas, variables de entorno o `~/.openclaw/`).

`.gitignore` inicial sugerido:

```gitignore
.DS_Store
.env
**/*.key
**/*.pem
**/secrets*
```

<div id="moving-the-workspace-to-a-new-machine">
  ## Mover el espacio de trabajo a una nueva máquina
</div>

1. Clona el repositorio en la ruta deseada (predeterminada `~/.openclaw/workspace`).
2. Configura `agents.defaults.workspace` con esa ruta en `~/.openclaw/openclaw.json`.
3. Ejecuta `openclaw setup --workspace <path>` para crear los archivos que falten.
4. Si necesitas sesiones, copia `~/.openclaw/agents/<agentId>/sessions/` desde la
   máquina anterior por separado.

<div id="advanced-notes">
  ## Notas avanzadas
</div>

* El enrutamiento multiagente puede utilizar diferentes espacios de trabajo por agente. Consulta
  [Channel routing](/es/concepts/channel-routing) para la configuración de enrutamiento.
* Si `agents.defaults.sandbox` está activado, las sesiones no principales pueden utilizar espacios de trabajo de sandbox por sesión
  ubicados en `agents.defaults.sandbox.workspaceRoot`.