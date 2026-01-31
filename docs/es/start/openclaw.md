---
title: OpenClaw
summary: "Guía integral para ejecutar OpenClaw como asistente personal con advertencias de seguridad"
read_when:
  - Puesta en marcha de una nueva instancia de asistente
  - Revisión de las implicaciones de seguridad y permisos
---

<div id="building-a-personal-assistant-with-openclaw">
  # Cómo crear un asistente personal con OpenClaw
</div>

OpenClaw es un Gateway de WhatsApp + Telegram + Discord + iMessage para agentes **Pi**. Los complementos añaden Mattermost. Esta guía describe la configuración de «asistente personal»: un número de WhatsApp dedicado que funciona como tu agente siempre activo.

<div id="safety-first">
  ## ⚠️ Primero, la seguridad
</div>

Estás poniendo a un agente en una posición en la que puede:

* ejecutar comandos en tu máquina (según la configuración de tus herramientas en la Raspberry Pi)
* leer/escribir archivos en tu espacio de trabajo
* enviar mensajes hacia el exterior vía WhatsApp/Telegram/Discord/Mattermost (complemento)

Empieza de forma conservadora:

* Configura siempre `channels.whatsapp.allowFrom` (nunca ejecutes una configuración con la política `open`, que permite aceptar mensajes sin restricciones, en tu Mac personal).
* Usa un número de WhatsApp dedicado para el asistente.
* Los latidos ahora se ejecutan, por defecto, cada 30 minutos. Desactívalos hasta que confíes en la configuración estableciendo `agents.defaults.heartbeat.every: "0m"`.

<div id="prerequisites">
  ## Requisitos previos
</div>

* Node **22+**
* OpenClaw disponible en el PATH del sistema (recomendado: instalación global)
* Un segundo número de teléfono (SIM/eSIM/prepago) para el asistente

```bash
npm install -g openclaw@latest
# o bien: pnpm add -g openclaw@latest
```

Desde el código fuente (desarrollo):

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm ui:build # instala automáticamente las dependencias de la UI en la primera ejecución
pnpm build
pnpm link --global
```

<div id="the-two-phone-setup-recommended">
  ## Configuración con dos teléfonos (recomendada)
</div>

Esto es lo que quieres:

```
Your Phone (personal)          Second Phone (assistant)
┌─────────────────┐           ┌─────────────────┐
│  Your WhatsApp  │  ──────▶  │  Assistant WA   │
│  +1-555-YOU     │  message  │  +1-555-ASSIST  │
└─────────────────┘           └────────┬────────┘
                                       │ linked via QR
                                       ▼
                              ┌─────────────────┐
                              │  Your Mac       │
                              │  (openclaw)      │
                              │    Pi agent     │
                              └─────────────────┘
```

Si vinculas tu WhatsApp personal a OpenClaw, cada mensaje que recibas se convierte en “entrada para el agente”. Eso casi nunca es lo que quieres.

<div id="5-minute-quick-start">
  ## Guía rápida de 5 minutos
</div>

1. Vincula WhatsApp Web (muestra un código QR; escanéalo con el teléfono que usas como asistente):

```bash
openclaw channels login
```

2. Inicia el Gateway (déjalo en ejecución):

```bash
openclaw gateway --port 18789
```

3. Crea una configuración mínima en `~/.openclaw/openclaw.json`:

```json5
{
  channels: { whatsapp: { allowFrom: ["+15555550123"] } }
}
```

Ahora envía un mensaje al número del asistente desde tu teléfono incluido en la lista de permitidos.

Cuando termine el onboarding, se abrirá automáticamente el panel con tu token del Gateway y se mostrará el enlace con el token incrustado. Para volver a abrirlo más tarde: `openclaw dashboard`.

<div id="give-the-agent-a-workspace-agents">
  ## Dale al agente un espacio de trabajo (AGENTS)
</div>

OpenClaw lee instrucciones de funcionamiento y “memoria” desde su directorio de espacio de trabajo.

De forma predeterminada, OpenClaw usa `~/.openclaw/workspace` como el espacio de trabajo del agente y lo creará (junto con los archivos iniciales `AGENTS.md`, `SOUL.md`, `TOOLS.md`, `IDENTITY.md`, `USER.md`) automáticamente durante la configuración o en la primera ejecución del agente. `BOOTSTRAP.md` solo se crea cuando el espacio de trabajo es completamente nuevo (no debería volver a aparecer después de que lo elimines).

Consejo: trata esta carpeta como la “memoria” de OpenClaw y conviértela en un repositorio de Git (idealmente privado) para que tus archivos `AGENTS.md` y los archivos de memoria tengan copia de seguridad. Si Git está instalado, los espacios de trabajo completamente nuevos se inicializan automáticamente.

```bash
openclaw setup
```

Guía completa de diseño del espacio de trabajo y copias de seguridad: [Espacio de trabajo del Agente](/es/concepts/agent-workspace)
Flujo de trabajo de la memoria: [Memoria](/es/concepts/memory)

Opcional: puedes elegir un espacio de trabajo diferente con `agents.defaults.workspace` (soporta `~`).

```json5
{
  agent: {
    workspace: "~/.openclaw/workspace"
  }
}
```

Si ya distribuyes tus propios archivos de espacio de trabajo desde un repositorio, puedes desactivar por completo la creación de archivos iniciales:

```json5
{
  agent: {
    skipBootstrap: true
  }
}
```

<div id="the-config-that-turns-it-into-an-assistant">
  ## La configuración que lo convierte en “un asistente”
</div>

OpenClaw viene por defecto con una buena configuración de asistente, pero normalmente querrás ajustar:

* la personalidad/instrucciones en `SOUL.md`
* los valores predeterminados de razonamiento (si lo deseas)
* los latidos (una vez que confíes en él)

Ejemplo:

```json5
{
  logging: { level: "info" },
  agent: {
    model: "anthropic/claude-opus-4-5",
    workspace: "~/.openclaw/workspace",
    thinkingDefault: "high",
    timeoutSeconds: 1800,
    // Comienza en 0; habilitar más adelante.
    heartbeat: { every: "0m" }
  },
  channels: {
    whatsapp: {
      allowFrom: ["+15555550123"],
      groups: {
        "*": { requireMention: true }
      }
    }
  },
  routing: {
    groupChat: {
      mentionPatterns: ["@openclaw", "openclaw"]
    }
  },
  session: {
    scope: "per-sender",
    resetTriggers: ["/new", "/reset"],
    reset: {
      mode: "daily",
      atHour: 4,
      idleMinutes: 10080
    }
  }
}
```

<div id="sessions-and-memory">
  ## Sesiones y memoria
</div>

* Archivos de sesión: `~/.openclaw/agents/<agentId>/sessions/{{SessionId}}.jsonl`
* Metadatos de la sesión (uso de tokens, última ruta, etc.): `~/.openclaw/agents/<agentId>/sessions/sessions.json` (legacy: `~/.openclaw/sessions/sessions.json`)
* `/new` o `/reset` inicia una sesión nueva para ese chat (configurable mediante `resetTriggers`). Si se envía solo, el agente responde con un breve saludo para confirmar el reinicio.
* `/compact [instructions]` compacta el contexto de la sesión e informa del presupuesto de contexto restante.

<div id="heartbeats-proactive-mode">
  ## Latidos (modo proactivo)
</div>

De forma predeterminada, OpenClaw ejecuta un latido cada 30 minutos con el siguiente prompt:
`Lee HEARTBEAT.md si existe (contexto del espacio de trabajo). Síguelo estrictamente. No infieras ni repitas tareas antiguas de chats anteriores. Si nada requiere atención, responde HEARTBEAT_OK.`
Configura `agents.defaults.heartbeat.every: "0m"` para desactivarlo.

* Si `HEARTBEAT.md` existe pero está efectivamente vacío (solo líneas en blanco y encabezados de markdown como `# Heading`), OpenClaw omite la ejecución del latido para ahorrar llamadas a la API.
* Si el archivo no existe, el latido igualmente se ejecuta y el modelo decide qué hacer.
* Si el agente responde con `HEARTBEAT_OK` (opcionalmente con un breve texto de relleno; consulta `agents.defaults.heartbeat.ackMaxChars`), OpenClaw suprime el envío saliente para ese latido.
* Los latidos ejecutan turnos completos del agente: intervalos más cortos consumen más tokens.

```json5
{
  agent: {
    heartbeat: { every: "30m" }
  }
}
```

<div id="media-in-and-out">
  ## Contenido multimedia de entrada y salida
</div>

Los adjuntos entrantes (imágenes/audio/documentos) se pueden poner a disposición de tu comando mediante plantillas:

* `{{MediaPath}}` (ruta local de archivo temporal)
* `{{MediaUrl}}` (pseudo-URL)
* `{{Transcript}}` (si la transcripción de audio está habilitada)

Adjuntos salientes del agente: incluye `MEDIA:<path-or-url>` en una línea independiente (sin espacios). Ejemplo:

```
Aquí está la captura de pantalla.
MEDIA:/tmp/screenshot.png
```

OpenClaw los extrae y los envía como archivos multimedia junto con el texto.

<div id="operations-checklist">
  ## Lista de verificación de operaciones
</div>

```bash
openclaw status          # estado local (credenciales, sesiones, eventos en cola)
openclaw status --all    # diagnóstico completo (solo lectura, se puede copiar)
openclaw status --deep   # añade sondeos de salud del Gateway (Telegram + Discord)
openclaw health --json   # instantánea de salud del Gateway (WS)
```

Los logs se guardan en `/tmp/openclaw/` (por defecto: `openclaw-AAAA-MM-DD.log`).

<div id="next-steps">
  ## Próximos pasos
</div>

* WebChat: [WebChat](/es/web/webchat)
* Operaciones del Gateway: [Runbook del Gateway](/es/gateway)
* Cron + activaciones: [Cron jobs](/es/automation/cron-jobs)
* App de la barra de menús de macOS: [App de macOS de OpenClaw](/es/platforms/macos)
* App de nodo para iOS: [App de iOS](/es/platforms/ios)
* App de nodo para Android: [App de Android](/es/platforms/android)
* Estado de Windows: [Windows (WSL2)](/es/platforms/windows)
* Estado de Linux: [App de Linux](/es/platforms/linux)
* Seguridad: [Seguridad](/es/gateway/security)