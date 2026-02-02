---
title: Extensión de Chrome
summary: "Extensión de Chrome: permite que OpenClaw controle tu pestaña actual de Chrome"
read_when:
  - Quieres que el agente controle una pestaña de Chrome existente (botón de la barra de herramientas)
  - Necesitas Gateway remoto + automatización local del navegador mediante Tailscale
  - Quieres comprender las implicaciones de seguridad de la toma de control del navegador
---

<div id="chrome-extension-browser-relay">
  # Extensión de Chrome (relay del navegador)
</div>

La extensión de Chrome de OpenClaw permite que el agente controle tus **pestañas de Chrome ya abiertas** (tu ventana normal de Chrome) en lugar de iniciar por separado un perfil de Chrome administrado por openclaw.

La conexión/desconexión se realiza mediante un **único botón en la barra de herramientas de Chrome**.

<div id="what-it-is-concept">
  ## Qué es (concepto)
</div>

Hay tres partes:

* **Servicio de control del navegador** (Gateway o nodo): la API que el agente o la herramienta invoca (a través del Gateway)
* **Servidor de retransmisión local** (CDP de loopback): actúa como puente entre el servidor de control y la extensión (`http://127.0.0.1:18792` por defecto)
* **Extensión Chrome MV3**: se conecta a la pestaña activa usando `chrome.debugger` y canaliza los mensajes CDP hacia el relay

Luego, OpenClaw controla la pestaña conectada a través de la interfaz normal de la herramienta `browser` (seleccionando el perfil adecuado).

<div id="install-load-unpacked">
  ## Instalar / cargar (desempaquetada)
</div>

1. Instala la extensión en una ruta local fija:

```bash
openclaw browser extension install
```

2. Muestra la ruta del directorio de la extensión instalada:

```bash
openclaw browser extension path
```

3. Chrome → `chrome://extensions`

* Habilita el “modo de desarrollador”
* “Cargar sin empaquetar” → selecciona el directorio mostrado arriba

4. Fija la extensión.

<div id="updates-no-build-step">
  ## Actualizaciones (sin paso de build)
</div>

La extensión se distribuye dentro del release de OpenClaw (paquete npm) como archivos estáticos. No hay un paso de “build” independiente.

Después de actualizar OpenClaw:

* Vuelve a ejecutar `openclaw browser extension install` para actualizar los archivos instalados en tu directorio de estado de OpenClaw.
* Chrome → `chrome://extensions` → haz clic en “Recargar” en la extensión.

<div id="use-it-no-extra-config">
  ## Úsalo (sin configuración adicional)
</div>

OpenClaw se distribuye con un perfil de navegador integrado llamado `chrome` que apunta al relay de la extensión en el puerto predeterminado.

Úsalo:

* CLI: `openclaw browser --browser-profile chrome tabs`
* Herramienta de agente: `browser` con `profile="chrome"`

Si quieres un nombre diferente o un puerto de relay distinto, crea tu propio perfil:

```bash
openclaw browser create-profile \
  --name my-chrome \
  --driver extension \
  --cdp-url http://127.0.0.1:18792 \
  --color "#00AA00"
```

<div id="attach-detach-toolbar-button">
  ## Vincular / desvincular (botón de la barra de herramientas)
</div>

* Abre la pestaña que quieres que OpenClaw controle.
* Haz clic en el icono de la extensión.
  * La insignia muestra `ON` cuando está vinculada.
* Haz clic de nuevo para desvincularla.

<div id="which-tab-does-it-control">
  ## ¿Qué pestaña controla?
</div>

* **No** controla automáticamente “la pestaña que estás viendo”.
* Controla **solo la(s) pestaña(s) que hayas vinculado explícitamente** haciendo clic en el botón de la barra de herramientas.
* Para cambiar: abre la otra pestaña y haz clic en el icono de la extensión allí.

<div id="badge-common-errors">
  ## Insignia + errores comunes
</div>

* `ON`: conectado; OpenClaw puede controlar esa pestaña.
* `…`: conectando al relay local.
* `!`: relay no accesible (lo más común: el servidor de relay del navegador no se está ejecutando en esta máquina).

Si ves `!`:

* Asegúrate de que el Gateway se esté ejecutando localmente (configuración predeterminada), o ejecuta un nodo host en esta máquina si el Gateway se ejecuta en otro lugar.
* Abre la página de Opciones de la extensión; muestra si el relay es accesible.

<div id="remote-gateway-use-a-node-host">
  ## Gateway remoto (usa un host de nodo)
</div>

<div id="local-gateway-same-machine-as-chrome-usually-no-extra-steps">
  ### Gateway local (misma máquina que Chrome) — normalmente **sin pasos extra**
</div>

Si el Gateway se ejecuta en la misma máquina que Chrome, inicia el servicio de control del navegador sobre la interfaz de loopback y arranca automáticamente el servidor de relay. La extensión se comunica con el relay local; las llamadas de la CLI o de herramientas van al Gateway.

<div id="remote-gateway-gateway-runs-elsewhere-run-a-node-host">
  ### Gateway remoto (Gateway se ejecuta en otro lugar) — **ejecutar un host de nodo**
</div>

Si tu Gateway se ejecuta en otra máquina, inicia un host de nodo en la máquina donde se ejecuta Chrome.
El Gateway actuará como proxy de las acciones del navegador hacia ese nodo; la extensión y el relay permanecerán locales en la máquina del navegador.

Si hay varios nodos conectados, fija uno con `gateway.nodes.browser.node` o configura `gateway.nodes.browser.mode`.

<div id="sandboxing-tool-containers">
  ## Sandboxing (contenedores de herramientas)
</div>

Si la sesión de tu agente está en sandbox (`agents.defaults.sandbox.mode != "off"`), la herramienta `browser` puede estar restringida:

* De forma predeterminada, las sesiones en sandbox suelen dirigirse al **navegador de sandbox** (`target="sandbox"`), no a tu Chrome del host.
* La toma de control mediante el relay de la extensión de Chrome requiere controlar el servidor de control del navegador del **host**.

Opciones:

* Lo más sencillo: usa la extensión desde una sesión/agente **sin sandbox**.
* O permite también el control del navegador del host para sesiones en sandbox:

```json5
{
  agents: {
    defaults: {
      sandbox: {
        browser: {
          allowHostControl: true
        }
      }
    }
  }
}
```

Luego asegúrate de que la herramienta no esté bloqueada por la política de herramientas y, si es necesario, llama a `browser` con `target="host"`.

Depuración: `openclaw sandbox explain`

<div id="remote-access-tips">
  ## Consejos para acceso remoto
</div>

* Mantén el Gateway y el host del nodo en la misma tailnet; evita exponer los puertos de relay a la LAN o a la Internet pública.
* Empareja nodos de forma explícita; desactiva el enrutamiento del proxy del navegador si no quieres habilitar el control remoto (`gateway.nodes.browser.mode="off"`).

<div id="how-extension-path-works">
  ## Cómo funciona “extension path”
</div>

`openclaw browser extension path` muestra la ruta del directorio en disco donde está instalada la extensión y que contiene sus archivos.

La CLI deliberadamente **no** muestra una ruta dentro de `node_modules`. Ejecuta siempre primero `openclaw browser extension install` para copiar la extensión a una ubicación estable dentro de tu directorio de estado de OpenClaw.

Si mueves o eliminas ese directorio de instalación, Chrome marcará la extensión como dañada hasta que la vuelvas a cargar desde una ruta válida.

<div id="security-implications-read-this">
  ## Implicaciones de seguridad (léelo)
</div>

Esto es potente y arriesgado. Trátalo como si le dieras al modelo “control directo de tu navegador”.

* La extensión usa la API de depuración de Chrome (`chrome.debugger`). Cuando está conectada, el modelo puede:
  * hacer clic/escribir/navegar en esa pestaña
  * leer el contenido de la página
  * acceder a todo lo que pueda acceder la sesión iniciada en esa pestaña
* **Esto no está aislado** como el perfil dedicado gestionado por openclaw.
  * Si lo conectas a tu perfil/pestaña de uso diario, estás otorgando acceso a ese estado de cuenta.

Recomendaciones:

* Prefiere un perfil dedicado de Chrome (separado de tu navegación personal) para usar el relay de la extensión.
* Mantén el Gateway y cualquier host de nodos solo accesibles vía Tailnet; confía en la autenticación del Gateway + el emparejamiento del nodo.
* Evita exponer puertos de relay en la LAN (`0.0.0.0`) y evita Funnel (público).

Relacionado:

* Descripción general de la herramienta de navegador: [Browser](/es/tools/browser)
* Auditoría de seguridad: [Security](/es/gateway/security)
* Configuración de Tailscale: [Tailscale](/es/gateway/tailscale)