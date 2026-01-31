---
title: Audio
summary: "Wie eingehende Audio-/Sprachnotizen heruntergeladen, transkribiert und in Antworten eingefügt werden"
read_when:
  - Bei Änderungen an der Audio-Transkription oder Medienverarbeitung
---

<div id="audio-voice-notes-2026-01-17">
  # Audio-/Sprachnotizen — 2026-01-17
</div>

<div id="what-works">
  ## Was funktioniert
</div>

* **Medienverständnis (Audio)**: Wenn Audioverständnis aktiviert (oder automatisch erkannt) ist, führt OpenClaw Folgendes aus:
  1. Ermittelt den ersten Audio‑Anhang (lokaler Pfad oder URL) und lädt ihn bei Bedarf herunter.
  2. Erzwingt `maxBytes`, bevor jeder Modelleintrag gesendet wird.
  3. Führt den ersten geeigneten Modelleintrag in der Reihenfolge aus (Anbieter oder CLI).
  4. Wenn dieser fehlschlägt oder übersprungen wird (Größe/Timeout), wird der nächste Eintrag ausprobiert.
  5. Bei Erfolg ersetzt es `Body` durch einen `[Audio]`‑Block und setzt `{{Transcript}}`.
* **Befehlserkennung**: Wenn die Transkription erfolgreich ist, werden `CommandBody`/`RawBody` auf das Transkript gesetzt, sodass Slash‑Befehle weiterhin funktionieren.
* **Ausführliches Logging**: In `--verbose` wird protokolliert, wann die Transkription ausgeführt wird und wann sie den Body ersetzt.

<div id="auto-detection-default">
  ## Auto-Erkennung (Standard)
</div>

Wenn du **keine Modelle konfigurierst** und `tools.media.audio.enabled` **nicht** auf `false` gesetzt ist,
führt OpenClaw eine Auto-Erkennung in dieser Reihenfolge durch und bricht bei der ersten funktionierenden Option ab:

1. **Lokale CLIs** (falls installiert)
   * `sherpa-onnx-offline` (erfordert `SHERPA_ONNX_MODEL_DIR` mit encoder/decoder/joiner/tokens)
   * `whisper-cli` (aus `whisper-cpp`; verwendet `WHISPER_CPP_MODEL` oder das mitgelieferte Tiny-Modell)
   * `whisper` (Python-CLI; lädt Modelle automatisch herunter)
2. **Gemini CLI** (`gemini`) mit `read_many_files`
3. **Provider-Keys** (OpenAI → Groq → Deepgram → Google)

Um die Auto-Erkennung zu deaktivieren, setze `tools.media.audio.enabled: false`.
Zum Anpassen setze `tools.media.audio.models`.
Hinweis: Die Erkennung der Binaries ist eine Best-Effort-Heuristik für macOS/Linux/Windows; stelle sicher, dass sich die CLI im `PATH` befindet (wir expandieren `~`), oder setze ein explizites CLI-Modell mit einem vollständigen Befehls-Pfad.

<div id="config-examples">
  ## Konfigurationsbeispiele
</div>

<div id="provider-cli-fallback-openai-whisper-cli">
  ### Anbieter + CLI-Fallback (OpenAI + Whisper CLI)
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
  ### Nur Anbieter mit Scope-Gating
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
  ### Nur als Anbieter (Deepgram)
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
  ## Hinweise &amp; Beschränkungen
</div>

* Die Anbieter-Authentifizierung folgt der standardmäßigen Reihenfolge für die Modell-Authentifizierung (Auth-Profile, Umgebungsvariablen, `models.providers.*.apiKey`).
* Deepgram verwendet `DEEPGRAM_API_KEY`, wenn `provider: "deepgram"` gesetzt ist.
* Details zur Deepgram-Einrichtung: [Deepgram (audio transcription)](/de/providers/deepgram).
* Audio-Anbieter können `baseUrl`, `headers` und `providerOptions` über `tools.media.audio` überschreiben.
* Das Standardlimit für die Dateigröße beträgt 20MB (`tools.media.audio.maxBytes`). Zu große Audiodateien werden für dieses Modell übersprungen und der nächste Eintrag wird ausprobiert.
* Standard-`maxChars` für Audio ist **nicht gesetzt** (vollständiges Transkript). Setze `tools.media.audio.maxChars` oder `maxChars` pro Eintrag, um die Ausgabe zu kürzen.
* Der automatische Standard bei OpenAI ist `gpt-4o-mini-transcribe`; setze `model: "gpt-4o-transcribe"` für höhere Genauigkeit.
* Verwende `tools.media.audio.attachments`, um mehrere Sprachnachrichten zu verarbeiten (`mode: "all"` + `maxAttachments`).
* Das Transkript ist in Templates als `{{Transcript}}` verfügbar.
* CLI-stdout ist auf 5MB begrenzt; halte die CLI-Ausgabe knapp.

<div id="gotchas">
  ## Stolperfallen
</div>

* Scope-Regeln verwenden das Prinzip „first match wins“. `chatType` wird auf `direct`, `group` oder `room` normalisiert.
* Stellen Sie sicher, dass Ihr CLI mit Exit-Code 0 beendet wird und Klartext ausgibt; JSON muss über `jq -r .text` aufbereitet werden.
* Halten Sie Timeouts angemessen (`timeoutSeconds`, Standard 60s), um das Blockieren der Reply-Queue zu vermeiden.