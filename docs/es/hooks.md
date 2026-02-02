---
title: Hooks
summary: "Hooks: automatización basada en eventos para comandos y eventos del ciclo de vida"
read_when:
  - Quieres automatización basada en eventos para /new, /reset, /stop y eventos del ciclo de vida de agentes
  - Quieres desarrollar, instalar o depurar hooks
---

<div id="hooks">
  # Hooks
</div>

Los hooks proporcionan un sistema extensible impulsado por eventos para automatizar acciones en respuesta a comandos y eventos de los agentes. Los hooks se detectan automáticamente en directorios y se pueden gestionar mediante comandos de la CLI, de forma similar a cómo funcionan las habilidades en OpenClaw.

<div id="getting-oriented">
  ## Orientación inicial
</div>

Los hooks son pequeños scripts que se ejecutan cuando ocurre algo. Hay dos tipos:

* **Hooks** (esta página): se ejecutan dentro del Gateway cuando se activan eventos del agente, como `/new`, `/reset`, `/stop` o eventos de ciclo de vida.
* **Webhooks**: webhooks HTTP externos que permiten que otros sistemas ejecuten trabajo en OpenClaw. Consulta [Webhook Hooks](/es/automation/webhook) o usa `openclaw webhooks` para comandos auxiliares de Gmail.

Los hooks también se pueden empaquetar dentro de complementos; consulta [Plugins](/es/plugin#plugin-hooks).

Usos comunes:

* Guardar una instantánea de memoria cuando reinicias una sesión
* Mantener un registro de auditoría de los comandos para resolución de problemas o cumplimiento normativo
* Desencadenar automatización de seguimiento cuando una sesión comienza o termina
* Escribir archivos en el espacio de trabajo del agente o invocar APIs externas cuando se activan eventos

Si puedes escribir una pequeña función en TypeScript, puedes escribir un hook. Los hooks se detectan automáticamente y se habilitan o deshabilitan mediante la CLI.

<div id="overview">
  ## Descripción general
</div>

El sistema de hooks te permite:

* Guardar el contexto de la sesión en memoria cuando se ejecuta `/new`
* Registrar todos los comandos para fines de auditoría
* Activar automatizaciones personalizadas en eventos del ciclo de vida de los agentes
* Extender el comportamiento de OpenClaw sin modificar el código base

<div id="getting-started">
  ## Primeros pasos
</div>

<div id="bundled-hooks">
  ### Hooks incluidos
</div>

OpenClaw incluye cuatro hooks integrados que se detectan automáticamente:

* **💾 session-memory**: Guarda el contexto de la sesión en el espacio de trabajo de tu agente (predeterminado `~/.openclaw/workspace/memory/`) cuando ejecutas `/new`
* **📝 command-logger**: Registra todos los eventos de comando en `~/.openclaw/logs/commands.log`
* **🚀 boot-md**: Ejecuta `BOOT.md` cuando el Gateway se inicia (requiere hooks internos habilitados)
* **😈 soul-evil**: Sustituye el contenido inyectado de `SOUL.md` por `SOUL_EVIL.md` durante una ventana de purga o de forma aleatoria

Lista los hooks disponibles:

```bash
openclaw hooks list
```

Habilitar un hook:

```bash
openclaw hooks enable session-memory
```

Comprueba el estado del hook:

```bash
openclaw hooks check
```

Consulta información detallada:

```bash
openclaw hooks info session-memory
```

<div id="onboarding">
  ### Configuración inicial
</div>

Durante la configuración inicial (`openclaw onboard`), se te pedirá que habilites los hooks recomendados. El asistente detecta automáticamente los hooks disponibles y te los muestra para que los selecciones.

<div id="hook-discovery">
  ## Descubrimiento de Hooks
</div>

Los hooks se detectan automáticamente en tres directorios (en orden de precedencia):

1. **Hooks del espacio de trabajo**: `<workspace>/hooks/` (por agente, mayor precedencia)
2. **Hooks gestionados**: `~/.openclaw/hooks/` (instalados por el usuario, compartidos entre espacios de trabajo)
3. **Hooks incluidos**: `<openclaw>/dist/hooks/bundled/` (se distribuyen con OpenClaw)

Los directorios de hooks gestionados pueden ser un **hook único** o un **paquete de hooks** (directorio de paquete).

Cada hook es un directorio que contiene:

```
my-hook/
├── HOOK.md          # Metadatos + documentación
└── handler.ts       # Implementación del handler
```

<div id="hook-packs-npmarchives">
  ## Paquetes de hooks (npm/archivos)
</div>

Los paquetes de hooks son paquetes npm estándar que exponen uno o más hooks mediante `openclaw.hooks` en
`package.json`. Instálalos con:

```bash
openclaw hooks install <path-or-spec>
```

Ejemplo de `package.json`:

```json
{
  "name": "@acme/my-hooks",
  "version": "0.1.0",
  "openclaw": {
    "hooks": ["./hooks/my-hook", "./hooks/other-hook"]
  }
}
```

Cada entrada hace referencia a un directorio de hook que contiene `HOOK.md` y `handler.ts` (o `index.ts`).
Los paquetes de hooks pueden incluir dependencias; se instalarán en `~/.openclaw/hooks/<id>`.

<div id="hook-structure">
  ## Estructura del hook
</div>

<div id="hookmd-format">
  ### Formato de HOOK.md
</div>

El archivo `HOOK.md` contiene metadatos en el frontmatter YAML y documentación en Markdown:

```markdown
---
name: my-hook
description: "Short description of what this hook does"
homepage: https://docs.openclaw.ai/hooks#my-hook
metadata: {"openclaw":{"emoji":"🔗","events":["command:new"],"requires":{"bins":["node"]}}}
---

# My Hook

Detailed documentation goes here...

## What It Does

- Listens for `/new` commands
- Performs some action
- Logs the result

## Requirements

- Node.js must be installed

## Configuration

No configuration needed.
```

<div id="metadata-fields">
  ### Campos de metadatos
</div>

El objeto `metadata.openclaw` admite:

* **`emoji`**: Emoji que se muestra en la CLI (por ejemplo, `"💾"`)
* **`events`**: Array de eventos a los que escuchar (por ejemplo, `["command:new", "command:reset"]`)
* **`export`**: Exportación con nombre que se va a usar (valor predeterminado `"default"`)
* **`homepage`**: URL de la documentación
* **`requires`**: Requisitos opcionales
  * **`bins`**: Binarios requeridos en PATH (por ejemplo, `["git", "node"]`)
  * **`anyBins`**: Al menos uno de estos binarios debe estar presente
  * **`env`**: Variables de entorno requeridas
  * **`config`**: Rutas de configuración requeridas (por ejemplo, `["workspace.dir"]`)
  * **`os`**: Plataformas requeridas (por ejemplo, `["darwin", "linux"]`)
* **`always`**: Omite las comprobaciones de elegibilidad (booleano)
* **`install`**: Métodos de instalación (para hooks integrados: `[{"id":"bundled","kind":"bundled"}]`)

<div id="handler-implementation">
  ### Implementación del controlador
</div>

El archivo `handler.ts` exporta la función `HookHandler`:

```typescript
import type { HookHandler } from '../../src/hooks/hooks.js';

const myHandler: HookHandler = async (event) => {
  // Solo activar en el comando 'new'
  if (event.type !== 'command' || event.action !== 'new') {
    return;
  }

  console.log(`[my-hook] Comando new activado`);
  console.log(`  Sesión: ${event.sessionKey}`);
  console.log(`  Timestamp: ${event.timestamp.toISOString()}`);

  // Coloca tu lógica personalizada aquí

  // Opcionalmente, envía un mensaje al usuario
  event.messages.push('✨ ¡Hook ejecutado!');
};

export default myHandler;
```

<div id="event-context">
  #### Contexto del evento
</div>

Cada evento incluye:

```typescript
{
  type: 'command' | 'session' | 'agent' | 'gateway',
  action: string,              // e.g., 'new', 'reset', 'stop'
  sessionKey: string,          // Session identifier
  timestamp: Date,             // When the event occurred
  messages: string[],          // Agregar mensajes aquí para enviarlos al usuario
  context: {
    sessionEntry?: SessionEntry,
    sessionId?: string,
    sessionFile?: string,
    commandSource?: string,    // e.g., 'whatsapp', 'telegram'
    senderId?: string,
    workspaceDir?: string,
    bootstrapFiles?: WorkspaceBootstrapFile[],
    cfg?: OpenClawConfig
  }
}
```

<div id="event-types">
  ## Tipos de eventos
</div>

<div id="command-events">
  ### Eventos de comando
</div>

Se activan cuando se emiten comandos del agente:

* **`command`**: Todos los eventos de comando (listener general)
* **`command:new`**: Cuando se emite el comando `/new`
* **`command:reset`**: Cuando se emite el comando `/reset`
* **`command:stop`**: Cuando se emite el comando `/stop`

<div id="agent-events">
  ### Eventos del agente
</div>

* **`agent:bootstrap`**: Antes de que se inyecten los archivos de bootstrap del espacio de trabajo (los hooks pueden modificar `context.bootstrapFiles`)

<div id="gateway-events">
  ### Eventos del Gateway
</div>

Se dispara cuando se inicia el Gateway:

* **`gateway:startup`**: Después de que los canales se hayan iniciado y se hayan cargado los hooks

<div id="tool-result-hooks-plugin-api">
  ### Hooks de resultados de herramientas (API de complementos)
</div>

Estos hooks no son listeners de flujos de eventos; permiten que los complementos ajusten de forma síncrona los resultados de herramientas antes de que OpenClaw los persista.

* **`tool_result_persist`**: transforma los resultados de herramientas antes de que se escriban en la transcripción de la sesión. Debe ser síncrono; devuelve la carga útil actualizada del resultado de herramienta o `undefined` para mantenerla sin cambios. Consulta [Agent Loop](/es/concepts/agent-loop).

<div id="future-events">
  ### Eventos futuros
</div>

Tipos de eventos previstos:

* **`session:start`**: Cuando comienza una nueva sesión
* **`session:end`**: Cuando finaliza una sesión
* **`agent:error`**: Cuando un agente presenta un error
* **`message:sent`**: Cuando se envía un mensaje
* **`message:received`**: Cuando se recibe un mensaje

<div id="creating-custom-hooks">
  ## Crear hooks personalizados
</div>

<div id="1-choose-location">
  ### 1. Elige la ubicación
</div>

* **Hooks del espacio de trabajo** (`<workspace>/hooks/`): A nivel de agente, con la precedencia más alta
* **Hooks gestionados** (`~/.openclaw/hooks/`): Compartidos entre todos los espacios de trabajo

<div id="2-create-directory-structure">
  ### 2. Crear la estructura de directorios
</div>

```bash
mkdir -p ~/.openclaw/hooks/my-hook
cd ~/.openclaw/hooks/my-hook
```

<div id="3-create-hookmd">
  ### 3. Crea HOOK.md
</div>

```markdown
---
name: my-hook
description: "Does something useful"
metadata: {"openclaw":{"emoji":"🎯","events":["command:new"]}}
---

# My Custom Hook

This hook does something useful when you issue `/new`.
```

<div id="4-create-handlerts">
  ### 4. Crea handler.ts
</div>

```typescript
import type { HookHandler } from '../../src/hooks/hooks.js';

const handler: HookHandler = async (event) => {
  if (event.type !== 'command' || event.action !== 'new') {
    return;
  }

  console.log('[my-hook] Running!');
  // Tu lógica aquí
};

export default handler;
```

<div id="5-enable-and-test">
  ### 5. Habilitar y probar
</div>

```bash
# Verify hook is discovered
openclaw hooks list

# Enable it
openclaw hooks enable my-hook

# Reiniciar el proceso del Gateway (reinicio de la aplicación de barra de menú en macOS, o reiniciar el proceso de desarrollo)

# Trigger the event
# Send /new via your messaging channel
```

<div id="configuration">
  ## Configuración
</div>

<div id="new-config-format-recommended">
  ### Nuevo formato de configuración (recomendado)
</div>

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "entries": {
        "session-memory": { "enabled": true },
        "command-logger": { "enabled": false }
      }
    }
  }
}
```

<div id="per-hook-configuration">
  ### Configuración específica por hook
</div>

Cada hook puede tener su propia configuración:

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "entries": {
        "my-hook": {
          "enabled": true,
          "env": {
            "MY_CUSTOM_VAR": "value"
          }
        }
      }
    }
  }
}
```

<div id="extra-directories">
  ### Directorios adicionales
</div>

Carga hooks desde directorios adicionales:

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "load": {
        "extraDirs": ["/path/to/more/hooks"]
      }
    }
  }
}
```

<div id="legacy-config-format-still-supported">
  ### Formato de configuración legado (todavía compatible)
</div>

El formato de configuración anterior sigue funcionando para mantener la compatibilidad con versiones anteriores:

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "handlers": [
        {
          "event": "command:new",
          "module": "./hooks/handlers/my-handler.ts",
          "export": "default"
        }
      ]
    }
  }
}
```

**Migración**: Utiliza el nuevo sistema basado en descubrimiento para los nuevos hooks. Los controladores heredados se cargan después de los hooks basados en directorios.

<div id="cli-commands">
  ## Comandos de la CLI
</div>

<div id="list-hooks">
  ### Listar hooks
</div>

```bash
# List all hooks
openclaw hooks list

# Show only eligible hooks
openclaw hooks list --eligible

# Salida detallada (mostrar requisitos faltantes)
openclaw hooks list --verbose

# JSON output
openclaw hooks list --json
```

<div id="hook-information">
  ### Información del hook
</div>

```bash
# Mostrar información detallada sobre un hook
openclaw hooks info session-memory

# JSON output
openclaw hooks info session-memory --json
```

<div id="check-eligibility">
  ### Comprobar requisitos
</div>

```bash
# Mostrar resumen de elegibilidad
openclaw hooks check

# Salida JSON
openclaw hooks check --json
```

<div id="enabledisable">
  ### Habilitar/Deshabilitar
</div>

```bash
# Enable a hook
openclaw hooks enable session-memory

# Deshabilitar un hook
openclaw hooks disable command-logger
```

## Hooks integrados

<div id="session-memory">
  ### session-memory
</div>

Guarda el contexto de la sesión en memoria cuando ejecutas `/new`.

**Eventos**: `command:new`

**Requisitos**: `workspace.dir` debe estar configurado

**Salida**: `<workspace>/memory/YYYY-MM-DD-slug.md` (por defecto `~/.openclaw/workspace`)

**Qué hace**:

1. Usa la entrada de la sesión previa al reinicio para localizar el historial correcto
2. Extrae las últimas 15 líneas de la conversación
3. Usa un LLM para generar un *slug* descriptivo para el nombre de archivo
4. Guarda los metadatos de la sesión en un archivo de memoria con fecha

**Ejemplo de salida**:

```markdown
# Sesión: 2026-01-16 14:30:00 UTC

- **Clave de sesión**: agent:main:main
- **ID de sesión**: abc123def456
- **Origen**: telegram
```

**Ejemplos de nombres de archivo**:

* `2026-01-16-vendor-pitch.md`
* `2026-01-16-api-design.md`
* `2026-01-16-1430.md` (marca de tiempo alternativa si la generación del *slug* falla)

**Activar**:

```bash
openclaw hooks enable session-memory
```

<div id="command-logger">
  ### command-logger
</div>

Registra todos los eventos de comandos en un archivo de auditoría centralizado.

**Eventos**: `command`

**Requisitos**: Ninguno

**Salida**: `~/.openclaw/logs/commands.log`

**Qué hace**:

1. Captura los detalles del evento (acción del comando, marca de tiempo, clave de sesión, ID del remitente, origen)
2. Agrega datos al archivo de registro en formato JSONL
3. Se ejecuta silenciosamente en segundo plano

**Ejemplos de entradas de registro**:

```jsonl
{"timestamp":"2026-01-16T14:30:00.000Z","action":"new","sessionKey":"agent:main:main","senderId":"+1234567890","source":"telegram"}
{"timestamp":"2026-01-16T15:45:22.000Z","action":"stop","sessionKey":"agent:main:main","senderId":"user@example.com","source":"whatsapp"}
```

**Ver registros**:

```bash
# View recent commands
tail -n 20 ~/.openclaw/logs/commands.log

# Imprimir con formato usando jq
cat ~/.openclaw/logs/commands.log | jq .

# Filter by action
grep '"action":"new"' ~/.openclaw/logs/commands.log | jq .
```

**Activar**:

```bash
openclaw hooks enable command-logger
```

<div id="soul-evil">
  ### soul-evil
</div>

Intercambia el contenido inyectado de `SOUL.md` por `SOUL_EVIL.md` durante una ventana de purga o de forma aleatoria.

**Eventos**: `agent:bootstrap`

**Documentación**: [SOUL Evil Hook](/es/hooks/soul-evil)

**Salida**: No se escriben archivos; los cambios se realizan únicamente en memoria.

**Habilitar**:

```bash
openclaw hooks enable soul-evil
```

**Configuración**:

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "entries": {
        "soul-evil": {
          "enabled": true,
          "file": "SOUL_EVIL.md",
          "chance": 0.1,
          "purge": { "at": "21:00", "duration": "15m" }
        }
      }
    }
  }
}
```

<div id="boot-md">
  ### boot-md
</div>

Ejecuta `BOOT.md` cuando el Gateway se inicia (después de que se inicien los canales).
Los hooks internos deben estar activados para que se ejecute.

**Eventos**: `gateway:startup`

**Requisitos**: `workspace.dir` debe estar configurado

**Qué hace**:

1. Lee `BOOT.md` desde tu espacio de trabajo
2. Ejecuta las instrucciones mediante el ejecutor de agentes
3. Envía cualquier mensaje saliente solicitado mediante la herramienta de mensajes

**Habilitar**:

```bash
openclaw hooks enable boot-md
```

<div id="best-practices">
  ## Buenas prácticas
</div>

<div id="keep-handlers-fast">
  ### Haz que los handlers sean rápidos
</div>

Los hooks se ejecutan mientras se procesan los comandos. Procura que sean ligeros:

```typescript
// ✓ Bien - trabajo asíncrono, retorna inmediatamente
const handler: HookHandler = async (event) => {
  void processInBackground(event); // Dispara y olvida
};

// ✗ Mal - bloquea el procesamiento de comandos
const handler: HookHandler = async (event) => {
  await slowDatabaseQuery(event);
  await evenSlowerAPICall(event);
};
```

<div id="handle-errors-gracefully">
  ### Gestiona los errores de forma controlada
</div>

Envuelve siempre las operaciones propensas a fallar:

```typescript
const handler: HookHandler = async (event) => {
  try {
    await riskyOperation(event);
  } catch (err) {
    console.error('[my-handler] Failed:', err instanceof Error ? err.message : String(err));
    // No lanzar excepciones - permitir que se ejecuten otros manejadores
  }
};
```

<div id="filter-events-early">
  ### Filtra eventos lo antes posible
</div>

Devuelve de inmediato si el evento no es relevante:

```typescript
const handler: HookHandler = async (event) => {
  // Solo manejar comandos 'new'
  if (event.type !== 'command' || event.action !== 'new') {
    return;
  }

  // Tu lógica aquí
};
```

<div id="use-specific-event-keys">
  ### Usa claves de eventos específicas
</div>

Especifica eventos concretos en los metadatos siempre que sea posible:

```yaml
metadata: {"openclaw":{"events":["command:new"]}}  # Específico
```

En lugar de:

```yaml
metadata: {"openclaw":{"events":["command"]}}      # General - más sobrecarga
```

<div id="debugging">
  ## Depuración
</div>

<div id="enable-hook-logging">
  ### Activar el registro de hooks
</div>

El Gateway registra la carga de hooks durante el inicio:

```
Registered hook: session-memory -> command:new
Registered hook: command-logger -> command
Registered hook: boot-md -> gateway:startup
```

<div id="check-discovery">
  ### Comprobar descubrimiento
</div>

Muestra todos los hooks detectados:

```bash
openclaw hooks list --verbose
```

<div id="check-registration">
  ### Verificar registro
</div>

En tu handler, registra en los logs cuándo se invoque:

```typescript
const handler: HookHandler = async (event) => {
  console.log('[my-handler] Triggered:', event.type, event.action);
  // Tu lógica
};
```

<div id="verify-eligibility">
  ### Verificar la elegibilidad
</div>

Comprueba por qué un hook no es elegible:

```bash
openclaw hooks info my-hook
```

Busca requisitos que falten en la salida.

<div id="testing">
  ## Pruebas
</div>

<div id="gateway-logs">
  ### Registros del Gateway
</div>

Consulta los registros del Gateway para ver la ejecución de los hooks:

```bash
# macOS
./scripts/clawlog.sh -f

# Otras plataformas
tail -f ~/.openclaw/gateway.log
```

<div id="test-hooks-directly">
  ### Probar hooks directamente
</div>

Prueba tus controladores de forma aislada:

```typescript
import { test } from 'vitest';
import { createHookEvent } from './src/hooks/hooks.js';
import myHandler from './hooks/my-hook/handler.js';

test('my handler works', async () => {
  const event = createHookEvent('command', 'new', 'test-session', {
    foo: 'bar'
  });

  await myHandler(event);

  // Assert side effects
});
```

<div id="architecture">
  ## Arquitectura
</div>

<div id="core-components">
  ### Componentes principales
</div>

* **`src/hooks/types.ts`**: Definiciones de tipos
* **`src/hooks/workspace.ts`**: Exploración y carga de directorios
* **`src/hooks/frontmatter.ts`**: Análisis de metadatos de HOOK.md
* **`src/hooks/config.ts`**: Verificación de elegibilidad
* **`src/hooks/hooks-status.ts`**: Informe de estado
* **`src/hooks/loader.ts`**: Cargador dinámico de módulos
* **`src/cli/hooks-cli.ts`**: Comandos de la CLI
* **`src/gateway/server-startup.ts`**: Carga los hooks al inicio del Gateway
* **`src/auto-reply/reply/commands-core.ts`**: Activa eventos de comandos

<div id="discovery-flow">
  ### Flujo de descubrimiento
</div>

```
Gateway startup
    ↓
Scan directories (workspace → managed → bundled)
    ↓
Parse HOOK.md files
    ↓
Check eligibility (bins, env, config, os)
    ↓
Load handlers from eligible hooks
    ↓
Register handlers for events
```

<div id="event-flow">
  ### Flujo de eventos
</div>

```
User sends /new
    ↓
Command validation
    ↓
Create hook event
    ↓
Trigger hook (all registered handlers)
    ↓
Command processing continues
    ↓
Session reset
```

<div id="troubleshooting">
  ## Solución de problemas
</div>

<div id="hook-not-discovered">
  ### Hook no detectado
</div>

1. Comprueba la estructura del directorio:
   ```bash
   ls -la ~/.openclaw/hooks/my-hook/
   # Debería mostrar: HOOK.md, handler.ts
   ```

2. Comprueba el formato de HOOK.md:
   ```bash
   cat ~/.openclaw/hooks/my-hook/HOOK.md
   # Debería tener front matter YAML con nombre y metadatos
   ```

3. Enumera todos los hooks detectados:
   ```bash
   openclaw hooks list
   ```

<div id="hook-not-eligible">
  ### Hook no apto
</div>

Verifica los requisitos:

```bash
openclaw hooks info my-hook
```

Comprueba si falta alguno de los siguientes elementos:

* Ejecutables (revisa el PATH)
* Variables de entorno
* Valores de configuración
* Compatibilidad con el sistema operativo

<div id="hook-not-executing">
  ### Hook no se ejecuta
</div>

1. Verifica que el hook esté habilitado:
   ```bash
   openclaw hooks list
   # Debería mostrar ✓ junto a los hooks habilitados
   ```

2. Reinicia tu proceso del Gateway para volver a cargar los hooks.

3. Revisa los registros del Gateway en busca de errores:
   ```bash
   ./scripts/clawlog.sh | grep hook
   ```

<div id="handler-errors">
  ### Errores del controlador
</div>

Comprueba si hay errores de TypeScript o de importación:

```bash
# Probar importación directamente
node -e "import('./path/to/handler.ts').then(console.log)"
```

<div id="migration-guide">
  ## Guía de migración
</div>

<div id="from-legacy-config-to-discovery">
  ### De la configuración heredada al descubrimiento
</div>

**Antes**:

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "handlers": [
        {
          "event": "command:new",
          "module": "./hooks/handlers/my-handler.ts"
        }
      ]
    }
  }
}
```

**Después**:

1. Crea el directorio del hook:
   ```bash
   mkdir -p ~/.openclaw/hooks/my-hook
   mv ./hooks/handlers/my-handler.ts ~/.openclaw/hooks/my-hook/handler.ts
   ```

2. Crea HOOK.md:
   ```markdown
   ---
   name: my-hook
   description: "My custom hook"
   metadata: {"openclaw":{"emoji":"🎯","events":["command:new"]}}
   ---

   # My Hook

   Hace algo útil.
   ```

3. Actualiza la configuración:
   ```json
   {
     "hooks": {
       "internal": {
         "enabled": true,
         "entries": {
           "my-hook": { "enabled": true }
         }
       }
     }
   }
   ```

4. Verifica y reinicia tu proceso del Gateway:
   ```bash
   openclaw hooks list
   # Debería mostrar: 🎯 my-hook ✓
   ```

**Beneficios de la migración**:

* Descubrimiento automático
* Gestión desde la CLI
* Comprobación de elegibilidad
* Mejor documentación
* Estructura consistente

<div id="see-also">
  ## Consulte también
</div>

* [Referencia de la CLI: hooks](/es/cli/hooks)
* [README de hooks incluidos](https://github.com/openclaw/openclaw/tree/main/src/hooks/bundled)
* [Hooks de webhooks](/es/automation/webhook)
* [Configuración](/es/gateway/configuration#hooks)