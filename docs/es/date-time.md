---
title: Fecha y hora
summary: "Gestión de fechas y horas en envolturas, prompts, herramientas y conectores"
read_when:
  - Estás cambiando cómo se muestran las marcas de tiempo al modelo o a los usuarios
  - Estás depurando el formato de fecha y hora en los mensajes o en la salida del prompt del sistema
---

<div id="date-time">
  # Fecha y hora
</div>

OpenClaw usa por defecto **la hora local del host para las marcas de tiempo de transporte** y **la zona horaria del usuario solo en el prompt del sistema**.
Las marcas de tiempo del proveedor se conservan para que las herramientas mantengan su semántica nativa (la hora actual está disponible mediante `session_status`).

<div id="message-envelopes-local-by-default">
  ## Envoltorios de mensajes (hora local de forma predeterminada)
</div>

Los mensajes entrantes se encapsulan con una marca de tiempo (precisión al minuto):

```
[Provider ... 2026-01-05 16:26 PST] message text
```

La marca de tiempo de este sobre es, **por defecto, local al host**, independientemente de la zona horaria del proveedor.

Puedes modificar este comportamiento:

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

* `envelopeTimezone: "utc"` utiliza UTC.
* `envelopeTimezone: "local"` utiliza la zona horaria del host.
* `envelopeTimezone: "user"` utiliza `agents.defaults.userTimezone` (si no, recurre a la zona horaria del host).
* Utiliza una zona horaria IANA explícita (por ejemplo, `"America/Chicago"`) para una zona horaria fija.
* `envelopeTimestamp: "off"` elimina las marcas de tiempo absolutas de los encabezados del sobre.
* `envelopeElapsed: "off"` elimina los sufijos de tiempo transcurrido (del estilo `+2m`).

<div id="examples">
  ### Ejemplos
</div>

**Local (por defecto):**

```
[WhatsApp +1555 2026-01-18 00:19 PST] hello
```

**Zona horaria del usuario:**

```
[WhatsApp +1555 2026-01-18 00:19 CST] hello
```

**Tiempo transcurrido activado:**

```
[WhatsApp +1555 +30s 2026-01-18T05:19Z] seguimiento
```

<div id="system-prompt-current-date-time">
  ## System prompt: Fecha y hora actuales
</div>

Si se conoce la zona horaria del usuario, el system prompt incluye una sección dedicada
**Fecha y hora actuales** con **solo la zona horaria** (sin formato de reloj/hora)
para mantener estable la caché del prompt:

```
Time zone: America/Chicago
```

Cuando el agente necesite la hora actual, utiliza la herramienta `session_status`; la tarjeta de estado incluye una línea con la marca de tiempo.

<div id="system-event-lines-local-by-default">
  ## Líneas de eventos del sistema (local por defecto)
</div>

Los eventos del sistema en cola que se insertan en el contexto del agente se anteceden con una marca de tiempo que utiliza la misma selección de zona horaria que las envolturas de mensajes (valor predeterminado: zona horaria local del host).

```
System: [2026-01-12 12:19:17 PST] Model switched.
```

<div id="configure-user-timezone-format">
  ### Configurar la zona horaria y el formato horario del usuario
</div>

```json5
{
  agents: {
    defaults: {
      userTimezone: "America/Chicago",
      timeFormat: "auto" // automático | 12 | 24
    }
  }
}
```

* `userTimezone` establece la **zona horaria local del usuario** para el contexto del prompt.
* `timeFormat` controla el **formato de hora de 12h/24h** en el prompt. `auto` sigue las preferencias del sistema operativo.

<div id="time-format-detection-auto">
  ## Detección automática del formato de hora
</div>

Cuando `timeFormat: "auto"`, OpenClaw inspecciona la preferencia del sistema operativo (macOS/Windows)
y, en caso necesario, utiliza el formato asociado a la configuración regional. El valor detectado se **almacena en caché por proceso**
para evitar llamadas repetidas al sistema.

<div id="tool-payloads-connectors-raw-provider-time-normalized-fields">
  ## Cargas útiles de herramientas + conectores (hora sin procesar del proveedor + campos normalizados)
</div>

Las herramientas de canal devuelven **marcas de tiempo nativas del proveedor** y añaden campos normalizados para mantener la coherencia:

* `timestampMs`: milisegundos desde la época Unix (UTC)
* `timestampUtc`: cadena UTC en formato ISO 8601

Los campos sin procesar del proveedor se conservan para que no se pierda nada.

* Slack: cadenas similares a epoch provenientes de la API
* Discord: marcas de tiempo UTC en formato ISO
* Telegram/WhatsApp: marcas de tiempo numéricas/ISO específicas del proveedor

Si necesitas la hora local, conviértela posteriormente usando la zona horaria conocida.

<div id="related-docs">
  ## Documentos relacionados
</div>

* [System Prompt](/es/concepts/system-prompt)
* [Zonas horarias](/es/concepts/timezone)
* [Mensajes](/es/concepts/messages)