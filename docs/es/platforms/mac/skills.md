---
title: Habilidades
summary: "UI de configuración de habilidades de macOS y estado proporcionado por el Gateway"
read_when:
  - Actualizar la UI de configuración de habilidades de macOS
  - Cambiar las restricciones de acceso de las habilidades o su comportamiento de instalación
---

<div id="skills-macos">
  # Habilidades (macOS)
</div>

La aplicación de macOS expone las habilidades de OpenClaw a través del Gateway; no procesa habilidades de forma local.

<div id="data-source">
  ## Fuente de datos
</div>

- `skills.status` (Gateway) devuelve todas las habilidades, además de su elegibilidad y los requisitos que falten
  (incluidos los bloqueos por lista de permitidos para las habilidades empaquetadas).
- Los requisitos se obtienen de `metadata.openclaw.requires` en cada `SKILL.md`.

<div id="install-actions">
  ## Acciones de instalación
</div>

- `metadata.openclaw.install` define opciones de instalación (brew/node/go/uv).
- La app llama a `skills.install` para ejecutar instaladores en el host del Gateway.
- El Gateway muestra solo un instalador preferido cuando se proporcionan varios
  (brew cuando está disponible; en caso contrario, el gestor de Node desde `skills.install`; si no, npm por defecto).

<div id="envapi-keys">
  ## Claves de entorno y API
</div>

- La aplicación almacena las claves en `~/.openclaw/openclaw.json` bajo `skills.entries.<skillKey>`.
- `skills.update` modifica `enabled`, `apiKey` y `env`.

<div id="remote-mode">
  ## Modo remoto
</div>

- La instalación y las actualizaciones de configuración se realizan en el host donde se ejecuta el Gateway (no en tu Mac local).