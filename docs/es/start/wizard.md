---
title: Asistente
summary: "Asistente de configuración inicial de la CLI: configuración guiada del Gateway, espacio de trabajo, canales y habilidades"
read_when:
  - Al ejecutar o configurar el asistente de configuración inicial
  - Al configurar una máquina nueva
---

<div id="onboarding-wizard-cli">
  # Asistente de configuración inicial (CLI)
</div>

El asistente de configuración inicial es la forma **recomendada** de configurar OpenClaw en macOS,
Linux o Windows (mediante WSL2; altamente recomendado).
Configura un Gateway local o una conexión a un Gateway remoto, además de canales, habilidades
y valores predeterminados del espacio de trabajo en un único flujo guiado.

Punto de entrada principal:

```bash
openclaw onboard
```

Forma más rápida de iniciar el primer chat: abre la Control UI (no necesitas configurar ningún canal). Ejecuta
`openclaw dashboard` y chatea en el navegador. Documentación: [Dashboard](/es/web/dashboard).

Reconfiguración posterior:

```bash
openclaw configure
```

Se recomienda configurar una clave de API de Brave Search para que el agente pueda usar `web_search`
(`web_fetch` funciona sin una clave). El camino más sencillo: `openclaw configure --section web`,
que guarda `tools.web.search.apiKey`. Documentación: [Herramientas web](/es/tools/web).

<div id="quickstart-vs-advanced">
  ## Inicio rápido vs Avanzado
</div>

El asistente comienza con **QuickStart** (valores predeterminados) frente a **Advanced** (control total).

**QuickStart** mantiene los valores predeterminados:

* Gateway local (loopback)
* Espacio de trabajo predeterminado (o espacio de trabajo existente)
* Puerto del Gateway **18789**
* Autenticación del Gateway mediante **Token** (generado automáticamente, incluso en loopback)
* Exposición de Tailscale **Off**
* Los mensajes directos (DM) de Telegram y WhatsApp usan de forma predeterminada **allowlist** (se te solicitará tu número de teléfono)

**Advanced** expone cada paso (modo, espacio de trabajo, Gateway, canales, daemon, habilidades).

<div id="what-the-wizard-does">
  ## Qué hace el asistente
</div>

El **modo local (predeterminado)** te guía paso a paso por:

* Modelo/autenticación (suscripción OpenAI Code (Codex) OAuth, clave de API de Anthropic (recomendado) o token de configuración (pegar), además de opciones para MiniMax/GLM/Moonshot/AI Gateway)
* Ubicación del espacio de trabajo + archivos de arranque
* Configuración del Gateway (puerto/bind/auth/tailscale)
* Proveedores (Telegram, WhatsApp, Discord, Google Chat, Mattermost (complemento), Signal)
* Instalación del daemon (LaunchAgent / unidad de usuario systemd)
* Comprobación de salud
* Habilidades (recomendado)

El **modo remoto** solo configura el cliente local para conectarse a un Gateway remoto.
**No** instala ni cambia nada en el host remoto.

Para añadir más agentes aislados (espacio de trabajo + sesiones + autenticación separados), usa:

```bash
openclaw agents add <name>
```

Consejo: `--json` **no** implica el modo no interactivo. Usa `--non-interactive` (y `--workspace`) para scripts.

<div id="flow-details-local">
  ## Detalles del flujo (local)
</div>

1. **Detección de configuración existente**
   * Si `~/.openclaw/openclaw.json` existe, elige **Conservar / Modificar / Restablecer**.
   * Volver a ejecutar el asistente **no** borra nada a menos que elijas explícitamente **Restablecer**
     (o pases `--reset`).
   * Si la configuración no es válida o contiene claves heredadas, el asistente se detiene y te pide
     que ejecutes `openclaw doctor` antes de continuar.
   * Restablecer usa `trash` (nunca `rm`) y ofrece ámbitos:
     * Solo configuración
     * Configuración + credenciales + sesiones
     * Restablecimiento completo (también elimina el espacio de trabajo)

2. **Modelo/Auth**
   * **Anthropic API key (recomendado)**: usa `ANTHROPIC_API_KEY` si está presente o solicita una clave y luego la guarda para uso del daemon.
   * **Anthropic OAuth (Claude Code CLI)**: en macOS el asistente comprueba el elemento del llavero &quot;Claude Code-credentials&quot; (elige &quot;Always Allow&quot; para que los inicios de launchd no se bloqueen); en Linux/Windows reutiliza `~/.claude/.credentials.json` si está presente.
   * **Anthropic token (pegar setup-token)**: ejecuta `claude setup-token` en cualquier máquina y luego pega el token (puedes darle un nombre; en blanco = predeterminado).
   * **OpenAI Code (suscripción Codex) (Codex CLI)**: si `~/.codex/auth.json` existe, el asistente puede reutilizarlo.
   * **OpenAI Code (suscripción Codex) (OAuth)**: flujo en el navegador; pega el `code#state`.
     * Establece `agents.defaults.model` en `openai-codex/gpt-5.2` cuando el modelo no está definido o es `openai/*`.
   * **OpenAI API key**: usa `OPENAI_API_KEY` si está presente o solicita una clave y luego la guarda en `~/.openclaw/.env` para que launchd pueda leerla.
   * **OpenCode Zen (proxy multi-modelo)**: solicita `OPENCODE_API_KEY` (o `OPENCODE_ZEN_API_KEY`, obténla en https://opencode.ai/auth).
   * **API key**: almacena la clave por ti.
   * **Vercel AI Gateway (proxy multi-modelo)**: solicita `AI_GATEWAY_API_KEY`.
   * Más detalles: [Vercel AI Gateway](/es/providers/vercel-ai-gateway)
   * **MiniMax M2.1**: la configuración se genera automáticamente.
   * Más detalles: [MiniMax](/es/providers/minimax)
   * **Synthetic (compatible con Anthropic)**: solicita `SYNTHETIC_API_KEY`.
   * Más detalles: [Synthetic](/es/providers/synthetic)
   * **Moonshot (Kimi K2)**: la configuración se genera automáticamente.
   * **Kimi Code**: la configuración se genera automáticamente.
   * Más detalles: [Moonshot AI (Kimi + Kimi Code)](/es/providers/moonshot)
   * **Omitir**: no se configura auth por ahora.
   * Elige un modelo predeterminado de entre las opciones detectadas (o introduce proveedor/modelo manualmente).
   * El asistente ejecuta una comprobación del modelo y avisa si el modelo configurado es desconocido o si falta auth.

* Las credenciales OAuth se almacenan en `~/.openclaw/credentials/oauth.json`; los perfiles de auth se almacenan en `~/.openclaw/agents/<agentId>/agent/auth-profiles.json` (API keys + OAuth).
  * Más detalles: [/concepts/oauth](/es/concepts/oauth)

3. **Workspace**
   * Predeterminado `~/.openclaw/workspace` (configurable).
   * Inicializa los archivos del espacio de trabajo necesarios para el ritual de arranque del agente.
   * Diseño completo del espacio de trabajo + guía de copia de seguridad: [Workspace del agente](/es/concepts/agent-workspace)

4. **Gateway**
   * Puerto, bind, modo de autenticación, exposición mediante Tailscale.
   * Recomendación de autenticación: mantén **Token** incluso para loopback para que los clientes WS locales deban autenticarse.
   * Desactiva la autenticación solo si confías completamente en todos los procesos locales.
   * Los binds que no son de loopback siguen requiriendo autenticación.

5. **Channels**
   * [WhatsApp](/es/channels/whatsapp): inicio de sesión opcional con QR.
   * [Telegram](/es/channels/telegram): token de bot.
   * [Discord](/es/channels/discord): token de bot.
   * [Google Chat](/es/channels/googlechat): JSON de cuenta de servicio + audiencia del webhook.
   * [Mattermost](/es/channels/mattermost) (plugin): token de bot + URL base.
   * [Signal](/es/channels/signal): instalación opcional de `signal-cli` + configuración de cuenta.
   * [iMessage](/es/channels/imessage): ruta local de la CLI `imsg` + acceso a la BD.
   * Seguridad de mensajes directos (DM): el valor predeterminado es el emparejamiento. El primer DM envía un código; apruébalo mediante `openclaw pairing approve <channel> <code>` o usa listas de permitidos.

6. **Daemon install**
   * macOS: LaunchAgent
     * Requiere una sesión de usuario iniciada; para entornos sin interfaz gráfica (headless), usa un LaunchDaemon personalizado (no incluido).
   * Linux (y Windows vía WSL2): unidad de usuario systemd
     * El asistente intenta habilitar lingering mediante `loginctl enable-linger <user>` para que el Gateway permanezca activo después de cerrar la sesión.
     * Puede solicitar sudo (escribe en `/var/lib/systemd/linger`); primero lo intenta sin sudo.
   * **Runtime selection:** Node (recomendado; requerido para WhatsApp/Telegram). Bun **no se recomienda**.

7. **Health check**
   * Inicia el Gateway (si es necesario) y ejecuta `openclaw health`.
   * Consejo: `openclaw status --deep` añade sondas de salud del Gateway a la salida de estado (requiere un Gateway accesible).

8. **Skills (recommended)**
   * Lee las habilidades disponibles y verifica los requisitos.
   * Te permite elegir un gestor de paquetes de Node: **npm / pnpm** (bun no recomendado).
   * Instala dependencias opcionales (algunas usan Homebrew en macOS).

9. **Finish**
   * Resumen + siguientes pasos, incluidas las apps de iOS/Android/macOS para funciones adicionales.

* Si no se detecta una GUI, el asistente imprime instrucciones de reenvío de puertos SSH para la Control UI en lugar de abrir un navegador.
  * Si faltan los recursos de la Control UI, el asistente intenta compilarlos; el modo de reserva es `pnpm ui:build` (instala automáticamente las dependencias de la UI).

<div id="remote-mode">
  ## Modo remoto
</div>

El modo remoto configura un cliente local para conectarse a un Gateway ubicado en otro lugar.

Lo que vas a configurar:

* URL del Gateway remoto (`ws://...`)
* Token si el Gateway remoto requiere autenticación (recomendado)

Notas:

* No se realizan instalaciones remotas ni cambios en servicios en segundo plano.
* Si el Gateway solo está disponible por loopback, usa un túnel SSH o una tailnet.
* Sugerencias para el descubrimiento:
  * macOS: Bonjour (`dns-sd`)
  * Linux: Avahi (`avahi-browse`)

<div id="add-another-agent">
  ## Añadir otro agente
</div>

Usa `openclaw agents add <name>` para crear un agente independiente con su propio espacio de trabajo,
sesiones y perfiles de autenticación. Ejecutarlo sin `--workspace` inicia el asistente.

Qué establece:

* `agents.list[].name`
* `agents.list[].workspace`
* `agents.list[].agentDir`

Notas:

* Los espacios de trabajo predeterminados siguen el patrón `~/.openclaw/workspace-<agentId>`.
* Añade `bindings` para enrutar mensajes entrantes (el asistente puede hacer esto).
* Flags no interactivas: `--model`, `--agent-dir`, `--bind`, `--non-interactive`.

<div id="noninteractive-mode">
  ## Modo no interactivo
</div>

Utiliza `--non-interactive` para automatizar o usar en scripts el proceso de configuración inicial:

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice apiKey \
  --anthropic-api-key "$ANTHROPIC_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback \
  --install-daemon \
  --daemon-runtime node \
  --skip-skills
```

Añade `--json` para obtener un resumen en formato legible por máquina.

Ejemplo con Gemini:

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice gemini-api-key \
  --gemini-api-key "$GEMINI_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

Ejemplo de Z.AI:

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice zai-api-key \
  --zai-api-key "$ZAI_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

Ejemplo de Vercel AI Gateway:

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice ai-gateway-api-key \
  --ai-gateway-api-key "$AI_GATEWAY_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

Ejemplo con Moonshot:

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice moonshot-api-key \
  --moonshot-api-key "$MOONSHOT_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

Ejemplo de prueba:

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice synthetic-api-key \
  --synthetic-api-key "$SYNTHETIC_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

Ejemplo de OpenCode Zen:

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice opencode-zen \
  --opencode-zen-api-key "$OPENCODE_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

Ejemplo para agregar un agente (no interactivo):

```bash
openclaw agents add work \
  --workspace ~/.openclaw/workspace-work \
  --model openai/gpt-5.2 \
  --bind whatsapp:biz \
  --non-interactive \
  --json
```

<div id="gateway-wizard-rpc">
  ## RPC del asistente del Gateway
</div>

El Gateway expone el flujo del asistente a través de RPC (`wizard.start`, `wizard.next`, `wizard.cancel`, `wizard.status`).
Los clientes (aplicación de macOS, Control UI) pueden renderizar los pasos sin tener que volver a implementar la lógica de incorporación.

<div id="signal-setup-signal-cli">
  ## Configuración de Signal (signal-cli)
</div>

El asistente puede instalar `signal-cli` desde las releases de GitHub:

* Descarga el asset de la release apropiada.
* Lo almacena en `~/.openclaw/tools/signal-cli/<version>/`.
* Escribe `channels.signal.cliPath` en tu configuración.

Notas:

* Las compilaciones de JVM requieren **Java 21**.
* Se utilizan compilaciones nativas cuando están disponibles.
* Windows usa WSL2; la instalación de signal-cli sigue el flujo de instalación de Linux dentro de WSL.

<div id="what-the-wizard-writes">
  ## Lo que escribe el asistente
</div>

Campos típicos en `~/.openclaw/openclaw.json`:

* `agents.defaults.workspace`
* `agents.defaults.model` / `models.providers` (si se elige Minimax)
* `gateway.*` (mode, bind, auth, tailscale)
* `channels.telegram.botToken`, `channels.discord.token`, `channels.signal.*`, `channels.imessage.*`
* Listas de permitidos de canales (Slack/Discord/Matrix/Microsoft Teams) cuando decides activarlas durante los prompts (los nombres se resuelven a IDs cuando es posible).
* `skills.install.nodeManager`
* `wizard.lastRunAt`
* `wizard.lastRunVersion`
* `wizard.lastRunCommit`
* `wizard.lastRunCommand`
* `wizard.lastRunMode`

`openclaw agents add` escribe `agents.list[]` y `bindings` opcionales.

Las credenciales de WhatsApp se guardan en `~/.openclaw/credentials/whatsapp/<accountId>/`.
Las sesiones se almacenan en `~/.openclaw/agents/<agentId>/sessions/`.

Algunos canales se distribuyen como complementos. Cuando eliges uno durante el onboarding, el asistente
te pedirá que lo instales (npm o una ruta local) antes de poder configurarlo.

<div id="related-docs">
  ## Documentación relacionada
</div>

* Incorporación en la aplicación de macOS: [Onboarding](/es/start/onboarding)
* Referencia de configuración: [Configuración de Gateway](/es/gateway/configuration)
* Proveedores: [WhatsApp](/es/channels/whatsapp), [Telegram](/es/channels/telegram), [Discord](/es/channels/discord), [Google Chat](/es/channels/googlechat), [Signal](/es/channels/signal), [iMessage](/es/channels/imessage)
* Habilidades: [Habilidades](/es/tools/skills), [Configuración de habilidades](/es/tools/skills-config)