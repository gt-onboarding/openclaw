---
title: Canvas
summary: "Panel Canvas controlado por el Agente, incrustado mediante WKWebView + esquema de URL personalizado"
read_when:
  - Implementar el panel Canvas de macOS
  - Añadir controles del Agente para el espacio de trabajo visual
  - Depurar las cargas del Canvas de WKWebView
---

<div id="canvas-macos-app">
  # Canvas (macOS app)
</div>

La aplicación de macOS integra un **panel Canvas** controlado por un agente mediante `WKWebView`. Es un espacio de trabajo visual liviano para HTML/CSS/JS, A2UI y pequeñas superficies de UI interactivas.

<div id="where-canvas-lives">
  ## Dónde se encuentra Canvas
</div>

El estado de Canvas se almacena en Application Support:

- `~/Library/Application Support/OpenClaw/canvas/<session>/...`

El panel de Canvas sirve estos archivos mediante un **esquema de URL personalizado**:

- `openclaw-canvas://<session>/<path>`

Ejemplos:

- `openclaw-canvas://main/` → `<canvasRoot>/main/index.html`
- `openclaw-canvas://main/assets/app.css` → `<canvasRoot>/main/assets/app.css`
- `openclaw-canvas://main/widgets/todo/` → `<canvasRoot>/main/widgets/todo/index.html`

Si no existe `index.html` en la raíz, la aplicación muestra una **página de plantilla integrada**.

<div id="panel-behavior">
  ## Comportamiento del panel
</div>

- Panel sin bordes y redimensionable, anclado cerca de la barra de menús (o del cursor del ratón).
- Recuerda el tamaño y la posición para cada sesión.
- Se recarga automáticamente cuando cambian los archivos locales de Canvas.
- Solo un panel de Canvas es visible a la vez (la sesión se cambia según sea necesario).

Canvas se puede desactivar desde Settings → **Allow Canvas**. Cuando está desactivado, los comandos del nodo de Canvas devuelven `CANVAS_DISABLED`.

<div id="agent-api-surface">
  ## Superficie de la API del Agente
</div>

Canvas se expone a través del **WebSocket del Gateway**, por lo que el agente puede:

* mostrar/ocultar el panel
* navegar a una ruta o URL
* evaluar JavaScript
* capturar una imagen instantánea

Ejemplos de CLI:

```bash
openclaw nodes canvas present --node <id>
openclaw nodes canvas navigate --node <id> --url "/"
openclaw nodes canvas eval --node <id> --js "document.title"
openclaw nodes canvas snapshot --node <id>
```

Notas:

* `canvas.navigate` acepta **rutas locales de Canvas**, direcciones URL `http(s)` y direcciones URL `file://`.
* Si proporcionas `"/"`, Canvas muestra la estructura inicial local o `index.html`.


<div id="a2ui-in-canvas">
  ## A2UI en Canvas
</div>

El host Canvas del Gateway aloja A2UI y lo renderiza dentro del panel de Canvas.
Cuando el Gateway anuncia un host de Canvas, la app de macOS navega automáticamente a la
página del host de A2UI la primera vez que se abre.

URL predeterminada del host de A2UI:

```
http://<gateway-host>:18793/__openclaw__/a2ui/
```


<div id="a2ui-commands-v08">
  ### Comandos A2UI (v0.8)
</div>

Actualmente, Canvas acepta mensajes de servidor a cliente de **A2UI v0.8**:

* `beginRendering`
* `surfaceUpdate`
* `dataModelUpdate`
* `deleteSurface`

`createSurface` (v0.9) no es compatible.

Ejemplo de CLI:

```bash
cat > /tmp/a2ui-v0.8.jsonl <<'EOFA2'
{"surfaceUpdate":{"surfaceId":"main","components":[{"id":"root","component":{"Column":{"children":{"explicitList":["title","content"]}}}},{"id":"title","component":{"Text":{"text":{"literalString":"Canvas (A2UI v0.8)"},"usageHint":"h1"}}},{"id":"content","component":{"Text":{"text":{"literalString":"If you can read this, A2UI push works."},"usageHint":"body"}}}]}}
{"beginRendering":{"surfaceId":"main","root":"root"}}
EOFA2

openclaw nodes canvas a2ui push --jsonl /tmp/a2ui-v0.8.jsonl --node <id>
```

Smoke test rápido:

```bash
openclaw nodes canvas a2ui push --node <id> --text "Hello from A2UI"
```


<div id="triggering-agent-runs-from-canvas">
  ## Iniciar ejecuciones de agentes desde Canvas
</div>

Canvas puede iniciar nuevas ejecuciones de agentes mediante enlaces profundos (deep links):

* `openclaw://agent?...`

Ejemplo (en JS):

```js
window.location.href = "openclaw://agent?message=Review%20this%20design";
```

La aplicación pedirá confirmación a menos que se proporcione una clave válida.


<div id="security-notes">
  ## Notas de seguridad
</div>

- El esquema de Canvas bloquea la navegación entre directorios; los archivos deben estar ubicados bajo la raíz de la sesión.
- El contenido local de Canvas utiliza un esquema personalizado (no se requiere servidor de loopback).
- Las URL externas `http(s)` solo están permitidas cuando se navega explícitamente a ellas.