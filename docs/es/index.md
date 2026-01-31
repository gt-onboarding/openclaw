---
title: Index
summary: "Top-level overview of OpenClaw, features, and purpose"
read_when:
  - Introducing OpenClaw to newcomers
---

<div id="openclaw">
  # OpenClaw 🦞
</div>

> *&quot;EXFOLIATE! EXFOLIATE!&quot;* — A space lobster, probably

<p align="center">
  <picture>
    <source media="(prefers-color-scheme: light)" srcset="https://raw.githubusercontent.com/openclaw/openclaw/main/docs/assets/openclaw-logo-text-dark.png" />

    <img src="https://raw.githubusercontent.com/openclaw/openclaw/main/docs/assets/openclaw-logo-text.png" alt="OpenClaw" width="500" />
  </picture>
</p>

<p align="center">
  <strong>
    Any OS + WhatsApp/Telegram/Discord/iMessage gateway for AI agents (Pi).
  </strong>

  <br />

  Plugins add Mattermost and more. Send a message, get an agent response — from
  your pocket.
</p>

<p align="center">
  <a href="https://github.com/openclaw/openclaw">GitHub</a> ·
  <a href="https://github.com/openclaw/openclaw/releases">Releases</a> ·
  <a href="/es/">Docs</a> ·<a href="/es/start/openclaw">OpenClaw assistant setup</a>
</p>

OpenClaw conecta WhatsApp (vía WhatsApp Web / Baileys), Telegram (Bot API / grammY), Discord (Bot API / channels.discord.js) e iMessage (imsg CLI) con agentes de código como [Pi](https://github.com/badlogic/pi-mono). Los complementos añaden Mattermost (Bot API + WebSocket) y más.
OpenClaw también impulsa el asistente OpenClaw.

<div id="start-here">
  ## Comienza aquí
</div>

* **Nueva instalación desde cero:** [Primeros pasos](/es/start/getting-started)
* **Configuración guiada (recomendada):** [Asistente](/es/start/wizard) (`openclaw onboard`)
* **Abrir el panel (Gateway local):** http://127.0.0.1:18789/ (o http://localhost:18789/)

Si el Gateway se está ejecutando en el mismo equipo, ese enlace abre la Control UI en el navegador de inmediato. Si falla, inicia primero el Gateway: `openclaw gateway`.

<div id="dashboard-browser-control-ui">
  ## Dashboard (Control UI en el navegador)
</div>

El dashboard es la Control UI en el navegador para chat, configuración, nodos, sesiones y más.
Acceso local predeterminado: http://127.0.0.1:18789/
Acceso remoto: [Interfaces web](/es/web) y [Tailscale](/es/gateway/tailscale)

<p align="center">
  <img src="/whatsapp-openclaw.jpg" alt="OpenClaw" width="420" />
</p>

<div id="how-it-works">
  ## Cómo funciona
</div>

```
WhatsApp / Telegram / Discord / iMessage (+ plugins)
        │
        ▼
  ┌───────────────────────────┐
  │          Gateway          │  ws://127.0.0.1:18789 (loopback-only)
  │     (single source)       │
  │                           │  http://<gateway-host>:18793
  │                           │    /__openclaw__/canvas/ (host de Canvas)
  └───────────┬───────────────┘
              │
              ├─ Pi agent (RPC)
              ├─ CLI (openclaw …)
              ├─ Chat UI (SwiftUI)
              ├─ macOS app (OpenClaw.app)
              ├─ iOS node via Gateway WS + pairing
              └─ Android node via Gateway WS + pairing
```

La mayoría de las operaciones pasan a través del **Gateway** (`openclaw gateway`), un único proceso de larga duración que gestiona las conexiones de los canales y el plano de control WebSocket.

<div id="network-model">
  ## Modelo de red
</div>

* **Un Gateway por host (recomendado)**: es el único proceso autorizado para tener la sesión de WhatsApp Web. Si necesitas un bot de rescate o un aislamiento estricto, ejecuta varios gateways con perfiles y puertos aislados; consulta [Múltiples gateways](/es/gateway/multiple-gateways).
* **Prioridad al loopback**: el WS del Gateway usa por defecto `ws://127.0.0.1:18789`.
  * El asistente ahora genera un token de gateway por defecto (incluso para loopback).
  * Para acceso vía Tailnet, ejecuta `openclaw gateway --bind tailnet --token ...` (el token es obligatorio para binds que no sean loopback).
* **Nodos**: se conectan al WebSocket del Gateway (LAN/tailnet/SSH según sea necesario); el puente TCP heredado está obsoleto y eliminado.
* **Host de Canvas**: servidor de archivos HTTP en `canvasHost.port` (por defecto `18793`), que sirve `/__openclaw__/canvas/` para las WebViews de los nodos; consulta [Configuración del Gateway](/es/gateway/configuration) (`canvasHost`).
* **Uso remoto**: túnel SSH o tailnet/VPN; consulta [Acceso remoto](/es/gateway/remote) y [Descubrimiento](/es/gateway/discovery).

<div id="features-high-level">
  ## Funcionalidades (vista general)
</div>

* 📱 **Integración con WhatsApp** — Usa Baileys para el protocolo de WhatsApp Web
* ✈️ **Bot de Telegram** — DMs + grupos vía grammY
* 🎮 **Bot de Discord** — DMs + canales de servidor vía channels.discord.js
* 🧩 **Bot de Mattermost (complemento)** — Token de bot + eventos WebSocket
* 💬 **iMessage** — Integración local con la CLI `imsg` (macOS)
* 🤖 **Puente de agente** — Pi (modo RPC) con streaming de herramientas
* ⏱️ **Streaming + fragmentación** — Streaming por bloques + detalles de streaming de borradores en Telegram ([/concepts/streaming](/es/concepts/streaming))
* 🧠 **Enrutamiento multiagente** — Enruta cuentas/pares de proveedores a agentes aislados (espacio de trabajo + sesiones por agente)
* 🔐 **Autenticación por suscripción** — Anthropic (Claude Pro/Max) + OpenAI (ChatGPT/Codex) vía OAuth
* 💬 **Sesiones** — Los chats directos se agrupan en `main` (predeterminado); los grupos están aislados
* 👥 **Compatibilidad con chat grupal** — Basado en menciones por defecto; el propietario puede cambiar `/activation always|mention`
* 📎 **Compatibilidad con contenido multimedia** — Enviar y recibir imágenes, audio y documentos
* 🎤 **Notas de voz** — Hook opcional de transcripción
* 🖥️ **WebChat + app de macOS** — UI local + aplicación en la barra de menús para operaciones y activación por voz
* 📱 **Nodo iOS** — Se empareja como nodo y expone una superficie Canvas
* 📱 **Nodo Android** — Se empareja como nodo y expone Canvas + Chat + Cámara

Nota: se han eliminado las rutas heredadas de Claude/Codex/Gemini/Opencode; Pi es la única ruta de agente para código.

<div id="quick-start">
  ## Inicio rápido
</div>

Requisito del entorno de ejecución: **Node ≥ 22**.

```bash
# Recommended: global install (npm/pnpm)
npm install -g openclaw@latest
# or: pnpm add -g openclaw@latest

# Onboard + install the service (launchd/systemd user service)
openclaw onboard --install-daemon

# Pair WhatsApp Web (shows QR)
openclaw channels login

# El Gateway se ejecuta como servicio tras la configuración inicial; la ejecución manual sigue siendo posible:
openclaw gateway --port 18789
```

Cambiar entre instalaciones de npm y git posteriormente es sencillo: instala la otra variante y ejecuta `openclaw doctor` para actualizar el punto de entrada del servicio Gateway.

Desde el código fuente (desarrollo):

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm ui:build # instala automáticamente las dependencias de UI en la primera ejecución
pnpm build
openclaw onboard --install-daemon
```

Si aún no tienes una instalación global, ejecuta el paso de onboarding con `pnpm openclaw ...` desde el repositorio.

Inicio rápido de múltiples instancias (opcional):

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json \
OPENCLAW_STATE_DIR=~/.openclaw-a \
openclaw gateway --port 19001
```

Envía un mensaje de prueba (requiere que el Gateway esté en ejecución):

```bash
openclaw message send --target +15555550123 --message "Hello from OpenClaw"
```

<div id="configuration-optional">
  ## Configuración (opcional)
</div>

La configuración se encuentra en `~/.openclaw/openclaw.json`.

* Si **no haces nada**, OpenClaw usa el binario de Pi incluido en modo RPC con sesiones individuales por remitente.
* Si quieres restringir el acceso, empieza con `channels.whatsapp.allowFrom` y (para grupos) las reglas de menciones.

Ejemplo:

```json5
{
  channels: {
    whatsapp: {
      allowFrom: ["+15555550123"],
      groups: { "*": { requireMention: true } }
    }
  },
  messages: { groupChat: { mentionPatterns: ["@openclaw"] } }
}
```

<div id="docs">
  ## Documentación
</div>

* Empieza aquí:
  * [Hubs de documentación (todas las páginas enlazadas)](/es/start/hubs)
  * [Ayuda](/es/help) ← *correcciones habituales + resolución de problemas*
  * [Configuración](/es/gateway/configuration)
  * [Ejemplos de configuración](/es/gateway/configuration-examples)
  * [Comandos slash](/es/tools/slash-commands)
  * [Enrutamiento multi-agente](/es/concepts/multi-agent)
  * [Actualización / rollback](/es/install/updating)
  * [Emparejamiento (DM + nodos)](/es/start/pairing)
  * [Modo Nix](/es/install/nix)
  * [Configuración del asistente OpenClaw](/es/start/openclaw)
  * [Habilidades](/es/tools/skills)
  * [Configuración de habilidades](/es/tools/skills-config)
  * [Plantillas de espacio de trabajo](/es/reference/templates/AGENTS)
  * [Adaptadores RPC](/es/reference/rpc)
  * [Runbook del Gateway](/es/gateway)
  * [Nodos (iOS/Android)](/es/nodes)
  * [Interfaces web (Control UI)](/es/web)
  * [Descubrimiento + transportes](/es/gateway/discovery)
  * [Acceso remoto](/es/gateway/remote)
* Proveedores y UX:
  * [WebChat](/es/web/webchat)
  * [Control UI (navegador)](/es/web/control-ui)
  * [Telegram](/es/channels/telegram)
  * [Discord](/es/channels/discord)
  * [Mattermost (complemento)](/es/channels/mattermost)
  * [iMessage](/es/channels/imessage)
  * [Grupos](/es/concepts/groups)
  * [Mensajes de grupo de WhatsApp](/es/concepts/group-messages)
  * [Multimedia: imágenes](/es/nodes/images)
  * [Multimedia: audio](/es/nodes/audio)
* Apps complementarias:
  * [App para macOS](/es/platforms/macos)
  * [App para iOS](/es/platforms/ios)
  * [App para Android](/es/platforms/android)
  * [Windows (WSL2)](/es/platforms/windows)
  * [App para Linux](/es/platforms/linux)
* Operaciones y seguridad:
  * [Sesiones](/es/concepts/session)
  * [Tareas cron](/es/automation/cron-jobs)
  * [Webhooks](/es/automation/webhook)
  * [Hooks de Gmail (Pub/Sub)](/es/automation/gmail-pubsub)
  * [Seguridad](/es/gateway/security)
  * [Resolución de problemas](/es/gateway/troubleshooting)

<div id="the-name">
  ## El nombre
</div>

**OpenClaw = CLAW + TARDIS** — porque toda langosta espacial necesita una máquina espacio‑tiempo.

***

*&quot;Al final, todos estamos jugando con nuestros propios prompts.&quot;* — una IA, probablemente pasada de tokens

<div id="credits">
  ## Créditos
</div>

* **Peter Steinberger** ([@steipete](https://twitter.com/steipete)) — Creador, susurrador de langostas
* **Mario Zechner** ([@badlogicc](https://twitter.com/badlogicgames)) — Creador de Pi, pentester de seguridad
* **Clawd** — La langosta espacial que exigió un mejor nombre

<div id="core-contributors">
  ## Colaboradores principales
</div>

* **Maxim Vovshin** (@Hyaxia, 36747317+Hyaxia@users.noreply.github.com) — skill Blogwatcher
* **Nacho Iacovino** (@nachoiacovino, nacho.iacovino@gmail.com) — procesamiento de ubicaciones (Telegram + WhatsApp)

<div id="license">
  ## Licencia
</div>

MIT — Libre como una langosta en el océano 🦞

***

*&quot;Al final todos estamos jugando con nuestros propios prompts.&quot;* — Una IA, probablemente colocada a base de tokens