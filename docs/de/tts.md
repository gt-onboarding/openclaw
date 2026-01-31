---
title: TTS
summary: "Text-to-Speech (TTS) für ausgehende Antworten"
read_when:
  - Text-to-Speech für Antworten aktivieren
  - TTS-Anbieter oder Grenzwerte konfigurieren
  - Verwendung von /tts-Befehlen
---

<div id="text-to-speech-tts">
  # Text-to-speech (TTS)
</div>

OpenClaw kann ausgehende Antworten mit ElevenLabs, OpenAI oder Edge TTS in Audioausgabe umwandeln.
Das funktioniert überall dort, wo OpenClaw Audio senden kann; in Telegram erscheint das als runde Sprachnachrichten-Blase.

<div id="supported-services">
  ## Unterstützte Dienste
</div>

* **ElevenLabs** (primärer oder Fallback-Anbieter)
* **OpenAI** (primärer oder Fallback-Anbieter; wird auch für Zusammenfassungen verwendet)
* **Edge TTS** (primärer oder Fallback-Anbieter; verwendet `node-edge-tts`, Standard, wenn keine API-Schlüssel vorhanden sind)

<div id="edge-tts-notes">
  ### Edge-TTS-Hinweise
</div>

Edge TTS verwendet den neuronalen Online-TTS-Dienst von Microsoft Edge über die
`node-edge-tts`-Bibliothek. Es ist ein gehosteter Dienst (nicht lokal), nutzt
Microsoft-Endpunkte und erfordert keinen API-Schlüssel. `node-edge-tts` stellt
Sprachkonfigurationsoptionen und Ausgabeformate bereit, aber nicht alle Optionen
werden vom Edge-Dienst unterstützt. citeturn2search0

Da Edge TTS ein öffentlicher Webdienst ohne veröffentlichte SLAs oder Quoten ist,
solltest du ihn nur als Best-Effort-Dienst einstufen. Wenn du garantierte
Grenzwerte und Support benötigst, verwende OpenAI oder ElevenLabs. Microsofts
Speech-REST-API dokumentiert ein Audio-Limit von 10 Minuten pro Anfrage; Edge TTS
veröffentlicht keine Limits, daher solltest du von ähnlichen oder niedrigeren
Grenzwerten ausgehen. citeturn0search3

<div id="optional-keys">
  ## Optionale Schlüssel
</div>

Wenn du OpenAI oder ElevenLabs verwenden möchtest:

* `ELEVENLABS_API_KEY` (oder `XI_API_KEY`)
* `OPENAI_API_KEY`

Edge TTS benötigt **keinen** API-Schlüssel. Wenn keine API-Schlüssel gefunden werden, verwendet OpenClaw standardmäßig
Edge TTS (sofern nicht über `messages.tts.edge.enabled=false` deaktiviert).

Wenn mehrere Anbieter konfiguriert sind, wird zuerst der ausgewählte Anbieter verwendet, und die anderen dienen als Fallback-Optionen.
Die automatische Zusammenfassung verwendet das konfigurierte `summaryModel` (oder `agents.defaults.model.primary`),
daher muss dieser Anbieter ebenfalls authentifiziert sein, wenn du Zusammenfassungen aktivierst.

<div id="service-links">
  ## Service-Links
</div>

* [OpenAI-Leitfaden für Text-to-Speech](https://platform.openai.com/docs/guides/text-to-speech)
* [OpenAI Audio-API-Referenz](https://platform.openai.com/docs/api-reference/audio)
* [ElevenLabs Text to Speech](https://elevenlabs.io/docs/api-reference/text-to-speech)
* [ElevenLabs-Authentifizierung](https://elevenlabs.io/docs/api-reference/authentication)
* [node-edge-tts](https://github.com/SchneeHertz/node-edge-tts)
* [Microsoft Sprachausgabeformate](https://learn.microsoft.com/azure/ai-services/speech-service/rest-text-to-speech#audio-outputs)

<div id="is-it-enabled-by-default">
  ## Ist es standardmäßig aktiviert?
</div>

Nein. Auto‑TTS ist standardmäßig **aus**. Aktiviere es in der Konfiguration mit
`messages.tts.auto` oder pro Sitzung mit `/tts always` (Alias: `/tts on`).

Edge TTS **ist** standardmäßig aktiviert, sobald TTS eingeschaltet ist, und wird automatisch verwendet,
wenn keine OpenAI‑ oder ElevenLabs‑API‑Schlüssel verfügbar sind.

<div id="config">
  ## Konfiguration
</div>

Die TTS-Konfiguration steht unter `messages.tts` in `openclaw.json`.
Das vollständige Schema findest du unter [Gateway-Konfiguration](/de/gateway/configuration).

<div id="minimal-config-enable-provider">
  ### Minimalkonfiguration (Aktivierung + Anbieter)
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
  ### OpenAI primär mit ElevenLabs als Fallback
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
  ### Edge TTS (primär, ohne API-Schlüssel)
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
  ### Edge-TTS deaktivieren
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
  ### Benutzerdefinierte Limits und Pfad für Einstellungen
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
  ### Nur per Audio antworten, nachdem eine Sprachnachricht eingegangen ist
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
  ### Automatische Zusammenfassung bei langen Antworten deaktivieren
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

Führen Sie dann Folgendes aus:

```
/tts summary off
```

<div id="notes-on-fields">
  ### Hinweise zu den Feldern
</div>

* `auto`: Auto‑TTS‑Modus (`off`, `always`, `inbound`, `tagged`).
  * `inbound` sendet Audio nur nach einer eingehenden Sprachnachricht.
  * `tagged` sendet Audio nur, wenn die Antwort `[[tts]]`‑Tags enthält.
* `enabled`: Legacy-Schalter (doctor migriert dies zu `auto`).
* `mode`: `"final"` (Standard) oder `"all"` (einschließlich Tool-/Block-Antworten).
* `provider`: `"elevenlabs"`, `"openai"` oder `"edge"` (Fallback ist automatisch).
* Wenn `provider` **nicht gesetzt** ist, bevorzugt OpenClaw `openai` (falls Key vorhanden), dann `elevenlabs` (falls Key vorhanden),
  andernfalls `edge`.
* `summaryModel`: optional günstiges Modell für automatische Zusammenfassungen; Standard ist `agents.defaults.model.primary`.
  * Akzeptiert `provider/model` oder einen konfigurierten Modellalias.
* `modelOverrides`: erlauben dem Modell, TTS-Direktiven auszugeben (standardmäßig aktiviert).
* `maxTextLength`: harte Obergrenze für TTS-Eingaben (Zeichen). `/tts audio` schlägt fehl, wenn sie überschritten wird.
* `timeoutMs`: Anfrage-Timeout (ms).
* `prefsPath`: überschreibt den lokalen Pfad zur JSON-Datei mit Voreinstellungen (provider/limit/summary).
* `apiKey`-Werte fallen auf Umgebungsvariablen zurück (`ELEVENLABS_API_KEY`/`XI_API_KEY`, `OPENAI_API_KEY`).
* `elevenlabs.baseUrl`: überschreibt die ElevenLabs-API‑Basis‑URL.
* `elevenlabs.voiceSettings`:
  * `stability`, `similarityBoost`, `style`: `0..1`
  * `useSpeakerBoost`: `true|false`
  * `speed`: `0.5..2.0` (1.0 = normal)
* `elevenlabs.applyTextNormalization`: `auto|on|off`
* `elevenlabs.languageCode`: 2‑stelliger ISO‑639‑1‑Code (z. B. `en`, `de`)
* `elevenlabs.seed`: Integer `0..4294967295` (Best-Effort‑Determinismus)
* `edge.enabled`: erlaubt die Nutzung von Edge TTS (Standard `true`; kein API-Key erforderlich).
* `edge.voice`: Edge-Neural-Voice-Name (z. B. `en-US-MichelleNeural`).
* `edge.lang`: Sprachcode (z. B. `en-US`).
* `edge.outputFormat`: Edge-Ausgabeformat (z. B. `audio-24khz-48kbitrate-mono-mp3`).
  * Siehe Microsoft Speech-Ausgabeformate für gültige Werte; nicht alle Formate werden von Edge unterstützt.
* `edge.rate` / `edge.pitch` / `edge.volume`: Prozentangaben als String (z. B. `+10%`, `-5%`).
* `edge.saveSubtitles`: schreibt JSON-Untertitel neben die Audiodatei.
* `edge.proxy`: Proxy-URL für Edge-TTS-Anfragen.
* `edge.timeoutMs`: Timeout-Überschreibung für Anfragen (ms).

<div id="model-driven-overrides-default-on">
  ## Modellgesteuerte Überschreibungen (standardmäßig aktiviert)
</div>

Standardmäßig **kann** das Modell TTS-Direktiven für eine einzelne Antwort ausgeben.
Wenn `messages.tts.auto` auf `tagged` gesetzt ist, sind diese Direktiven erforderlich, um Audio auszulösen.

Wenn aktiviert, kann das Modell `[[tts:...]]`-Direktiven ausgeben, um die Stimme
für eine einzelne Antwort zu überschreiben, plus einen optionalen `[[tts:text]]...[[/tts:text]]`-Block, um
ausdrucksstarke Tags (Lachen, Gesangshinweise usw.) bereitzustellen, die nur in
der Audioausgabe erscheinen sollen.

Beispiel-Antwort-Payload:

```
Here you go.

[[tts:provider=elevenlabs voiceId=pMsXgVXv3BLzUgSXRplE model=eleven_v3 speed=1.1]]
[[tts:text]](laughs) Read the song once more.[[/tts:text]]
```

Verfügbare Direktiven-Schlüssel (wenn aktiviert):

* `provider` (`openai` | `elevenlabs` | `edge`)
* `voice` (OpenAI-Stimme) oder `voiceId` (ElevenLabs)
* `model` (OpenAI-TTS-Modell oder ElevenLabs-Modell-ID)
* `stability`, `similarityBoost`, `style`, `speed`, `useSpeakerBoost`
* `applyTextNormalization` (`auto|on|off`)
* `languageCode` (ISO 639-1)
* `seed`

Alle Modellüberschreibungen deaktivieren:

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

Optionale Allowlist (deaktiviert bestimmte Überschreibungen, lässt Tags jedoch aktiviert):

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
  ## Benutzerspezifische Einstellungen
</div>

Slash-Befehle schreiben lokale Überschreibungen in `prefsPath` (Standard:
`~/.openclaw/settings/tts.json`, überschreibbar mit `OPENCLAW_TTS_PREFS` oder
`messages.tts.prefsPath`).

Gespeicherte Felder:

* `enabled`
* `provider`
* `maxLength` (Schwellenwert für Zusammenfassungen; Standard 1500 Zeichen)
* `summarize` (Standard: `true`)

Diese Werte überschreiben `messages.tts.*` für diesen Host.

<div id="output-formats-fixed">
  ## Ausgabeformate (fix)
</div>

* **Telegram**: Opus-Sprachnachricht (`opus_48000_64` von ElevenLabs, `opus` von OpenAI).
  * 48kHz / 64kbps ist ein guter Kompromiss für Sprachnachrichten und erforderlich für die runde Blase.
* **Andere Kanäle**: MP3 (`mp3_44100_128` von ElevenLabs, `mp3` von OpenAI).
  * 44,1kHz / 128kbps ist der Standardkompromiss für Sprachverständlichkeit.
* **Edge TTS**: verwendet `edge.outputFormat` (Standardwert `audio-24khz-48kbitrate-mono-mp3`).
  * `node-edge-tts` akzeptiert ein `outputFormat`, aber nicht alle Formate sind
    vom Edge-Dienst verfügbar. citeturn2search0
  * Die Werte für das Ausgabeformat folgen den Microsoft-Speech-Ausgabeformaten (einschließlich Ogg/WebM Opus). citeturn1search0
  * Telegram `sendVoice` akzeptiert OGG/MP3/M4A; verwende OpenAI/ElevenLabs, wenn du
    garantiert Opus-Sprachnachrichten benötigst. citeturn1search1
  * Wenn das konfigurierte Edge-Ausgabeformat fehlschlägt, versucht OpenClaw es erneut mit MP3.

OpenAI-/ElevenLabs-Formate sind fest; Telegram erwartet Opus für die Sprachnachrichten-UX.

<div id="auto-tts-behavior">
  ## Automatisches TTS-Verhalten
</div>

Wenn aktiviert, führt OpenClaw Folgendes aus:

* überspringt TTS, wenn die Antwort bereits Medien oder eine `MEDIA:`-Anweisung enthält.
* überspringt sehr kurze Antworten (&lt; 10 Zeichen).
* fasst lange Antworten zusammen, wenn aktiviert, unter Verwendung von `agents.defaults.model.primary` (oder `summaryModel`).
* hängt das generierte Audio an die Antwort an.

Wenn die Antwort `maxLength` überschreitet und die Zusammenfassung deaktiviert ist (oder kein API-Schlüssel für das Zusammenfassungsmodell vorhanden ist), wird das Audio übersprungen und die normale Textantwort wird gesendet.

<div id="flow-diagram">
  ## Ablaufdiagramm
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
  ## Verwendung von Slash-Befehlen
</div>

Es gibt genau einen Befehl: `/tts`.
Siehe [Slash-Befehle](/de/tools/slash-commands) für Einzelheiten zur Aktivierung.

Discord-Hinweis: `/tts` ist ein integrierter Discord-Befehl, daher registriert OpenClaw dort
`/voice` als nativen Befehl. Die Eingabe `/tts ...` funktioniert weiterhin.

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

Hinweise:

* Befehle erfordern einen autorisierten Absender (Allowlist-/Owner-Regeln gelten weiterhin).
* `commands.text` oder die native Command-Registrierung muss aktiviert sein.
* `off|always|inbound|tagged` sind sitzungsspezifische Umschalter (`/tts on` ist ein Alias für `/tts always`).
* `limit` und `summary` werden in lokalen Einstellungen gespeichert, nicht in der Hauptkonfiguration.
* `/tts audio` erzeugt eine einmalige Audioantwort (schaltet TTS nicht ein).

<div id="agent-tool">
  ## Agent-Tool
</div>

Das `tts`-Tool konvertiert Text in gesprochene Sprache und gibt einen `MEDIA:`-Pfad zurück. Wenn das
Ergebnis Telegram-kompatibel ist, fügt das Tool `[[audio_as_voice]]` hinzu, damit
Telegram es als Sprachnachrichten-Blase darstellt.

<div id="gateway-rpc">
  ## Gateway-RPC
</div>

Methoden des Gateways:

* `tts.status`
* `tts.enable`
* `tts.disable`
* `tts.convert`
* `tts.setProvider`
* `tts.providers`