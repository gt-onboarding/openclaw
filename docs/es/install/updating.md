---
title: Actualización
summary: "Actualizar OpenClaw de forma segura (instalación global o desde el código fuente) y estrategia de reversión"
read_when:
  - Actualizar OpenClaw
  - Algo falla después de una actualización
---

<div id="updating">
  # Actualización
</div>

OpenClaw evoluciona rápido (antes de la versión “1.0”). Trata las actualizaciones como despliegues de infraestructura: actualiza → ejecuta comprobaciones → reinicia (o usa `openclaw update`, que reinicia) → verifica.

<div id="recommended-re-run-the-website-installer-upgrade-in-place">
  ## Recomendado: volver a ejecutar el instalador del sitio web (actualización sobre la instalación existente)
</div>

La ruta de actualización **preferida** es volver a ejecutar el instalador desde el sitio web. Detecta instalaciones existentes, las actualiza sobre la instalación actual y ejecuta `openclaw doctor` cuando es necesario.

```bash
curl -fsSL https://openclaw.bot/install.sh | bash
```

Notas:

* Añade `--no-onboard` si no quieres que el asistente de incorporación vuelva a ejecutarse.
* Para **instalaciones desde código fuente**, usa:
  ```bash
  curl -fsSL https://openclaw.bot/install.sh | bash -s -- --install-method git --no-onboard
  ```
  El instalador ejecutará `git pull --rebase` **solo** si el repositorio está limpio.
* Para **instalaciones globales**, el script usa `npm install -g openclaw@latest` internamente.
* Nota sobre compatibilidad: `openclaw` sigue disponible como shim de compatibilidad.

<div id="before-you-update">
  ## Antes de actualizar
</div>

* Asegúrate de saber cómo lo instalaste: **global** (npm/pnpm) vs **desde código fuente** (`git clone`).
* Asegúrate de saber cómo se está ejecutando tu Gateway: **en primer plano en la terminal** vs **como servicio supervisado** (launchd/systemd).
* Haz una copia de tu configuración personalizada:
  * Configuración: `~/.openclaw/openclaw.json`
  * Credenciales: `~/.openclaw/credentials/`
  * Espacio de trabajo: `~/.openclaw/workspace`

<div id="update-global-install">
  ## Actualización (instalación global)
</div>

Instalación global (elige una opción):

```bash
npm i -g openclaw@latest
```

```bash
pnpm add -g openclaw@latest
```

**No** recomendamos Bun como entorno de ejecución del Gateway (errores con WhatsApp/Telegram).

Para cambiar de canal de actualización (instalaciones con git + npm):

```bash
openclaw update --channel beta
openclaw update --channel dev
openclaw update --channel stable
```

Usa `--tag <dist-tag|version>` para una instalación puntual con una etiqueta/versión.

Consulta [Canales de desarrollo](/es/install/development-channels) para la semántica de canales y las notas de lanzamiento.

Nota: en instalaciones con npm, el Gateway registra una sugerencia de actualización al arrancar (verifica la etiqueta de canal actual). Desactívalo mediante `update.checkOnStart: false`.

Luego:

```bash
openclaw doctor
openclaw gateway restart
openclaw health
```

Notas:

* Si el Gateway se ejecuta como servicio, es preferible usar `openclaw gateway restart` en lugar de matar procesos/PIDs.
* Si tienes una versión fijada, consulta «Rollback / pinning» más abajo.

<div id="update-openclaw-update">
  ## Actualizar (`openclaw update`)
</div>

Para **instalaciones desde código fuente** (git checkout), se recomienda:

```bash
openclaw update
```

Ejecuta un flujo de actualización relativamente seguro:

* Requiere un árbol de trabajo limpio.
* Cambia al canal seleccionado (tag o rama).
* Realiza fetch + rebase contra el upstream configurado (canal dev).
* Instala dependencias, compila, compila la Control UI y ejecuta `openclaw doctor`.
* Reinicia el Gateway de forma predeterminada (usa `--no-restart` para omitirlo).

Si instalaste mediante **npm/pnpm** (sin metadatos de git), `openclaw update` intentará actualizar a través de tu gestor de paquetes. Si no puede detectar la instalación, usa en su lugar “Update (global install)”.

<div id="update-control-ui-rpc">
  ## Actualización (Control UI / RPC)
</div>

La Control UI incluye **Update &amp; Restart** (RPC: `update.run`). Esta acción:

1. Ejecuta el mismo flujo de actualización desde código fuente que `openclaw update` (solo `git checkout`).
2. Escribe un centinela de reinicio con un informe estructurado (tramo final de stdout/stderr).
3. Reinicia el Gateway y hace ping a la última sesión activa con el informe.

Si el rebase falla, el Gateway aborta y se reinicia sin aplicar la actualización.

<div id="update-from-source">
  ## Actualizar (desde el código fuente)
</div>

Desde el checkout del repositorio:

Recomendado:

```bash
openclaw update
```

Manual (aproximadamente equivalente):

```bash
git pull
pnpm install
pnpm build
pnpm ui:build # instala automáticamente las dependencias de UI en la primera ejecución
openclaw doctor
openclaw health
```

Notas:

* `pnpm build` es importante cuando ejecutas el binario empaquetado `openclaw` ([`openclaw.mjs`](https://github.com/openclaw/openclaw/blob/main/openclaw.mjs)) o usas Node para ejecutar `dist/`.
* Si ejecutas desde un checkout del repositorio sin una instalación global, usa `pnpm openclaw ...` para comandos de la CLI.
* Si ejecutas directamente desde TypeScript (`pnpm openclaw ...`), por lo general no es necesario recompilar, pero **las migraciones de configuración siguen aplicándose** → ejecuta `openclaw doctor`.
* Cambiar entre instalaciones globales y desde git es fácil: instala la otra variante y luego ejecuta `openclaw doctor` para que el punto de entrada del servicio del Gateway se reescriba a la instalación actual.

<div id="always-run-openclaw-doctor">
  ## Ejecuta siempre: `openclaw doctor`
</div>

Doctor es el comando de «actualización segura». Es deliberadamente aburrido: reparar + migrar + advertir.

Nota: si tienes una **instalación desde código fuente** (git checkout), `openclaw doctor` ofrecerá ejecutar primero `openclaw update`.

Tareas típicas que realiza:

* Migrar claves de configuración obsoletas y ubicaciones heredadas de archivos de configuración.
* Auditar las políticas de mensajes directos (DM) y advertir sobre ajustes open arriesgados (open permite aceptar mensajes sin restricciones de cualquier usuario).
* Comprobar el estado del Gateway y, si es posible, ofrecer reiniciarlo.
* Detectar y migrar servicios antiguos del Gateway (launchd/systemd; schtasks heredados) a los servicios actuales de OpenClaw.
* En Linux, garantizar el user lingering de systemd (para que el Gateway sobreviva al cierre de sesión).

Detalles: [Doctor](/es/gateway/doctor)

<div id="start-stop-restart-the-gateway">
  ## Iniciar / detener / reiniciar el Gateway
</div>

CLI (funciona sin importar el sistema operativo):

```bash
openclaw gateway status
openclaw gateway stop
openclaw gateway restart
openclaw gateway --port 18789
openclaw logs --follow
```

Si lo ejecutas bajo un supervisor de servicios:

* launchd de macOS (LaunchAgent incluido con la app): `launchctl kickstart -k gui/$UID/bot.molt.gateway` (usa `bot.molt.<profile>`; los `com.openclaw.*` heredados siguen funcionando)
* Servicio de usuario systemd en Linux: `systemctl --user restart openclaw-gateway[-<profile>].service`
* Windows (WSL2): `systemctl --user restart openclaw-gateway[-<profile>].service`
  * `launchctl`/`systemctl` solo funcionan si el servicio está instalado; de lo contrario, ejecuta `openclaw gateway install`.

Runbook y etiquetas exactas del servicio: [Gateway runbook](/es/gateway)

<div id="rollback-pinning-when-something-breaks">
  ## Reversión / fijar versión (cuando algo falla)
</div>

<div id="pin-global-install">
  ### Fijar (instalación global)
</div>

Instala una versión que sabes que funciona (sustituye `<version>` por la última que funcionó correctamente):

```bash
npm i -g openclaw@<version>
```

```bash
pnpm add -g openclaw@<version>
```

Consejo: para ver la versión actualmente publicada, ejecuta `npm view openclaw version`.

Luego reinicia y vuelve a ejecutar el comando doctor:

```bash
openclaw doctor
openclaw gateway restart
```

<div id="pin-source-by-date">
  ### Fijar (origen) por fecha
</div>

Selecciona un commit de una fecha (ejemplo: “estado de main a 2026-01-01”):

```bash
git fetch origin
git checkout "$(git rev-list -n 1 --before=\"2026-01-01\" origin/main)"
```

Luego reinstala las dependencias y reinicia:

```bash
pnpm install
pnpm build
openclaw gateway restart
```

Si más adelante quieres volver a la versión más reciente:

```bash
git checkout main
git pull
```

<div id="if-youre-stuck">
  ## Si te quedas atascado
</div>

* Vuelve a ejecutar `openclaw doctor` y lee la salida con atención (a menudo indica cómo solucionarlo).
* Consulta: [Solución de problemas](/es/gateway/troubleshooting)
* Pregunta en Discord: https://channels.discord.gg/clawd