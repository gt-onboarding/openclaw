---
title: Nix
summary: "Instalar OpenClaw declarativamente con Nix"
read_when:
  - Quieres instalaciones reproducibles, con posibilidad de reversión
  - Ya usas Nix/NixOS/Home Manager
  - Quieres tenerlo todo fijado ("pinned") y gestionado de forma declarativa
---

<div id="nix-installation">
  # Instalación con Nix
</div>

La forma recomendada de ejecutar OpenClaw con Nix es mediante **[nix-openclaw](https://github.com/openclaw/nix-openclaw)**, un módulo de Home Manager que viene con todo lo necesario.

<div id="quick-start">
  ## Inicio rápido
</div>

Pega lo siguiente en tu agente de IA (Claude, Cursor, etc.):

```text
I want to set up nix-openclaw on my Mac.
Repository: github:openclaw/nix-openclaw

What I need you to do:
1. Check if Determinate Nix is installed (if not, install it)
2. Create a local flake at ~/code/openclaw-local using templates/agent-first/flake.nix
3. Help me create a Telegram bot (@BotFather) and get my chat ID (@userinfobot)
4. Set up secrets (bot token, Anthropic key) - plain files at ~/.secrets/ is fine
5. Fill in the template placeholders and run home-manager switch
6. Verify: launchd running, bot responds to messages

Reference the nix-openclaw README for module options.
```

> **📦 Guía completa: [github.com/openclaw/nix-openclaw](https://github.com/openclaw/nix-openclaw)**
>
> El repositorio nix-openclaw es la fuente de referencia para la instalación con Nix. Esta página es solo un resumen rápido.

<div id="what-you-get">
  ## Qué obtienes
</div>

* Gateway + app de macOS + herramientas (whisper, spotify, cámaras), todo con versiones fijadas
* Servicio launchd que sobrevive a los reinicios
* Sistema de complementos con configuración declarativa
* Reversión instantánea: `home-manager switch --rollback`

***

<div id="nix-mode-runtime-behavior">
  ## Comportamiento en tiempo de ejecución del modo Nix
</div>

Cuando se establece `OPENCLAW_NIX_MODE=1` (automático con nix-openclaw):

OpenClaw ofrece un **modo Nix** que hace que la configuración sea determinista y desactiva los procesos de instalación automática.
Actívalo exportando:

```bash
OPENCLAW_NIX_MODE=1
```

En macOS, la aplicación con GUI no hereda automáticamente las variables de entorno de la shell. También puedes activar el modo Nix mediante `defaults`:

```bash
defaults write bot.molt.mac openclaw.nixMode -bool true
```

<div id="config-state-paths">
  ### Rutas de configuración y estado
</div>

OpenClaw lee la configuración en formato JSON5 desde `OPENCLAW_CONFIG_PATH` y almacena los datos mutables en `OPENCLAW_STATE_DIR`.

* `OPENCLAW_STATE_DIR` (por defecto: `~/.openclaw`)
* `OPENCLAW_CONFIG_PATH` (por defecto: `$OPENCLAW_STATE_DIR/openclaw.json`)

Cuando se ejecute con Nix, define estos valores explícitamente a ubicaciones gestionadas por Nix para que el estado y la configuración en tiempo de ejecución se mantengan fuera del almacén inmutable.

<div id="runtime-behavior-in-nix-mode">
  ### Comportamiento en tiempo de ejecución en modo Nix
</div>

* Los flujos de autoinstalación y automodificación están deshabilitados
* Las dependencias que faltan generan mensajes de resolución específicos de Nix
* La UI muestra un banner que indica el modo Nix de solo lectura cuando corresponde

<div id="packaging-note-macos">
  ## Nota sobre el empaquetado (macOS)
</div>

El proceso de empaquetado de macOS espera una plantilla estable de Info.plist en:

```
apps/macos/Sources/OpenClaw/Resources/Info.plist
```

[`scripts/package-mac-app.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/package-mac-app.sh) copia esta plantilla en el bundle de la app y actualiza los campos dinámicos
(bundle ID, versión/build, Git SHA, claves de Sparkle). Esto mantiene el plist determinístico para el empaquetado con SwiftPM
y las compilaciones de Nix (que no dependen de una toolchain completa de Xcode).

<div id="related">
  ## Relacionado
</div>

* [nix-openclaw](https://github.com/openclaw/nix-openclaw) — guía completa de instalación y configuración
* [Wizard](/es/start/wizard) — configuración de la CLI sin Nix
* [Docker](/es/install/docker) — configuración con Docker (contenedores)