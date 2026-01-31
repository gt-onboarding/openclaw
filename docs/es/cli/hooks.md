---
title: Hooks
summary: "Referencia de la CLI para `openclaw hooks` (hooks de agentes)"
read_when:
  - Quieres administrar hooks de agentes
  - Quieres instalar o actualizar hooks
---

<div id="openclaw-hooks">
  # `openclaw hooks`
</div>

Administra hooks de agente (automatizaciones basadas en eventos para comandos como `/new`, `/reset` y el inicio del Gateway).

Relacionado:

* Hooks: [Hooks](/es/hooks)
* Hooks de complementos: [Plugins](/es/plugin#plugin-hooks)

<div id="list-all-hooks">
  ## Mostrar todos los hooks
</div>

```bash
openclaw hooks list
```

Enumera todos los hooks encontrados en los directorios del espacio de trabajo, administrados y empaquetados.

**Opciones:**

* `--eligible`: Muestra solo los hooks elegibles (requisitos cumplidos)
* `--json`: Muestra la salida en formato JSON
* `-v, --verbose`: Muestra información detallada, incluidos los requisitos que faltan

**Ejemplo de salida:**

```
Hooks (4/4 listos)

Listos:
  🚀 boot-md ✓ - Ejecutar BOOT.md al iniciar la puerta de enlace
  📝 command-logger ✓ - Registrar todos los eventos de comandos en un archivo de auditoría centralizado
  💾 session-memory ✓ - Guardar el contexto de la sesión en memoria cuando se emite el comando /new
  😈 soul-evil ✓ - Intercambiar contenido SOUL inyectado durante una ventana de purga o por casualidad
```

**Ejemplo (modo detallado):**

```bash
openclaw hooks list --verbose
```

Muestra los requisitos que faltan en los hooks no aptos.

**Ejemplo (JSON):**

```bash
openclaw hooks list --json
```

Devuelve JSON estructurado para su uso desde código.

<div id="get-hook-information">
  ## Consultar información del hook
</div>

```bash
openclaw hooks info <name>
```

Muestra información detallada sobre un hook específico.

**Argumentos:**

* `<name>`: Nombre del hook (por ejemplo, `session-memory`)

**Opciones:**

* `--json`: Muestra la salida en formato JSON

**Ejemplo:**

```bash
openclaw hooks info session-memory
```

**Salida:**

```
💾 session-memory ✓ Ready

Save session context to memory when /new command is issued

Details:
  Source: openclaw-bundled
  Path: /path/to/openclaw/hooks/bundled/session-memory/HOOK.md
  Handler: /path/to/openclaw/hooks/bundled/session-memory/handler.ts
  Homepage: https://docs.openclaw.ai/hooks#session-memory
  Events: command:new

Requirements:
  Config: ✓ workspace.dir
```

<div id="check-hooks-eligibility">
  ## Comprobar si los hooks cumplen los requisitos
</div>

```bash
openclaw hooks check
```

Mostrar un resumen del estado de elegibilidad de los hooks (cuántos están listos y cuántos no).

**Opciones:**

* `--json`: Salida como JSON

**Ejemplo de salida:**

```
Hooks Status

Total hooks: 4
Ready: 4
Not ready: 0
```

<div id="enable-a-hook">
  ## Habilitar un Hook
</div>

```bash
openclaw hooks enable <name>
```

Habilita un hook específico añadiéndolo a tu archivo de configuración (`~/.openclaw/config.json`).

**Nota:** Los hooks administrados por complementos muestran `plugin:<id>` en `openclaw hooks list` y
no se pueden habilitar o deshabilitar aquí. En su lugar, habilita o deshabilita el complemento.

**Argumentos:**

* `<name>`: Nombre del hook (por ejemplo, `session-memory`)

**Ejemplo:**

```bash
openclaw hooks enable session-memory
```

**Salida:**

```
✓ Hook habilitado: 💾 session-memory
```

**Qué hace:**

* Comprueba si el hook existe y es válido
* Actualiza `hooks.internal.entries.<name>.enabled = true` en tu configuración
* Guarda la configuración en disco

**Después de habilitarlo:**

* Reinicia el Gateway para que se recarguen los hooks (reinicia la app de la barra de menú en macOS o reinicia tu proceso de Gateway en desarrollo).

<div id="disable-a-hook">
  ## Desactivar un hook
</div>

```bash
openclaw hooks disable <name>
```

Desactiva un hook específico actualizando la configuración.

**Argumentos:**

* `<name>`: Nombre del hook (por ejemplo, `command-logger`)

**Ejemplo:**

```bash
openclaw hooks disable command-logger
```

**Salida:**

```
⏸ Disabled hook: 📝 command-logger
```

**Después de desactivar:**

* Reinicia el Gateway para que los hooks se vuelvan a cargar

<div id="install-hooks">
  ## Instalar hooks
</div>

```bash
openclaw hooks install <path-or-spec>
```

Instala un paquete de hooks desde una carpeta o archivo local, o desde npm.

**Qué hace:**

* Copia el paquete de hooks en `~/.openclaw/hooks/<id>`
* Habilita los hooks instalados en `hooks.internal.entries.*`
* Registra la instalación en `hooks.internal.installs`

**Opciones:**

* `-l, --link`: Vincula un directorio local en lugar de copiarlo (lo añade a `hooks.internal.load.extraDirs`)

**Formatos de archivo compatibles:** `.zip`, `.tgz`, `.tar.gz`, `.tar`

**Ejemplos:**

```bash
# Local directory
openclaw hooks install ./my-hook-pack

# Local archive
openclaw hooks install ./my-hook-pack.zip

# NPM package
openclaw hooks install @openclaw/my-hook-pack

# Enlazar un directorio local sin copiar
openclaw hooks install -l ./my-hook-pack
```

<div id="update-hooks">
  ## Hooks de actualización
</div>

```bash
openclaw hooks update <id>
openclaw hooks update --all
```

Actualiza los paquetes de hooks instalados (solo instalaciones mediante npm).

**Opciones:**

* `--all`: Actualiza todos los paquetes de hooks controlados
* `--dry-run`: Muestra qué cambiaría sin realizar cambios

<div id="bundled-hooks">
  ## Hooks integrados
</div>

<div id="session-memory">
  ### session-memory
</div>

Guarda el contexto de la sesión en memoria al ejecutar `/new`.

**Habilitar:**

```bash
openclaw hooks enable session-memory
```

**Salida:** `~/.openclaw/workspace/memory/YYYY-MM-DD-slug.md`

**Consulta:** [documentación de session-memory](/es/hooks#session-memory)

<div id="command-logger">
  ### command-logger
</div>

Registra todos los eventos de comandos en un archivo de auditoría centralizado.

**Habilitar:**

```bash
openclaw hooks enable command-logger
```

**Salida:** `~/.openclaw/logs/commands.log`

**Ver logs:**

```bash
# Comandos recientes
tail -n 20 ~/.openclaw/logs/commands.log

# Formato legible
cat ~/.openclaw/logs/commands.log | jq .

# Filtrar por acción
grep '"action":"new"' ~/.openclaw/logs/commands.log | jq .
```

**Consulta:** [la documentación de command-logger](/es/hooks#command-logger)

<div id="soul-evil">
  ### soul-evil
</div>

Sustituye el contenido inyectado de `SOUL.md` por el de `SOUL_EVIL.md` durante una ventana de purga o de forma aleatoria.

**Activar:**

```bash
openclaw hooks enable soul-evil
```

**Consulta:** [SOUL Evil Hook](/es/hooks/soul-evil)

<div id="boot-md">
  ### boot-md
</div>

Ejecuta `BOOT.md` cuando arranca el Gateway (después de iniciar los canales).

**Eventos**: `gateway:startup`

**Habilitar**:

```bash
openclaw hooks enable boot-md
```

**Consulta:** la [documentación de boot-md](/es/hooks#boot-md)
