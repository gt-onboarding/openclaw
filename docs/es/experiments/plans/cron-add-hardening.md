---
title: Refuerzo de seguridad de cron.add
summary: "Reforzar la seguridad del manejo de entrada de cron.add, alinear los esquemas y mejorar las herramientas de UI y de agente para cron"
owner: "openclaw"
status: "completado"
last_updated: "2026-01-05"
---

<div id="cron-add-hardening-schema-alignment">
  # Endurecimiento de cron add y alineación de esquemas
</div>

<div id="context">
  ## Contexto
</div>

Los registros recientes del Gateway muestran fallos repetidos de `cron.add` con parámetros inválidos (faltan `sessionTarget`, `wakeMode`, `payload` y un `schedule` malformado). Esto indica que al menos un cliente (probablemente la ruta de llamadas de herramienta del agente) está enviando payloads de jobs encapsulados o solo parcialmente especificados. Por separado, existe un desajuste entre los enums del proveedor de cron en TypeScript, el esquema del Gateway, los flags de la CLI y los tipos de formulario de la UI, además de una discrepancia en la UI para `cron.status` (espera `jobCount` mientras que el Gateway devuelve `jobs`).

<div id="goals">
  ## Objetivos
</div>

* Detener el spam de INVALID&#95;REQUEST de `cron.add` normalizando los payloads de wrapper comunes e infiriendo los campos `kind` que falten.
* Alinear las listas de proveedores cron en todo el esquema del Gateway, los tipos de cron, la documentación de la CLI y los formularios de la UI.
* Hacer explícito el esquema de la herramienta de cron del agente para que el LLM produzca payloads de tareas correctos.
* Corregir la visualización del recuento de tareas en la vista de estado de cron del Control UI.
* Añadir pruebas para cubrir la normalización y el comportamiento de la herramienta.

<div id="non-goals">
  ## Fuera de alcance
</div>

* Cambiar la semántica de programación con cron o el comportamiento de ejecución de tareas.
* Añadir nuevos tipos de programación o cambios en el análisis de expresiones cron.
* Renovar la UI/UX de cron más allá de las correcciones necesarias en los campos.

<div id="findings-current-gaps">
  ## Hallazgos (lagunas actuales)
</div>

* `CronPayloadSchema` en el Gateway excluye `signal` + `imessage`, mientras que los tipos de TS los incluyen.
* Control UI CronStatus espera `jobCount`, pero el Gateway devuelve `jobs`.
* El esquema de la herramienta cron del agente permite objetos `job` arbitrarios, lo que permite entradas malformadas.
* El Gateway valida estrictamente `cron.add` sin normalización, por lo que los payloads encapsulados fallan.

<div id="what-changed">
  ## Qué cambió
</div>

* `cron.add` y `cron.update` ahora normalizan estructuras de envoltura comunes e infieren los campos `kind` que faltan.
* El esquema de la herramienta cron del Agente coincide con el esquema del Gateway, lo que reduce los payloads no válidos.
* Los enums de proveedor están alineados en Gateway, CLI, UI y el selector de macOS.
* Control UI usa el campo `jobs` del Gateway para mostrar el estado.

<div id="current-behavior">
  ## Comportamiento actual
</div>

* **Normalización:** las cargas `data`/`job` encapsuladas se desempaquetan; `schedule.kind` y `payload.kind` se infieren cuando es seguro.
* **Valores predeterminados:** se aplican valores seguros para `wakeMode` y `sessionTarget` cuando no están definidos.
* **Proveedores:** Discord/Slack/Signal/iMessage ahora se muestran de forma coherente en CLI/UI.

Consulta [Tareas cron](/es/automation/cron-jobs) para ver la estructura normalizada y ejemplos.

<div id="verification">
  ## Verificación
</div>

* Observa los registros del Gateway para comprobar que se reduzcan los errores `cron.add` INVALID&#95;REQUEST.
* Confirma que el estado de cron en el Control UI muestre un recuento de tareas tras actualizar la página.

<div id="optional-follow-ups">
  ## Seguimientos opcionales
</div>

* Verificación rápida manual en Control UI: añade una tarea cron por proveedor y comprueba el número de tareas de estado.

<div id="open-questions">
  ## Preguntas abiertas
</div>

* ¿Debería `cron.add` aceptar un `state` explícito enviado por los clientes (actualmente no permitido por el esquema)?
* ¿Deberíamos permitir `webchat` como proveedor de entrega explícito (actualmente se filtra en la resolución de entrega)?