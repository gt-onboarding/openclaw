---
title: Comprensione dei contenuti multimediali
summary: "Comprensione di immagini/audio/video in ingresso (opzionale) con fallback su provider e CLI"
read_when:
  - Progettazione o refactoring della comprensione dei contenuti multimediali
  - Messa a punto della pre-elaborazione di audio/video/immagini in ingresso
---

<div id="media-understanding-inbound-2026-01-17">
  # Comprensione dei media (in ingresso) — 2026-01-17
</div>

OpenClaw può **riassumere i media in ingresso** (immagini/audio/video) prima dell’esecuzione della pipeline di risposta. Rileva automaticamente quando sono disponibili strumenti locali o chiavi provider e può essere disattivata o configurata. Se la comprensione è disattivata, i modelli continuano comunque a ricevere i file/URL originali come al solito.

<div id="goals">
  ## Obiettivi
</div>

* Opzionale: pre‑elaborare i contenuti multimediali in ingresso in testo compatto, per un instradamento più rapido e un parsing dei comandi migliore.
* Preservare sempre l&#39;invio dei contenuti multimediali originali al modello.
* Supportare le **API dei provider** e i **fallback via CLI**.
* Consentire l&#39;uso di più modelli con fallback ordinato (errore/dimensione/timeout).

<div id="highlevel-behavior">
  ## Comportamento generale
</div>

1. Raccogli gli allegati in ingresso (`MediaPaths`, `MediaUrls`, `MediaTypes`).
2. Per ciascuna capacità abilitata (immagine/audio/video), seleziona gli allegati in base alla policy (predefinito: **primo**).
3. Scegli la prima voce di modello idonea (dimensione + capacità + autenticazione).
4. Se un modello fallisce o il contenuto multimediale è troppo grande, **passa alla voce successiva**.
5. In caso di successo:
   * `Body` diventa un blocco `[Image]`, `[Audio]` o `[Video]`.
   * L&#39;audio imposta `{{Transcript}}`; il parsing dei comandi usa il testo della didascalia quando presente,
     altrimenti la trascrizione.
   * Le didascalie vengono mantenute come `User text:` all&#39;interno del blocco.

Se la comprensione fallisce o è disabilitata, **il flusso di risposta continua** con il `Body` originale + allegati.

<div id="config-overview">
  ## Panoramica della configurazione
</div>

`tools.media` supporta **modelli condivisi** più override per singola capability:

* `tools.media.models`: elenco di modelli condivisi (usa `capabilities` per controllarne l&#39;uso).
* `tools.media.image` / `tools.media.audio` / `tools.media.video`:
  * valori predefiniti (`prompt`, `maxChars`, `maxBytes`, `timeoutSeconds`, `language`)
  * override del provider (`baseUrl`, `headers`, `providerOptions`)
  * opzioni audio Deepgram tramite `tools.media.audio.providerOptions.deepgram`
  * **elenco di `models` per capability** opzionale (preferito rispetto ai modelli condivisi)
  * policy `attachments` (`mode`, `maxAttachments`, `prefer`)
  * `scope` (vincolo opzionale basato su channel/chatType/session key)
* `tools.media.concurrency`: numero massimo di esecuzioni concorrenti per capability (predefinito **2**).

```json5
{
  tools: {
    media: {
      models: [ /* lista condivisa */ ],
      image: { /* optional overrides */ },
      audio: { /* optional overrides */ },
      video: { /* optional overrides */ }
    }
  }
}
```

<div id="model-entries">
  ### Voci del modello
</div>

Ogni voce di `models[]` può essere un **provider** o la **CLI**:

```json5
{
  type: "provider",        // default if omitted
  provider: "openai",
  model: "gpt-5.2",
  prompt: "Describe the image in <= 500 chars.",
  maxChars: 500,
  maxBytes: 10485760,
  timeoutSeconds: 60,
  capabilities: ["image"], // opzionale, utilizzato per voci multi‑modali
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

I template della CLI possono usare anche:

* `{{MediaDir}}` (directory contenente il file multimediale)
* `{{OutputDir}}` (directory di lavoro temporanea creata per questa esecuzione)
* `{{OutputBase}}` (percorso base del file di lavoro temporaneo, senza estensione)

<div id="defaults-and-limits">
  ## Valori predefiniti e limiti
</div>

Impostazioni predefinite consigliate:

* `maxChars`: **500** per immagini/video (brevi, adatte ai comandi)
* `maxChars`: **non definito** per audio (trascrizione completa a meno che tu non imponga un limite)
* `maxBytes`:
  * immagini: **10MB**
  * audio: **20MB**
  * video: **50MB**

Regole:

* Se il contenuto multimediale supera `maxBytes`, quel modello viene saltato e si prova il **modello successivo**.
* Se il modello restituisce più di `maxChars`, l&#39;output viene troncato.
* Il valore predefinito di `prompt` è la semplice frase “Describe the {media}.” più l&#39;indicazione di `maxChars` (solo per immagini/video).
* Se `<capability>.enabled: true` ma non sono configurati modelli, OpenClaw prova a usare il
  **modello di risposta attivo** quando il suo provider supporta la funzionalità.

<div id="auto-detect-media-understanding-default">
  ### Rilevamento automatico della comprensione dei media (predefinito)
</div>

Se `tools.media.<capability>.enabled` **non** è impostato su `false` e non hai
configurato alcun modello, OpenClaw esegue il rilevamento automatico in quest’ordine e **si ferma alla prima
opzione funzionante**:

1. **CLI locali** (solo audio; se installato)
   * `sherpa-onnx-offline` (richiede `SHERPA_ONNX_MODEL_DIR` con encoder/decoder/joiner/tokens)
   * `whisper-cli` (`whisper-cpp`; usa `WHISPER_CPP_MODEL` o il modello tiny incluso)
   * `whisper` (CLI Python; scarica automaticamente i modelli)
2. **Gemini CLI** (`gemini`) usando `read_many_files`
3. **Chiavi dei provider**
   * Audio: OpenAI → Groq → Deepgram → Google
   * Immagini: OpenAI → Anthropic → Google → MiniMax
   * Video: Google

Per disabilitare il rilevamento automatico, imposta:

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

Nota: il rilevamento dell&#39;eseguibile è basato sul massimo sforzo possibile su macOS/Linux/Windows; assicurati che la CLI sia nel `PATH` (espandiamo `~`) oppure imposta esplicitamente un modello CLI con un percorso di comando completo.

<div id="capabilities-optional">
  ## Functionalità (facoltativo)
</div>

Se imposti `capabilities`, la voce viene eseguita solo per quei tipi di contenuti multimediali. Per le liste condivise, OpenClaw può inferire automaticamente i valori predefiniti:

* `openai`, `anthropic`, `minimax`: **image**
* `google` (Gemini API): **image + audio + video**
* `groq`: **audio**
* `deepgram`: **audio**

Per le voci della CLI, **imposta `capabilities` in modo esplicito** per evitare corrispondenze inattese.
Se ometti `capabilities`, la voce è idonea per la lista in cui compare.

<div id="provider-support-matrix-openclaw-integrations">
  ## Matrice di supporto dei provider (integrazioni OpenClaw)
</div>

| Funzionalità | Integrazione provider | Note |
|--------------|-----------------------|------|
| Immagine | OpenAI / Anthropic / Google / altri tramite `pi-ai` | Qualsiasi modello con capacità di elaborazione immagini presente nel registro funziona. |
| Audio | OpenAI, Groq, Deepgram, Google | Trascrizione fornita dal provider (Whisper/Deepgram/Gemini). |
| Video | Google (Gemini API) | Comprensione video fornita dal provider. |

<div id="recommended-providers">
  ## Provider consigliati
</div>

**Immagine**

* Preferisci il modello attivo se supporta le immagini.
* Buoni valori predefiniti: `openai/gpt-5.2`, `anthropic/claude-opus-4-5`, `google/gemini-3-pro-preview`.

**Audio**

* `openai/gpt-4o-mini-transcribe`, `groq/whisper-large-v3-turbo` o `deepgram/nova-3`.
* Fallback via CLI: `whisper-cli` (whisper-cpp) o `whisper`.
* Configurazione di Deepgram: [Deepgram (trascrizione audio)](/it/providers/deepgram).

**Video**

* `google/gemini-3-flash-preview` (veloce), `google/gemini-3-pro-preview` (più ricco).
* Fallback via CLI: `gemini` CLI (supporta `read_file` su video/audio).

<div id="attachment-policy">
  ## Criteri per gli allegati
</div>

Il campo `attachments` a livello di singola funzionalità controlla quali allegati vengono elaborati:

* `mode`: `first` (predefinito) oppure `all`
* `maxAttachments`: limita il numero elaborato (predefinito **1**)
* `prefer`: `first`, `last`, `path`, `url`

Quando `mode: "all"`, gli output sono etichettati come `[Image 1/2]`, `[Audio 2/2]`, ecc.

<div id="config-examples">
  ## Esempi di configurazione
</div>

<div id="1-shared-models-list-overrides">
  ### 1) Elenco dei modelli condivisi + override
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
            "Read the media at {{MediaPath}} and describe it in <= {{MaxChars}} characters."
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
  ### 2) Solo audio e video (immagine disattivata)
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
  ### 3) Analisi opzionale delle immagini
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
              "Leggi il media in {{MediaPath}} e descrivilo in <= {{MaxChars}} caratteri."
            ]
          }
        ]
      }
    }
  }
}
```

<div id="4-multimodal-single-entry-explicit-capabilities">
  ### 4) Ingresso unico multimodale (funzionalità esplicite)
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
  ## Output dello stato
</div>

Quando l&#39;analisi dei contenuti multimediali è in esecuzione, `/status` include una breve riga di riepilogo:

```
📎 Media: image ok (openai/gpt-5.2) · audio skipped (maxBytes)
```

Questo mostra gli esiti per singola capacità e il provider/modello scelto, quando applicabile.

<div id="notes">
  ## Note
</div>

* La comprensione è su base **best‑effort**. Gli errori non bloccano le risposte.
* Gli allegati vengono comunque passati ai modelli anche quando la comprensione è disabilitata.
* Usa `scope` per limitare dove viene eseguita la comprensione (ad es. solo messaggi diretti/DM).

<div id="related-docs">
  ## Documentazione correlata
</div>

* [Configurazione](/it/gateway/configuration)
* [Supporto per immagini e contenuti multimediali](/it/nodes/images)