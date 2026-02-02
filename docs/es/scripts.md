---
title: Scripts
summary: "Scripts del repositorio: propósito, ámbito y notas de seguridad"
read_when:
  - Al ejecutar scripts desde el repositorio
  - Al agregar o modificar scripts en ./scripts
---

<div id="scripts">
  # Scripts
</div>

El directorio `scripts/` contiene scripts auxiliares para flujos de trabajo locales y tareas de operación.
Utilízalos cuando una tarea esté claramente asociada a un script; de lo contrario, prioriza la CLI.

<div id="conventions">
  ## Convenciones
</div>

* Los scripts son **opcionales** a menos que se mencionen en la documentación o en las listas de comprobación de lanzamiento.
* Da preferencia a la CLI cuando exista (ejemplo: la monitorización de autenticación usa `openclaw models status --check`).
* Asume que los scripts son específicos del host; léelos antes de ejecutarlos en una máquina nueva.

<div id="git-hooks">
  ## Git hooks
</div>

* `scripts/setup-git-hooks.js`: configuración best-effort de `core.hooksPath` cuando se ejecuta dentro de un repositorio Git.
* `scripts/format-staged.js`: formateador de pre-commit para archivos en estado *staged* en `src/` y `test/`.

<div id="auth-monitoring-scripts">
  ## Scripts de monitorización de autenticación
</div>

La documentación de los scripts de monitorización de autenticación se encuentra aquí:
[/automation/auth-monitoring](/es/automation/auth-monitoring)

<div id="when-adding-scripts">
  ## Al agregar scripts
</div>

* Mantén los scripts bien delimitados y documentados.
* Agrega una breve entrada en la documentación correspondiente (o crea una si no existe).