---
summary: "Cómo funciona la sandbox de OpenClaw: modos, ámbitos, acceso al espacio de trabajo e imágenes"
title: Ejecución en sandbox
read_when: "Quieres una explicación detallada sobre la sandbox o necesitas ajustar agents.defaults.sandbox."
status: active
---

<div id="sandboxing">
  # Sandboxing
</div>

OpenClaw puede ejecutar **herramientas dentro de contenedores Docker** para reducir el radio de impacto.
Esto es **opcional** y se controla mediante la configuración (`agents.defaults.sandbox` o
`agents.list[].sandbox`). Si el sandbox está desactivado, las herramientas se ejecutan en el host.
El Gateway permanece en el host; la ejecución de herramientas se realiza en un sandbox aislado
cuando está habilitado.

Esto no es un límite de seguridad perfecto, pero reduce de forma significativa el acceso al sistema de archivos
y a los procesos cuando el modelo hace alguna tontería.

<div id="what-gets-sandboxed">
  ## Qué se ejecuta en sandbox
</div>

* Ejecución de herramientas (`exec`, `read`, `write`, `edit`, `apply_patch`, `process`, etc.).
* Navegador opcional en sandbox (`agents.defaults.sandbox.browser`).
  * De forma predeterminada, el navegador en sandbox se inicia automáticamente (garantiza que CDP sea accesible) cuando la herramienta de navegador lo necesita.
    Configúralo mediante `agents.defaults.sandbox.browser.autoStart` y `agents.defaults.sandbox.browser.autoStartTimeoutMs`.
  * `agents.defaults.sandbox.browser.allowHostControl` permite que las sesiones en sandbox apunten explícitamente al navegador del host.
  * Las listas de permitidos opcionales limitan `target: "custom"`: `allowedControlUrls`, `allowedControlHosts`, `allowedControlPorts`.

No se ejecuta en sandbox:

* El propio proceso del Gateway.
* Cualquier herramienta a la que se le permita explícitamente ejecutarse en el host (por ejemplo, `tools.elevated`).
  * **El `exec` elevado se ejecuta en el host y evita el sandboxing.**
  * Si el sandboxing está desactivado, `tools.elevated` no cambia la ejecución (ya es en el host). Consulta [Elevated Mode](/es/tools/elevated).

<div id="modes">
  ## Modos
</div>

`agents.defaults.sandbox.mode` controla **cuándo** se usa el sandbox:

* `"off"`: sin sandbox.
* `"non-main"`: sandbox solo para sesiones **non-main** (valor predeterminado si deseas chats normales en el host).
* `"all"`: cada sesión se ejecuta en un sandbox.
  Nota: `"non-main"` se basa en `session.mainKey` (predeterminado `"main"`), no en el ID del agente.
  Las sesiones de grupo/canal usan sus propias claves, por lo que cuentan como `"non-main"` y se ejecutarán en un sandbox.

<div id="scope">
  ## Ámbito
</div>

`agents.defaults.sandbox.scope` controla **cuántos contenedores** se crean:

* `"session"` (predeterminado): un contenedor por sesión.
* `"agent"`: un contenedor por agente.
* `"shared"`: un contenedor compartido por todas las sesiones sandbox.

<div id="workspace-access">
  ## Acceso al espacio de trabajo
</div>

`agents.defaults.sandbox.workspaceAccess` controla **qué puede ver el sandbox**:

* `"none"` (predeterminado): las herramientas ven un espacio de trabajo del sandbox en `~/.openclaw/sandboxes`.
* `"ro"`: monta el espacio de trabajo del agente en modo solo lectura en `/agent` (desactiva `write`/`edit`/`apply_patch`).
* `"rw"`: monta el espacio de trabajo del agente con lectura/escritura en `/workspace`.

El contenido multimedia entrante se copia en el espacio de trabajo activo del sandbox (`media/inbound/*`).
Nota sobre habilidades: la herramienta `read` está anclada a la raíz del sandbox. Con `workspaceAccess: "none"`,
OpenClaw replica las habilidades elegibles dentro del espacio de trabajo del sandbox (`.../skills`) para que
puedan leerse. Con `"rw"`, las habilidades del espacio de trabajo se pueden leer desde
`/workspace/skills`.

<div id="custom-bind-mounts">
  ## Bind mounts personalizados
</div>

`agents.defaults.sandbox.docker.binds` monta directorios adicionales del host en el contenedor.
Formato: `host:container:mode` (por ejemplo, `"/home/user/source:/source:rw"`).

Los binds globales y por agente se **combinan** (no se reemplazan). Con `scope: "shared"`, se ignoran los binds por agente.

Ejemplo (origen de solo lectura + socket de Docker):

```json5
{
  agents: {
    defaults: {
      sandbox: {
        docker: {
          binds: [
            "/home/user/source:/source:ro",
            "/var/run/docker.sock:/var/run/docker.sock"
          ]
        }
      }
    },
    list: [
      {
        id: "build",
        sandbox: {
          docker: {
            binds: ["/mnt/cache:/cache:rw"]
          }
        }
      }
    ]
  }
}
```

Notas de seguridad:

* Los binds eluden el sistema de archivos del sandbox: exponen rutas del host con el modo que definas (`:ro` o `:rw`).
* Los montajes sensibles (p. ej., `docker.sock`, secretos, claves SSH) deberían ser `:ro` salvo que sea absolutamente imprescindible lo contrario.
* Combínalo con `workspaceAccess: "ro"` si solo necesitas acceso de lectura al espacio de trabajo; los modos de los binds se mantienen independientes.
* Consulta [Sandbox vs Tool Policy vs Elevated](/es/gateway/sandbox-vs-tool-policy-vs-elevated) para ver cómo interactúan los binds con la política de herramientas y la ejecución con privilegios elevados.

<div id="images-setup">
  ## Imágenes y configuración
</div>

Imagen predeterminada: `openclaw-sandbox:bookworm-slim`

Compílala una sola vez:

```bash
scripts/sandbox-setup.sh
```

Nota: la imagen predeterminada **no** incluye Node. Si una skill necesita Node (u otros runtimes), genera una imagen personalizada o instálalo mediante
`sandbox.docker.setupCommand` (requiere salida de red + raíz con permisos de escritura + usuario root).

Imagen de navegador en sandbox:

```bash
scripts/sandbox-browser-setup.sh
```

De forma predeterminada, los contenedores de sandbox se ejecutan **sin red**.
Puedes anularlo con `agents.defaults.sandbox.docker.network`.

Las instalaciones de Docker y el Gateway en contenedor se encuentran aquí:
[Docker](/es/install/docker)

<div id="setupcommand-one-time-container-setup">
  ## setupCommand (configuración única del contenedor)
</div>

`setupCommand` se ejecuta **una vez** después de que se crea el contenedor de sandbox (no en cada ejecución).
Se ejecuta dentro del contenedor mediante `sh -lc`.

Rutas:

* Global: `agents.defaults.sandbox.docker.setupCommand`
* Por agente: `agents.list[].sandbox.docker.setupCommand`

Errores comunes:

* El valor predeterminado de `docker.network` es `"none"` (sin salida a la red), por lo que las instalaciones de paquetes fallarán.
* `readOnlyRoot: true` impide escrituras; establece `readOnlyRoot: false` o crea una imagen personalizada.
* `user` debe ser root para instalaciones de paquetes (omite `user` o establece `user: "0:0"`).
* La ejecución en el sandbox **no** hereda el `process.env` del host. Usa
  `agents.defaults.sandbox.docker.env` (o una imagen personalizada) para las claves de API de las skills.

<div id="tool-policy-escape-hatches">
  ## Política de herramientas + mecanismos de escape
</div>

Las políticas de permitir/denegar el uso de herramientas siguen aplicándose antes que las reglas de sandbox. Si una herramienta está denegada
a nivel global o por agente, el sandbox no la volverá a habilitar.

`tools.elevated` es un mecanismo de escape explícito que ejecuta `exec` en el host.
Las directivas `/exec` solo se aplican a remitentes autorizados y persisten por sesión; para deshabilitar
`exec` de forma estricta, usa una política de denegación de herramientas (consulta [Sandbox vs Tool Policy vs Elevated](/es/gateway/sandbox-vs-tool-policy-vs-elevated)).

Depuración:

* Usa `openclaw sandbox explain` para inspeccionar el modo efectivo de sandbox, la política de herramientas y las claves de configuración de corrección.
* Consulta [Sandbox vs Tool Policy vs Elevated](/es/gateway/sandbox-vs-tool-policy-vs-elevated) para el modelo mental de “¿por qué está bloqueado esto?”.

Manténlo bien restringido.

<div id="multi-agent-overrides">
  ## Anulaciones en configuración multi‑agente
</div>

Cada agente puede anular sandbox + herramientas:
`agents.list[].sandbox` y `agents.list[].tools` (además de `agents.list[].tools.sandbox.tools` para la política de herramientas del sandbox).
Consulta [Sandbox y herramientas multi‑agente](/es/multi-agent-sandbox-tools) para ver el orden de precedencia.

<div id="minimal-enable-example">
  ## Ejemplo mínimo de activación
</div>

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main",
        scope: "session",
        workspaceAccess: "none"
      }
    }
  }
}
```

<div id="related-docs">
  ## Documentos relacionados
</div>

* [Configuración de la sandbox](/es/gateway/configuration#agentsdefaults-sandbox)
* [Sandbox y herramientas multiagente](/es/multi-agent-sandbox-tools)
* [Seguridad](/es/gateway/security)