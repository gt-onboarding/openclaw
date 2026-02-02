---
title: Deepgram
summary: "Transcripción con Deepgram para notas de voz entrantes"
read_when:
  - Quieres conversión de voz a texto con Deepgram para adjuntos de audio
  - Necesitas un ejemplo rápido de configuración de Deepgram
---

<div id="deepgram-audio-transcription">
  # Deepgram (Transcripción de audio)
</div>

Deepgram es una API de voz a texto. En OpenClaw se utiliza para la **transcripción de notas de voz/audio entrantes** mediante `tools.media.audio`.

Cuando está habilitado, OpenClaw carga el archivo de audio en Deepgram e inyecta la transcripción
en el flujo de respuesta (bloque `{{Transcript}}` + `[Audio]`). Esto **no es en streaming**;
usa el endpoint de transcripción de audio pregrabado.

Sitio web: https://deepgram.com  
Documentación: https://developers.deepgram.com

<div id="quick-start">
  ## Inicio rápido
</div>

1. Configura tu clave de API:

```
DEEPGRAM_API_KEY=dg_...
```

2. Habilita el proveedor:

```json5
{
  tools: {
    media: {
      audio: {
        enabled: true,
        models: [{ provider: "deepgram", model: "nova-3" }]
      }
    }
  }
}
```


<div id="options">
  ## Opciones
</div>

* `model`: ID del modelo de Deepgram (predeterminado: `nova-3`)
* `language`: indicación de idioma (opcional)
* `tools.media.audio.providerOptions.deepgram.detect_language`: habilitar detección del idioma (opcional)
* `tools.media.audio.providerOptions.deepgram.punctuate`: habilitar puntuación (opcional)
* `tools.media.audio.providerOptions.deepgram.smart_format`: habilitar formato inteligente (opcional)

Ejemplo con idioma:

```json5
{
  tools: {
    media: {
      audio: {
        enabled: true,
        models: [
          { provider: "deepgram", model: "nova-3", language: "en" }
        ]
      }
    }
  }
}
```

Ejemplo con opciones de Deepgram:

```json5
{
  tools: {
    media: {
      audio: {
        enabled: true,
        providerOptions: {
          deepgram: {
            detect_language: true,
            punctuate: true,
            smart_format: true
          }
        },
        models: [{ provider: "deepgram", model: "nova-3" }]
      }
    }
  }
}
```


<div id="notes">
  ## Notas
</div>

- La autenticación sigue el orden estándar de autenticación de proveedores; `DEEPGRAM_API_KEY` es la vía más sencilla.
- Anula los endpoints o las cabeceras con `tools.media.audio.baseUrl` y `tools.media.audio.headers` cuando uses un proxy.
- La salida sigue las mismas reglas de audio que otros proveedores (límites de tamaño, tiempos de espera, inyección de transcripciones).