---
title: Tts
summary: "Síntesis de texto a voz (TTS) para respuestas salientes"
read_when:
  - Habilitar la síntesis de texto a voz para las respuestas
  - Configurar proveedores TTS o límites
  - Usar comandos /tts
---

<div id="text-to-speech-tts">
  # Conversión de texto a voz (TTS)
</div>

OpenClaw puede convertir las respuestas salientes en audio usando ElevenLabs, OpenAI o Edge TTS.
Funciona en cualquier lugar donde OpenClaw pueda enviar audio; en Telegram aparece como una burbuja redonda de nota de voz.

<div id="supported-services">
  ## Servicios admitidos
</div>

* **ElevenLabs** (proveedor principal o de respaldo)
* **OpenAI** (proveedor principal o de respaldo; también se utiliza para resúmenes)
* **Edge TTS** (proveedor principal o de respaldo; usa `node-edge-tts`, predeterminado cuando no hay claves de api)

<div id="edge-tts-notes">
  ### Notas sobre Edge TTS
</div>

Edge TTS utiliza el servicio de TTS neuronal en línea de Microsoft Edge a través de la
librería `node-edge-tts`. Es un servicio alojado (no local), utiliza los endpoints de
Microsoft y no requiere una clave de API. `node-edge-tts` expone opciones de
configuración de voz y formatos de salida, pero no todas las opciones son
admitidas por el servicio de Edge. citeturn2search0

Dado que Edge TTS es un servicio web público sin un SLA ni cuotas publicados, trátalo
como un servicio de mejor esfuerzo (sin garantías). Si necesitas límites garantizados y soporte,
utiliza OpenAI o ElevenLabs. La Speech REST API de Microsoft documenta un límite de
audio de 10 minutos por solicitud; Edge TTS no publica límites, así que asume
límites similares o inferiores. citeturn0search3

<div id="optional-keys">
  ## Claves opcionales
</div>

Si quieres usar OpenAI o ElevenLabs:

* `ELEVENLABS_API_KEY` (o `XI_API_KEY`)
* `OPENAI_API_KEY`

Edge TTS **no** requiere una clave de API. Si no se encuentra ninguna clave de API, OpenClaw usa Edge TTS de forma predeterminada
(a menos que se deshabilite con `messages.tts.edge.enabled=false`).

Si se configuran varios proveedores, primero se usa el proveedor seleccionado y los demás actúan como opciones de respaldo.
El resumen automático usa el `summaryModel` configurado (o `agents.defaults.model.primary`),
por lo que ese proveedor también debe estar autenticado si habilitas resúmenes.

<div id="service-links">
  ## Enlaces de servicios
</div>

* [Guía de texto a voz de OpenAI](https://platform.openai.com/docs/guides/text-to-speech)
* [Referencia de la API de audio de OpenAI](https://platform.openai.com/docs/api-reference/audio)
* [Texto a voz de ElevenLabs](https://elevenlabs.io/docs/api-reference/text-to-speech)
* [Autenticación de ElevenLabs](https://elevenlabs.io/docs/api-reference/authentication)
* [node-edge-tts](https://github.com/SchneeHertz/node-edge-tts)
* [Formatos de salida de Microsoft Speech](https://learn.microsoft.com/azure/ai-services/speech-service/rest-text-to-speech#audio-outputs)

<div id="is-it-enabled-by-default">
  ## ¿Está habilitado de forma predeterminada?
</div>

No. Auto‑TTS está **desactivado** de forma predeterminada. Habilítalo en la configuración con
`messages.tts.auto` o por sesión con `/tts always` (alias: `/tts on`).

Edge TTS **sí** está habilitado de forma predeterminada una vez que TTS está habilitado, y se usa automáticamente
cuando no hay claves de api de OpenAI o ElevenLabs disponibles.

<div id="config">
  ## Configuración
</div>

La configuración de TTS se define en `messages.tts` dentro de `openclaw.json`.
El esquema completo está en [Configuración del Gateway](/es/gateway/configuration).

<div id="minimal-config-enable-provider">
  ### Configuración mínima (activación + proveedor)
</div>

```json5
{
  messages: {
    tts: {
      auto: "always",
      provider: "elevenlabs"
    }
  }
}
```

<div id="openai-primary-with-elevenlabs-fallback">
  ### OpenAI principal con ElevenLabs como respaldo
</div>

```json5
{
  messages: {
    tts: {
      auto: "always",
      provider: "openai",
      summaryModel: "openai/gpt-4.1-mini",
      modelOverrides: {
        enabled: true
      },
      openai: {
        apiKey: "openai_api_key",
        model: "gpt-4o-mini-tts",
        voice: "alloy"
      },
      elevenlabs: {
        apiKey: "elevenlabs_api_key",
        baseUrl: "https://api.elevenlabs.io",
        voiceId: "voice_id",
        modelId: "eleven_multilingual_v2",
        seed: 42,
        applyTextNormalization: "auto",
        languageCode: "en",
        voiceSettings: {
          stability: 0.5,
          similarityBoost: 0.75,
          style: 0.0,
          useSpeakerBoost: true,
          speed: 1.0
        }
      }
    }
  }
}
```

<div id="edge-tts-primary-no-api-key">
  ### Edge TTS principal (sin clave de api)
</div>

```json5
{
  messages: {
    tts: {
      auto: "always",
      provider: "edge",
      edge: {
        enabled: true,
        voice: "en-US-MichelleNeural",
        lang: "en-US",
        outputFormat: "audio-24khz-48kbitrate-mono-mp3",
        rate: "+10%",
        pitch: "-5%"
      }
    }
  }
}
```

<div id="disable-edge-tts">
  ### Desactivar Edge TTS
</div>

```json5
{
  messages: {
    tts: {
      edge: {
        enabled: false
      }
    }
  }
}
```

<div id="custom-limits-prefs-path">
  ### Límites personalizados y ruta de preferencias
</div>

```json5
{
  messages: {
    tts: {
      auto: "always",
      maxTextLength: 4000,
      timeoutMs: 30000,
      prefsPath: "~/.openclaw/settings/tts.json"
    }
  }
}
```

<div id="only-reply-with-audio-after-an-inbound-voice-note">
  ### Responde solo con audio después de recibir una nota de voz entrante
</div>

```json5
{
  messages: {
    tts: {
      auto: "inbound"
    }
  }
}
```

<div id="disable-auto-summary-for-long-replies">
  ### Desactivar el resumen automático para las respuestas largas
</div>

```json5
{
  messages: {
    tts: {
      auto: "always"
    }
  }
}
```

A continuación, ejecuta:

```
/tts summary off
```

<div id="notes-on-fields">
  ### Notas sobre los campos
</div>

* `auto`: modo TTS automático (`off`, `always`, `inbound`, `tagged`).
  * `inbound` solo envía audio después de una nota de voz entrante.
  * `tagged` solo envía audio cuando la respuesta incluye etiquetas `[[tts]]`.
* `enabled`: conmutador heredado (doctor lo migra a `auto`).
* `mode`: `"final"` (predeterminado) o `"all"` (incluye respuestas de herramientas/bloques).
* `provider`: `"elevenlabs"`, `"openai"` o `"edge"` (el fallback es automático).
* Si `provider` no está **establecido**, OpenClaw prefiere `openai` (si hay clave), luego `elevenlabs` (si hay clave),
  de lo contrario `edge`.
* `summaryModel`: modelo económico opcional para resumen automático; de forma predeterminada es `agents.defaults.model.primary`.
  * Acepta `provider/model` o un alias de modelo configurado.
* `modelOverrides`: permite que el modelo emita directivas de TTS (activado de forma predeterminada).
* `maxTextLength`: límite estricto para la entrada de TTS (caracteres). `/tts audio` falla si se excede.
* `timeoutMs`: tiempo de espera de la solicitud (ms).
* `prefsPath`: anula la ruta local del JSON de preferencias (provider/limit/summary).
* Los valores de `apiKey` recurren a variables de entorno (`ELEVENLABS_API_KEY`/`XI_API_KEY`, `OPENAI_API_KEY`).
* `elevenlabs.baseUrl`: sobrescribe la URL base de la API de ElevenLabs.
* `elevenlabs.voiceSettings`:
  * `stability`, `similarityBoost`, `style`: `0..1`
  * `useSpeakerBoost`: `true|false`
  * `speed`: `0.5..2.0` (1.0 = normal)
* `elevenlabs.applyTextNormalization`: `auto|on|off`
* `elevenlabs.languageCode`: código ISO 639-1 de 2 letras (p. ej. `en`, `de`)
* `elevenlabs.seed`: entero `0..4294967295` (determinismo en la medida de lo posible)
* `edge.enabled`: permite el uso de TTS de Edge (predeterminado `true`; sin clave de API).
* `edge.voice`: nombre de la voz neural de Edge (p. ej. `en-US-MichelleNeural`).
* `edge.lang`: código de idioma (p. ej. `en-US`).
* `edge.outputFormat`: formato de salida de Edge (p. ej. `audio-24khz-48kbitrate-mono-mp3`).
  * Consulta los formatos de salida de Microsoft Speech para valores válidos; no todos los formatos son compatibles con Edge.
* `edge.rate` / `edge.pitch` / `edge.volume`: cadenas de porcentaje (p. ej. `+10%`, `-5%`).
* `edge.saveSubtitles`: guarda subtítulos JSON junto al archivo de audio.
* `edge.proxy`: URL de proxy para solicitudes de Edge TTS.
* `edge.timeoutMs`: anula el tiempo de espera de la solicitud (ms).

<div id="model-driven-overrides-default-on">
  ## Anulaciones dirigidas por el modelo (activadas de forma predeterminada)
</div>

De forma predeterminada, el modelo **puede** emitir directivas de TTS para una única respuesta.
Cuando `messages.tts.auto` es `tagged`, estas directivas son necesarias para activar el audio.

Cuando esta opción está habilitada, el modelo puede emitir directivas `[[tts:...]]` para anular la voz
en una única respuesta, además de un bloque opcional `[[tts:text]]...[[/tts:text]]`
para proporcionar etiquetas expresivas (risas, indicaciones de canto, etc.) que solo
deben aparecer en el audio.

Ejemplo de payload de respuesta:

```
Here you go.

[[tts:provider=elevenlabs voiceId=pMsXgVXv3BLzUgSXRplE model=eleven_v3 speed=1.1]]
[[tts:text]](laughs) Read the song once more.[[/tts:text]]
```

Claves de directiva disponibles (cuando están habilitadas):

* `provider` (`openai` | `elevenlabs` | `edge`)
* `voice` (voz de OpenAI) o `voiceId` (ElevenLabs)
* `model` (modelo TTS de OpenAI o identificador de modelo de ElevenLabs)
* `stability`, `similarityBoost`, `style`, `speed`, `useSpeakerBoost`
* `applyTextNormalization` (`auto|on|off`)
* `languageCode` (ISO 639-1)
* `seed`

Desactivar todas las sobrescrituras de modelo:

```json5
{
  messages: {
    tts: {
      modelOverrides: {
        enabled: false
      }
    }
  }
}
```

Lista de permitidos opcional (para desactivar anulaciones específicas manteniendo las etiquetas habilitadas):

```json5
{
  messages: {
    tts: {
      modelOverrides: {
        enabled: true,
        allowProvider: false,
        allowSeed: false
      }
    }
  }
}
```

<div id="per-user-preferences">
  ## Preferencias por usuario
</div>

Los comandos de barra diagonal (`/`) guardan anulaciones locales en `prefsPath` (por defecto:
`~/.openclaw/settings/tts.json`, se puede sobrescribir con `OPENCLAW_TTS_PREFS` o
`messages.tts.prefsPath`).

Campos almacenados:

* `enabled`
* `provider`
* `maxLength` (umbral de resumen; valor por defecto 1500 caracteres)
* `summarize` (valor por defecto `true`)

Estos valores sobrescriben `messages.tts.*` para ese host.

<div id="output-formats-fixed">
  ## Formatos de salida (fijos)
</div>

* **Telegram**: nota de voz Opus (`opus_48000_64` de ElevenLabs, `opus` de OpenAI).
  * 48kHz / 64kbps es un buen compromiso para notas de voz y es obligatorio para la burbuja redonda.
* **Otros canales**: MP3 (`mp3_44100_128` de ElevenLabs, `mp3` de OpenAI).
  * 44.1kHz / 128kbps es el equilibrio predeterminado para la claridad del habla.
* **Edge TTS**: utiliza `edge.outputFormat` (valor predeterminado `audio-24khz-48kbitrate-mono-mp3`).
  * `node-edge-tts` acepta un `outputFormat`, pero no todos los formatos están disponibles
    en el servicio Edge. citeturn2search0
  * Los valores de formato de salida siguen los formatos de salida de Microsoft Speech (incluido Ogg/WebM Opus). citeturn1search0
  * Telegram `sendVoice` acepta OGG/MP3/M4A; usa OpenAI/ElevenLabs si necesitas
    notas de voz Opus garantizadas. citeturn1search1
  * Si el formato de salida configurado para Edge falla, OpenClaw reintenta con MP3.

Los formatos de OpenAI/ElevenLabs son fijos; Telegram espera Opus para la UX de notas de voz.

<div id="auto-tts-behavior">
  ## Comportamiento de Auto-TTS
</div>

Cuando está activado, OpenClaw:

* omite TTS si la respuesta ya contiene contenido multimedia o una directiva `MEDIA:`.
* omite respuestas muy cortas (&lt; 10 caracteres).
* hace un resumen de las respuestas largas cuando está habilitado, usando `agents.defaults.model.primary` (o `summaryModel`).
* adjunta el audio generado a la respuesta.

Si la respuesta supera `maxLength` y el resumen está desactivado (o no hay clave de API para el modelo de resumen), se omite el audio y se envía la respuesta de texto normal.

<div id="flow-diagram">
  ## Diagrama de flujo
</div>

```
Reply -> TTS enabled?
  no  -> send text
  yes -> has media / MEDIA: / short?
          yes -> send text
          no  -> length > limit?
                   no  -> TTS -> attach audio
                   yes -> summary enabled?
                            no  -> send text
                            yes -> summarize (summaryModel or agents.defaults.model.primary)
                                      -> TTS -> attach audio
```

<div id="slash-command-usage">
  ## Uso del comando slash
</div>

Hay un solo comando: `/tts`.
Consulta [Slash commands](/es/tools/slash-commands) para obtener detalles sobre cómo activarlos.

Nota para Discord: `/tts` es un comando integrado de Discord, por lo que OpenClaw registra
`/voice` como el comando nativo allí. El comando de texto `/tts ...` sigue funcionando.

```
/tts off
/tts always
/tts inbound
/tts tagged
/tts status
/tts provider openai
/tts limit 2000
/tts summary off
/tts audio Hello from OpenClaw
```

Notas:

* Los comandos requieren un remitente autorizado (las reglas de lista de permitidos/propietario se siguen aplicando).
* `commands.text` o el registro nativo de comandos deben estar habilitados.
* `off|always|inbound|tagged` son conmutadores por sesión (`/tts on` es un alias de `/tts always`).
* `limit` y `summary` se almacenan en las preferencias locales, no en la configuración principal.
* `/tts audio` genera una respuesta de audio única (no cambia el estado de TTS).

<div id="agent-tool">
  ## Herramienta de agente
</div>

La herramienta `tts` convierte texto en voz y devuelve una ruta `MEDIA:`. Cuando
el resultado es compatible con Telegram, la herramienta incluye `[[audio_as_voice]]` para que
Telegram envíe un mensaje de voz.

<div id="gateway-rpc">
  ## Gateway RPC
</div>

Métodos del Gateway:

* `tts.status`
* `tts.enable`
* `tts.disable`
* `tts.convert`
* `tts.setProvider`
* `tts.providers`