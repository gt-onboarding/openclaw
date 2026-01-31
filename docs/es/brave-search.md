---
title: Brave Search
summary: "Configuración de la API de Brave Search para web_search"
read_when:
  - Quieres usar Brave Search con web_search
  - Necesitas una BRAVE_API_KEY o información sobre el plan
---

<div id="brave-search-api">
  # Brave Search API
</div>

OpenClaw utiliza Brave Search como el proveedor predeterminado para `web_search`.

<div id="get-an-api-key">
  ## Obtener una clave de API
</div>

1. Crea una cuenta de Brave Search API en https://brave.com/search/api/
2. En el panel, elige el plan **Data for Search** y genera una clave de API.
3. Guarda la clave en la configuración (recomendado) o define `BRAVE_API_KEY` en el entorno del Gateway.

<div id="config-example">
  ## Ejemplo de configuración
</div>

```json5
{
  tools: {
    web: {
      search: {
        provider: "brave",
        apiKey: "BRAVE_API_KEY_HERE",
        maxResults: 5,
        timeoutSeconds: 30
      }
    }
  }
}
```

<div id="notes">
  ## Notas
</div>

* El plan Data for AI **no** es compatible con `web_search`.
* Brave ofrece un nivel gratuito y planes de pago; consulta el portal de la API de Brave para conocer los límites vigentes.

Consulta [Herramientas web](/es/tools/web) para ver la configuración completa de `web_search`.