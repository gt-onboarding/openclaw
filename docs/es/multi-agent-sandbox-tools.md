---
summary: "Sandbox y restricciones de herramientas por agente, precedencia y ejemplos"
title: Sandbox y herramientas multiagente
read_when: "Quieres sandbox por agente o políticas de permitir/denegar herramientas por agente en un Gateway multiagente."
status: active
---

<div id="multi-agent-sandbox-tools-configuration">
  # Configuración de sandbox y herramientas para múltiples agentes
</div>

<div id="overview">
  ## Descripción general
</div>

Cada agente en una configuración multitagente ahora puede tener su propia:

* **Configuración de sandbox** (`agents.list[].sandbox` anula `agents.defaults.sandbox`)
* **Restricciones de herramientas** (`tools.allow` / `tools.deny`, más `agents.list[].tools`)

Esto te permite ejecutar múltiples agentes con diferentes perfiles de seguridad:

* Asistente personal con acceso completo
* Agentes familiares/laborales con herramientas restringidas
* Agentes orientados al público en sandboxes

`setupCommand` pertenece a `sandbox.docker` (global o por agente) y se ejecuta una vez
cuando se crea el contenedor.

La autenticación es por agente: cada agente lee desde su propio almacén de autenticación `agentDir` en:

```
~/.openclaw/agents/<agentId>/agent/auth-profiles.json
```

Las credenciales **no** se comparten entre agentes. Nunca reutilices `agentDir` entre agentes.
Si quieres compartir credenciales, copia `auth-profiles.json` en el `agentDir` del otro agente.

Para entender cómo se comporta el sandbox en tiempo de ejecución, consulta [Sandboxing](/es/gateway/sandboxing).
Para depurar casos de «¿por qué está bloqueado esto?», consulta [Sandbox vs Tool Policy vs Elevated](/es/gateway/sandbox-vs-tool-policy-vs-elevated) y `openclaw sandbox explain`.

***

<div id="configuration-examples">
  ## Ejemplos de configuración
</div>

<div id="example-1-personal-restricted-family-agent">
  ### Ejemplo 1: Agente personal + familiar con acceso restringido
</div>

```json
{
  "agents": {
    "list": [
      {
        "id": "main",
        "default": true,
        "name": "Personal Assistant",
        "workspace": "~/.openclaw/workspace",
        "sandbox": { "mode": "off" }
      },
      {
        "id": "family",
        "name": "Family Bot",
        "workspace": "~/.openclaw/workspace-family",
        "sandbox": {
          "mode": "all",
          "scope": "agent"
        },
        "tools": {
          "allow": ["read"],
          "deny": ["exec", "write", "edit", "apply_patch", "process", "browser"]
        }
      }
    ]
  },
  "bindings": [
    {
      "agentId": "family",
      "match": {
        "provider": "whatsapp",
        "accountId": "*",
        "peer": {
          "kind": "group",
          "id": "120363424282127706@g.us"
        }
      }
    }
  ]
}
```

**Resultado:**

* Agente `main`: se ejecuta en el host, acceso completo a herramientas
* Agente `family`: se ejecuta en Docker (un contenedor por agente), solo la herramienta `read`

***

<div id="example-2-work-agent-with-shared-sandbox">
  ### Ejemplo 2: Agente de trabajo con sandbox compartida
</div>

```json
{
  "agents": {
    "list": [
      {
        "id": "personal",
        "workspace": "~/.openclaw/workspace-personal",
        "sandbox": { "mode": "off" }
      },
      {
        "id": "work",
        "workspace": "~/.openclaw/workspace-work",
        "sandbox": {
          "mode": "all",
          "scope": "shared",
          "workspaceRoot": "/tmp/work-sandboxes"
        },
        "tools": {
          "allow": ["read", "write", "apply_patch", "exec"],
          "deny": ["browser", "gateway", "discord"]
        }
      }
    ]
  }
}
```

***

<div id="example-2b-global-coding-profile-messaging-only-agent">
  ### Ejemplo 2b: Perfil global de codificación + agente solo para mensajería
</div>

```json
{
  "tools": { "profile": "coding" },
  "agents": {
    "list": [
      {
        "id": "support",
        "tools": { "profile": "messaging", "allow": ["slack"] }
      }
    ]
  }
}
```

**Resultado:**

* los agentes predeterminados reciben herramientas de programación
* el agente `support` es solo para mensajería (+ herramienta de Slack)

***

<div id="example-3-different-sandbox-modes-per-agent">
  ### Ejemplo 3: Diferentes modos de sandbox por Agente
</div>

```json
{
  "agents": {
    "defaults": {
      "sandbox": {
        "mode": "non-main",  // Predeterminado global
        "scope": "session"
      }
    },
    "list": [
      {
        "id": "main",
        "workspace": "~/.openclaw/workspace",
        "sandbox": {
          "mode": "off"  // Anulación: main nunca en sandbox
        }
      },
      {
        "id": "public",
        "workspace": "~/.openclaw/workspace-public",
        "sandbox": {
          "mode": "all",  // Anulación: public siempre en sandbox
          "scope": "agent"
        },
        "tools": {
          "allow": ["read"],
          "deny": ["exec", "write", "edit", "apply_patch"]
        }
      }
    ]
  }
}
```

***

<div id="configuration-precedence">
  ## Prioridad de configuración
</div>

Cuando existan configuraciones tanto globales (`agents.defaults.*`) como específicas de agentes (`agents.list[].*`):

<div id="sandbox-config">
  ### Configuración de la sandbox
</div>

La configuración específica de un agente tiene prioridad sobre la global:

```
agents.list[].sandbox.mode > agents.defaults.sandbox.mode
agents.list[].sandbox.scope > agents.defaults.sandbox.scope
agents.list[].sandbox.workspaceRoot > agents.defaults.sandbox.workspaceRoot
agents.list[].sandbox.workspaceAccess > agents.defaults.sandbox.workspaceAccess
agents.list[].sandbox.docker.* > agents.defaults.sandbox.docker.*
agents.list[].sandbox.browser.* > agents.defaults.sandbox.browser.*
agents.list[].sandbox.prune.* > agents.defaults.sandbox.prune.*
```

**Notas:**

* `agents.list[].sandbox.{docker,browser,prune}.*` tiene prioridad sobre `agents.defaults.sandbox.{docker,browser,prune}.*` para ese agente (se ignora cuando el ámbito de la sandbox se resuelve como `"shared"`).

<div id="tool-restrictions">
  ### Restricciones de herramientas
</div>

El orden de filtrado es:

1. **Perfil de herramientas** (`tools.profile` o `agents.list[].tools.profile`)
2. **Perfil de herramientas del proveedor** (`tools.byProvider[provider].profile` o `agents.list[].tools.byProvider[provider].profile`)
3. **Política global de herramientas** (`tools.allow` / `tools.deny`)
4. **Política de herramientas del proveedor** (`tools.byProvider[provider].allow/deny`)
5. **Política de herramientas específica del agente** (`agents.list[].tools.allow/deny`)
6. **Política de proveedor del agente** (`agents.list[].tools.byProvider[provider].allow/deny`)
7. **Política de herramientas del sandbox** (`tools.sandbox.tools` o `agents.list[].tools.sandbox.tools`)
8. **Política de herramientas del subagente** (`tools.subagents.tools`, si corresponde)

Cada nivel puede restringir aún más las herramientas, pero no puede volver a permitir herramientas que hayan sido denegadas en niveles anteriores.
Si se establece `agents.list[].tools.sandbox.tools`, reemplaza `tools.sandbox.tools` para ese agente.
Si se establece `agents.list[].tools.profile`, reemplaza `tools.profile` para ese agente.
Las claves de herramientas del proveedor aceptan tanto `provider` (por ejemplo, `google-antigravity`) como `provider/model` (por ejemplo, `openai/gpt-5.2`).

<div id="tool-groups-shorthands">
  ### Grupos de herramientas (abreviaturas)
</div>

Las políticas de herramientas (globales, de agente, de sandbox) admiten entradas del tipo `group:*` que se expanden en varias herramientas específicas:

* `group:runtime`: `exec`, `bash`, `process`
* `group:fs`: `read`, `write`, `edit`, `apply_patch`
* `group:sessions`: `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status`
* `group:memory`: `memory_search`, `memory_get`
* `group:ui`: `browser`, `canvas`
* `group:automation`: `cron`, `gateway`
* `group:messaging`: `message`
* `group:nodes`: `nodes`
* `group:openclaw`: todas las herramientas integradas de OpenClaw (excluye complementos de proveedores)

<div id="elevated-mode">
  ### Modo elevado
</div>

`tools.elevated` es la base global (lista de permitidos basada en remitentes). `agents.list[].tools.elevated` puede restringir aún más los privilegios elevados para agentes específicos (ambos deben permitirlo).

Patrones de mitigación:

* Deniega `exec` para agentes no confiables (`agents.list[].tools.deny: ["exec"]`)
* Evita incluir en la lista de permitidos remitentes que se enrutan a agentes restringidos
* Desactiva el modo elevado globalmente (`tools.elevated.enabled: false`) si solo deseas ejecución en sandbox
* Desactiva el modo elevado por agente (`agents.list[].tools.elevated.enabled: false`) para perfiles sensibles

***

<div id="migration-from-single-agent">
  ## Migración desde un único agente
</div>

**Antes (agente único):**

```json
{
  "agents": {
    "defaults": {
      "workspace": "~/.openclaw/workspace",
      "sandbox": {
        "mode": "non-main"
      }
    }
  },
  "tools": {
    "sandbox": {
      "tools": {
        "allow": ["read", "write", "apply_patch", "exec"],
        "deny": []
      }
    }
  }
}
```

**Después (varios agentes con perfiles diferentes):**

```json
{
  "agents": {
    "list": [
      {
        "id": "main",
        "default": true,
        "workspace": "~/.openclaw/workspace",
        "sandbox": { "mode": "off" }
      }
    ]
  }
}
```

Las configuraciones heredadas `agent.*` las migra `openclaw doctor`; en adelante, utiliza `agents.defaults` + `agents.list`.

***

<div id="tool-restriction-examples">
  ## Ejemplos de restricciones de herramientas
</div>

<div id="read-only-agent">
  ### Agente de solo lectura
</div>

```json
{
  "tools": {
    "allow": ["read"],
    "deny": ["exec", "write", "edit", "apply_patch", "process"]
  }
}
```

<div id="safe-execution-agent-no-file-modifications">
  ### Agente de ejecución segura (sin modificación de archivos)
</div>

```json
{
  "tools": {
    "allow": ["read", "exec", "process"],
    "deny": ["write", "edit", "apply_patch", "browser", "gateway"]
  }
}
```

<div id="communication-only-agent">
  ### Agente únicamente de comunicación
</div>

```json
{
  "tools": {
    "allow": ["sessions_list", "sessions_send", "sessions_history", "session_status"],
    "deny": ["exec", "write", "edit", "apply_patch", "read", "browser"]
  }
}
```

***

<div id="common-pitfall-non-main">
  ## Error común: &quot;non-main&quot;
</div>

`agents.defaults.sandbox.mode: "non-main"` se basa en `session.mainKey` (valor predeterminado `"main"`),
no en el ID del agente. Las sesiones de grupo/canal siempre obtienen sus propias claves, por lo que
se tratan como &quot;non-main&quot; y se ejecutarán en sandbox. Si quieres que un agente nunca use
sandbox, configura `agents.list[].sandbox.mode: "off"`.

***

<div id="testing">
  ## Pruebas
</div>

Después de configurar el sandbox multiagente y las herramientas:

1. **Comprueba la resolución de agentes:**
   ```exec
   openclaw agents list --bindings
   ```

2. **Verifica los contenedores del sandbox:**
   ```exec
   docker ps --filter "name=openclaw-sbx-"
   ```

3. **Prueba las restricciones de herramientas:**
   * Envía un mensaje que requiera herramientas restringidas
   * Verifica que el agente no pueda usar las herramientas denegadas

4. **Supervisa los logs:**
   ```exec
   tail -f "${OPENCLAW_STATE_DIR:-$HOME/.openclaw}/logs/gateway.log" | grep -E "routing|sandbox|tools"
   ```

***

<div id="troubleshooting">
  ## Resolución de problemas
</div>

<div id="agent-not-sandboxed-despite-mode-all">
  ### Agente sin sandbox a pesar de `mode: "all"`
</div>

* Comprueba si existe un `agents.defaults.sandbox.mode` global que lo anule
* La configuración específica del agente tiene prioridad, así que establece `agents.list[].sandbox.mode: "all"`

<div id="tools-still-available-despite-deny-list">
  ### Herramientas aún disponibles a pesar de la lista de denegación
</div>

* Comprueba el orden de filtrado de herramientas: global → agente → sandbox → subagente
* Cada nivel solo puede restringir más, no puede volver a conceder acceso
* Verifica en los registros: `[tools] filtering tools for agent:${agentId}`

<div id="container-not-isolated-per-agent">
  ### Contenedor no aislado por agente
</div>

* Configura `scope: "agent"` en la configuración de la sandbox específica del agente
* El valor predeterminado es `"session"`, lo que crea un contenedor por sesión

***

<div id="see-also">
  ## Ver también
</div>

* [Enrutamiento multiagente](/es/concepts/multi-agent)
* [Configuración de sandbox](/es/gateway/configuration#agentsdefaults-sandbox)
* [Gestión de sesiones](/es/concepts/session)