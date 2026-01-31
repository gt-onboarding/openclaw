---
title: Salud
summary: "Cómo la app de macOS informa sobre los estados de salud de Gateway/Baileys"
read_when:
  - Depuración de los indicadores de salud de la app de macOS
---

<div id="health-checks-on-macos">
  # Comprobaciones de estado en macOS
</div>

Cómo comprobar si el canal vinculado está en buen estado desde la aplicación de la barra de menús.

<div id="menu-bar">
  ## Barra de menús
</div>

* El punto de estado ahora refleja el estado de Baileys:
  * Verde: vinculado + socket abierto recientemente.
  * Naranja: conectando/reintentando.
  * Rojo: sesión cerrada o falló la comprobación de estado.
* La línea secundaria muestra &quot;linked · auth 12m&quot; o el motivo del fallo.
* El elemento de menú &quot;Run Health Check&quot; ejecuta una comprobación de estado bajo demanda.

<div id="settings">
  ## Configuración
</div>

* La pestaña General incorpora una tarjeta de estado Health que muestra lo siguiente: antigüedad de la autenticación vinculada, ruta/recuento del almacén de sesión, hora de la última comprobación, último error/código de estado y botones “Run Health Check” / “Reveal Logs”.
* Usa una instantánea en caché para que la UI cargue al instante y se degrade correctamente cuando no haya conexión.
* La **pestaña Channels** muestra el estado del canal y los controles para WhatsApp/Telegram (QR de inicio de sesión, cierre de sesión, sondeo, última desconexión/error).

<div id="how-the-probe-works">
  ## Cómo funciona la sonda
</div>

* La app ejecuta `openclaw health --json` mediante `ShellExecutor` aproximadamente cada 60 segundos y bajo demanda. La sonda carga las credenciales e informa el estado sin enviar mensajes.
* Almacena en caché por separado la última instantánea válida y el último error para evitar parpadeos; muestra la marca de tiempo de cada uno.

<div id="when-in-doubt">
  ## En caso de duda
</div>

* Aún puedes usar el flujo de la CLI en [Gateway health](/es/gateway/health) (`openclaw status`, `openclaw status --deep`, `openclaw health --json`) y usar `tail` sobre `/tmp/openclaw/openclaw-*.log` para buscar `web-heartbeat` / `web-reconnect`.