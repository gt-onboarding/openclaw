---
title: Registros
summary: "Puntos de registro, archivos de registro, estilos de registro por WS y formato de consola"
read_when:
  - Cambiar la salida o los formatos de registro
  - Depurar la salida de la CLI o del Gateway
---

<div id="logging">
  # Registro
</div>

Para una descripción general orientada al usuario (CLI + Control UI + configuración), consulta [/logging](/es/logging).

OpenClaw tiene dos tipos de registro:

* **Salida de consola** (lo que ves en la terminal / Debug UI).
* **Registros en archivos** (líneas JSON) escritos por el logger del Gateway.

<div id="file-based-logger">
  ## Logger basado en archivos
</div>

* El archivo de log rotativo predeterminado se encuentra en `/tmp/openclaw/` (un archivo por día): `openclaw-YYYY-MM-DD.log`
  * La fecha usa la zona horaria local del host del Gateway.
* La ruta del archivo de log y el nivel se pueden configurar mediante `~/.openclaw/openclaw.json`:
  * `logging.file`
  * `logging.level`

El formato del archivo es de un objeto JSON por línea.

La pestaña Logs del Control UI sigue este archivo en tiempo real a través del Gateway (`logs.tail`).
La CLI puede hacer lo mismo:

```bash
openclaw logs --follow
```

**Verbose vs. niveles de log**

* **Los logs de archivo** se controlan exclusivamente mediante `logging.level`.
* `--verbose` solo afecta la **verbosidad de la consola** (y el estilo de logs WS); **no**
  incrementa el nivel de log de archivo.
* Para capturar en los logs de archivo los detalles visibles solo en modo verbose, establece `logging.level` en `debug` o
  `trace`.

<div id="console-capture">
  ## Captura de consola
</div>

La CLI captura `console.log/info/warn/error/debug/trace` y los escribe en archivos de registro,
mientras sigue imprimiendo en stdout/stderr.

Puedes ajustar la verbosidad de la consola de forma independiente mediante:

* `logging.consoleLevel` (por defecto `info`)
* `logging.consoleStyle` (`pretty` | `compact` | `json`)

<div id="tool-summary-redaction">
  ## Redacción del resumen de herramientas
</div>

Los resúmenes detallados de herramientas (p. ej. `🛠️ Exec: ...`) pueden enmascarar tokens sensibles antes de que lleguen al flujo de la consola. Esto aplica **solo a herramientas** y no modifica los registros en archivos.

* `logging.redactSensitive`: `off` | `tools` (predeterminado: `tools`)
* `logging.redactPatterns`: array de cadenas regex (sobrescribe los valores predeterminados)
  * Usa cadenas regex sin procesar (auto `gi`), o `/pattern/flags` si necesitas flags personalizadas.
  * Las coincidencias se enmascaran conservando los primeros 6 + últimos 4 caracteres (longitud &gt;= 18), en caso contrario `***`.
  * Los valores predeterminados cubren asignaciones de claves comunes, flags de CLI, campos JSON, encabezados bearer, bloques PEM y prefijos de tokens habituales.

<div id="gateway-websocket-logs">
  ## Registros WebSocket del Gateway
</div>

El Gateway imprime registros del protocolo WebSocket en dos modos:

* **Modo normal (sin `--verbose`)**: solo se imprimen los resultados de RPC “interesantes”:
  * errores (`ok=false`)
  * llamadas lentas (umbral predeterminado: `>= 50ms`)
  * errores de análisis
* **Modo detallado (`--verbose`)**: imprime todo el tráfico de solicitudes/respuestas WS.

<div id="ws-log-style">
  ### Estilo de registro WS
</div>

`openclaw gateway` admite un selector de estilo por Gateway:

* `--ws-log auto` (predeterminado): el modo normal está optimizado; el modo detallado utiliza salida compacta
* `--ws-log compact`: salida compacta (solicitud/respuesta emparejada) cuando el modo detallado está activado
* `--ws-log full`: salida completa por frame cuando el modo detallado está activado
* `--compact`: alias de `--ws-log compact`

Ejemplos:

```bash
# optimized (only errors/slow)
openclaw gateway

# show all WS traffic (paired)
openclaw gateway --verbose --ws-log compact

# mostrar todo el tráfico WS (metadatos completos)
openclaw gateway --verbose --ws-log full
```

<div id="console-formatting-subsystem-logging">
  ## Formato de consola (registro por subsistemas)
</div>

El formateador de consola es **compatible con TTY** y escribe líneas coherentes con prefijo.
Los registradores de subsistema mantienen la salida agrupada y fácil de leer de un vistazo.

Comportamiento:

* **Prefijos de subsistema** en cada línea (por ejemplo, `[gateway]`, `[canvas]`, `[tailscale]`)
* **Colores por subsistema** (estables por subsistema) más color según nivel
* **Color cuando la salida es un TTY o el entorno parece un terminal avanzado** (`TERM`/`COLORTERM`/`TERM_PROGRAM`), respeta `NO_COLOR`
* **Prefijos de subsistema acortados**: elimina los prefijos iniciales `gateway/` + `channels/`, mantiene los últimos 2 segmentos (por ejemplo, `whatsapp/outbound`)
* **Sub-registradores por subsistema** (prefijo automático + campo estructurado `{ subsystem }`)
* **`logRaw()`** para salida de QR/UX (sin prefijo, sin formato)
* **Estilos de consola** (por ejemplo, `pretty | compact | json`)
* **Nivel de registro de consola** separado del nivel de registro a archivo (el archivo conserva todo el detalle cuando `logging.level` está configurado en `debug`/`trace`)
* **Los cuerpos de los mensajes de WhatsApp** se registran en `debug` (usa `--verbose` para verlos)

Esto mantiene estables los registros de archivo existentes mientras hace que la salida interactiva sea fácilmente escaneable.