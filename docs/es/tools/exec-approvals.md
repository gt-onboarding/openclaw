---
title: Aprobaciones de ejecución
summary: "Aprobaciones de ejecución, listas de permitidos y prompts de escape del sandbox"
read_when:
  - Configurar aprobaciones de ejecución o listas de permitidos
  - Implementar la UX de aprobaciones de ejecución en la app de macOS
  - Revisar prompts de escape del sandbox y sus implicaciones
---

<div id="exec-approvals">
  # Aprobaciones de ejecución
</div>

Las aprobaciones de ejecución son la **barrera de protección en el host de la aplicación complementaria / nodo** para permitir que un agente en sandbox ejecute
comandos en un host real (`gateway` o `node`). Piénsalo como un mecanismo de enclavamiento de seguridad:
los comandos solo se permiten cuando la política + la lista de permitidos + (opcionalmente) la aprobación del usuario coinciden.
Las aprobaciones de ejecución son **además** de la política de herramientas y del control de privilegios elevados (a menos que `elevated` se establezca en `full`, lo que omite las aprobaciones).
La política efectiva es la **más restrictiva** entre `tools.exec.*` y los valores predeterminados de aprobaciones; si se omite un campo de aprobaciones, se usa el valor de `tools.exec`.

Si la UI de la aplicación complementaria **no está disponible**, cualquier solicitud que requiera un prompt se
resuelve mediante el **ask fallback** (predeterminado: denegar).

<div id="where-it-applies">
  ## Dónde se aplica
</div>

Las aprobaciones de ejecución se aplican localmente en el host de ejecución:

* **gateway host** → proceso `openclaw` en la máquina del Gateway
* **node host** → ejecutor de nodo (aplicación complementaria para macOS o host de nodo headless)

macOS, división:

* **node host service** reenvía `system.run` a la **app de macOS** mediante IPC local.
* La **app de macOS** aplica las aprobaciones y ejecuta el comando en contexto de UI.

<div id="settings-and-storage">
  ## Configuración y almacenamiento
</div>

Las aprobaciones se almacenan en un archivo JSON local en el host de ejecución:

`~/.openclaw/exec-approvals.json`

Esquema de ejemplo:

```json
{
  "version": 1,
  "socket": {
    "path": "~/.openclaw/exec-approvals.sock",
    "token": "base64url-token"
  },
  "defaults": {
    "security": "deny",
    "ask": "on-miss",
    "askFallback": "deny",
    "autoAllowSkills": false
  },
  "agents": {
    "main": {
      "security": "allowlist",
      "ask": "on-miss",
      "askFallback": "deny",
      "autoAllowSkills": true,
      "allowlist": [
        {
          "id": "B0C8C0B3-2C2D-4F8A-9A3C-5A4B3C2D1E0F",
          "pattern": "~/Projects/**/bin/rg",
          "lastUsedAt": 1737150000000,
          "lastUsedCommand": "rg -n TODO",
          "lastResolvedPath": "/Users/user/Projects/.../bin/rg"
        }
      ]
    }
  }
}
```

<div id="policy-knobs">
  ## Parámetros de política
</div>

<div id="security-execsecurity">
  ### Seguridad (`exec.security`)
</div>

* **deny**: bloquea todas las solicitudes de exec en el host.
* **allowlist**: permite únicamente los comandos en la lista de permitidos.
* **full**: permite todo (equivalente a &quot;elevated&quot;).

<div id="ask-execask">
  ### Ask (`exec.ask`)
</div>

* **off**: nunca solicitar confirmación.
* **on-miss**: solicitar confirmación solo cuando la lista de permitidos no coincida.
* **always**: solicitar confirmación para cada comando.

<div id="ask-fallback-askfallback">
  ### Ask fallback (`askFallback`)
</div>

Si se requiere un prompt pero no hay ninguna UI disponible, el fallback decide:

* **deny**: bloquear.
* **allowlist**: permitir solo si la lista de permitidos coincide.
* **full**: permitir.

<div id="allowlist-per-agent">
  ## Lista de permitidos (por agente)
</div>

Las listas de permitidos son **por agente**. Si existen varios agentes, cambia qué agente
estás editando en la app de macOS. Los patrones son **coincidencias glob insensibles a mayúsculas/minúsculas**.
Los patrones deben resolverse a **rutas de binarios** (las entradas que solo especifican el basename se ignoran).
Las entradas heredadas de `agents.default` se migran a `agents.main` al cargar.

Ejemplos:

* `~/Projects/**/bin/bird`
* `~/.local/bin/*`
* `/opt/homebrew/bin/rg`

Cada entrada de la lista de permitidos registra:

* **id** UUID estable usado para la identidad en la UI (opcional)
* **last used** marca de tiempo del último uso
* **last used command** último comando usado
* **last resolved path** última ruta resuelta

<div id="auto-allow-skill-clis">
  ## CLI de habilidades con autorización automática
</div>

Cuando **CLI de habilidades con autorización automática** está activado, los ejecutables referenciados por habilidades conocidas se consideran incluidos en la lista de permitidos de los nodos (nodo de macOS o host de nodo sin interfaz). Esto utiliza `skills.bins` a través del RPC del Gateway para obtener la lista de binarios de habilidades. Desactiva esta opción si quieres listas de permitidos estrictas y gestionadas manualmente.

<div id="safe-bins-stdin-only">
  ## Bins seguros (solo stdin)
</div>

`tools.exec.safeBins` define una pequeña lista de binarios **solo stdin** (por ejemplo `jq`)
que pueden ejecutarse en modo de lista de permitidos **sin** entradas explícitas en la lista de permitidos. Los bins seguros rechazan
argumentos posicionales de archivos y tokens con forma de ruta, por lo que solo pueden operar sobre el flujo de entrada.
El encadenamiento en el shell y las redirecciones no se permiten automáticamente en modo de lista de permitidos.

El encadenamiento en el shell (`&&`, `||`, `;`) se permite cuando cada segmento de nivel superior cumple la lista de permitidos
(incluyendo bins seguros o autorización automática de skills). Las redirecciones siguen sin estar permitidas en modo de lista de permitidos.

Bins seguros predeterminados: `jq`, `grep`, `cut`, `sort`, `uniq`, `head`, `tail`, `tr`, `wc`.

<div id="control-ui-editing">
  ## Edición en la Control UI
</div>

Usa la tarjeta **Control UI → Nodes → Exec approvals** para editar los valores
predeterminados, las anulaciones por agente y las listas de permitidos. Elige un
ámbito (Defaults o un agente), ajusta la política, añade/elimina patrones de la
lista de permitidos y luego haz clic en **Save**. La UI muestra metadatos de
**last used** para cada patrón, de modo que puedas mantener la lista ordenada.

El selector de destino elige **Gateway** (aprobaciones locales) o un **nodo**.
Los nodos deben anunciar `system.execApprovals.get/set` (app de macOS o host de
nodo sin interfaz). Si un nodo aún no anuncia exec approvals, edita
directamente su archivo local `~/.openclaw/exec-approvals.json`.

CLI: `openclaw approvals` admite la edición del Gateway o de un nodo (consulta
[Approvals CLI](/es/cli/approvals)).

<div id="approval-flow">
  ## Flujo de aprobación
</div>

Cuando se requiere una confirmación, el Gateway transmite `exec.approval.requested` a los clientes de operador.
La Control UI y la app de macOS la resuelven mediante `exec.approval.resolve`, luego el Gateway reenvía la
solicitud aprobada al host del nodo.

Cuando se requieren aprobaciones, la herramienta exec devuelve inmediatamente un ID de aprobación. Usa ese ID para
correlacionar eventos posteriores del sistema (`Exec finished` / `Exec denied`). Si no llega ninguna decisión antes de que
expire el tiempo de espera, la solicitud se trata como un tiempo de espera de aprobación y se muestra como motivo de denegación.

El cuadro de confirmación incluye:

* command + args
* cwd
* ID de agente
* ruta del ejecutable resuelta
* host + metadatos de políticas

Acciones:

* **Allow once** → ejecutar ahora
* **Always allow** → añadir a la lista de permitidos + ejecutar
* **Deny** → bloquear

<div id="approval-forwarding-to-chat-channels">
  ## Reenvío de aprobaciones a canales de chat
</div>

Puedes reenviar solicitudes de aprobación de ejecución a cualquier canal de chat (incluidos los canales de complemento) y aprobarlas con `/approve`. Esto utiliza la canalización de entrega saliente estándar.

Configuración:

```json5
{
  approvals: {
    exec: {
      enabled: true,
      mode: "session", // "sesión" | "objetivos" | "ambos"
      agentFilter: ["main"],
      sessionFilter: ["discord"], // substring or regex
      targets: [
        { channel: "slack", to: "U12345678" },
        { channel: "telegram", to: "123456789" }
      ]
    }
  }
}
```

Responder en el chat:

```
/approve <id> allow-once
/approve <id> allow-always
/approve <id> deny
```

<div id="macos-ipc-flow">
  ### Flujo de IPC en macOS
</div>

```
Gateway -> Node Service (WS)
                 |  IPC (UDS + token + HMAC + TTL)
                 v
             Mac App (UI + approvals + system.run)
```

Notas sobre seguridad:

* Socket Unix en modo `0600`, token almacenado en `exec-approvals.json`.
* Verificación de par con el mismo UID.
* Challenge/response (nonce + token HMAC + hash de la solicitud) + TTL breve.

<div id="system-events">
  ## Eventos del sistema
</div>

El ciclo de vida de Exec se expone como mensajes del sistema:

* `Exec running` (solo si el comando supera el umbral de notificación de ejecución)
* `Exec finished`
* `Exec denied`

Estos se publican en la sesión del agente después de que el nodo notifica el evento.
Las aprobaciones de Exec en el host del Gateway emiten los mismos eventos de ciclo de vida cuando el comando termina (y opcionalmente cuando se ejecuta durante más tiempo que el umbral).
Las ejecuciones Exec sujetas a aprobación reutilizan el id de aprobación como `runId` en estos mensajes para facilitar la correlación.

<div id="implications">
  ## Implicaciones
</div>

* **full** es potente; es preferible usar listas de permitidos cuando sea posible.
* **ask** te mantiene al tanto y sigue permitiendo aprobaciones rápidas.
* Las listas de permitidos por agente evitan que las aprobaciones de un agente se filtren a otros.
* Las aprobaciones solo se aplican a solicitudes de host exec de **remitentes autorizados**. Los remitentes no autorizados no pueden emitir `/exec`.
* `/exec security=full` es una conveniencia a nivel de sesión para operadores autorizados y omite las aprobaciones por diseño.
  Para bloquear por completo el host exec, establece la seguridad de aprobaciones en `deny` o deniega la herramienta `exec` mediante la política de herramientas.

Relacionado:

* [Herramienta Exec](/es/tools/exec)
* [Modo elevado](/es/tools/elevated)
* [Habilidades](/es/tools/skills)