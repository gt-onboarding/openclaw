---
title: Talk
summary: "Talk-Modus: fortlaufende Sprachdialoge mit ElevenLabs TTS"
read_when:
  - Implementierung des Talk-Modus auf macOS/iOS/Android
  - Anpassen von Stimme/TTS/Unterbrechungsverhalten
---

<div id="talk-mode">
  # Talk-Modus
</div>

Der Talk-Modus ist eine kontinuierliche Sprachdialogschleife:

1) Spracheingaben abhören
2) Transkript an das Modell senden (Hauptsitzung, chat.send)
3) Auf die Antwort warten
4) Antwort über ElevenLabs ausgeben (Streaming-Wiedergabe)

<div id="behavior-macos">
  ## Verhalten (macOS)
</div>

- **Ständig eingeblendetes Overlay**, solange der Talk-Modus aktiviert ist.
- Phasenübergänge von **Zuhören → Denken → Sprechen**.
- Bei einer **kurzen Pause** (Stillefenster) wird das aktuelle Transkript gesendet.
- Antworten werden **in WebChat geschrieben** (entspricht Tippen).
- **Unterbrechen bei Spracheingabe** (standardmäßig aktiviert): Wenn der Benutzer zu sprechen beginnt, während der Assistent spricht, stoppen wir die Wiedergabe und notieren den Zeitstempel der Unterbrechung für den nächsten Prompt.

<div id="voice-directives-in-replies">
  ## Sprachanweisungen in Antworten
</div>

Der Assistent kann seiner Antwort **eine einzelne JSON-Zeile** voranstellen, um die Stimme zu steuern:

```json
{"voice":"<voice-id>","once":true}
```

Regeln:

* Es zählt nur die erste nicht-leere Zeile.
* Unbekannte Schlüssel werden ignoriert.
* `once: true` gilt nur für die aktuelle Antwort.
* Ohne `once` wird die Stimme zum neuen Standard für den Talk-Modus.
* Die JSON-Zeile wird vor der TTS-Wiedergabe entfernt.

Unterstützte Schlüssel:

* `voice` / `voice_id` / `voiceId`
* `model` / `model_id` / `modelId`
* `speed`, `rate` (WPM), `stability`, `similarity`, `style`, `speakerBoost`
* `seed`, `normalize`, `lang`, `output_format`, `latency_tier`
* `once`


<div id="config-openclawopenclawjson">
  ## Konfiguration (`~/.openclaw/openclaw.json`)
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

Standardwerte:

* `interruptOnSpeech`: true
* `voiceId`: fällt auf `ELEVENLABS_VOICE_ID` / `SAG_VOICE_ID` zurück (oder auf die erste ElevenLabs-Stimme, wenn der API-Schlüssel verfügbar ist)
* `modelId`: standardmäßig `eleven_v3`, wenn nicht gesetzt
* `apiKey`: fällt auf `ELEVENLABS_API_KEY` zurück (oder auf das Gateway-Shell-Profil, falls verfügbar)
* `outputFormat`: standardmäßig `pcm_44100` auf macOS/iOS und `pcm_24000` auf Android (setze `mp3_*`, um MP3-Streaming zu erzwingen)


<div id="macos-ui">
  ## macOS-UI
</div>

- Schalter in der Menüleiste: **Talk**
- Tab „Config“: Gruppe **Talk Mode** (Voice-ID + Schalter zum Unterbrechen)
- Overlay:
  - **Listening**: Wolke pulsiert mit Mikrofonpegel
  - **Thinking**: sich absenkende Animation
  - **Speaking**: ausstrahlende Ringe
  - Wolke anklicken: Sprachausgabe beenden
  - X anklicken: Talk Mode beenden

<div id="notes">
  ## Hinweise
</div>

- Erfordert Berechtigungen für Sprache + Mikrofon.
- Verwendet `chat.send` für den Sitzungsschlüssel `main`.
- TTS verwendet die ElevenLabs-Streaming-API mit `ELEVENLABS_API_KEY` und inkrementeller Wiedergabe auf macOS/iOS/Android für geringere Latenz.
- `stability` für `eleven_v3` wird auf `0.0`, `0.5` oder `1.0` geprüft; andere Modelle akzeptieren `0..1`.
- `latency_tier` wird auf `0..4` geprüft, falls gesetzt.
- Android unterstützt die Ausgabeformate `pcm_16000`, `pcm_22050`, `pcm_24000` und `pcm_44100` für niedriglatenzfähiges AudioTrack-Streaming.