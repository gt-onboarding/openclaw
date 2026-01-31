---
title: Configuración
summary: "Guía de configuración: mantén tu configuración personalizada de OpenClaw siempre actualizada"
read_when:
  - Al configurar una máquina nueva
  - Quieres tener “lo último y lo mejor” sin estropear tu configuración personal
---

<div id="setup">
  # Configuración
</div>

Última actualización: 2026-01-01

<div id="tldr">
  ## TL;DR
</div>

* **Los ajustes viven fuera del repo:** `~/.openclaw/workspace` (espacio de trabajo) + `~/.openclaw/openclaw.json` (configuración).
* **Flujo de trabajo estable:** instala la app de macOS y deja que ejecute el Gateway empaquetado.
* **Flujo de trabajo de última generación:** ejecuta tú mismo el Gateway mediante `pnpm gateway:watch` y luego deja que la app de macOS se conecte en modo Local.

<div id="prereqs-from-source">
  ## Requisitos previos (desde el código fuente)
</div>

* Node `>=22`
* `pnpm`
* Docker (opcional; solo para configuración en contenedor/e2e — consulta [Docker](/es/install/docker))

<div id="tailoring-strategy-so-updates-dont-hurt">
  ## Estrategia de personalización (para que las actualizaciones no duelan)
</div>

Si quieres &quot;100 % personalizado para mí&quot; *y* actualizaciones sencillas, mantén tu personalización en:

* **Configuración:** `~/.openclaw/openclaw.json` (tipo JSON/JSON5)
* **Espacio de trabajo:** `~/.openclaw/workspace` (habilidades, prompts, memorias; conviértelo en un repositorio git privado)

Realiza la configuración inicial una sola vez:

```bash
openclaw setup
```

Dentro de este repositorio, usa el punto de entrada local de la CLI:

```bash
openclaw setup
```

Si aún no tienes una instalación global, ejecútalo con `pnpm openclaw setup`.

<div id="stable-workflow-macos-app-first">
  ## Flujo estable (primero la app de macOS)
</div>

1. Instala y abre **OpenClaw.app** (barra de menús).
2. Completa la lista de verificación de onboarding/permisos (avisos TCC).
3. Asegúrate de que el Gateway está en **Local** y en ejecución (la app lo gestiona).
4. Conecta superficies (ejemplo: WhatsApp):

```bash
openclaw channels login
```

5. Comprobación rápida:

```bash
openclaw health
```

Si el flujo de onboarding no está disponible en tu build:

* Ejecuta `openclaw setup`, luego `openclaw channels login` y, por último, inicia el Gateway manualmente (`openclaw gateway`).

<div id="bleeding-edge-workflow-gateway-in-a-terminal">
  ## Flujo de trabajo de última generación (Gateway en una terminal)
</div>

Objetivo: trabajar en el Gateway de TypeScript, tener recarga en caliente y mantener la UI de la app de macOS vinculada.

<div id="0-optional-run-the-macos-app-from-source-too">
  ### 0) (Opcional) Ejecutar también la app de macOS desde el código fuente
</div>

Si también quieres tener la app de macOS en la versión más reciente (bleeding edge):

```bash
./scripts/restart-mac.sh
```

<div id="1-start-the-dev-gateway">
  ### 1) Inicia el Gateway de desarrollo
</div>

```bash
pnpm install
pnpm gateway:watch
```

`gateway:watch` ejecuta el Gateway en modo de observación y se recarga al detectar cambios en TypeScript.

<div id="2-point-the-macos-app-at-your-running-gateway">
  ### 2) Apunta la app de macOS a tu Gateway en ejecución
</div>

En **OpenClaw.app**:

* Modo de conexión: **Local**
  La app se conectará al Gateway en ejecución en el puerto configurado.

<div id="3-verify">
  ### 3) Verificar
</div>

* El estado del Gateway en la aplicación debería mostrar **“Using existing gateway …”**
* O mediante la CLI:

```bash
openclaw health
```

<div id="common-footguns">
  ### Trampas habituales
</div>

* **Puerto incorrecto:** el WS del Gateway usa de forma predeterminada `ws://127.0.0.1:18789`; mantén la app y la CLI en el mismo puerto.
* **Dónde se almacena el estado:**
  * Credenciales: `~/.openclaw/credentials/`
  * Sesiones: `~/.openclaw/agents/<agentId>/sessions/`
  * Logs: `/tmp/openclaw/`

<div id="credential-storage-map">
  ## Mapa de almacenamiento de credenciales
</div>

Usa esto al depurar la autenticación o decidir qué respaldar:

* **WhatsApp**: `~/.openclaw/credentials/whatsapp/<accountId>/creds.json`
* **Token de bot de Telegram**: config/env o `channels.telegram.tokenFile`
* **Token de bot de Discord**: config/env (archivo de token aún no compatible)
* **Tokens de Slack**: config/env (`channels.slack.*`)
* **Listas de permitidos de emparejamiento**: `~/.openclaw/credentials/<channel>-allowFrom.json`
* **Perfiles de autenticación de modelos**: `~/.openclaw/agents/<agentId>/agent/auth-profiles.json`
* **Importación OAuth heredada**: `~/.openclaw/credentials/oauth.json`
  Más detalles: [Seguridad](/es/gateway/security#credential-storage-map).

<div id="updating-without-wrecking-your-setup">
  ## Actualización (sin romper tu configuración)
</div>

* Mantén `~/.openclaw/workspace` y `~/.openclaw/` como “tus cosas”; no metas prompts/configuración personales en el repositorio de `openclaw`.
* Para actualizar el código fuente: `git pull` + `pnpm install` (cuando cambie el lockfile) + sigue usando `pnpm gateway:watch`.

<div id="linux-systemd-user-service">
  ## Linux (servicio de usuario systemd)
</div>

Las instalaciones en Linux usan un servicio de **usuario** de systemd. De forma predeterminada, systemd detiene los servicios de usuario al cerrar la sesión o por inactividad, lo que termina el Gateway. El asistente de configuración intenta habilitar el *lingering* por ti (puede solicitar sudo). Si sigue desactivado, ejecuta:

```bash
sudo loginctl enable-linger $USER
```

Para servidores siempre encendidos o multiusuario, considera usar un servicio de **sistema** en lugar de un
servicio de usuario (no hace falta habilitar *lingering*). Consulta el [Gateway runbook](/es/gateway) para las notas sobre systemd.

<div id="related-docs">
  ## Documentación relacionada
</div>

* [Gateway runbook](/es/gateway) (flags, supervisión, puertos)
* [Configuración del Gateway](/es/gateway/configuration) (esquema de configuración + ejemplos)
* [Discord](/es/channels/discord) y [Telegram](/es/channels/telegram) (etiquetas de respuesta + ajustes de replyToMode)
* [Configuración del asistente de OpenClaw](/es/start/openclaw)
* [Aplicación de macOS](/es/platforms/macos) (ciclo de vida del Gateway)