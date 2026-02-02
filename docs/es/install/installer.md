---
title: Instalador
summary: "Cómo funcionan los scripts de instalación (install.sh + install-cli.sh), flags y automatización"
read_when:
  - Quieres entender `openclaw.bot/install.sh`
  - Quieres automatizar instalaciones (CI / modo no interactivo)
  - Quieres instalar desde un checkout de GitHub
---

<div id="installer-internals">
  # Detalles internos del instalador
</div>

OpenClaw incluye dos scripts de instalación (distribuidos desde `openclaw.ai`):

* `https://openclaw.bot/install.sh` — instalador “recomendado” (instalación global con npm por defecto; también puede instalar desde un checkout de GitHub)
* `https://openclaw.bot/install-cli.sh` — instalador de CLI apto para entornos sin root (instala en un prefijo con su propio Node)
* `https://openclaw.ai/install.ps1` — instalador de Windows PowerShell (npm por defecto; instalación opcional con git)

Para ver las flags y el comportamiento actuales, ejecuta:

```bash
curl -fsSL https://openclaw.bot/install.sh | bash -s -- --help
```

Ayuda para Windows (PowerShell):

```powershell
& ([scriptblock]::Create((iwr -useb https://openclaw.ai/install.ps1))) -?
```

Si el instalador finaliza pero `openclaw` no se encuentra en una nueva terminal, normalmente se trata de un problema con la variable PATH de Node.js/npm. Consulta: [Install](/es/install#nodejs--npm-path-sanity).

<div id="installsh-recommended">
  ## install.sh (recomendado)
</div>

Qué hace (a grandes rasgos):

* Detecta el sistema operativo (macOS / Linux / WSL).
* Garantiza Node.js **22+** (macOS vía Homebrew; Linux vía NodeSource).
* Elige el método de instalación:
  * `npm` (predeterminado): `npm install -g openclaw@latest`
  * `git`: clona/compila un checkout del código fuente e instala un script envoltorio
* En Linux: evita errores de permisos con instalaciones globales de npm cambiando el prefijo de npm a `~/.npm-global` cuando sea necesario.
* Si estás actualizando una instalación existente: ejecuta `openclaw doctor --non-interactive` (en la medida de lo posible).
* Para instalaciones con git: ejecuta `openclaw doctor --non-interactive` después de la instalación/actualización (en la medida de lo posible).
* Mitiga problemas con la instalación nativa de `sharp` estableciendo por defecto `SHARP_IGNORE_GLOBAL_LIBVIPS=1` (evita compilar contra la libvips del sistema).

Si *quieres* que `sharp` se vincule con una libvips instalada globalmente (o estás depurando), establece:

```bash
SHARP_IGNORE_GLOBAL_LIBVIPS=0 curl -fsSL https://openclaw.bot/install.sh | bash
```

<div id="discoverability-git-install-prompt">
  ### Descubrimiento / aviso de «git install»
</div>

Si ejecutas el instalador **ya dentro de un checkout del código fuente de OpenClaw** (detectado mediante `package.json` + `pnpm-workspace.yaml`), mostrará:

* actualizar y usar este checkout (`git`)
* o migrar a la instalación global con npm (`npm`)

En contextos no interactivos (sin TTY / `--no-prompt`), debes pasar `--install-method git|npm` (o establecer `OPENCLAW_INSTALL_METHOD`); de lo contrario, el script finaliza con el código de salida `2`.

<div id="why-git-is-needed">
  ### Por qué se necesita Git
</div>

Git es obligatorio para la opción `--install-method git` (clone / pull).

Para instalaciones con `npm`, Git *normalmente* no es obligatorio, pero en algunos entornos termina siendo necesario (p. ej., cuando un paquete o dependencia se obtiene mediante una URL de git). El instalador actualmente garantiza que Git esté presente para evitar errores inesperados del tipo `spawn git ENOENT` en distribuciones recién instaladas.

<div id="why-npm-hits-eacces-on-fresh-linux">
  ### Por qué npm devuelve `EACCES` en un Linux recién instalado
</div>

En algunas configuraciones de Linux (especialmente después de instalar Node mediante el gestor de paquetes del sistema o NodeSource), el prefijo global de npm apunta a una ubicación propiedad de root. En consecuencia, `npm install -g ...` falla con errores de permisos `EACCES` / `mkdir`.

`install.sh` mitiga esto cambiando el prefijo a:

* `~/.npm-global` (y añadiéndolo a `PATH` en `~/.bashrc` / `~/.zshrc` cuando existen)

<div id="install-clish-non-root-cli-installer">
  ## install-cli.sh (instalador de la CLI sin privilegios de root)
</div>

Este script instala `openclaw` en un prefijo (predeterminado: `~/.openclaw`) y también instala un entorno de ejecución de Node dedicado bajo ese prefijo, para que pueda funcionar en máquinas donde no quieras modificar el Node/npm del sistema.

Ayuda:

```bash
curl -fsSL https://openclaw.bot/install-cli.sh | bash -s -- --help
```

<div id="installps1-windows-powershell">
  ## install.ps1 (Windows PowerShell)
</div>

Qué hace (a grandes rasgos):

* Comprueba que haya Node.js **22+** (winget/Chocolatey/Scoop o instalación manual).
* Elige el método de instalación:
  * `npm` (predeterminado): `npm install -g openclaw@latest`
  * `git`: clona/compila un checkout del código fuente e instala un script wrapper
* Ejecuta `openclaw doctor --non-interactive` en actualizaciones e instalaciones con git (en la medida de lo posible).

Ejemplos:

```powershell
iwr -useb https://openclaw.ai/install.ps1 | iex
```

```powershell
iwr -useb https://openclaw.ai/install.ps1 | iex -InstallMethod git
```

```powershell
iwr -useb https://openclaw.ai/install.ps1 | iex -InstallMethod git -GitDir "C:\\openclaw"
```

Variables de entorno:

* `OPENCLAW_INSTALL_METHOD=git|npm`
* `OPENCLAW_GIT_DIR=...`

Requisito de Git:

Si seleccionas `-InstallMethod git` y Git no está instalado, el instalador mostrará el enlace de
Git for Windows (`https://git-scm.com/download/win`) y se cerrará.

Problemas comunes en Windows:

* **npm error spawn git / ENOENT**: instala Git for Windows y vuelve a abrir PowerShell, luego vuelve a ejecutar el instalador.
* **&quot;openclaw&quot; is not recognized**: tu carpeta global de npm no está en `PATH`. La mayoría de los sistemas usan
  `%AppData%\\npm`. También puedes ejecutar `npm config get prefix` y agregar `\\bin` a `PATH`, luego volver a abrir PowerShell.
