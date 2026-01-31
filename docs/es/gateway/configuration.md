---
title: Configuración
summary: "Todas las opciones de configuración para ~/.openclaw/openclaw.json con ejemplos"
read_when:
  - Al añadir o modificar campos de configuración
---

<div id="configuration">
  # Configuración 🔧
</div>

OpenClaw lee un archivo de configuración opcional en formato **JSON5** desde `~/.openclaw/openclaw.json` (se permiten comentarios y comas finales).

Si el archivo no existe, OpenClaw usa valores predeterminados bastante seguros (agente Pi incrustado + sesiones por remitente + espacio de trabajo `~/.openclaw/workspace`). Normalmente solo necesitas un archivo de configuración para:

* restringir quién puede activar el bot (`channels.whatsapp.allowFrom`, `channels.telegram.allowFrom`, etc.)
* controlar listas de permitidos de grupos y el comportamiento de las menciones (`channels.whatsapp.groups`, `channels.telegram.groups`, `channels.discord.guilds`, `agents.list[].groupChat`)
* personalizar prefijos de mensajes (`messages`)
* establecer el espacio de trabajo del agente (`agents.defaults.workspace` o `agents.list[].workspace`)
* ajustar los valores predeterminados del agente incrustado (`agents.defaults`) y el comportamiento de la sesión (`session`)
* definir la identidad de cada agente (`agents.list[].identity`)

> **¿Nuevo en la configuración?** Consulta la guía de [Ejemplos de configuración](/es/gateway/configuration-examples) para ver ejemplos completos con explicaciones detalladas.

<div id="strict-config-validation">
  ## Validación estricta de la configuración
</div>

OpenClaw solo acepta configuraciones que coincidan por completo con el esquema.
Claves desconocidas, tipos malformados o valores inválidos hacen que el Gateway **se niegue a iniciarse** por motivos de seguridad.

Cuando la validación falla:

* El Gateway no arranca.
* Solo se permiten comandos de diagnóstico (por ejemplo: `openclaw doctor`, `openclaw logs`, `openclaw health`, `openclaw status`, `openclaw service`, `openclaw help`).
* Ejecuta `openclaw doctor` para ver los problemas exactos.
* Ejecuta `openclaw doctor --fix` (o `--yes`) para aplicar migraciones/reparaciones.

Doctor nunca realiza cambios a menos que habilites explícitamente `--fix`/`--yes`.

<div id="schema-ui-hints">
  ## Esquema + sugerencias de UI
</div>

El Gateway expone una representación de JSON Schema de la configuración mediante `config.schema` para editores de UI.
El Control UI renderiza un formulario a partir de este esquema, con un editor de **Raw JSON** como vía de escape.

Los complementos y extensiones de canal pueden registrar esquema + sugerencias de UI para su configuración, de modo que los ajustes de canal
sigan siendo basados en esquemas en todas las aplicaciones, sin formularios con lógica fija.

Las sugerencias (etiquetas, agrupación, campos sensibles) se distribuyen junto con el esquema para que los clientes puedan generar
mejores formularios sin codificar de forma rígida el conocimiento específico de la configuración.

<div id="apply-restart-rpc">
  ## Apply + restart (RPC)
</div>

Usa `config.apply` para validar y escribir la configuración completa y reiniciar el Gateway en un solo paso.
Escribe un centinela de reinicio y envía un ping a la última sesión activa después de que el Gateway vuelva a estar en línea.

Advertencia: `config.apply` reemplaza **toda la configuración**. Si quieres cambiar solo unas pocas claves,
usa `config.patch` u `openclaw config set`. Conserva una copia de seguridad de `~/.openclaw/openclaw.json`.

Parámetros:

* `raw` (string) — payload JSON5 para toda la configuración
* `baseHash` (opcional) — hash de configuración de `config.get` (requerido cuando ya existe una configuración)
* `sessionKey` (opcional) — clave de la última sesión activa para el ping de activación
* `note` (opcional) — nota que se incluirá en el centinela de reinicio
* `restartDelayMs` (opcional) — retraso antes del reinicio (por defecto 2000)

Ejemplo (vía `gateway call`):

```bash
openclaw gateway call config.get --params '{}' # capturar payload.hash
openclaw gateway call config.apply --params '{
  "raw": "{\\n  agents: { defaults: { workspace: \\"~/.openclaw/workspace\\" } }\\n}\\n",
  "baseHash": "<hash-from-config.get>",
  "sessionKey": "agent:main:whatsapp:dm:+15555550123",
  "restartDelayMs": 1000
}'
```

<div id="partial-updates-rpc">
  ## Actualizaciones parciales (RPC)
</div>

Usa `config.patch` para fusionar una actualización parcial en la configuración existente sin sobrescribir
claves no relacionadas. Aplica la semántica de JSON Merge Patch:

* los objetos se fusionan de forma recursiva
* `null` elimina una clave
* los arrays reemplazan por completo
  Al igual que `config.apply`, valida, escribe la configuración, guarda un marcador de reinicio y programa
  el reinicio del Gateway (con una reactivación opcional cuando se proporciona `sessionKey`).

Parámetros:

* `raw` (string) — payload JSON5 que contiene solo las claves que se van a cambiar
* `baseHash` (obligatorio) — hash de configuración de `config.get`
* `sessionKey` (opcional) — última clave de sesión activa para el ping de reactivación
* `note` (opcional) — nota que se incluirá en el marcador de reinicio
* `restartDelayMs` (opcional) — retraso antes del reinicio (por defecto 2000)

Ejemplo:

```bash
openclaw gateway call config.get --params '{}' # captura payload.hash
openclaw gateway call config.patch --params '{
  "raw": "{\\n  channels: { telegram: { groups: { \\"*\\": { requireMention: false } } } }\\n}\\n",
  "baseHash": "<hash-from-config.get>",
  "sessionKey": "agent:main:whatsapp:dm:+15555550123",
  "restartDelayMs": 1000
}'
```

<div id="minimal-config-recommended-starting-point">
  ## Configuración mínima (recomendada como punto de partida)
</div>

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
  channels: { whatsapp: { allowFrom: ["+15555550123"] } }
}
```

Construye la imagen predeterminada una vez con:

```bash
scripts/sandbox-setup.sh
```

<div id="self-chat-mode-recommended-for-group-control">
  ## Modo de autochat (recomendado para la gestión de grupos)
</div>

Para evitar que el bot responda a las menciones con @ en grupos de WhatsApp (solo responder a disparadores de texto específicos):

```json5
{
  agents: {
    defaults: { workspace: "~/.openclaw/workspace" },
    list: [
      {
        id: "main",
        groupChat: { mentionPatterns: ["@openclaw", "reisponde"] }
      }
    ]
  },
  channels: {
    whatsapp: {
      // La lista de permitidos es solo para DMs; incluir tu propio número habilita el modo de auto-chat.
      allowFrom: ["+15555550123"],
      groups: { "*": { requireMention: true } }
    }
  }
}
```

<div id="config-includes-include">
  ## Inclusiones de configuración (`$include`)
</div>

Divide tu configuración en varios archivos usando la directiva `$include`. Esto es útil para:

* Organizar configuraciones grandes (por ejemplo, definiciones de agentes por cliente)
* Compartir ajustes comunes entre entornos
* Mantener separada la configuración sensible

<div id="basic-usage">
  ### Uso básico
</div>

```json5
// ~/.openclaw/openclaw.json
{
  gateway: { port: 18789 },
  
  // Include a single file (replaces the key's value)
  agents: { "$include": "./agents.json5" },
  
  // Incluye múltiples archivos (fusionados en profundidad en orden)
  broadcast: { 
    "$include": [
      "./clients/mueller.json5",
      "./clients/schmidt.json5"
    ]
  }
}
```

```json5
// ~/.openclaw/agents.json5
{
  defaults: { sandbox: { mode: "all", scope: "session" } },
  list: [
    { id: "main", workspace: "~/.openclaw/workspace" }
  ]
}
```

<div id="merge-behavior">
  ### Comportamiento de fusión
</div>

* **Archivo único**: Reemplaza el objeto que contiene `$include`
* **Array de archivos**: Fusiona en profundidad los archivos en orden (los archivos posteriores sobrescriben a los anteriores)
* **Con claves hermanas**: Las claves hermanas se fusionan después de los `$include` (sobrescriben los valores incluidos)
* **Claves hermanas + arrays/primitivos**: No es compatible (el contenido incluido debe ser un objeto)

```json5
// Sibling keys override included values
{
  "$include": "./base.json5",   // { a: 1, b: 2 }
  b: 99                          // Resultado: { a: 1, b: 99 }
}
```

<div id="nested-includes">
  ### Includes anidados
</div>

Los archivos incluidos pueden, a su vez, contener directivas `$include` (hasta 10 niveles de anidación):

```json5
// clients/mueller.json5
{
  agents: { "$include": "./mueller/agents.json5" },
  broadcast: { "$include": "./mueller/broadcast.json5" }
}
```

<div id="path-resolution">
  ### Resolución de rutas
</div>

* **Rutas relativas**: Se resuelven con respecto al archivo que las incluye
* **Rutas absolutas**: Se usan tal cual
* **Directorios superiores**: Las referencias `../` funcionan como se espera

```json5
{ "$include": "./sub/config.json5" }      // relative
{ "$include": "/etc/openclaw/base.json5" } // absolute
{ "$include": "../shared/common.json5" }   // directorio padre
```

<div id="error-handling">
  ### Manejo de errores
</div>

* **Archivo no encontrado**: Error claro con la ruta resuelta
* **Error de parseo**: Muestra qué archivo incluido falló
* **Inclusiones circulares**: Se detectan y se informan junto con la cadena de inclusiones

<div id="example-multi-client-legal-setup">
  ### Ejemplo: configuración legal para varios clientes
</div>

```json5
// ~/.openclaw/openclaw.json
{
  gateway: { port: 18789, auth: { token: "secret" } },
  
  // Common agent defaults
  agents: {
    defaults: {
      sandbox: { mode: "all", scope: "session" }
    },
    // Combinar listas de agentes de todos los clientes
    list: { "$include": [
      "./clients/mueller/agents.json5",
      "./clients/schmidt/agents.json5"
    ]}
  },
  
  // Merge broadcast configs
  broadcast: { "$include": [
    "./clients/mueller/broadcast.json5",
    "./clients/schmidt/broadcast.json5"
  ]},
  
  channels: { whatsapp: { groupPolicy: "allowlist" } }
}
```

```json5
// ~/.openclaw/clients/mueller/agents.json5
[
  { id: "mueller-transcribe", workspace: "~/clients/mueller/transcribe" },
  { id: "mueller-docs", workspace: "~/clients/mueller/docs" }
]
```

```json5
// ~/.openclaw/clients/mueller/broadcast.json5
{
  "120363403215116621@g.us": ["mueller-transcribe", "mueller-docs"]
}
```

<div id="common-options">
  ## Opciones comunes
</div>

<div id="env-vars-env">
  ### Variables de entorno + `.env`
</div>

OpenClaw lee las variables de entorno del proceso padre (shell, launchd/systemd, CI, etc.).

Además, carga:

* `.env` del directorio de trabajo actual (si existe)
* un `.env` global de respaldo desde `~/.openclaw/.env` (es decir, `$OPENCLAW_STATE_DIR/.env`)

Ningún archivo `.env` sobrescribe variables de entorno ya existentes.

También puedes definir variables de entorno en línea en la configuración. Estas solo se aplican si
al entorno del proceso le falta esa clave (mismo comportamiento de no sobrescritura):

```json5
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: {
      GROQ_API_KEY: "gsk-..."
    }
  }
}
```

Consulta [/environment](/es/environment) para ver el orden de precedencia completo y todas las fuentes.

<div id="envshellenv-optional">
  ### `env.shellEnv` (opcional)
</div>

Comodidad opcional: si está activado y ninguna de las variables de entorno esperadas se ha definido aún, OpenClaw ejecuta tu shell de inicio de sesión e importa solo las variables esperadas que falten (nunca sobrescribe).
Esto, en la práctica, equivale a hacer `source` de tu perfil de shell.

```json5
{
  env: {
    shellEnv: {
      enabled: true,
      timeoutMs: 15000
    }
  }
}
```

Equivalente con variables de entorno:

* `OPENCLAW_LOAD_SHELL_ENV=1`
* `OPENCLAW_SHELL_ENV_TIMEOUT_MS=15000`

<div id="env-var-substitution-in-config">
  ### Sustitución de variables de entorno en la configuración
</div>

Puedes hacer referencia directamente a variables de entorno en cualquier valor de cadena de la configuración usando la sintaxis
`${VAR_NAME}`. Las variables se sustituyen al cargar la configuración, antes de la validación.

```json5
{
  models: {
    providers: {
      "vercel-gateway": {
        apiKey: "${VERCEL_GATEWAY_API_KEY}"
      }
    }
  },
  gateway: {
    auth: {
      token: "${OPENCLAW_GATEWAY_TOKEN}"
    }
  }
}
```

**Reglas:**

* Solo se consideran nombres de variables de entorno en mayúsculas: `[A-Z_][A-Z0-9_]*`
* Las variables de entorno ausentes o vacías generan un error al cargar la configuración
* Escapa usando `$${VAR}` para mostrar literalmente `${VAR}`
* Funciona con `$include` (los archivos incluidos también se someten a sustitución)

**Sustitución en línea:**

```json5
{
  models: {
    providers: {
      custom: {
        baseUrl: "${CUSTOM_API_BASE}/v1"  // → "https://api.example.com/v1"
      }
    }
  }
}
```

<div id="auth-storage-oauth-api-keys">
  ### Almacenamiento de autenticación (OAuth + API keys)
</div>

OpenClaw almacena perfiles de autenticación **por agente** (OAuth + API keys) en:

* `<agentDir>/auth-profiles.json` (predeterminado: `~/.openclaw/agents/<agentId>/agent/auth-profiles.json`)

Consulta también: [/concepts/oauth](/es/concepts/oauth)

Importaciones heredadas de OAuth:

* `~/.openclaw/credentials/oauth.json` (o `$OPENCLAW_STATE_DIR/credentials/oauth.json`)

El agente Pi integrado mantiene una caché en tiempo de ejecución en:

* `<agentDir>/auth.json` (gestionado automáticamente; no lo modifiques manualmente)

Directorio de agente heredado (antes de la compatibilidad multiagente):

* `~/.openclaw/agent/*` (migrado por `openclaw doctor` a `~/.openclaw/agents/<defaultAgentId>/agent/*`)

Overrides:

* Directorio OAuth (solo para importación heredada): `OPENCLAW_OAUTH_DIR`
* Directorio de agente (override de la raíz del agente predeterminado): `OPENCLAW_AGENT_DIR` (preferido), `PI_CODING_AGENT_DIR` (heredado)

En el primer uso, OpenClaw importa las entradas de `oauth.json` en `auth-profiles.json`.

<div id="auth">
  ### `auth`
</div>

Metadatos opcionales de perfiles de autenticación. Esto **no** almacena secretos; asigna IDs de perfil a un proveedor + modo (y un correo electrónico opcional) y define el orden de rotación de proveedores usado para la conmutación por error.

```json5
{
  auth: {
    profiles: {
      "anthropic:me@example.com": { provider: "anthropic", mode: "oauth", email: "me@example.com" },
      "anthropic:work": { provider: "anthropic", mode: "api_key" }
    },
    order: {
      anthropic: ["anthropic:me@example.com", "anthropic:work"]
    }
  }
}
```

<div id="agentslistidentity">
  ### `agents.list[].identity`
</div>

Identidad opcional para cada agente, usada para valores predeterminados y la experiencia de usuario (UX). Esto lo escribe el asistente de incorporación de macOS.

Si se configura, OpenClaw deriva valores predeterminados (solo cuando no los hayas establecido explícitamente):

* `messages.ackReaction` a partir de `identity.emoji` del **agente activo** (con valor de reserva 👀)
* `agents.list[].groupChat.mentionPatterns` a partir de `identity.name`/`identity.emoji` del agente (de modo que “@Samantha” funcione en grupos en Telegram/Slack/Discord/Google Chat/iMessage/WhatsApp)
* `identity.avatar` acepta una ruta de imagen relativa al espacio de trabajo o una URL remota/URL de datos. Los archivos locales deben residir dentro del espacio de trabajo del agente.

`identity.avatar` acepta:

* Ruta relativa al espacio de trabajo (debe permanecer dentro del espacio de trabajo del agente)
* URL `http(s)`
* URI `data:`

```json5
{
  agents: {
    list: [
      {
        id: "main",
        identity: {
          name: "Samantha",
          theme: "perezoso útil",
          emoji: "🦥",
          avatar: "avatars/samantha.png"
        }
      }
    ]
  }
}
```

<div id="wizard">
  ### `wizard`
</div>

Metadatos generados por los asistentes de la CLI (`onboard`, `configure`, `doctor`).

```json5
{
  wizard: {
    lastRunAt: "2026-01-01T00:00:00.000Z",
    lastRunVersion: "2026.1.4",
    lastRunCommit: "abc1234",
    lastRunCommand: "configure",
    lastRunMode: "local"
  }
}
```

<div id="logging">
  ### `logging`
</div>

* Archivo de registro predeterminado: `/tmp/openclaw/openclaw-YYYY-MM-DD.log`
* Si necesitas una ruta estable, establece `logging.file` en `/tmp/openclaw/openclaw.log`.
* La salida de la consola se puede ajustar por separado mediante:
  * `logging.consoleLevel` (valor predeterminado `info`, pasa a `debug` cuando se usa `--verbose`)
  * `logging.consoleStyle` (`pretty` | `compact` | `json`)
* Se pueden ocultar los resúmenes de herramientas para evitar la filtración de secretos:
  * `logging.redactSensitive` (`off` | `tools`, predeterminado: `tools`)
  * `logging.redactPatterns` (lista de cadenas de expresiones regulares (regex); sustituye los valores predeterminados)

```json5
{
  logging: {
    level: "info",
    file: "/tmp/openclaw/openclaw.log",
    consoleLevel: "info",
    consoleStyle: "pretty",
    redactSensitive: "tools",
    redactPatterns: [
      // Ejemplo: sobrescribe los valores predeterminados con tus propias reglas.
      "\\bTOKEN\\b\\s*[=:]\\s*([\"']?)([^\\s\"']+)\\1",
      "/\\bsk-[A-Za-z0-9_-]{8,}\\b/gi"
    ]
  }
}
```

<div id="channelswhatsappdmpolicy">
  ### `channels.whatsapp.dmPolicy`
</div>

Controla cómo se gestionan los chats directos (DM) de WhatsApp:

* `"pairing"` (predeterminado): los remitentes desconocidos reciben un código de emparejamiento; el propietario debe aprobarlo
* `"allowlist"`: solo permite remitentes en `channels.whatsapp.allowFrom` (o en el almacén de allowlist emparejado)
* `"open"`: permite todos los DM entrantes (**requiere** que `channels.whatsapp.allowFrom` incluya `"*"`)
* `"disabled"`: ignora todos los DM entrantes

Los códigos de emparejamiento caducan después de 1 hora; el bot solo envía un código de emparejamiento cuando se crea una nueva solicitud. De forma predeterminada, las solicitudes de emparejamiento de DM pendientes se limitan a un máximo de **3 por canal**.

Aprobaciones de emparejamiento:

* `openclaw pairing list whatsapp`
* `openclaw pairing approve whatsapp <code>`

<div id="channelswhatsappallowfrom">
  ### `channels.whatsapp.allowFrom`
</div>

Lista de permitidos de números de teléfono E.164 que pueden activar respuestas automáticas de WhatsApp (**solo DMs**).
Si la lista está vacía y `channels.whatsapp.dmPolicy="pairing"`, los remitentes desconocidos recibirán un código de emparejamiento.
Para grupos, utiliza `channels.whatsapp.groupPolicy` + `channels.whatsapp.groupAllowFrom`.

```json5
{
  channels: {
    whatsapp: {
      dmPolicy: "pairing", // emparejamiento | lista de permitidos | open | deshabilitado
      allowFrom: ["+15555550123", "+447700900123"],
      textChunkLimit: 4000, // optional outbound chunk size (chars)
      chunkMode: "length", // optional chunking mode (length | newline)
      mediaMaxMb: 50 // optional inbound media cap (MB)
    }
  }
}
```

<div id="channelswhatsappsendreadreceipts">
  ### `channels.whatsapp.sendReadReceipts`
</div>

Controla si los mensajes entrantes de WhatsApp se marcan como leídos (doble tilde azul). Valor por defecto: `true`.

El modo de chat contigo mismo siempre omite las confirmaciones de lectura, incluso cuando está activada esta opción.

Anulación por cuenta: `channels.whatsapp.accounts.<id>.sendReadReceipts`.

```json5
{
  channels: {
    whatsapp: { sendReadReceipts: false }
  }
}
```

<div id="channelswhatsappaccounts-multi-account">
  ### `channels.whatsapp.accounts` (varias cuentas)
</div>

Ejecuta varias cuentas de WhatsApp en un único Gateway:

```json5
{
  channels: {
    whatsapp: {
      accounts: {
        default: {}, // optional; keeps the default id stable
        personal: {},
        biz: {
          // Anulación opcional. Por defecto: ~/.openclaw/credentials/whatsapp/biz
          // authDir: "~/.openclaw/credentials/whatsapp/biz",
        }
      }
    }
  }
}
```

Notas:

* Los comandos salientes usan por defecto la cuenta `default` si está presente; de lo contrario, el primer id de cuenta configurado (en orden).
* El directorio de autenticación Baileys heredado de una sola cuenta se migra con `openclaw doctor` a `whatsapp/default`.

<div id="channelstelegramaccounts-channelsdiscordaccounts-channelsgooglechataccounts-channelsslackaccounts-channelsmattermostaccounts-channelssignalaccounts-channelsimessageaccounts">
  ### `channels.telegram.accounts` / `channels.discord.accounts` / `channels.googlechat.accounts` / `channels.slack.accounts` / `channels.mattermost.accounts` / `channels.signal.accounts` / `channels.imessage.accounts`
</div>

Utiliza varias cuentas por canal (cada cuenta tiene su propio `accountId` y un `name` opcional):

```json5
{
  channels: {
    telegram: {
      accounts: {
        default: {
          name: "Primary bot",
          botToken: "123456:ABC..."
        },
        alerts: {
          name: "Alerts bot",
          botToken: "987654:XYZ..."
        }
      }
    }
  }
}
```

Notas:

* `default` se usa cuando se omite `accountId` (CLI + enrutamiento).
* Los tokens de entorno solo se aplican a la cuenta **default**.
* La configuración básica del canal (política de grupo, restricción por mención, etc.) se aplica a todas las cuentas, a menos que se sobrescriba por cuenta.
* Usa `bindings[].match.accountId` para enrutar cada cuenta a un `agents.defaults` diferente.

<div id="group-chat-mention-gating-agentslistgroupchat-messagesgroupchat">
  ### Control de menciones en chats grupales (`agents.list[].groupChat` + `messages.groupChat`)
</div>

De forma predeterminada, los mensajes de grupo **requieren mención** (ya sea mediante metadatos o patrones regex). Aplica a los chats grupales de WhatsApp, Telegram, Discord, Google Chat e iMessage.

**Tipos de mención:**

* **Menciones por metadatos**: Menciones nativas de la plataforma con @ (por ejemplo, tocar para mencionar en WhatsApp). Se ignoran en el modo de chat contigo mismo en WhatsApp (consulta `channels.whatsapp.allowFrom`).
* **Patrones de texto**: Patrones regex definidos en `agents.list[].groupChat.mentionPatterns`. Siempre se verifican, independientemente del modo de chat contigo mismo.
* El control de menciones solo se aplica cuando es posible detectar menciones (menciones nativas o al menos un `mentionPattern`).

```json5
{
  messages: {
    groupChat: { historyLimit: 50 }
  },
  agents: {
    list: [
      { id: "main", groupChat: { mentionPatterns: ["@openclaw", "openclaw"] } }
    ]
  }
}
```

`messages.groupChat.historyLimit` establece el valor predeterminado global para el contexto de historial de grupos. Los canales pueden sobrescribirlo con `channels.<channel>.historyLimit` (o `channels.<channel>.accounts.*.historyLimit` para múltiples cuentas). Establece `0` para desactivar el recorte automático del historial.

<div id="dm-history-limits">
  #### Límites de historial de DM
</div>

Las conversaciones por DM usan un historial basado en sesión gestionado por el agente. Puedes limitar el número de intervenciones de usuario que se conservan por cada sesión de DM:

```json5
{
  channels: {
    telegram: {
      dmHistoryLimit: 30,  // limitar sesiones de DM a 30 turnos de usuario
      dms: {
        "123456789": { historyLimit: 50 }  // sobrescritura por usuario (ID de usuario)
      }
    }
  }
}
```

Orden de resolución:

1. Anulación por DM: `channels.<provider>.dms[userId].historyLimit`
2. Valor predeterminado del proveedor: `channels.<provider>.dmHistoryLimit`
3. Sin límite (se conserva todo el historial)

Proveedores compatibles: `telegram`, `whatsapp`, `discord`, `slack`, `signal`, `imessage`, `msteams`.

Anulación por agente (tiene prioridad cuando está definida, incluso `[]`):

```json5
{
  agents: {
    list: [
      { id: "work", groupChat: { mentionPatterns: ["@workbot", "\\+15555550123"] } },
      { id: "personal", groupChat: { mentionPatterns: ["@homebot", "\\+15555550999"] } }
    ]
  }
}
```

Los valores predeterminados de control de menciones se definen por canal (`channels.whatsapp.groups`, `channels.telegram.groups`, `channels.imessage.groups`, `channels.discord.guilds`). Cuando se configura `*.groups`, también actúa como una lista de permitidos de grupos; incluye `"*"` para permitir todos los grupos.

Para responder **solo** a activadores de texto específicos (ignorando las menciones nativas con @):

```json5
{
  channels: {
    whatsapp: {
      // Incluye tu propio número para habilitar el modo de auto-chat (ignora las @-menciones nativas).
      allowFrom: ["+15555550123"],
      groups: { "*": { requireMention: true } }
    }
  },
  agents: {
    list: [
      {
        id: "main",
        groupChat: {
          // Only these text patterns will trigger responses
          mentionPatterns: ["reisponde", "@openclaw"]
        }
      }
    ]
  }
}
```

<div id="group-policy-per-channel">
  ### Política de grupos (por canal)
</div>

Usa `channels.*.groupPolicy` para controlar si se aceptan o no mensajes en grupos/salas:

```json5
{
  channels: {
    whatsapp: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15551234567"]
    },
    telegram: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["tg:123456789", "@alice"]
    },
    signal: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15551234567"]
    },
    imessage: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["chat_id:123"]
    },
    msteams: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["user@org.com"]
    },
    discord: {
      groupPolicy: "allowlist",
      guilds: {
        "GUILD_ID": {
          channels: { help: { allow: true } }
        }
      }
    },
    slack: {
      groupPolicy: "allowlist",
      channels: { "#general": { allow: true } }
    }
  }
}
```

Notas:

* `"open"`: los grupos omiten las listas de permitidos; esta configuración permite aceptar mensajes sin restricciones de cualquier usuario; el filtrado por menciones sigue aplicándose.
* `"disabled"`: bloquea todos los mensajes de grupos/salas.
* `"allowlist"`: solo permite grupos/salas que coincidan con la lista de permitidos configurada.
* `channels.defaults.groupPolicy` establece el valor predeterminado cuando `groupPolicy` de un proveedor no está configurado.
* WhatsApp/Telegram/Signal/iMessage/Microsoft Teams usan `groupAllowFrom` (con retroceso a `allowFrom` explícito).
* Discord/Slack usan listas de permitidos de canales (`channels.discord.guilds.*.channels`, `channels.slack.channels`).
* Los mensajes directos grupales (Discord/Slack) siguen estando controlados por `dm.groupEnabled` + `dm.groupChannels`.
* El valor predeterminado es `groupPolicy: "allowlist"` (a menos que se anule mediante `channels.defaults.groupPolicy`); si no se configura ninguna lista de permitidos, los mensajes de grupo se bloquean.

<div id="multi-agent-routing-agentslist-bindings">
  ### Enrutamiento multi-agente (`agents.list` + `bindings`)
</div>

Ejecuta múltiples agentes aislados (espacio de trabajo separado, `agentDir`, sesiones) dentro de un único Gateway.
Los mensajes entrantes se enrutan a un agente mediante bindings.

* `agents.list[]`: overrides específicos por agente.
  * `id`: id de agente estable (obligatorio).
  * `default`: opcional; cuando hay varios definidos, gana el primero y se registra una advertencia.
    Si no se define ninguno, la **primera entrada** de la lista es el agente predeterminado.
  * `name`: nombre visible para el agente.
  * `workspace`: `~/.openclaw/workspace-<agentId>` por defecto (para `main`, recurre a `agents.defaults.workspace`).
  * `agentDir`: `~/.openclaw/agents/<agentId>/agent` por defecto.
  * `model`: modelo predeterminado por agente, reemplaza `agents.defaults.model` para ese agente.
    * forma de cadena: `"provider/model"`, reemplaza solo `agents.defaults.model.primary`
    * forma de objeto: `{ primary, fallbacks }` (los fallbacks reemplazan `agents.defaults.model.fallbacks`; `[]` desactiva los fallbacks globales para ese agente)
  * `identity`: nombre/tema/emoji por agente (utilizado para patrones de mención + reacciones de confirmación/ack).
  * `groupChat`: control de acceso por mención por agente (`mentionPatterns`).
  * `sandbox`: configuración de sandbox por agente (reemplaza `agents.defaults.sandbox`).
    * `mode`: `"off"` | `"non-main"` | `"all"`
    * `workspaceAccess`: `"none"` | `"ro"` | `"rw"`
    * `scope`: `"session"` | `"agent"` | `"shared"`
    * `workspaceRoot`: raíz personalizada del espacio de trabajo del sandbox
    * `docker`: overrides de docker por agente (p. ej. `image`, `network`, `env`, `setupCommand`, límites; se ignora cuando `scope: "shared"`)
    * `browser`: overrides por agente del navegador en sandbox (se ignora cuando `scope: "shared"`)
    * `prune`: overrides por agente del pruning del sandbox (se ignora cuando `scope: "shared"`)
  * `subagents`: valores predeterminados de sub-agente por agente.
    * `allowAgents`: lista de permitidos de ids de agente para `sessions_spawn` desde este agente (`["*"]` = permitir cualquiera; por defecto: solo el mismo agente)
  * `tools`: restricciones de herramientas por agente (aplicadas antes de la política de herramientas del sandbox).
    * `profile`: perfil base de herramientas (se aplica antes de allow/deny)
    * `allow`: array de nombres de herramientas permitidos
    * `deny`: array de nombres de herramientas denegados (deny prevalece)
* `agents.defaults`: valores predeterminados compartidos de agente (modelo, espacio de trabajo, sandbox, etc.).
* `bindings[]`: enruta los mensajes entrantes a un `agentId`.
  * `match.channel` (obligatorio)
  * `match.accountId` (opcional; `*` = cualquier cuenta; si se omite = cuenta predeterminada)
  * `match.peer` (opcional; `{ kind: dm|group|channel, id }`)
  * `match.guildId` / `match.teamId` (opcional; específico del canal)

Orden de coincidencia determinista:

1. `match.peer`
2. `match.guildId`
3. `match.teamId`
4. `match.accountId` (exacto, sin peer/guild/team)
5. `match.accountId: "*"` (a nivel de canal, sin peer/guild/team)
6. agente predeterminado (`agents.list[].default`, si no, primera entrada de la lista; si no, `"main"`)

Dentro de cada nivel de coincidencia, gana la primera entrada de `bindings` que coincida.

<div id="per-agent-access-profiles-multi-agent">
  #### Perfiles de acceso por agente (multiagente)
</div>

Cada agente puede tener su propia sandbox y política de herramientas. Usa esto para combinar distintos niveles de acceso en un Gateway:

* **Acceso completo** (agente personal)
* Herramientas de **solo lectura** + espacio de trabajo
* **Sin acceso al sistema de archivos** (solo herramientas de mensajería y de sesión)

Consulta [Sandbox y herramientas multiagente](/es/multi-agent-sandbox-tools) para conocer la precedencia y ver ejemplos adicionales.

Acceso completo (sin sandbox):

```json5
{
  agents: {
    list: [
      {
        id: "personal",
        workspace: "~/.openclaw/workspace-personal",
        sandbox: { mode: "off" }
      }
    ]
  }
}
```

Herramientas y espacio de trabajo de solo lectura:

```json5
{
  agents: {
    list: [
      {
        id: "family",
        workspace: "~/.openclaw/workspace-family",
        sandbox: {
          mode: "all",
          scope: "agent",
          workspaceAccess: "ro"
        },
        tools: {
          allow: ["read", "sessions_list", "sessions_history", "sessions_send", "sessions_spawn", "session_status"],
          deny: ["write", "edit", "apply_patch", "exec", "process", "browser"]
        }
      }
    ]
  }
}
```

Sin acceso al sistema de archivos (con las herramientas de mensajería/sesión habilitadas):

```json5
{
  agents: {
    list: [
      {
        id: "public",
        workspace: "~/.openclaw/workspace-public",
        sandbox: {
          mode: "all",
          scope: "agent",
          workspaceAccess: "none"
        },
        tools: {
          allow: ["sessions_list", "sessions_history", "sessions_send", "sessions_spawn", "session_status", "whatsapp", "telegram", "slack", "discord", "gateway"],
          deny: ["read", "write", "edit", "apply_patch", "exec", "process", "browser", "canvas", "nodes", "cron", "gateway", "image"]
        }
      }
    ]
  }
}
```

Ejemplo: dos cuentas de WhatsApp → dos agentes:

```json5
{
  agents: {
    list: [
      { id: "home", default: true, workspace: "~/.openclaw/workspace-home" },
      { id: "work", workspace: "~/.openclaw/workspace-work" }
    ]
  },
  bindings: [
    { agentId: "home", match: { channel: "whatsapp", accountId: "personal" } },
    { agentId: "work", match: { channel: "whatsapp", accountId: "biz" } }
  ],
  channels: {
    whatsapp: {
      accounts: {
        personal: {},
        biz: {},
      }
    }
  }
}
```

<div id="toolsagenttoagent-optional">
  ### `tools.agentToAgent` (opcional)
</div>

La mensajería entre agentes requiere activación explícita:

```json5
{
  tools: {
    agentToAgent: {
      enabled: false,
      allow: ["home", "work"]
    }
  }
}
```

<div id="messagesqueue">
  ### `messages.queue`
</div>

Controla cómo se comportan los mensajes entrantes cuando ya hay una ejecución de un agente en curso.

```json5
{
  messages: {
    queue: {
      mode: "collect", // steer | followup | collect | steer-backlog (steer+backlog ok) | interrupt (queue=steer legado)
      debounceMs: 1000,
      cap: 20,
      drop: "summarize", // old | new | summarize
      byChannel: {
        whatsapp: "collect",
        telegram: "collect",
        discord: "collect",
        imessage: "collect",
        webchat: "collect"
      }
    }
  }
}
```

<div id="messagesinbound">
  ### `messages.inbound`
</div>

Aplica *debounce* a los mensajes entrantes rápidos del **mismo remitente** para que varios
mensajes consecutivos se traten como un único turno del agente. El *debounce* se delimita por canal + conversación
y usa el mensaje más reciente para el encadenamiento de respuestas/IDs.

```json5
{
  messages: {
    inbound: {
      debounceMs: 2000, // 0 para deshabilitar
      byChannel: {
        whatsapp: 5000,
        slack: 1500,
        discord: 1500
      }
    }
  }
}
```

Notas:

* Se aplica *debounce* solo a las tandas de mensajes **de texto**; los mensajes con contenido multimedia/adjuntos se envían inmediatamente.
* Los comandos de control (p. ej., `/queue`, `/new`) omiten el mecanismo de *debounce* para procesarse de forma independiente.

<div id="commands-chat-command-handling">
  ### `commands` (gestión de comandos de chat)
</div>

Controla cómo se habilitan los comandos de chat en los distintos conectores.

```json5
{
  commands: {
    native: "auto",         // register native commands when supported (auto)
    text: true,             // parse slash commands in chat messages
    bash: false,            // permitir ! (alias: /bash) (solo host; requiere listas de permitidos tools.elevated)
    bashForegroundMs: 2000, // bash foreground window (0 backgrounds immediately)
    config: false,          // allow /config (writes to disk)
    debug: false,           // allow /debug (runtime-only overrides)
    restart: false,         // allow /restart + gateway restart tool
    useAccessGroups: true   // enforce access-group allowlists/policies for commands
  }
}
```

Notas:

* Los comandos de texto deben enviarse como un mensaje **independiente** y usar el prefijo `/` (sin alias en texto plano).
* `commands.text: false` desactiva el análisis de mensajes de chat en busca de comandos.
* `commands.native: "auto"` (valor predeterminado) activa los comandos nativos para Discord/Telegram y deja Slack desactivado; los canales no compatibles permanecen solo con texto.
* Establece `commands.native: true|false` para forzarlo en todos los canales, o anúlalo por canal con `channels.discord.commands.native`, `channels.telegram.commands.native`, `channels.slack.commands.native` (bool o `"auto"`). `false` borra los comandos registrados previamente en Discord/Telegram al inicio; los comandos de Slack se gestionan en la app de Slack.
* `channels.telegram.customCommands` añade entradas adicionales al menú del bot de Telegram. Los nombres se normalizan; los conflictos con comandos nativos se ignoran.
* `commands.bash: true` habilita `! <cmd>` para ejecutar comandos del shell del host (`/bash <cmd>` también funciona como alias). Requiere `tools.elevated.enabled` e incluir al remitente en la lista de permitidos en `tools.elevated.allowFrom.<channel>`.
* `commands.bashForegroundMs` controla cuánto tiempo espera bash antes de pasar a segundo plano. Mientras se está ejecutando un trabajo de bash, se rechazan nuevas solicitudes `! <cmd>` (una a la vez).
* `commands.config: true` habilita `/config` (lee/escribe `openclaw.json`).
* `channels.<provider>.configWrites` controla las mutaciones de configuración iniciadas por ese canal (predeterminado: true). Esto se aplica a `/config set|unset` más las migraciones automáticas específicas del proveedor (cambios de ID de supergrupo de Telegram, cambios de ID de canal de Slack).
* `commands.debug: true` habilita `/debug` (anulaciones solo en tiempo de ejecución).
* `commands.restart: true` habilita `/restart` y la acción de reinicio de la herramienta del Gateway.
* `commands.useAccessGroups: false` permite que los comandos omitan las listas de permitidos/políticas de grupos de acceso.
* Los comandos con `/` y las directivas solo se aplican a **remitentes autorizados**. La autorización se deriva de
  las listas de permitidos/emparejamiento del canal más `commands.useAccessGroups`.

<div id="web-whatsapp-web-channel-runtime">
  ### `web` (entorno de ejecución del canal web de WhatsApp)
</div>

WhatsApp se ejecuta a través del canal web del Gateway (Baileys Web). Se inicia automáticamente cuando existe una sesión vinculada.
Configura `web.enabled: false` para que permanezca desactivado de forma predeterminada.

```json5
{
  web: {
    enabled: true,
    heartbeatSeconds: 60,
    reconnect: {
      initialMs: 2000,
      maxMs: 120000,
      factor: 1.4,
      jitter: 0.2,
      maxAttempts: 0
    }
  }
}
```

<div id="channelstelegram-bot-transport">
  ### `channels.telegram` (transporte de bot)
</div>

OpenClaw inicia Telegram únicamente cuando existe una sección de configuración `channels.telegram`. El token del bot se obtiene de `channels.telegram.botToken` (o `channels.telegram.tokenFile`), usando `TELEGRAM_BOT_TOKEN` como alternativa para la cuenta predeterminada.
Configura `channels.telegram.enabled: false` para desactivar el inicio automático.
El soporte para múltiples cuentas se encuentra bajo `channels.telegram.accounts` (consulta la sección de múltiples cuentas más arriba). Los tokens de entorno solo se aplican a la cuenta predeterminada.
Configura `channels.telegram.configWrites: false` para bloquear las escrituras de configuración iniciadas por Telegram (incluidas las migraciones de ID de supergrupo y `/config set|unset`).

```json5
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "your-bot-token",
      dmPolicy: "pairing",                 // pairing | allowlist | open | disabled
      allowFrom: ["tg:123456789"],         // optional; "open" requires ["*"]
      groups: {
        "*": { requireMention: true },
        "-1001234567890": {
          allowFrom: ["@admin"],
          systemPrompt: "Keep answers brief.",
          topics: {
            "99": {
              requireMention: false,
              skills: ["search"],
              systemPrompt: "Stay on topic."
            }
          }
        }
      },
      customCommands: [
        { command: "backup", description: "Git backup" },
        { command: "generate", description: "Create an image" }
      ],
      historyLimit: 50,                     // include last N group messages as context (0 disables)
      replyToMode: "first",                 // off | first | all
      linkPreview: true,                   // toggle outbound link previews
      streamMode: "partial",               // off | partial | block (transmisión de borrador; separado de la transmisión de bloques)
      draftChunk: {                        // optional; only for streamMode=block
        minChars: 200,
        maxChars: 800,
        breakPreference: "paragraph"       // paragraph | newline | sentence
      },
      actions: { reactions: true, sendMessage: true }, // tool action gates (false disables)
      reactionNotifications: "own",   // off | own | all
      mediaMaxMb: 5,
      retry: {                             // outbound retry policy
        attempts: 3,
        minDelayMs: 400,
        maxDelayMs: 30000,
        jitter: 0.1
      },
      network: {                           // transport overrides
        autoSelectFamily: false
      },
      proxy: "socks5://localhost:9050",
      webhookUrl: "https://example.com/telegram-webhook",
      webhookSecret: "secret",
      webhookPath: "/telegram-webhook"
    }
  }
}
```

Notas sobre el streaming de borradores:

* Utiliza `sendMessageDraft` de Telegram (burbuja de borrador, no un mensaje real).
* Requiere **temas de chat privados** (`message&#95;thread&#95;id` en DMs; el bot tiene los temas habilitados).
* `/reasoning stream` hace streaming del razonamiento al borrador y luego envía la respuesta final.
  La configuración predeterminada y el comportamiento de la política de reintentos se documentan en [Política de reintentos](/es/concepts/retry).

<div id="channelsdiscord-bot-transport">
  ### `channels.discord` (transporte del bot)
</div>

Configura el bot de Discord estableciendo el token del bot y las restricciones de acceso opcionales:
La compatibilidad con varias cuentas se gestiona en `channels.discord.accounts` (consulta la sección de varias cuentas anterior). Los tokens de entorno solo se aplican a la cuenta predeterminada.

```json5
{
  channels: {
    discord: {
      enabled: true,
      token: "your-bot-token",
      mediaMaxMb: 8,                          // clamp inbound media size
      allowBots: false,                       // allow bot-authored messages
      actions: {                              // tool action gates (false disables)
        reactions: true,
        stickers: true,
        polls: true,
        permissions: true,
        messages: true,
        threads: true,
        pins: true,
        search: true,
        memberInfo: true,
        roleInfo: true,
        roles: false,
        channelInfo: true,
        voiceStatus: true,
        events: true,
        moderation: false
      },
      replyToMode: "off",                     // off | first | all
      dm: {
        enabled: true,                        // disable all DMs when false
        policy: "pairing",                    // pairing | allowlist | open | disabled
        allowFrom: ["1234567890", "steipete"], // optional DM allowlist ("open" requires ["*"])
        groupEnabled: false,                 // enable group DMs
        groupChannels: ["openclaw-dm"]          // optional group DM allowlist
      },
      guilds: {
        "123456789012345678": {               // guild id (preferred) or slug
          slug: "friends-of-openclaw",
          requireMention: false,              // per-guild default
          reactionNotifications: "own",       // off | own | all | allowlist
          users: ["987654321098765432"],      // optional per-guild user allowlist
          channels: {
            general: { allow: true },
            help: {
              allow: true,
              requireMention: true,
              users: ["987654321098765432"],
              skills: ["docs"],
              systemPrompt: "Short answers only."
            }
          }
        }
      },
      historyLimit: 20,                       // include last N guild messages as context
      textChunkLimit: 2000,                   // optional outbound text chunk size (chars)
      chunkMode: "length",                    // optional chunking mode (length | newline)
      maxLinesPerMessage: 17,                 // máximo flexible de líneas por mensaje (recorte de UI de Discord)
      retry: {                                // outbound retry policy
        attempts: 3,
        minDelayMs: 500,
        maxDelayMs: 30000,
        jitter: 0.1
      }
    }
  }
}
```

OpenClaw inicia Discord solo cuando existe una sección de configuración `channels.discord`. El token se resuelve desde `channels.discord.token`, con `DISCORD_BOT_TOKEN` como respaldo para la cuenta predeterminada (a menos que `channels.discord.enabled` sea `false`). Usa `user:<id>` (DM) o `channel:<id>` (canal de servidor/guild) al especificar destinos de entrega para comandos de cron/CLI; los IDs numéricos sin prefijo son ambiguos y se rechazan.
Los slugs de servidor/guild están en minúsculas, con los espacios reemplazados por `-`; las claves de canal usan el nombre del canal en formato slug (sin `#` inicial). Prefiere usar los IDs de servidor/guild como claves para evitar ambigüedad en caso de renombrar.
Los mensajes generados por bots se ignoran de forma predeterminada. Habilítalo con `channels.discord.allowBots` (los mensajes propios siguen filtrándose para evitar bucles de autorrespuesta).
Modos de notificación de reacciones:

* `off`: sin eventos de reacción.
* `own`: reacciones sobre los mensajes propios del bot (predeterminado).
* `all`: todas las reacciones sobre todos los mensajes.
* `allowlist`: reacciones de `guilds.<id>.users` sobre todos los mensajes (una lista vacía las deshabilita).
  El texto saliente se divide en fragmentos según `channels.discord.textChunkLimit` (2000 por defecto). Establece `channels.discord.chunkMode="newline"` para dividir en líneas en blanco (límites de párrafo) antes de segmentar por longitud. Los clientes de Discord pueden recortar mensajes muy altos, por lo que `channels.discord.maxLinesPerMessage` (17 por defecto) divide las respuestas multilínea largas incluso cuando están por debajo de 2000 caracteres.
  Los valores predeterminados y el comportamiento de la política de reintentos están documentados en [Retry policy](/es/concepts/retry).

<div id="channelsgooglechat-chat-api-webhook">
  ### `channels.googlechat` (webhook de Chat API)
</div>

Google Chat funciona mediante webhooks HTTP con autenticación a nivel de aplicación (cuenta de servicio).
El soporte para varias cuentas se configura en `channels.googlechat.accounts` (consulta la sección sobre múltiples cuentas más arriba). Las variables de entorno solo se aplican a la cuenta predeterminada.

```json5
{
  channels: {
    "googlechat": {
      enabled: true,
      serviceAccountFile: "/path/to/service-account.json",
      audienceType: "app-url",             // app-url | project-number
      audience: "https://gateway.example.com/googlechat",
      webhookPath: "/googlechat",
      botUser: "users/1234567890",        // opcional; mejora la detección de menciones
      dm: {
        enabled: true,
        policy: "pairing",                // pairing | allowlist | open | disabled
        allowFrom: ["users/1234567890"]   // opcional; "open" requiere ["*"]
      },
      groupPolicy: "allowlist",
      groups: {
        "spaces/AAAA": { allow: true, requireMention: true }
      },
      actions: { reactions: true },
      typingIndicator: "message",
      mediaMaxMb: 20
    }
  }
}
```

Notas:

* El JSON de la cuenta de servicio puede ir en línea (`serviceAccount`) o en un archivo (`serviceAccountFile`).
* Variables de entorno alternativas para la cuenta predeterminada: `GOOGLE_CHAT_SERVICE_ACCOUNT` o `GOOGLE_CHAT_SERVICE_ACCOUNT_FILE`.
* `audienceType` + `audience` deben coincidir con la configuración de autenticación del webhook de la aplicación de Chat.
* Usa `spaces/<spaceId>` o `users/<userId|email>` al definir los destinos de envío.

<div id="channelsslack-socket-mode">
  ### `channels.slack` (Socket Mode)
</div>

Slack se ejecuta en Socket Mode y requiere tanto un token de bot como un token de aplicación:

```json5
{
  channels: {
    slack: {
      enabled: true,
      botToken: "xoxb-...",
      appToken: "xapp-...",
      dm: {
        enabled: true,
        policy: "pairing", // pairing | allowlist | open | disabled
        allowFrom: ["U123", "U456", "*"], // optional; "open" requires ["*"]
        groupEnabled: false,
        groupChannels: ["G123"]
      },
      channels: {
        C123: { allow: true, requireMention: true, allowBots: false },
        "#general": {
          allow: true,
          requireMention: true,
          allowBots: false,
          users: ["U123"],
          skills: ["docs"],
          systemPrompt: "Short answers only."
        }
      },
      historyLimit: 50,          // incluir los últimos N mensajes de canal/grupo como contexto (0 deshabilita)
      allowBots: false,
      reactionNotifications: "own", // off | own | all | allowlist
      reactionAllowlist: ["U123"],
      replyToMode: "off",           // off | first | all
      thread: {
        historyScope: "thread",     // thread | channel
        inheritParent: false
      },
      actions: {
        reactions: true,
        messages: true,
        pins: true,
        memberInfo: true,
        emojiList: true
      },
      slashCommand: {
        enabled: true,
        name: "openclaw",
        sessionPrefix: "slack:slash",
        ephemeral: true
      },
      textChunkLimit: 4000,
      chunkMode: "length",
      mediaMaxMb: 20
    }
  }
}
```

La compatibilidad con múltiples cuentas se encuentra en `channels.slack.accounts` (ver la sección de múltiples cuentas más arriba). Los tokens de entorno solo se aplican a la cuenta predeterminada.

OpenClaw inicia Slack cuando el proveedor está habilitado y ambos tokens están configurados (mediante la configuración o `SLACK_BOT_TOKEN` + `SLACK_APP_TOKEN`). Usa `user:<id>` (DM) o `channel:<id>` al especificar destinos de envío para comandos de cron/CLI.
Establece `channels.slack.configWrites: false` para bloquear escrituras de configuración iniciadas por Slack (incluidas las migraciones de ID de canal y `/config set|unset`).

Los mensajes generados por bots se ignoran de forma predeterminada. Habilítalos con `channels.slack.allowBots` o `channels.slack.channels.<id>.allowBots`.

Modos de notificación de reacciones:

* `off`: sin eventos de reacción.
* `own`: reacciones en los mensajes propios del bot (predeterminado).
* `all`: todas las reacciones en todos los mensajes.
* `allowlist`: reacciones desde `channels.slack.reactionAllowlist` en todos los mensajes (una lista vacía las deshabilita).

Aislamiento de sesiones de hilos:

* `channels.slack.thread.historyScope` controla si el historial del hilo es por hilo (`thread`, predeterminado) o se comparte a nivel de canal (`channel`).
* `channels.slack.thread.inheritParent` controla si las nuevas sesiones de hilo heredan la transcripción del canal padre (predeterminado: false).

Grupos de acciones de Slack (limitan las acciones de la herramienta `slack`):

| Grupo de acciones | Predeterminado | Notas                          |
| ----------------- | -------------- | ------------------------------ |
| reactions         | enabled        | Reaccionar + listar reacciones |
| messages          | enabled        | Leer/enviar/editar/borrar      |
| pins              | enabled        | Fijar/desfijar/listar          |
| memberInfo        | enabled        | Información de miembros        |
| emojiList         | enabled        | Lista de emojis personalizados |

<div id="channelsmattermost-bot-token">
  ### `channels.mattermost` (token de bot)
</div>

Mattermost se distribuye como un complemento y no viene incluido con la instalación principal.
Instálalo primero: `openclaw plugins install @openclaw/mattermost` (o `./extensions/mattermost` desde un clon de git).

Mattermost requiere un token de bot y la URL base de tu servidor:

```json5
{
  channels: {
    mattermost: {
      enabled: true,
      botToken: "mm-token",
      baseUrl: "https://chat.example.com",
      dmPolicy: "pairing",
      chatmode: "oncall", // oncall | onmessage | onchar
      oncharPrefixes: [">", "!"],
      textChunkLimit: 4000,
      chunkMode: "length"
    }
  }
}
```

OpenClaw inicia Mattermost cuando la cuenta está configurada (bot token + URL base) y habilitada. El token y la URL base se resuelven a partir de `channels.mattermost.botToken` + `channels.mattermost.baseUrl` o de `MATTERMOST_BOT_TOKEN` + `MATTERMOST_URL` para la cuenta predeterminada (a menos que `channels.mattermost.enabled` sea `false`).

Modos de chat:

* `oncall` (predeterminado): responde a los mensajes del canal solo cuando se le menciona con @.
* `onmessage`: responde a cada mensaje del canal.
* `onchar`: responde cuando un mensaje comienza con un prefijo de activación (`channels.mattermost.oncharPrefixes`, valor predeterminado `[">", "!"]`).

Control de acceso:

* DMs predeterminados: `channels.mattermost.dmPolicy="pairing"` (los remitentes desconocidos reciben un código de emparejamiento).
* DMs públicos: `channels.mattermost.dmPolicy="open"` más `channels.mattermost.allowFrom=["*"]`.
* Grupos: `channels.mattermost.groupPolicy="allowlist"` de forma predeterminada (restringido por mención). Usa `channels.mattermost.groupAllowFrom` para restringir remitentes.

La compatibilidad con múltiples cuentas se encuentra en `channels.mattermost.accounts` (consulta la sección de múltiples cuentas más arriba). Las variables de entorno solo se aplican a la cuenta predeterminada.
Usa `channel:<id>` o `user:<id>` (o `@username`) al especificar destinos de entrega; los ID sin prefijo se tratan como ID de canal.

<div id="channelssignal-signal-cli">
  ### `channels.signal` (signal-cli)
</div>

Las reacciones de Signal pueden emitir eventos del sistema (tooling compartido para reacciones):

```json5
{
  channels: {
    signal: {
      reactionNotifications: "own", // off | own | all | allowlist
      reactionAllowlist: ["+15551234567", "uuid:123e4567-e89b-12d3-a456-426614174000"],
      historyLimit: 50 // incluir los últimos N mensajes de grupo como contexto (0 lo deshabilita)
    }
  }
}
```

Modos de notificación de reacciones:

* `off`: sin eventos de reacción.
* `own`: reacciones en los propios mensajes del bot (predeterminado).
* `all`: todas las reacciones en todos los mensajes.
* `allowlist`: reacciones de `channels.signal.reactionAllowlist` en todos los mensajes (una lista vacía las desactiva).

<div id="channelsimessage-imsg-cli">
  ### `channels.imessage` (CLI imsg)
</div>

OpenClaw lanza `imsg rpc` (JSON-RPC a través de stdio). No se requiere ningún daemon ni puerto.

```json5
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "imsg",
      dbPath: "~/Library/Messages/chat.db",
      remoteHost: "user@gateway-host", // SCP for remote attachments when using SSH wrapper
      dmPolicy: "pairing", // pairing | allowlist | open | disabled
      allowFrom: ["+15555550123", "user@example.com", "chat_id:123"],
      historyLimit: 50,    // incluir los últimos N mensajes de grupo como contexto (0 para deshabilitar)
      includeAttachments: false,
      mediaMaxMb: 16,
      service: "auto",
      region: "US"
    }
  }
}
```

La compatibilidad con varias cuentas se gestiona en `channels.imessage.accounts` (consulta la sección de varias cuentas más arriba).

Notas:

* Requiere el permiso de Acceso total al disco para la base de datos de Mensajes.
* El primer send solicitará permiso de automatización de Mensajes.
* Da preferencia a destinos `chat_id:&lt;id&gt;`. Usa `imsg chats --limit 20` para listar chats.
* `channels.imessage.cliPath` puede apuntar a un script envoltorio (por ejemplo, `ssh` a otro Mac que ejecute `imsg rpc`); usa claves SSH para evitar solicitudes de contraseña.
* Para scripts envoltorios SSH remotos, establece `channels.imessage.remoteHost` para obtener archivos adjuntos vía SCP cuando `includeAttachments` está activado.

Ejemplo de script envoltorio:

```bash
#!/usr/bin/env bash
exec ssh -T gateway-host imsg "$@"
```

<div id="agentsdefaultsworkspace">
  ### `agents.defaults.workspace`
</div>

Configura el **único directorio de espacio de trabajo global** que usa el agente para las operaciones de archivos.

Valor predeterminado: `~/.openclaw/workspace`.

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } }
}
```

Si `agents.defaults.sandbox` está habilitado, las sesiones secundarias pueden anular esto con sus propios espacios de trabajo por ámbito bajo `agents.defaults.sandbox.workspaceRoot`.

<div id="agentsdefaultsreporoot">
  ### `agents.defaults.repoRoot`
</div>

Raíz de repositorio opcional que se mostrará en la línea Runtime del prompt del sistema. Si no se configura, OpenClaw intenta detectar un directorio `.git` buscando hacia arriba desde el espacio de trabajo (y el directorio de trabajo actual). La ruta debe existir para poder usarse.

```json5
{
  agents: { defaults: { repoRoot: "~/Projects/openclaw" } }
}
```

<div id="agentsdefaultsskipbootstrap">
  ### `agents.defaults.skipBootstrap`
</div>

Deshabilita la creación automática de los archivos de arranque del espacio de trabajo (`AGENTS.md`, `SOUL.md`, `TOOLS.md`, `IDENTITY.md`, `USER.md` y `BOOTSTRAP.md`).

Utiliza esto para despliegues con datos precargados en los que los archivos de tu espacio de trabajo provienen de un repositorio.

```json5
{
  agents: { defaults: { skipBootstrap: true } }
}
```

<div id="agentsdefaultsbootstrapmaxchars">
  ### `agents.defaults.bootstrapMaxChars`
</div>

Número máximo de caracteres de cada archivo de arranque del espacio de trabajo inyectado en el *system prompt* antes de truncarlo. Valor predeterminado: `20000`.

Cuando un archivo supera este límite, OpenClaw registra una advertencia e inyecta un fragmento inicial y final truncados con un marcador.

```json5
{
  agents: { defaults: { bootstrapMaxChars: 20000 } }
}
```

<div id="agentsdefaultsusertimezone">
  ### `agents.defaults.userTimezone`
</div>

Configura la zona horaria del usuario para el **contexto del prompt de sistema** (no para las marcas de tiempo en las envolturas de mensajes). Si no se establece, OpenClaw usa la zona horaria del host en tiempo de ejecución.

```json5
{
  agents: { defaults: { userTimezone: "America/Chicago" } }
}
```

<div id="agentsdefaultstimeformat">
  ### `agents.defaults.timeFormat`
</div>

Controla el **formato de hora** que se muestra en la sección «Current Date &amp; Time» del prompt del sistema.
Valor predeterminado: `auto` (preferencia del sistema operativo).

```json5
{
  agents: { defaults: { timeFormat: "auto" } } // auto | 12 | 24
}
```

<div id="messages">
  ### `messages`
</div>

Controla los prefijos de mensajes entrantes/salientes y las reacciones de acuse de recibo (ack) opcionales.
Consulta [Messages](/es/concepts/messages) para obtener información sobre encolamiento, sesiones y contexto de streaming.

```json5
{
  messages: {
    responsePrefix: "🦞", // o "auto"
    ackReaction: "👀",
    ackReactionScope: "group-mentions",
    removeAckAfterReply: false
  }
}
```

`responsePrefix` se aplica a **todas las respuestas salientes** (resúmenes de herramientas, streaming
por bloques, respuestas finales) en todos los canales, a menos que ya esté presente.

Si `messages.responsePrefix` no está definido, no se aplica ningún prefijo de forma predeterminada. Las respuestas en chats que te envías a ti mismo en WhatsApp son la excepción: por defecto usan `[{identity.name}]` cuando está configurado, y en caso contrario
`[openclaw]`, para que las conversaciones en el mismo teléfono sigan siendo legibles.
Establécelo en `"auto"` para derivar `[{identity.name}]` para el agente enrutado (cuando esté configurado).

<div id="template-variables">
  #### Variables de plantilla
</div>

La cadena `responsePrefix` puede incluir variables de plantilla que se resuelven dinámicamente:

| Variable          | Descripción                       | Ejemplo                         |
| ----------------- | --------------------------------- | ------------------------------- |
| `{model}`         | Nombre corto del modelo           | `claude-opus-4-5`, `gpt-4o`     |
| `{modelFull}`     | Identificador completo del modelo | `anthropic/claude-opus-4-5`     |
| `{provider}`      | Nombre del proveedor              | `anthropic`, `openai`           |
| `{thinkingLevel}` | Nivel de razonamiento actual      | `high`, `low`, `off`            |
| `{identity.name}` | Nombre de la identidad del Agente | (igual que en el modo `"auto"`) |

Las variables no distinguen mayúsculas de minúsculas (`{MODEL}` = `{model}`). `{think}` es un alias de `{thinkingLevel}`.
Las variables no resueltas permanecen como texto literal.

```json5
{
  messages: {
    responsePrefix: "[{model} | think:{thinkingLevel}]"
  }
}
```

Ejemplo de salida: `[claude-opus-4-5 | think:high] Here's my response...`

El prefijo de entrada de WhatsApp se configura mediante `channels.whatsapp.messagePrefix` (obsoleto:
`messages.messagePrefix`). El valor predeterminado se mantiene **sin cambios**: `"[openclaw]"` cuando
`channels.whatsapp.allowFrom` está vacío, en caso contrario `""` (sin prefijo). Cuando se usa
`"[openclaw]"`, OpenClaw usará `[{identity.name}]` cuando el agente enrutado tenga `identity.name` configurado.

`ackReaction` envía, en la medida de lo posible, una reacción con emoji para reconocer mensajes entrantes
en canales que admiten reacciones (Slack/Discord/Telegram/Google Chat). De manera predeterminada usa
`identity.emoji` del agente activo cuando está definido; en caso contrario, `"👀"`. Configúralo en `""` para desactivarlo.

`ackReactionScope` controla cuándo se envían las reacciones:

* `group-mentions` (predeterminado): solo cuando un grupo/sala requiere menciones **y** el bot fue mencionado
* `group-all`: todos los mensajes de grupo/sala
* `direct`: solo mensajes directos
* `all`: todos los mensajes

`removeAckAfterReply` elimina la reacción de acuse de recibo del bot después de que se envía una respuesta
(solo Slack/Discord/Telegram/Google Chat). Predeterminado: `false`.

<div id="messagestts">
  #### `messages.tts`
</div>

Habilita la conversión de texto a voz para las respuestas salientes. Cuando está habilitada, OpenClaw genera audio usando ElevenLabs u OpenAI y lo adjunta a las respuestas. Telegram usa notas de voz Opus; otros canales envían audio MP3.

```json5
{
  messages: {
    tts: {
      auto: "always", // off | always | inbound | tagged
      mode: "final", // final | all (incluye respuestas de herramientas/bloques)
      provider: "elevenlabs",
      summaryModel: "openai/gpt-4.1-mini",
      modelOverrides: {
        enabled: true
      },
      maxTextLength: 4000,
      timeoutMs: 30000,
      prefsPath: "~/.openclaw/settings/tts.json",
      elevenlabs: {
        apiKey: "elevenlabs_api_key",
        baseUrl: "https://api.elevenlabs.io",
        voiceId: "voice_id",
        modelId: "eleven_multilingual_v2",
        seed: 42,
        applyTextNormalization: "auto",
        languageCode: "en",
        voiceSettings: {
          stability: 0.5,
          similarityBoost: 0.75,
          style: 0.0,
          useSpeakerBoost: true,
          speed: 1.0
        }
      },
      openai: {
        apiKey: "openai_api_key",
        model: "gpt-4o-mini-tts",
        voice: "alloy"
      }
    }
  }
}
```

Notas:

* `messages.tts.auto` controla el TTS automático (`off`, `always`, `inbound`, `tagged`).
* `/tts off|always|inbound|tagged` establece el modo automático por sesión (sobrescribe la configuración).
* `messages.tts.enabled` es un ajuste heredado; `doctor` lo migra a `messages.tts.auto`.
* `prefsPath` almacena sobrescrituras locales (proveedor/límite/resumen).
* `maxTextLength` es un límite estricto para la entrada TTS; los resúmenes se truncan para ajustarse.
* `summaryModel` sobrescribe `agents.defaults.model.primary` para el resumen automático.
  * Acepta `provider/model` o un alias de `agents.defaults.models`.
* `modelOverrides` habilita sobrescrituras basadas en modelo como etiquetas `[[tts:...]]` (activado por defecto).
* `/tts limit` y `/tts summary` controlan la configuración de resumen por usuario.
* Los valores de `apiKey` recurren a `ELEVENLABS_API_KEY`/`XI_API_KEY` y `OPENAI_API_KEY` como respaldo.
* `elevenlabs.baseUrl` sobrescribe la URL base de la API de ElevenLabs.
* `elevenlabs.voiceSettings` admite `stability`/`similarityBoost`/`style` (0..1),
  `useSpeakerBoost` y `speed` (0.5..2.0).

<div id="talk">
  ### `talk`
</div>

Valores predeterminados para el modo Talk (macOS/iOS/Android). Los identificadores de voz usan `ELEVENLABS_VOICE_ID` o `SAG_VOICE_ID` como valor predeterminado cuando no están definidos.
`apiKey` usa `ELEVENLABS_API_KEY` (o el perfil de shell de Gateway) como valor predeterminado cuando no está definida.
`voiceAliases` permite que las directivas de Talk usen nombres fáciles de recordar (por ejemplo, `"voice":"Clawd"`).

```json5
{
  talk: {
    voiceId: "elevenlabs_voice_id",
    voiceAliases: {
      Clawd: "EXAVITQu4vr4xnSDxMaL",
      Roger: "CwhRBWXzGAHq8TQ4Fs17"
    },
    modelId: "eleven_v3",
    outputFormat: "mp3_44100_128",
    apiKey: "elevenlabs_api_key",
    interruptOnSpeech: true
  }
}
```

<div id="agentsdefaults">
  ### `agents.defaults`
</div>

Controla el runtime integrado del agente (modelo/razonamiento/verbose/timeouts).
`agents.defaults.models` define el catálogo de modelos configurados (y actúa como la lista de permitidos para `/model`).
`agents.defaults.model.primary` establece el modelo predeterminado; `agents.defaults.model.fallbacks` son conmutaciones por error globales.
`agents.defaults.imageModel` es opcional y **solo se usa si el modelo primario no admite entrada de imagen**.
Cada entrada de `agents.defaults.models` puede incluir:

* `alias` (atajo de modelo opcional, p. ej. `/opus`).
* `params` (parámetros de API específicos del proveedor, opcionales, que se pasan directamente en la solicitud al modelo).

`params` también se aplica a ejecuciones en streaming (agente integrado + compactación). Actualmente se admiten las claves: `temperature`, `maxTokens`. Estas se combinan con las opciones en tiempo de llamada; los valores proporcionados por quien realiza la llamada prevalecen. `temperature` es un control avanzado: déjalo sin establecer a menos que conozcas los valores predeterminados del modelo y necesites cambiarlos.

Ejemplo:

```json5
{
  agents: {
    defaults: {
      models: {
        "anthropic/claude-sonnet-4-5-20250929": {
          params: { temperature: 0.6 }
        },
        "openai/gpt-5.2": {
          params: { maxTokens: 8192 }
        }
      }
    }
  }
}
```

Los modelos Z.AI GLM-4.x activan automáticamente el modo de pensamiento a menos que:

* establezcas `--thinking off`, o
* definas `agents.defaults.models["zai/<model>"].params.thinking` por tu cuenta.

OpenClaw también incluye algunos alias abreviados integrados. Los valores predeterminados solo se aplican cuando el modelo
ya está presente en `agents.defaults.models`:

* `opus` -&gt; `anthropic/claude-opus-4-5`
* `sonnet` -&gt; `anthropic/claude-sonnet-4-5`
* `gpt` -&gt; `openai/gpt-5.2`
* `gpt-mini` -&gt; `openai/gpt-5-mini`
* `gemini` -&gt; `google/gemini-3-pro-preview`
* `gemini-flash` -&gt; `google/gemini-3-flash-preview`

Si configuras por tu cuenta el mismo nombre de alias (sin distinguir mayúsculas/minúsculas), prevalece tu valor (los valores predeterminados nunca lo sobrescriben).

Ejemplo: Opus 4.5 como principal con MiniMax M2.1 como respaldo (MiniMax alojado):

```json5
{
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-5": { alias: "opus" },
        "minimax/MiniMax-M2.1": { alias: "minimax" }
      },
      model: {
        primary: "anthropic/claude-opus-4-5",
        fallbacks: ["minimax/MiniMax-M2.1"]
      }
    }
  }
}
```

Autenticación de MiniMax: define la variable de entorno `MINIMAX_API_KEY` o configura `models.providers.minimax`.

<div id="agentsdefaultsclibackends-cli-fallback">
  #### `agents.defaults.cliBackends` (CLI fallback)
</div>

Backends de CLI opcionales para ejecuciones de respaldo solo de texto (sin llamadas a herramientas). Son útiles como ruta de respaldo cuando fallan los proveedores de API. Se admite el paso directo de imágenes cuando configuras un `imageArg` que acepta rutas de archivos.

Notas:

* Los backends de CLI son **text-first**; las herramientas siempre están deshabilitadas.
* Las sesiones se admiten cuando se establece `sessionArg`; los ID de sesión se persisten por backend.
* Para `claude-cli`, los valores predeterminados ya vienen preconfigurados. Sobrescribe la ruta del comando si PATH es muy limitado
  (launchd/systemd).

Ejemplo:

```json5
{
  agents: {
    defaults: {
      cliBackends: {
        "claude-cli": {
          command: "/opt/homebrew/bin/claude"
        },
        "my-cli": {
          command: "my-cli",
          args: ["--json"],
          output: "json",
          modelArg: "--model",
          sessionArg: "--session",
          sessionMode: "existing",
          systemPromptArg: "--system",
          systemPromptWhen: "first",
          imageArg: "--image",
          imageMode: "repeat"
        }
      }
    }
  }
}
```

```json5
{
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-5": { alias: "Opus" },
        "anthropic/claude-sonnet-4-1": { alias: "Sonnet" },
        "openrouter/deepseek/deepseek-r1:free": {},
        "zai/glm-4.7": {
          alias: "GLM",
          params: {
            thinking: {
              type: "enabled",
              clear_thinking: false
            }
          }
        }
      },
      model: {
        primary: "anthropic/claude-opus-4-5",
        fallbacks: [
          "openrouter/deepseek/deepseek-r1:free",
          "openrouter/meta-llama/llama-3.3-70b-instruct:free"
        ]
      },
      imageModel: {
        primary: "openrouter/qwen/qwen-2.5-vl-72b-instruct:free",
        fallbacks: [
          "openrouter/google/gemini-2.0-flash-vision:free"
        ]
      },
      thinkingDefault: "low",
      verboseDefault: "off",
      elevatedDefault: "on",
      timeoutSeconds: 600,
      mediaMaxMb: 5,
      heartbeat: {
        every: "30m",
        target: "last"
      },
      maxConcurrent: 3,
      subagents: {
        model: "minimax/MiniMax-M2.1",
        maxConcurrent: 1,
        archiveAfterMinutes: 60
      },
      exec: {
        backgroundMs: 10000,
        timeoutSec: 1800,
        cleanupMs: 1800000
      },
      contextTokens: 200000
    }
  }
}
```

<div id="agentsdefaultscontextpruning-tool-result-pruning">
  #### `agents.defaults.contextPruning` (poda de resultados de herramientas)
</div>

`agents.defaults.contextPruning` elimina **resultados de herramientas antiguos** del contexto en memoria justo antes de que se envíe una solicitud al LLM.
**No** modifica el historial de la sesión en disco (los archivos `*.jsonl` permanecen completos).

Está pensado para reducir el uso de tokens en agentes muy habladores que acumulan grandes salidas de herramientas con el tiempo.

A alto nivel:

* Nunca toca los mensajes de usuario/asistente.
* Protege los últimos mensajes de asistente `keepLastAssistants` (no se podan resultados de herramientas posteriores a ese punto).
* Protege el prefijo de arranque (no se poda nada antes del primer mensaje de usuario).
* Modos:
  * `adaptive`: recorta suavemente resultados de herramientas sobredimensionados (mantiene cabeza/cola) cuando la proporción estimada de contexto supera `softTrimRatio`.
    Luego limpia de forma contundente los resultados de herramientas elegibles más antiguos cuando la proporción estimada de contexto supera `hardClearRatio` **y**
    hay suficiente volumen de resultados de herramientas podables (`minPrunableToolChars`).
  * `aggressive`: siempre reemplaza los resultados de herramientas elegibles antes del corte con `hardClear.placeholder` (sin comprobar proporciones).

Poda suave vs. fuerte (qué cambia en el contexto enviado al LLM):

* **Soft-trim**: solo para resultados de herramientas *sobredimensionados*. Mantiene el principio + final e inserta `...` en el medio.
  * Antes: `toolResult("…very long output…")`
  * Después: `toolResult("HEAD…\n...\n…TAIL\n\n[Tool result trimmed: …]")`
* **Hard-clear**: reemplaza todo el resultado de la herramienta con el marcador de posición.
  * Antes: `toolResult("…very long output…")`
  * Después: `toolResult("[Old tool result content cleared]")`

Notas / limitaciones actuales:

* Los resultados de herramientas que contienen **bloques de imagen se omiten** (por ahora nunca se recortan ni se limpian).
* La “proporción de contexto” estimada se basa en **caracteres** (aproximada), no en tokens exactos.
* Si la sesión aún no contiene al menos `keepLastAssistants` mensajes de asistente, se omite la poda.
* En modo `aggressive`, `hardClear.enabled` se ignora (los resultados de herramientas elegibles siempre se reemplazan con `hardClear.placeholder`).

Valor por defecto (`adaptive`):

```json5
{
  agents: { defaults: { contextPruning: { mode: "adaptive" } } }
}
```

Para desactivar:

```json5
{
  agents: { defaults: { contextPruning: { mode: "off" } } }
}
```

Valores predeterminados (cuando `mode` es `"adaptive"` o `"aggressive"`):

* `keepLastAssistants`: `3`
* `softTrimRatio`: `0.3` (solo con `mode: "adaptive"`)
* `hardClearRatio`: `0.5` (solo con `mode: "adaptive"`)
* `minPrunableToolChars`: `50000` (solo con `mode: "adaptive"`)
* `softTrim`: `{ maxChars: 4000, headChars: 1500, tailChars: 1500 }` (solo con `mode: "adaptive"`)
* `hardClear`: `{ enabled: true, placeholder: "[Old tool result content cleared]" }`

Ejemplo (`"aggressive"`, mínimo):

```json5
{
  agents: { defaults: { contextPruning: { mode: "aggressive" } } }
}
```

Ejemplo (ajuste adaptativo):

```json5
{
  agents: {
    defaults: {
      contextPruning: {
        mode: "adaptive",
        keepLastAssistants: 3,
        softTrimRatio: 0.3,
        hardClearRatio: 0.5,
        minPrunableToolChars: 50000,
        softTrim: { maxChars: 4000, headChars: 1500, tailChars: 1500 },
        hardClear: { enabled: true, placeholder: "[Old tool result content cleared]" },
        // Opcional: restringir la poda a herramientas específicas (deny tiene prioridad; admite comodines "*")
        tools: { deny: ["browser", "canvas"] },
      }
    }
  }
}
```

Consulta [/concepts/session-pruning](/es/concepts/session-pruning) para más detalles sobre el comportamiento.

<div id="agentsdefaultscompaction-reserve-headroom-memory-flush">
  #### `agents.defaults.compaction` (reservar margen + vaciado de memoria)
</div>

`agents.defaults.compaction.mode` selecciona la estrategia de resumen de compactación. El valor predeterminado es `default`; configura `safeguard` para habilitar el resumen por bloques para historiales muy largos. Consulta [/concepts/compaction](/es/concepts/compaction).

`agents.defaults.compaction.reserveTokensFloor` aplica un valor mínimo de `reserveTokens`
para la compactación Pi (valor predeterminado: `20000`). Establécelo en `0` para desactivar este mínimo.

`agents.defaults.compaction.memoryFlush` ejecuta un turno de agente **silencioso** antes
de la compactación automática, indicando al modelo que almacene memorias persistentes en disco (p. ej.
`memory/YYYY-MM-DD.md`). Se activa cuando la estimación de tokens de la sesión supera un
umbral flexible situado por debajo del límite de compactación.

Valores predeterminados heredados:

* `memoryFlush.enabled`: `true`
* `memoryFlush.softThresholdTokens`: `4000`
* `memoryFlush.prompt` / `memoryFlush.systemPrompt`: valores predeterminados integrados con `NO_REPLY`
* Nota: el vaciado de memoria se omite cuando el espacio de trabajo de la sesión es de solo lectura
  (`agents.defaults.sandbox.workspaceAccess: "ro"` o `"none"`).

Ejemplo (ajustado):

```json5
{
  agents: {
    defaults: {
      compaction: {
        mode: "safeguard",
        reserveTokensFloor: 24000,
        memoryFlush: {
          enabled: true,
          softThresholdTokens: 6000,
          systemPrompt: "Session nearing compaction. Store durable memories now.",
          prompt: "Escribe cualquier nota duradera en memory/YYYY-MM-DD.md; responde con NO_REPLY si no hay nada que almacenar."
        }
      }
    }
  }
}
```

Transmisión en bloques:

* `agents.defaults.blockStreamingDefault`: `"on"`/`"off"` (valor predeterminado: `"off"`).
* Anulaciones por canal: `*.blockStreaming` (y variantes por cuenta) para forzar la transmisión en bloques activada/desactivada.
  Los canales que no son de Telegram requieren un `*.blockStreaming: true` explícito para habilitar respuestas en bloque.
* `agents.defaults.blockStreamingBreak`: `"text_end"` o `"message_end"` (valor predeterminado: `text_end`).
* `agents.defaults.blockStreamingChunk`: fragmentación suave para bloques transmitidos. El valor predeterminado es
  800–1200 caracteres; prioriza saltos de párrafo (`\n\n`), luego saltos de línea y, después, oraciones.
  Ejemplo:
  ```json5
  {
    agents: { defaults: { blockStreamingChunk: { minChars: 800, maxChars: 1200 } } }
  }
  ```
* `agents.defaults.blockStreamingCoalesce`: combina bloques transmitidos antes de enviarlos.
  El valor predeterminado es `{ idleMs: 1000 }` y hereda `minChars` de `blockStreamingChunk`,
  con `maxChars` limitado por el máximo de texto del canal. Signal/Slack/Discord/Google Chat usan por defecto
  `minChars: 1500`, a menos que se anule.
  Anulaciones por canal: `channels.whatsapp.blockStreamingCoalesce`, `channels.telegram.blockStreamingCoalesce`,
  `channels.discord.blockStreamingCoalesce`, `channels.slack.blockStreamingCoalesce`, `channels.mattermost.blockStreamingCoalesce`,
  `channels.signal.blockStreamingCoalesce`, `channels.imessage.blockStreamingCoalesce`, `channels.msteams.blockStreamingCoalesce`,
  `channels.googlechat.blockStreamingCoalesce`
  (y variantes por cuenta).
* `agents.defaults.humanDelay`: pausa aleatoria entre **respuestas en bloque** después de la primera.
  Modos: `off` (por defecto), `natural` (800–2500ms), `custom` (utiliza `minMs`/`maxMs`).
  Anulación por agente: `agents.list[].humanDelay`.
  Ejemplo:
  ```json5
  {
    agents: { defaults: { humanDelay: { mode: "natural" } } }
  }
  ```

Consulta [/concepts/streaming](/es/concepts/streaming) para detalles de comportamiento y fragmentación.

Indicadores de escritura:

* `agents.defaults.typingMode`: `"never" | "instant" | "thinking" | "message"`. El valor predeterminado es
  `instant` para chats directos/menciones y `message` para chats grupales sin mención.
* `session.typingMode`: anulación por sesión para el modo.
* `agents.defaults.typingIntervalSeconds`: con qué frecuencia se actualiza la señal de escritura (por defecto: 6s).
* `session.typingIntervalSeconds`: anulación por sesión para el intervalo de actualización.
  Consulta [/concepts/typing-indicators](/es/concepts/typing-indicators) para detalles de comportamiento.

`agents.defaults.model.primary` debe configurarse como `provider/model` (p. ej. `anthropic/claude-opus-4-5`).
Los alias provienen de `agents.defaults.models.*.alias` (p. ej. `Opus`).
Si omites el proveedor, OpenClaw actualmente asume `anthropic` como un mecanismo
de compatibilidad temporal ante la deprecación.
Los modelos de Z.AI están disponibles como `zai/<model>` (p. ej. `zai/glm-4.7`) y requieren
`ZAI_API_KEY` (o el heredado `Z_AI_API_KEY`) en el entorno.

`agents.defaults.heartbeat` configura ejecuciones periódicas de latido:

* `every`: cadena de duración (`ms`, `s`, `m`, `h`); la unidad predeterminada son los minutos. Predeterminado:
  `30m`. Establece `0m` para desactivar.
* `model`: modelo opcional de sobrescritura para las ejecuciones de latido (`provider/model`).
* `includeReasoning`: cuando es `true`, los latidos también entregarán el mensaje separado `Reasoning:` cuando esté disponible (misma forma que `/reasoning on`). Predeterminado: `false`.
* `session`: clave de sesión opcional para controlar en qué sesión se ejecuta el latido. Predeterminado: `main`.
* `to`: reemplazo opcional del destinatario (id específico del canal, p. ej. E.164 para WhatsApp, id de chat para Telegram).
* `target`: canal de entrega opcional (`last`, `whatsapp`, `telegram`, `discord`, `slack`, `msteams`, `signal`, `imessage`, `none`). Predeterminado: `last`.
* `prompt`: sobrescritura opcional para el cuerpo del latido (predeterminado: `Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.`). Las sobrescrituras se envían literalmente; incluye una línea `Read HEARTBEAT.md` si aún quieres que se lea el archivo.
* `ackMaxChars`: máximo de caracteres permitidos después de `HEARTBEAT_OK` antes de la entrega (predeterminado: 300).

Latidos por agente:

* Establece `agents.list[].heartbeat` para habilitar o sobrescribir la configuración de latido para un agente específico.
* Si alguna entrada de agente define `heartbeat`, **solo esos agentes** ejecutan latidos; los valores
  predeterminados se convierten en la base compartida para esos agentes.

Los latidos ejecutan turnos completos del agente. Intervalos más cortos consumen más tokens; ten en cuenta
`every`, mantén `HEARTBEAT.md` muy pequeño y/o elige un `model` más barato.

`tools.exec` configura los valores predeterminados de ejecución en segundo plano:

* `backgroundMs`: tiempo antes de mandar automáticamente a segundo plano (ms, predeterminado 10000)
* `timeoutSec`: auto-terminación tras este tiempo de ejecución (segundos, predeterminado 1800)
* `cleanupMs`: cuánto tiempo mantener las sesiones finalizadas en memoria (ms, predeterminado 1800000)
* `notifyOnExit`: encola un evento de sistema + solicita un latido cuando una ejecución en segundo plano termina (predeterminado true)
* `applyPatch.enabled`: habilita `apply_patch` experimental (solo OpenAI/OpenAI Codex; predeterminado false)
* `applyPatch.allowModels`: lista de permitidos opcional de ids de modelo (p. ej. `gpt-5.2` o `openai/gpt-5.2`)
  Nota: `applyPatch` solo está bajo `tools.exec`.

`tools.web` configura las herramientas de búsqueda web + fetch:

* `tools.web.search.enabled` (predeterminado: true cuando la clave está presente)
* `tools.web.search.apiKey` (recomendado: configurar vía `openclaw configure --section web`, o usar la variable de entorno `BRAVE_API_KEY`)
* `tools.web.search.maxResults` (1–10, predeterminado 5)
* `tools.web.search.timeoutSeconds` (predeterminado 30)
* `tools.web.search.cacheTtlMinutes` (predeterminado 15)
* `tools.web.fetch.enabled` (predeterminado true)
* `tools.web.fetch.maxChars` (predeterminado 50000)
* `tools.web.fetch.timeoutSeconds` (predeterminado 30)
* `tools.web.fetch.cacheTtlMinutes` (predeterminado 15)
* `tools.web.fetch.userAgent` (sobrescritura opcional)
* `tools.web.fetch.readability` (predeterminado true; desactívalo para usar solo una limpieza básica de HTML)
* `tools.web.fetch.firecrawl.enabled` (predeterminado true cuando se establece una API key)
* `tools.web.fetch.firecrawl.apiKey` (opcional; por defecto `FIRECRAWL_API_KEY`)
* `tools.web.fetch.firecrawl.baseUrl` (predeterminado https://api.firecrawl.dev)
* `tools.web.fetch.firecrawl.onlyMainContent` (predeterminado true)
* `tools.web.fetch.firecrawl.maxAgeMs` (opcional)
* `tools.web.fetch.firecrawl.timeoutSeconds` (opcional)

`tools.media` configura la comprensión de medios entrantes (imagen/audio/video):

* `tools.media.models`: lista de modelos compartida (con etiquetas de capacidades; se usa después de las listas por capacidad).
* `tools.media.concurrency`: máximo de ejecuciones de capacidades concurrentes (por defecto 2).
* `tools.media.image` / `tools.media.audio` / `tools.media.video`:
  * `enabled`: interruptor de desactivación (opt-out) (por defecto true cuando hay modelos configurados).
  * `prompt`: prompt opcional de sustitución (image/video añaden automáticamente una indicación `maxChars`).
  * `maxChars`: máximo de caracteres de salida (por defecto 500 para image/video; sin definir para audio).
  * `maxBytes`: tamaño máximo del contenido multimedia a enviar (por defecto: image 10MB, audio 20MB, video 50MB).
  * `timeoutSeconds`: tiempo de espera de la solicitud (por defecto: image 60s, audio 60s, video 120s).
  * `language`: indicación opcional de idioma para audio.
  * `attachments`: política de archivos adjuntos (`mode`, `maxAttachments`, `prefer`).
  * `scope`: limitación opcional (gana la primera coincidencia) con `match.channel`, `match.chatType` o `match.keyPrefix`.
  * `models`: lista ordenada de entradas de modelos; los fallos o contenido multimedia de tamaño excesivo pasan a la siguiente entrada.
* Cada entrada en `models[]`:
  * Entrada de proveedor (`type: "provider"` u omitido):
    * `provider`: id del proveedor de API (`openai`, `anthropic`, `google`/`gemini`, `groq`, etc.).
    * `model`: sustitución del id de modelo (obligatorio para image; por defecto `gpt-4o-mini-transcribe`/`whisper-large-v3-turbo` para proveedores de audio, y `gemini-3-flash-preview` para video).
    * `profile` / `preferredProfile`: selección del perfil de autenticación.
  * Entrada CLI (`type: "cli"`):
    * `command`: ejecutable que se debe ejecutar.
    * `args`: argumentos con plantilla (soporta `{{MediaPath}}`, `{{Prompt}}`, `{{MaxChars}}`, etc.).
  * `capabilities`: lista opcional (`image`, `audio`, `video`) para restringir una entrada compartida. Valores por defecto cuando se omite: `openai`/`anthropic`/`minimax` → image, `google` → image+audio+video, `groq` → audio.
  * `prompt`, `maxChars`, `maxBytes`, `timeoutSeconds`, `language` se pueden sobrescribir por entrada.

Si no hay modelos configurados (o `enabled: false`), se omite el entendimiento; el modelo sigue recibiendo los archivos adjuntos originales.

La autenticación de proveedores sigue el orden estándar de autenticación de modelos (perfiles de autenticación, variables de entorno como `OPENAI_API_KEY`/`GROQ_API_KEY`/`GEMINI_API_KEY`, o `models.providers.*.apiKey`).

Ejemplo:

```json5
{
  tools: {
    media: {
      audio: {
        enabled: true,
        maxBytes: 20971520,
        scope: {
          default: "deny",
          rules: [{ action: "allow", match: { chatType: "direct" } }]
        },
        models: [
          { provider: "openai", model: "gpt-4o-mini-transcribe" },
          { type: "cli", command: "whisper", args: ["--model", "base", "{{MediaPath}}"] }
        ]
      },
      video: {
        enabled: true,
        maxBytes: 52428800,
        models: [{ provider: "google", model: "gemini-3-flash-preview" }]
      }
    }
  }
}
```

`agents.defaults.subagents` configura los valores predeterminados de los subagentes:

* `model`: modelo predeterminado para subagentes creados (string o `{ primary, fallbacks }`). Si se omite, los subagentes heredan el modelo del agente que los invoca, a menos que se sobrescriba por agente o por llamada.
* `maxConcurrent`: número máximo de ejecuciones de subagentes concurrentes (valor predeterminado 1)
* `archiveAfterMinutes`: archivado automático de sesiones de subagente después de N minutos (valor predeterminado 60; establece `0` para desactivar)
* Política de herramientas por subagente: `tools.subagents.tools.allow` / `tools.subagents.tools.deny` (predomina `deny`)

`tools.profile` establece una **lista de permitidos base de herramientas** antes de `tools.allow`/`tools.deny`:

* `minimal`: solo `session_status`
* `coding`: `group:fs`, `group:runtime`, `group:sessions`, `group:memory`, `image`
* `messaging`: `group:messaging`, `sessions_list`, `sessions_history`, `sessions_send`, `session_status`
* `full`: sin restricciones (igual que sin configurar)

Anulación por agente: `agents.list[].tools.profile`.

Ejemplo (de forma predeterminada solo mensajería, permitiendo también herramientas de Slack y Discord):

```json5
{
  tools: {
    profile: "messaging",
    allow: ["slack", "discord"]
  }
}
```

Ejemplo (perfil para programación, pero deniega exec/process en todas partes):

```json5
{
  tools: {
    profile: "coding",
    deny: ["group:runtime"]
  }
}
```

`tools.byProvider` te permite **restringir aún más** las herramientas para proveedores específicos (o un único `provider/model`).
Anulación por agente: `agents.list[].tools.byProvider`.

Orden: perfil base → perfil del proveedor → políticas de permitir/denegar.
Las claves de proveedor admiten tanto `provider` (p. ej., `google-antigravity`) como `provider/model`
(p. ej., `openai/gpt-5.2`).

Ejemplo (mantener el perfil de codificación global, pero con herramientas mínimas para Google Antigravity):

```json5
{
  tools: {
    profile: "coding",
    byProvider: {
      "google-antigravity": { profile: "minimal" }
    }
  }
}
```

Ejemplo (lista de permitidos específica de proveedor/modelo):

```json5
{
  tools: {
    allow: ["group:fs", "group:runtime", "sessions_list"],
    byProvider: {
      "openai/gpt-5.2": { allow: ["group:fs", "sessions_list"] }
    }
  }
}
```

`tools.allow` / `tools.deny` configuran una política global de permitir/denegar herramientas (la denegación tiene prioridad).
La coincidencia de nombres no distingue mayúsculas/minúsculas y admite comodines `*` (`"*"` significa todas las herramientas).
Esto se aplica incluso cuando el sandbox de Docker está **off**.

Ejemplo (desactivar browser/canvas en todas partes):

```json5
{
  tools: { deny: ["browser", "canvas"] }
}
```

Los grupos de herramientas (abreviaturas) funcionan en políticas de herramientas **globales** y **por agente**:

* `group:runtime`: `exec`, `bash`, `process`
* `group:fs`: `read`, `write`, `edit`, `apply_patch`
* `group:sessions`: `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status`
* `group:memory`: `memory_search`, `memory_get`
* `group:web`: `web_search`, `web_fetch`
* `group:ui`: `browser`, `canvas`
* `group:automation`: `cron`, `gateway`
* `group:messaging`: `message`
* `group:nodes`: `nodes`
* `group:openclaw`: todas las herramientas integradas de OpenClaw (excluye complementos de proveedores)

`tools.elevated` controla el acceso elevado (del host) para `exec`:

* `enabled`: permite el modo elevado (valor predeterminado: `true`)
* `allowFrom`: listas de permitidos por canal (vacío = deshabilitado)
  * `whatsapp`: números E.164
  * `telegram`: ID de chat o nombres de usuario
  * `discord`: ID de usuario o nombres de usuario (recurre a `channels.discord.dm.allowFrom` si se omite)
  * `signal`: números E.164
  * `imessage`: identificadores de conversación/ID de chat
  * `webchat`: ID de sesión o nombres de usuario

Ejemplo:

```json5
{
  tools: {
    elevated: {
      enabled: true,
      allowFrom: {
        whatsapp: ["+15555550123"],
        discord: ["steipete", "1234567890123"]
      }
    }
  }
}
```

Anulación por agente (restricción adicional):

```json5
{
  agents: {
    list: [
      {
        id: "family",
        tools: {
          elevated: { enabled: false }
        }
      }
    ]
  }
}
```

Notas:

* `tools.elevated` es la base global. `agents.list[].tools.elevated` solo puede restringir aún más (ambos deben permitir).
* `/elevated on|off|ask|full` almacena el estado por clave de sesión; las directivas en línea se aplican a un único mensaje.
* `exec` elevado se ejecuta en el host y evita el sandbox.
* La política de herramientas sigue aplicándose; si `exec` está denegado, no se puede usar elevated.

`agents.defaults.maxConcurrent` establece el número máximo de ejecuciones de agentes incrustados que pueden
ejecutarse en paralelo a través de sesiones. Cada sesión sigue siendo secuencial (una ejecución
por clave de sesión a la vez). Valor predeterminado: 1.

<div id="agentsdefaultssandbox">
  ### `agents.defaults.sandbox`
</div>

**Sandbox de Docker** opcional para el agente incrustado. Pensado para sesiones
no principales, de modo que no puedan acceder a tu sistema host.

Detalles: [Sandboxing](/es/gateway/sandboxing)

Valores predeterminados (si está habilitado):

* scope: `"agent"` (un contenedor + espacio de trabajo por agente)
* imagen basada en Debian bookworm-slim
* acceso al espacio de trabajo del agente: `workspaceAccess: "none"` (predeterminado)
  * `"none"`: usa un espacio de trabajo de sandbox por ámbito bajo `~/.openclaw/sandboxes`
* `"ro"`: mantiene el espacio de trabajo de la sandbox en `/workspace` y monta el espacio de trabajo del agente en modo de solo lectura en `/agent` (desactiva `write`/`edit`/`apply_patch`)
  * `"rw"`: monta el espacio de trabajo del agente con lectura/escritura en `/workspace`
* limpieza automática (auto-prune): inactividad &gt; 24h O antigüedad &gt; 7d
* política de herramientas: permite solo `exec`, `process`, `read`, `write`, `edit`, `apply_patch`, `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status` (la denegación tiene prioridad)
  * configura mediante `tools.sandbox.tools`, anula por agente mediante `agents.list[].tools.sandbox.tools`
  * abreviaturas de grupos de herramientas compatibles en la política de sandbox: `group:runtime`, `group:fs`, `group:sessions`, `group:memory` (consulta [Sandbox vs Tool Policy vs Elevated](/es/gateway/sandbox-vs-tool-policy-vs-elevated#tool-groups-shorthands))
* navegador opcional en sandbox (Chromium + CDP, observador noVNC)
* opciones de endurecimiento de seguridad: `network`, `user`, `pidsLimit`, `memory`, `cpus`, `ulimits`, `seccompProfile`, `apparmorProfile`

Advertencia: `scope: "shared"` significa un contenedor compartido y un espacio de trabajo
compartido. Sin aislamiento entre sesiones. Usa `scope: "session"` para aislamiento por sesión.

Compatibilidad heredada: `perSession` sigue siendo compatible (`true` → `scope: "session"`,
`false` → `scope: "shared"`).

`setupCommand` se ejecuta **una vez** después de que se crea el contenedor (dentro del contenedor mediante `sh -lc`).
Para instalaciones de paquetes, asegúrate de tener egreso de red, un sistema de archivos raíz escribible y un usuario root.

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main", // off | non-main | all
        scope: "agent", // session | agent | shared (agent is default)
        workspaceAccess: "none", // none | ro | rw
        workspaceRoot: "~/.openclaw/sandboxes",
        docker: {
          image: "openclaw-sandbox:bookworm-slim",
          containerPrefix: "openclaw-sbx-",
          workdir: "/workspace",
          readOnlyRoot: true,
          tmpfs: ["/tmp", "/var/tmp", "/run"],
          network: "none",
          user: "1000:1000",
          capDrop: ["ALL"],
          env: { LANG: "C.UTF-8" },
          setupCommand: "apt-get update && apt-get install -y git curl jq",
          // Anulación por agente (multi-agente): agents.list[].sandbox.docker.*
          pidsLimit: 256,
          memory: "1g",
          memorySwap: "2g",
          cpus: 1,
          ulimits: {
            nofile: { soft: 1024, hard: 2048 },
            nproc: 256
          },
          seccompProfile: "/path/to/seccomp.json",
          apparmorProfile: "openclaw-sandbox",
          dns: ["1.1.1.1", "8.8.8.8"],
          extraHosts: ["internal.service:10.0.0.5"],
          binds: ["/var/run/docker.sock:/var/run/docker.sock", "/home/user/source:/source:rw"]
        },
        browser: {
          enabled: false,
          image: "openclaw-sandbox-browser:bookworm-slim",
          containerPrefix: "openclaw-sbx-browser-",
          cdpPort: 9222,
          vncPort: 5900,
          noVncPort: 6080,
          headless: false,
          enableNoVnc: true,
          allowHostControl: false,
          allowedControlUrls: ["http://10.0.0.42:18791"],
          allowedControlHosts: ["browser.lab.local", "10.0.0.42"],
          allowedControlPorts: [18791],
          autoStart: true,
          autoStartTimeoutMs: 12000
        },
        prune: {
          idleHours: 24,  // 0 disables idle pruning
          maxAgeDays: 7   // 0 disables max-age pruning
        }
      }
    }
  },
  tools: {
    sandbox: {
      tools: {
        allow: ["exec", "process", "read", "write", "edit", "apply_patch", "sessions_list", "sessions_history", "sessions_send", "sessions_spawn", "session_status"],
        deny: ["browser", "canvas", "nodes", "cron", "discord", "gateway"]
      }
    }
  }
}
```

Construye la imagen de sandbox predeterminada una vez con:

```bash
scripts/sandbox-setup.sh
```

Nota: los contenedores de sandbox usan por defecto `network: "none"`; establece `agents.defaults.sandbox.docker.network`
en `"bridge"` (o tu red personalizada) si el agente necesita acceso saliente.

Nota: los archivos adjuntos entrantes se preparan en el espacio de trabajo activo en `media/inbound/*`. Con `workspaceAccess: "rw"`, eso significa que los archivos se escriben en el espacio de trabajo del agente.

Nota: `docker.binds` monta directorios adicionales del host; las vinculaciones globales y por agente se combinan.

Compila la imagen opcional del navegador con:

```bash
scripts/sandbox-browser-setup.sh
```

Cuando `agents.defaults.sandbox.browser.enabled=true`, la herramienta de navegador usa una instancia de Chromium aislada en sandbox (CDP). Si noVNC está habilitado (valor predeterminado cuando headless=false),
la URL de noVNC se inyecta en el prompt del sistema para que el agente pueda referenciarla.
Esto no requiere que `browser.enabled` esté activado en la configuración principal; la URL de control
del sandbox se inyecta por sesión.

`agents.defaults.sandbox.browser.allowHostControl` (valor predeterminado: false) permite
que las sesiones en sandbox apunten explícitamente al servidor de control del navegador del **host**
a través de la herramienta de navegador (`target: "host"`). Deja esta opción desactivada si quieres un aislamiento
estricto del sandbox.

Listas de permitidos para control remoto:

* `allowedControlUrls`: URLs de control exactas permitidas para `target: "custom"`.
* `allowedControlHosts`: nombres de host permitidos (solo nombre de host, sin puerto).
* `allowedControlPorts`: puertos permitidos (valores predeterminados: http=80, https=443).
  Valores predeterminados: todas las listas de permitidos están sin definir (sin restricciones). `allowHostControl` es false de forma predeterminada.

<div id="models-custom-providers-base-urls">
  ### `models` (proveedores personalizados + URLs base)
</div>

OpenClaw usa el catálogo de modelos **pi-coding-agent**. Puedes añadir proveedores personalizados
(LiteLLM, servidores locales compatibles con OpenAI, proxies de Anthropic, etc.) escribiendo
`~/.openclaw/agents/<agentId>/agent/models.json` o definiendo el mismo esquema dentro de tu
configuración de OpenClaw bajo `models.providers`.
Resumen por proveedor + ejemplos: [/concepts/model-providers](/es/concepts/model-providers).

Cuando `models.providers` está presente, OpenClaw escribe/fusiona un `models.json` en
`~/.openclaw/agents/<agentId>/agent/` al inicio:

* comportamiento predeterminado: **merge** (mantiene los proveedores existentes y sobrescribe por nombre)
* establece `models.mode: "replace"` para sobrescribir el contenido del archivo

Selecciona el modelo mediante `agents.defaults.model.primary` (proveedor/modelo).

```json5
{
  agents: {
    defaults: {
      model: { primary: "custom-proxy/llama-3.1-8b" },
      models: {
        "custom-proxy/llama-3.1-8b": {}
      }
    }
  },
  models: {
    mode: "merge",
    providers: {
      "custom-proxy": {
        baseUrl: "http://localhost:4000/v1",
        apiKey: "LITELLM_KEY",
        api: "openai-completions",
        models: [
          {
            id: "llama-3.1-8b",
            name: "Llama 3.1 8B",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 128000,
            maxTokens: 32000
          }
        ]
      }
    }
  }
}
```

<div id="opencode-zen-multi-model-proxy">
  ### OpenCode Zen (proxy multi‑modelo)
</div>

OpenCode Zen es un gateway multi‑modelo con endpoints específicos por modelo. OpenClaw usa
el proveedor integrado `opencode` de pi-ai; configura `OPENCODE_API_KEY` (o
`OPENCODE_ZEN_API_KEY`) desde https://opencode.ai/auth.

Notas:

* Las referencias de modelo usan `opencode/<modelId>` (por ejemplo: `opencode/claude-opus-4-5`).
* Si habilitas una lista de permitidos mediante `agents.defaults.models`, agrega cada modelo que planeas usar.
* Atajo: `openclaw onboard --auth-choice opencode-zen`.

```json5
{
  agents: {
    defaults: {
      model: { primary: "opencode/claude-opus-4-5" },
      models: { "opencode/claude-opus-4-5": { alias: "Opus" } }
    }
  }
}
```

<div id="zai-glm-47-provider-alias-support">
  ### Z.AI (GLM-4.7): compatibilidad con alias de proveedor
</div>

Los modelos de Z.AI están disponibles mediante el proveedor integrado `zai`. Configura `ZAI_API_KEY`
en tu entorno y haz referencia al modelo por proveedor/modelo.

Atajo: `openclaw onboard --auth-choice zai-api-key`.

```json5
{
  agents: {
    defaults: {
      model: { primary: "zai/glm-4.7" },
      models: { "zai/glm-4.7": {} }
    }
  }
}
```

Notas:

* `z.ai/*` y `z-ai/*` son alias aceptados y se normalizan a `zai/*`.
* Si falta `ZAI_API_KEY`, las solicitudes a `zai/*` fallarán con un error de autenticación en tiempo de ejecución.
* Error de ejemplo: `No API key found for provider "zai".`
* El endpoint general de la API de Z.AI es `https://api.z.ai/api/paas/v4`. Las solicitudes de GLM coding
  usan el endpoint dedicado de Coding `https://api.z.ai/api/coding/paas/v4`.
  El proveedor integrado `zai` usa el endpoint de Coding. Si necesitas el endpoint general,
  define un proveedor personalizado en `models.providers` con la sobrescritura de la URL base
  (consulta la sección de proveedores personalizados más arriba).
* Usa un valor ficticio como marcador de posición en la documentación y en las configuraciones; nunca hagas commit de claves de API reales.

<div id="moonshot-ai-kimi">
  ### Moonshot AI (Kimi)
</div>

Usa el endpoint compatible con OpenAI de Moonshot:

```json5
{
  env: { MOONSHOT_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "moonshot/kimi-k2.5" },
      models: { "moonshot/kimi-k2.5": { alias: "Kimi K2.5" } }
    }
  },
  models: {
    mode: "merge",
    providers: {
      moonshot: {
        baseUrl: "https://api.moonshot.ai/v1",
        apiKey: "${MOONSHOT_API_KEY}",
        api: "openai-completions",
        models: [
          {
            id: "kimi-k2.5",
            name: "Kimi K2.5",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 256000,
            maxTokens: 8192
          }
        ]
      }
    }
  }
}
```

Notas:

* Configura `MOONSHOT_API_KEY` en la variable de entorno o usa `openclaw onboard --auth-choice moonshot-api-key`.
* Referencia del modelo: `moonshot/kimi-k2.5`.
* Usa `https://api.moonshot.cn/v1` si necesitas el endpoint de China.

<div id="kimi-code">
  ### Kimi Code
</div>

Usa el endpoint dedicado compatible con OpenAI específico de Kimi Code (independiente de Moonshot):

```json5
{
  env: { KIMICODE_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "kimi-code/kimi-for-coding" },
      models: { "kimi-code/kimi-for-coding": { alias: "Kimi Code" } }
    }
  },
  models: {
    mode: "merge",
    providers: {
      "kimi-code": {
        baseUrl: "https://api.kimi.com/coding/v1",
        apiKey: "${KIMICODE_API_KEY}",
        api: "openai-completions",
        models: [
          {
            id: "kimi-for-coding",
            name: "Kimi For Coding",
            reasoning: true,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 262144,
            maxTokens: 32768,
            headers: { "User-Agent": "KimiCLI/0.77" },
            compat: { supportsDeveloperRole: false }
          }
        ]
      }
    }
  }
}
```

Notas:

* Configura `KIMICODE_API_KEY` en las variables de entorno o usa `openclaw onboard --auth-choice kimi-code-api-key`.
* Referencia del modelo: `kimi-code/kimi-for-coding`.

<div id="synthetic-anthropic-compatible">
  ### Synthetic (compatible con Anthropic)
</div>

Utiliza el endpoint de Synthetic compatible con Anthropic:

```json5
{
  env: { SYNTHETIC_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "synthetic/hf:MiniMaxAI/MiniMax-M2.1" },
      models: { "synthetic/hf:MiniMaxAI/MiniMax-M2.1": { alias: "MiniMax M2.1" } }
    }
  },
  models: {
    mode: "merge",
    providers: {
      synthetic: {
        baseUrl: "https://api.synthetic.new/anthropic",
        apiKey: "${SYNTHETIC_API_KEY}",
        api: "anthropic-messages",
        models: [
          {
            id: "hf:MiniMaxAI/MiniMax-M2.1",
            name: "MiniMax M2.1",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 192000,
            maxTokens: 65536
          }
        ]
      }
    }
  }
}
```

Notas:

* Establece `SYNTHETIC_API_KEY` o usa `openclaw onboard --auth-choice synthetic-api-key`.
* Ref. de modelo: `synthetic/hf:MiniMaxAI/MiniMax-M2.1`.
* La URL base debe omitir `/v1` porque el cliente de Anthropic lo añade automáticamente.

<div id="local-models-lm-studio-recommended-setup">
  ### Modelos locales (LM Studio): configuración recomendada
</div>

Consulta [/gateway/local-models](/es/gateway/local-models) para ver la guía actual sobre modelos locales. TL;DR: ejecuta MiniMax M2.1 mediante la Responses API de LM Studio en hardware potente; mantén los modelos alojados combinados como respaldo.

<div id="minimax-m21">
  ### MiniMax M2.1
</div>

Utiliza MiniMax M2.1 directamente, sin LM Studio:

```json5
{
  agent: {
    model: { primary: "minimax/MiniMax-M2.1" },
    models: {
      "anthropic/claude-opus-4-5": { alias: "Opus" },
      "minimax/MiniMax-M2.1": { alias: "Minimax" }
    }
  },
  models: {
    mode: "merge",
    providers: {
      minimax: {
        baseUrl: "https://api.minimax.io/anthropic",
        apiKey: "${MINIMAX_API_KEY}",
        api: "anthropic-messages",
        models: [
          {
            id: "MiniMax-M2.1",
            name: "MiniMax M2.1",
            reasoning: false,
            input: ["text"],
            // Precios: actualizar en models.json si necesitas seguimiento exacto de costos.
            cost: { input: 15, output: 60, cacheRead: 2, cacheWrite: 10 },
            contextWindow: 200000,
            maxTokens: 8192
          }
        ]
      }
    }
  }
}
```

Notas:

* Configura la variable de entorno `MINIMAX_API_KEY` o usa `openclaw onboard --auth-choice minimax-api`.
* Modelo disponible: `MiniMax-M2.1` (predeterminado).
* Actualiza los precios en `models.json` si necesitas un seguimiento exacto de los costos.

<div id="cerebras-glm-46-47">
  ### Cerebras (GLM 4.6 / 4.7)
</div>

Usa Cerebras mediante su endpoint compatible con OpenAI:

```json5
{
  env: { CEREBRAS_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: {
        primary: "cerebras/zai-glm-4.7",
        fallbacks: ["cerebras/zai-glm-4.6"]
      },
      models: {
        "cerebras/zai-glm-4.7": { alias: "GLM 4.7 (Cerebras)" },
        "cerebras/zai-glm-4.6": { alias: "GLM 4.6 (Cerebras)" }
      }
    }
  },
  models: {
    mode: "merge",
    providers: {
      cerebras: {
        baseUrl: "https://api.cerebras.ai/v1",
        apiKey: "${CEREBRAS_API_KEY}",
        api: "openai-completions",
        models: [
          { id: "zai-glm-4.7", name: "GLM 4.7 (Cerebras)" },
          { id: "zai-glm-4.6", name: "GLM 4.6 (Cerebras)" }
        ]
      }
    }
  }
}
```

Notas:

* Usa `cerebras/zai-glm-4.7` para Cerebras; usa `zai/glm-4.7` para acceso directo a Z.AI.
* Configura `CEREBRAS_API_KEY` en el entorno o en la configuración.

Notas:

* APIs compatibles: `openai-completions`, `openai-responses`, `anthropic-messages`,
  `google-generative-ai`
* Usa `authHeader: true` + `headers` para necesidades de autenticación personalizadas.
* Sobrescribe la raíz de configuración del agente con `OPENCLAW_AGENT_DIR` (o `PI_CODING_AGENT_DIR`)
  si quieres guardar `models.json` en otra ubicación (valor predeterminado: `~/.openclaw/agents/main/agent`).

<div id="session">
  ### `session`
</div>

Controla el ámbito de la sesión, la política de restablecimiento, los desencadenantes de restablecimiento y la ubicación del almacén de sesiones.

```json5
{
  session: {
    scope: "per-sender",
    dmScope: "main",
    identityLinks: {
      alice: ["telegram:123456789", "discord:987654321012345678"]
    },
    reset: {
      mode: "daily",
      atHour: 4,
      idleMinutes: 60
    },
    resetByType: {
      thread: { mode: "daily", atHour: 4 },
      dm: { mode: "idle", idleMinutes: 240 },
      group: { mode: "idle", idleMinutes: 120 }
    },
    resetTriggers: ["/new", "/reset"],
    // Por defecto ya es por agente bajo ~/.openclaw/agents/<agentId>/sessions/sessions.json
    // Puedes sobrescribirlo con plantillas {agentId}:
    store: "~/.openclaw/agents/{agentId}/sessions/sessions.json",
    // Direct chats collapse to agent:<agentId>:<mainKey> (default: "main").
    mainKey: "main",
    agentToAgent: {
      // Max ping-pong reply turns between requester/target (0–5).
      maxPingPongTurns: 5
    },
    sendPolicy: {
      rules: [
        { action: "deny", match: { channel: "discord", chatType: "group" } }
      ],
      default: "allow"
    }
  }
}
```

Fields:

* `mainKey`: clave de bucket de chat directo (valor predeterminado: `"main"`). Útil cuando quieres “renombrar” el hilo principal de DM sin cambiar `agentId`.
  * Nota sobre sandbox: `agents.defaults.sandbox.mode: "non-main"` usa esta clave para detectar la sesión principal. Cualquier clave de sesión que no coincida con `mainKey` (grupos/canales) se ejecuta en sandbox.
* `dmScope`: cómo se agrupan las sesiones de DM (valor predeterminado: `"main"`).
  * `main`: todos los DMs comparten la sesión principal para mantener la continuidad.
  * `per-peer`: aísla los DMs por ID del remitente a través de canales.
  * `per-channel-peer`: aísla los DMs por canal + remitente (recomendado para bandejas de entrada multiusuario).
  * `per-account-channel-peer`: aísla los DMs por cuenta + canal + remitente (recomendado para bandejas de entrada con múltiples cuentas).
* `identityLinks`: asigna IDs canónicos a peers con prefijo de proveedor para que la misma persona comparta una sesión de DM entre canales cuando se usa `per-peer`, `per-channel-peer` o `per-account-channel-peer`.
  * Ejemplo: `alice: ["telegram:123456789", "discord:987654321012345678"]`.
* `reset`: política de reinicio principal. De forma predeterminada realiza reinicios diarios a las 4:00 AM hora local en el host del Gateway.
  * `mode`: `daily` o `idle` (valor predeterminado: `daily` cuando `reset` está presente).
  * `atHour`: hora local (0-23) para el límite de reinicio diario.
  * `idleMinutes`: ventana deslizante de inactividad en minutos. Cuando `daily` e `idle` están ambos configurados, se aplica primero el que expire antes.
* `resetByType`: anulaciones específicas por sesión para `dm`, `group` y `thread`.
  * Si solo configuras el valor heredado `session.idleMinutes` sin ningún `reset`/`resetByType`, OpenClaw permanece en modo solo inactividad por compatibilidad retroactiva.
* `heartbeatIdleMinutes`: anulación opcional de inactividad para comprobaciones de heartbeat (el reinicio diario sigue aplicando cuando está habilitado).
* `agentToAgent.maxPingPongTurns`: máximo de turnos de ida y vuelta entre solicitante/destino (0–5, valor predeterminado 5).
* `sendPolicy.default`: `allow` o `deny` como valor de respaldo cuando ninguna regla coincide.
* `sendPolicy.rules[]`: coincide por `channel`, `chatType` (`direct|group|room`) o `keyPrefix` (por ejemplo, `cron:`). La primera regla `deny` tiene prioridad; en caso contrario, se permite.

<div id="skills-skills-config">
  ### `skills` (configuración de habilidades)
</div>

Controla la lista de permitidos empaquetada, las preferencias de instalación, las carpetas adicionales de habilidades y las anulaciones por habilidad. Se aplica a las habilidades **empaquetadas** y a `~/.openclaw/skills` (las habilidades del espacio de trabajo siguen teniendo prioridad en caso de conflictos de nombre).

Campos:

* `allowBundled`: lista de permitidos opcional solo para habilidades **empaquetadas**. Si se establece, solo esas habilidades empaquetadas son elegibles (las habilidades gestionadas/del espacio de trabajo no se ven afectadas).
* `load.extraDirs`: directorios adicionales de habilidades que se escanearán (menor precedencia).
* `install.preferBrew`: prefiere instaladores de brew cuando estén disponibles (predeterminado: true).
* `install.nodeManager`: preferencia de instalador para Node (`npm` | `pnpm` | `yarn`, predeterminado: npm).
* `entries.<skillKey>`: anulaciones de configuración por habilidad.

Campos por habilidad:

* `enabled`: establece `false` para deshabilitar una habilidad incluso si es empaquetada/está instalada.
* `env`: variables de entorno inyectadas para la ejecución del agente (solo si aún no están definidas).
* `apiKey`: opción práctica para habilidades que declaran una variable de entorno principal (p. ej., `nano-banana-pro` → `GEMINI_API_KEY`).

Ejemplo:

```json5
{
  skills: {
    allowBundled: ["gemini", "peekaboo"],
    load: {
      extraDirs: [
        "~/Projects/agent-scripts/skills",
        "~/Projects/oss/some-skill-pack/skills"
      ]
    },
    install: {
      preferBrew: true,
      nodeManager: "npm"
    },
    entries: {
      "nano-banana-pro": {
        apiKey: "GEMINI_KEY_HERE",
        env: {
          GEMINI_API_KEY: "GEMINI_KEY_HERE"
        }
      },
      peekaboo: { enabled: true },
      sag: { enabled: false }
    }
  }
}
```

<div id="plugins-extensions">
  ### `plugins` (extensiones)
</div>

Controla el descubrimiento de complementos, su permiso/denegación y la configuración por complemento. Los complementos se cargan
desde `~/.openclaw/extensions`, `<workspace>/.openclaw/extensions`, además de cualquier
entrada en `plugins.load.paths`. **Los cambios de configuración requieren reiniciar el Gateway.**
Consulta [/plugin](/es/plugin) para ver el uso completo.

Campos:

* `enabled`: interruptor principal para la carga de complementos (valor predeterminado: true).
* `allow`: lista de permitidos opcional de ids de complementos; cuando está definida, solo se cargan los complementos listados.
* `deny`: lista de bloqueados opcional de ids de complementos (la denegación tiene prioridad).
* `load.paths`: archivos o directorios de complementos adicionales que cargar (absolutos o `~`).
* `entries.<pluginId>`: anulaciones por complemento.
  * `enabled`: establece `false` para desactivar.
  * `config`: objeto de configuración específico del complemento (validado por el complemento si se proporciona).

Ejemplo:

```json5
{
  plugins: {
    enabled: true,
    allow: ["voice-call"],
    load: {
      paths: ["~/Projects/oss/voice-call-extension"]
    },
    entries: {
      "voice-call": {
        enabled: true,
        config: {
          provider: "twilio"
        }
      }
    }
  }
}
```

<div id="browser-openclaw-managed-browser">
  ### `browser` (navegador administrado por openclaw)
</div>

OpenClaw puede iniciar una instancia de Chrome/Brave/Edge/Chromium **dedicada y aislada** para openclaw y exponer un pequeño servicio de control en loopback.
Los perfiles pueden apuntar a un navegador basado en Chromium **remoto** mediante `profiles.<name>.cdpUrl`. Los perfiles remotos son solo de conexión («attach-only»; las operaciones de inicio/detención/restablecimiento están deshabilitadas).

`browser.cdpUrl` se mantiene para configuraciones heredadas de un solo perfil y como esquema/host base
para perfiles que solo configuran `cdpPort`.

Valores predeterminados:

* enabled: `true`
* evaluateEnabled: `true` (configura `false` para deshabilitar `act:evaluate` y `wait --fn`)
* servicio de control: solo loopback (puerto derivado de `gateway.port`, valor predeterminado `18791`)
* URL de CDP: `http://127.0.0.1:18792` (servicio de control + 1, configuración heredada de un solo perfil)
* color de perfil: `#FF4500` (naranja langosta)
* Nota: el servidor de control es iniciado por el Gateway en ejecución (barra de menús de OpenClaw.app o `openclaw gateway`).
* Orden de detección automática: navegador predeterminado si está basado en Chromium; de lo contrario Chrome → Brave → Edge → Chromium → Chrome Canary.

```json5
{
  browser: {
    enabled: true,
    evaluateEnabled: true,
    // cdpUrl: "http://127.0.0.1:18792", // legacy single-profile override
    defaultProfile: "chrome",
    profiles: {
      openclaw: { cdpPort: 18800, color: "#FF4500" },
      work: { cdpPort: 18801, color: "#0066CC" },
      remote: { cdpUrl: "http://10.0.0.42:9222", color: "#00AA00" }
    },
    color: "#FF4500",
    // Advanced:
    // headless: false,
    // noSandbox: false,
    // executablePath: "/Applications/Brave Browser.app/Contents/MacOS/Brave Browser",
    // attachOnly: false, // establecer como true al hacer túnel de un CDP remoto hacia localhost
  }
}
```

<div id="ui-appearance">
  ### `ui` (Apariencia)
</div>

Color de acento opcional usado por las aplicaciones nativas para el chrome de la UI (por ejemplo, el tinte de la burbuja de Talk Mode).

Si no se establece, los clientes usarán un azul claro apagado.

```json5
{
  ui: {
    seamColor: "#FF4500", // hex (RRGGBB or #RRGGBB)
    // Opcional: Anulación de identidad del asistente de Control UI.
    // Si no se establece, Control UI usa la identidad del agente activo (config o IDENTITY.md).
    assistant: {
      name: "OpenClaw",
      avatar: "CB" // emoji, short text, or image URL/data URI
    }
  }
}
```

<div id="gateway-gateway-server-mode-bind">
  ### `gateway` (modo servidor del Gateway + bind)
</div>

Usa `gateway.mode` para declarar explícitamente si esta máquina debe ejecutar el Gateway.

Valores predeterminados:

* modo: **sin definir** (se interpreta como “no iniciar automáticamente”)
* bind: `loopback`
* port: `18789` (puerto único para WS + HTTP)

```json5
{
  gateway: {
    mode: "local", // or "remote"
    port: 18789, // WS + HTTP multiplex
    bind: "loopback",
    // controlUi: { enabled: true, basePath: "/openclaw" }
    // auth: { mode: "token", token: "your-token" } // el token controla el acceso a WS + Control UI
    // tailscale: { mode: "off" | "serve" | "funnel" }
  }
}
```

Ruta base de la Control UI:

* `gateway.controlUi.basePath` establece el prefijo de la URL donde se sirve la Control UI.
* Ejemplos: `"/ui"`, `"/openclaw"`, `"/apps/openclaw"`.
* Predeterminado: raíz (`/`) (sin cambios).
* `gateway.controlUi.allowInsecureAuth` permite autenticación solo por token para la Control UI cuando
  se omite la identidad del dispositivo (normalmente sobre HTTP). Predeterminado: `false`. Se recomienda usar HTTPS
  (Tailscale Serve) o `127.0.0.1`.
* `gateway.controlUi.dangerouslyDisableDeviceAuth` deshabilita las comprobaciones de identidad de dispositivo para la
  Control UI (solo token/contraseña). Predeterminado: `false`. Solo para casos de emergencia.

Documentación relacionada:

* [Control UI](/es/web/control-ui)
* [Descripción general web](/es/web)
* [Tailscale](/es/gateway/tailscale)
* [Acceso remoto](/es/gateway/remote)

Proxies de confianza:

* `gateway.trustedProxies`: lista de direcciones IP de proxies inversos que terminan TLS delante del Gateway.
* Cuando una conexión proviene de una de estas IP, OpenClaw usa `x-forwarded-for` (o `x-real-ip`) para determinar la IP del cliente para comprobaciones de emparejamiento local y comprobaciones HTTP de auth/local.
* Enumera solo proxies que controles completamente y asegúrate de que **sobrescriban** cualquier `x-forwarded-for` entrante.

Notas:

* `openclaw gateway` rechaza iniciarse a menos que `gateway.mode` esté establecido en `local` (o pases la flag de override).
* `gateway.port` controla el único puerto multiplexado usado para WebSocket + HTTP (Control UI, hooks, A2UI).
* Endpoint de OpenAI Chat Completions: **deshabilitado de forma predeterminada**; habilítalo con `gateway.http.endpoints.chatCompletions.enabled: true`.
* Precedencia: `--port` &gt; `OPENCLAW_GATEWAY_PORT` &gt; `gateway.port` &gt; predeterminado `18789`.
* La autenticación del Gateway es obligatoria de forma predeterminada (token/contraseña o identidad de Tailscale Serve). Los bindings que no sean de loopback requieren un token/contraseña compartidos.
* El asistente de incorporación genera un token de gateway de forma predeterminada (incluso en loopback).
* `gateway.remote.token` es **solo** para llamadas CLI remotas; no habilita la autenticación local del Gateway. `gateway.token` se ignora.

Autenticación y Tailscale:

* `gateway.auth.mode` establece los requisitos del handshake (`token` o `password`). Si no se establece, se asume autenticación por token.
* `gateway.auth.token` almacena el token compartido para la autenticación por token (usado por la CLI en la misma máquina).
* Cuando `gateway.auth.mode` está establecido, solo se acepta ese método (más cabeceras opcionales de Tailscale).
* `gateway.auth.password` puede establecerse aquí o mediante `OPENCLAW_GATEWAY_PASSWORD` (recomendado).
* `gateway.auth.allowTailscale` permite que las cabeceras de identidad de Tailscale Serve
  (`tailscale-user-login`) satisfagan la autenticación cuando la solicitud llega por loopback
  con `x-forwarded-for`, `x-forwarded-proto` y `x-forwarded-host`. OpenClaw
  verifica la identidad resolviendo la dirección `x-forwarded-for` mediante
  `tailscale whois` antes de aceptarla. Cuando es `true`, las solicitudes Serve no necesitan
  token/contraseña; establece `false` para requerir credenciales explícitas. Es `true` de forma
  predeterminada cuando `tailscale.mode = "serve"` y el modo de auth no es `password`.
* `gateway.tailscale.mode: "serve"` usa Tailscale Serve (solo tailnet, bind en loopback).
* `gateway.tailscale.mode: "funnel"` expone el dashboard públicamente; requiere autenticación.
* `gateway.tailscale.resetOnExit` restablece la configuración de Serve/Funnel al apagarse.

Valores predeterminados del cliente remoto (CLI):

* `gateway.remote.url` establece la URL WS predeterminada del Gateway para las llamadas de la CLI cuando `gateway.mode = "remote"`.
* `gateway.remote.transport` selecciona el transporte remoto de macOS (`ssh` por defecto, `direct` para ws/wss). Cuando es `direct`, `gateway.remote.url` debe usar `ws://` o `wss://`. `ws://host` usa por defecto el puerto `18789`.
* `gateway.remote.token` proporciona el token para llamadas remotas (déjalo sin establecer para no usar autenticación).
* `gateway.remote.password` proporciona la contraseña para llamadas remotas (déjalo sin establecer para no usar autenticación).

Comportamiento de la app de macOS:

* OpenClaw.app supervisa `~/.openclaw/openclaw.json` y cambia de modo en vivo cuando cambian `gateway.mode` o `gateway.remote.url`.
* Si `gateway.mode` no está definido pero `gateway.remote.url` sí, la app de macOS lo trata como modo remoto.
* Cuando cambias el modo de conexión en la app de macOS, esta escribe `gateway.mode` (y `gateway.remote.url` + `gateway.remote.transport` en modo remoto) de vuelta en el archivo de configuración.

```json5
{
  gateway: {
    mode: "remote",
    remote: {
      url: "ws://gateway.tailnet:18789",
      token: "your-token",
      password: "your-password"
    }
  }
}
```

Ejemplo de transporte directo (aplicación de macOS):

```json5
{
  gateway: {
    mode: "remote",
    remote: {
      transport: "direct",
      url: "wss://gateway.example.ts.net",
      token: "your-token"
    }
  }
}
```

<div id="gatewayreload-config-hot-reload">
  ### `gateway.reload` (Recarga en caliente de la configuración)
</div>

El Gateway supervisa `~/.openclaw/openclaw.json` (o `OPENCLAW_CONFIG_PATH`) y aplica los cambios automáticamente.

Modos:

* `hybrid` (predeterminado): aplica en caliente los cambios seguros; reinicia el Gateway para los cambios críticos.
* `hot`: solo aplica los cambios seguros para recarga en caliente; escribe en el registro cuando se requiere un reinicio.
* `restart`: reinicia el Gateway ante cualquier cambio de configuración.
* `off`: desactiva la recarga en caliente.

```json5
{
  gateway: {
    reload: {
      mode: "hybrid",
      debounceMs: 300
    }
  }
}
```

<div id="hot-reload-matrix-files-impact">
  #### Matriz de recarga en caliente (archivos e impacto)
</div>

Archivos monitorizados:

* `~/.openclaw/openclaw.json` (o `OPENCLAW_CONFIG_PATH`)

Aplicado en caliente (sin reinicio completo del Gateway):

* `hooks` (autenticación/ruta/asignaciones de webhook) + `hooks.gmail` (observador de Gmail reiniciado)
* `browser` (reinicio del servidor de control del navegador)
* `cron` (reinicio del servicio cron + actualización de concurrencia)
* `agents.defaults.heartbeat` (reinicio del proceso de latido)
* `web` (reinicio del canal web de WhatsApp)
* `telegram`, `discord`, `signal`, `imessage` (reinicios de canales)
* `agent`, `models`, `routing`, `messages`, `session`, `whatsapp`, `logging`, `skills`, `ui`, `talk`, `identity`, `wizard` (lecturas dinámicas)

Requiere reinicio completo del Gateway:

* `gateway` (puerto/bind/auth/Control UI/Tailscale)
* `bridge` (legado)
* `discovery`
* `canvasHost`
* `plugins`
* Cualquier ruta de configuración desconocida/no compatible (por seguridad, se reinicia por defecto)

<div id="multi-instance-isolation">
  ### Aislamiento de múltiples instancias
</div>

Para ejecutar múltiples instancias de Gateway en un solo host (para redundancia o un bot de rescate), aísla el estado y la configuración por instancia y usa puertos únicos:

* `OPENCLAW_CONFIG_PATH` (configuración por instancia)
* `OPENCLAW_STATE_DIR` (sesiones/credenciales)
* `agents.defaults.workspace` (memorias)
* `gateway.port` (único por instancia)

Flags de conveniencia (CLI):

* `openclaw --dev …` → usa `~/.openclaw-dev` y desplaza los puertos a partir de la base `19001`
* `openclaw --profile <name> …` → usa `~/.openclaw-<name>` (puerto vía config/env/flags)

Consulta el [runbook de Gateway](/es/gateway) para la asignación de puertos resultante (gateway/navegador/canvas).
Consulta [Múltiples gateways](/es/gateway/multiple-gateways) para detalles sobre el aislamiento de puertos para navegador/CDP.

Ejemplo:

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json \
OPENCLAW_STATE_DIR=~/.openclaw-a \
openclaw gateway --port 19001
```

<div id="hooks-gateway-webhooks">
  ### `hooks` (webhooks del Gateway)
</div>

Habilita un endpoint HTTP de webhook sencillo en el servidor HTTP del Gateway.

Valores predeterminados:

* enabled: `false`
* path: `/hooks`
* maxBodyBytes: `262144` (256 KB)

```json5
{
  hooks: {
    enabled: true,
    token: "shared-secret",
    path: "/hooks",
    presets: ["gmail"],
    transformsDir: "~/.openclaw/hooks",
    mappings: [
      {
        match: { path: "gmail" },
        action: "agent",
        wakeMode: "now",
        name: "Gmail",
        sessionKey: "hook:gmail:{{messages[0].id}}",
        messageTemplate:
          "From: {{messages[0].from}}\nSubject: {{messages[0].subject}}\n{{messages[0].snippet}}",
        deliver: true,
        channel: "last",
        model: "openai/gpt-5.2-mini",
      },
    ],
  }
}
```

Las solicitudes deben incluir el token del hook:

* `Authorization: Bearer <token>` **o**
* `x-openclaw-token: <token>` **o**
* `?token=<token>`

Endpoints:

* `POST /hooks/wake` → `{ text, mode?: "now"|"next-heartbeat" }`
* `POST /hooks/agent` → `{ message, name?, sessionKey?, wakeMode?, deliver?, channel?, to?, model?, thinking?, timeoutSeconds? }`
* `POST /hooks/<name>` → resuelto vía `hooks.mappings`

`/hooks/agent` siempre publica un resumen en la sesión principal (y opcionalmente puede activar un latido inmediato vía `wakeMode: "now"`).

Notas de mapeo:

* `match.path` coincide con la subruta después de `/hooks` (p. ej. `/hooks/gmail` → `gmail`).
* `match.source` coincide con un campo del payload (p. ej. `{ source: "gmail" }`) para que puedas usar una ruta genérica `/hooks/ingest`.
* Plantillas como `{{messages[0].subject}}` leen desde el payload.
* `transform` puede apuntar a un módulo JS/TS que devuelve una acción de hook.
* `deliver: true` envía la respuesta final a un canal; `channel` tiene como valor predeterminado `last` (con WhatsApp como opción de reserva).
* Si no hay una ruta de entrega existente, establece `channel` + `to` explícitamente (requerido para Telegram/Discord/Google Chat/Slack/Signal/iMessage/MS Teams).
* `model` anula el LLM para esta ejecución del hook (`provider/model` o alias; debe estar permitido si `agents.defaults.models` está configurado).

Configuración auxiliar de Gmail (usada por `openclaw webhooks gmail setup` / `run`):

```json5
{
  hooks: {
    gmail: {
      account: "openclaw@gmail.com",
      topic: "projects/<project-id>/topics/gog-gmail-watch",
      subscription: "gog-gmail-watch-push",
      pushToken: "shared-push-token",
      hookUrl: "http://127.0.0.1:18789/hooks/gmail",
      includeBody: true,
      maxBytes: 20000,
      renewEveryMinutes: 720,
      serve: { bind: "127.0.0.1", port: 8788, path: "/" },
      tailscale: { mode: "funnel", path: "/gmail-pubsub" },

      // Opcional: usar un modelo más económico para el procesamiento de hooks de Gmail
      // Recurre a agents.defaults.model.fallbacks, luego al primario, en caso de auth/rate-limit/timeout
      model: "openrouter/meta-llama/llama-3.3-70b-instruct:free",
      // Optional: default thinking level for Gmail hooks
      thinking: "off",
    }
  }
}
```

Sobrescritura de modelo para hooks de Gmail:

* `hooks.gmail.model` especifica un modelo que se usará para el procesamiento de hooks de Gmail (por defecto, el principal de la sesión).
* Acepta referencias `provider/model` o alias de `agents.defaults.models`.
* Recurre a `agents.defaults.model.fallbacks` y luego a `agents.defaults.model.primary` en casos de errores de autenticación, límites de tasa o timeouts.
* Si `agents.defaults.models` está configurado, incluye el modelo de hooks en la lista de permitidos.
* Al inicio, muestra una advertencia si el modelo configurado no está en el catálogo de modelos o en la lista de permitidos.
* `hooks.gmail.thinking` establece el nivel de thinking predeterminado para los hooks de Gmail y puede ser sobrescrito por un `thinking` definido por hook.

Inicio automático del Gateway:

* Si `hooks.enabled=true` y `hooks.gmail.account` está configurado, el Gateway inicia
  `gog gmail watch serve` al arrancar y renueva automáticamente el watch.
* Establece `OPENCLAW_SKIP_GMAIL_WATCHER=1` para desactivar el inicio automático (para ejecuciones manuales).
* Evita ejecutar una instancia separada de `gog gmail watch serve` junto al Gateway; fallará con
  `listen tcp 127.0.0.1:8788: bind: address already in use`.

Nota: cuando `tailscale.mode` está activado, OpenClaw establece por defecto `serve.path` en `/` para que
Tailscale pueda hacer proxy de `/gmail-pubsub` correctamente (elimina el prefijo de ruta configurado).
Si necesitas que el backend reciba la ruta con prefijo, configura
`hooks.gmail.tailscale.target` con una URL completa (y alinea `serve.path`).

<div id="canvashost-lantailnet-canvas-file-server-live-reload">
  ### `canvasHost` (servidor de archivos Canvas en LAN/Tailnet + recarga en vivo)
</div>

El Gateway sirve un directorio de HTML/CSS/JS sobre HTTP para que los nodos iOS/Android puedan simplemente ejecutar `canvas.navigate` hacia él.

Raíz predeterminada: `~/.openclaw/workspace/canvas`
Puerto predeterminado: `18793` (elegido para evitar el puerto CDP del navegador de openclaw `18792`)
El servidor escucha en el **gateway bind host** (LAN o Tailnet) para que los nodos puedan alcanzarlo.

El servidor:

* sirve archivos bajo `canvasHost.root`
* inyecta un pequeño cliente de recarga en vivo en el HTML servido
* vigila el directorio y transmite recargas a través de un endpoint WebSocket en `/__openclaw__/ws`
* crea automáticamente un `index.html` inicial cuando el directorio está vacío (para que veas algo de inmediato)
* también sirve A2UI en `/__openclaw__/a2ui/` y se anuncia a los nodos como `canvasHostUrl`
  (siempre utilizado por los nodos para Canvas/A2UI)

Desactiva la recarga en vivo (y la vigilancia de archivos) si el directorio es grande o recibes errores `EMFILE`:

* config: `canvasHost: { liveReload: false }`

```json5
{
  canvasHost: {
    root: "~/.openclaw/workspace/canvas",
    port: 18793,
    liveReload: true
  }
}
```

Los cambios en `canvasHost.*` requieren reiniciar el Gateway (al recargar la configuración se reiniciará).

Deshabilítalo con:

* config: `canvasHost: { enabled: false }`
* env: `OPENCLAW_SKIP_CANVAS_HOST=1`

<div id="bridge-legacy-tcp-bridge-removed">
  ### `bridge` (puente TCP heredado, eliminado)
</div>

Las compilaciones actuales ya no incluyen el listener del puente TCP; las claves de configuración `bridge.*` se ignoran.
Los nodos se conectan a través del WebSocket del Gateway. Esta sección se mantiene como referencia histórica.

Comportamiento heredado:

* El Gateway podía exponer un puente TCP sencillo para nodos (iOS/Android), normalmente en el puerto `18790`.

Valores predeterminados:

* enabled: `true`
* port: `18790`
* bind: `lan` (se asocia a `0.0.0.0`)

Modos de bind:

* `lan`: `0.0.0.0` (accesible en cualquier interfaz, incluida LAN/Wi‑Fi y Tailscale)
* `tailnet`: se asocia solo a la IP de Tailscale de la máquina (recomendado para Vienna ⇄ London)
* `loopback`: `127.0.0.1` (solo local)
* `auto`: prefiere la IP de tailnet si está presente; en caso contrario, `lan`

TLS:

* `bridge.tls.enabled`: habilita TLS para las conexiones del puente (solo TLS cuando está habilitado).
* `bridge.tls.autoGenerate`: genera un certificado autofirmado cuando no hay cert/key presentes (valor predeterminado: true).
* `bridge.tls.certPath` / `bridge.tls.keyPath`: rutas PEM para el certificado del puente y la clave privada.
* `bridge.tls.caPath`: bundle PEM de CA opcional (raíces personalizadas o futuro mTLS).

Cuando TLS está habilitado, el Gateway anuncia `bridgeTls=1` y `bridgeTlsSha256` en los registros TXT
de discovery para que los nodos puedan fijar (pin) el certificado. Las conexiones manuales usan trust-on-first-use
(confiar en el primer uso) si aún no se ha almacenado ninguna huella digital.
Los certificados generados automáticamente requieren `openssl` en PATH; si la generación falla, el puente no se iniciará.

```json5
{
  bridge: {
    enabled: true,
    port: 18790,
    bind: "tailnet",
    tls: {
      enabled: true,
      // Utiliza ~/.openclaw/bridge/tls/bridge-{cert,key}.pem cuando se omite.
      // certPath: "~/.openclaw/bridge/tls/bridge-cert.pem",
      // keyPath: "~/.openclaw/bridge/tls/bridge-key.pem"
    }
  }
}
```

<div id="discoverymdns-bonjour-mdns-broadcast-mode">
  ### `discovery.mdns` (modo de difusión Bonjour / mDNS)
</div>

Controla los anuncios de descubrimiento mDNS en la LAN (`_openclaw-gw._tcp`).

* `minimal` (predeterminado): omite `cliPath` + `sshPort` de los registros TXT
* `full`: incluye `cliPath` + `sshPort` en los registros TXT
* `off`: desactiva completamente los anuncios mDNS
* Nombre de host: el valor predeterminado es `openclaw` (anuncia `openclaw.local`). Puedes sobrescribir este valor con `OPENCLAW_MDNS_HOSTNAME`.

```json5
{
  discovery: { mdns: { modo: "minimal" } }
}
```

<div id="discoverywidearea-wide-area-bonjour-unicast-dnssd">
  ### `discovery.wideArea` (Bonjour de área amplia / DNS‑SD unicast)
</div>

Cuando está habilitado, el Gateway escribe una zona DNS-SD unicast para `_openclaw-gw._tcp` en `~/.openclaw/dns/` usando el dominio de descubrimiento configurado (por ejemplo: `openclaw.internal.`).

Para que iOS/Android puedan descubrir a través de redes (Viena ⇄ Londres), combínalo con:

* un servidor DNS en el host del Gateway que sirva el dominio que hayas elegido (se recomienda CoreDNS)
* **split DNS** de Tailscale para que los clientes resuelvan ese dominio a través del servidor DNS del Gateway

Asistente de configuración de una sola vez (host del Gateway):

```bash
openclaw dns setup --apply
```

```json5
{
  discovery: { wideArea: { enabled: true } }
}
```

## Variables de plantilla

Los marcadores de posición de plantilla se expanden en `tools.media.*.models[].args` y `tools.media.models[].args` (y en cualquier campo de argumentos con plantilla en el futuro).

| Variable | Descripción |
|----------|-------------|
| `{{Body}}` | Cuerpo completo del mensaje entrante |
| `{{RawBody}}` | Cuerpo sin procesar del mensaje entrante (sin envoltorios de historial/remitente; mejor para análisis de comandos) |
| `{{BodyStripped}}` | Cuerpo con menciones de grupo eliminadas (mejor valor predeterminado para agentes) |
| `{{From}}` | Identificador del remitente (E.164 para WhatsApp; puede variar según el canal) |
| `{{To}}` | Identificador de destino |
| `{{MessageSid}}` | ID del mensaje del canal (cuando está disponible) |
| `{{SessionId}}` | UUID de la sesión actual |
| `{{IsNewSession}}` | `"true"` cuando se crea una nueva sesión |
| `{{MediaUrl}}` | Pseudo-URL de medio entrante (si existe) |
| `{{MediaPath}}` | Ruta local del medio (si se descargó) |
| `{{MediaType}}` | Tipo de medio (imagen/audio/documento/…) |
| `{{Transcript}}` | Transcripción de audio (cuando está habilitada) |
| `{{Prompt}}` | Prompt de medios determinado para entradas de CLI |
| `{{MaxChars}}` | Máximo de caracteres de salida determinado para entradas de CLI |
| `{{ChatType}}` | `"direct"` o `"group"` |
| `{{GroupSubject}}` | Asunto del grupo (en la medida de lo posible) |
| `{{GroupMembers}}` | Vista previa de los miembros del grupo (en la medida de lo posible) |
| `{{SenderName}}` | Nombre visible del remitente (en la medida de lo posible) |
| `{{SenderE164}}` | Número de teléfono del remitente (en la medida de lo posible) |
| `{{Provider}}` | Indicador de proveedor (whatsapp|telegram|discord|googlechat|slack|signal|imessage|msteams|webchat|…) |

<div id="cron-gateway-scheduler">
  ## Cron (planificador del Gateway)
</div>

Cron es un planificador propio del Gateway para activaciones y trabajos programados. Consulta la sección [Trabajos de Cron](/es/automation/cron-jobs) para una descripción general de esta funcionalidad y ejemplos de uso de la CLI.

```json5
{
  cron: {
    enabled: true,
    maxConcurrentRuns: 2
  }
}
```

***

*Siguiente: [Tiempo de ejecución del agente](/es/concepts/agent)* 🦞
