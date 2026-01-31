---
title: Barra de menús
summary: "Lógica del estado de la barra de menús y qué se expone a los usuarios"
read_when:
  - Ajustar la UI de la barra de menús de macOS o la lógica de estado
---

<div id="menu-bar-status-logic">
  # Lógica del estado de la barra de menús
</div>

<div id="what-is-shown">
  ## Qué se muestra
</div>

- Mostramos el estado de trabajo actual del agente en el icono de la barra de menús y en la primera fila de estado del menú.
- El estado de salud se oculta mientras el trabajo está activo; vuelve a mostrarse cuando todas las sesiones están inactivas.
- El bloque “Nodes” en el menú enumera únicamente **dispositivos** (nodos emparejados mediante `node.list`), no entradas de cliente/presencia.
- Una sección “Usage” aparece en “Context” cuando hay instantáneas disponibles del uso del proveedor.

<div id="state-model">
  ## Modelo de estado
</div>

- Sesiones: los eventos llegan con `runId` (por ejecución) más `sessionKey` en el payload. La sesión “principal” es la clave `main`; si falta, recurrimos a la sesión actualizada más recientemente.
- Prioridad: la sesión principal siempre gana. Si la principal está activa, su estado se muestra inmediatamente. Si la principal está inactiva, se muestra la sesión no principal más recientemente activa. No alternamos en medio de una actividad; solo cambiamos cuando la sesión actual pasa a estar inactiva o la principal se vuelve activa.
- Tipos de actividad:
  - `job`: ejecución de comando de alto nivel (`state: started|streaming|done|error`).
  - `tool`: `phase: start|result` con `toolName` y `meta/args`.

<div id="iconstate-enum-swift">
  ## Enumeración IconState (Swift)
</div>

- `idle`
- `workingMain(ActivityKind)`
- `workingOther(ActivityKind)`
- `overridden(ActivityKind)` (forzado para depuración)

<div id="activitykind-glyph">
  ### ActivityKind → glifo
</div>

- `exec` → 💻
- `read` → 📄
- `write` → ✍️
- `edit` → 📝
- `attach` → 📎
- por defecto → 🛠️

<div id="visual-mapping">
  ### Mapeo visual
</div>

- `idle`: criatura normal.
- `workingMain`: insignia con glifo, tinte completo, animación de patas “trabajando”.
- `workingOther`: insignia con glifo, tinte atenuado, sin animación de correteo.
- `overridden`: usa el glifo y tinte elegidos independientemente de la actividad.

<div id="status-row-text-menu">
  ## Texto de la fila de estado (menú)
</div>

- Mientras haya trabajo en curso: `<Session role> · <activity label>`
  - Ejemplos: `Main · exec: pnpm test`, `Other · read: apps/macos/Sources/OpenClaw/AppState.swift`.
- Cuando esté inactivo: vuelve al resumen de estado.

<div id="event-ingestion">
  ## Ingesta de eventos
</div>

- Origen: eventos `agent` del canal de control (`ControlChannel.handleAgentEvent`).
- Campos interpretados:
  - `stream: "job"` con `data.state` para inicio/detención.
  - `stream: "tool"` con `data.phase`, `name`, `meta`/`args` opcionales.
- Etiquetas:
  - `exec`: primera línea de `args.command`.
  - `read`/`write`: ruta abreviada.
  - `edit`: ruta más el tipo de cambio inferido a partir de `meta`/recuentos de diferencias (diff).
  - valor de respaldo: nombre de la herramienta.

<div id="debug-override">
  ## Anulación de depuración
</div>

- Settings ▸ Debug ▸ selector “Icon override”:
  - `System (auto)` (predeterminado)
  - `Working: main` (por tipo de herramienta)
  - `Working: other` (por tipo de herramienta)
  - `Idle`
- Se almacena mediante `@AppStorage("iconOverride")`; se asigna a `IconState.overridden`.

<div id="testing-checklist">
  ## Lista de verificación de pruebas
</div>

- Ejecuta un trabajo de la sesión principal: verifica que el icono cambie inmediatamente y que la fila de estado muestre la etiqueta principal.
- Ejecuta un trabajo de una sesión no principal mientras la principal está inactiva: el icono/estado muestra la no principal y permanece estable hasta que finaliza.
- Inicia la principal mientras otra sesión está activa: el icono cambia a la principal al instante.
- Ráfagas rápidas de herramientas: verifica que la insignia no parpadee (período de gracia de TTL en los resultados de herramientas).
- La fila de estado de salud vuelve a aparecer una vez que todas las sesiones están inactivas.