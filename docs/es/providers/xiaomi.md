---
title: Xiaomi
summary: "Usar Xiaomi MiMo (mimo-v2-flash) con OpenClaw"
read_when:
  - Quieres usar modelos Xiaomi MiMo en OpenClaw
  - Necesitas tener configurado XIAOMI_API_KEY
---

<div id="xiaomi-mimo">
  # Xiaomi MiMo
</div>

Xiaomi MiMo es la plataforma de API para los modelos **MiMo**. Proporciona APIs REST compatibles con
los formatos de OpenAI y Anthropic y utiliza claves de API para la autenticación. Crea tu clave de API en
la [consola de Xiaomi MiMo](https://platform.xiaomimimo.com/#/console/api-keys). OpenClaw utiliza
el proveedor `xiaomi` con una clave de API de Xiaomi MiMo.

<div id="model-overview">
  ## Descripción general del modelo
</div>

* **mimo-v2-flash**: ventana de contexto de 262144 tokens, compatible con la API Messages de Anthropic.
* URL base: `https://api.xiaomimimo.com/anthropic`
* Autorización: `Bearer $XIAOMI_API_KEY`

<div id="cli-setup">
  ## Configuración de la CLI
</div>

```bash
openclaw onboard --auth-choice xiaomi-api-key
# o en modo no interactivo
openclaw onboard --auth-choice xiaomi-api-key --xiaomi-api-key "$XIAOMI_API_KEY"
```

<div id="config-snippet">
  ## Ejemplo de configuración
</div>

```json5
{
  env: { XIAOMI_API_KEY: "your-key" },
  agents: { defaults: { model: { primary: "xiaomi/mimo-v2-flash" } } },
  models: {
    mode: "merge",
    providers: {
      xiaomi: {
        baseUrl: "https://api.xiaomimimo.com/anthropic",
        api: "anthropic-messages",
        apiKey: "XIAOMI_API_KEY",
        models: [
          {
            id: "mimo-v2-flash",
            name: "Xiaomi MiMo V2 Flash",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 262144,
            maxTokens: 8192
          }
        ]
      }
    }
  }
}
```

<div id="notes">
  ## Notas
</div>

* Referencia del modelo: `xiaomi/mimo-v2-flash`.
* El proveedor se inyecta automáticamente cuando `XIAOMI_API_KEY` está configurada (o existe un perfil de autenticación).
* Consulta [/concepts/model-providers](/es/concepts/model-providers) para conocer las reglas del proveedor.