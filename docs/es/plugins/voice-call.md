---
title: Llamada de voz
summary: "Complemento de llamada de voz: llamadas salientes y entrantes con Twilio/Telnyx/Plivo (instalación del complemento + configuración + CLI)"
read_when:
  - Quieres realizar una llamada de voz saliente desde OpenClaw
  - Estás configurando o desarrollando el complemento de llamada de voz
---

<div id="voice-call-plugin">
  # Llamada de voz (complemento)
</div>

Llamadas de voz para OpenClaw a través de un complemento. Admite notificaciones salientes y
conversaciones de varios turnos con políticas para llamadas entrantes.

Proveedores actuales:

- `twilio` (Programmable Voice + Media Streams)
- `telnyx` (Call Control v2)
- `plivo` (Voice API + XML transfer + GetInput speech)
- `mock` (dev/sin red)

Modelo mental rápido:

- Instalar el complemento
- Reiniciar el Gateway
- Configurar en `plugins.entries.voice-call.config`
- Usar `openclaw voicecall ...` o la herramienta `voice_call`

<div id="where-it-runs-local-vs-remote">
  ## Dónde se ejecuta (local vs remoto)
</div>

El complemento Voice Call se ejecuta **dentro del proceso del Gateway**.

Si usas un Gateway remoto, instala y configura el complemento en la **máquina donde se ejecuta el Gateway** y luego reinicia el Gateway para cargarlo.

<div id="install">
  ## Instalación
</div>

<div id="option-a-install-from-npm-recommended">
  ### Opción A: instalar desde npm (recomendado)
</div>

```bash
openclaw plugins install @openclaw/voice-call
```

Luego, reinicia el Gateway.


<div id="option-b-install-from-a-local-folder-dev-no-copying">
  ### Opción B: instalar desde una carpeta local (desarrollo, sin copiar)
</div>

```bash
openclaw plugins install ./extensions/voice-call
cd ./extensions/voice-call && pnpm install
```

Luego, reinicia el Gateway.


<div id="config">
  ## Configuración
</div>

Configura los valores en `plugins.entries.voice-call.config`:

```json5
{
  plugins: {
    entries: {
      "voice-call": {
        enabled: true,
        config: {
          provider: "twilio", // o "telnyx" | "plivo" | "mock"
          fromNumber: "+15550001234",
          toNumber: "+15550005678",

          twilio: {
            accountSid: "ACxxxxxxxx",
            authToken: "..."
          },

          plivo: {
            authId: "MAxxxxxxxxxxxxxxxxxxxx",
            authToken: "..."
          },

          // Webhook server
          serve: {
            port: 3334,
            path: "/voice/webhook"
          },

          // Public exposure (pick one)
          // publicUrl: "https://example.ngrok.app/voice/webhook",
          // tunnel: { provider: "ngrok" },
          // tailscale: { mode: "funnel", path: "/voice/webhook" }

          outbound: {
            defaultMode: "notify" // notify | conversation
          },

          streaming: {
            enabled: true,
            streamPath: "/voice/stream"
          }
        }
      }
    }
  }
}
```

Notas:

* Twilio/Telnyx exigen una URL de webhook **públicamente accesible**.
* Plivo exige una URL de webhook **públicamente accesible**.
* `mock` es un proveedor local de desarrollo (sin llamadas de red).
* `skipSignatureVerification` es solo para pruebas locales.
* Si usas la capa gratuita de ngrok, establece `publicUrl` en la URL exacta de ngrok; la verificación de la firma siempre se aplica.
* `tunnel.allowNgrokFreeTierLoopbackBypass: true` permite webhooks de Twilio con firmas no válidas **solo** cuando `tunnel.provider="ngrok"` y `serve.bind` es loopback (agente local de ngrok). Úsalo solo para desarrollo local.
* Las URLs de la capa gratuita de ngrok pueden cambiar o añadir comportamiento intersticial; si `publicUrl` deja de coincidir exactamente, las firmas de Twilio fallarán. Para producción, prefiere un dominio estable o Tailscale Funnel.


<div id="tts-for-calls">
  ## TTS para llamadas
</div>

Voice Call usa la configuración principal de `messages.tts` (OpenAI o ElevenLabs) para
transmitir voz en streaming en las llamadas. Puedes sobrescribirla en la configuración del complemento con la
**misma estructura**: se fusiona en profundidad con `messages.tts`.

```json5
{
  tts: {
    provider: "elevenlabs",
    elevenlabs: {
      voiceId: "pMsXgVXv3BLzUgSXRplE",
      modelId: "eleven_multilingual_v2"
    }
  }
}
```

Notas:

* **Edge TTS se ignora en las llamadas de voz** (el audio de telefonía necesita PCM; la salida de Edge no es fiable).
* Se usa Core TTS cuando la transmisión de medios de Twilio está habilitada; de lo contrario, las llamadas recurren a las voces nativas del proveedor.


<div id="more-examples">
  ### Más ejemplos
</div>

Usa solo el TTS principal (sin overrides):

```json5
{
  messages: {
    tts: {
      provider: "openai",
      openai: { voice: "alloy" }
    }
  }
}
```

Sobrescribir con ElevenLabs solo para las llamadas (mantén el valor predeterminado global en el resto):

```json5
{
  plugins: {
    entries: {
      "voice-call": {
        config: {
          tts: {
            provider: "elevenlabs",
            elevenlabs: {
              apiKey: "elevenlabs_key",
              voiceId: "pMsXgVXv3BLzUgSXRplE",
              modelId: "eleven_multilingual_v2"
            }
          }
        }
      }
    }
  }
}
```

Sobrescribe solo el modelo de OpenAI para las llamadas (ejemplo de fusión profunda):

```json5
{
  plugins: {
    entries: {
      "voice-call": {
        config: {
          tts: {
            openai: {
              model: "gpt-4o-mini-tts",
              voice: "marin"
            }
          }
        }
      }
    }
  }
}
```


<div id="inbound-calls">
  ## Llamadas entrantes
</div>

De forma predeterminada, la política de entrada de llamadas está establecida en `disabled`. Para habilitar las llamadas entrantes, configura:

```json5
{
  inboundPolicy: "allowlist",
  allowFrom: ["+15550001234"],
  inboundGreeting: "Hello! How can I help?"
}
```

Las respuestas automáticas utilizan el sistema de agentes. Configúralas con:

* `responseModel`
* `responseSystemPrompt`
* `responseTimeoutMs`


<div id="cli">
  ## CLI
</div>

```bash
openclaw voicecall call --to "+15555550123" --message "Hello from OpenClaw"
openclaw voicecall continue --call-id <id> --message "Any questions?"
openclaw voicecall speak --call-id <id> --message "One moment"
openclaw voicecall end --call-id <id>
openclaw voicecall status --call-id <id>
openclaw voicecall tail
openclaw voicecall expose --mode funnel
```


<div id="agent-tool">
  ## Herramienta del agente
</div>

Nombre de la herramienta: `voice_call`

Acciones:

- `initiate_call` (message, to?, mode?)
- `continue_call` (callId, message)
- `speak_to_user` (callId, message)
- `end_call` (callId)
- `get_status` (callId)

Este repositorio incluye un documento de habilidad asociado en `skills/voice-call/SKILL.md`.

<div id="gateway-rpc">
  ## RPC del Gateway
</div>

- `voicecall.initiate` (`to?`, `message`, `mode?`)
- `voicecall.continue` (`callId`, `message`)
- `voicecall.speak` (`callId`, `message`)
- `voicecall.end` (`callId`)
- `voicecall.status` (`callId`)