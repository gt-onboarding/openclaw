---
summary: "Enrutamiento multiagente: agentes aislados, cuentas de canal y asociaciones"
title: Enrutamiento multiagente
read_when: "Necesitas varios agentes aislados (espacios de trabajo + autenticación) en un solo proceso de Gateway."
status: active
---

<div id="multi-agent-routing">
  # Enrutamiento multiagente
</div>

Objetivo: múltiples agentes *aislados* (espacio de trabajo separado + `agentDir` + sesiones), además de múltiples cuentas de canal (por ejemplo, dos WhatsApps) en un único Gateway en ejecución. El tráfico entrante se enruta a un agente mediante bindings.

<div id="what-is-one-agent">
  ## ¿Qué es “un agente”?
</div>

Un **agente** es un cerebro con ámbito completo que tiene su propio:

* **Espacio de trabajo** (archivos, AGENTS.md/SOUL.md/USER.md, notas locales, reglas de la persona).
* **Directorio de estado** (`agentDir`) para perfiles de autenticación, registro de modelos y configuración por agente.
* **Almacenamiento de sesiones** (historial de chat + estado de enrutamiento) en `~/.openclaw/agents/<agentId>/sessions`.

Los perfiles de autenticación son **por agente**. Cada agente lee a partir de su propio:

```
~/.openclaw/agents/<agentId>/agent/auth-profiles.json
```

Las credenciales del agente principal **no** se comparten de forma automática. Nunca reutilices `agentDir`
entre agentes (provoca colisiones de autenticación/sesión). Si quieres compartir credenciales,
copia `auth-profiles.json` en el `agentDir` del otro agente.

Las habilidades se gestionan por agente mediante la carpeta `skills/` de cada espacio de trabajo, con habilidades
compartidas disponibles en `~/.openclaw/skills`. Consulta [Habilidades: por agente vs compartidas](/es/tools/skills#per-agent-vs-shared-skills).

El Gateway puede alojar **un agente** (predeterminado) o **muchos agentes** en paralelo.

**Nota sobre el espacio de trabajo:** el espacio de trabajo de cada agente es el **cwd predeterminado**, no un
sandbox estricto. Las rutas relativas se resuelven dentro del espacio de trabajo, pero las rutas absolutas pueden
alcanzar otras ubicaciones del host a menos que el sandboxing esté habilitado. Consulta
[Sandboxing](/es/gateway/sandboxing).

<div id="paths-quick-map">
  ## Rutas (mapa rápido)
</div>

* Configuración: `~/.openclaw/openclaw.json` (o `OPENCLAW_CONFIG_PATH`)
* Directorio de estado: `~/.openclaw` (o `OPENCLAW_STATE_DIR`)
* Espacio de trabajo: `~/.openclaw/workspace` (o `~/.openclaw/workspace-<agentId>`)
* Directorio del agente: `~/.openclaw/agents/<agentId>/agent` (o `agents.list[].agentDir`)
* Sesiones: `~/.openclaw/agents/<agentId>/sessions`

<div id="single-agent-mode-default">
  ### Modo de agente único (predeterminado)
</div>

Si no cambias nada, OpenClaw ejecuta un solo agente:

* `agentId` toma por defecto el valor **`main`**.
* Las sesiones usan claves de la forma `agent:main:<mainKey>`.
* De forma predeterminada, el espacio de trabajo es `~/.openclaw/workspace` (o `~/.openclaw/workspace-<profile>` cuando se establece `OPENCLAW_PROFILE`).
* De forma predeterminada, el estado se almacena en `~/.openclaw/agents/main/agent`.

<div id="agent-helper">
  ## Asistente de agente
</div>

Usa el asistente de agente para agregar un nuevo agente aislado:

```bash
openclaw agents add work
```

Luego añade `bindings` (o deja que el asistente lo haga) para enrutar los mensajes entrantes.

Verifica con:

```bash
openclaw agents list --bindings
```

<div id="multiple-agents-multiple-people-multiple-personalities">
  ## Varios agentes = múltiples personas, múltiples personalidades
</div>

Con **varios agentes**, cada `agentId` se convierte en una **persona completamente aislada**:

* **Distintos números de teléfono/cuentas** (por `accountId` de canal).
* **Diferentes personalidades** (archivos de espacio de trabajo específicos de cada agente como `AGENTS.md` y `SOUL.md`).
* **Autenticación y sesiones separadas** (sin comunicación cruzada a menos que se habilite explícitamente).

Esto permite que **varias personas** compartan un servidor Gateway manteniendo aislados sus “cerebros” de IA y sus datos.

<div id="one-whatsapp-number-multiple-people-dm-split">
  ## Un número de WhatsApp, varias personas (división de DM)
</div>

Puedes enrutar **diferentes DM de WhatsApp** a distintos agentes usando **una sola cuenta de WhatsApp**. Haz coincidir por remitente en formato E.164 (por ejemplo, `+15551234567`) con `peer.kind: "dm"`. Las respuestas siguen saliendo desde el mismo número de WhatsApp (no hay identidad de remitente por agente).

Detalle importante: los chats directos se consolidan en la **clave de sesión principal** del agente, así que el aislamiento real requiere **un agente por persona**.

Ejemplo:

```json5
{
  agents: {
    list: [
      { id: "alex", workspace: "~/.openclaw/workspace-alex" },
      { id: "mia", workspace: "~/.openclaw/workspace-mia" }
    ]
  },
  bindings: [
    { agentId: "alex", match: { channel: "whatsapp", peer: { kind: "dm", id: "+15551230001" } } },
    { agentId: "mia",  match: { channel: "whatsapp", peer: { kind: "dm", id: "+15551230002" } } }
  ],
  channels: {
    whatsapp: {
      dmPolicy: "allowlist",
      allowFrom: ["+15551230001", "+15551230002"]
    }
  }
}
```

Notas:

* El control de acceso a los mensajes directos (DM) es **global para cada cuenta de WhatsApp** (emparejamiento/lista de permitidos), no por agente.
* Para grupos compartidos, vincula el grupo a un solo agente o usa [Grupos de difusión](/es/broadcast-groups).

<div id="routing-rules-how-messages-pick-an-agent">
  ## Reglas de enrutamiento (cómo los mensajes eligen un agente)
</div>

Las asignaciones son **deterministas** y **gana la más específica**:

1. coincidencia de `peer` (ID exacto de DM/grupo/canal)
2. `guildId` (Discord)
3. `teamId` (Slack)
4. coincidencia de `accountId` para un canal
5. coincidencia a nivel de canal (`accountId: "*"`)
6. recurrir al agente predeterminado (`agents.list[].default`, en caso contrario, la primera entrada de la lista; valor predeterminado: `main`)

<div id="multiple-accounts-phone-numbers">
  ## Varias cuentas / números de teléfono
</div>

Los canales que permiten **varias cuentas** (por ejemplo, WhatsApp) usan `accountId` para identificar
cada inicio de sesión. Cada `accountId` se puede enrutar a un agente diferente, de modo que un único servidor pueda alojar
varios números de teléfono sin mezclar sesiones.

<div id="concepts">
  ## Conceptos
</div>

* `agentId`: un “cerebro” (espacio de trabajo, autenticación por agente, almacén de sesión por agente).
* `accountId`: una instancia de cuenta de canal (p. ej., cuenta de WhatsApp `"personal"` vs `"biz"`).
* `binding`: enruta los mensajes entrantes a un `agentId` por `(channel, accountId, peer)` y, opcionalmente, IDs de guild/equipo.
* Los chats directos se asocian a `agent:<agentId>:<mainKey>` (principal de cada agente; `session.mainKey`).

<div id="example-two-whatsapps-two-agents">
  ## Ejemplo: dos WhatsApps → dos agentes
</div>

`~/.openclaw/openclaw.json` (JSON5):

```js
{
  agents: {
    list: [
      {
        id: "home",
        default: true,
        name: "Home",
        workspace: "~/.openclaw/workspace-home",
        agentDir: "~/.openclaw/agents/home/agent",
      },
      {
        id: "work",
        name: "Work",
        workspace: "~/.openclaw/workspace-work",
        agentDir: "~/.openclaw/agents/work/agent",
      },
    ],
  },

  // Deterministic routing: first match wins (most-specific first).
  bindings: [
    { agentId: "home", match: { channel: "whatsapp", accountId: "personal" } },
    { agentId: "work", match: { channel: "whatsapp", accountId: "biz" } },

    // Optional per-peer override (example: send a specific group to work agent).
    {
      agentId: "work",
      match: {
        channel: "whatsapp",
        accountId: "personal",
        peer: { kind: "group", id: "1203630...@g.us" },
      },
    },
  ],

  // Desactivado por defecto: la mensajería agente-a-agente debe habilitarse explícitamente y estar en la lista de permitidos.
  tools: {
    agentToAgent: {
      enabled: false,
      allow: ["home", "work"],
    },
  },

  channels: {
    whatsapp: {
      accounts: {
        personal: {
          // Optional override. Default: ~/.openclaw/credentials/whatsapp/personal
          // authDir: "~/.openclaw/credentials/whatsapp/personal",
        },
        biz: {
          // Optional override. Default: ~/.openclaw/credentials/whatsapp/biz
          // authDir: "~/.openclaw/credentials/whatsapp/biz",
        },
      },
    },
  },
}
```

<div id="example-whatsapp-daily-chat-telegram-deep-work">
  ## Ejemplo: chat diario en WhatsApp + trabajo profundo en Telegram
</div>

Divide por canal: enruta WhatsApp a un agente rápido para el día a día y Telegram a un agente Opus.

```json5
{
  agents: {
    list: [
      {
        id: "chat",
        name: "Everyday",
        workspace: "~/.openclaw/workspace-chat",
        model: "anthropic/claude-sonnet-4-5"
      },
      {
        id: "opus",
        name: "Deep Work",
        workspace: "~/.openclaw/workspace-opus",
        model: "anthropic/claude-opus-4-5"
      }
    ]
  },
  bindings: [
    { agentId: "chat", match: { channel: "whatsapp" } },
    { agentId: "opus", match: { channel: "telegram" } }
  ]
}
```

Notas:

* Si tienes varias cuentas para un canal, añade `accountId` al binding (por ejemplo `{ channel: "whatsapp", accountId: "personal" }`).
* Para enrutar un único DM/grupo a Opus mientras mantienes el resto en chat, añade un binding `match.peer` para ese peer; las coincidencias por peer siempre prevalecen sobre las reglas a nivel de canal.

<div id="example-same-channel-one-peer-to-opus">
  ## Ejemplo: mismo canal, un contacto hacia Opus
</div>

Mantén WhatsApp en el agente rápido, pero enruta un mensaje directo a Opus:

```json5
{
  agents: {
    list: [
      { id: "chat", name: "Everyday", workspace: "~/.openclaw/workspace-chat", model: "anthropic/claude-sonnet-4-5" },
      { id: "opus", name: "Deep Work", workspace: "~/.openclaw/workspace-opus", model: "anthropic/claude-opus-4-5" }
    ]
  },
  bindings: [
    { agentId: "opus", match: { channel: "whatsapp", peer: { kind: "dm", id: "+15551234567" } } },
    { agentId: "chat", match: { channel: "whatsapp" } }
  ]
}
```

Los enlaces entre peers siempre tienen prioridad, así que manténlos por encima de la regla global del canal.

<div id="family-agent-bound-to-a-whatsapp-group">
  ## Agente familiar vinculado a un grupo de WhatsApp
</div>

Vincula un agente familiar dedicado a un solo grupo de WhatsApp, con control de acceso por menciones
y una política de herramientas más restrictiva:

```json5
{
  agents: {
    list: [
      {
        id: "family",
        name: "Family",
        workspace: "~/.openclaw/workspace-family",
        identity: { name: "Family Bot" },
        groupChat: {
          mentionPatterns: ["@family", "@familybot", "@Family Bot"]
        },
        sandbox: {
          mode: "all",
          scope: "agent"
        },
        tools: {
          allow: ["exec", "read", "sessions_list", "sessions_history", "sessions_send", "sessions_spawn", "session_status"],
          deny: ["write", "edit", "apply_patch", "browser", "canvas", "nodes", "cron"]
        }
      }
    ]
  },
  bindings: [
    {
      agentId: "family",
      match: {
        channel: "whatsapp",
        peer: { kind: "group", id: "120363999999999999@g.us" }
      }
    }
  ]
}
```

Notas:

* Las listas de permitidos/denegados de herramientas son **herramientas**, no habilidades. Si una habilidad necesita ejecutar un binario, asegúrate de que `exec` esté permitido y de que el binario exista en el sandbox.
* Para un control más estricto, configura `agents.list[].groupChat.mentionPatterns` y mantén las listas de permitidos de grupo habilitadas para el canal.

<div id="per-agent-sandbox-and-tool-configuration">
  ## Configuración de sandbox y herramientas por agente
</div>

A partir de la versión 2026.1.6, cada agente puede tener su propio sandbox y sus propias restricciones de herramientas:

```js
{
  agents: {
    list: [
      {
        id: "personal",
        workspace: "~/.openclaw/workspace-personal",
        sandbox: {
          mode: "off",  // No sandbox for personal agent
        },
        // No tool restrictions - all tools available
      },
      {
        id: "family",
        workspace: "~/.openclaw/workspace-family",
        sandbox: {
          mode: "all",     // Always sandboxed
          scope: "agent",  // One container per agent
          docker: {
            // Optional one-time setup after container creation
            setupCommand: "apt-get update && apt-get install -y git curl",
          },
        },
        tools: {
          allow: ["read"],                    // Only read tool
          deny: ["exec", "write", "edit", "apply_patch"],    // Denegar las demás
        },
      },
    ],
  },
}
```

Nota: `setupCommand` se define dentro de `sandbox.docker` y se ejecuta una vez al crear el contenedor.
Las anulaciones `sandbox.docker.*` específicas de cada agente se ignoran cuando el ámbito resuelto es `"shared"`.

**Beneficios:**

* **Aislamiento de seguridad**: Restringe herramientas para agentes no confiables
* **Control de recursos**: Aísla en sandbox a agentes específicos mientras mantienes otros en el host
* **Políticas flexibles**: Permisos diferentes por agente

Nota: `tools.elevated` es **global** y depende del remitente; no es configurable por agente.
Si necesitas límites por agente, usa `agents.list[].tools` para denegar `exec`.
Para la segmentación por grupo, usa `agents.list[].groupChat.mentionPatterns` para que las @menciones se asignen claramente al agente destinado.

Consulta [Multi-Agent Sandbox &amp; Tools](/es/multi-agent-sandbox-tools) para ver ejemplos detallados.
