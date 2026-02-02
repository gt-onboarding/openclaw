---
title: Navegador
summary: "Servicio integrado de control del navegador y comandos de acción"
read_when:
  - Añadiendo automatización del navegador controlada por agentes
  - Depurando por qué openclaw está interfiriendo con tu propio Chrome
  - Implementando la configuración y el ciclo de vida del navegador en la aplicación de macOS
---

<div id="browser-openclaw-managed">
  # Navegador (administrado por openclaw)
</div>

OpenClaw puede ejecutar un **perfil dedicado de Chrome/Brave/Edge/Chromium** controlado por el agente.
Está aislado de tu navegador personal y se gestiona mediante un pequeño
servicio de control local dentro del Gateway (solo loopback).

Vista para principiantes:

* Piénsalo como un **navegador separado, solo para el agente**.
* El perfil `openclaw` **no** toca tu perfil de navegador personal.
* El agente puede **abrir pestañas, leer páginas, hacer clic y escribir** en un entorno seguro.
* El perfil predeterminado `chrome` usa el **navegador Chromium predeterminado del sistema** mediante el
  relé de la extensión; cambia a `openclaw` para usar el navegador aislado y administrado.

<div id="what-you-get">
  ## Lo que obtienes
</div>

* Un perfil de navegador independiente llamado **openclaw** (con acento naranja de forma predeterminada).
* Control determinista de pestañas (listar/abrir/enfocar/cerrar).
* Acciones del agente (clic/escribir/arrastrar/seleccionar), instantáneas, capturas de pantalla, archivos PDF.
* Compatibilidad opcional con múltiples perfiles (`openclaw`, `work`, `remote`, ...).

Este navegador **no** es tu navegador de uso diario. Es un entorno seguro y aislado para la automatización y verificación por parte de agentes.

<div id="quick-start">
  ## Inicio rápido
</div>

```bash
openclaw browser --browser-profile openclaw status
openclaw browser --browser-profile openclaw start
openclaw browser --browser-profile openclaw open https://example.com
openclaw browser --browser-profile openclaw snapshot
```

Si ves «Browser disabled», habilítalo en la configuración (consulta más abajo) y reinicia el
Gateway.

<div id="profiles-openclaw-vs-chrome">
  ## Perfiles: `openclaw` vs `chrome`
</div>

* `openclaw`: navegador gestionado y aislado (no requiere extensión).
* `chrome`: relay mediante extensión a tu **navegador predeterminado del sistema** (requiere que la extensión de OpenClaw esté asociada a una pestaña).

Configura `browser.defaultProfile: "openclaw"` si quieres el modo gestionado como predeterminado.

<div id="configuration">
  ## Configuración
</div>

Los ajustes del navegador se guardan en `~/.openclaw/openclaw.json`.

```json5
{
  browser: {
    enabled: true,                    // default: true
    // cdpUrl: "http://127.0.0.1:18792", // legacy single-profile override
    remoteCdpTimeoutMs: 1500,         // remote CDP HTTP timeout (ms)
    remoteCdpHandshakeTimeoutMs: 3000, // tiempo de espera de handshake de WebSocket de CDP remoto (ms)
    defaultProfile: "chrome",
    color: "#FF4500",
    headless: false,
    noSandbox: false,
    attachOnly: false,
    executablePath: "/Applications/Brave Browser.app/Contents/MacOS/Brave Browser",
    profiles: {
      openclaw: { cdpPort: 18800, color: "#FF4500" },
      work: { cdpPort: 18801, color: "#0066CC" },
      remote: { cdpUrl: "http://10.0.0.42:9222", color: "#00AA00" }
    }
  }
}
```

Notas:

* El servicio de control del navegador se vincula a loopback en un puerto derivado de `gateway.port`
  (predeterminado: `18791`, que es gateway + 2). El relay usa el siguiente puerto (`18792`).
* Si anulas el puerto del Gateway (`gateway.port` o `OPENCLAW_GATEWAY_PORT`),
  los puertos derivados del navegador se ajustan para permanecer en la misma “familia”.
* `cdpUrl` usa por defecto el puerto del relay cuando no se establece.
* `remoteCdpTimeoutMs` se aplica a las comprobaciones de accesibilidad de CDP remoto (no loopback).
* `remoteCdpHandshakeTimeoutMs` se aplica a las comprobaciones de accesibilidad de CDP remoto vía WebSocket.
* `attachOnly: true` significa “nunca inicies un navegador local; solo se conectará si ya se está ejecutando”.
* `color` + `color` por perfil tiñen la UI del navegador para que puedas ver qué perfil está activo.
* El perfil predeterminado es `chrome` (relay de extensión). Usa `defaultProfile: "openclaw"` para el navegador gestionado.
* Orden de autodetección: navegador predeterminado del sistema si está basado en Chromium; en caso contrario Chrome → Brave → Edge → Chromium → Chrome Canary.
* Los perfiles locales de `openclaw` asignan automáticamente `cdpPort`/`cdpUrl`; establece esos valores solo para CDP remoto.

<div id="use-brave-or-another-chromium-based-browser">
  ## Usa Brave (u otro navegador basado en Chromium)
</div>

Si el navegador **predeterminado de tu sistema** está basado en Chromium (Chrome/Brave/Edge/etc),
OpenClaw lo usa automáticamente. Configura `browser.executablePath` para anular
la autodetección:

Ejemplo de CLI:

```bash
openclaw config set browser.executablePath "/usr/bin/google-chrome"
```

```json5
// macOS
{
  browser: {
    executablePath: "/Applications/Brave Browser.app/Contents/MacOS/Brave Browser"
  }
}

// Windows
{
  browser: {
    executablePath: "C:\\Program Files\\BraveSoftware\\Brave-Browser\\Application\\brave.exe"
  }
}

// Linux
{
  browser: {
    executablePath: "/usr/bin/brave-browser"
  }
}
```

<div id="local-vs-remote-control">
  ## Control local vs remoto
</div>

* **Control local (predeterminado):** el Gateway inicia el servicio de control loopback y puede lanzar un navegador local.
* **Control remoto (host de nodo):** ejecuta un host de nodo en la máquina que tiene el navegador; el Gateway hace de proxy de las acciones del navegador hacia él.
* **CDP remoto:** establece `browser.profiles.<name>.cdpUrl` (o `browser.cdpUrl`) para
  conectarte a un navegador remoto basado en Chromium. En este caso, OpenClaw no lanzará un navegador local.

Las URL de CDP remoto pueden incluir autenticación:

* Tokens en la cadena de consulta (por ejemplo, `https://provider.example?token=<token>`)
* Autenticación HTTP Basic (por ejemplo, `https://user:pass@provider.example`)

OpenClaw preserva la autenticación al llamar a los endpoints `/json/*` y al conectarse
al WebSocket de CDP. Usa variables de entorno o gestores de secretos para
los tokens en lugar de almacenarlos en archivos de configuración.

<div id="node-browser-proxy-zero-config-default">
  ## Proxy de navegador de nodo (valor predeterminado sin configuración)
</div>

Si ejecutas un **host de nodo** en la máquina que tiene tu navegador, OpenClaw puede
redirigir automáticamente las llamadas de herramientas del navegador a ese nodo sin ninguna configuración adicional del navegador.
Esta es la ruta predeterminada para Gateways remotos.

Notas:

* El host de nodo expone su servidor local de control de navegador mediante un **comando proxy**.
* Los perfiles provienen de la propia configuración `browser.profiles` del nodo (igual que en local).
* Desactívalo si no lo necesitas:
  * En el nodo: `nodeHost.browserProxy.enabled=false`
  * En el Gateway: `gateway.nodes.browser.mode="off"`

<div id="browserless-hosted-remote-cdp">
  ## Browserless (CDP remoto alojado)
</div>

[Browserless](https://browserless.io) es un servicio de Chromium alojado que expone
endpoints CDP sobre HTTPS. Puedes apuntar un perfil de navegador de OpenClaw a un
endpoint regional de Browserless y autenticarte con tu clave de API.

Ejemplo:

```json5
{
  browser: {
    enabled: true,
    defaultProfile: "browserless",
    remoteCdpTimeoutMs: 2000,
    remoteCdpHandshakeTimeoutMs: 4000,
    profiles: {
      browserless: {
        cdpUrl: "https://production-sfo.browserless.io?token=<BROWSERLESS_API_KEY>",
        color: "#00AA00"
      }
    }
  }
}
```

Notas:

* Reemplaza `<BROWSERLESS_API_KEY>` por tu token real de Browserless.
* Elige el endpoint regional que corresponda a tu cuenta de Browserless (consulta su documentación).

<div id="security">
  ## Seguridad
</div>

Ideas clave:

* El control del navegador es solo mediante loopback; el acceso pasa a través de la autenticación del Gateway o el emparejamiento del nodo.
* Mantén el Gateway y cualquier host de nodo en una red privada (Tailscale); evita la exposición pública.
* Trata las URL/tokens CDP remotos como secretos; prioriza variables de entorno o un gestor de secretos.

Consejos para CDP remoto:

* Prefiere endpoints HTTPS y tokens de corta duración siempre que sea posible.
* Evita incrustar tokens de larga duración directamente en los archivos de configuración.

<div id="profiles-multi-browser">
  ## Perfiles (multinavegador)
</div>

OpenClaw admite múltiples perfiles con nombre (configuraciones de enrutamiento). Los perfiles pueden ser:

* **openclaw-managed**: una instancia de navegador dedicada basada en Chromium con su propio directorio de datos de usuario + puerto CDP
* **remote**: una URL CDP explícita (navegador basado en Chromium ejecutándose en otro lugar)
* **extension relay**: tus pestañas de Chrome existentes a través del relé local + extensión de Chrome

Valores predeterminados:

* El perfil `openclaw` se crea automáticamente si no existe.
* El perfil `chrome` es integrado para el relé de la extensión de Chrome (apunta a `http://127.0.0.1:18792` de forma predeterminada).
* Los puertos CDP locales se asignan entre **18800–18899** de forma predeterminada.
* Al eliminar un perfil, su directorio de datos local se mueve a la Papelera.

Todos los endpoints de control aceptan `?profile=<name>`; la CLI usa `--browser-profile`.

<div id="chrome-extension-relay-use-your-existing-chrome">
  ## Relay de extensión de Chrome (usa tu Chrome actual)
</div>

OpenClaw también puede controlar **tus pestañas de Chrome actuales** (sin una instancia separada de Chrome para “openclaw”) mediante un relay CDP local y una extensión de Chrome.

Guía completa: [Extensión de Chrome](/es/tools/chrome-extension)

Flujo:

* El Gateway se ejecuta localmente (en la misma máquina) o un host de nodo se ejecuta en la máquina del navegador.
* Un **servidor de relay** local escucha en un `cdpUrl` de loopback (de forma predeterminada: `http://127.0.0.1:18792`).
* Haces clic en el icono de la extensión **OpenClaw Browser Relay** en una pestaña para conectarlo a esa pestaña (no se conecta automáticamente).
* El agente controla esa pestaña mediante la herramienta `browser` normal, seleccionando el perfil correcto.

Si el Gateway se ejecuta en otro lugar, ejecuta un host de nodo en la máquina del navegador para que el Gateway pueda actuar como proxy de las acciones del navegador.

<div id="sandboxed-sessions">
  ### Sesiones en sandbox
</div>

Si la sesión del agente está en sandbox, la herramienta `browser` puede usar de forma predeterminada `target="sandbox"` (navegador en sandbox).
La toma de control mediante el relay de la extensión de Chrome requiere control del navegador host, así que:

* ejecuta la sesión sin sandbox, o
* establece `agents.defaults.sandbox.browser.allowHostControl: true` y usa `target="host"` al llamar a la herramienta.

<div id="setup">
  ### Configuración
</div>

1. Carga la extensión (dev/unpacked):

```bash
openclaw browser extension install
```

* Chrome → `chrome://extensions` → activa el “Modo de desarrollador”
* “Load unpacked” → selecciona el directorio mostrado por `openclaw browser extension path`
* Fija la extensión y luego haz clic en ella en la pestaña que quieras controlar (la insignia muestra `ON`).

2. Úsalo:

* CLI: `openclaw browser --browser-profile chrome tabs`
* Herramienta de agente: `browser` con `profile="chrome"`

Opcional: si quieres un nombre o puerto de relay diferente, crea tu propio perfil:

```bash
openclaw browser create-profile \
  --name my-chrome \
  --driver extension \
  --cdp-url http://127.0.0.1:18792 \
  --color "#00AA00"
```

Notas:

* Este modo utiliza Playwright-on-CDP para la mayoría de las operaciones (capturas de pantalla/instantáneas/acciones).
* Para desacoplar, haz clic de nuevo en el icono de la extensión.

<div id="isolation-guarantees">
  ## Garantías de aislamiento
</div>

* **Directorio dedicado para datos de usuario**: nunca accede a tu perfil personal del navegador.
* **Puertos dedicados**: evita `9222` para prevenir colisiones con flujos de trabajo de desarrollo.
* **Control determinista de pestañas**: dirige las pestañas por `targetId`, no por la “última pestaña”.

<div id="browser-selection">
  ## Selección del navegador
</div>

Al ejecutarse localmente, OpenClaw selecciona el primero disponible:

1. Chrome
2. Brave
3. Edge
4. Chromium
5. Chrome Canary

Puedes cambiarlo con `browser.executablePath`.

Plataformas:

* macOS: comprueba `/Applications` y `~/Applications`.
* Linux: busca `google-chrome`, `brave`, `microsoft-edge`, `chromium`, etc.
* Windows: comprueba ubicaciones de instalación habituales.

<div id="control-api-optional">
  ## API de control (opcional)
</div>

Solo para integraciones locales, el Gateway expone una pequeña API HTTP de loopback local:

* Estado/inicio/parada: `GET /`, `POST /start`, `POST /stop`
* Pestañas: `GET /tabs`, `POST /tabs/open`, `POST /tabs/focus`, `DELETE /tabs/:targetId`
* Instantánea/captura de pantalla: `GET /snapshot`, `POST /screenshot`
* Acciones: `POST /navigate`, `POST /act`
* Hooks: `POST /hooks/file-chooser`, `POST /hooks/dialog`
* Descargas: `POST /download`, `POST /wait/download`
* Depuración: `GET /console`, `POST /pdf`
* Depuración: `GET /errors`, `GET /requests`, `POST /trace/start`, `POST /trace/stop`, `POST /highlight`
* Red: `POST /response/body`
* Estado: `GET /cookies`, `POST /cookies/set`, `POST /cookies/clear`
* Estado: `GET /storage/:kind`, `POST /storage/:kind/set`, `POST /storage/:kind/clear`
* Configuración: `POST /set/offline`, `POST /set/headers`, `POST /set/credentials`, `POST /set/geolocation`, `POST /set/media`, `POST /set/timezone`, `POST /set/locale`, `POST /set/device`

Todos los endpoints aceptan `?profile=<nombre>`.

<div id="playwright-requirement">
  ### Requisito de Playwright
</div>

Algunas funciones (navegar/actuar/captura de IA/captura de rol, capturas de elementos, PDF) requieren
Playwright. Si Playwright no está instalado, esos endpoints devuelven un error 501 claro.
Las capturas ARIA y las capturas de pantalla básicas siguen funcionando para el Chrome gestionado por openclaw.
Para el controlador de relay de la extensión de Chrome, las capturas ARIA y las capturas de pantalla requieren Playwright.

Si ves `Playwright is not available in this gateway build`, instala el paquete completo
de Playwright (no `playwright-core`) y reinicia el Gateway, o vuelve a instalar
OpenClaw con compatibilidad con navegador.

<div id="how-it-works-internal">
  ## Cómo funciona (interno)
</div>

Flujo general:

* Un pequeño **servidor de control** acepta solicitudes HTTP.
* Se conecta a navegadores basados en Chromium (Chrome/Brave/Edge/Chromium) vía **CDP**.
* Para acciones avanzadas (clic/escritura/captura de pantalla/PDF), usa **Playwright** sobre
  CDP.
* Cuando Playwright no está disponible, solo están disponibles las operaciones que no usan Playwright.

Este diseño mantiene el agente en una interfaz estable y determinista, mientras te permite
cambiar entre navegadores y perfiles locales/remotos.

<div id="cli-quick-reference">
  ## Referencia rápida de la CLI
</div>

Todos los comandos admiten `--browser-profile <name>` para apuntar a un perfil específico.\
Todos los comandos también admiten `--json` para una salida legible por máquina (payloads estables).

Conceptos básicos:

* `openclaw browser status`
* `openclaw browser start`
* `openclaw browser stop`
* `openclaw browser tabs`
* `openclaw browser tab`
* `openclaw browser tab new`
* `openclaw browser tab select 2`
* `openclaw browser tab close 2`
* `openclaw browser open https://example.com`
* `openclaw browser focus abcd1234`
* `openclaw browser close abcd1234`

Inspección:

* `openclaw browser screenshot`
* `openclaw browser screenshot --full-page`
* `openclaw browser screenshot --ref 12`
* `openclaw browser screenshot --ref e12`
* `openclaw browser snapshot`
* `openclaw browser snapshot --format aria --limit 200`
* `openclaw browser snapshot --interactive --compact --depth 6`
* `openclaw browser snapshot --efficient`
* `openclaw browser snapshot --labels`
* `openclaw browser snapshot --selector "#main" --interactive`
* `openclaw browser snapshot --frame "iframe#main" --interactive`
* `openclaw browser console --level error`
* `openclaw browser errors --clear`
* `openclaw browser requests --filter api --clear`
* `openclaw browser pdf`
* `openclaw browser responsebody "**/api" --max-chars 5000`

Acciones:

* `openclaw browser navigate https://example.com`
* `openclaw browser resize 1280 720`
* `openclaw browser click 12 --double`
* `openclaw browser click e12 --double`
* `openclaw browser type 23 "hello" --submit`
* `openclaw browser press Enter`
* `openclaw browser hover 44`
* `openclaw browser scrollintoview e12`
* `openclaw browser drag 10 11`
* `openclaw browser select 9 OptionA OptionB`
* `openclaw browser download e12 /tmp/report.pdf`
* `openclaw browser waitfordownload /tmp/report.pdf`
* `openclaw browser upload /tmp/file.pdf`
* `openclaw browser fill --fields '[{"ref":"1","type":"text","value":"Ada"}]'`
* `openclaw browser dialog --accept`
* `openclaw browser wait --text "Done"`
* `openclaw browser wait "#main" --url "**/dash" --load networkidle --fn "window.ready===true"`
* `openclaw browser evaluate --fn '(el) => el.textContent' --ref 7`
* `openclaw browser highlight e12`
* `openclaw browser trace start`
* `openclaw browser trace stop`

Estado:

* `openclaw browser cookies`
* `openclaw browser cookies set session abc123 --url "https://example.com"`
* `openclaw browser cookies clear`
* `openclaw browser storage local get`
* `openclaw browser storage local set theme dark`
* `openclaw browser storage session clear`
* `openclaw browser set offline on`
* `openclaw browser set headers --json '{"X-Debug":"1"}'`
* `openclaw browser set credentials user pass`
* `openclaw browser set credentials --clear`
* `openclaw browser set geo 37.7749 -122.4194 --origin "https://example.com"`
* `openclaw browser set geo --clear`
* `openclaw browser set media dark`
* `openclaw browser set timezone America/New_York`
* `openclaw browser set locale en-US`
* `openclaw browser set device "iPhone 14"`

Notas:

* `upload` y `dialog` son llamadas de **preparación**; ejecútalas antes del clic/pulsación
  que dispara el selector/cuadro de diálogo.
* `upload` también puede establecer directamente entradas de archivo mediante `--input-ref` o `--element`.
* `snapshot`:
  * `--format ai` (predeterminado cuando Playwright está instalado): devuelve una captura de IA con referencias numéricas (`aria-ref="<n>"`).
  * `--format aria`: devuelve el árbol de accesibilidad (sin referencias; solo inspección).
  * `--efficient` (o `--mode efficient`): ajuste predefinido de captura de roles compacta (interactiva + compacta + profundidad + menor `maxChars`).
  * Valor predeterminado de configuración (solo herramienta/CLI): establece `browser.snapshotDefaults.mode: "efficient"` para usar capturas eficientes cuando el invocador no especifica un modo (ver [Gateway configuration](/es/gateway/configuration#browser-openclaw-managed-browser)).
  * Las opciones de captura de roles (`--interactive`, `--compact`, `--depth`, `--selector`) fuerzan una captura basada en roles con referencias como `ref=e12`.
  * `--frame "<iframe selector>"` restringe el ámbito de las capturas de roles a un iframe (se empareja con referencias de rol como `e12`).
  * `--interactive` produce una lista plana y fácil de seleccionar de elementos interactivos (lo mejor para ejecutar acciones).
  * `--labels` agrega una captura de pantalla solo del viewport con etiquetas de referencia superpuestas (imprime `MEDIA:<path>`).
* `click`/`type`/etc requieren un `ref` de `snapshot` (ya sea numérico `12` o referencia de rol `e12`).
  A propósito no se admiten selectores CSS para las acciones.

<div id="snapshots-and-refs">
  ## Instantáneas y refs
</div>

OpenClaw admite dos estilos de “instantánea”:

* **Instantánea de IA (refs numéricas)**: `openclaw browser snapshot` (predeterminado; `--format ai`)
  * Resultado: una instantánea de texto que incluye refs (referencias) numéricas.
  * Acciones: `openclaw browser click 12`, `openclaw browser type 23 "hello"`.
  * Internamente, la ref se resuelve mediante `aria-ref` de Playwright.

* **Instantánea por rol (refs de rol como `e12`)**: `openclaw browser snapshot --interactive` (o `--compact`, `--depth`, `--selector`, `--frame`)
  * Resultado: una lista/árbol basada en roles con `[ref=e12]` (y opcionalmente `[nth=1]`).
  * Acciones: `openclaw browser click e12`, `openclaw browser highlight e12`.
  * Internamente, la ref se resuelve mediante `getByRole(...)` (más `nth()` para duplicados).
  * Añade `--labels` para incluir una captura de pantalla del viewport con etiquetas `e12` superpuestas.

Comportamiento de las refs:

* Las refs (referencias) **no son estables entre navegaciones**; si algo falla, vuelve a ejecutar `snapshot` y usa una ref nueva.
* Si la instantánea por rol se tomó con `--frame`, las refs de rol quedan limitadas a ese iframe hasta la siguiente instantánea por rol.

<div id="wait-power-ups">
  ## Funciones avanzadas de espera
</div>

Puedes hacer esperas basadas no solo en tiempo o texto:

* Esperar una URL (globs compatibles con Playwright):
  * `openclaw browser wait --url "**/dash"`
* Esperar un estado de carga:
  * `openclaw browser wait --load networkidle`
* Esperar un predicado de JS:
  * `openclaw browser wait --fn "window.ready===true"`
* Esperar a que un selector se vuelva visible:
  * `openclaw browser wait "#main"`

Puedes combinarlos:

```bash
openclaw browser wait "#main" \
  --url "**/dash" \
  --load networkidle \
  --fn "window.ready===true" \
  --timeout-ms 15000
```

<div id="debug-workflows">
  ## Depurar flujos de trabajo
</div>

Cuando una acción falla (por ejemplo, “not visible”, “strict mode violation”, “covered”):

1. `openclaw browser snapshot --interactive`
2. Usa `click <ref>` / `type <ref>` (prioriza referencias de rol en modo interactivo)
3. Si sigue fallando: `openclaw browser highlight <ref>` para ver qué está intentando seleccionar Playwright
4. Si la página se comporta de forma extraña:
   * `openclaw browser errors --clear`
   * `openclaw browser requests --filter api --clear`
5. Para una depuración más detallada: registra una traza:
   * `openclaw browser trace start`
   * reproduce el problema
   * `openclaw browser trace stop` (muestra `TRACE:<path>`)

<div id="json-output">
  ## Salida JSON
</div>

`--json` es para automatización mediante scripts y herramientas estructuradas.

Ejemplos:

```bash
openclaw browser status --json
openclaw browser snapshot --interactive --json
openclaw browser requests --filter api --json
openclaw browser cookies --json
```

Las instantáneas de roles en JSON incluyen `refs` y un pequeño bloque `stats` (lines/chars/refs/interactive) para que las herramientas puedan evaluar el tamaño y la densidad del payload.

<div id="state-and-environment-knobs">
  ## Controles de estado y entorno
</div>

Estos son útiles para flujos del tipo “haz que el sitio se comporte como X”:

* Cookies: `cookies`, `cookies set`, `cookies clear`
* Almacenamiento: `storage local|session get|set|clear`
* Sin conexión: `set offline on|off`
* Encabezados HTTP: `set headers --json '{"X-Debug":"1"}'` (o `--clear`)
* Autenticación básica HTTP: `set credentials user pass` (o `--clear`)
* Geolocalización: `set geo <lat> <lon> --origin "https://example.com"` (o `--clear`)
* Preferencias de medios: `set media dark|light|no-preference|none`
* Zona horaria / configuración regional: `set timezone ...`, `set locale ...`
* Dispositivo / viewport:
  * `set device "iPhone 14"` (ajustes predefinidos de dispositivo de Playwright)
  * `set viewport 1280 720`

<div id="security-privacy">
  ## Seguridad y privacidad
</div>

* El perfil de navegador de openclaw puede contener sesiones iniciadas; trátalo como información sensible.
* `browser act kind=evaluate` / `openclaw browser evaluate` y `wait --fn`
  ejecutan JavaScript arbitrario en el contexto de la página. La inyección de prompts puede manipularlo. Desactívalo con `browser.evaluateEnabled=false` si no lo necesitas.
* Para inicios de sesión y notas anti-bot (X/Twitter, etc.), consulta [Inicio de sesión del navegador + publicación en X/Twitter](/es/tools/browser-login).
* Mantén privado el host del Gateway/nodo (solo loopback o solo tailnet).
* Los endpoints CDP remotos son muy potentes; protégelos y accede a ellos solo mediante túneles.

<div id="troubleshooting">
  ## Solución de problemas
</div>

Para problemas específicos de Linux (en especial con Chromium instalado mediante snap), consulta la sección
[Solución de problemas del navegador](/es/tools/browser-linux-troubleshooting).

<div id="agent-tools-how-control-works">
  ## Herramientas del agente y cómo funciona el control
</div>

El agente dispone de **una herramienta** para la automatización del navegador:

* `browser` — status/start/stop/tabs/open/focus/close/snapshot/screenshot/navigate/act

Cómo se asigna:

* `browser snapshot` devuelve un árbol de UI estable (AI o ARIA).
* `browser act` usa los IDs de `ref` del snapshot para hacer clic/escribir/arrastrar/seleccionar.
* `browser screenshot` captura píxeles (página completa o elemento).
* `browser` acepta:
  * `profile` para elegir un perfil de navegador con nombre (openclaw, chrome o CDP remoto).
  * `target` (`sandbox` | `host` | `node`) para seleccionar dónde se ejecuta el navegador.
  * En sesiones con sandbox, `target: "host"` requiere `agents.defaults.sandbox.browser.allowHostControl=true`.
  * Si se omite `target`: las sesiones con sandbox usan por defecto `sandbox`, las sesiones sin sandbox usan por defecto `host`.
  * Si hay conectado un nodo con capacidad de navegador, la herramienta puede enrutar automáticamente hacia él, a menos que fijes `target="host"` o `target="node"`.

Esto mantiene un comportamiento determinista del agente y evita selectores frágiles.