---
title: Exec
summary: "Uso de la herramienta Exec, modos de stdin y compatibilidad con TTY"
read_when:
  - Al usar o modificar la herramienta Exec
  - Al depurar el comportamiento de stdin o TTY
---

<div id="exec-tool">
  # Herramienta exec
</div>

Ejecuta comandos de shell en el espacio de trabajo. Admite ejecución en primer plano y en segundo plano mediante `process`.
Si `process` no está permitido, `exec` se ejecuta de forma síncrona e ignora `yieldMs`/`background`.
Las sesiones en segundo plano se delimitan por agente; `process` solo ve sesiones del mismo agente.

<div id="parameters">
  ## Parámetros
</div>

* `command` (obligatorio)
* `workdir` (por defecto cwd)
* `env` (sobrescrituras de clave/valor)
* `yieldMs` (por defecto 10000): pasa a segundo plano automáticamente después del retraso
* `background` (bool): pasa a segundo plano inmediatamente
* `timeout` (segundos, por defecto 1800): termina el proceso al expirar
* `pty` (bool): ejecuta en un pseudo-terminal cuando esté disponible (CLIs que solo funcionan en TTY, agentes de programación, UIs de terminal)
* `host` (`sandbox | gateway | node`): dónde ejecutar
* `security` (`deny | allowlist | full`): modo de aplicación para `gateway`/`node`
* `ask` (`off | on-miss | always`): solicitudes de aprobación para `gateway`/`node`
* `node` (string): id/nombre de nodo para `host=node`
* `elevated` (bool): solicita modo elevado (host del gateway); `security=full` solo se fuerza cuando `elevated` se resuelve como `full`

Notas:

* `host` usa `sandbox` por defecto.
* `elevated` se ignora cuando el sandbox está desactivado (exec ya se ejecuta en el host).
* Las aprobaciones de `gateway`/`node` se controlan mediante `~/.openclaw/exec-approvals.json`.
* `node` requiere un nodo emparejado (aplicación compañera o host de nodo sin interfaz/gráfico).
* Si hay varios nodos disponibles, establece `exec.node` o `tools.exec.node` para seleccionar uno.
* En hosts que no son Windows, exec usa `SHELL` cuando está definido; si `SHELL` es `fish`, prefiere `bash` (o `sh`)
  a partir de `PATH` para evitar scripts incompatibles con fish, y luego recurre a `SHELL` si ninguno existe.
* Importante: el sandbox está **desactivado por defecto**. Si el sandbox está desactivado, `host=sandbox` se ejecuta directamente en
  el host del gateway (sin contenedor) y **no requiere aprobaciones**. Para requerir aprobaciones, ejecuta con
  `host=gateway` y configura las aprobaciones de exec (o habilita el sandbox).

<div id="config">
  ## Config
</div>

* `tools.exec.notifyOnExit` (predeterminado: `true`): cuando es `true`, las sesiones de exec en segundo plano ponen en cola un evento del sistema y solicitan un latido al salir.
* `tools.exec.approvalRunningNoticeMs` (predeterminado: 10000): emite un único aviso de “en ejecución” cuando un exec sujeto a aprobación se ejecuta durante más tiempo que este valor (0 lo desactiva).
* `tools.exec.host` (predeterminado: `sandbox`)
* `tools.exec.security` (predeterminado: `deny` para sandbox, `allowlist` para Gateway + nodo cuando no está establecido)
* `tools.exec.ask` (predeterminado: `on-miss`)
* `tools.exec.node` (predeterminado: no establecido)
* `tools.exec.pathPrepend`: lista de directorios que se anteponen a `PATH` para ejecuciones de exec.
* `tools.exec.safeBins`: binarios seguros que solo aceptan entrada por stdin y que pueden ejecutarse sin entradas explícitas en la lista de permitidos.

Ejemplo:

```json5
{
  tools: {
    exec: {
      pathPrepend: ["~/bin", "/opt/oss/bin"]
    }
  }
}
```

<div id="path-handling">
  ### Manejo de `PATH`
</div>

* `host=gateway`: fusiona el `PATH` de tu *login-shell* en el entorno de `exec` (a menos que la llamada
  `exec` ya configure `env.PATH`). El propio demonio sigue ejecutándose con un `PATH` mínimo:
  * macOS: `/opt/homebrew/bin`, `/usr/local/bin`, `/usr/bin`, `/bin`
  * Linux: `/usr/local/bin`, `/usr/bin`, `/bin`
* `host=sandbox`: ejecuta `sh -lc` (*login shell*) dentro del contenedor, por lo que `/etc/profile` puede restablecer `PATH`.
  OpenClaw antepone `env.PATH` después de cargar el perfil mediante una variable de entorno interna
  (sin interpolación de shell); `tools.exec.pathPrepend` también se aplica aquí.
* `host=node`: solo se envían al nodo las modificaciones de entorno que pases. `tools.exec.pathPrepend`
  solo se aplica si la llamada `exec` ya configura `env.PATH`. Los hosts de nodo *headless* aceptan `PATH`
  solo cuando este antepone el `PATH` del host del nodo (sin reemplazo). Los nodos en macOS descartan
  por completo las modificaciones de `PATH`.

Vinculación nodo‑por‑agente (utiliza el índice de la lista de agentes en la configuración):

```bash
openclaw config get agents.list
openclaw config set agents.list[0].tools.exec.node "node-id-or-name"
```

Control UI: la pestaña Nodes incluye un pequeño panel «Exec node binding» con los mismos ajustes.

<div id="session-overrides-exec">
  ## Anulaciones de sesión (`/exec`)
</div>

Usa `/exec` para establecer valores predeterminados **por sesión** de `host`, `security`, `ask` y `node`.
Envía el comando `/exec` sin argumentos para mostrar los valores actuales.

Ejemplo:

```
/exec host=gateway security=allowlist ask=on-miss node=mac-1
```

<div id="authorization-model">
  ## Modelo de autorización
</div>

`/exec` solo se aplica para **remitentes autorizados** (lista de permitidos del canal/emparejamiento más `commands.useAccessGroups`).
Solo actualiza el **estado de la sesión** y no escribe configuración. Para inhabilitar por completo exec, denégalo mediante la
política de herramientas (`tools.deny: ["exec"]` o por agente). Las aprobaciones del host siguen siendo necesarias a menos que configures explícitamente
`security=full` y `ask=off`.

<div id="exec-approvals-companion-app-node-host">
  ## Aprobaciones de Exec (app compañera / host del nodo)
</div>

Los agentes en sandbox pueden requerir aprobación para cada solicitud antes de que `exec` se ejecute en el Gateway o en el host del nodo.
Consulta [Aprobaciones de Exec](/es/tools/exec-approvals) para conocer la política, la lista de permitidos y el flujo de UI.

Cuando se requieren aprobaciones, la herramienta de exec devuelve inmediatamente
`status: "approval-pending"` y un ID de aprobación. Una vez aprobada (o denegada / se agote el tiempo de espera),
el Gateway emite eventos del sistema (`Exec finished` / `Exec denied`). Si el comando sigue
ejecutándose después de `tools.exec.approvalRunningNoticeMs`, se emite un único aviso `Exec running`.

<div id="allowlist-safe-bins">
  ## Lista de permitidos + bins seguros
</div>

La política de lista de permitidos se aplica **solo a rutas binarias resueltas** (sin coincidencias por `basename`). Cuando
`security=allowlist`, los comandos de shell se permiten automáticamente solo si cada segmento del pipeline está
en la lista de permitidos o es un bin seguro. El encadenamiento (`;`, `&&`, `||`) y las redirecciones se rechazan en
modo allowlist.

<div id="examples">
  ## Ejemplos
</div>

En primer plano:

```json
{"tool":"exec","command":"ls -la"}
```

Contexto + encuesta:

```json
{"tool":"exec","command":"npm run build","yieldMs":1000}
{"tool":"process","action":"poll","sessionId":"<id>"}
```

Enviar teclas (al estilo tmux):

```json
{"tool":"process","action":"send-keys","sessionId":"<id>","keys":["Enter"]}
{"tool":"process","action":"send-keys","sessionId":"<id>","keys":["C-c"]}
{"tool":"process","action":"send-keys","sessionId":"<id>","keys":["Up","Up","Enter"]}
```

Enviar (solo send CR):

```json
{"tool":"process","action":"submit","sessionId":"<id>"}
```

Pegar (entre corchetes por defecto):

```json
{"tool":"process","action":"paste","sessionId":"<id>","text":"line1\nline2\n"}
```

<div id="apply_patch-experimental">
  ## apply_patch (experimental)
</div>

`apply_patch` es una subherramienta de `exec` para realizar ediciones estructuradas en varios archivos.
Habilítala explícitamente:

```json5
{
  tools: {
    exec: {
      applyPatch: { enabled: true, allowModels: ["gpt-5.2"] }
    }
  }
}
```

Notas:

* Solo está disponible para modelos de OpenAI/OpenAI Codex.
* La política de herramientas sigue vigente; `allow: ["exec"]` permite implícitamente `apply_patch`.
* La configuración se encuentra en `tools.exec.applyPatch`.
