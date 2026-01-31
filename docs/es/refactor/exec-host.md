---
title: Host de ejecución
summary: "Plan de refactorización: enrutamiento del host de ejecución, aprobaciones de nodos y runner headless"
read_when:
  - Diseñar el enrutamiento del host de ejecución o las aprobaciones de ejecución
  - Implementar el runner del nodo + IPC de la UI
  - Añadir modos de seguridad del host de ejecución y comandos slash
---

<div id="exec-host-refactor-plan">
  # Plan de refactorización del host de ejecución
</div>

<div id="goals">
  ## Objetivos
</div>

* Añadir `exec.host` + `exec.security` para enrutar la ejecución entre **sandbox**, **Gateway** y **nodo**.
* Mantener valores predeterminados **seguros**: sin ejecución entre hosts a menos que se habilite explícitamente.
* Dividir la ejecución en un **servicio de ejecución headless** con UI opcional (app para macOS) vía IPC local.
* Proporcionar política **por agente**, lista de permitidos, modo de confirmación y vinculación de nodo.
* Compatibilidad con **modos de confirmación** que funcionen *con* o *sin* listas de permitidos.
* Multiplataforma: socket Unix + autenticación por token (paridad macOS/Linux/Windows).

<div id="non-goals">
  ## Fuera de alcance
</div>

* Sin migración de la lista de permitidos heredada ni compatibilidad con esquemas heredados.
* Sin PTY/streaming para la ejecución en nodo (solo salida agregada).
* Sin nueva capa de red más allá de Bridge + Gateway existentes.

<div id="decisions-locked">
  ## Decisiones (bloqueadas)
</div>

* **Claves de configuración:** `exec.host` + `exec.security` (se permite override por agente).
* **Elevación:** mantener `/elevated` como alias para acceso completo al Gateway.
* **Valor predeterminado de Ask:** `on-miss`.
* **Almacenamiento de aprobaciones:** `~/.openclaw/exec-approvals.json` (JSON, sin migración heredada).
* **Runner:** servicio de sistema headless; la app de UI aloja un socket Unix para aprobaciones.
* **Identidad del nodo:** usar el `nodeId` existente.
* **Autenticación del socket:** socket Unix + token (multiplataforma); dividir más adelante si es necesario.
* **Estado del host del nodo:** `~/.openclaw/node.json` (id del nodo + token de emparejamiento).
* **Host de ejecución en macOS:** ejecutar `system.run` dentro de la app de macOS; el servicio host del nodo reenvía solicitudes mediante IPC local.
* **Sin helper XPC:** ceñirse a socket Unix + token + comprobaciones de peer.

<div id="key-concepts">
  ## Conceptos clave
</div>

<div id="host">
  ### Host
</div>

* `sandbox`: exec de Docker (comportamiento actual).
* `gateway`: exec en el host del Gateway.
* `node`: exec en el runner del nodo mediante Bridge (`system.run`).

<div id="security-mode">
  ### Modo de seguridad
</div>

* `deny`: bloquear siempre.
* `allowlist`: permitir solo coincidencias (lista de permitidos).
* `full`: permitir todo (equivalente a `elevated`).

<div id="ask-mode">
  ### Modo de consulta
</div>

* `off`: nunca preguntar.
* `on-miss`: preguntar solo cuando no haya coincidencia en la lista de permitidos.
* `always`: preguntar cada vez.

La opción de consulta es **independiente** de la lista de permitidos; la lista de permitidos se puede usar con `always` o `on-miss`.

<div id="policy-resolution-per-exec">
  ### Resolución de políticas (por ejecución)
</div>

1. Resolver `exec.host` (parámetro de herramienta → anulación del agente → valor predeterminado global).
2. Resolver `exec.security` y `exec.ask` (con la misma precedencia).
3. Si el host es `sandbox`, proceder con la ejecución en la sandbox local.
4. Si el host es `gateway` o `node`, aplicar la política de seguridad + ask en ese host.

<div id="default-safety">
  ## Seguridad predeterminada
</div>

* Valor por defecto `exec.host = sandbox`.
* Valor por defecto `exec.security = deny` para `Gateway` y `nodo`.
* Valor por defecto `exec.ask = on-miss` (solo relevante si la configuración de seguridad lo permite).
* Si no se establece ninguna vinculación de nodo, **el agente puede dirigirse a cualquier nodo**, pero solo si la política lo permite.

<div id="config-surface">
  ## Superficie de configuración
</div>

<div id="tool-parameters">
  ### Parámetros de la herramienta
</div>

* `exec.host` (opcional): `sandbox | gateway | node`.
* `exec.security` (opcional): `deny | allowlist | full`.
* `exec.ask` (opcional): `off | on-miss | always`.
* `exec.node` (opcional): ID/nombre del nodo que se usará cuando `host=node`.

<div id="config-keys-global">
  ### Claves de configuración (globales)
</div>

* `tools.exec.host`
* `tools.exec.security`
* `tools.exec.ask`
* `tools.exec.node` (asociación de nodo predeterminada)

<div id="config-keys-per-agent">
  ### Claves de configuración (por agente)
</div>

* `agents.list[].tools.exec.host`
* `agents.list[].tools.exec.security`
* `agents.list[].tools.exec.ask`
* `agents.list[].tools.exec.node`

<div id="alias">
  ### Alias
</div>

* `/elevated on` = establece `tools.exec.host=gateway`, `tools.exec.security=full` para la sesión del agente.
* `/elevated off` = restaura la configuración de ejecución previa para la sesión del agente.

<div id="approvals-store-json">
  ## Almacén de aprobaciones (JSON)
</div>

Ruta: `~/.openclaw/exec-approvals.json`

Propósito:

* Política local + lista de permitidos para el **host de ejecución** (Gateway o runner de nodo).
* Solicitar confirmación como mecanismo de respaldo cuando no haya UI disponible.
* Credenciales de IPC para clientes de UI.

Esquema propuesto (v1):

```json
{
  "version": 1,
  "socket": {
    "path": "~/.openclaw/exec-approvals.sock",
    "token": "base64-opaque-token"
  },
  "defaults": {
    "security": "deny",
    "ask": "on-miss",
    "askFallback": "deny"
  },
  "agents": {
    "agent-id-1": {
      "security": "allowlist",
      "ask": "on-miss",
      "allowlist": [
        {
          "pattern": "~/Projects/**/bin/rg",
          "lastUsedAt": 0,
          "lastUsedCommand": "rg -n TODO",
          "lastResolvedPath": "/Users/user/Projects/.../bin/rg"
        }
      ]
    }
  }
}
```

Notas:

* No se admiten formatos heredados de lista de permitidos.
* `askFallback` solo se aplica cuando `ask` es obligatorio y no se puede acceder a ninguna UI.
* Permisos de archivo: `0600`.

<div id="runner-service-headless">
  ## Servicio de ejecución en modo headless
</div>

<div id="role">
  ### Rol
</div>

* Hacer cumplir `exec.security` + `exec.ask` localmente.
* Ejecutar comandos del sistema y devolver la salida.
* Emitir eventos de Bridge para el ciclo de vida de exec (opcional pero recomendable).

<div id="service-lifecycle">
  ### Ciclo de vida del servicio
</div>

* Launchd/daemon en macOS; servicio del sistema en Linux/Windows.
* El JSON de aprobaciones es local al host de ejecución.
* La UI expone un socket Unix local; los runners se conectan bajo demanda.

<div id="ui-integration-macos-app">
  ## Integración de la UI (aplicación de macOS)
</div>

<div id="ipc">
  ### IPC
</div>

* Socket Unix en `~/.openclaw/exec-approvals.sock` (0600).
* Token almacenado en `exec-approvals.json` (0600).
* Verificaciones del peer: solo mismo UID.
* Desafío/respuesta: nonce + HMAC(token, request-hash) para prevenir ataques de repetición (replay).
* TTL corto (p. ej., 10s) + carga útil máxima + limitación de velocidad.

<div id="ask-flow-macos-app-exec-host">
  ### Flujo de solicitud (host de ejecución de la app en macOS)
</div>

1. El servicio de nodo recibe `system.run` desde el Gateway.
2. El servicio de nodo se conecta al socket local y envía la solicitud de prompt/exec.
3. La app valida el peer + token + HMAC + TTL y, si es necesario, muestra un cuadro de diálogo.
4. La app ejecuta el comando en el contexto de la UI y devuelve la salida.
5. El servicio de nodo devuelve la salida al Gateway.

Si no hay UI disponible:

* Aplica `askFallback` (`deny|allowlist|full`).

<div id="diagram-sci">
  ### Diagrama (SCI)
</div>

```
Agent -> Gateway -> Bridge -> Node Service (TS)
                         |  IPC (UDS + token + HMAC + TTL)
                         v
                     Mac App (UI + TCC + system.run)
```

<div id="node-identity-binding">
  ## Identidad y vinculación del nodo
</div>

* Usa el `nodeId` existente del emparejamiento de Bridge.
* Modelo de vinculación:
  * `tools.exec.node` restringe el agente a un nodo específico.
  * Si no se establece, el agente puede seleccionar cualquier nodo (la política sigue aplicando los valores predeterminados).
* Resolución para la selección de nodos:
  * Coincidencia exacta de `nodeId`
  * `displayName` (normalizado)
  * `remoteIp`
  * Prefijo de `nodeId` (&gt;= 6 caracteres)

<div id="eventing">
  ## Eventos
</div>

<div id="who-sees-events">
  ### Quién ve los eventos
</div>

* Los eventos del sistema son **por sesión** y se muestran al agente en el siguiente mensaje de entrada (prompt).
* Se almacenan en la cola en memoria del Gateway (`enqueueSystemEvent`).

<div id="event-text">
  ### Texto de eventos
</div>

* `Exec started (node=<id>, id=<runId>)`
* `Exec finished (node=<id>, id=<runId>, code=<code>)` + segmento final opcional de la salida
* `Exec denied (node=<id>, id=<runId>, <reason>)`

<div id="transport">
  ### Transporte
</div>

Opción A (recomendada):

* Runner envía al Bridge frames `event` `exec.started` / `exec.finished`.
* Gateway `handleBridgeEvent` los mapea a `enqueueSystemEvent`.

Opción B:

* La herramienta `exec` de Gateway gestiona directamente el ciclo de vida (solo sincrónico).

<div id="exec-flows">
  ## Flujos de Exec
</div>

<div id="sandbox-host">
  ### Host de sandbox
</div>

* Comportamiento `exec` existente (Docker o host cuando está sin sandbox).
* PTY solo es compatible en modo sin sandbox.

<div id="gateway-host">
  ### Host de Gateway
</div>

* El proceso de Gateway se ejecuta en su propia máquina.
* Aplica el `exec-approvals.json` local (seguridad/preguntar/lista de permitidos).

<div id="node-host">
  ### Host de nodo
</div>

* Gateway llama a `node.invoke` con `system.run`.
* Runner hace cumplir las aprobaciones locales.
* Runner devuelve stdout/stderr agregados.
* Eventos Bridge opcionales para inicio/finalización/denegación.

<div id="output-caps">
  ## Límites de salida
</div>

* Limita stdout+stderr combinados a **200k**; conserva los **últimos 20k** para eventos.
* Trunca con un sufijo claro (por ejemplo, `"… (truncated)"`).

<div id="slash-commands">
  ## Comandos slash
</div>

* `/exec host=<sandbox|gateway|node> security=<deny|allowlist|full> ask=<off|on-miss|always> node=<id>`
* Anulaciones por agente y por sesión; no son persistentes a menos que se guarden en la configuración.
* `/elevated on|off|ask|full` sigue siendo un atajo para `host=gateway security=full` (con `full` omitiendo aprobaciones).

<div id="cross-platform-story">
  ## Compatibilidad multiplataforma
</div>

* El servicio runner es el destino de ejecución portátil.
* La UI es opcional; si falta, se aplica `askFallback`.
* Windows y Linux utilizan el mismo protocolo de aprobaciones JSON + socket.

<div id="implementation-phases">
  ## Fases de implementación
</div>

<div id="phase-1-config-exec-routing">
  ### Fase 1: configuración + enrutamiento de exec
</div>

* Agregar el esquema de configuración para `exec.host`, `exec.security`, `exec.ask`, `exec.node`.
* Actualizar la lógica interna de las herramientas para respetar `exec.host`.
* Agregar el comando slash `/exec` y mantener el alias `/elevated`.

<div id="phase-2-approvals-store-gateway-enforcement">
  ### Fase 2: almacén de aprobaciones + aplicación en el Gateway
</div>

* Implementar un lector/escritor de `exec-approvals.json`.
* Hacer cumplir la lista de permitidos y los modos `ask` para el host `gateway`.
* Añadir límites de salida.

<div id="phase-3-node-runner-enforcement">
  ### Fase 3: aplicación de controles en el node runner
</div>

* Actualizar el node runner para aplicar la lista de permitidos + ask.
* Añadir un puente de prompt por socket Unix a la UI de la app de macOS.
* Conectar `askFallback`.

<div id="phase-4-events">
  ### Fase 4: eventos
</div>

* Agregar eventos de Bridge de nodo → Gateway para el ciclo de vida de la ejecución.
* Vincular con `enqueueSystemEvent` para los prompts del agente.

<div id="phase-5-ui-polish">
  ### Fase 5: refinado de la UI
</div>

* Aplicación para Mac: editor de la lista de permitidos, conmutador por agente, UI de política de consultas.
* Controles de vinculación de nodos (opcional).

<div id="testing-plan">
  ## Plan de pruebas
</div>

* Pruebas unitarias: coincidencia de la lista de permitidos (glob + sin distinción de mayúsculas/minúsculas).
* Pruebas unitarias: precedencia en la resolución de políticas (parámetro de herramienta → anulación en el agente → global).
* Pruebas de integración: flujos de denegar/permitir/preguntar del ejecutor de nodo.
* Pruebas de eventos de Bridge: enrutamiento de eventos de nodo → eventos de sistema.

<div id="open-risks">
  ## Riesgos conocidos
</div>

* Indisponibilidad de la UI: garantiza que se respete `askFallback`.
* Comandos de larga duración: usa tiempos de espera y límites de salida.
* Ambigüedad multinodo: error salvo que exista vinculación a un nodo o un parámetro de nodo explícito.

<div id="related-docs">
  ## Documentos relacionados
</div>

* [Herramienta de Exec](/es/tools/exec)
* [Aprobaciones de Exec](/es/tools/exec-approvals)
* [Nodos](/es/nodes)
* [Modo elevado](/es/tools/elevated)