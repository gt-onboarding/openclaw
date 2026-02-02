---
title: Audio
summary: "Come l'audio/le note vocali in arrivo vengono scaricati, trascritti e inseriti nelle risposte"
read_when:
  - Modificare la trascrizione audio o la gestione dei contenuti multimediali
---

<div id="audio-voice-notes-2026-01-17">
  # Audio / Messaggi vocali — 2026-01-17
</div>

<div id="what-works">
  ## Cosa funziona
</div>

* **Comprensione dei contenuti multimediali (audio)**: Se la comprensione audio è abilitata (o rilevata automaticamente), OpenClaw:
  1. Individua il primo allegato audio (percorso locale o URL) e lo scarica se necessario.
  2. Applica `maxBytes` prima di inviare a ciascuna entry di modello.
  3. Esegue in ordine la prima entry di modello idonea (provider o CLI).
  4. Se fallisce o viene saltata (dimensione/timeout), prova con l’entry successiva.
  5. In caso di successo, sostituisce `Body` con un blocco `[Audio]` e imposta `{{Transcript}}`.
* **Analisi dei comandi**: Quando la trascrizione riesce, `CommandBody`/`RawBody` vengono impostati alla trascrizione in modo che i comandi slash continuino a funzionare.
* **Logging dettagliato**: Con `--verbose`, viene registrato quando viene eseguita la trascrizione e quando sostituisce il body.

<div id="auto-detection-default">
  ## Rilevamento automatico (predefinito)
</div>

Se **non configuri i modelli** e `tools.media.audio.enabled` **non** è impostato su `false`,
OpenClaw esegue il rilevamento automatico in questo ordine e si ferma alla prima opzione funzionante:

1. **CLI locali** (se installate)
   * `sherpa-onnx-offline` (richiede `SHERPA_ONNX_MODEL_DIR` con encoder/decoder/joiner/tokens)
   * `whisper-cli` (da `whisper-cpp`; usa `WHISPER_CPP_MODEL` o il modello tiny incluso)
   * `whisper` (CLI Python; scarica automaticamente i modelli)
2. **CLI Gemini** (`gemini`) usando `read_many_files`
3. **Chiavi dei provider** (OpenAI → Groq → Deepgram → Google)

Per disabilitare il rilevamento automatico, imposta `tools.media.audio.enabled: false`.
Per personalizzare, imposta `tools.media.audio.models`.
Nota: Il rilevamento dei binari è su base best-effort su macOS/Linux/Windows; assicurati che la CLI sia nel `PATH` (espandiamo `~`), oppure imposta un modello CLI esplicito con il percorso completo del comando.

<div id="config-examples">
  ## Esempi di configurazione
</div>

<div id="provider-cli-fallback-openai-whisper-cli">
  ### Provider + CLI di fallback (OpenAI + Whisper CLI)
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
  ### Solo provider con limitazione tramite scope
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
  ### Solo provider: Deepgram
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
  ## Note e limiti
</div>

* L&#39;autenticazione del provider segue l&#39;ordine standard per l&#39;autenticazione dei modelli (profili di autenticazione, variabili d&#39;ambiente, `models.providers.*.apiKey`).
* Deepgram utilizza `DEEPGRAM_API_KEY` quando si usa `provider: "deepgram"`.
* Dettagli sulla configurazione di Deepgram: [Deepgram (audio transcription)](/it/providers/deepgram).
* I provider audio possono sovrascrivere `baseUrl`, `headers` e `providerOptions` tramite `tools.media.audio`.
* Il limite di dimensione predefinito è 20MB (`tools.media.audio.maxBytes`). L&#39;audio troppo grande viene ignorato per quel modello e si passa alla voce successiva.
* Il valore predefinito di `maxChars` per l&#39;audio è **non impostato** (trascrizione completa). Imposta `tools.media.audio.maxChars` o `maxChars` per singola voce per ridurre l&#39;output.
* Il modello predefinito automatico di OpenAI è `gpt-4o-mini-transcribe`; imposta `model: "gpt-4o-transcribe"` per una precisione maggiore.
* Usa `tools.media.audio.attachments` per elaborare più note vocali (`mode: "all"` + `maxAttachments`).
* La trascrizione è disponibile nei template come `{{Transcript}}`.
* Lo stdout della CLI è limitato (5MB); mantieni l&#39;output della CLI conciso.

<div id="gotchas">
  ## Note importanti
</div>

* Le regole di scope applicano la prima corrispondenza trovata. `chatType` viene normalizzato a `direct`, `group` o `room`.
* Assicurati che la tua CLI termini con codice di uscita 0 e produca testo semplice; il JSON va elaborato tramite `jq -r .text`.
* Mantieni i timeout ragionevoli (`timeoutSeconds`, predefinito 60s) per evitare di bloccare la coda delle risposte.