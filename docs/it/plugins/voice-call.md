---
title: Chiamata vocale
summary: "plugin Voice Call: chiamate vocali in uscita e in entrata tramite Twilio/Telnyx/Plivo (installazione plugin + configurazione + CLI)"
read_when:
  - Vuoi effettuare una chiamata vocale in uscita da OpenClaw
  - Stai configurando o sviluppando il plugin voice-call
---

<div id="voice-call-plugin">
  # Voice Call (plugin)
</div>

Chiamate vocali per OpenClaw tramite un plugin. Supporta notifiche in uscita e
conversazioni multi-turno con policy per le chiamate in ingresso.

Provider attuali:

- `twilio` (Programmable Voice + Media Streams)
- `telnyx` (Call Control v2)
- `plivo` (Voice API + trasferimento XML + GetInput speech)
- `mock` (dev/senza rete)

Schema mentale rapido:

- Installa il plugin
- Riavvia il Gateway
- Configura in `plugins.entries.voice-call.config`
- Usa `openclaw voicecall ...` oppure lo strumento `voice_call`

<div id="where-it-runs-local-vs-remote">
  ## Dove viene eseguito (locale vs remoto)
</div>

Il plugin Voice Call viene eseguito **all'interno del processo del Gateway**.

Se usi un Gateway remoto, installa/configura il plugin sulla **macchina su cui è in esecuzione il Gateway**, quindi riavvia il Gateway per caricarlo.

<div id="install">
  ## Installazione
</div>

<div id="option-a-install-from-npm-recommended">
  ### Opzione A: installazione da npm (consigliata)
</div>

```bash
openclaw plugins install @openclaw/voice-call
```

Poi riavvia il Gateway.


<div id="option-b-install-from-a-local-folder-dev-no-copying">
  ### Opzione B: installa da una cartella locale (dev, senza copia)
</div>

```bash
openclaw plugins install ./extensions/voice-call
cd ./extensions/voice-call && pnpm install
```

Quindi riavvia il Gateway.


<div id="config">
  ## Configurazione
</div>

Imposta la configurazione in `plugins.entries.voice-call.config`:

```json5
{
  plugins: {
    entries: {
      "voice-call": {
        enabled: true,
        config: {
          provider: "twilio", // or "telnyx" | "plivo" | "mock"
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

          // Esposizione pubblica (scegline uno)
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

Note:

* Twilio/Telnyx richiedono un URL di webhook **pubblicamente raggiungibile**.
* Plivo richiede un URL di webhook **pubblicamente raggiungibile**.
* `mock` è un provider locale per lo sviluppo (nessuna chiamata di rete).
* `skipSignatureVerification` è solo per i test locali.
* Se usi il piano gratuito di ngrok, imposta `publicUrl` all&#39;URL ngrok esatto; la verifica della firma è sempre applicata.
* `tunnel.allowNgrokFreeTierLoopbackBypass: true` consente webhook Twilio con firme non valide **solo** quando `tunnel.provider="ngrok"` e `serve.bind` è loopback (agente locale ngrok). Usalo solo per lo sviluppo locale.
* Gli URL del piano gratuito di ngrok possono cambiare o aggiungere comportamento interstiziale; se `publicUrl` non corrisponde più, le firme Twilio non verranno convalidate. Per la produzione, preferisci un dominio stabile o Tailscale Funnel.


<div id="tts-for-calls">
  ## TTS per le chiamate
</div>

Voice Call usa la configurazione principale `messages.tts` (OpenAI o ElevenLabs) per
lo streaming vocale durante le chiamate. Puoi sovrascriverla nella configurazione del plugin con la
**stessa struttura** — viene eseguito un merge profondo con `messages.tts`.

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

Note:

* **Edge TTS viene ignorato per le chiamate vocali** (l&#39;audio per la telefonia richiede PCM; l&#39;output di Edge non è affidabile).
* Il TTS Core viene utilizzato quando lo streaming multimediale Twilio è abilitato; in caso contrario, le chiamate ripiegano sulle voci native del provider.


<div id="more-examples">
  ### Ulteriori esempi
</div>

Usa solo il TTS core (senza override):

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

Esegui l&#39;override su ElevenLabs solo per le chiamate (mantieni l&#39;impostazione predefinita globale per il resto):

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

Sovrascrivi solo il modello OpenAI per le chiamate (esempio di deep‑merge):

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
  ## Chiamate in ingresso
</div>

Per impostazione predefinita, la policy per le chiamate in ingresso è `disabled`. Per abilitare le chiamate in ingresso, imposta:

```json5
{
  inboundPolicy: "allowlist",
  allowFrom: ["+15550001234"],
  inboundGreeting: "Ciao! Come posso aiutarti?"
}
```

Le risposte automatiche utilizzano il sistema degli agenti. Puoi ottimizzarle con:

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
  ## Strumento dell'Agente
</div>

Nome dello strumento: `voice_call`

Azioni:

- `initiate_call` (message, to?, mode?)
- `continue_call` (callId, message)
- `speak_to_user` (callId, message)
- `end_call` (callId)
- `get_status` (callId)

Questo repository include un documento relativo all'abilità corrispondente in `skills/voice-call/SKILL.md`.

<div id="gateway-rpc">
  ## RPC del Gateway
</div>

- `voicecall.initiate` (`to?`, `message`, `mode?`)
- `voicecall.continue` (`callId`, `message`)
- `voicecall.speak` (`callId`, `message`)
- `voicecall.end` (`callId`)
- `voicecall.status` (`callId`)