---
title: Manifiesto
summary: "Manifiesto del complemento + requisitos del esquema JSON (validación estricta de la configuración)"
read_when:
  - Estás desarrollando un complemento de OpenClaw
  - Necesitas publicar un esquema de configuración de un complemento o depurar errores de validación de complementos
---

<div id="plugin-manifest-openclawpluginjson">
  # Manifiesto de complemento (openclaw.plugin.json)
</div>

Cada complemento **debe** incluir un archivo `openclaw.plugin.json` en la **raíz del complemento**.
OpenClaw utiliza este manifiesto para validar la configuración **sin ejecutar el código del complemento**.
Los manifiestos faltantes o no válidos se tratan como errores del complemento y bloquean
la validación de la configuración.

Consulta la guía completa del sistema de complementos: [Plugins](/es/plugin).

<div id="required-fields">
  ## Campos obligatorios
</div>

```json
{
  "id": "voice-call",
  "configSchema": {
    "type": "object",
    "additionalProperties": false,
    "properties": {}
  }
}
```

Claves obligatorias:

* `id` (string): ID canónico del complemento.
* `configSchema` (object): JSON Schema para la configuración del complemento (en línea).

Claves opcionales:

* `kind` (string): tipo de complemento (ejemplo: `"memory"`).
* `channels` (array): ID de canal registrados por este complemento (ejemplo: `["matrix"]`).
* `providers` (array): ID de proveedor registrados por este complemento.
* `skills` (array): directorios de habilidades que se cargarán (relativos a la raíz del complemento).
* `name` (string): nombre visible del complemento.
* `description` (string): breve resumen del complemento.
* `uiHints` (object): etiquetas de campos de configuración/placeholders/indicadores de sensibilidad para el renderizado en la UI.
* `version` (string): versión del complemento (informativa).

<div id="json-schema-requirements">
  ## Requisitos de JSON Schema
</div>

* **Cada complemento debe incluir un JSON Schema**, incluso si no acepta ninguna configuración.
* Un esquema vacío es aceptable (por ejemplo, `{ "type": "object", "additionalProperties": false }`).
* Los esquemas se validan al leer o escribir la configuración, no en tiempo de ejecución.

<div id="validation-behavior">
  ## Comportamiento de validación
</div>

* Las claves `channels.*` desconocidas se consideran **errores**, a menos que el ID del canal esté declarado por un manifiesto de complemento.
* `plugins.entries.<id>`, `plugins.allow`, `plugins.deny` y `plugins.slots.*`
  deben hacer referencia a IDs de complemento **detectables**. Los IDs desconocidos son **errores**.
* Si un complemento está instalado pero su manifiesto o esquema está dañado o falta,
  la validación falla y Doctor reporta el error del complemento.
* Si existe configuración del complemento pero el complemento está **deshabilitado**, la configuración se conserva y
  se muestra una **advertencia** en Doctor + logs.

<div id="notes">
  ## Notas
</div>

* El manifiesto es **obligatorio para todos los complementos**, incluidas las cargas desde el sistema de archivos local.
* En tiempo de ejecución, el módulo del complemento sigue cargándose por separado; el manifiesto es solo para
  descubrimiento y validación.
* Si tu complemento depende de módulos nativos, documenta los pasos de compilación y cualquier requisito de lista de permitidos del gestor de paquetes (por ejemplo, pnpm `allow-build-scripts`
  * `pnpm rebuild &lt;package&gt;`).