---
title: Reintento
summary: "Política de reintentos para llamadas salientes a proveedores"
read_when:
  - Al actualizar el comportamiento de reintento del proveedor o sus valores predeterminados
  - Al depurar errores de envío del proveedor o límites de velocidad (rate limits)
---

<div id="retry-policy">
  # Política de reintentos
</div>

<div id="goals">
  ## Objetivos
</div>

- Reintentar por cada petición HTTP, no por flujo de varios pasos.
- Conservar el orden reintentando únicamente el paso actual.
- Evitar duplicar operaciones no idempotentes.

<div id="defaults">
  ## Valores predeterminados
</div>

- Intentos: 3
- Límite máximo de espera: 30000 ms
- Jitter: 0,1 (10 %)
- Valores predeterminados del proveedor:
  - Tiempo mínimo de espera en Telegram: 400 ms
  - Tiempo mínimo de espera en Discord: 500 ms

<div id="behavior">
  ## Comportamiento
</div>

<div id="discord">
  ### Discord
</div>

- Reintenta solo ante errores de límite de velocidad (HTTP 429).
- Usa `retry_after` de Discord cuando está disponible; de lo contrario, aplica espera exponencial.

<div id="telegram">
  ### Telegram
</div>

- Se reintenta en errores transitorios (429, timeout, connect/reset/closed, no disponible temporalmente).
- Usa `retry_after` cuando está disponible; de lo contrario, aplica backoff exponencial.
- Los errores de análisis de Markdown no se vuelven a intentar; se recurre a texto sin formato.

<div id="configuration">
  ## Configuración
</div>

Configura la política de reintentos por proveedor en `~/.openclaw/openclaw.json`:

```json5
{
  channels: {
    telegram: {
      retry: {
        attempts: 3,
        minDelayMs: 400,
        maxDelayMs: 30000,
        jitter: 0.1
      }
    },
    discord: {
      retry: {
        attempts: 3,
        minDelayMs: 500,
        maxDelayMs: 30000,
        jitter: 0.1
      }
    }
  }
}
```


<div id="notes">
  ## Notas
</div>

- Los reintentos se aplican por cada solicitud (envío de mensajes, carga de contenido multimedia, reacción, encuesta, sticker).
- Los flujos compuestos no reintentan los pasos ya completados.