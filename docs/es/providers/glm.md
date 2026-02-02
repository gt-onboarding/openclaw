---
title: GLM
summary: "Visión general de la familia de modelos GLM + cómo usarla en OpenClaw"
read_when:
  - Quieres usar modelos GLM en OpenClaw
  - Necesitas conocer la convención de nombres de los modelos y su configuración
---

<div id="glm-models">
  # Modelos GLM
</div>

GLM es una **familia de modelos** (no una empresa) disponible a través de la plataforma Z.AI. En OpenClaw, se accede a los modelos GLM a través del proveedor `zai` y de IDs de modelo como `zai/glm-4.7`.

<div id="cli-setup">
  ## Configuración de la CLI
</div>

```bash
openclaw onboard --auth-choice zai-api-key
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

* Las versiones y la disponibilidad de GLM pueden cambiar; consulta la documentación de Z.AI para obtener la información más reciente.
* Ejemplos de ID de modelo incluyen `glm-4.7` y `glm-4.6`.
* Para obtener más detalles sobre el proveedor, consulta [/providers/zai](/es/providers/zai).