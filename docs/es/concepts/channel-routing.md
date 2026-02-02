---
title: Enrutamiento de canales
summary: "Reglas de enrutamiento por canal (WhatsApp, Telegram, Discord, Slack) y contexto compartido"
read_when:
  - Al cambiar el enrutamiento de canales o el comportamiento de la bandeja de entrada
---

<div id="channels-routing">
  # Canales y enrutamiento
</div>

OpenClaw enruta las respuestas **al canal del que provino el mensaje**. El
modelo no elige un canal; el enrutamiento es determinista y está controlado por la
configuración del host.

<div id="key-terms">
  ## Términos clave
</div>

* **Canal**: `whatsapp`, `telegram`, `discord`, `slack`, `signal`, `imessage`, `webchat`.
* **AccountId**: instancia de cuenta específica de cada canal (cuando sea compatible).
* **AgentId**: un espacio de trabajo + almacén de sesiones aislado (“cerebro”).
* **SessionKey**: la clave de bucket utilizada para almacenar contexto y controlar la concurrencia.

<div id="session-key-shapes-examples">
  ## Estructuras de claves de sesión (ejemplos)
</div>

Los mensajes directos se consolidan en la **sesión principal** del agente:

* `agent:<agentId>:<mainKey>` (predeterminado: `agent:main:main`)

Los grupos y canales permanecen aislados por canal:

* Grupos: `agent:<agentId>:<channel>:group:<id>`
* Canales/salas: `agent:<agentId>:<channel>:channel:<id>`

Hilos:

* Los hilos de Slack/Discord añaden `:thread:<threadId>` a la clave base.
* Los temas de foro de Telegram incluyen `:topic:<topicId>` en la clave del grupo.

Ejemplos:

* `agent:main:telegram:group:-1001234567890:topic:42`
* `agent:main:discord:channel:123456:thread:987654`

<div id="routing-rules-how-an-agent-is-chosen">
  ## Reglas de enrutamiento (cómo se elige un agente)
</div>

El enrutamiento selecciona **un agente** para cada mensaje entrante:

1. **Coincidencia exacta de peer** (`bindings` con `peer.kind` + `peer.id`).
2. **Coincidencia de guild** (Discord) mediante `guildId`.
3. **Coincidencia de equipo** (Slack) mediante `teamId`.
4. **Coincidencia de cuenta** (`accountId` en el canal).
5. **Coincidencia de canal** (cualquier cuenta en ese canal).
6. **Agente predeterminado** (`agents.list[].default`, si no, la primera entrada de la lista y, en última instancia, `main`).

El agente seleccionado determina qué espacio de trabajo y qué almacenamiento de sesiones se usan.

<div id="broadcast-groups-run-multiple-agents">
  ## Grupos de difusión (ejecutar varios agentes)
</div>

Los grupos de difusión te permiten ejecutar **varios agentes** para el mismo interlocutor **cuando OpenClaw normalmente respondería** (por ejemplo: en grupos de WhatsApp, después de la mención/activación con controles de acceso).

Config:

```json5
{
  broadcast: {
    strategy: "parallel",
    "120363403215116621@g.us": ["alfred", "baerbel"],
    "+15555550123": ["support", "logger"]
  }
}
```

Consulta: [Grupos de difusión](/es/broadcast-groups).

<div id="config-overview">
  ## Resumen de la configuración
</div>

* `agents.list`: definiciones de agentes con nombre (espacio de trabajo, modelo, etc.).
* `bindings`: asigna canales/cuentas/pares entrantes a agentes.

Ejemplo:

```json5
{
  agents: {
    list: [
      { id: "support", name: "Support", workspace: "~/.openclaw/workspace-support" }
    ]
  },
  bindings: [
    { match: { channel: "slack", teamId: "T123" }, agentId: "support" },
    { match: { channel: "telegram", peer: { kind: "group", id: "-100123" } }, agentId: "support" }
  ]
}
```

<div id="session-storage">
  ## Almacenamiento de sesiones
</div>

Los almacenes de sesión se encuentran bajo el directorio de estado (predeterminado `~/.openclaw`):

* `~/.openclaw/agents/<agentId>/sessions/sessions.json`
* Las transcripciones JSONL se guardan junto al almacén

Puedes anular la ruta del almacén mediante `session.store` y la plantilla `{agentId}`.

<div id="webchat-behavior">
  ## Comportamiento de WebChat
</div>

WebChat se conecta al **agente seleccionado** y, por defecto, usa la sesión principal de ese agente. Gracias a esto, WebChat te permite ver el contexto entre canales para ese agente en un único lugar.

<div id="reply-context">
  ## Contexto de respuesta
</div>

Las respuestas entrantes incluyen:

* `ReplyToId`, `ReplyToBody` y `ReplyToSender` cuando están disponibles.
* El contexto citado se agrega a `Body` como un bloque `[Replying to ...]`.

Esto es coherente en todos los canales.