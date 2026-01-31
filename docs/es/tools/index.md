---
title: Herramientas
summary: "Superficie de herramientas del agente para OpenClaw (navegador, canvas, nodos, mensaje, cron) que reemplaza las habilidades heredadas `openclaw-*`"
read_when:
  - Agregar o modificar herramientas de agente
  - Retirar o modificar las habilidades `openclaw-*`
---

<div id="tools-openclaw">
  # Herramientas (OpenClaw)
</div>

OpenClaw expone **herramientas de agente de primera clase** para navegador, lienzo, nodos y cron.
Estas reemplazan las antiguas habilidades `openclaw-*`: las herramientas tienen tipos definidos, sin llamadas al shell,
y el agente debe basarse directamente en ellas.

<div id="disabling-tools">
  ## Desactivación de herramientas
</div>

Puedes permitir o denegar herramientas a nivel global mediante `tools.allow` / `tools.deny` en `openclaw.json`
(deny tiene prioridad). Esto impide que las herramientas no autorizadas se envíen a los proveedores de modelos.

```json5
{
  tools: { deny: ["browser"] }
}
```

Notas:

* La comparación no distingue entre mayúsculas y minúsculas.
* Se admiten comodines `*` (`"*"` significa todas las herramientas).
* Si `tools.allow` solo hace referencia a nombres de herramientas de complementos desconocidos o no cargados, OpenClaw registra una advertencia e ignora la lista de permitidos para que las herramientas centrales sigan disponibles.

<div id="tool-profiles-base-allowlist">
  ## Perfiles de herramientas (lista de permitidos base)
</div>

`tools.profile` establece una **lista de permitidos base de herramientas** antes de `tools.allow`/`tools.deny`.
Sobrescritura por agente: `agents.list[].tools.profile`.

Perfiles:

* `minimal`: solo `session_status`
* `coding`: `group:fs`, `group:runtime`, `group:sessions`, `group:memory`, `image`
* `messaging`: `group:messaging`, `sessions_list`, `sessions_history`, `sessions_send`, `session_status`
* `full`: sin restricción (igual que no configurado)

Ejemplo (solo mensajería por defecto, permitiendo también herramientas de Slack + Discord):

```json5
{
  tools: {
    profile: "messaging",
    allow: ["slack", "discord"]
  }
}
```

Ejemplo (perfil de programación, pero con exec/process denegados en todas partes):

```json5
{
  tools: {
    profile: "coding",
    deny: ["group:runtime"]
  }
}
```

Ejemplo (perfil de programación global, agente de soporte solo por mensajería):

```json5
{
  tools: { profile: "coding" },
  agents: {
    list: [
      {
        id: "support",
        tools: { profile: "messaging", allow: ["slack"] }
      }
    ]
  }
}
```

<div id="provider-specific-tool-policy">
  ## Política de herramientas específica del proveedor
</div>

Usa `tools.byProvider` para **restringir aún más** las herramientas para proveedores concretos
(o un solo `provider/model`) sin cambiar tus valores predeterminados globales.
Sobrescritura por agente: `agents.list[].tools.byProvider`.

Esto se aplica **después** del perfil de herramientas base y **antes** de las listas de allow/deny,
por lo que solo puede reducir el conjunto de herramientas.
Las claves de proveedor aceptan `provider` (por ejemplo, `google-antigravity`) o
`provider/model` (por ejemplo, `openai/gpt-5.2`).

Ejemplo (mantener el perfil global de código, pero herramientas mínimas para Google Antigravity):

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

Ejemplo (lista de permitidos específica del proveedor/modelo para un endpoint inestable):

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

Ejemplo (sobrescritura específica de agente para un único proveedor):

```json5
{
  agents: {
    list: [
      {
        id: "support",
        tools: {
          byProvider: {
            "google-antigravity": { allow: ["message", "sessions_list"] }
          }
        }
      }
    ]
  }
}
```

<div id="tool-groups-shorthands">
  ## Grupos de herramientas (atajos)
</div>

Las políticas de herramientas (globales, de agente, de sandbox) admiten entradas `group:*` que se expanden en varias herramientas.
Úsalas en `tools.allow` / `tools.deny`.

Grupos disponibles:

* `group:runtime`: `exec`, `bash`, `process`
* `group:fs`: `read`, `write`, `edit`, `apply_patch`
* `group:sessions`: `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status`
* `group:memory`: `memory_search`, `memory_get`
* `group:web`: `web_search`, `web_fetch`
* `group:ui`: `browser`, `canvas`
* `group:automation`: `cron`, `gateway`
* `group:messaging`: `message`
* `group:nodes`: `nodes`
* `group:openclaw`: todas las herramientas integradas de OpenClaw (excluye complementos de proveedor)

Ejemplo (permitir solo herramientas de archivos + navegador):

```json5
{
  tools: {
    allow: ["group:fs", "browser"]
  }
}
```

<div id="plugins-tools">
  ## Complementos + herramientas
</div>

Los complementos pueden registrar **herramientas adicionales** (y comandos de la CLI) más allá del conjunto principal.
Consulta [Complementos](/es/plugin) para instalación y configuración, y [Habilidades](/es/tools/skills) para ver cómo se inyecta en los prompts la guía de uso de herramientas. Algunos complementos incluyen sus propias habilidades junto con herramientas (por ejemplo, el complemento de llamadas de voz).

Herramientas opcionales de complementos:

* [Lobster](/es/tools/lobster): runtime de flujos de trabajo tipados, con aprobaciones reanudables (requiere la Lobster CLI en el host del Gateway).
* [LLM Task](/es/tools/llm-task): paso de LLM solo en JSON para salida estructurada de flujos de trabajo (validación de esquema opcional).

<div id="tool-inventory">
  ## Inventario de herramientas
</div>

<div id="apply_patch">
  ### `apply_patch`
</div>

Aplica parches estructurados a uno o varios archivos. Úsalo para cambios con múltiples bloques.
Experimental: habilítalo mediante `tools.exec.applyPatch.enabled` (solo modelos de OpenAI).

<div id="exec">
  ### `exec`
</div>

Ejecuta comandos de shell en el espacio de trabajo.

Parámetros principales:

* `command` (obligatorio)
* `yieldMs` (pasa a segundo plano automáticamente tras el tiempo de espera, por defecto 10000)
* `background` (envío inmediato a segundo plano)
* `timeout` (segundos; termina el proceso si se supera, por defecto 1800)
* `elevated` (bool; ejecuta en el host si el modo elevado está activado/permitido; solo cambia el comportamiento cuando el agente se ejecuta en sandbox)
* `host` (`sandbox | gateway | node`)
* `security` (`deny | allowlist | full`)
* `ask` (`off | on-miss | always`)
* `node` (id/nombre del nodo para `host=node`)
* ¿Necesitas un TTY real? Establece `pty: true`.

Notas:

* Devuelve `status: "running"` con un `sessionId` cuando se envía a segundo plano.
* Usa `process` para consultar/registrar/escribir/terminar/limpiar sesiones en segundo plano.
* Si `process` no está permitido, `exec` se ejecuta de forma sincrónica e ignora `yieldMs`/`background`.
* `elevated` está controlado por `tools.elevated` más cualquier override `agents.list[].tools.elevated` (ambos deben permitirlo) y es un alias de `host=gateway` + `security=full`.
* `elevated` solo cambia el comportamiento cuando el agente se ejecuta en sandbox (de lo contrario no hace nada).
* `host=node` puede dirigirse a una aplicación complementaria de macOS o a un host de nodo sin interfaz (`openclaw node run`).
* Aprobaciones de gateway/node y lista de permitidos: [Exec approvals](/es/tools/exec-approvals).

<div id="process">
  ### `process`
</div>

Gestiona sesiones de ejecución en segundo plano.

Acciones principales:

* `list`, `poll`, `log`, `write`, `kill`, `clear`, `remove`

Notas:

* `poll` devuelve nueva salida y el estado de salida cuando finaliza.
* `log` admite `offset`/`limit` por líneas (omite `offset` para obtener las últimas N líneas).
* `process` está limitado al ámbito de cada agente; las sesiones de otros agentes no son visibles.

<div id="web_search">
  ### `web_search`
</div>

Busca en la web mediante la Brave Search API.

Parámetros principales:

* `query` (obligatorio)
* `count` (1–10; valor predeterminado tomado de `tools.web.search.maxResults`)

Notas:

* Requiere una clave de API de Brave (recomendado: `openclaw configure --section web`, o establecer `BRAVE_API_KEY`).
* Actívalo mediante `tools.web.search.enabled`.
* Las respuestas se almacenan en caché (por defecto, 15 min).
* Consulta [Herramientas web](/es/tools/web) para la configuración.

<div id="web_fetch">
  ### `web_fetch`
</div>

Obtiene y extrae contenido legible de una URL (HTML → markdown/text).

Parámetros principales:

* `url` (obligatorio)
* `extractMode` (`markdown` | `text`)
* `maxChars` (trunca páginas largas)

Notas:

* Activa esta herramienta mediante `tools.web.fetch.enabled`.
* Las respuestas se almacenan en caché (15 min por defecto).
* Para sitios con mucho JavaScript, usa preferentemente la herramienta del navegador.
* Consulta [Web tools](/es/tools/web) para la configuración.
* Consulta [Firecrawl](/es/tools/firecrawl) para el mecanismo opcional de respaldo antibots.

<div id="browser">
  ### `browser`
</div>

Controla el navegador dedicado gestionado por OpenClaw.

Acciones principales:

* `status`, `start`, `stop`, `tabs`, `open`, `focus`, `close`
* `snapshot` (aria/ai)
* `screenshot` (devuelve un bloque de imagen + `MEDIA:<path>`)
* `act` (acciones de UI: click/type/press/hover/drag/select/fill/resize/wait/evaluate)
* `navigate`, `console`, `pdf`, `upload`, `dialog`

Gestión de perfiles:

* `profiles` — lista todos los perfiles del navegador con su estado
* `create-profile` — crea un nuevo perfil con puerto asignado automáticamente (o `cdpUrl`)
* `delete-profile` — detiene el navegador, elimina los datos de usuario y lo elimina de la config (solo en local)
* `reset-profile` — mata el proceso huérfano en el puerto del perfil (solo en local)

Parámetros comunes:

* `profile` (opcional; por defecto `browser.defaultProfile`)
* `target` (`sandbox` | `host` | `node`)
* `node` (opcional; selecciona un id/nombre de nodo específico)

Notas:

* Requiere `browser.enabled=true` (el valor por defecto es `true`; establécelo en `false` para deshabilitarlo).
* Todas las acciones aceptan el parámetro opcional `profile` para admitir múltiples instancias.
* Cuando se omite `profile`, se usa `browser.defaultProfile` (por defecto &quot;chrome&quot;).
* Nombres de perfil: solo minúsculas alfanuméricas + guiones (máx. 64 caracteres).
* Rango de puertos: 18800-18899 (~100 perfiles máx.).
* Los perfiles remotos solo permiten adjuntarse (sin start/stop/reset).
* Si hay conectado un nodo con capacidad de navegador, la herramienta puede enrutar automáticamente hacia él (a menos que fijes `target`).
* `snapshot` usa por defecto `ai` cuando Playwright está instalado; usa `aria` para el árbol de accesibilidad.
* `snapshot` también admite opciones de role-snapshot (`interactive`, `compact`, `depth`, `selector`) que devuelven referencias como `e12`.
* `act` requiere un `ref` de `snapshot` (numérico `12` de snapshots de AI, o `e12` de snapshots de roles); usa `evaluate` para las necesidades poco frecuentes de selectores CSS.
* Evita `act` → `wait` por defecto; úsalo solo en casos excepcionales (cuando no haya un estado de UI fiable al que esperar).
* `upload` puede pasar opcionalmente un `ref` para hacer clic automáticamente después de armarlo.
* `upload` también admite `inputRef` (referencia aria) o `element` (selector CSS) para establecer `<input type="file">` directamente.

<div id="canvas">
  ### `canvas`
</div>

Controla el Canvas del nodo (present, eval, snapshot, A2UI).

Acciones principales:

* `present`, `hide`, `navigate`, `eval`
* `snapshot` (devuelve un bloque de imagen + `MEDIA:<path>`)
* `a2ui_push`, `a2ui_reset`

Notas:

* Utiliza `gateway node.invoke` internamente.
* Si no se proporciona ningún `node`, la herramienta elige un valor predeterminado (un único nodo conectado o nodo local de macOS).
* A2UI solo es compatible con la v0.8 (sin `createSurface`); la CLI rechaza JSONL de la v0.9 informando errores por línea.
* Prueba rápida: `openclaw nodes canvas a2ui push --node <id> --text "Hello from A2UI"`.

<div id="nodes">
  ### `nodes`
</div>

Descubre y selecciona nodos emparejados; envía notificaciones; captura cámara/pantalla.

Acciones principales:

* `status`, `describe`
* `pending`, `approve`, `reject` (emparejamiento)
* `notify` (macOS `system.notify`)
* `run` (macOS `system.run`)
* `camera_snap`, `camera_clip`, `screen_record`
* `location_get`

Notas:

* Los comandos de cámara/pantalla requieren que la app del nodo esté en primer plano.
* Las imágenes devuelven bloques de imagen + `MEDIA:<path>`.
* Los vídeos devuelven `FILE:<path>` (mp4).
* La ubicación devuelve un payload JSON (lat/lon/accuracy/timestamp).
* Parámetros de `run`: array argv del comando; `cwd` opcional, `env` (`KEY=VAL`), `commandTimeoutMs`, `invokeTimeoutMs`, `needsScreenRecording`.

Ejemplo (`run`):

```json
{
  "action": "run",
  "node": "office-mac",
  "command": ["echo", "Hello"],
  "env": ["FOO=bar"],
  "commandTimeoutMs": 12000,
  "invokeTimeoutMs": 45000,
  "needsScreenRecording": false
}
```

<div id="image">
  ### `image`
</div>

Analiza una imagen con el modelo de imagen configurado.

Parámetros principales:

* `image` (ruta o URL obligatoria)
* `prompt` (opcional; valor predeterminado: &quot;Describe la imagen.&quot;)
* `model` (anulación opcional)
* `maxBytesMb` (límite de tamaño opcional)

Notas:

* Solo está disponible cuando `agents.defaults.imageModel` está configurado (principal o alternativas), o cuando se puede inferir implícitamente un modelo de imagen a partir de tu modelo predeterminado y la autenticación configurada (emparejamiento de mejor esfuerzo).
* Usa el modelo de imagen directamente (independiente del modelo principal de chat).

<div id="message">
  ### `message`
</div>

Envía mensajes y acciones de canal en Discord/Google Chat/Slack/Telegram/WhatsApp/Signal/iMessage/MS Teams.

Acciones principales:

* `send` (texto + contenido multimedia opcional; MS Teams también admite `card` para Adaptive Cards)
* `poll` (encuestas de WhatsApp/Discord/MS Teams)
* `react` / `reactions` / `read` / `edit` / `delete`
* `pin` / `unpin` / `list-pins`
* `permissions`
* `thread-create` / `thread-list` / `thread-reply`
* `search`
* `sticker`
* `member-info` / `role-info`
* `emoji-list` / `emoji-upload` / `sticker-upload`
* `role-add` / `role-remove`
* `channel-info` / `channel-list`
* `voice-status`
* `event-list` / `event-create`
* `timeout` / `kick` / `ban`

Notas:

* `send` enruta WhatsApp a través del Gateway; los demás canales se envían directamente.
* `poll` usa el Gateway para WhatsApp y MS Teams; las encuestas de Discord se envían directamente.
* Cuando una llamada a la herramienta de mensajes está vinculada a una sesión de chat activa, los envíos de mensajes se limitan al destino de esa sesión para evitar fugas entre contextos.

<div id="cron">
  ### `cron`
</div>

Gestiona trabajos cron y activaciones del Gateway.

Acciones principales:

* `status`, `list`
* `add`, `update`, `remove`, `run`, `runs`
* `wake` (pone en cola un evento de sistema + latido opcional inmediato)

Notas:

* `add` espera un objeto de trabajo cron completo (el mismo esquema que el RPC `cron.add`).
* `update` usa `{ id, patch }`.

<div id="gateway">
  ### `gateway`
</div>

Reinicia o aplica actualizaciones al proceso Gateway en ejecución (en el mismo proceso, in-place).

Acciones principales:

* `restart` (autoriza y envía `SIGUSR1` para un reinicio en proceso; reinicio in-place de `openclaw gateway`)
* `config.get` / `config.schema`
* `config.apply` (validar + escribir configuración + reiniciar + despertar)
* `config.patch` (fusionar actualización parcial + reiniciar + despertar)
* `update.run` (ejecutar actualización + reiniciar + despertar)

Notas:

* Usa `delayMs` (valor predeterminado: 2000) para evitar interrumpir una respuesta en curso.
* `restart` está deshabilitado de forma predeterminada; actívalo con `commands.restart: true`.

<div id="sessions_list-sessions_history-sessions_send-sessions_spawn-session_status">
  ### `sessions_list` / `sessions_history` / `sessions_send` / `sessions_spawn` / `session_status`
</div>

Lista las sesiones, inspecciona el historial de conversación o envía mensajes a otra sesión.

Parámetros principales:

* `sessions_list`: `kinds?`, `limit?`, `activeMinutes?`, `messageLimit?` (0 = sin límite)
* `sessions_history`: `sessionKey` (o `sessionId`), `limit?`, `includeTools?`
* `sessions_send`: `sessionKey` (o `sessionId`), `message`, `timeoutSeconds?` (0 = fire-and-forget)
* `sessions_spawn`: `task`, `label?`, `agentId?`, `model?`, `runTimeoutSeconds?`, `cleanup?`
* `session_status`: `sessionKey?` (por defecto la actual; acepta `sessionId`), `model?` (`default` borra la anulación)

Notas:

* `main` es la clave canónica de chat directo; global/unknown se ocultan.
* `messageLimit > 0` obtiene los últimos N mensajes por sesión (se filtran los mensajes de herramientas).
* `sessions_send` espera a la finalización cuando `timeoutSeconds > 0`.
* La entrega/anuncio se produce después de la finalización y es best-effort; `status: "ok"` confirma que la ejecución del agente terminó, no que el anuncio se haya entregado.
* `sessions_spawn` inicia una ejecución de sub-agente y publica una respuesta de anuncio de vuelta en el chat solicitante.
* `sessions_spawn` no bloquea y devuelve `status: "accepted"` inmediatamente.
* `sessions_send` ejecuta un ping‑pong de respuestas (responde `REPLY_SKIP` para detenerlo; turnos máximos vía `session.agentToAgent.maxPingPongTurns`, 0–5).
* Después del ping‑pong, el agente de destino ejecuta un **paso de anuncio**; responde `ANNOUNCE_SKIP` para suprimir el anuncio.

<div id="agents_list">
  ### `agents_list`
</div>

Enumera los ID de agentes a los que la sesión actual puede dirigirse con `sessions_spawn`.

Notas:

* El resultado está restringido por las listas de permitidos por agente (`agents.list[].subagents.allowAgents`).
* Cuando se configura `["*"]`, la herramienta incluye todos los agentes configurados y marca `allowAny: true`.

<div id="parameters-common">
  ## Parámetros (comunes)
</div>

Herramientas basadas en el Gateway (`canvas`, `nodes`, `cron`):

* `gatewayUrl` (valor predeterminado `ws://127.0.0.1:18789`)
* `gatewayToken` (si la autenticación está habilitada)
* `timeoutMs`

Herramienta de navegador:

* `profile` (opcional; valor predeterminado `browser.defaultProfile`)
* `target` (`sandbox` | `host` | `node`)
* `node` (opcional; fijar un ID/nombre de nodo específico)

<div id="recommended-agent-flows">
  ## Flujos de agentes recomendados
</div>

Automatización del navegador:

1. `browser` → `status` / `start`
2. `snapshot` (ai o aria)
3. `act` (click/type/press)
4. `screenshot` si necesitas confirmación visual

Renderizado de canvas:

1. `canvas` → `present`
2. `a2ui_push` (opcional)
3. `snapshot`

Dirección a nodos:

1. `nodes` → `status`
2. `describe` en el nodo elegido
3. `notify` / `run` / `camera_snap` / `screen_record`

<div id="safety">
  ## Seguridad
</div>

* Evita usar directamente `system.run`; usa `nodos` → `run` solo con el consentimiento explícito del usuario.
* Respeta el consentimiento del usuario para la captura de cámara o pantalla.
* Usa `status/describe` para verificar los permisos antes de invocar comandos de medios.

<div id="how-tools-are-presented-to-the-agent">
  ## Cómo se presentan las herramientas al agente
</div>

Las herramientas se exponen en dos canales paralelos:

1. **Texto del prompt del sistema**: una lista legible por humanos + orientación.
2. **Esquema de herramientas**: las definiciones de funciones estructuradas que se envían a la API del modelo.

Esto significa que el agente ve tanto “qué herramientas existen” como “cómo invocarlas”. Si una herramienta
no aparece en el prompt del sistema o en el esquema, el modelo no puede invocarla.