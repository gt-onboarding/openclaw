---
title: Higiene de transcripción
summary: "Referencia: reglas específicas de cada proveedor para la sanitización y reparación de transcripciones"
read_when:
  - Estás depurando rechazos de solicitudes del proveedor relacionados con la estructura de la transcripción
  - Estás cambiando la lógica de sanitización de transcripciones o de reparación de llamadas a herramientas
  - Estás investigando desajustes de identificadores de llamadas a herramientas entre proveedores
---

<div id="transcript-hygiene-provider-fixups">
  # Higiene de transcripciones (ajustes del proveedor)
</div>

Este documento describe **correcciones específicas del proveedor** aplicadas a las transcripciones antes de una ejecución
(al construir el contexto del modelo). Estas son correcciones **en memoria** utilizadas para cumplir requisitos
estrictos del proveedor. **No** reescriben la transcripción JSONL almacenada en disco.

El alcance incluye:

* Saneamiento de ID de llamadas de herramientas
* Corrección del emparejamiento de resultados de herramientas
* Validación / ordenación de turnos
* Limpieza de firmas de pensamiento
* Saneamiento de payloads de imagen

Si necesitas detalles sobre el almacenamiento de transcripciones, consulta:

* [/reference/session-management-compaction](/es/reference/session-management-compaction)

***

<div id="where-this-runs">
  ## Dónde se ejecuta esto
</div>

Toda la higiene de las transcripciones se centraliza en el runner integrado:

* Selección de política: `src/agents/transcript-policy.ts`
* Aplicación de sanitización/reparación: `sanitizeSessionHistory` en `src/agents/pi-embedded-runner/google.ts`

La política usa `provider`, `modelApi` y `modelId` para decidir qué aplicar.

***

<div id="global-rule-image-sanitization">
  ## Regla global: sanitización de imágenes
</div>

Las cargas de imágenes siempre se sanitizan para evitar que el proveedor las rechace debido a límites de tamaño (reducción de escala/recompresión de imágenes base64 demasiado grandes).

Implementación:

* `sanitizeSessionMessagesImages` en `src/agents/pi-embedded-helpers/images.ts`
* `sanitizeContentBlocksImages` en `src/agents/tool-images.ts`

***

<div id="provider-matrix-current-behavior">
  ## Matriz de proveedores (comportamiento actual)
</div>

**OpenAI / OpenAI Codex**

* Solo saneamiento de imágenes.
* Al cambiar de modelo a OpenAI Responses/Codex, descartar firmas de razonamiento huérfanas (elementos de razonamiento independientes sin un bloque de contenido siguiente).
* Sin saneamiento de ID de llamada de herramienta.
* Sin reparación de emparejamiento de resultados de herramientas.
* Sin validación ni reordenamiento de turnos.
* Sin resultados de herramientas sintéticos.
* Sin eliminación de firmas de razonamiento.

**Google (Generative AI / Gemini CLI / Antigravity)**

* Saneamiento de ID de llamada de herramienta: alfanumérico estricto.
* Reparación de emparejamiento de resultados de herramientas y resultados de herramientas sintéticos.
* Validación de turnos (alternancia de turnos al estilo Gemini).
* Corrección del orden de turnos de Google (anteponer un pequeño mensaje de arranque de usuario si el historial comienza con assistant).
* Antigravity Claude: normalizar firmas de razonamiento; descartar bloques de razonamiento sin firmar.

**Anthropic / Minimax (compatible con Anthropic)**

* Reparación de emparejamiento de resultados de herramientas y resultados de herramientas sintéticos.
* Validación de turnos (fusionar turnos de usuario consecutivos para cumplir la alternancia estricta).

**Mistral (incluida la detección basada en model-id)**

* Saneamiento de ID de llamada de herramienta: strict9 (alfanumérico de longitud 9).

**OpenRouter Gemini**

* Limpieza de firmas de razonamiento: eliminar valores `thought_signature` que no sean base64 (mantener base64).

**Todo lo demás**

* Solo saneamiento de imágenes.

***

<div id="historical-behavior-pre-2026122">
  ## Comportamiento histórico (antes de 2026.1.22)
</div>

Antes de la versión 2026.1.22, OpenClaw aplicaba varias capas de higiene sobre el transcript:

* Una **extensión transcript-sanitize** se ejecutaba en cada construcción de contexto y podía:
  * Reparar el emparejamiento uso/resultado de herramientas.
  * Sanear los ID de llamadas de herramienta (incluido un modo no estricto que preservaba `_`/`-`).
* El runner también realizaba un saneamiento específico del proveedor, lo que duplicaba trabajo.
* Se producían mutaciones adicionales fuera de la política del proveedor, entre ellas:
  * Eliminar etiquetas `<final>` del texto del asistente antes de persistirlo.
  * Descartar turnos de error vacíos del asistente.
  * Recortar contenido del asistente después de llamadas de herramienta.

Esta complejidad provocó regresiones entre proveedores (en particular el emparejamiento
`call_id|fc_id` de `openai-responses`). La limpieza de 2026.1.22 eliminó la extensión, centralizó
la lógica en el runner y dejó OpenAI en modo **no-touch** (sin modificaciones) más allá del saneamiento de imágenes.