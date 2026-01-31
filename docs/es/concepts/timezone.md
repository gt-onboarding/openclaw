---
title: Zona horaria
summary: "Gestión de la zona horaria para agentes, envolturas y prompts"
read_when:
  - Necesitas entender cómo se normalizan las marcas de tiempo para el modelo
  - Configurar la zona horaria del usuario para los prompts del sistema
---

<div id="timezones">
  # Zonas horarias
</div>

OpenClaw estandariza las marcas de tiempo para que el modelo vea un **único tiempo de referencia**.

<div id="message-envelopes-local-by-default">
  ## Sobres de mensajes (zona horaria local por defecto)
</div>

Los mensajes entrantes se encapsulan en un sobre como:

```
[Provider ... 2026-01-05 16:26 PST] message text
```

La marca de tiempo en el sobre del mensaje es **del host local de forma predeterminada**, con precisión de minutos.

Puedes anular este valor con:

```json5
{
  agents: {
    defaults: {
      envelopeTimezone: "local", // "utc" | "local" | "user" | zona horaria IANA
      envelopeTimestamp: "on", // "on" | "off"
      envelopeElapsed: "on" // "on" | "off"
    }
  }
}
```

* `envelopeTimezone: "utc"` usa UTC.
* `envelopeTimezone: "user"` usa `agents.defaults.userTimezone` (si no está definida, recurre a la zona horaria del host).
* Usa una zona horaria IANA explícita (por ejemplo, `"Europe/Vienna"`) para un desplazamiento fijo.
* `envelopeTimestamp: "off"` elimina las marcas de tiempo absolutas de las cabeceras del sobre.
* `envelopeElapsed: "off"` elimina los sufijos de tiempo transcurrido (del estilo `+2m`).

<div id="examples">
  ### Ejemplos
</div>

**Local (por defecto):**

```
[Signal Alice +1555 2026-01-18 00:19 PST] hello
```

**Zona horaria fija:**

```
[Signal Alice +1555 2026-01-18 06:19 GMT+1] hello
```

**Tiempo transcurrido:**

```
[Signal Alice +1555 +2m 2026-01-18T05:19Z] follow-up
```

<div id="tool-payloads-raw-provider-data-normalized-fields">
  ## Cargas útiles de herramientas (datos sin procesar del proveedor + campos normalizados)
</div>

Las llamadas a herramientas (`channels.discord.readMessages`, `channels.slack.readMessages`, etc.) devuelven **marcas de tiempo sin procesar del proveedor**.
También añadimos campos normalizados para mantener la coherencia:

* `timestampMs` (milisegundos desde la época Unix en UTC)
* `timestampUtc` (cadena UTC en formato ISO 8601)

Se conservan los campos sin procesar del proveedor.

<div id="user-timezone-for-the-system-prompt">
  ## Zona horaria del usuario para el prompt del sistema
</div>

Establece `agents.defaults.userTimezone` para indicar al modelo la zona horaria local del usuario. Si no se establece, OpenClaw determina la **zona horaria del host en tiempo de ejecución** (sin escribir en la configuración).

```json5
{
  agents: { defaults: { userTimezone: "America/Chicago" } }
}
```

El prompt del sistema incluye:

* la sección `Current Date & Time` con hora local y zona horaria
* `Time format: 12-hour` o `24-hour`

Puedes controlar el formato del prompt con `agents.defaults.timeFormat` (`auto` | `12` | `24`).

Consulta [Date &amp; Time](/es/date-time) para conocer el comportamiento completo y ver ejemplos.
