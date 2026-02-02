---
title: "Node.js + npm (comprobación de PATH)"
summary: "Instalación correcta de Node.js + npm: versiones, PATH e instalaciones globales"
read_when:
  - "Instalaste OpenClaw pero `openclaw` da “command not found”"
  - "Estás configurando Node.js/npm en una máquina nueva"
  - "npm install -g ... falla por problemas de permisos o de PATH"
---

<div id="nodejs-npm-path-sanity">
  # Node.js + npm (verificación de PATH)
</div>

El requisito mínimo de ejecución de OpenClaw es **Node 22+**.

Si puedes ejecutar `npm install -g openclaw@latest` pero luego ves `openclaw: command not found`, casi siempre es un problema de **PATH**: el directorio donde npm coloca los binarios globales no está en el PATH de tu shell.

<div id="quick-diagnosis">
  ## Diagnóstico rápido
</div>

Ejecuta:

```bash
node -v
npm -v
npm prefix -g
echo "$PATH"
```

Si `$(npm prefix -g)/bin` (macOS/Linux) o `$(npm prefix -g)` (Windows) **no** aparece en la salida de `echo "$PATH"`, tu shell no puede encontrar los binarios globales de npm (incluido `openclaw`).


<div id="fix-put-npms-global-bin-dir-on-path">
  ## Solución: incluye el directorio `bin` global de npm en el PATH
</div>

1. Busca tu prefijo global de npm:

```bash
npm prefix -g
```

2. Añade el directorio global bin de npm al archivo de inicio de tu shell:

* zsh: `~/.zshrc`
* bash: `~/.bashrc`

Ejemplo (reemplaza la ruta con el resultado de tu `npm prefix -g`):

```bash
# macOS / Linux
export PATH="/path/from/npm/prefix/bin:$PATH"
```

Luego, abre una **nueva terminal** (o ejecuta `rehash` en zsh / `hash -r` en bash).

En Windows, agrega el resultado de ejecutar `npm prefix -g` a tu PATH.


<div id="fix-avoid-sudo-npm-install-g-permission-errors-linux">
  ## Solución: evita usar `sudo npm install -g` / errores de permisos (Linux)
</div>

Si `npm install -g ...` falla con `EACCES`, cambia el prefijo global de npm a un directorio con permisos de escritura para tu usuario:

```bash
mkdir -p "$HOME/.npm-global"
npm config set prefix "$HOME/.npm-global"
export PATH="$HOME/.npm-global/bin:$PATH"
```

Añade la línea `export PATH=...` al archivo de inicio de tu shell.


<div id="recommended-node-install-options">
  ## Opciones recomendadas de instalación de Node
</div>

Tendrás menos sorpresas si Node/npm están instalados de una manera que:

- mantenga Node actualizado (22+)
- haga que el directorio `bin` global de npm sea estable y esté en el PATH en nuevas sesiones de shell

Opciones comunes:

- macOS: Homebrew (`brew install node`) o un gestor de versiones
- Linux: tu gestor de versiones preferido, o una instalación admitida por la distribución que proporcione Node 22+
- Windows: instalador oficial de Node, `winget`, o un gestor de versiones de Node para Windows

Si usas un gestor de versiones (nvm/fnm/asdf/etc.), asegúrate de que se inicialice en la shell que utilizas en el día a día (zsh vs bash) para que el PATH que configura esté presente cuando ejecutes instaladores.