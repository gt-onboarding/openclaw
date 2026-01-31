---
title: Índice
summary: "Visión general de alto nivel de OpenClaw, sus funciones y su propósito"
read_when:
  - Introducir OpenClaw a personas que se inician en la plataforma
---

<div id="openclaw">
  # OpenClaw 🦞
</div>

> *&quot;¡EXFÓLIATE! ¡EXFÓLIATE!&quot;* — Una langosta espacial, probablemente

<p align="center">
  <picture>
    <source media="(prefers-color-scheme: light)" srcset="https://raw.githubusercontent.com/openclaw/openclaw/main/docs/assets/openclaw-logo-text-dark.png" />

    <img src="https://raw.githubusercontent.com/openclaw/openclaw/main/docs/assets/openclaw-logo-text.png" alt="OpenClaw" width="500" />
  </picture>
</p>

<p align="center">
  <strong>Gateway de IA para agentes (como Pi) en cualquier sistema operativo, con soporte para WhatsApp/Telegram/Discord/iMessage.</strong><br />
  Los complementos añaden Mattermost y más.
  Envía un mensaje, recibe la respuesta de un agente — desde tu bolsillo.
</p>

<p align="center">
  <a href="https://github.com/openclaw/openclaw">GitHub</a> ·
  <a href="https://github.com/openclaw/openclaw/releases">Releases</a> ·
  <a href="/es/">Documentación</a> ·
  <a href="/es/start/openclaw">Configuración del asistente OpenClaw</a>
</p>

OpenClaw conecta WhatsApp (a través de WhatsApp Web / Baileys), Telegram (Bot API / grammY), Discord (Bot API / channels.discord.js) e iMessage (imsg CLI) con agentes de programación como [Pi](https://github.com/badlogic/pi-mono). Los complementos añaden Mattermost (Bot API + WebSocket) y más.
OpenClaw también es la base del asistente de OpenClaw.

<div id="start-here">
  ## Comienza aquí
</div>

* **Nueva instalación desde cero:** [Primeros pasos](/es/start/getting-started)
* **Configuración guiada (recomendada):** [Asistente](/es/start/wizard) (`openclaw onboard`)
* **Abrir el dashboard (Gateway local):** http://127.0.0.1:18789/ (o http://localhost:18789/)

Si el Gateway se está ejecutando en este mismo equipo, ese enlace abre inmediatamente la Control UI en el navegador. Si no funciona, inicia primero el Gateway: `openclaw gateway`.

<div id="dashboard-browser-control-ui">
  ## Panel de control (Control UI en el navegador)
</div>

El panel de control es la Control UI en el navegador para chat, configuración, nodos, sesiones y más.
Predeterminado en local: http://127.0.0.1:18789/
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
  │                           │    /__openclaw__/canvas/ (Canvas host)
  └───────────┬───────────────┘
              │
              ├─ Pi agent (RPC)
              ├─ CLI (openclaw …)
              ├─ Chat UI (SwiftUI)
              ├─ macOS app (OpenClaw.app)
              ├─ iOS node via Gateway WS + pairing
              └─ nodo Android vía Gateway WS + emparejamiento
```

La mayoría de las operaciones pasan a través del **Gateway** (`openclaw gateway`), un único proceso de larga duración que gestiona las conexiones de los canales y el plano de control WebSocket.

<div id="network-model">
  ## Modelo de red
</div>

* **Un Gateway por host (recomendado)**: es el único proceso al que se le permite mantener la sesión de WhatsApp Web. Si necesitas un bot de rescate o aislamiento estricto, ejecuta múltiples Gateways con perfiles y puertos aislados; consulta [Múltiples gateways](/es/gateway/multiple-gateways).
* **Loopback primero**: el WS del Gateway usa por defecto `ws://127.0.0.1:18789`.
  * El asistente ahora genera un token de gateway por defecto (incluso para loopback).
  * Para acceso mediante Tailnet, ejecuta `openclaw gateway --bind tailnet --token ...` (el token es obligatorio para enlaces que no sean de loopback).
* **Nodos**: se conectan al WebSocket del Gateway (LAN/tailnet/SSH según sea necesario); el puente TCP heredado está obsoleto/eliminado.
* **Host de Canvas**: servidor de archivos HTTP en `canvasHost.port` (por defecto `18793`), que sirve `/__openclaw__/canvas/` para WebViews del nodo; consulta [Configuración del Gateway](/es/gateway/configuration) (`canvasHost`).
* **Uso remoto**: túnel SSH o tailnet/VPN; consulta [Acceso remoto](/es/gateway/remote) y [Descubrimiento](/es/gateway/discovery).

<div id="features-high-level">
  ## Funciones (a alto nivel)
</div>

* 📱 **Integración con WhatsApp** — Usa Baileys para el protocolo WhatsApp Web
* ✈️ **Bot de Telegram** — MD y grupos vía grammY
* 🎮 **Bot de Discord** — MD y canales de servidores vía channels.discord.js
* 🧩 **Bot de Mattermost (complemento)** — Token de bot + eventos WebSocket
* 💬 **iMessage** — Integración local con la CLI `imsg` (macOS)
* 🤖 **Puente de agente** — Pi (modo RPC) con streaming de herramientas
* ⏱️ **Streaming + segmentación en bloques** — Detalles de streaming por bloques + streaming de borradores de Telegram ([/concepts/streaming](/es/concepts/streaming))
* 🧠 **Enrutamiento multiagente** — Redirige cuentas/pares de proveedores a agentes aislados (espacio de trabajo + sesiones por agente)
* 🔐 **Autenticación por suscripción** — Anthropic (Claude Pro/Max) + OpenAI (ChatGPT/Codex) vía OAuth
* 💬 **Sesiones** — Los chats directos se combinan en una `main` compartida (predeterminada); los grupos están aislados
* 👥 **Compatibilidad con chat de grupo** — Basado en menciones de forma predeterminada; el propietario puede alternar `/activation always|mention`
* 📎 **Compatibilidad con contenido multimedia** — Enviar y recibir imágenes, audio y documentos
* 🎤 **Notas de voz** — Hook de transcripción opcional
* 🖥️ **WebChat + app de macOS** — UI local + app en la barra de menús para operaciones y activación por voz
* 📱 **Nodo de iOS** — Se empareja como un nodo y expone una superficie de Canvas
* 📱 **Nodo de Android** — Se empareja como un nodo y expone Canvas + Chat + Cámara

Nota: se han eliminado las rutas heredadas de Claude/Codex/Gemini/Opencode; Pi es la única ruta de agente de programación.

<div id="quick-start">
  ## Inicio rápido
</div>

Requisito de entorno de ejecución: **Node.js ≥ 22**.

```bash
# Recommended: global install (npm/pnpm)
npm install -g openclaw@latest
# or: pnpm add -g openclaw@latest

# Onboard + install the service (launchd/systemd user service)
openclaw onboard --install-daemon

# Pair WhatsApp Web (shows QR)
openclaw channels login

# Gateway se ejecuta mediante el servicio después de la incorporación; la ejecución manual sigue siendo posible:
openclaw gateway --port 18789
```

Cambiar más adelante entre instalaciones con npm y con git es sencillo: instala la otra modalidad y ejecuta `openclaw doctor` para actualizar el punto de entrada del servicio Gateway.

Desde el código fuente (desarrollo):

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm ui:build # instala automáticamente las dependencias de UI en la primera ejecución
pnpm build
openclaw onboard --install-daemon
```

Si aún no tienes una instalación global, ejecuta el paso de configuración inicial mediante `pnpm openclaw ...` desde el repositorio.

Inicio rápido para varias instancias (opcional):

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

El archivo de configuración está en `~/.openclaw/openclaw.json`.

* Si **no haces nada**, OpenClaw usa el binario de Pi incluido en modo RPC con sesiones por remitente.
* Si quieres restringirlo, empieza por `channels.whatsapp.allowFrom` y, para grupos, las reglas de menciones.

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
  * [Ayuda](/es/help) ← *correcciones comunes + resolución de problemas*
  * [Configuración](/es/gateway/configuration)
  * [Ejemplos de configuración](/es/gateway/configuration-examples)
  * [Comandos slash](/es/tools/slash-commands)
  * [Enrutamiento multiagente](/es/concepts/multi-agent)
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
* Aplicaciones complementarias:
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

**OpenClaw = CLAW + TARDIS** — porque toda langosta espacial necesita una máquina para viajar en el tiempo y el espacio.

***

*&quot;En el fondo, todos estamos jugando con nuestros propios prompts.&quot;* — una IA, probablemente colocada de tantos tokens

<div id="credits">
  ## Créditos
</div>

* **Peter Steinberger** ([@steipete](https://twitter.com/steipete)) — Creador, susurrador de langostas
* **Mario Zechner** ([@badlogicc](https://twitter.com/badlogicgames)) — Creador para Pi, especialista en seguridad y pentesting
* **Clawd** — La langosta espacial que exigió un nombre mejor

<div id="core-contributors">
  ## Colaboradores principales
</div>

* **Maxim Vovshin** (@Hyaxia, 36747317+Hyaxia@users.noreply.github.com) — skill Blogwatcher
* **Nacho Iacovino** (@nachoiacovino, nacho.iacovino@gmail.com) — procesamiento de ubicaciones (Telegram y WhatsApp)

<div id="license">
  ## Licencia
</div>

MIT — Libre como una langosta en el océano 🦞

***

*&quot;Al final todos estamos jugando con nuestros propios prompts.&quot;* — Una IA, probablemente colocada de tokens