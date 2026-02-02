---
title: Actualización
summary: "Referencia de la CLI de `openclaw update` (actualización del código fuente relativamente segura + reinicio automático del Gateway)"
read_when:
  - Quieres actualizar de forma segura un checkout del código fuente
  - Necesitas entender el comportamiento de la forma abreviada `--update`
---

<div id="openclaw-update">
  # `openclaw update`
</div>

Actualiza OpenClaw de forma segura y cambia entre los canales stable/beta/dev.

Si lo instalaste mediante **npm/pnpm** (instalación global, sin metadatos de git), las actualizaciones se gestionan mediante el flujo del gestor de paquetes descrito en [Actualización](/es/install/updating).

<div id="usage">
  ## Uso
</div>

```bash
openclaw update
openclaw update status
openclaw update wizard
openclaw update --channel beta
openclaw update --channel dev
openclaw update --tag beta
openclaw update --no-restart
openclaw update --json
openclaw --update
```

<div id="options">
  ## Opciones
</div>

* `--no-restart`: omite reiniciar el servicio Gateway después de una actualización correcta.
* `--channel <stable|beta|dev>`: establece el canal de actualización (git + npm; se persiste en la configuración).
* `--tag <dist-tag|version>`: anula el npm dist-tag o la versión solo para esta actualización.
* `--json`: imprime JSON `UpdateRunResult` legible por máquina.
* `--timeout <seconds>`: tiempo de espera por paso (el valor predeterminado es 1200s).

Nota: los retrocesos de versión requieren confirmación porque las versiones anteriores pueden romper la configuración.

<div id="update-status">
  ## `update status`
</div>

Muestra el canal de actualización activo y la etiqueta/rama/SHA de git (para instalaciones desde el código fuente), junto con la disponibilidad de actualizaciones.

```bash
openclaw update status
openclaw update status --json
openclaw update status --timeout 10
```

Opciones:

* `--json`: imprime JSON de estado en un formato legible por máquina.
* `--timeout <seconds>`: tiempo de espera para las comprobaciones (el valor predeterminado es 3 s).

<div id="update-wizard">
  ## `update wizard`
</div>

Flujo interactivo para elegir un canal de actualización y confirmar si quieres reiniciar el Gateway
tras la actualización (por defecto se reinicia). Si seleccionas `dev` sin un checkout de Git,
te ofrece crear uno.

<div id="what-it-does">
  ## Qué hace
</div>

Cuando cambias de canal explícitamente (`--channel ...`), OpenClaw también mantiene coherente
el método de instalación:

* `dev` → asegura un checkout de git (predeterminado: `~/openclaw`, puedes sobrescribirlo con `OPENCLAW_GIT_DIR`),
  lo actualiza e instala la CLI global desde ese checkout.
* `stable`/`beta` → instala desde npm usando la etiqueta `dist-tag` correspondiente.

<div id="git-checkout-flow">
  ## Flujo de checkout de Git
</div>

Canales:

* `stable`: hace checkout de la última etiqueta no beta, luego build + doctor.
* `beta`: hace checkout de la última etiqueta `-beta`, luego build + doctor.
* `dev`: hace checkout de `main`, luego fetch + rebase.

A grandes rasgos:

1. Requiere un worktree limpio (sin cambios sin confirmar).
2. Cambia al canal seleccionado (etiqueta o rama).
3. Hace fetch desde upstream (solo dev).
4. Solo dev: realiza un lint de preflight + un build de TypeScript en un worktree temporal; si el HEAD falla, retrocede hasta 10 commits para encontrar el build limpio más reciente.
5. Hace rebase sobre el commit seleccionado (solo dev).
6. Instala dependencias (se prefiere pnpm; npm como alternativa).
7. Compila + construye la Control UI.
8. Ejecuta `openclaw doctor` como comprobación final de “actualización segura”.
9. Sincroniza los complementos con el canal activo (dev usa extensiones integradas; stable/beta usa npm) y actualiza los complementos instalados vía npm.

<div id="update-shorthand">
  ## Forma abreviada `--update`
</div>

`openclaw --update` se expande a `openclaw update` (útil para shells y scripts de inicio).

<div id="see-also">
  ## Ver también
</div>

* `openclaw doctor` (ofrece ejecutar primero la actualización en checkouts de Git)
* [Canales de desarrollo](/es/install/development-channels)
* [Actualizaciones](/es/install/updating)
* [Referencia de la CLI](/es/cli)