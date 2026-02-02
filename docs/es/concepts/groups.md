---
title: Grupos
summary: "Comportamiento de los chats de grupo en distintas plataformas (WhatsApp/Telegram/Discord/Slack/Signal/iMessage/Microsoft Teams)"
read_when:
  - Cambiar el comportamiento de los chats de grupo o el control de acceso por menciones
---

<div id="groups">
  # Grupos
</div>

OpenClaw gestiona los chats de grupo de manera uniforme en todas las plataformas: WhatsApp, Telegram, Discord, Slack, Signal, iMessage, Microsoft Teams.

<div id="beginner-intro-2-minutes">
  ## Introducción básica (2 minutos)
</div>

OpenClaw “vive” en tus propias cuentas de mensajería. No hay un bot de WhatsApp separado.
Si **tú** estás en un grupo, OpenClaw puede ver ese grupo y responder allí.

Comportamiento predeterminado:

* Los grupos están restringidos (`groupPolicy: "allowlist"`).
* Las respuestas requieren una mención, a menos que desactives explícitamente la limitación por mención.

Traducción: los remitentes en la lista de permitidos pueden activar OpenClaw mencionándolo.

> TL;DR
>
> * **Acceso por DM** se controla mediante `*.allowFrom`.
> * **Acceso a grupos** se controla mediante `*.groupPolicy` + listas de permitidos (`*.groups`, `*.groupAllowFrom`).
> * **Activación de respuestas** se controla con la limitación por mención (`requireMention`, `/activation`).

Flujo rápido (qué ocurre con un mensaje de grupo):

```
groupPolicy? disabled -> drop
groupPolicy? allowlist -> group allowed? no -> drop
requireMention? yes -> mentioned? no -> store for context only
otherwise -> reply
```

![Flujo de mensajes de grupo](/images/groups-flow.svg)

Si quieres...

| Objetivo                                                   | Qué configurar                                             |
| ---------------------------------------------------------- | ---------------------------------------------------------- |
| Permitir todos los grupos pero responder solo a @menciones | `groups: { "*": { requireMention: true } }`                |
| Desactivar todas las respuestas en grupos                  | `groupPolicy: "disabled"`                                  |
| Solo grupos específicos                                    | `groups: { "<group-id>": { ... } }` (sin clave `"*"`)      |
| Solo tú puedes activar al asistente en grupos              | `groupPolicy: "allowlist"`, `groupAllowFrom: ["+1555..."]` |

<div id="session-keys">
  ## Claves de sesión
</div>

* Las sesiones de grupo usan claves de sesión `agent:<agentId>:<channel>:group:<id>` (las salas/canales usan `agent:<agentId>:<channel>:channel:<id>`).
* Los temas de foro de Telegram añaden `:topic:<threadId>` al id de grupo para que cada tema tenga su propia sesión.
* Los chats directos usan la sesión principal (o una por remitente, si está configurado así).
* Se omiten los latidos para las sesiones de grupo.

<div id="pattern-personal-dms-public-groups-single-agent">
  ## Patrón: DMs personales + grupos públicos (un solo agente)
</div>

Sí — esto funciona bien si tu tráfico “personal” son **DMs** y tu tráfico “público” son **grupos**.

Por qué: en modo de agente único, las DMs suelen ir a la clave de **sesión** **main** (`agent:main:main`), mientras que los grupos siempre usan claves de sesión **non-main** (`agent:main:<channel>:group:<id>`). Si habilitas el sandbox con `mode: "non-main"`, esas sesiones de grupo se ejecutan en Docker mientras tu sesión principal de DM permanece en el host.

Esto te da un solo “cerebro” de agente (espacio de trabajo compartido + memoria), pero dos modos de ejecución:

* **DMs**: herramientas completas (host)
* **Grupos**: sandbox + herramientas restringidas (Docker)

> Si necesitas espacios de trabajo/personas realmente separados (lo “personal” y lo “público” nunca deben mezclarse), usa un segundo agente + bindings. Consulta [Multi-Agent Routing](/es/concepts/multi-agent).

Ejemplo (DMs en el host, grupos en sandbox + solo herramientas de mensajería):

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main", // groups/channels are non-main -> sandboxed
        scope: "session", // strongest isolation (one container per group/channel)
        workspaceAccess: "none"
      }
    }
  },
  tools: {
    sandbox: {
      tools: {
        // Si allow no está vacío, todo lo demás se bloquea (deny siempre tiene prioridad).
        allow: ["group:messaging", "group:sessions"],
        deny: ["group:runtime", "group:fs", "group:ui", "nodes", "cron", "gateway"]
      }
    }
  }
}
```

¿Quieres que «los grupos solo puedan ver la carpeta X» en lugar de «sin acceso al host»? Mantén `workspaceAccess: "none"` y monta en la sandbox únicamente las rutas incluidas en la lista de permitidos:

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main",
        scope: "session",
        workspaceAccess: "none",
        docker: {
          binds: [
            // rutaHost:rutaContenedor:modo
            "~/FriendsShared:/data:ro"
          ]
        }
      }
    }
  }
}
```

Relacionado:

* Claves de configuración y valores predeterminados: [Configuración del Gateway](/es/gateway/configuration#agentsdefaultssandbox)
* Depuración de por qué una herramienta está bloqueada: [Sandbox vs Tool Policy vs Elevated](/es/gateway/sandbox-vs-tool-policy-vs-elevated)
* Detalles de los bind mounts: [Sandboxing](/es/gateway/sandboxing#custom-bind-mounts)

<div id="display-labels">
  ## Etiquetas de visualización
</div>

* Las etiquetas de la UI usan `displayName` cuando está disponible, con el formato `<channel>:<token>`.
* `#room` está reservado para salas/canales; los chats de grupo usan `g-<slug>` (en minúsculas, espacios -&gt; `-`, conservar `#@+._-`).

<div id="group-policy">
  ## Política de grupo
</div>

Controla cómo se manejan los mensajes de grupo/sala por canal:

```json5
{
  channels: {
    whatsapp: {
      groupPolicy: "disabled", // "open" (permite mensajes de cualquier usuario) | "disabled" (deshabilitado) | "allowlist" (lista de permitidos)
      groupAllowFrom: ["+15551234567"]
    },
    telegram: {
      groupPolicy: "disabled",
      groupAllowFrom: ["123456789", "@username"]
    },
    signal: {
      groupPolicy: "disabled",
      groupAllowFrom: ["+15551234567"]
    },
    imessage: {
      groupPolicy: "disabled",
      groupAllowFrom: ["chat_id:123"]
    },
    msteams: {
      groupPolicy: "disabled",
      groupAllowFrom: ["user@org.com"]
    },
    discord: {
      groupPolicy: "allowlist",
      guilds: {
        "GUILD_ID": { channels: { help: { allow: true } } }
      }
    },
    slack: {
      groupPolicy: "allowlist",
      channels: { "#general": { allow: true } }
    },
    matrix: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["@owner:example.org"],
      groups: {
        "!roomId:example.org": { allow: true },
        "#alias:example.org": { allow: true }
      }
    }
  }
}
```

| Policy        | Behavior                                                                               |
| ------------- | -------------------------------------------------------------------------------------- |
| `"open"`      | Los grupos omiten la lista de permitidos; el filtrado por menciones sigue aplicándose. |
| `"disabled"`  | Bloquea por completo todos los mensajes de grupo.                                      |
| `"allowlist"` | Solo permite grupos/salas que coincidan con la lista de permitidos configurada.        |

Notas:

* `groupPolicy` es independiente del filtrado por menciones (que requiere @menciones).
* WhatsApp/Telegram/Signal/iMessage/Microsoft Teams: usa `groupAllowFrom` (como respaldo: `allowFrom` explícito).
* Discord: la lista de permitidos usa `channels.discord.guilds.<id>.channels`.
* Slack: la lista de permitidos usa `channels.slack.channels`.
* Matrix: la lista de permitidos usa `channels.matrix.groups` (IDs de sala, alias o nombres). Usa `channels.matrix.groupAllowFrom` para restringir remitentes; también se admiten listas de permitidos de `users` por sala.
* Los DM de grupo se controlan por separado (`channels.discord.dm.*`, `channels.slack.dm.*`).
* La lista de permitidos de Telegram puede coincidir con IDs de usuario (`"123456789"`, `"telegram:123456789"`, `"tg:123456789"`) o nombres de usuario (`"@alice"` o `"alice"`); los prefijos no distinguen mayúsculas y minúsculas.
* El valor predeterminado es `groupPolicy: "allowlist"`; si tu lista de permitidos de grupos está vacía, los mensajes de grupo se bloquean.

Modelo mental rápido (orden de evaluación para mensajes de grupo):

1. `groupPolicy` (open/disabled/allowlist)
2. Listas de permitidos de grupos (`*.groups`, `*.groupAllowFrom`, lista de permitidos específica del canal)
3. Filtrado por menciones (`requireMention`, `/activation`)

<div id="mention-gating-default">
  ## Restricción por menciones (predeterminado)
</div>

Los mensajes de grupo requieren una mención, a menos que se sobrescriba esta configuración por grupo. Los valores predeterminados se definen por subsistema en `*.groups."*"`.

Responder a un mensaje de un bot cuenta como una mención implícita (cuando el canal admite metadatos de respuesta). Esto se aplica a Telegram, WhatsApp, Slack, Discord y Microsoft Teams.

```json5
{
  channels: {
    whatsapp: {
      groups: {
        "*": { requireMention: true },
        "123@g.us": { requireMention: false }
      }
    },
    telegram: {
      groups: {
        "*": { requireMention: true },
        "123456789": { requireMention: false }
      }
    },
    imessage: {
      groups: {
        "*": { requireMention: true },
        "123": { requireMention: false }
      }
    }
  },
  agents: {
    list: [
      {
        id: "main",
        groupChat: {
          mentionPatterns: ["@openclaw", "openclaw", "\\+15555550123"],
          historyLimit: 50
        }
      }
    ]
  }
}
```

Notas:

* `mentionPatterns` son expresiones regulares que no distinguen mayúsculas/minúsculas.
* Las superficies/canales que proporcionan menciones explícitas siguen pasando; los patrones son un mecanismo de respaldo.
* Sobrescritura por agente: `agents.list[].groupChat.mentionPatterns` (útil cuando varios agentes comparten un grupo).
* La restricción basada en menciones solo se aplica cuando la detección de menciones es posible (cuando hay menciones nativas o se han configurado `mentionPatterns`).
* Los valores predeterminados de Discord se encuentran en `channels.discord.guilds."*"` (se pueden sobrescribir por servidor/canal).
* El contexto de historial de grupo se envuelve de forma uniforme en todos los canales y aplica **solo a pendientes** (mensajes omitidos debido a la restricción por mención); usa `messages.groupChat.historyLimit` para el valor predeterminado global y `channels.<channel>.historyLimit` (o `channels.<channel>.accounts.*.historyLimit`) para las sobrescrituras. Establece `0` para desactivarlo.

<div id="groupchannel-tool-restrictions-optional">
  ## Restricciones de herramientas por grupo/canal (opcional)
</div>

Algunas configuraciones de canales permiten restringir qué herramientas están disponibles **dentro de un grupo/sala/canal específico**.

* `tools`: permite/deniega herramientas para todo el grupo.
* `toolsBySender`: anulaciones por remitente dentro del grupo (las claves son IDs de remitente/nombres de usuario/correos electrónicos/números de teléfono, según el canal). Usa `"*"` como comodín.

Orden de resolución (gana la más específica):

1. coincidencia de `toolsBySender` del grupo/canal
2. `tools` del grupo/canal
3. coincidencia de `toolsBySender` predeterminada (`"*"`)
4. `tools` predeterminada (`"*"`)

Ejemplo (Telegram):

```json5
{
  channels: {
    telegram: {
      groups: {
        "*": { tools: { deny: ["exec"] } },
        "-1001234567890": {
          tools: { deny: ["exec", "read", "write"] },
          toolsBySender: {
            "123456789": { alsoAllow: ["exec"] }
          }
        }
      }
    }
  }
}
```

Notas:

* Las restricciones de herramientas por grupo/canal se aplican además de la política global/de agente de herramientas (una denegación sigue teniendo prioridad).
* Algunos canales utilizan una anidación diferente para salas/canales (por ejemplo, Discord `guilds.*.channels.*`, Slack `channels.*`, MS Teams `teams.*.channels.*`).

<div id="group-allowlists">
  ## Listas de permitidos para grupos
</div>

Cuando se configura `channels.whatsapp.groups`, `channels.telegram.groups` o `channels.imessage.groups`, las claves actúan como una lista de permitidos para grupos. Usa `"*"` para permitir todos los grupos y, al mismo tiempo, establecer el comportamiento predeterminado de menciones.

Patrones habituales (copiar/pegar):

1. Desactivar todas las respuestas en grupos

```json5
{
  channels: { whatsapp: { groupPolicy: "disabled" } }
}
```

2. Permitir únicamente grupos específicos (WhatsApp)

```json5
{
  channels: {
    whatsapp: {
      groups: {
        "123@g.us": { requireMention: true },
        "456@g.us": { requireMention: false }
      }
    }
  }
}
```

3. Permitir todos los grupos, pero exigir mención explícita

```json5
{
  channels: {
    whatsapp: {
      groups: { "*": { requireMention: true } }
    }
  }
}
```

4. Solo el propietario puede activar el bot en grupos (WhatsApp)

```json5
{
  channels: {
    whatsapp: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15551234567"],
      groups: { "*": { requireMention: true } }
    }
  }
}
```

<div id="activation-owner-only">
  ## Activación (solo para el propietario)
</div>

Los propietarios de grupos pueden activar o desactivar la activación por grupo:

* `/activation mention`
* `/activation always`

El propietario se determina mediante `channels.whatsapp.allowFrom` (o el propio número E.164 del bot cuando no se ha configurado). Envía el comando como un mensaje independiente. Otras interfaces actualmente ignoran `/activation`.

<div id="context-fields">
  ## Campos de contexto
</div>

En los payloads entrantes de grupo se establecen:

* `ChatType=group`
* `GroupSubject` (si se conoce)
* `GroupMembers` (si se conoce)
* `WasMentioned` (resultado del control de acceso por mención)
* Los temas de foros de Telegram también incluyen `MessageThreadId` e `IsForum`.

El prompt de sistema del agente incluye una introducción al grupo en el primer turno de una nueva sesión de grupo. Le recuerda al modelo que responda como un humano, que evite usar tablas en Markdown y que evite escribir secuencias `\n` literales.

<div id="imessage-specifics">
  ## Detalles de iMessage
</div>

* Usa preferentemente `chat_id:<id>` al enrutar o al usar la lista de permitidos.
* Enumera los chats con: `imsg chats --limit 20`.
* Las respuestas de grupo siempre vuelven al mismo `chat_id`.

<div id="whatsapp-specifics">
  ## Aspectos específicos de WhatsApp
</div>

Consulta [Mensajes de grupo](/es/concepts/group-messages) para conocer el comportamiento específico de WhatsApp (inyección de historial, detalles sobre el manejo de menciones).