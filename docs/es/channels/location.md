---
title: Ubicación
summary: "Procesamiento de ubicación de canales entrantes (Telegram + WhatsApp) y campos de contexto"
read_when:
  - Agregar o modificar el procesamiento de ubicación de canales
  - Usar campos de contexto de ubicación en prompts o herramientas de agentes
---

<div id="channel-location-parsing">
  # Análisis de ubicación del canal
</div>

OpenClaw normaliza las ubicaciones compartidas desde canales de chat en:

- texto legible por humanos anexado al cuerpo del mensaje entrante, y
- campos estructurados en el payload de contexto de respuesta automática.

Actualmente se admite:

- **Telegram** (marcadores/pins de ubicación + locales/establecimientos + ubicaciones en tiempo real)
- **WhatsApp** (`locationMessage` + `liveLocationMessage`)
- **Matrix** (`m.location` con `geo_uri`)

<div id="text-formatting">
  ## Formato de texto
</div>

Las ubicaciones se representan como líneas legibles sin corchetes:

* Pin:
  * `📍 48.858844, 2.294351 ±12m`
* Lugar con nombre:
  * `📍 Eiffel Tower — Champ de Mars, Paris (48.858844, 2.294351 ±12m)`
* Compartición en vivo:
  * `🛰 Live location: 48.858844, 2.294351 ±12m`

Si el canal incluye un título o comentario, se añade en la siguiente línea:

```
📍 48.858844, 2.294351 ±12m
Meet here
```


<div id="context-fields">
  ## Campos de contexto
</div>

Cuando hay información de ubicación, estos campos se agregan a `ctx`:

- `LocationLat` (número)
- `LocationLon` (número)
- `LocationAccuracy` (número, metros; opcional)
- `LocationName` (cadena; opcional)
- `LocationAddress` (cadena; opcional)
- `LocationSource` (`pin | place | live`)
- `LocationIsLive` (booleano)

<div id="channel-notes">
  ## Notas del canal
</div>

- **Telegram**: los lugares se asignan a `LocationName/LocationAddress`; las ubicaciones en directo usan `live_period`.
- **WhatsApp**: `locationMessage.comment` y `liveLocationMessage.caption` se añaden como pie de foto.
- **Matrix**: `geo_uri` se interpreta como un punto de ubicación (pin); la altitud se ignora y `LocationIsLive` siempre es false.