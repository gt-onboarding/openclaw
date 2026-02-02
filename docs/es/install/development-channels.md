---
title: Canales de desarrollo
summary: "Canales estable, beta y de desarrollo: semántica, cambio de canal y etiquetado"
read_when:
  - Si quieres cambiar entre estable/beta/dev
  - Si estás etiquetando o publicando versiones preliminares
---

<div id="development-channels">
  # Canales de desarrollo
</div>

Última actualización: 2026-01-21

OpenClaw se distribuye en tres canales de actualización:

- **stable**: npm dist-tag `latest`.
- **beta**: npm dist-tag `beta` (builds en prueba).
- **dev**: cabeza en movimiento de `main` (git). npm dist-tag: `dev` (cuando se publica).

Publicamos builds en **beta**, las probamos y luego **promocionamos un build validado a `latest`**
sin cambiar el número de versión: los dist-tags son la fuente de verdad para las instalaciones de npm.

<div id="switching-channels">
  ## Cambio de canales
</div>

Git checkout:

```bash
openclaw update --channel stable
openclaw update --channel beta
openclaw update --channel dev
```

* `stable`/`beta` cambia a la última etiqueta coincidente (a menudo la misma etiqueta).
* `dev` cambia a `main` y hace rebase sobre el upstream.

Instalación global con npm/pnpm:

```bash
openclaw update --channel stable
openclaw update --channel beta
openclaw update --channel dev
```

Esto se actualiza mediante la dist-tag de npm correspondiente (`latest`, `beta`, `dev`).

Cuando cambias de canal **explícitamente** con `--channel`, OpenClaw también adapta
el método de instalación:

* `dev` garantiza que exista un checkout de git (por defecto `~/openclaw`, se puede sobrescribir con `OPENCLAW_GIT_DIR`),
  lo actualiza e instala la CLI global desde ese checkout.
* `stable`/`beta` se instalan desde npm usando la dist-tag correspondiente.

Consejo: si quieres `stable` y `dev` en paralelo, mantén dos clones y apunta tu Gateway al clon estable.


<div id="plugins-and-channels">
  ## Complementos y canales
</div>

Cuando cambias de canal con `openclaw update`, OpenClaw también sincroniza las fuentes de los complementos:

- `dev` prefiere los complementos incluidos en el checkout de Git.
- `stable` y `beta` restauran los paquetes de complementos instalados mediante npm.

<div id="tagging-best-practices">
  ## Mejores prácticas de etiquetado
</div>

- Etiqueta las versiones en las que quieras que aterricen los checkouts de Git (`vYYYY.M.D` o `vYYYY.M.D-<patch>`).
- Mantén las etiquetas inmutables: nunca muevas ni reutilices una etiqueta.
- Los dist-tags de npm siguen siendo la fuente de verdad para las instalaciones con npm:
  - `latest` → estable
  - `beta` → versión candidata
  - `dev` → snapshot de la rama main (opcional)

<div id="macos-app-availability">
  ## Disponibilidad de la app para macOS
</div>

Las compilaciones beta y de desarrollo **pueden** no incluir una versión de la app para macOS. No es un problema:

- La etiqueta de git y el dist-tag de npm se pueden seguir publicando.
- Indica “no hay build de macOS para esta beta” en las notas de la versión o en el changelog.