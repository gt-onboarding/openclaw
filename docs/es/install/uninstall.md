---
title: Desinstalar
summary: "Desinstalar OpenClaw completamente (CLI, servicio, estado, espacio de trabajo)"
read_when:
  - Quieres eliminar OpenClaw de una máquina
  - El servicio Gateway sigue ejecutándose después de la desinstalación
---

<div id="uninstall">
  # Desinstalación
</div>

Dos opciones:

- **Opción sencilla** si `openclaw` sigue instalado.
- **Eliminación manual del servicio** si la CLI ya no está instalada pero el servicio sigue en ejecución.

<div id="easy-path-cli-still-installed">
  ## Opción sencilla (CLI aún instalada)
</div>

Se recomienda usar el desinstalador integrado:

```bash
openclaw uninstall
```

Modo no interactivo (automatización / npx):

```bash
openclaw uninstall --all --yes --non-interactive
npx -y openclaw uninstall --all --yes --non-interactive
```

Pasos manuales (mismo resultado):

1. Detén el servicio Gateway:

```bash
openclaw gateway stop
```

2. Desinstala el servicio de Gateway (launchd/systemd/schtasks):

```bash
openclaw gateway uninstall
```

3. Eliminar el estado y la configuración:

```bash
rm -rf "${OPENCLAW_STATE_DIR:-$HOME/.openclaw}"
```

Si configuraste `OPENCLAW_CONFIG_PATH` en una ubicación personalizada fuera del directorio de estado, elimina también ese archivo.

4. Elimina tu espacio de trabajo (opcional, elimina archivos de los agentes):

```bash
rm -rf ~/.openclaw/workspace
```

5. Desinstala la CLI (elige el método que usaste):

```bash
npm rm -g openclaw
pnpm remove -g openclaw
bun remove -g openclaw
```

6. Si has instalado la aplicación para macOS:

```bash
rm -rf /Applications/OpenClaw.app
```

Notas:

* Si utilizaste perfiles (`--profile` / `OPENCLAW_PROFILE`), repite el paso 3 para cada directorio de estado (de forma predeterminada, `~/.openclaw-<profile>`).
* En modo remoto, el directorio de estado reside en el **host del Gateway**, así que ejecuta allí también los pasos 1‑4.


<div id="manual-service-removal-cli-not-installed">
  ## Eliminación manual del servicio (CLI no instalada)
</div>

Usa esto si el servicio del Gateway sigue ejecutándose pero `openclaw` no está instalado.

<div id="macos-launchd">
  ### macOS (launchd)
</div>

La etiqueta predeterminada es `bot.molt.gateway` (o `bot.molt.<profile>`; es posible que la heredada `com.openclaw.*` siga existiendo):

```bash
launchctl bootout gui/$UID/bot.molt.gateway
rm -f ~/Library/LaunchAgents/bot.molt.gateway.plist
```

Si utilizaste un perfil, reemplaza la etiqueta y el nombre del plist por `bot.molt.<profile>`. Elimina cualquier plist heredado `com.openclaw.*` que pueda existir.


<div id="linux-systemd-user-unit">
  ### Linux (unidad de usuario de systemd)
</div>

El nombre predeterminado de la unidad es `openclaw-gateway.service` (o `openclaw-gateway-<profile>.service`):

```bash
systemctl --user disable --now openclaw-gateway.service
rm -f ~/.config/systemd/user/openclaw-gateway.service
systemctl --user daemon-reload
```


<div id="windows-scheduled-task">
  ### Windows (tarea programada)
</div>

El nombre predeterminado de la tarea es `OpenClaw Gateway` (o `OpenClaw Gateway (<profile>)`).
El script de la tarea se encuentra en tu directorio de estado.

```powershell
schtasks /Delete /F /TN "OpenClaw Gateway"
Remove-Item -Force "$env:USERPROFILE\.openclaw\gateway.cmd"
```

Si usaste un perfil, elimina el nombre de la tarea correspondiente y `~\.openclaw-<profile>\gateway.cmd`.


<div id="normal-install-vs-source-checkout">
  ## Instalación normal vs. checkout del código fuente
</div>

<div id="normal-install-installsh-npm-pnpm-bun">
  ### Instalación normal (install.sh / npm / pnpm / bun)
</div>

Si utilizaste `https://openclaw.bot/install.sh` o `install.ps1`, la CLI se instaló con `npm install -g openclaw@latest`.
Desinstálalo con `npm rm -g openclaw` (o `pnpm remove -g` / `bun remove -g` si lo instalaste de ese modo).

<div id="source-checkout-git-clone">
  ### Checkout desde el código fuente (git clone)
</div>

Si ejecutas desde un checkout del repositorio (`git clone` + `openclaw ...` / `bun run openclaw ...`):

1) Desinstala el servicio Gateway **antes** de eliminar el repositorio (usa el método sencillo anterior o la eliminación manual del servicio).
2) Elimina el directorio del repositorio.
3) Elimina el estado + el espacio de trabajo como se muestra arriba.