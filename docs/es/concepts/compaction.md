---
title: Compactación
summary: "Ventana de contexto y compactación: cómo OpenClaw mantiene las sesiones dentro de los límites del modelo"
read_when:
  - Quieres entender la compactación automática y /compact
  - Estás depurando sesiones largas que alcanzan los límites de contexto del modelo
---

<div id="context-window-compaction">
  # Ventana de contexto y compactación
</div>

Cada modelo tiene una **ventana de contexto** (máximo de tokens que puede procesar). Los chats de larga duración acumulan mensajes y resultados de herramientas; cuando esta ventana se queda sin margen, OpenClaw **compacta** el historial más antiguo para mantenerse dentro de esos límites.

<div id="what-compaction-is">
  ## Qué es la compactación
</div>

La compactación **resume las partes más antiguas de la conversación** en una entrada de resumen compacta y mantiene intactos los mensajes recientes. El resumen se almacena en el historial de la sesión, por lo que las peticiones futuras usan:

* El resumen de compactación
* Mensajes recientes posteriores al punto de compactación

La compactación **persiste** en el historial JSONL de la sesión.

<div id="configuration">
  ## Configuración
</div>

Consulta [Configuración y modos de compactación](/es/concepts/compaction) para conocer la configuración de `agents.defaults.compaction`.

<div id="auto-compaction-default-on">
  ## Compactación automática (activada por defecto)
</div>

Cuando una sesión se acerca o supera la ventana de contexto del modelo, OpenClaw activa la compactación automática y puede reintentar la solicitud original usando el contexto compactado.

Verás:

* `🧹 Auto-compaction complete` en modo detallado
* `/status` mostrando `🧹 Compactions: <count>`

Antes de la compactación, OpenClaw puede ejecutar un turno de **vaciado de memoria silencioso** para almacenar notas persistentes en disco. Consulta [Memory](/es/concepts/memory) para más detalles y configuración.

<div id="manual-compaction">
  ## Compacción manual
</div>

Usa `/compact` (opcionalmente con instrucciones) para forzar una ejecución de compacción:

```
/compact Focus on decisions and open questions
```

<div id="context-window-source">
  ## Origen de la ventana de contexto
</div>

La ventana de contexto es específica del modelo. OpenClaw usa la definición del modelo del catálogo de proveedores configurado para determinar sus límites.

<div id="compaction-vs-pruning">
  ## Compactación vs poda
</div>

* **Compactación**: genera un resumen y lo **persiste** en JSONL.
* **Poda de sesiones**: elimina solo resultados antiguos de **herramientas**, **en memoria**, por cada petición.

Consulta [/concepts/session-pruning](/es/concepts/session-pruning) para más detalles sobre la poda de sesiones.

<div id="tips">
  ## Consejos
</div>

* Usa `/compact` cuando las sesiones se sientan rancias o el contexto esté sobrecargado.
* Las salidas de herramientas muy grandes ya se truncan; un recorte adicional puede reducir aún más la acumulación de resultados de herramientas.
* Si necesitas empezar de cero, `/new` o `/reset` inicia un nuevo ID de sesión.