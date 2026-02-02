---
title: Comprensión de medios
summary: "Comprensión de imágenes/audio/video entrantes (opcional) con proveedor + CLI como respaldo"
read_when:
  - Diseñando o refactorizando la comprensión de medios
  - Ajustando el preprocesamiento de contenido de audio/video/imagen entrante
---

<div id="media-understanding-inbound-2026-01-17">
  # Comprensión de medios (entrante) — 2026-01-17
</div>

OpenClaw puede **resumir medios entrantes** (imagen/audio/video) antes de que se ejecute el pipeline de respuesta. Detecta automáticamente cuándo hay herramientas locales o claves de proveedor disponibles, y puede desactivarse o personalizarse. Si la comprensión está desactivada, los modelos siguen recibiendo los archivos/URL originales como siempre.

<div id="goals">
  ## Objetivos
</div>

* Opcional: preprocesar el contenido multimedia entrante en texto breve para un enrutamiento más rápido y un mejor análisis de comandos.
* Conservar siempre el contenido multimedia original al entregarlo al modelo.
* Admitir **APIs de proveedores** y mecanismos de reserva vía **CLI**.
* Permitir múltiples modelos con mecanismos de reserva en orden de prioridad (error/tamaño/timeout).

<div id="highlevel-behavior">
  ## Comportamiento de alto nivel
</div>

1. Recopila los archivos adjuntos entrantes (`MediaPaths`, `MediaUrls`, `MediaTypes`).
2. Para cada capacidad habilitada (imagen/audio/video), selecciona archivos adjuntos según la política (predeterminado: **primero**).
3. Elige la primera entrada de modelo elegible (tamaño + capacidad + autenticación).
4. Si un modelo falla o el contenido multimedia es demasiado grande, **pasa a la siguiente entrada**.
5. En caso de éxito:
   * `Body` se convierte en un bloque `[Image]`, `[Audio]` o `[Video]`.
   * El audio establece `{{Transcript}}`; el análisis de comandos usa el texto de los subtítulos cuando está presente,
     en caso contrario usa la transcripción.
   * Los subtítulos se conservan como `User text:` dentro del bloque.

Si la comprensión falla o está deshabilitada, **el flujo de respuesta continúa** con el cuerpo original + archivos adjuntos.

<div id="config-overview">
  ## Descripción general de la configuración
</div>

`tools.media` admite **modelos compartidos** más anulaciones por capacidad:

* `tools.media.models`: lista de modelos compartidos (usa `capabilities` para limitar).
* `tools.media.image` / `tools.media.audio` / `tools.media.video`:
  * valores predeterminados (`prompt`, `maxChars`, `maxBytes`, `timeoutSeconds`, `language`)
  * anulaciones de proveedor (`baseUrl`, `headers`, `providerOptions`)
  * opciones de audio de Deepgram mediante `tools.media.audio.providerOptions.deepgram`
  * **lista opcional de `models` por capacidad** (preferida frente a los modelos compartidos)
  * política de `attachments` (`mode`, `maxAttachments`, `prefer`)
  * `scope` (limitación opcional por clave de canal/chatType/session)
* `tools.media.concurrency`: máximo de ejecuciones concurrentes de capacidades (valor predeterminado **2**).

```json5
{
  tools: {
    media: {
      models: [ /* shared list */ ],
      image: { /* sobrescrituras opcionales */ },
      audio: { /* sobrescrituras opcionales */ },
      video: { /* sobrescrituras opcionales */ }
    }
  }
}
```

<div id="model-entries">
  ### Entradas de modelo
</div>

Cada entrada de `models[]` puede ser de tipo **proveedor** o **CLI**:

```json5
{
  type: "provider",        // default if omitted
  provider: "openai",
  model: "gpt-5.2",
  prompt: "Describe the image in <= 500 chars.",
  maxChars: 500,
  maxBytes: 10485760,
  timeoutSeconds: 60,
  capabilities: ["image"], // opcional, se usa para entradas multimodales
  profile: "vision-profile",
  preferredProfile: "vision-fallback"
}
```

```json5
{
  type: "cli",
  command: "gemini",
  args: [
    "-m",
    "gemini-3-flash",
    "--allowed-tools",
    "read_file",
    "Read the media at {{MediaPath}} and describe it in <= {{MaxChars}} characters."
  ],
  maxChars: 500,
  maxBytes: 52428800,
  timeoutSeconds: 120,
  capabilities: ["video", "image"]
}
```

Las plantillas de la CLI también pueden utilizar:

* `{{MediaDir}}` (directorio que contiene el archivo multimedia)
* `{{OutputDir}}` (directorio de trabajo temporal creado para esta ejecución)
* `{{OutputBase}}` (ruta base del archivo temporal, sin extensión)

<div id="defaults-and-limits">
  ## Valores predeterminados y límites
</div>

Valores predeterminados recomendados:

* `maxChars`: **500** para imagen/vídeo (corto, apto para comandos)
* `maxChars`: **unset** para audio (transcripción completa, salvo que definas un límite)
* `maxBytes`:
  * imagen: **10MB**
  * audio: **20MB**
  * vídeo: **50MB**

Reglas:

* Si el archivo multimedia supera `maxBytes`, ese modelo se omite y se **prueba el siguiente modelo**.
* Si el modelo devuelve más de `maxChars`, la salida se trunca.
* `prompt` usa por defecto el sencillo “Describe the {media}.” más la indicación de `maxChars` (solo imagen/vídeo).
* Si `<capability>.enabled: true` pero no hay modelos configurados, OpenClaw intenta usar el
  **modelo de respuesta activo** cuando su proveedor admite esta capacidad.

<div id="auto-detect-media-understanding-default">
  ### Detección automática de comprensión de contenido multimedia (predeterminado)
</div>

Si `tools.media.<capability>.enabled` **no** está establecido en `false` y no has
configurado modelos, OpenClaw detecta automáticamente en este orden y **se detiene en la primera
opción que funciona**:

1. **CLIs locales** (solo audio; si están instaladas)
   * `sherpa-onnx-offline` (requiere `SHERPA_ONNX_MODEL_DIR` con encoder/decoder/joiner/tokens)
   * `whisper-cli` (`whisper-cpp`; usa `WHISPER_CPP_MODEL` o el modelo tiny incluido)
   * `whisper` (CLI de Python; descarga modelos automáticamente)
2. **Gemini CLI** (`gemini`) usando `read_many_files`
3. **Claves de proveedor**
   * Audio: OpenAI → Groq → Deepgram → Google
   * Imagen: OpenAI → Anthropic → Google → MiniMax
   * Video: Google

Para desactivar la detección automática, establece:

```json5
{
  tools: {
    media: {
      audio: {
        enabled: false
      }
    }
  }
}
```

Nota: la detección del binario es de tipo best-effort en macOS/Linux/Windows; asegúrate de que la CLI esté en `PATH` (expandimos `~`) o establece un modelo CLI explícito con la ruta completa del comando.

<div id="capabilities-optional">
  ## Capacidades (opcional)
</div>

Si configuras `capabilities`, la entrada solo se ejecuta para esos tipos de medios. Para listas compartidas, OpenClaw puede inferir valores predeterminados:

* `openai`, `anthropic`, `minimax`: **imagen**
* `google` (Gemini API): **imagen + audio + video**
* `groq`: **audio**
* `deepgram`: **audio**

Para entradas de CLI, **configura `capabilities` explícitamente** para evitar coincidencias inesperadas.
Si omites `capabilities`, la entrada podrá usarse en la lista en la que aparece.

<div id="provider-support-matrix-openclaw-integrations">
  ## Matriz de soporte de proveedores (integraciones de OpenClaw)
</div>

| Capacidad | Integración de proveedor | Notas |
|----------|--------------------------|-------|
| Imagen | OpenAI / Anthropic / Google / otros mediante `pi-ai` | Cualquier modelo del registro con capacidad de imagen funciona. |
| Audio | OpenAI, Groq, Deepgram, Google | Transcripción del proveedor (Whisper/Deepgram/Gemini). |
| Video | Google (Gemini API) | Comprensión de video del proveedor. |

<div id="recommended-providers">
  ## Proveedores recomendados
</div>

**Imagen**

* Usa tu modelo activo si admite imágenes.
* Buenos valores predeterminados: `openai/gpt-5.2`, `anthropic/claude-opus-4-5`, `google/gemini-3-pro-preview`.

**Audio**

* `openai/gpt-4o-mini-transcribe`, `groq/whisper-large-v3-turbo` o `deepgram/nova-3`.
* Fallback en la CLI: `whisper-cli` (whisper-cpp) o `whisper`.
* Configuración de Deepgram: [Deepgram (transcripción de audio)](/es/providers/deepgram).

**Video**

* `google/gemini-3-flash-preview` (rápido), `google/gemini-3-pro-preview` (más completo).
* Fallback en la CLI: CLI `gemini` (admite `read_file` en vídeo/audio).

<div id="attachment-policy">
  ## Política de adjuntos
</div>

Para cada capacidad, `attachments` controla qué adjuntos se procesan:

* `mode`: `first` (predeterminado) o `all`
* `maxAttachments`: limita el número de adjuntos procesados (valor predeterminado **1**)
* `prefer`: `first`, `last`, `path`, `url`

Cuando `mode: "all"`, los resultados se etiquetan como `[Image 1/2]`, `[Audio 2/2]`, etc.

<div id="config-examples">
  ## Ejemplos de configuración
</div>

<div id="1-shared-models-list-overrides">
  ### 1) Lista de modelos compartidos + sobrescrituras
</div>

```json5
{
  tools: {
    media: {
      models: [
        { provider: "openai", model: "gpt-5.2", capabilities: ["image"] },
        { provider: "google", model: "gemini-3-flash-preview", capabilities: ["image", "audio", "video"] },
        {
          type: "cli",
          command: "gemini",
          args: [
            "-m",
            "gemini-3-flash",
            "--allowed-tools",
            "read_file",
            "Lee el medio en {{MediaPath}} y descríbelo en <= {{MaxChars}} caracteres."
          ],
          capabilities: ["image", "video"]
        }
      ],
      audio: {
        attachments: { mode: "all", maxAttachments: 2 }
      },
      video: {
        maxChars: 500
      }
    }
  }
}
```

<div id="2-audio-video-only-image-off">
  ### 2) Solo audio y video (sin imagen)
</div>

```json5
{
  tools: {
    media: {
      audio: {
        enabled: true,
        models: [
          { provider: "openai", model: "gpt-4o-mini-transcribe" },
          {
            type: "cli",
            command: "whisper",
            args: ["--model", "base", "{{MediaPath}}"]
          }
        ]
      },
      video: {
        enabled: true,
        maxChars: 500,
        models: [
          { provider: "google", model: "gemini-3-flash-preview" },
          {
            type: "cli",
            command: "gemini",
            args: [
              "-m",
              "gemini-3-flash",
              "--allowed-tools",
              "read_file",
              "Read the media at {{MediaPath}} and describe it in <= {{MaxChars}} characters."
            ]
          }
        ]
      }
    }
  }
}
```

<div id="3-optional-image-understanding">
  ### 3) Comprensión opcional de imágenes
</div>

```json5
{
  tools: {
    media: {
      image: {
        enabled: true,
        maxBytes: 10485760,
        maxChars: 500,
        models: [
          { provider: "openai", model: "gpt-5.2" },
          { provider: "anthropic", model: "claude-opus-4-5" },
          {
            type: "cli",
            command: "gemini",
            args: [
              "-m",
              "gemini-3-flash",
              "--allowed-tools",
              "read_file",
              "Read the media at {{MediaPath}} and describe it in <= {{MaxChars}} characters."
            ]
          }
        ]
      }
    }
  }
}
```

<div id="4-multimodal-single-entry-explicit-capabilities">
  ### 4) Entrada única multimodal (capacidades explícitas)
</div>

```json5
{
  tools: {
    media: {
      image: { models: [{ provider: "google", model: "gemini-3-pro-preview", capabilities: ["image", "video", "audio"] }] },
      audio: { models: [{ provider: "google", model: "gemini-3-pro-preview", capabilities: ["image", "video", "audio"] }] },
      video: { models: [{ provider: "google", model: "gemini-3-pro-preview", capabilities: ["image", "video", "audio"] }] }
    }
  }
}
```

<div id="status-output">
  ## Salida de estado
</div>

Cuando se ejecuta Media Understanding, `/status` incluye una breve línea de resumen:

```
📎 Media: image ok (openai/gpt-5.2) · audio skipped (maxBytes)
```

Esto muestra los resultados por capacidad y, cuando corresponda, el proveedor/modelo seleccionado.

<div id="notes">
  ## Notas
</div>

* La comprensión es de tipo **best‑effort**: los errores no bloquean las respuestas.
* Los archivos adjuntos se siguen enviando a los modelos incluso cuando la comprensión está deshabilitada.
* Usa `scope` para limitar dónde se ejecuta la comprensión (por ejemplo, solo en mensajes directos).

<div id="related-docs">
  ## Documentos relacionados
</div>

* [Configuración](/es/gateway/configuration)
* [Soporte de imágenes y contenido multimedia](/es/nodes/images)