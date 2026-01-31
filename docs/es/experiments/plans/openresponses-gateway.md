---
title: Gateway OpenResponses
summary: "Plan: añadir el endpoint /v1/responses de OpenResponses y depreciar limpiamente la funcionalidad de chat completions"
owner: "openclaw"
status: "draft"
last_updated: "2026-01-19"
---

<div id="openresponses-gateway-integration-plan">
  # Plan de integración del Gateway OpenResponses
</div>

<div id="context">
  ## Contexto
</div>

Actualmente, el Gateway de OpenClaw expone un endpoint mínimo de Chat Completions compatible con OpenAI en
`/v1/chat/completions` (consulta [OpenAI Chat Completions](/es/gateway/openai-http-api)).

Open Responses es un estándar de inferencia abierto basado en la API OpenAI Responses. Está diseñado
para flujos de trabajo basados en agentes y utiliza entradas basadas en elementos junto con eventos de transmisión semántica. La especificación de OpenResponses
define `/v1/responses`, no `/v1/chat/completions`.

<div id="goals">
  ## Objetivos
</div>

* Añadir un endpoint `/v1/responses` que se adhiera a la semántica de OpenResponses.
* Mantener Chat Completions como una capa de compatibilidad que sea fácil de desactivar y eventualmente eliminar.
* Estandarizar la validación y el análisis sintáctico con esquemas aislados y reutilizables.

<div id="non-goals">
  ## No objetivos
</div>

* Alcanzar paridad completa de funcionalidades con OpenResponses en la primera iteración (imágenes, archivos, herramientas alojadas).
* Reemplazar la lógica interna de ejecución de agentes o la orquestación de herramientas.
* Cambiar el comportamiento existente de `/v1/chat/completions` durante la primera fase.

<div id="research-summary">
  ## Resumen de la investigación
</div>

Fuentes: OpenResponses OpenAPI, sitio de especificación de OpenResponses y la publicación del blog de Hugging Face.

Puntos clave extraídos:

* `POST /v1/responses` acepta campos de `CreateResponseBody` como `model`, `input` (cadena o
  `ItemParam[]`), `instructions`, `tools`, `tool_choice`, `stream`, `max_output_tokens` y
  `max_tool_calls`.
* `ItemParam` es una unión discriminada de:
  * elementos `message` con roles `system`, `developer`, `user`, `assistant`
  * `function_call` y `function_call_output`
  * `reasoning`
  * `item_reference`
* Las respuestas correctas devuelven un `ResponseResource` con `object: "response"`, `status` y
  elementos `output`.
* El streaming utiliza eventos semánticos como:
  * `response.created`, `response.in_progress`, `response.completed`, `response.failed`
  * `response.output_item.added`, `response.output_item.done`
  * `response.content_part.added`, `response.content_part.done`
  * `response.output_text.delta`, `response.output_text.done`
* La especificación requiere:
  * `Content-Type: text/event-stream`
  * `event:` debe coincidir con el campo `type` en el JSON
  * el evento terminal debe ser literalmente `[DONE]`
* Los elementos de razonamiento pueden exponer `content`, `encrypted_content` y `summary`.
* Los ejemplos de HF incluyen `OpenResponses-Version: latest` en las solicitudes (encabezado opcional).

<div id="proposed-architecture">
  ## Arquitectura propuesta
</div>

* Agregar `src/gateway/open-responses.schema.ts` que contenga solo esquemas Zod (sin imports de Gateway).
* Agregar `src/gateway/openresponses-http.ts` (u `open-responses-http.ts`) para `/v1/responses`.
* Mantener `src/gateway/openai-http.ts` intacto como adaptador de compatibilidad heredado.
* Agregar la configuración `gateway.http.endpoints.responses.enabled` (por defecto `false`).
* Mantener `gateway.http.endpoints.chatCompletions.enabled` independiente; permitir que ambos endpoints se activen por separado.
* Emitir una advertencia en el arranque cuando Chat Completions esté habilitado para señalar que es una funcionalidad heredada.

<div id="deprecation-path-for-chat-completions">
  ## Ruta de deprecación de Chat Completions
</div>

* Mantén límites estrictos entre módulos: no compartas tipos de esquema entre Responses y Chat Completions.
* Haz que Chat Completions sea opcional mediante configuración para que pueda deshabilitarse sin cambios de código.
* Actualiza la documentación para marcar Chat Completions como obsoleto una vez que `/v1/responses` sea estable.
* Paso futuro opcional: redirigir las solicitudes de Chat Completions al manejador de Responses para una ruta de eliminación más simple.

<div id="phase-1-support-subset">
  ## Subconjunto de soporte de la Fase 1
</div>

* Aceptar `input` como string o `ItemParam[]` con roles de mensaje y `function_call_output`.
* Extraer mensajes de sistema y de desarrollador en `extraSystemPrompt`.
* Usar el `user` o `function_call_output` más reciente como el mensaje actual para ejecuciones de agentes.
* Rechazar tipos de contenido no admitidos (imagen/archivo) con `invalid_request_error`.
* Devolver un único mensaje de assistant con contenido `output_text`.
* Devolver `usage` con valores en cero hasta que se implemente el cómputo de tokens.

<div id="validation-strategy-no-sdk">
  ## Estrategia de validación (sin SDK)
</div>

* Implementa esquemas de Zod para el subconjunto compatible de:
  * `CreateResponseBody`
  * `ItemParam` + uniones de partes de contenido de mensaje
  * `ResponseResource`
  * Estructuras de eventos de streaming usadas por Gateway
* Mantén los esquemas en un único módulo aislado para evitar desviaciones y permitir la futura generación de código.

<div id="streaming-implementation-phase-1">
  ## Implementación de streaming (Fase 1)
</div>

* Líneas SSE con ambos `event:` y `data:`.
* Secuencia requerida (mínimo viable):
  * `response.created`
  * `response.output_item.added`
  * `response.content_part.added`
  * `response.output_text.delta` (repetir según sea necesario)
  * `response.output_text.done`
  * `response.content_part.done`
  * `response.completed`
  * `[DONE]`

<div id="tests-and-verification-plan">
  ## Plan de pruebas y verificación
</div>

* Añadir cobertura e2e para `/v1/responses`:
  * Autenticación requerida
  * Estructura de respuesta no en streaming
  * Orden de eventos de streaming y `[DONE]`
  * Enrutamiento de sesión con cabeceras y `user`
* Mantener `src/gateway/openai-http.e2e.test.ts` sin cambios.
* Manual: ejecutar curl a `/v1/responses` con `stream: true` y verificar el orden de los eventos y el
  `[DONE]` final.

<div id="doc-updates-follow-up">
  ## Actualizaciones de la documentación (seguimiento)
</div>

* Añadir una nueva página de documentación para el uso y los ejemplos de `/v1/responses`.
* Actualizar `/gateway/openai-http-api` con una nota de obsolescencia y un enlace a `/v1/responses`.