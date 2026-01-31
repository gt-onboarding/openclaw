---
title: Comando de ubicación
summary: "Comando de ubicación para nodos (location.get), modos de permiso y comportamiento en segundo plano"
read_when:
  - Al agregar compatibilidad de nodos de ubicación o UI de permisos de ubicación
  - Al diseñar flujos de ubicación en segundo plano y notificaciones push
---

<div id="location-command-nodes">
  # Comando de localización (nodos)
</div>

<div id="tldr">
  ## TL;DR
</div>

- `location.get` es un comando de nodo (a través de `node.invoke`).
- Desactivado de forma predeterminada.
- La configuración utiliza un selector: Desactivada / Mientras se usa / Siempre.
- Interruptor independiente: Ubicación precisa.

<div id="why-a-selector-not-just-a-switch">
  ## Por qué un selector (no solo un interruptor)
</div>

Los permisos del sistema operativo son multinivel. Podemos exponer un selector en la app, pero el sistema operativo sigue decidiendo el permiso efectivo.

- iOS/macOS: el usuario puede elegir **While Using** o **Always** en los avisos del sistema/Settings. La app puede solicitar elevar el nivel de permiso, pero el sistema operativo puede exigir ir a Settings.
- Android: la ubicación en segundo plano es un permiso independiente; en Android 10+ a menudo requiere un flujo que pasa por Settings.
- La ubicación precisa es una concesión independiente (iOS 14+ “Precise”, Android “fine” vs “coarse”).

El selector en la UI define el modo que solicitamos; el permiso efectivo se controla desde la configuración del sistema operativo.

<div id="settings-model">
  ## Modelo de configuración
</div>

Por dispositivo nodo:

- `location.enabledMode`: `off | whileUsing | always`
- `location.preciseEnabled`: bool

Comportamiento de la UI:

- Al seleccionar `whileUsing` se solicita permiso de ubicación en primer plano.
- Al seleccionar `always` primero se asegura `whileUsing` y luego se solicita permiso en segundo plano (o se lleva al usuario a Configuración si es necesario).
- Si el sistema operativo rechaza el nivel solicitado, se revierte al nivel más alto concedido y se muestra el estado.

<div id="permissions-mapping-nodepermissions">
  ## Asignación de permisos (node.permissions)
</div>

Opcional. El node de macOS reporta `location` mediante el mapa de permisos; iOS y Android pueden omitirlo.

<div id="command-locationget">
  ## Comando: `location.get`
</div>

Se llama a través de `node.invoke`.

Parámetros recomendados:

```json
{
  "timeoutMs": 10000,
  "maxAgeMs": 15000,
  "desiredAccuracy": "coarse|balanced|precise"
}
```

Cuerpo de la respuesta:

```json
{
  "lat": 48.20849,
  "lon": 16.37208,
  "accuracyMeters": 12.5,
  "altitudeMeters": 182.0,
  "speedMps": 0.0,
  "headingDeg": 270.0,
  "timestamp": "2026-01-03T12:34:56.000Z",
  "isPrecise": true,
  "source": "gps|wifi|cell|unknown"
}
```

Errores (códigos estables):

* `LOCATION_DISABLED`: el selector está desactivado.
* `LOCATION_PERMISSION_REQUIRED`: falta el permiso para el modo solicitado.
* `LOCATION_BACKGROUND_UNAVAILABLE`: la aplicación está en segundo plano pero solo se permite While Using.
* `LOCATION_TIMEOUT`: no se obtuvo una posición a tiempo.
* `LOCATION_UNAVAILABLE`: fallo del sistema / sin proveedores.


<div id="background-behavior-future">
  ## Comportamiento en segundo plano (futuro)
</div>

Objetivo: que el modelo pueda solicitar la ubicación incluso cuando el nodo está en segundo plano, pero solo cuando:

- El usuario haya seleccionado **Siempre**.
- El sistema operativo conceda acceso a la ubicación en segundo plano.
- A la app se le permita ejecutarse en segundo plano para ubicación (modo en segundo plano de iOS / servicio en primer plano de Android o permiso especial).

Flujo activado por push (futuro):

1) El Gateway envía un push al nodo (push silencioso o datos FCM).
2) El nodo se activa brevemente y solicita la ubicación al dispositivo.
3) El nodo reenvía el payload al Gateway.

Notas:

- iOS: se requieren el permiso «Siempre» y el modo de ubicación en segundo plano. El push silencioso puede ser limitado; es de esperar fallos intermitentes.
- Android: la ubicación en segundo plano puede requerir un servicio en primer plano; de lo contrario, es de esperar denegaciones.

<div id="modeltooling-integration">
  ## Integración de modelos y herramientas
</div>

- Superficie de la herramienta: la herramienta `nodes` incorpora la acción `location_get` (se requiere un nodo).
- CLI: `openclaw nodes location get --node <id>`.
- Directrices para el agente: solo llamar cuando el usuario haya habilitado la ubicación y entienda el ámbito.

<div id="ux-copy-suggested">
  ## Texto de UX (sugerido)
</div>

- Desactivado: “El uso compartido de ubicación está desactivado.”
- Mientras se usa: “Solo cuando OpenClaw está en uso.”
- Siempre: “Permitir ubicación en segundo plano. Requiere permiso del sistema.”
- Precisa: “Usar ubicación GPS precisa. Desactiva para compartir una ubicación aproximada.”