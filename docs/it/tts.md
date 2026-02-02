---
title: Sintesi vocale
summary: "Sintesi vocale (TTS) per le risposte in uscita"
read_when:
  - Abilitare la sintesi vocale per le risposte in uscita
  - Configurare i provider TTS o i relativi limiti
  - Utilizzare i comandi /tts
---

<div id="text-to-speech-tts">
  # Sintesi vocale (TTS)
</div>

OpenClaw può convertire le risposte in uscita in audio usando ElevenLabs, OpenAI o Edge TTS.
Funziona ovunque OpenClaw possa inviare audio; su Telegram viene generata la classica bolla rotonda di nota vocale.

<div id="supported-services">
  ## Servizi supportati
</div>

* **ElevenLabs** (provider primario o di fallback)
* **OpenAI** (provider primario o di fallback; usato anche per i riepiloghi)
* **Edge TTS** (provider primario o di fallback; utilizza `node-edge-tts`, predefinito quando non sono configurate chiavi API)

<div id="edge-tts-notes">
  ### Note su Edge TTS
</div>

Edge TTS utilizza il servizio TTS neurale online di Microsoft Edge tramite la
libreria `node-edge-tts`. È un servizio ospitato (non locale), usa gli endpoint
di Microsoft e non richiede alcuna chiave API. `node-edge-tts` espone opzioni di
configurazione della sintesi vocale e formati di output, ma non tutte le opzioni
sono supportate dal servizio Edge. citeturn2search0

Poiché Edge TTS è un servizio web pubblico senza un SLA o quote ufficiali pubblicate,
consideralo un servizio best effort. Se ti servono limiti garantiti e supporto, usa OpenAI
o ElevenLabs. La Speech REST API di Microsoft indica un limite di 10 minuti di
audio per richiesta; Edge TTS non pubblica limiti, quindi presumi limiti simili o
più bassi. citeturn0search3

<div id="optional-keys">
  ## Chiavi opzionali
</div>

Se vuoi usare OpenAI o ElevenLabs:

* `ELEVENLABS_API_KEY` (o `XI_API_KEY`)
* `OPENAI_API_KEY`

Edge TTS **non** richiede una chiave API. Se non viene trovata alcuna chiave API, OpenClaw utilizza
Edge TTS come impostazione predefinita (a meno che non sia disabilitato tramite `messages.tts.edge.enabled=false`).

Se sono configurati più provider, viene utilizzato prima il provider selezionato e gli altri fungono da opzioni di fallback.
Il riassunto automatico utilizza il `summaryModel` configurato (o `agents.defaults.model.primary`),
quindi anche quel provider deve essere autenticato se abiliti i riassunti.

<div id="service-links">
  ## Link ai servizi
</div>

* [Guida OpenAI Text-to-Speech](https://platform.openai.com/docs/guides/text-to-speech)
* [Riferimento API audio OpenAI](https://platform.openai.com/docs/api-reference/audio)
* [ElevenLabs Text to Speech](https://elevenlabs.io/docs/api-reference/text-to-speech)
* [Autenticazione ElevenLabs](https://elevenlabs.io/docs/api-reference/authentication)
* [node-edge-tts](https://github.com/SchneeHertz/node-edge-tts)
* [Formati di output per Microsoft Speech](https://learn.microsoft.com/azure/ai-services/speech-service/rest-text-to-speech#audio-outputs)

<div id="is-it-enabled-by-default">
  ## È abilitato per impostazione predefinita?
</div>

No. L&#39;Auto‑TTS è **disattivato** per impostazione predefinita. Abilitalo nella configurazione con
`messages.tts.auto` oppure per singola sessione con `/tts always` (alias: `/tts on`).

Edge TTS **è** abilitato per impostazione predefinita non appena il TTS è attivo e viene usato automaticamente
quando non sono disponibili chiavi API OpenAI o ElevenLabs.

<div id="config">
  ## Configurazione
</div>

La configurazione TTS si trova in `messages.tts` all&#39;interno di `openclaw.json`.
Lo schema completo è descritto in [Configurazione del Gateway](/it/gateway/configuration).

<div id="minimal-config-enable-provider">
  ### Configurazione minima (enable + provider)
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
  ### OpenAI come principale con ElevenLabs come fallback
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
  ### Edge TTS primario (senza chiave API)
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
  ### Disabilita Edge TTS
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
  ### Limiti personalizzati + percorso delle preferenze
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
  ### Rispondi con un messaggio audio solo dopo un messaggio vocale in arrivo
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
  ### Disattivare il riepilogo automatico per le risposte lunghe
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

Poi esegui:

```
/tts summary off
```

<div id="notes-on-fields">
  ### Note sui campi
</div>

* `auto`: modalità TTS automatica (`off`, `always`, `inbound`, `tagged`).
  * `inbound` invia l&#39;audio solo dopo una nota vocale ricevuta.
  * `tagged` invia l&#39;audio solo quando la risposta include i tag `[[tts]]`.
* `enabled`: interruttore legacy (doctor lo migra a `auto`).
* `mode`: `"final"` (predefinito) oppure `"all"` (include risposte di strumenti/blocchi).
* `provider`: `"elevenlabs"`, `"openai"` o `"edge"` (fallback automatico).
* Se `provider` non è **impostato**, OpenClaw preferisce `openai` (se presente la key), poi `elevenlabs` (se presente la key),
  altrimenti `edge`.
* `summaryModel`: modello a basso costo opzionale per l&#39;auto‑riepilogo; predefinito `agents.defaults.model.primary`.
  * Accetta `provider/model` oppure un alias di modello configurato.
* `modelOverrides`: consente al modello di emettere direttive TTS (attivo per impostazione predefinita).
* `maxTextLength`: limite massimo per l&#39;input TTS (caratteri). `/tts audio` fallisce se superato.
* `timeoutMs`: timeout della richiesta (ms).
* `prefsPath`: override del percorso JSON delle preferenze locali (provider/limite/riepilogo).
* I valori `apiKey` ricadono sulle variabili d&#39;ambiente come fallback (`ELEVENLABS_API_KEY`/`XI_API_KEY`, `OPENAI_API_KEY`).
* `elevenlabs.baseUrl`: override della base URL dell&#39;API ElevenLabs.
* `elevenlabs.voiceSettings`:
  * `stability`, `similarityBoost`, `style`: `0..1`
  * `useSpeakerBoost`: `true|false`
  * `speed`: `0.5..2.0` (1.0 = normale)
* `elevenlabs.applyTextNormalization`: `auto|on|off`
* `elevenlabs.languageCode`: codice ISO 639-1 a 2 lettere (es. `en`, `de`)
* `elevenlabs.seed`: intero `0..4294967295` (determinismo best‑effort)
* `edge.enabled`: consente l&#39;uso di Edge TTS (predefinito `true`; nessuna API key).
* `edge.voice`: nome della voce neurale Edge (es. `en-US-MichelleNeural`).
* `edge.lang`: codice lingua (es. `en-US`).
* `edge.outputFormat`: formato di output Edge (es. `audio-24khz-48kbitrate-mono-mp3`).
  * Consulta Microsoft Speech output formats per i valori validi; non tutti i formati sono supportati da Edge.
* `edge.rate` / `edge.pitch` / `edge.volume`: stringhe percentuali (es. `+10%`, `-5%`).
* `edge.saveSubtitles`: scrive i sottotitoli JSON insieme al file audio.
* `edge.proxy`: URL del proxy per le richieste Edge TTS.
* `edge.timeoutMs`: override del timeout della richiesta (ms).

<div id="model-driven-overrides-default-on">
  ## Override basati sul modello (attivi per impostazione predefinita)
</div>

Per impostazione predefinita, il modello **può** emettere direttive TTS per una singola risposta.
Quando `messages.tts.auto` è `tagged`, queste direttive sono necessarie per attivare l&#39;audio.

Quando è abilitato, il modello può emettere direttive `[[tts:...]]` per sovrascrivere la voce
per una singola risposta, oltre a un blocco opzionale `[[tts:text]]...[[/tts:text]]` per
fornire tag espressivi (risate, indicazioni per il canto, ecc.) che devono apparire solo
nell&#39;audio.

Esempio di payload di risposta:

```
Ecco fatto.

[[tts:provider=elevenlabs voiceId=pMsXgVXv3BLzUgSXRplE model=eleven_v3 speed=1.1]]
[[tts:text]](ride) Read la canzone ancora una volta.[[/tts:text]]
```

Chiavi di direttiva disponibili (se abilitate):

* `provider` (`openai` | `elevenlabs` | `edge`)
* `voice` (voce OpenAI) oppure `voiceId` (ElevenLabs)
* `model` (modello TTS OpenAI o id modello ElevenLabs)
* `stability`, `similarityBoost`, `style`, `speed`, `useSpeakerBoost`
* `applyTextNormalization` (`auto|on|off`)
* `languageCode` (ISO 639-1)
* `seed`

Disabilita tutti gli override di modello:

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

Lista di autorizzati opzionale (disattiva override specifici mantenendo abilitati i tag):

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
  ## Preferenze per singolo utente
</div>

I comandi slash registrano override locali in `prefsPath` (predefinito:
`~/.openclaw/settings/tts.json`, sovrascrivibile con `OPENCLAW_TTS_PREFS` o
`messages.tts.prefsPath`).

Campi salvati:

* `enabled`
* `provider`
* `maxLength` (soglia per il riassunto; predefinito 1500 caratteri)
* `summarize` (predefinito `true`)

Questi valori sovrascrivono `messages.tts.*` per quell&#39;host.

<div id="output-formats-fixed">
  ## Formati di output (fissi)
</div>

* **Telegram**: nota vocale Opus (`opus_48000_64` da ElevenLabs, `opus` da OpenAI).
  * 48kHz / 64kbps è un buon compromesso per le note vocali ed è richiesto per la bolla rotonda.
* **Altri canali**: MP3 (`mp3_44100_128` da ElevenLabs, `mp3` da OpenAI).
  * 44.1kHz / 128kbps è il compromesso predefinito per la chiarezza della voce.
* **Edge TTS**: usa `edge.outputFormat` (predefinito `audio-24khz-48kbitrate-mono-mp3`).
  * `node-edge-tts` accetta un `outputFormat`, ma non tutti i formati sono disponibili
    dal servizio Edge. citeturn2search0
  * I valori del formato di output seguono i formati di output Microsoft Speech (inclusi Ogg/WebM Opus). citeturn1search0
  * Telegram `sendVoice` accetta OGG/MP3/M4A; usa OpenAI/ElevenLabs se ti servono
    note vocali Opus garantite. citeturn1search1
  * Se il formato di output Edge configurato non funziona, OpenClaw riprova con MP3.

I formati OpenAI/ElevenLabs sono fissi; Telegram si aspetta Opus per l’UX delle note vocali.

<div id="auto-tts-behavior">
  ## Comportamento Auto-TTS
</div>

Quando Auto-TTS è abilitato, OpenClaw:

* non esegue il TTS se la risposta contiene già contenuti multimediali o una direttiva `MEDIA:`.
* non esegue il TTS per le risposte molto brevi (&lt; 10 caratteri).
* riassume le risposte lunghe, quando questa opzione è attivata, usando `agents.defaults.model.primary` (o `summaryModel`).
* allega l&#39;audio generato alla risposta.

Se la risposta supera `maxLength` e la funzione di riepilogo è disattivata (o non è presente alcuna chiave API per il
modello di riepilogo), l&#39;audio
viene omesso e viene inviata la normale risposta testuale.

<div id="flow-diagram">
  ## Diagramma di flusso
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
  ## Utilizzo del comando slash
</div>

C’è un unico comando: `/tts`.
Consulta [Comandi slash](/it/tools/slash-commands) per i dettagli su come abilitarlo.

Nota per Discord: `/tts` è un comando integrato di Discord, quindi OpenClaw registra
`/voice` come comando nativo in Discord. L’invocazione `/tts ...` continua comunque a funzionare.

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

Note:

* I comandi richiedono un mittente autorizzato (le regole di lista di autorizzati/proprietario si applicano comunque).
* `commands.text` o la registrazione nativa dei comandi devono essere abilitate.
* `off|always|inbound|tagged` sono impostazioni per sessione (`/tts on` è un alias di `/tts always`).
* `limit` e `summary` sono memorizzati nelle preferenze locali, non nella configurazione principale.
* `/tts audio` genera una risposta audio una tantum (non attiva il TTS).

<div id="agent-tool">
  ## Strumento dell&#39;agente
</div>

Lo strumento `tts` converte il testo in parlato e restituisce un percorso `MEDIA:`. Quando il
risultato è compatibile con Telegram, lo strumento include `[[audio_as_voice]]` in modo che
Telegram lo invii come messaggio vocale.

<div id="gateway-rpc">
  ## Gateway RPC
</div>

Metodi del Gateway:

* `tts.status`
* `tts.enable`
* `tts.disable`
* `tts.convert`
* `tts.setProvider`
* `tts.providers`