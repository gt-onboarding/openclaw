---
title: Clawhub
summary: "Guía de ClawHub: registro público de habilidades y flujos de trabajo con la CLI"
read_when:
  - Presentar ClawHub a nuevos usuarios
  - Instalar, buscar o publicar habilidades
  - Explicar los flags de la CLI de ClawHub y su comportamiento de sincronización
---

<div id="clawhub">
  # ClawHub
</div>

ClawHub es el **registro público de habilidades para OpenClaw**. Es un servicio gratuito: todas las habilidades son públicas, abiertas y visibles para todo el mundo, para su compartición y reutilización. Una habilidad es solo una carpeta con un archivo `SKILL.md` (más archivos de texto auxiliares). Puedes explorar habilidades en la aplicación web o usar la CLI para buscar, instalar, actualizar y publicar habilidades.

Sitio: [clawhub.com](https://clawhub.com)

<div id="who-this-is-for-beginner-friendly">
  ## Para quién es esto (ideal para principiantes)
</div>

Si quieres añadir nuevas capacidades a tu agente de OpenClaw, ClawHub es la forma más sencilla de encontrar e instalar habilidades. No necesitas saber cómo funciona el backend. Puedes:

* Buscar habilidades con lenguaje natural.
* Instalar una habilidad en tu espacio de trabajo.
* Actualizar habilidades más adelante con un solo comando.
* Hacer una copia de seguridad de tus propias habilidades publicándolas.

<div id="quick-start-non-technical">
  ## Inicio rápido (no técnico)
</div>

1. Instala la CLI (consulta la siguiente sección).
2. Busca algo que necesites:
   * `clawhub search "calendar"`
3. Instala una skill:
   * `clawhub install <skill-slug>`
4. Inicia una nueva sesión de OpenClaw para que cargue la nueva skill.

<div id="install-the-cli">
  ## Instala la CLI
</div>

Elige una opción:

```bash
npm i -g clawhub
```

```bash
pnpm add -g clawhub
```

<div id="how-it-fits-into-openclaw">
  ## Cómo encaja en OpenClaw
</div>

De forma predeterminada, la CLI instala habilidades en `./skills` dentro de tu directorio de trabajo actual. Si hay configurado un espacio de trabajo de OpenClaw, `clawhub` usará ese espacio de trabajo a menos que anules `--workdir` (o `CLAWHUB_WORKDIR`). OpenClaw carga las habilidades del espacio de trabajo desde `<workspace>/skills` y las aplicará en la **siguiente** sesión. Si ya usas `~/.openclaw/skills` o habilidades empaquetadas, las habilidades del espacio de trabajo tienen prioridad.

Para más detalles sobre cómo se cargan, comparten y restringen las habilidades, consulta
[Habilidades](/es/tools/skills).

<div id="what-the-service-provides-features">
  ## Qué ofrece el servicio (funcionalidades)
</div>

* **Exploración pública** de habilidades y su contenido `SKILL.md`.
* **Búsqueda** basada en embeddings (búsqueda vectorial), no solo por palabras clave.
* **Control de versiones** con semver, registros de cambios (changelogs) y etiquetas (incluida `latest`).
* **Descargas** como archivo zip por versión.
* **Estrellas y comentarios** para las opiniones de la comunidad.
* Hooks de **moderación** para aprobaciones y auditorías.
* **API compatible con la CLI** para automatización y scripting.

<div id="cli-commands-and-parameters">
  ## Comandos y parámetros de la CLI
</div>

Opciones globales (se aplican a todos los comandos):

* `--workdir <dir>`: Directorio de trabajo (predeterminado: directorio actual; si no, espacio de trabajo de OpenClaw).
* `--dir <dir>`: Directorio de habilidades, relativo a workdir (predeterminado: `skills`).
* `--site <url>`: URL base del sitio (inicio de sesión en el navegador).
* `--registry <url>`: URL base de la API del registro.
* `--no-input`: Deshabilitar solicitudes/preguntas (modo no interactivo).
* `-V, --cli-version`: Mostrar la versión de la CLI.

Autenticación:

* `clawhub login` (flujo por navegador) o `clawhub login --token <token>`
* `clawhub logout`
* `clawhub whoami`

Opciones:

* `--token <token>`: Pegar un token de API.
* `--label <label>`: Etiqueta almacenada para tokens de inicio de sesión por navegador (predeterminado: `CLI token`).
* `--no-browser`: No abrir un navegador (requiere `--token`).

Búsqueda:

* `clawhub search "query"`
* `--limit <n>`: Número máximo de resultados.

Instalación:

* `clawhub install <slug>`
* `--version <version>`: Instalar una versión específica.
* `--force`: Sobrescribir si la carpeta ya existe.

Actualización:

* `clawhub update <slug>`
* `clawhub update --all`
* `--version <version>`: Actualizar a una versión específica (solo un slug).
* `--force`: Sobrescribir cuando los archivos locales no coinciden con ninguna versión publicada.

Listado:

* `clawhub list` (lee `.clawhub/lock.json`)

Publicación:

* `clawhub publish <path>`
* `--slug <slug>`: Slug de la habilidad.
* `--name <name>`: Nombre visible.
* `--version <version>`: Versión semántica (semver).
* `--changelog <text>`: Texto del registro de cambios (puede estar vacío).
* `--tags <tags>`: Etiquetas separadas por comas (predeterminado: `latest`).

Eliminar/recuperar (solo propietario/administrador):

* `clawhub delete <slug> --yes`
* `clawhub undelete <slug> --yes`

Sync (escanea habilidades locales y publica las nuevas/actualizadas):

* `clawhub sync`
* `--root <dir...>`: Directorios raíz adicionales para el escaneo.
* `--all`: Cargar todo sin solicitudes/preguntas.
* `--dry-run`: Mostrar qué se cargaría.
* `--bump <type>`: `patch|minor|major` para actualizaciones (predeterminado: `patch`).
* `--changelog <text>`: Registro de cambios para actualizaciones no interactivas.
* `--tags <tags>`: Etiquetas separadas por comas (predeterminado: `latest`).
* `--concurrency <n>`: Comprobaciones contra el registro (predeterminado: 4).

<div id="common-workflows-for-agents">
  ## Flujos de trabajo habituales para agentes
</div>

<div id="search-for-skills">
  ### Buscar habilidades
</div>

```bash
clawhub search "postgres backups"
```

<div id="download-new-skills">
  ### Descargar nuevas habilidades
</div>

```bash
clawhub install my-skill-pack
```

<div id="update-installed-skills">
  ### Actualizar habilidades instaladas
</div>

```bash
clawhub update --all
```

<div id="back-up-your-skills-publish-or-sync">
  ### Realiza una copia de seguridad de tus habilidades (publica o sincroniza)
</div>

Para una única carpeta de habilidad:

```bash
clawhub publish ./my-skill --slug my-skill --name "My Skill" --version 1.0.0 --tags latest
```

Para escanear y respaldar muchas habilidades a la vez:

```bash
clawhub sync --all
```

<div id="advanced-details-technical">
  ## Detalles técnicos avanzados
</div>

<div id="versioning-and-tags">
  ### Versiones y etiquetas
</div>

* Cada publicación crea una nueva `SkillVersion` con **semver**.
* Las etiquetas (como `latest`) apuntan a una versión; mover etiquetas te permite revertir cambios.
* Los registros de cambios (changelogs) se adjuntan por versión y pueden estar vacíos al sincronizar o publicar actualizaciones.

<div id="local-changes-vs-registry-versions">
  ### Cambios locales vs versiones del registro
</div>

Las actualizaciones comparan el contenido de las skills locales con las versiones del registro mediante un hash de contenido. Si los archivos locales no coinciden con ninguna versión publicada, la CLI te pedirá confirmación antes de sobrescribir (o requerirá `--force` en ejecuciones no interactivas).

<div id="sync-scanning-and-fallback-roots">
  ### Exploración de sincronización y raíces de respaldo
</div>

`clawhub sync` analiza primero tu directorio de trabajo actual. Si no se encuentran habilidades, recurre a rutas heredadas conocidas (por ejemplo, `~/openclaw/skills` y `~/.openclaw/skills`). Esto está diseñado para encontrar instalaciones antiguas de habilidades sin flags adicionales.

<div id="storage-and-lockfile">
  ### Almacenamiento y archivo de bloqueo
</div>

* Las habilidades instaladas se registran en `.clawhub/lock.json` dentro de tu directorio de trabajo.
* Los tokens de autenticación se almacenan en el archivo de configuración de la CLI de ClawHub (se puede sobrescribir mediante `CLAWHUB_CONFIG_PATH`).

<div id="telemetry-install-counts">
  ### Telemetría (conteo de instalaciones)
</div>

Cuando ejecutas `clawhub sync` mientras tienes la sesión iniciada, la CLI envía una instantánea mínima para calcular el conteo de instalaciones. Puedes desactivar esto por completo:

```bash
export CLAWHUB_DISABLE_TELEMETRY=1
```

<div id="environment-variables">
  ## Variables de entorno
</div>

* `CLAWHUB_SITE`: Sobrescribe la URL del sitio.
* `CLAWHUB_REGISTRY`: Sobrescribe la URL de la API del registro.
* `CLAWHUB_CONFIG_PATH`: Sobrescribe dónde la CLI almacena el token y la configuración.
* `CLAWHUB_WORKDIR`: Sobrescribe el directorio de trabajo predeterminado.
* `CLAWHUB_DISABLE_TELEMETRY=1`: Desactiva la telemetría durante `sync`.