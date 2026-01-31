---
title: Deepgram
summary: "Deepgram-Transkription für eingehende Sprachnachrichten"
read_when:
  - Du möchtest Deepgram-Speech-to-Text für Audioanhänge verwenden
  - Du brauchst ein kurzes Deepgram-Konfigurationsbeispiel
---

<div id="deepgram-audio-transcription">
  # Deepgram (Audio Transcription)
</div>

Deepgram ist eine Speech-to-Text-API. In OpenClaw wird sie für die **Transkription eingehender Audio-/Sprachnachrichten** über `tools.media.audio` verwendet.

Wenn aktiviert, lädt OpenClaw die Audiodatei zu Deepgram hoch und fügt das Transkript
in die Antwort-Pipeline ein (`{{Transcript}}` + `[Audio]`-Block). Dies ist **kein Streaming**;
es nutzt den Endpunkt für die Transkription vorab aufgezeichneter Audiodaten.

Website: https://deepgram.com  
Docs: https://developers.deepgram.com

<div id="quick-start">
  ## Schnellstart
</div>

1. Setzen Sie Ihren API-Schlüssel:

```
DEEPGRAM_API_KEY=dg_...
```

2. Aktivieren Sie den anbieter:

```json5
{
  tools: {
    media: {
      audio: {
        enabled: true, // aktiviert
        models: [{ provider: "deepgram", model: "nova-3" }]
      }
    }
  }
}
```


<div id="options">
  ## Optionen
</div>

* `model`: Deepgram-Modell-ID (Standard: `nova-3`)
* `language`: Sprachhinweis (optional)
* `tools.media.audio.providerOptions.deepgram.detect_language`: Spracherkennung aktivieren (optional)
* `tools.media.audio.providerOptions.deepgram.punctuate`: Zeichensetzung aktivieren (optional)
* `tools.media.audio.providerOptions.deepgram.smart_format`: intelligente Formatierung aktivieren (optional)

Beispiel mit Sprachangabe:

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

Beispiel für Deepgram-Optionen:

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
  ## Hinweise
</div>

- Die Authentifizierung folgt der Standard-Reihenfolge für die Anbieter-Authentifizierung; `DEEPGRAM_API_KEY` ist der einfachste Weg.
- Überschreibe Endpunkte oder Header mit `tools.media.audio.baseUrl` und `tools.media.audio.headers`, wenn du einen Proxy verwendest.
- Die Ausgabe folgt denselben Audioregeln wie bei anderen Anbietern (Größenlimits, Timeouts, Transkript-Injektion).