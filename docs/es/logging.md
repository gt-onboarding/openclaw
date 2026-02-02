---
title: Registro
summary: "Descripción general del registro: archivos de log, salida de consola, seguimiento con la CLI y el Control UI"
read_when:
  - Necesitas una descripción general del sistema de registro para principiantes
  - Quieres configurar niveles o formatos de registro
  - Estás solucionando problemas y necesitas encontrar logs rápidamente
---

<div id="logging">
  # Registros
</div>

OpenClaw genera registros en dos ubicaciones:

* **Registros en archivos** (líneas JSON) escritos por el Gateway.
* **Salida de consola** mostrada en terminales y en la Control UI.

Esta página explica dónde se encuentran los registros, cómo leerlos y cómo configurar
los niveles y formatos de registro.

<div id="where-logs-live">
  ## Dónde se guardan los registros
</div>

De forma predeterminada, el Gateway escribe un archivo de registro rotativo en:

`/tmp/openclaw/openclaw-YYYY-MM-DD.log`

La fecha usa la zona horaria local del host donde se ejecuta el Gateway.

Puedes sobrescribir esta ruta en `~/.openclaw/openclaw.json`:

```json
{
  "logging": {
    "file": "/path/to/openclaw.log"
  }
}
```

<div id="how-to-read-logs">
  ## Cómo leer registros
</div>

<div id="cli-live-tail-recommended">
  ### CLI: seguimiento en tiempo real (recomendado)
</div>

Usa la CLI para seguir en tiempo real el archivo de registro del Gateway mediante RPC:

```bash
openclaw logs --follow
```

Modos de salida:

* **sesiones TTY**: líneas de registro claras, con color y estructuradas.
* **sesiones no TTY**: texto sin formato.
* `--json`: JSON delimitado por líneas (un evento de registro por línea).
* `--plain`: fuerza texto sin formato en sesiones TTY.
* `--no-color`: desactiva los colores ANSI.

En modo JSON, la CLI emite objetos etiquetados con `type`:

* `meta`: metadatos de la secuencia (archivo, cursor, tamaño)
* `log`: entrada de registro analizada
* `notice`: avisos de truncamiento/rotación
* `raw`: línea de registro sin analizar

Si el Gateway no está disponible, la CLI imprime una breve sugerencia para ejecutar:

```bash
openclaw doctor
```

<div id="control-ui-web">
  ### Control UI (web)
</div>

La pestaña **Logs** de Control UI muestra en tiempo real el mismo archivo usando `logs.tail`.
Consulta [/web/control-ui](/es/web/control-ui) para saber cómo abrirla.

<div id="channel-only-logs">
  ### Registros solo de canales
</div>

Para filtrar la actividad de los canales (WhatsApp/Telegram/etc), utiliza:

```bash
openclaw channels logs --channel whatsapp
```

<div id="log-formats">
  ## Formatos de registro
</div>

<div id="file-logs-jsonl">
  ### Registros en archivo (JSONL)
</div>

Cada línea del archivo de registros es un objeto JSON. La CLI y el Control UI procesan estas
entradas para mostrar una salida estructurada (hora, nivel, subsistema, mensaje).

<div id="console-output">
  ### Salida de consola
</div>

Los registros de consola son **adaptados a TTY** y están formateados para mejorar la legibilidad:

* Prefijos de subsistema (por ejemplo, `gateway/channels/whatsapp`)
* Colores por nivel (info/warn/error)
* Modo compacto o JSON opcional

El formato de la consola se controla mediante `logging.consoleStyle`.

<div id="configuring-logging">
  ## Configuración del registro de logs
</div>

Toda la configuración de logging se encuentra dentro de `logging` en `~/.openclaw/openclaw.json`.

```json
{
  "logging": {
    "level": "info",
    "file": "/tmp/openclaw/openclaw-YYYY-MM-DD.log",
    "consoleLevel": "info",
    "consoleStyle": "pretty",
    "redactSensitive": "tools",
    "redactPatterns": [
      "sk-.*"
    ]
  }
}
```

<div id="log-levels">
  ### Niveles de log
</div>

* `logging.level`: nivel de **logs en archivo** (JSONL).
* `logging.consoleLevel`: nivel de verbosidad de la **consola**.

`--verbose` solo afecta la salida en consola; no modifica los niveles de log en archivo.

<div id="console-styles">
  ### Estilos de consola
</div>

`logging.consoleStyle`:

* `pretty`: amigable para personas, con colores y marcas de tiempo.
* `compact`: salida más compacta (mejor para sesiones largas).
* `json`: JSON por línea (para procesadores de registros).

<div id="redaction">
  ### Redacción
</div>

Los resúmenes de herramientas pueden redactar tokens sensibles antes de que se impriman en la consola:

* `logging.redactSensitive`: `off` | `tools` (predeterminado: `tools`)
* `logging.redactPatterns`: lista de cadenas de expresiones regulares (regex) para sobrescribir el conjunto predeterminado

La redacción afecta **solo a la salida de la consola** y no modifica los registros en archivos.

<div id="diagnostics-opentelemetry">
  ## Diagnósticos + OpenTelemetry
</div>

Los diagnósticos son eventos estructurados y legibles por máquina para ejecuciones de modelos **y**
telemetría del flujo de mensajes (webhooks, encolado, estado de la sesión). **No**
reemplazan los registros; existen para alimentar métricas, trazas y otros exportadores.

Los eventos de diagnóstico se emiten dentro del proceso, pero los exportadores solo se conectan cuando
los diagnósticos + el complemento de exportación están habilitados.

<div id="opentelemetry-vs-otlp">
  ### OpenTelemetry vs OTLP
</div>

* **OpenTelemetry (OTel)**: el modelo de datos + SDKs para trazas, métricas y registros.
* **OTLP**: el protocolo de red usado para exportar datos de OTel a un colector o backend.
* OpenClaw exporta actualmente vía **OTLP/HTTP (protobuf)**.

<div id="signals-exported">
  ### Señales exportadas
</div>

* **Métricas**: contadores + histogramas (uso de tokens, flujo de mensajes, colas).
* **Trazas**: spans para el uso del modelo y el procesamiento de webhooks/mensajes.
* **Logs**: exportados mediante OTLP cuando `diagnostics.otel.logs` está habilitado. El
  volumen de logs puede ser alto; ten en cuenta `logging.level` y los filtros del exportador.

<div id="diagnostic-event-catalog">
  ### Catálogo de eventos de diagnóstico
</div>

Uso de modelos:

* `model.usage`: tokens, costo, duración, contexto, proveedor/modelo/canal, ID de sesión.

Flujo de mensajes:

* `webhook.received`: webhook recibido por canal.
* `webhook.processed`: webhook procesado + duración.
* `webhook.error`: errores del controlador de webhook.
* `message.queued`: mensaje encolado para procesamiento.
* `message.processed`: resultado + duración + error opcional.

Cola + sesión:

* `queue.lane.enqueue`: encolado en la lane de la cola de comandos + profundidad.
* `queue.lane.dequeue`: desencolado en la lane de la cola de comandos + tiempo de espera.
* `session.state`: transición de estado de la sesión + motivo.
* `session.stuck`: advertencia de sesión atascada + antigüedad.
* `run.attempt`: metadatos de intento/reintento de ejecución.
* `diagnostic.heartbeat`: contadores agregados (webhooks/cola/sesión).

<div id="enable-diagnostics-no-exporter">
  ### Habilitar diagnósticos (sin exportador)
</div>

Usa esta opción si quieres que los eventos de diagnóstico estén disponibles para complementos o sumideros personalizados:

```json
{
  "diagnostics": {
    "enabled": true
  }
}
```

<div id="diagnostics-flags-targeted-logs">
  ### Flags de diagnóstico (logs específicos)
</div>

Usa flags para activar logs de depuración adicionales y específicos sin elevar `logging.level`.
Las flags no distinguen mayúsculas de minúsculas y admiten comodines (por ejemplo, `telegram.*` o `*`).

```json
{
  "diagnostics": {
    "flags": ["telegram.http"]
  }
}
```

Sobrescritura de entorno (puntual):

```
OPENCLAW_DIAGNOSTICS=telegram.http,telegram.payload
```

Notas:

* Los registros de flags van al archivo de registro estándar (el mismo archivo que `logging.file`).
* La salida se sigue ocultando según `logging.redactSensitive`.
* Guía completa: [/diagnostics/flags](/es/diagnostics/flags).

<div id="export-to-opentelemetry">
  ### Exportar a OpenTelemetry
</div>

Los diagnósticos se pueden exportar mediante el complemento `diagnostics-otel` (OTLP/HTTP). Esto
funciona con cualquier colector o backend de OpenTelemetry que acepte OTLP/HTTP.

```json
{
  "plugins": {
    "allow": ["diagnostics-otel"],
    "entries": {
      "diagnostics-otel": {
        "enabled": true
      }
    }
  },
  "diagnostics": {
    "enabled": true,
    "otel": {
      "enabled": true,
      "endpoint": "http://otel-collector:4318",
      "protocol": "http/protobuf",
      "serviceName": "openclaw-gateway",
      "traces": true,
      "metrics": true,
      "logs": true,
      "sampleRate": 0.2,
      "flushIntervalMs": 60000
    }
  }
}
```

Notas:

* También puedes habilitar el complemento con `openclaw plugins enable diagnostics-otel`.
* `protocol` actualmente solo admite `http/protobuf`. `grpc` se ignora.
* Las métricas incluyen uso de tokens, costo, tamaño de contexto, duración de ejecución y
  contadores/histogramas de flujo de mensajes (webhooks, encolamiento, estado de sesión, profundidad/tiempo de espera de la cola).
* Las trazas y métricas se pueden activar o desactivar con `traces` / `metrics` (predeterminado: activado). Las trazas
  incluyen segmentos (spans) de uso de modelo más segmentos de procesamiento de webhooks/mensajes cuando están habilitadas.
* Define `headers` cuando tu colector requiera autenticación.
* Variables de entorno admitidas: `OTEL_EXPORTER_OTLP_ENDPOINT`,
  `OTEL_SERVICE_NAME`, `OTEL_EXPORTER_OTLP_PROTOCOL`.

<div id="exported-metrics-names-types">
  ### Métricas exportadas (nombres + tipos)
</div>

Uso del modelo:

* `openclaw.tokens` (contador, atributos: `openclaw.token`, `openclaw.channel`,
  `openclaw.provider`, `openclaw.model`)
* `openclaw.cost.usd` (contador, atributos: `openclaw.channel`, `openclaw.provider`,
  `openclaw.model`)
* `openclaw.run.duration_ms` (histograma, atributos: `openclaw.channel`,
  `openclaw.provider`, `openclaw.model`)
* `openclaw.context.tokens` (histograma, atributos: `openclaw.context`,
  `openclaw.channel`, `openclaw.provider`, `openclaw.model`)

Flujo de mensajes:

* `openclaw.webhook.received` (contador, atributos: `openclaw.channel`,
  `openclaw.webhook`)
* `openclaw.webhook.error` (contador, atributos: `openclaw.channel`,
  `openclaw.webhook`)
* `openclaw.webhook.duration_ms` (histograma, atributos: `openclaw.channel`,
  `openclaw.webhook`)
* `openclaw.message.queued` (contador, atributos: `openclaw.channel`,
  `openclaw.source`)
* `openclaw.message.processed` (contador, atributos: `openclaw.channel`,
  `openclaw.outcome`)
* `openclaw.message.duration_ms` (histograma, atributos: `openclaw.channel`,
  `openclaw.outcome`)

Colas y sesiones:

* `openclaw.queue.lane.enqueue` (contador, atributos: `openclaw.lane`)
* `openclaw.queue.lane.dequeue` (contador, atributos: `openclaw.lane`)
* `openclaw.queue.depth` (histograma, atributos: `openclaw.lane` o
  `openclaw.channel=heartbeat`)
* `openclaw.queue.wait_ms` (histograma, atributos: `openclaw.lane`)
* `openclaw.session.state` (contador, atributos: `openclaw.state`, `openclaw.reason`)
* `openclaw.session.stuck` (contador, atributos: `openclaw.state`)
* `openclaw.session.stuck_age_ms` (histograma, atributos: `openclaw.state`)
* `openclaw.run.attempt` (contador, atributos: `openclaw.attempt`)

<div id="exported-spans-names-key-attributes">
  ### Spans exportados (nombres y atributos clave)
</div>

* `openclaw.model.usage`
  * `openclaw.channel`, `openclaw.provider`, `openclaw.model`
  * `openclaw.sessionKey`, `openclaw.sessionId`
  * `openclaw.tokens.*` (input/output/cache&#95;read/cache&#95;write/total)
* `openclaw.webhook.processed`
  * `openclaw.channel`, `openclaw.webhook`, `openclaw.chatId`
* `openclaw.webhook.error`
  * `openclaw.channel`, `openclaw.webhook`, `openclaw.chatId`,
    `openclaw.error`
* `openclaw.message.processed`
  * `openclaw.channel`, `openclaw.outcome`, `openclaw.chatId`,
    `openclaw.messageId`, `openclaw.sessionKey`, `openclaw.sessionId`,
    `openclaw.reason`
* `openclaw.session.stuck`
  * `openclaw.state`, `openclaw.ageMs`, `openclaw.queueDepth`,
    `openclaw.sessionKey`, `openclaw.sessionId`

<div id="sampling-flushing">
  ### Muestreo y vaciado
</div>

* Muestreo de trazas: `diagnostics.otel.sampleRate` (0.0–1.0, solo spans raíz).
* Intervalo de exportación de métricas: `diagnostics.otel.flushIntervalMs` (mín. 1000 ms).

<div id="protocol-notes">
  ### Notas sobre el protocolo
</div>

* Los endpoints OTLP/HTTP se pueden configurar mediante `diagnostics.otel.endpoint` o
  `OTEL_EXPORTER_OTLP_ENDPOINT`.
* Si el endpoint ya contiene `/v1/traces` o `/v1/metrics`, se utiliza sin modificaciones.
* Si el endpoint ya contiene `/v1/logs`, se utiliza sin modificaciones para los logs.
* `diagnostics.otel.logs` habilita la exportación de logs OTLP para la salida principal del logger.

<div id="log-export-behavior">
  ### Comportamiento de exportación de logs
</div>

* Los logs OTLP usan los mismos registros estructurados que se escriben en `logging.file`.
* Los logs OTLP respetan `logging.level` (nivel de registro para archivos). El enmascarado en consola **no** se aplica
  a los logs OTLP.
* En instalaciones de alto volumen se debe preferir el muestreo y filtrado mediante el colector OTLP.

<div id="troubleshooting-tips">
  ## Consejos para solucionar problemas
</div>

* **¿No se puede acceder al Gateway?** Ejecuta primero `openclaw doctor`.
* **¿Los registros están vacíos?** Verifica que el Gateway se esté ejecutando y escribiendo en la ruta de archivo indicada en
  `logging.file`.
* **¿Necesitas más detalles?** Establece `logging.level` en `debug` o `trace` y vuelve a intentarlo.