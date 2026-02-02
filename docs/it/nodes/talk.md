---
title: Modalità Talk
summary: "Modalità Talk: conversazioni vocali continue con ElevenLabs TTS"
read_when:
  - Implementare la modalità Talk su macOS/iOS/Android
  - Modificare la voce/TTS/il comportamento delle interruzioni
---

<div id="talk-mode">
  # Modalità Talk
</div>

La modalità Talk è un ciclo continuo di conversazione vocale:

1) Ascolta la voce
2) Invia la trascrizione al modello (sessione principale, chat.send)
3) Attendi la risposta
4) Pronunciala tramite ElevenLabs (riproduzione in streaming)

<div id="behavior-macos">
  ## Comportamento (macOS)
</div>

- **Overlay sempre visibile** mentre la modalità Talk è abilitata.
- Transizioni tra le fasi **Ascolto → Elaborazione → Parlato**.
- In caso di **breve pausa** (finestra di silenzio), viene inviata la trascrizione corrente.
- Le risposte vengono **scritte in WebChat** (come se fossero digitate).
- **Interruzione tramite voce** (attiva per impostazione predefinita): se l'utente inizia a parlare mentre l'assistente sta parlando, interrompiamo la riproduzione e annotiamo il timestamp dell'interruzione per il prompt successivo.

<div id="voice-directives-in-replies">
  ## Direttive vocali nelle risposte
</div>

L&#39;assistente può anteporre alla sua risposta una **singola riga JSON** per controllare la voce utilizzata:

```json
{"voice":"<voice-id>","once":true}
```

Regole:

* Viene considerata solo la prima riga non vuota.
* Le chiavi sconosciute vengono ignorate.
* `once: true` si applica solo alla risposta corrente.
* Senza `once`, la voce diventa il nuovo valore predefinito per la modalità Talk.
* La riga JSON viene eliminata prima della riproduzione TTS.

Chiavi supportate:

* `voice` / `voice_id` / `voiceId`
* `model` / `model_id` / `modelId`
* `speed`, `rate` (WPM), `stability`, `similarity`, `style`, `speakerBoost`
* `seed`, `normalize`, `lang`, `output_format`, `latency_tier`
* `once`


<div id="config-openclawopenclawjson">
  ## Configurazione (`~/.openclaw/openclaw.json`)
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

Predefiniti:

* `interruptOnSpeech`: true
* `voiceId`: se non impostato, assume `ELEVENLABS_VOICE_ID` / `SAG_VOICE_ID` (oppure la prima voce ElevenLabs quando la chiave API è disponibile)
* `modelId`: predefinito a `eleven_v3` se non impostato
* `apiKey`: se non impostato, assume `ELEVENLABS_API_KEY` (oppure il profilo shell del Gateway, se disponibile)
* `outputFormat`: predefinito a `pcm_44100` su macOS/iOS e `pcm_24000` su Android (imposta `mp3_*` per forzare lo streaming in MP3)


<div id="macos-ui">
  ## UI macOS
</div>

- Interruttore nella barra dei menu: **Talk**
- Scheda Configurazione: gruppo **Talk Mode** (ID voce + interruttore per l'interruzione)
- Overlay:
  - **Listening**: nuvola che pulsa in base al livello del microfono
  - **Thinking**: animazione di sprofondamento
  - **Speaking**: anelli radianti
  - Clic sulla nuvola: interrompe la risposta vocale
  - Clic su X: esci dalla modalità Talk

<div id="notes">
  ## Note
</div>

- Richiede le autorizzazioni per Voce + Microfono.
- Usa `chat.send` con la chiave di sessione `main`.
- Il TTS usa l’API di streaming ElevenLabs con `ELEVENLABS_API_KEY` e riproduzione incrementale su macOS/iOS/Android per ridurre la latenza.
- `stability` per `eleven_v3` viene convalidato ai valori `0.0`, `0.5` o `1.0`; altri modelli accettano `0..1`.
- `latency_tier` viene convalidato a `0..4` quando impostato.
- Android supporta i formati di output `pcm_16000`, `pcm_22050`, `pcm_24000` e `pcm_44100` per lo streaming a bassa latenza tramite AudioTrack.