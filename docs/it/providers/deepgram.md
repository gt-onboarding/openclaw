---
title: Deepgram
summary: "Trascrizione Deepgram per note vocali in arrivo"
read_when:
  - Vuoi usare la trascrizione automatica di Deepgram per gli allegati audio
  - Ti serve un esempio rapido di configurazione di Deepgram
---

<div id="deepgram-audio-transcription">
  # Deepgram (Trascrizione Audio)
</div>

Deepgram è una API di riconoscimento vocale (speech-to-text). In OpenClaw viene usata per la **trascrizione in ingresso di audio/note vocali**
tramite `tools.media.audio`.

Quando è abilitato, OpenClaw carica il file audio su Deepgram e inserisce la trascrizione
nella pipeline di risposta (blocco `{{Transcript}}` + `[Audio]`). Questo **non è in streaming**:
usa l'endpoint di trascrizione per audio preregistrato.

Sito web: https://deepgram.com  
Documentazione: https://developers.deepgram.com

<div id="quick-start">
  ## Avvio rapido
</div>

1. Imposta la tua chiave API:

```
DEEPGRAM_API_KEY=dg_...
```

2. Attiva il provider:

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
  ## Opzioni
</div>

* `model`: ID del modello Deepgram (predefinito: `nova-3`)
* `language`: suggerimento per la lingua (opzionale)
* `tools.media.audio.providerOptions.deepgram.detect_language`: abilita il rilevamento della lingua (opzionale)
* `tools.media.audio.providerOptions.deepgram.punctuate`: abilita la punteggiatura automatica (opzionale)
* `tools.media.audio.providerOptions.deepgram.smart_format`: abilita la formattazione intelligente (opzionale)

Esempio con specifica della lingua:

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

Esempio con le opzioni di Deepgram:

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
  ## Note
</div>

- L'autenticazione segue l'ordine standard di auth del provider; `DEEPGRAM_API_KEY` è l'opzione più semplice.
- Quando usi un proxy, puoi sovrascrivere endpoint e header con `tools.media.audio.baseUrl` e `tools.media.audio.headers`.
- L'output segue le stesse regole audio degli altri provider (limiti di dimensione, timeout, injection della trascrizione).