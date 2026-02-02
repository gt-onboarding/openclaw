---
title: Configuración estricta
summary: "Validación estricta de configuración + migraciones solo mediante doctor"
read_when:
  - Diseñar o implementar el comportamiento de validación de configuración
  - Trabajar en migraciones de configuración o flujos de trabajo de doctor
  - Gestionar esquemas de configuración de complementos o el control de la carga de complementos
---

<div id="strict-config-validation-doctor-only-migrations">
  # Validación estricta de la configuración (migraciones solo mediante `doctor`)
</div>

<div id="goals">
  ## Objetivos
</div>

- **Rechazar claves de configuración desconocidas en todos los niveles** (raíz + anidadas).
- **Rechazar la configuración de un complemento sin esquema**; no cargar ese complemento.
- **Eliminar la migración automática heredada al cargar**; las migraciones se ejecutan solo mediante doctor.
- **Ejecutar automáticamente doctor en modo dry-run al inicio**; si la configuración es inválida, bloquear los comandos no diagnósticos.

<div id="non-goals">
  ## No objetivos
</div>

- Compatibilidad con versiones anteriores durante la carga (las claves heredadas no se migran automáticamente).
- Descarte silencioso de claves no reconocidas.

<div id="strict-validation-rules">
  ## Reglas de validación estrictas
</div>

- La configuración debe coincidir exactamente con el esquema en todos los niveles.
- Las claves desconocidas son errores de validación (no se permiten claves adicionales ni en la raíz ni en niveles anidados).
- `plugins.entries.<id>.config` debe validarse según el esquema del complemento.
  - Si un complemento no tiene esquema, **rechaza la carga del complemento** y muestra un error claro.
- Las claves desconocidas `channels.<id>` son errores, a menos que un manifiesto de complemento declare ese ID de canal.
- Los manifiestos de complemento (`openclaw.plugin.json`) son obligatorios para todos los complementos.

<div id="plugin-schema-enforcement">
  ## Aplicación estricta del esquema de complementos
</div>

- Cada complemento proporciona un JSON Schema estricto para su configuración (definido en línea en el manifiesto).
- Flujo de carga del complemento:
  1) Resolver el manifiesto del complemento + esquema (`openclaw.plugin.json`).
  2) Validar la configuración contra el esquema.
  3) Si falta el esquema o la configuración no es válida: bloquear la carga del complemento y registrar el error.
- El mensaje de error incluye:
  - ID del complemento
  - Motivo (falta el esquema / configuración no válida)
  - Ruta(s) que no superaron la validación
- Los complementos deshabilitados conservan su configuración, pero Doctor + los registros muestran una advertencia.

<div id="doctor-flow">
  ## Flujo de Doctor
</div>

- Doctor se ejecuta **cada vez** que se carga la configuración (simulación *dry-run* de forma predeterminada).
- Si la configuración no es válida:
  - Imprime un resumen + errores con acciones concretas.
  - Indica que ejecutes: `openclaw doctor --fix`.
- `openclaw doctor --fix`:
  - Aplica migraciones.
  - Elimina claves desconocidas.
  - Escribe la configuración actualizada.

<div id="command-gating-when-config-is-invalid">
  ## Limitación de comandos (cuando la configuración no es válida)
</div>

Permitidos (solo diagnóstico):

- `openclaw doctor`
- `openclaw logs`
- `openclaw health`
- `openclaw help`
- `openclaw status`
- `openclaw gateway status`

Todo lo demás debe fallar con un error fatal: “Config invalid. Run `openclaw doctor --fix`.”

<div id="error-ux-format">
  ## Formato de UX de errores
</div>

- Encabezado de resumen único.
- Secciones agrupadas:
  - Claves desconocidas (rutas completas)
  - Claves heredadas / migraciones necesarias
  - Fallos de carga de complementos (ID del complemento + motivo + ruta)

<div id="implementation-touchpoints">
  ## Puntos de implementación
</div>

- `src/config/zod-schema.ts`: eliminar el passthrough en la raíz; objetos estrictos en todas partes.
- `src/config/zod-schema.providers.ts`: garantizar esquemas de canal estrictos.
- `src/config/validation.ts`: fallar si hay claves desconocidas; no aplicar migraciones heredadas.
- `src/config/io.ts`: eliminar auto-migraciones heredadas; ejecutar siempre doctor en modo dry-run.
- `src/config/legacy*.ts`: limitar su uso al doctor.
- `src/plugins/*`: agregar registro de esquemas + control de acceso.
- Gating de comandos de la CLI en `src/cli`.

<div id="tests">
  ## Pruebas
</div>

- Rechazo de claves desconocidas (en la raíz y anidadas).
- Falta el esquema del complemento → se bloquea la carga del complemento con un error claro.
- Configuración no válida → se bloquea el inicio del Gateway salvo para los comandos de diagnóstico.
- Dry-run automático de `doctor`; `doctor --fix` escribe la configuración corregida.