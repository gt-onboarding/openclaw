---
title: Grupos de difusión
summary: "Envía un mensaje de WhatsApp a varios agentes"
read_when:
  - Configurar grupos de difusión
  - Depurar respuestas de múltiples agentes en WhatsApp
status: experimental
---

<div id="broadcast-groups">
  # Grupos de difusión
</div>

**Estado:** Experimental\
**Versión:** Incorporado en la versión 2026.1.9

<div id="overview">
  ## Descripción general
</div>

Los Grupos de difusión permiten que varios agentes procesen y respondan al mismo mensaje al mismo tiempo. Esto te permite crear equipos de agentes especializados que trabajen juntos en un único grupo de WhatsApp o en un mensaje directo (DM), todos usando un solo número de teléfono.

Ámbito actual: **solo WhatsApp** (canal web).

Los grupos de difusión se evalúan después de las listas de permitidos del canal y de las reglas de activación de grupos. En los grupos de WhatsApp, esto significa que las difusiones ocurren cuando OpenClaw normalmente respondería (por ejemplo, al ser mencionado, según la configuración de tu grupo).

<div id="use-cases">
  ## Casos de uso
</div>

<div id="1-specialized-agent-teams">
  ### 1. Equipos de agentes especializados
</div>

Despliega varios agentes con responsabilidades atómicas y bien delimitadas:

```
Group: "Development Team"
Agents:
  - CodeReviewer (reviews code snippets)
  - DocumentationBot (generates docs)
  - SecurityAuditor (checks for vulnerabilities)
  - TestGenerator (suggests test cases)
```

Cada agente procesa el mismo mensaje y ofrece su perspectiva especializada.

<div id="2-multi-language-support">
  ### 2. Compatibilidad con varios idiomas
</div>

```
Group: "International Support"
Agents:
  - Agent_EN (responds in English)
  - Agent_DE (responds in German)
  - Agent_ES (responds in Spanish)
```

<div id="3-quality-assurance-workflows">
  ### 3. Flujos de trabajo de control de calidad
</div>

```
Group: "Customer Support"
Agents:
  - SupportAgent (provides answer)
  - QAAgent (reviews quality, only responds if issues found)
```

<div id="4-task-automation">
  ### 4. Automatización de tareas
</div>

```
Group: "Project Management"
Agents:
  - TaskTracker (updates task database)
  - TimeLogger (logs time spent)
  - ReportGenerator (creates summaries)
```

<div id="configuration">
  ## Configuración
</div>

<div id="basic-setup">
  ### Configuración básica
</div>

Añade una sección de nivel superior `broadcast` (junto a `bindings`). Las claves son identificadores de peers de WhatsApp:

* chats de grupo: JID de grupo (por ejemplo, `120363403215116621@g.us`)
* DMs (mensajes directos): número de teléfono en formato E.164 (por ejemplo, `+15551234567`)

```json
{
  "broadcast": {
    "120363403215116621@g.us": ["alfred", "baerbel", "assistant3"]
  }
}
```

**Resultado:** Cuando OpenClaw responda en este chat, ejecutará los tres agentes.

<div id="processing-strategy">
  ### Estrategia de procesamiento
</div>

Controla cómo procesan los mensajes los agentes:

<div id="parallel-default">
  #### Paralelo (por defecto)
</div>

Todos los agentes se ejecutan simultáneamente:

```json
{
  "broadcast": {
    "strategy": "parallel",
    "120363403215116621@g.us": ["alfred", "baerbel"]
  }
}
```

<div id="sequential">
  #### Secuencial
</div>

Los agentes se ejecutan en orden (cada uno espera a que el anterior termine):

```json
{
  "broadcast": {
    "strategy": "sequential",
    "120363403215116621@g.us": ["alfred", "baerbel"]
  }
}
```

<div id="complete-example">
  ### Ejemplo completo
</div>

```json
{
  "agents": {
    "list": [
      {
        "id": "code-reviewer",
        "name": "Code Reviewer",
        "workspace": "/path/to/code-reviewer",
        "sandbox": { "mode": "all" }
      },
      {
        "id": "security-auditor",
        "name": "Security Auditor",
        "workspace": "/path/to/security-auditor",
        "sandbox": { "mode": "all" }
      },
      {
        "id": "docs-generator",
        "name": "Documentation Generator",
        "workspace": "/path/to/docs-generator",
        "sandbox": { "mode": "all" }
      }
    ]
  },
  "broadcast": {
    "strategy": "parallel",
    "120363403215116621@g.us": ["code-reviewer", "security-auditor", "docs-generator"],
    "120363424282127706@g.us": ["support-en", "support-de"],
    "+15555550123": ["assistant", "logger"]
  }
}
```

<div id="how-it-works">
  ## Cómo funciona
</div>

<div id="message-flow">
  ### Flujo de mensajes
</div>

1. **Mensaje entrante** llega a un grupo de WhatsApp
2. **Verificación de broadcast**: el sistema comprueba si el ID del peer está en `broadcast`
3. **Si está en la lista de broadcast**:
   * Todos los agentes listados procesan el mensaje
   * Cada agente tiene su propia clave de sesión y un contexto aislado
   * Los agentes procesan en paralelo (predeterminado) o de forma secuencial
4. **Si no está en la lista de broadcast**:
   * Se aplica el enrutamiento normal (primer binding que coincida)

Nota: los grupos de broadcast no omiten las listas de permitidos del canal ni las reglas de activación del grupo (menciones/comandos/etc). Solo cambian *qué agentes se ejecutan* cuando un mensaje es elegible para ser procesado.

<div id="session-isolation">
  ### Aislamiento de sesiones
</div>

Cada agente en un grupo de difusión mantiene completamente separados:

* **Claves de sesión** (`agent:alfred:whatsapp:group:120363...` vs `agent:baerbel:whatsapp:group:120363...`)
* **Historial de conversación** (el agente no ve los mensajes de otros agentes)
* **Espacio de trabajo** (sandboxes separados si se configuran)
* **Acceso a herramientas** (listas de permitidos/denegados distintas)
* **Memoria/contexto** (IDENTITY.md, SOUL.md, etc. separados)
* **Búfer de contexto de grupo** (mensajes recientes del grupo usados como contexto) se comparte por peer, por lo que todos los agentes de difusión ven el mismo contexto cuando se activan

Esto permite que cada agente tenga:

* Personalidades diferentes
* Acceso a herramientas diferente (p. ej., solo lectura vs. lectura y escritura)
* Modelos diferentes (p. ej., opus vs. sonnet)
* Diferentes habilidades instaladas

<div id="example-isolated-sessions">
  ### Ejemplo: Sesiones aisladas
</div>

En el grupo `120363403215116621@g.us` con agentes `["alfred", "baerbel"]`:

**Contexto de Alfred:**

```
Session: agent:alfred:whatsapp:group:120363403215116621@g.us
History: [user message, alfred's previous responses]
Workspace: /Users/pascal/openclaw-alfred/
Tools: read, write, exec
```

**Contexto de Bärbel:**

```
Sesión: agent:baerbel:whatsapp:group:120363403215116621@g.us  
Historial: [mensaje del usuario, respuestas previas de baerbel]
Espacio de trabajo: /Users/pascal/openclaw-baerbel/
Herramientas: solo lectura
```

<div id="best-practices">
  ## Buenas prácticas
</div>

<div id="1-keep-agents-focused">
  ### 1. Mantén los agentes enfocados
</div>

Diseña cada agente con una responsabilidad única y clara:

```json
{
  "broadcast": {
    "DEV_GROUP": ["formatter", "linter", "tester"]
  }
}
```

✅ **Bueno:** Cada agente tiene una sola tarea
❌ **Malo:** Un único agente genérico «dev-helper»

<div id="2-use-descriptive-names">
  ### 2. Usa nombres descriptivos
</div>

Deja claro qué hace cada agente:

```json
{
  "agents": {
    "security-scanner": { "name": "Security Scanner" },
    "code-formatter": { "name": "Code Formatter" },
    "test-generator": { "name": "Test Generator" }
  }
}
```

<div id="3-configure-different-tool-access">
  ### 3. Configura distintos accesos a las herramientas
</div>

Dales a los agentes solo las herramientas que necesitan:

```json
{
  "agents": {
    "reviewer": {
      "tools": { "allow": ["read", "exec"] }  // Read-only
    },
    "fixer": {
      "tools": { "allow": ["read", "write", "edit", "exec"] }  // Lectura y escritura
    }
  }
}
```

<div id="4-monitor-performance">
  ### 4. Supervisar el rendimiento
</div>

Si tienes muchos agentes, ten en cuenta:

* Usar `"strategy": "parallel"` (valor predeterminado) para obtener mayor velocidad
* Limitar los grupos de difusión a 5-10 agentes
* Usar modelos más rápidos para agentes más simples

<div id="5-handle-failures-gracefully">
  ### 5. Gestiona los fallos de forma adecuada
</div>

Los agentes fallan de manera independiente. El error de un agente no bloquea a los demás:

```
Mensaje → [Agente A ✓, Agente B ✗ error, Agente C ✓]
Resultado: Los agentes A y C responden, el agente B registra el error
```

<div id="compatibility">
  ## Compatibilidad
</div>

<div id="providers">
  ### Proveedores
</div>

Actualmente, los grupos de difusión funcionan con:

* ✅ WhatsApp (implementado)
* 🚧 Telegram (previsto)
* 🚧 Discord (previsto)
* 🚧 Slack (previsto)

<div id="routing">
  ### Enrutamiento
</div>

Los grupos de difusión funcionan en conjunto con el enrutamiento existente:

```json
{
  "bindings": [
    { "match": { "channel": "whatsapp", "peer": { "kind": "group", "id": "GROUP_A" } }, "agentId": "alfred" }
  ],
  "broadcast": {
    "GROUP_B": ["agent1", "agent2"]
  }
}
```

* `GROUP_A`: Solo alfred responde (enrutamiento normal)
* `GROUP_B`: agent1 y agent2 responden (difusión)

**Precedencia:** `broadcast` tiene prioridad sobre `bindings`.

<div id="troubleshooting">
  ## Solución de problemas
</div>

<div id="agents-not-responding">
  ### Agentes que no responden
</div>

**Comprueba:**

1. Que los ID de los agentes existan en `agents.list`
2. Que el formato del Peer ID sea correcto (por ejemplo, `120363403215116621@g.us`)
3. Que los agentes no estén en listas de bloqueo

**Depurar:**

```bash
tail -f ~/.openclaw/logs/gateway.log | grep broadcast
```

<div id="only-one-agent-responding">
  ### Solo responde un agente
</div>

**Causa:** El ID de peer puede estar en `bindings` pero no en `broadcast`.

**Solución:** Agrégalo a la configuración de `broadcast` o elimínalo de `bindings`.

<div id="performance-issues">
  ### Problemas de rendimiento
</div>

**Si el sistema va lento con muchos agentes:**

* Reduce el número de agentes por grupo
* Usa modelos más ligeros (sonnet en lugar de opus)
* Comprueba el tiempo de arranque del sandbox

<div id="examples">
  ## Ejemplos
</div>

<div id="example-1-code-review-team">
  ### Ejemplo 1: Equipo de revisión de código
</div>

```json
{
  "broadcast": {
    "strategy": "parallel",
    "120363403215116621@g.us": [
      "code-formatter",
      "security-scanner",
      "test-coverage",
      "docs-checker"
    ]
  },
  "agents": {
    "list": [
      { "id": "code-formatter", "workspace": "~/agents/formatter", "tools": { "allow": ["read", "write"] } },
      { "id": "security-scanner", "workspace": "~/agents/security", "tools": { "allow": ["read", "exec"] } },
      { "id": "test-coverage", "workspace": "~/agents/testing", "tools": { "allow": ["read", "exec"] } },
      { "id": "docs-checker", "workspace": "~/agents/docs", "tools": { "allow": ["read"] } }
    ]
  }
}
```

**Usuario send:** Fragmento de código
**Respuestas:**

* code-formatter: &quot;Indentación corregida y se añadieron anotaciones de tipo&quot;
* security-scanner: &quot;⚠️ Vulnerabilidad de inyección SQL en la línea 12&quot;
* test-coverage: &quot;La cobertura es del 45 %, faltan pruebas para los casos de error&quot;
* docs-checker: &quot;Falta la docstring de la función `process_data`&quot;

<div id="example-2-multi-language-support">
  ### Ejemplo 2: Compatibilidad con varios idiomas
</div>

```json
{
  "broadcast": {
    "strategy": "sequential",
    "+15555550123": ["detect-language", "translator-en", "translator-de"]
  },
  "agents": {
    "list": [
      { "id": "detect-language", "workspace": "~/agents/lang-detect" },
      { "id": "translator-en", "workspace": "~/agents/translate-en" },
      { "id": "translator-de", "workspace": "~/agents/translate-de" }
    ]
  }
}
```

<div id="api-reference">
  ## Referencia de la API
</div>

<div id="config-schema">
  ### Esquema de configuración
</div>

```typescript
interface OpenClawConfig {
  broadcast?: {
    strategy?: "parallel" | "sequential";
    [peerId: string]: string[];
  };
}
```

<div id="fields">
  ### Campos
</div>

* `strategy` (opcional): Cómo procesar a los agentes
  * `"parallel"` (predeterminado): Todos los agentes se procesan simultáneamente
  * `"sequential"`: Los agentes se procesan en el orden del array

* `[peerId]`: JID de grupo de WhatsApp, número E.164 u otro ID de peer
  * Valor: Array de IDs de agentes que deben procesar mensajes

<div id="limitations">
  ## Limitaciones
</div>

1. **Máx. de agentes:** No hay un límite estricto, pero más de 10 agentes pueden ser lentos
2. **Contexto compartido:** Los agentes no ven las respuestas de los demás (por diseño)
3. **Orden de mensajes:** Las respuestas en paralelo pueden llegar en cualquier orden
4. **Límites de tasa:** Todos los agentes cuentan para los límites de tasa de WhatsApp

<div id="future-enhancements">
  ## Mejoras futuras
</div>

Funciones previstas:

* [ ] Modo de contexto compartido (los agentes pueden ver las respuestas de los demás)
* [ ] Coordinación entre agentes (los agentes pueden avisarse entre sí)
* [ ] Selección dinámica de agentes (seleccionar agentes según el contenido del mensaje)
* [ ] Prioridades de agentes (algunos agentes responden antes que otros)

<div id="see-also">
  ## Véase también
</div>

* [Configuración multiagente](/es/multi-agent-sandbox-tools)
* [Configuración de enrutamiento](/es/concepts/channel-routing)
* [Administración de sesiones](/es/concepts/sessions)