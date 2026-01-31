---
title: Peekaboo
summary: "Integración de PeekabooBridge para la automatización de la UI en macOS"
read_when:
  - Hospedar PeekabooBridge en OpenClaw.app
  - Integrar Peekaboo mediante Swift Package Manager
  - Cambiar el protocolo y las rutas de PeekabooBridge
---

<div id="peekaboo-bridge-macos-ui-automation">
  # Peekaboo Bridge (automatización de la UI en macOS)
</div>

OpenClaw puede alojar **PeekabooBridge** como un broker local de automatización de la UI, sensible a los permisos. Esto permite que la CLI `peekaboo` controle la automatización de la UI reutilizando los permisos TCC de la app de macOS.

<div id="what-this-is-and-isnt">
  ## Qué es esto (y qué no lo es)
</div>

- **Host**: OpenClaw.app puede actuar como host de PeekabooBridge.
- **Cliente**: usa la CLI `peekaboo` (no hay ninguna interfaz `openclaw ui ...` separada).
- **UI**: las superposiciones visuales permanecen en Peekaboo.app; OpenClaw es un host intermediario ligero.

<div id="enable-the-bridge">
  ## Habilitar el puente
</div>

En la app de macOS:

- Settings → **Enable Peekaboo Bridge**

Cuando está habilitado, OpenClaw inicia un servidor de sockets UNIX local. Si está deshabilitado, el host
deja de ejecutarse y `peekaboo` recurrirá a otros hosts disponibles.

<div id="client-discovery-order">
  ## Orden de descubrimiento del cliente
</div>

Los clientes de Peekaboo normalmente prueban los hosts en este orden:

1. Peekaboo.app (experiencia completa)
2. Claude.app (si está instalada)
3. OpenClaw.app (broker ligero)

Usa `peekaboo bridge status --verbose` para ver qué host está activo y qué
ruta de socket está en uso. Puedes anularlo con:

```bash
export PEEKABOO_BRIDGE_SOCKET=/path/to/bridge.sock
```


<div id="security-permissions">
  ## Seguridad y permisos
</div>

- El bridge valida las **firmas de código de quien realiza la llamada**; se aplica una lista de permitidos de TeamIDs
  (TeamID del host Peekaboo + TeamID de la app OpenClaw).
- Las solicitudes expiran tras ~10 segundos.
- Si faltan los permisos requeridos, el bridge devuelve un mensaje de error claro
  en lugar de abrir Configuración del Sistema.

<div id="snapshot-behavior-automation">
  ## Comportamiento de las instantáneas (automatización)
</div>

Las instantáneas se almacenan en memoria y caducan automáticamente después de un breve período de tiempo.
Si necesitas conservarlas durante más tiempo, vuelve a capturarlas desde el cliente.

<div id="troubleshooting">
  ## Solución de problemas
</div>

- Si `peekaboo` muestra “bridge client is not authorized”, asegúrate de que el cliente
  esté correctamente firmado o ejecuta el host con `PEEKABOO_ALLOW_UNSIGNED_SOCKET_CLIENTS=1`
  solo en modo **debug**.
- Si no se encuentra ningún host, abre una de las aplicaciones host (Peekaboo.app u OpenClaw.app)
  y confirma que se hayan otorgado los permisos.