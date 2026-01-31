---
title: Audio
summary: "Cómo se descargan y transcriben las notas de audio/notas de voz entrantes, y cómo se incorporan a las respuestas"
read_when:
  - Al cambiar la transcripción de audio o el manejo de medios
---

<div id="audio-voice-notes-2026-01-17">
  # Audio / Notas de voz — 2026-01-17
</div>

<div id="what-works">
  ## Qué funciona
</div>

* **Comprensión de medios (audio)**: Si la comprensión de audio está habilitada (o se detecta automáticamente), OpenClaw:
  1. Localiza el primer archivo de audio adjunto (ruta local o URL) y lo descarga si es necesario.
  2. Aplica `maxBytes` antes de enviarlo a cada entrada de modelo.
  3. Ejecuta, en orden, la primera entrada de modelo apta (proveedor o CLI).
  4. Si falla o se omite (por tamaño/timeout), prueba con la siguiente entrada.
  5. Si tiene éxito, reemplaza `Body` con un bloque `[Audio]` y establece `{{Transcript}}`.
* **Análisis de comandos**: Cuando la transcripción se completa correctamente, `CommandBody`/`RawBody` se configuran con la transcripción para que los comandos con barra (slash commands) sigan funcionando.
* **Registro detallado**: Con `--verbose`, registramos cuándo se ejecuta la transcripción y cuándo reemplaza el cuerpo.

<div id="auto-detection-default">
  ## Detección automática (predeterminada)
</div>

Si **no configuras ningún modelo** y `tools.media.audio.enabled` **no** está establecido en `false`,
OpenClaw realiza la detección automática en este orden y se detiene en la primera opción que funcione:

1. **CLI locales** (si están instaladas)
   * `sherpa-onnx-offline` (requiere `SHERPA_ONNX_MODEL_DIR` con encoder/decoder/joiner/tokens)
   * `whisper-cli` (de `whisper-cpp`; usa `WHISPER_CPP_MODEL` o el modelo tiny integrado)
   * `whisper` (CLI de Python; descarga modelos automáticamente)
2. **Gemini CLI** (`gemini`) mediante `read_many_files`
3. **Claves de proveedores** (OpenAI → Groq → Deepgram → Google)

Para desactivar la detección automática, establece `tools.media.audio.enabled: false`.
Para personalizarla, establece `tools.media.audio.models`.
Nota: La detección de binarios es “best effort” en macOS/Linux/Windows; asegúrate de que la CLI esté en el `PATH` (expandimos `~`), o define un modelo de CLI explícito con una ruta de comando completa.

<div id="config-examples">
  ## Ejemplos de configuración
</div>

<div id="provider-cli-fallback-openai-whisper-cli">
  ### Proveedor + CLI de respaldo (OpenAI + Whisper CLI)
</div>

```json5
{
  tools: {
    media: {
      audio: {
        enabled: true,
        maxBytes: 20971520,
        models: [
          { provider: "openai", model: "gpt-4o-mini-transcribe" },
          {
            type: "cli",
            command: "whisper",
            args: ["--model", "base", "{{MediaPath}}"],
            timeoutSeconds: 45
          }
        ]
      }
    }
  }
}
```

<div id="provider-only-with-scope-gating">
  ### Solo proveedor con limitación por ámbito
</div>

```json5
{
  tools: {
    media: {
      audio: {
        enabled: true,
        scope: {
          default: "allow",
          rules: [
            { action: "deny", match: { chatType: "group" } }
          ]
        },
        models: [
          { provider: "openai", model: "gpt-4o-mini-transcribe" }
        ]
      }
    }
  }
}
```

<div id="provider-only-deepgram">
  ### Solo con proveedor (Deepgram)
</div>

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

<div id="notes-limits">
  ## Notas y límites
</div>

* La autenticación de proveedores sigue el orden estándar de autenticación de modelos (perfiles de autenticación, variables de entorno, `models.providers.*.apiKey`).
* Deepgram detecta `DEEPGRAM_API_KEY` cuando se usa `provider: "deepgram"`.
* Detalles de configuración de Deepgram: [Deepgram (audio transcription)](/es/providers/deepgram).
* Los proveedores de audio pueden sobrescribir `baseUrl`, `headers` y `providerOptions` mediante `tools.media.audio`.
* El límite de tamaño predeterminado es 20 MB (`tools.media.audio.maxBytes`). El audio que exceda este tamaño se omite para ese modelo y se prueba con la siguiente entrada.
* El `maxChars` predeterminado para audio no está establecido (transcripción completa). Define `tools.media.audio.maxChars` o `maxChars` por entrada para recortar la salida.
* El valor predeterminado automático en OpenAI es `gpt-4o-mini-transcribe`; define `model: "gpt-4o-transcribe"` para mayor precisión.
* Usa `tools.media.audio.attachments` para procesar múltiples notas de voz (`mode: "all"` + `maxAttachments`).
* La transcripción está disponible en las plantillas como `{{Transcript}}`.
* La salida estándar (stdout) de la CLI tiene un límite (5 MB); mantén la salida de la CLI concisa.

<div id="gotchas">
  ## Advertencias
</div>

* Las reglas de ámbito aplican la primera coincidencia (&quot;first match wins&quot;). `chatType` se normaliza a `direct`, `group` o `room`.
* Asegúrate de que tu CLI termine con código de salida 0 y emita texto plano; el JSON debe transformarse con `jq -r .text`.
* Mantén los tiempos de espera razonables (`timeoutSeconds`, valor predeterminado 60s) para evitar bloquear la cola de respuestas.