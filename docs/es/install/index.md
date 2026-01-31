---
title: Instalar
summary: "Instala OpenClaw (instalador recomendado, instalación global o desde el código fuente)"
read_when:
  - Estás instalando OpenClaw
  - Quieres instalarlo desde GitHub
---

<div id="install">
  # Instalación
</div>

Usa el instalador a menos que tengas una buena razón para no hacerlo. Configura la CLI y ejecuta el asistente de incorporación.

<div id="quick-install-recommended">
  ## Instalación rápida (recomendada)
</div>

```bash
curl -fsSL https://openclaw.bot/install.sh | bash
```

Windows (PowerShell):

```powershell
iwr -useb https://openclaw.ai/install.ps1 | iex
```

Siguiente paso (si omitiste el asistente de configuración):

```bash
openclaw onboard --install-daemon
```

<div id="system-requirements">
  ## Requisitos del sistema
</div>

* **Node &gt;=22**
* macOS, Linux o Windows mediante WSL2
* `pnpm` solo si compilas desde el código fuente

<div id="choose-your-install-path">
  ## Elige tu método de instalación
</div>

<div id="1-installer-script-recommended">
  ### 1) Script de instalación (recomendado)
</div>

Instala `openclaw` de forma global mediante npm y ejecuta el asistente de configuración inicial.

```bash
curl -fsSL https://openclaw.bot/install.sh | bash
```

Opciones del instalador:

```bash
curl -fsSL https://openclaw.bot/install.sh | bash -s -- --help
```

Detalles: [Aspectos internos del instalador](/es/install/installer).

Modo no interactivo (omitir el onboarding inicial):

```bash
curl -fsSL https://openclaw.bot/install.sh | bash -s -- --no-onboard
```

<div id="2-global-install-manual">
  ### 2) Instalación global (manual)
</div>

Si ya tienes Node.js:

```bash
npm install -g openclaw@latest
```

Si tienes libvips instalado a nivel del sistema (habitual en macOS mediante Homebrew) y `sharp` no se instala correctamente, fuerza el uso de binarios precompilados:

```bash
SHARP_IGNORE_GLOBAL_LIBVIPS=1 npm install -g openclaw@latest
```

Si ves `sharp: Please add node-gyp to your dependencies`, instala las herramientas de compilación (macOS: Xcode CLT + `npm install -g node-gyp`) o usa la solución alternativa `SHARP_IGNORE_GLOBAL_LIBVIPS=1` mencionada arriba para omitir la compilación nativa.

O:

```bash
pnpm add -g openclaw@latest
```

A continuación:

```bash
openclaw onboard --install-daemon
```

<div id="3-from-source-contributorsdev">
  ### 3) Desde el código fuente (colaboradores/desarrolladores)
</div>

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm ui:build # instala automáticamente las dependencias de la UI en la primera ejecución
pnpm build
openclaw onboard --install-daemon
```

Consejo: si todavía no tienes una instalación global de openclaw, ejecuta los comandos del repositorio con `pnpm openclaw ...`.

<div id="4-other-install-options">
  ### 4) Otras opciones de instalación
</div>

* Docker: [Docker](/es/install/docker)
* Nix: [Nix](/es/install/nix)
* Ansible: [Ansible](/es/install/ansible)
* Bun (solo CLI): [Bun](/es/install/bun)

<div id="after-install">
  ## Después de instalar
</div>

* Ejecuta el onboarding: `openclaw onboard --install-daemon`
* Comprobación rápida: `openclaw doctor`
* Comprueba el estado del Gateway: `openclaw status` + `openclaw health`
* Abre el panel de control: `openclaw dashboard`

<div id="install-method-npm-vs-git-installer">
  ## Método de instalación: npm vs git (instalador)
</div>

El instalador ofrece dos métodos:

* `npm` (predeterminado): `npm install -g openclaw@latest`
* `git`: clonar/compilar desde GitHub y ejecutar desde un checkout del repositorio

<div id="cli-flags">
  ### Opciones de la CLI
</div>

```bash
# npm explícito
curl -fsSL https://openclaw.bot/install.sh | bash -s -- --install-method npm

# Instalar desde GitHub (checkout del código fuente)
curl -fsSL https://openclaw.bot/install.sh | bash -s -- --install-method git
```

Opciones comunes:

* `--install-method npm|git`
* `--git-dir <path>` (valor predeterminado: `~/openclaw`)
* `--no-git-update` (omite `git pull` cuando se usa un clon/copia de trabajo existente)
* `--no-prompt` (desactiva los mensajes interactivos; obligatorio en CI/automatización)
* `--dry-run` (muestra lo que ocurriría; no realiza cambios)
* `--no-onboard` (omite el proceso de configuración inicial)

<div id="environment-variables">
  ### Variables de entorno
</div>

Variables de entorno equivalentes (útiles para la automatización):

* `OPENCLAW_INSTALL_METHOD=git|npm`
* `OPENCLAW_GIT_DIR=...`
* `OPENCLAW_GIT_UPDATE=0|1`
* `OPENCLAW_NO_PROMPT=1`
* `OPENCLAW_DRY_RUN=1`
* `OPENCLAW_NO_ONBOARD=1`
* `SHARP_IGNORE_GLOBAL_LIBVIPS=0|1` (valor predeterminado: `1`; evita que `sharp` se compile contra la libvips del sistema)

<div id="troubleshooting-openclaw-not-found-path">
  ## Solución de problemas: `openclaw` no se encuentra en el PATH
</div>

Diagnóstico rápido:

```bash
node -v
npm -v
npm prefix -g
echo "$PATH"
```

Si `$(npm prefix -g)/bin` (macOS/Linux) o `$(npm prefix -g)` (Windows) **no** aparece en la salida de `echo "$PATH"`, tu shell no puede encontrar los binarios globales de npm (incluido `openclaw`).

Solución: añádelo a tu archivo de inicio del shell (zsh: `~/.zshrc`, bash: `~/.bashrc`):

```bash
# macOS / Linux
export PATH="$(npm prefix -g)/bin:$PATH"
```

En Windows, añade la salida de `npm prefix -g` al PATH de tu sistema.

Luego abre una nueva terminal (o ejecuta `rehash` en zsh / `hash -r` en bash).

<div id="update-uninstall">
  ## Actualizar / desinstalar
</div>

* Actualizaciones: [Actualización](/es/install/updating)
* Migrar a una nueva máquina: [Migración](/es/install/migrating)
* Desinstalar: [Desinstalación](/es/install/uninstall)