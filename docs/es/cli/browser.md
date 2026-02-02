---
title: Navegador
summary: "Referencia de la CLI para `openclaw browser` (perfiles, pestañas, acciones, relé de extensión)"
read_when:
  - Usas `openclaw browser` y quieres ejemplos de tareas comunes
  - Quieres controlar un navegador que se ejecuta en otra máquina mediante un nodo host
  - Quieres usar el relé de la extensión de Chrome (conectar/desconectar mediante el botón de la barra de herramientas)
---

<div id="openclaw-browser">
  # `openclaw browser`
</div>

Administra el servidor de control del navegador de OpenClaw y ejecuta acciones en el navegador (pestañas, instantáneas, capturas de pantalla, navegación, clics, introducción de texto).

Relacionado:

* Herramienta de navegador + API: [Herramienta de navegador](/es/tools/browser)
* Relay mediante extensión de Chrome: [Extensión de Chrome](/es/tools/chrome-extension)

<div id="common-flags">
  ## Opciones comunes
</div>

* `--url <gatewayWsUrl>`: URL WS del Gateway (por defecto según la configuración).
* `--token <token>`: token del Gateway (si se requiere).
* `--timeout <ms>`: tiempo de espera de la solicitud (ms).
* `--browser-profile <name>`: elige un perfil de navegador (por defecto según la configuración).
* `--json`: salida legible por máquina (cuando sea compatible).

<div id="quick-start-local">
  ## Inicio rápido (local)
</div>

```bash
openclaw browser --browser-profile chrome tabs
openclaw browser --browser-profile openclaw start
openclaw browser --browser-profile openclaw open https://example.com
openclaw browser --browser-profile openclaw snapshot
```

<div id="profiles">
  ## Perfiles
</div>

Los perfiles son configuraciones de enrutamiento del navegador identificadas por nombre. En la práctica:

* `openclaw`: inicia o se conecta a una instancia dedicada de Chrome gestionada por OpenClaw (directorio de datos de usuario aislado).
* `chrome`: controla tus pestañas de Chrome existentes mediante el relé de la extensión de Chrome.

```bash
openclaw browser profiles
openclaw browser create-profile --name work --color "#FF5A36"
openclaw browser delete-profile --name work
```

Utiliza un perfil específico:

```bash
openclaw browser --browser-profile work tabs
```

<div id="tabs">
  ## Pestañas
</div>

```bash
openclaw browser tabs
openclaw browser open https://docs.openclaw.ai
openclaw browser focus <targetId>
openclaw browser close <targetId>
```

<div id="snapshot-screenshot-actions">
  ## Instantánea / captura de pantalla / acciones
</div>

Instantánea:

```bash
openclaw browser snapshot
```

Captura de pantalla:

```bash
openclaw browser screenshot
```

Navegar/hacer clic/escribir (automatización de la UI basada en referencias):

```bash
openclaw browser navigate https://example.com
openclaw browser click <ref>
openclaw browser type <ref> "hello"
```

<div id="chrome-extension-relay-attach-via-toolbar-button">
  ## Relé de extensión de Chrome (adjuntar mediante botón de la barra de herramientas)
</div>

Este modo permite que el agente controle una pestaña de Chrome existente que vinculas manualmente (no se asocia de forma automática).

Instala la extensión desempaquetada en una ruta estable:

```bash
openclaw browser extension install
openclaw browser extension path
```

Luego, en Chrome → `chrome://extensions` → habilita “Modo de desarrollador” → “Cargar descomprimida” → selecciona la carpeta indicada.

Guía completa: [Extensión de Chrome](/es/tools/chrome-extension)

<div id="remote-browser-control-node-host-proxy">
  ## Control remoto del navegador (proxy del nodo anfitrión)
</div>

Si el Gateway se ejecuta en una máquina distinta a la del navegador, ejecuta un **nodo anfitrión** en la máquina que tenga Chrome/Brave/Edge/Chromium. El Gateway actuará como proxy y reenviará las acciones del navegador a ese nodo (no se requiere un servidor de control de navegador independiente).

Usa `gateway.nodes.browser.mode` para controlar el enrutamiento automático y `gateway.nodes.browser.node` para anclar un nodo específico si hay varios conectados.

Seguridad + configuración remota: [Herramienta de navegador](/es/tools/browser), [Acceso remoto](/es/gateway/remote), [Tailscale](/es/gateway/tailscale), [Seguridad](/es/gateway/security)