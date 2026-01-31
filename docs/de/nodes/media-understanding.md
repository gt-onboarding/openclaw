---
title: Medienverständnis
summary: "Verstehen eingehender Bild-/Audio-/Video-Daten (optional) mit Anbieter- und CLI-Fallbacks"
read_when:
  - Entwurf oder Überarbeitung des Medienverständnisses
  - Feinabstimmung der Vorverarbeitung eingehender Audio-/Video-/Bilddaten
---

<div id="media-understanding-inbound-2026-01-17">
  # Medienverständnis (eingehend) — 2026-01-17
</div>

OpenClaw kann **eingehende Medien** (Bild/Audio/Video) zusammenfassen, bevor die Antwort-Pipeline ausgeführt wird. Es erkennt automatisch, ob lokale Tools oder Anbieterschlüssel verfügbar sind, und kann deaktiviert oder angepasst werden. Wenn das Medienverständnis deaktiviert ist, erhalten Modelle die ursprünglichen Dateien/URLs trotzdem wie gewohnt.

<div id="goals">
  ## Ziele
</div>

* Optional: eingehende Medien vorverarbeiten und zu kurzem Text verdichten, um schnelleres Routing und bessere Befehlsinterpretation zu ermöglichen.
* Originalmedien bei der Übergabe an das Modell stets unverändert beibehalten.
* **Anbieter-APIs** und **CLI-Fallbacks** unterstützen.
* Mehrere Modelle mit geordneter Fallback-Reihenfolge (Fehler/Größe/Timeout) ermöglichen.

<div id="highlevel-behavior">
  ## Verhalten auf hoher Ebene
</div>

1. Eingehende Anhänge erfassen (`MediaPaths`, `MediaUrls`, `MediaTypes`).
2. Für jede aktivierte Fähigkeit (Bild/Audio/Video) Anhänge gemäß Richtlinie auswählen (Standard: **erster**).
3. Den ersten geeigneten Modelleintrag auswählen (Größe + Fähigkeit + Auth).
4. Wenn ein Modell fehlschlägt oder das Medium zu groß ist, **auf den nächsten Eintrag zurückgreifen**.
5. Bei Erfolg:
   * `Body` wird zu einem `[Image]`‑, `[Audio]`‑ oder `[Video]`‑Block.
   * Audio setzt `{{Transcript}}`; das Befehls‑Parsing verwendet den Untertiteltext, wenn vorhanden,
     andernfalls das Transkript.
   * Untertitel werden als `User text:` innerhalb des Blocks beibehalten.

Wenn das Verständnis fehlschlägt oder deaktiviert ist, **läuft der Antwort‑Flow mit dem ursprünglichen Body + Anhängen weiter**.

<div id="config-overview">
  ## Konfigurationsübersicht
</div>

`tools.media` unterstützt **gemeinsame Modelle** plus Überschreibungen pro Fähigkeit:

* `tools.media.models`: gemeinsame Modellliste (verwenden Sie `capabilities` für das Gating).
* `tools.media.image` / `tools.media.audio` / `tools.media.video`:
  * Vorgaben (`prompt`, `maxChars`, `maxBytes`, `timeoutSeconds`, `language`)
  * Anbieter‑Überschreibungen (`baseUrl`, `headers`, `providerOptions`)
  * Deepgram‑Audiooptionen über `tools.media.audio.providerOptions.deepgram`
  * optionale **`models`‑Liste pro Fähigkeit** (bevorzugt vor gemeinsamen Modellen)
  * `attachments`‑Richtlinie (`mode`, `maxAttachments`, `prefer`)
  * `scope` (optionales Gating nach channel/chatType/session‑Key)
* `tools.media.concurrency`: maximale Anzahl gleichzeitiger Capability‑Ausführungen (Standard **2**).

```json5
{
  tools: {
    media: {
      models: [ /* shared list */ ],
      image: { /* optionale Überschreibungen */ },
      audio: { /* optionale Überschreibungen */ },
      video: { /* optionale Überschreibungen */ }
    }
  }
}
```

<div id="model-entries">
  ### Modelleinträge
</div>

Jeder `models[]`-Eintrag kann entweder ein **anbieter** oder die **CLI** sein:

```json5
{
  type: "provider",        // default if omitted
  provider: "openai",
  model: "gpt-5.2",
  prompt: "Describe the image in <= 500 chars.",
  maxChars: 500,
  maxBytes: 10485760,
  timeoutSeconds: 60,
  capabilities: ["image"], // optional, für multimodale Einträge verwendet
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

CLI-Vorlagen können außerdem Folgendes verwenden:

* `{{MediaDir}}` (Verzeichnis, das die Mediendatei enthält)
* `{{OutputDir}}` (temporäres Arbeitsverzeichnis, das für diesen Lauf erstellt wird)
* `{{OutputBase}}` (Basis-Pfad der temporären Datei, ohne Dateiendung)

<div id="defaults-and-limits">
  ## Standardwerte und Limits
</div>

Empfohlene Standardwerte:

* `maxChars`: **500** für Bild/Video (kurz, CLI‑/befehlsfreundlich)
* `maxChars`: **nicht gesetzt** für Audio (vollständiges Transkript, sofern du kein Limit setzt)
* `maxBytes`:
  * Bild: **10MB**
  * Audio: **20MB**
  * Video: **50MB**

Regeln:

* Wenn der Medieninhalt `maxBytes` überschreitet, wird dieses Modell übersprungen und das **nächste Modell ausprobiert**.
* Wenn das Modell mehr als `maxChars` zurückgibt, wird die Ausgabe gekürzt.
* `prompt` ist standardmäßig ein einfacher Text „Describe the {media}.“ plus die `maxChars`‑Vorgabe (nur Bild/Video).
* Wenn `<capability>.enabled: true` gesetzt ist, aber keine Modelle konfiguriert sind, versucht OpenClaw das
  **aktive Antwortmodell**, sofern dessen Anbieter diese Funktion unterstützt.

<div id="auto-detect-media-understanding-default">
  ### Medienverständnis automatisch erkennen (Standard)
</div>

Wenn `tools.media.<capability>.enabled` **nicht** auf `false` gesetzt ist und du
keine Modelle konfiguriert hast, führt OpenClaw in dieser Reihenfolge eine automatische Erkennung durch
und **stoppt bei der ersten funktionierenden Option**:

1. **Lokale CLIs** (nur Audio; falls installiert)
   * `sherpa-onnx-offline` (erfordert `SHERPA_ONNX_MODEL_DIR` mit Encoder/Decoder/Joiner/Tokens)
   * `whisper-cli` (`whisper-cpp`; verwendet `WHISPER_CPP_MODEL` oder das mitgelieferte tiny-Modell)
   * `whisper` (Python-CLI; lädt Modelle automatisch herunter)
2. **Gemini CLI** (`gemini`) mit `read_many_files`
3. **Anbieterschlüssel**
   * Audio: OpenAI → Groq → Deepgram → Google
   * Bild: OpenAI → Anthropic → Google → MiniMax
   * Video: Google

Um die automatische Erkennung zu deaktivieren, setze:

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

Hinweis: Die Erkennung von Binaries erfolgt auf macOS/Linux/Windows bestmöglich; stelle sicher, dass die CLI im `PATH` liegt (wir expandieren `~`), oder setze ein explizites CLI-Modell mit einem vollständigen Befehlspfad.

<div id="capabilities-optional">
  ## Fähigkeiten (optional)
</div>

Wenn du `capabilities` setzt, gilt der Eintrag nur für diese Medientypen. Für gemeinsame
Listen kann OpenClaw Standardwerte ableiten:

* `openai`, `anthropic`, `minimax`: **image**
* `google` (Gemini API): **image + audio + video**
* `groq`: **audio**
* `deepgram`: **audio**

Für CLI-Einträge **setze `capabilities` explizit**, um unerwartete Treffer zu vermeiden.
Wenn du `capabilities` weglässt, kann der Eintrag in der Liste, in der er erscheint, verwendet werden.

<div id="provider-support-matrix-openclaw-integrations">
  ## Support-Matrix für Anbieter (OpenClaw-Integrationen)
</div>

| Fähigkeit | Anbieter-Integration | Anmerkungen |
|----------|----------------------|-------------|
| Bild     | OpenAI / Anthropic / Google / andere über `pi-ai` | Jedes Modell mit Bildunterstützung in der Registry funktioniert. |
| Audio    | OpenAI, Groq, Deepgram, Google | Anbieter-Transkription (Whisper/Deepgram/Gemini). |
| Video    | Google (Gemini API) | Anbieter-Videoverständnis. |

<div id="recommended-providers">
  ## Empfohlene Anbieter
</div>

**Bild**

* Verwende bevorzugt dein aktives Modell, wenn es Bilder unterstützt.
* Gute Standardvorgaben: `openai/gpt-5.2`, `anthropic/claude-opus-4-5`, `google/gemini-3-pro-preview`.

**Audio**

* `openai/gpt-4o-mini-transcribe`, `groq/whisper-large-v3-turbo` oder `deepgram/nova-3`.
* CLI-Fallback: `whisper-cli` (whisper-cpp) oder `whisper`.
* Deepgram-Konfiguration: [Deepgram (Audio-Transkription)](/de/providers/deepgram).

**Video**

* `google/gemini-3-flash-preview` (schnell), `google/gemini-3-pro-preview` (umfangreicher).
* CLI-Fallback: `gemini` CLI (unterstützt `read_file` für Video und Audio).

<div id="attachment-policy">
  ## Richtlinie für Anhänge
</div>

Die je Fähigkeit konfigurierbare Einstellung `attachments` steuert, welche Anhänge verarbeitet werden:

* `mode`: `first` (Standard) oder `all`
* `maxAttachments`: begrenzt die Anzahl der verarbeiteten Anhänge (Standardwert **1**)
* `prefer`: `first`, `last`, `path`, `url`

Wenn `mode: "all"` verwendet wird, werden Ausgaben als `[Image 1/2]`, `[Audio 2/2]` usw. gekennzeichnet.

<div id="config-examples">
  ## Konfigurationsbeispiele
</div>

<div id="1-shared-models-list-overrides">
  ### 1) Gemeinsame Modellliste + Überschreibungen
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
  ### 2) Nur Audio + Video (ohne Bild)
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
  ### 3) Optionales Bildverständnis
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
              "Lesen Sie die Medien unter {{MediaPath}} und beschreiben Sie sie in <= {{MaxChars}} Zeichen."
            ]
          }
        ]
      }
    }
  }
}
```

<div id="4-multimodal-single-entry-explicit-capabilities">
  ### 4) Multimodaler Single-Entry-Punkt (explizite Fähigkeiten)
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
  ## Statusausgabe
</div>

Wenn Media Understanding ausgeführt wird, enthält `/status` eine kurze Zusammenfassungszeile:

```
📎 Media: image ok (openai/gpt-5.2) · audio skipped (maxBytes)
```

Dies zeigt die Ergebnisse je Fähigkeit sowie den gewählten Anbieter bzw. das gewählte Modell, sofern zutreffend.

<div id="notes">
  ## Hinweise
</div>

* Understanding arbeitet nach dem **Best‑Effort**‑Prinzip. Fehler verhindern Antworten nicht.
* Anhänge werden weiterhin an Modelle übergeben, selbst wenn Understanding deaktiviert ist.
* Verwende `scope`, um einzuschränken, wo Understanding ausgeführt wird (z. B. nur DMs).

<div id="related-docs">
  ## Verwandte Dokumente
</div>

* [Konfiguration](/de/gateway/configuration)
* [Bild- und Medienunterstützung](/de/nodes/images)