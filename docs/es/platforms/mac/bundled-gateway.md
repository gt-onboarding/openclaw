---
title: Gateway empaquetado
summary: "Runtime del Gateway en macOS (servicio launchd externo)"
read_when:
  - Empaquetar OpenClaw.app
  - Depurar el servicio launchd del Gateway en macOS
  - Instalar la CLI del Gateway en macOS
---

<div id="gateway-on-macos-external-launchd">
  # Gateway en macOS (launchd externo)
</div>

OpenClaw.app ya no incluye Node/Bun ni el runtime del Gateway. La app de macOS
espera una instalación **externa** de la CLI `openclaw`, no inicia el Gateway como
proceso hijo y gestiona un servicio launchd por usuario para mantener el Gateway
en ejecución (o se conecta a un Gateway local existente si ya se está ejecutando).

<div id="install-the-cli-required-for-local-mode">
  ## Instala la CLI (obligatoria para el modo local)
</div>

Necesitas Node 22 o superior en tu Mac y luego instalar `openclaw` de forma global:

```bash
npm install -g openclaw@<version>
```

El botón **Install CLI** de la aplicación de macOS ejecuta el mismo flujo a través de npm/pnpm (bun no se recomienda para el entorno de ejecución de Gateway).


<div id="launchd-gateway-as-launchagent">
  ## Launchd (Gateway como LaunchAgent)
</div>

Etiqueta:

- `bot.molt.gateway` (o `bot.molt.<profile>`; el heredado `com.openclaw.*` puede permanecer)

Ubicación del archivo plist (por usuario):

- `~/Library/LaunchAgents/bot.molt.gateway.plist`
  (o `~/Library/LaunchAgents/bot.molt.<profile>.plist`)

Administrador:

- La app de macOS se encarga de la instalación/actualización del LaunchAgent en modo local.
- La CLI también puede instalarlo: `openclaw gateway install`.

Comportamiento:

- “OpenClaw Active” habilita/deshabilita el LaunchAgent.
- Cerrar la app **no** detiene el Gateway (launchd lo mantiene activo).
- Si ya se está ejecutando un Gateway en el puerto configurado, la app se conecta a
  él en lugar de iniciar uno nuevo.

Registro:

- stdout/err de launchd: `/tmp/openclaw/openclaw-gateway.log`

<div id="version-compatibility">
  ## Compatibilidad de versiones
</div>

La app de macOS comprueba la versión del Gateway con respecto a su propia versión. Si no son
compatibles, actualiza la CLI global para que coincida con la versión de la app.

<div id="smoke-check">
  ## Comprobación rápida
</div>

```bash
openclaw --version

OPENCLAW_SKIP_CHANNELS=1 \
OPENCLAW_SKIP_CANVAS_HOST=1 \
openclaw gateway --port 18999 --bind loopback
```

A continuación:

```bash
openclaw gateway call health --url ws://127.0.0.1:18999 --timeout 3000
```
