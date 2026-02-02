---
title: Zai
summary: "Usar Z.AI (modelos GLM) con OpenClaw"
read_when:
  - Quieres usar modelos Z.AI / GLM en OpenClaw
  - Necesitas configurar ZAI_API_KEY de forma sencilla
---

<div id="zai">
  # Z.AI
</div>

Z.AI es la plataforma de API para modelos **GLM**. Ofrece API REST para GLM y utiliza claves de API
para la autenticación. Genera tu clave de API en la consola de Z.AI. OpenClaw utiliza el proveedor `zai`
con una clave de API de Z.AI.

<div id="cli-setup">
  ## Configuración de la CLI
</div>

```bash
openclaw onboard --auth-choice zai-api-key
# o de forma no interactiva
openclaw onboard --zai-api-key "$ZAI_API_KEY"
```

<div id="config-snippet">
  ## Ejemplo de configuración
</div>

```json5
{
  env: { ZAI_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "zai/glm-4.7" } } }
}
```

<div id="notes">
  ## Notas
</div>

* Los modelos GLM están disponibles como `zai/<model>` (por ejemplo: `zai/glm-4.7`).
* Consulta [/providers/glm](/es/providers/glm) para la descripción general de la familia de modelos.
* Z.AI usa autenticación de tipo Bearer con tu api key.