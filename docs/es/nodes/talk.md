---
title: Talk
summary: "Modo Talk: conversaciones de voz continuas con ElevenLabs TTS"
read_when:
  - Implementar el modo Talk en macOS/iOS/Android
  - Cambiar el comportamiento de voz/TTS/interrupción
---

<div id="talk-mode">
  # Modo de conversación
</div>

El modo de conversación es un bucle continuo de conversación por voz:

1) Escuchar al usuario
2) Enviar la transcripción al modelo (sesión principal, chat.send)
3) Esperar la respuesta
4) Reproducirla con ElevenLabs (reproducción en streaming)

<div id="behavior-macos">
  ## Comportamiento (macOS)
</div>

- **Superposición siempre visible** mientras el modo Hablar está habilitado.
- Transiciones de fase **Escuchando → Pensando → Hablando**.
- En una **pausa corta** (ventana de silencio), se envía la transcripción actual.
- Las respuestas se **escriben en WebChat** (igual que al escribir con el teclado).
- **Interrumpir al hablar** (activado de forma predeterminada): cuando el usuario empieza a hablar mientras el asistente está hablando, detenemos la reproducción y registramos la marca de tiempo de la interrupción para el siguiente prompt.

<div id="voice-directives-in-replies">
  ## Directivas de voz en las respuestas
</div>

El asistente puede anteponer a su respuesta una **sola línea JSON** para controlar la voz:

```json
{"voice":"<voice-id>","once":true}
```

Reglas:

* Solo se tiene en cuenta la primera línea no vacía.
* Las claves desconocidas se ignoran.
* `once: true` se aplica solo a la respuesta actual.
* Sin `once`, la voz se convierte en el nuevo valor predeterminado para el modo Talk.
* La línea JSON se elimina antes de la reproducción de TTS.

Claves admitidas:

* `voice` / `voice_id` / `voiceId`
* `model` / `model_id` / `modelId`
* `speed`, `rate` (WPM), `stability`, `similarity`, `style`, `speakerBoost`
* `seed`, `normalize`, `lang`, `output_format`, `latency_tier`
* `once`


<div id="config-openclawopenclawjson">
  ## Configuración (`~/.openclaw/openclaw.json`)
</div>

```json5
{
  "talk": {
    "voiceId": "elevenlabs_voice_id",
    "modelId": "eleven_v3",
    "outputFormat": "mp3_44100_128",
    "apiKey": "elevenlabs_api_key",
    "interruptOnSpeech": true
  }
}
```

Valores predeterminados:

* `interruptOnSpeech`: true
* `voiceId`: usa `ELEVENLABS_VOICE_ID` / `SAG_VOICE_ID` (o la primera voz de ElevenLabs cuando la clave de API está disponible)
* `modelId`: usa `eleven_v3` de forma predeterminada cuando no se establece
* `apiKey`: usa `ELEVENLABS_API_KEY` de forma predeterminada (o el perfil de shell del Gateway si está disponible)
* `outputFormat`: usa `pcm_44100` de forma predeterminada en macOS/iOS y `pcm_24000` en Android (establece `mp3_*` para forzar el streaming en MP3)


<div id="macos-ui">
  ## UI de macOS
</div>

- Interruptor en la barra de menús: **Talk**
- Pestaña de configuración: grupo **Talk Mode** (ID de voz + interruptor de interrupción)
- Superposición:
  - **Escuchando**: la nube pulsa según el nivel del micrófono
  - **Pensando**: animación descendente
  - **Hablando**: anillos expansivos
  - Clic en la nube: detener la voz
  - Clic en X: salir del modo Talk

<div id="notes">
  ## Notas
</div>

- Requiere permisos de voz y micrófono.
- Usa `chat.send` con la clave de sesión `main`.
- TTS usa la API de streaming de ElevenLabs con `ELEVENLABS_API_KEY` y reproducción incremental en macOS/iOS/Android para reducir la latencia.
- `stability` para `eleven_v3` se valida a `0.0`, `0.5` o `1.0`; otros modelos aceptan `0..1`.
- `latency_tier` se valida a `0..4` cuando se establece.
- Android admite los formatos de salida `pcm_16000`, `pcm_22050`, `pcm_24000` y `pcm_44100` para streaming de AudioTrack de baja latencia.