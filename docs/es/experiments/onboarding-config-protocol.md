---
title: Protocolo de configuración de onboarding
summary: "Notas del protocolo RPC para el asistente de onboarding y el esquema de configuración"
read_when: "Al modificar los pasos del asistente de onboarding o los endpoints del esquema de configuración"
---

<div id="onboarding-config-protocol">
  # Protocolo de onboarding y configuración
</div>

Propósito: puntos comunes de onboarding y configuración en la CLI, la app para macOS y la UI web.

<div id="components">
  ## Componentes
</div>

- Motor del asistente (sesión compartida + prompts + estado de onboarding).
- El onboarding en la CLI utiliza el mismo flujo de asistente que los clientes de la UI.
- Gateway RPC expone endpoints del asistente y del esquema de configuración.
- El onboarding en macOS utiliza el modelo de pasos del asistente.
- La UI web renderiza formularios de configuración a partir de JSON Schema + sugerencias de UI.

<div id="gateway-rpc">
  ## RPC del Gateway
</div>

- `wizard.start` parámetros: `{ mode?: "local"|"remote", workspace?: string }`
- `wizard.next` parámetros: `{ sessionId, answer?: { stepId, value? } }`
- `wizard.cancel` parámetros: `{ sessionId }`
- `wizard.status` parámetros: `{ sessionId }`
- `config.schema` parámetros: `{}`

Respuestas (estructura)

- Asistente (Wizard): `{ sessionId, done, step?, status?, error? }`
- Esquema de configuración (Config schema): `{ schema, uiHints, version, generatedAt }`

<div id="ui-hints">
  ## Indicaciones para la UI
</div>

- `uiHints` con clave por ruta; metadatos opcionales (label/help/group/order/advanced/sensitive/placeholder).
- Los campos sensibles se muestran como entradas de contraseña; no hay capa adicional de ocultamiento.
- Los nodos de esquema no admitidos recurren al editor JSON sin procesar.

<div id="notes">
  ## Notas
</div>

- Este documento es el único punto de referencia para registrar las refactorizaciones del protocolo de onboarding/configuración.