---
title: Icono
summary: "Estados y animaciones del icono de la barra de menús de OpenClaw en macOS"
read_when:
  - Al cambiar el comportamiento del icono de la barra de menús
---

<div id="menu-bar-icon-states">
  # Estados del icono en la barra de menús
</div>

Autor: steipete · Actualizado: 2025-12-06 · Ámbito: app para macOS (`apps/macos`)

- **Inactivo:** Animación normal del icono (parpadeo, balanceo ocasional).
- **Pausado:** El elemento de estado usa `appearsDisabled`; sin movimiento.
- **Disparador de voz (orejas grandes):** El detector de activación por voz llama a `AppState.triggerVoiceEars(ttl: nil)` cuando se oye la palabra de activación, manteniendo `earBoostActive=true` mientras se captura el enunciado. Las orejas se agrandan (1,9x), obtienen orificios circulares para mejorar la legibilidad y luego se desactivan mediante `stopVoiceEars()` tras 1 s de silencio. Solo se activa desde el pipeline de voz dentro de la app.
- **Trabajando (agente en ejecución):** `AppState.isWorking=true` activa una micromoción de “cola/patas correteando”: movimiento de patas más rápido y ligero desplazamiento mientras el trabajo está en curso. Actualmente se activa/desactiva alrededor de las ejecuciones del agente de WebChat; añade el mismo control alrededor de otras tareas largas cuando las conectes.

Puntos de conexión

- Activación por voz: en tiempo de ejecución o pruebas llama a `AppState.triggerVoiceEars(ttl: nil)` al dispararse y a `stopVoiceEars()` tras 1 s de silencio para que coincida con la ventana de captura.
- Actividad del agente: establece `AppStateStore.shared.setWorking(true/false)` alrededor de los intervalos de trabajo (ya se hace en la llamada del agente de WebChat). Mantén los intervalos cortos y restablécelos en bloques `defer` para evitar animaciones bloqueadas.

Formas y tamaños

- Icono base dibujado en `CritterIconRenderer.makeIcon(blink:legWiggle:earWiggle:earScale:earHoles:)`.
- La escala de las orejas por defecto es `1.0`; el refuerzo de voz establece `earScale=1.9` y activa `earHoles=true` sin cambiar el marco general (imagen de plantilla de 18×18 pt renderizada en un búfer Retina de 36×36 px).
- El “correteo” usa movimiento de patas de hasta ~1.0 con un pequeño vaivén horizontal; es aditivo a cualquier balanceo inactivo existente.

Notas de comportamiento

- No hay conmutador externo de CLI/broker para orejas/trabajando; mantenlo interno a las propias señales de la app para evitar oscilaciones accidentales.
- Mantén los TTL cortos (&lt;10 s) para que el icono vuelva rápido al estado base si un trabajo se cuelga.