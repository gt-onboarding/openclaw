---
title: Primeros pasos
summary: "Guía para principiantes: de cero a tu primer mensaje (asistente, autenticación, canales, emparejamiento)"
read_when:
  - Configuración inicial desde cero
  - Quieres la ruta más rápida desde la instalación → puesta en marcha → primer mensaje
---

<div id="getting-started">
  # Primeros pasos
</div>

Objetivo: pasar de **cero** → **primer chat funcionando** (con valores predeterminados razonables) lo más rápido posible.

Chat más rápido: abre el Control UI (no se necesita configurar ningún canal). Ejecuta `openclaw dashboard`
y chatea en el navegador, o abre `http://127.0.0.1:18789/` en el host del Gateway.
Documentación: [Dashboard](/es/web/dashboard) y [Control UI](/es/web/control-ui).

Ruta recomendada: usa el **asistente de onboarding en la CLI** (`openclaw onboard`). Configura:

* modelo/autenticación (se recomienda OAuth)
* configuración del Gateway
* canales (WhatsApp/Telegram/Discord/Mattermost (complemento)/...)
* valores predeterminados de emparejamiento (DMs seguras)
* inicialización del espacio de trabajo + habilidades
* servicio en segundo plano opcional

Si quieres las páginas de referencia más detalladas, ve a: [Wizard](/es/start/wizard), [Setup](/es/start/setup), [Pairing](/es/start/pairing), [Security](/es/gateway/security).

Nota sobre sandbox: `agents.defaults.sandbox.mode: "non-main"` usa `session.mainKey` (por defecto `"main"`),
por lo que las sesiones de grupo/canal se ejecutan en sandbox. Si quieres que el agente principal
siempre se ejecute en el host, define un override explícito por agente:

```json
{
  "routing": {
    "agents": {
      "main": {
        "workspace": "~/.openclaw/workspace",
        "sandbox": { "mode": "off" }
      }
    }
  }
}
```

<div id="0-prereqs">
  ## 0) Requisitos previos
</div>

* Node `>=22`
* `pnpm` (opcional; recomendado si compilas desde el código fuente)
* **Recomendado:** clave de API de Brave Search para búsqueda web. La forma más sencilla es:
  `openclaw configure --section web` (almacena `tools.web.search.apiKey`).
  Consulta [Herramientas web](/es/tools/web).

macOS: si planeas compilar las aplicaciones, instala Xcode / CLT. Para usar solo la CLI y el Gateway, basta con Node.
Windows: usa **WSL2** (se recomienda Ubuntu). WSL2 es muy recomendable; Windows nativo no está probado, es más problemático y tiene peor compatibilidad con herramientas. Instala primero WSL2 y luego ejecuta los pasos de Linux dentro de WSL2. Consulta [Windows (WSL2)](/es/platforms/windows).

<div id="1-install-the-cli-recommended">
  ## 1) Instala la CLI (recomendado)
</div>

```bash
curl -fsSL https://openclaw.bot/install.sh | bash
```

Opciones de instalación (método de instalación, modo no interactivo, desde GitHub): [Instalación](/es/install).

Windows (PowerShell):

```powershell
iwr -useb https://openclaw.ai/install.ps1 | iex
```

Alternativa (instalación global):

```bash
npm install -g openclaw@latest
```

```bash
pnpm add -g openclaw@latest
```

<div id="2-run-the-onboarding-wizard-and-install-the-service">
  ## 2) Ejecuta el asistente de configuración inicial (e instala el servicio)
</div>

```bash
openclaw onboard --install-daemon
```

Lo que vas a elegir:

* **Gateway** local vs remoto
* **Auth**: suscripción a OpenAI Code (Codex) (OAuth) o claves de API. Para Anthropic recomendamos una clave de API; `claude setup-token` también es compatible.
* **Proveedores**: inicio de sesión por QR de WhatsApp, tokens de bot de Telegram/Discord, tokens de complemento de Mattermost, etc.
* **Daemon**: instalación en segundo plano (launchd/systemd; WSL2 usa systemd)
  * **Runtime**: Node (recomendado; obligatorio para WhatsApp/Telegram). Bun **no se recomienda**.
* **Token del Gateway**: el asistente genera uno de forma predeterminada (incluso en loopback) y lo guarda en `gateway.auth.token`.

Documentación del asistente: [Wizard](/es/start/wizard)

<div id="auth-where-it-lives-important">
  ### Autenticación: dónde se almacena (importante)
</div>

* **Ruta Anthropic recomendada:** configura una clave de API (el asistente puede guardarla para uso del servicio). También se admite `claude setup-token` si quieres reutilizar las credenciales de Claude Code.

* Credenciales OAuth (importación heredada): `~/.openclaw/credentials/oauth.json`

* Perfiles de autenticación (OAuth + claves de API): `~/.openclaw/agents/<agentId>/agent/auth-profiles.json`

Consejo para modo headless/servidor: realiza primero OAuth en una máquina normal y luego copia `oauth.json` al host del Gateway.

<div id="3-start-the-gateway">
  ## 3) Inicia el Gateway
</div>

Si instalaste el servicio durante la configuración inicial, el Gateway ya debería estar ejecutándose:

```bash
openclaw gateway status
```

Ejecución manual (en primer plano):

```bash
openclaw gateway --port 18789 --verbose
```

Panel (loopback local): `http://127.0.0.1:18789/`
Si se ha configurado un token, pégalo en la configuración del Control UI (se almacena como `connect.params.auth.token`).

⚠️ **Advertencia sobre Bun (WhatsApp + Telegram):** Bun tiene problemas conocidos con estos canales. Si usas WhatsApp o Telegram, ejecuta el Gateway con **Node**.

<div id="35-quick-verify-2-min">
  ## 3.5) Verificación rápida (2 min)
</div>

```bash
openclaw status
openclaw health
openclaw security audit --deep
```

<div id="4-pair-connect-your-first-chat-surface">
  ## 4) Vincula y conecta tu primer canal de chat
</div>

<div id="whatsapp-qr-login">
  ### WhatsApp (inicio de sesión con código QR)
</div>

```bash
openclaw channels login
```

Escanea desde WhatsApp → Configuración → Dispositivos vinculados.

Documentación de WhatsApp: [WhatsApp](/es/channels/whatsapp)

<div id="telegram-discord-others">
  ### Telegram / Discord / otros
</div>

El asistente puede generar los tokens y la configuración por ti. Si prefieres la configuración manual, comienza con:

* Telegram: [Telegram](/es/channels/telegram)
* Discord: [Discord](/es/channels/discord)
* Mattermost (complemento): [Mattermost](/es/channels/mattermost)

**Consejo sobre DMs en Telegram:** tu primer DM devuelve un código de emparejamiento. Apruébalo (consulta el siguiente paso) o el bot no responderá.

<div id="5-dm-safety-pairing-approvals">
  ## 5) Seguridad de DM (aprobaciones de emparejamiento)
</div>

Comportamiento predeterminado: los DM de remitentes desconocidos reciben un código breve y los mensajes no se procesan hasta que se aprueben.
Si tu primer DM no recibe respuesta, aprueba el emparejamiento:

```bash
openclaw pairing list whatsapp
openclaw pairing approve whatsapp <code>
```

Documentación de emparejamiento: [Emparejamiento](/es/start/pairing)

<div id="from-source-development">
  ## Desde el código fuente (desarrollo)
</div>

Si estás trabajando en el propio OpenClaw, ejecútalo desde el código fuente:

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm ui:build # instala automáticamente las dependencias de la UI en la primera ejecución
pnpm build
openclaw onboard --install-daemon
```

Si aún no tienes una instalación global, ejecuta el paso de onboarding con `pnpm openclaw ...` desde el repositorio.
`pnpm build` también empaqueta los recursos de A2UI; si necesitas ejecutar solo ese paso, usa `pnpm canvas:a2ui:bundle`.

Gateway (desde este repositorio):

```bash
node openclaw.mjs gateway --port 18789 --verbose
```

<div id="7-verify-end-to-end">
  ## 7) Verificar de extremo a extremo
</div>

En una terminal nueva, envía un mensaje de prueba:

```bash
openclaw message send --target +15555550123 --message "Hello from OpenClaw"
```

Si `openclaw health` muestra “no auth configured”, vuelve al asistente y configura OAuth/autenticación mediante clave: el agente no podrá responder sin ello.

Consejo: `openclaw status --all` es el mejor informe de depuración de solo lectura, ideal para pegar.
Comprobaciones de estado: `openclaw health` (o `openclaw status --deep`) le pide al Gateway en ejecución una instantánea de su estado de salud.

<div id="next-steps-optional-but-great">
  ## Próximos pasos (opcionales, pero muy recomendables)
</div>

* App de la barra de menús de macOS + activación por voz: [app de macOS](/es/platforms/macos)
* nodos iOS/Android (Canvas/cámara/voz): [Nodos](/es/nodes)
* Acceso remoto (túnel SSH / Tailscale Serve): [Acceso remoto](/es/gateway/remote) y [Tailscale](/es/gateway/tailscale)
* Configuraciones siempre encendidas / VPN: [Acceso remoto](/es/gateway/remote), [exe.dev](/es/platforms/exe-dev), [Hetzner](/es/platforms/hetzner), [macOS remoto](/es/platforms/mac/remote)