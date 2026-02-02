---
title: Firecrawl
summary: "Fallback de Firecrawl para web_fetch (antibot + extracción en caché)"
read_when:
  - Quieres usar extracción web respaldada por Firecrawl
  - Necesitas una clave de API de Firecrawl
  - Quieres extracción con protección antibot para web_fetch
---

<div id="firecrawl">
  # Firecrawl
</div>

OpenClaw puede usar **Firecrawl** como extractor de respaldo para `web_fetch`. Es un servicio
de extracción de contenido alojado que admite la evasión de bloqueos a bots y el almacenamiento en caché, lo que ayuda
con sitios muy cargados de JavaScript o páginas que bloquean solicitudes HTTP directas.

<div id="get-an-api-key">
  ## Obtener una clave de API
</div>

1. Crea una cuenta de Firecrawl y genera una clave de API.
2. Guárdala en la configuración o configura `FIRECRAWL_API_KEY` en el entorno del Gateway.

<div id="configure-firecrawl">
  ## Configurar Firecrawl
</div>

```json5
{
  tools: {
    web: {
      fetch: {
        firecrawl: {
          apiKey: "FIRECRAWL_API_KEY_HERE",
          baseUrl: "https://api.firecrawl.dev",
          onlyMainContent: true,
          maxAgeMs: 172800000,
          timeoutSeconds: 60
        }
      }
    }
  }
}
```

Notas:

* `firecrawl.enabled` se establece en `true` de forma predeterminada cuando hay una clave de API.
* `maxAgeMs` controla la antigüedad máxima de los resultados en caché (ms). El valor predeterminado es de 2 días.

<div id="stealth-bot-circumvention">
  ## Sigilo / elusión de bots
</div>

Firecrawl expone un parámetro de **modo proxy** para la elusión de bots (`basic`, `stealth` o `auto`).
OpenClaw siempre usa `proxy: "auto"` más `storeInCache: true` para las solicitudes a Firecrawl.
Si se omite el proxy, Firecrawl usa `auto` de forma predeterminada. `auto` reintenta con proxies en modo stealth si un intento básico falla, lo que puede consumir más créditos
que usar solo scraping básico.

<div id="how-web_fetch-uses-firecrawl">
  ## Cómo `web_fetch` utiliza Firecrawl
</div>

Orden de extracción de `web_fetch`:

1. Readability (local)
2. Firecrawl (si está configurado)
3. Limpieza básica de HTML (último recurso)

Consulta [Herramientas web](/es/tools/web) para ver la configuración completa de las herramientas web.