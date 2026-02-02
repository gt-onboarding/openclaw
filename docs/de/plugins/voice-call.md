---
title: Sprachanruf
summary: "Sprachanruf-Plugin: ausgehende + eingehende Anrufe über Twilio/Telnyx/Plivo (Plugin-Installation + Konfiguration + CLI)"
read_when:
  - Du möchtest von OpenClaw aus einen ausgehenden Sprachanruf tätigen
  - Du konfigurierst oder entwickelst das Sprachanruf-Plugin
---

<div id="voice-call-plugin">
  # Voice Call (Plugin)
</div>

Sprachanrufe für OpenClaw über ein Plugin. Unterstützt ausgehende Benachrichtigungen und
mehrere Gesprächsrunden (Multi-Turn-Dialoge) mit Richtlinien für eingehende Anrufe.

Aktuelle Anbieter:

- `twilio` (Programmable Voice + Media Streams)
- `telnyx` (Call Control v2)
- `plivo` (Voice API + XML transfer + GetInput speech)
- `mock` (dev/kein Netzwerkzugriff)

Gedankliches Modell (Kurzfassung):

- Plugin installieren
- Gateway neu starten
- Unter `plugins.entries.voice-call.config` konfigurieren
- `openclaw voicecall ...` oder das Tool `voice_call` verwenden

<div id="where-it-runs-local-vs-remote">
  ## Wo es läuft (lokal vs. remote)
</div>

Das Voice Call Plugin läuft **im Gateway-Prozess**.

Wenn du ein Remote-Gateway verwendest, installiere und konfiguriere das Plugin auf dem **Host, auf dem das Gateway läuft**, und starte das Gateway anschließend neu, damit es geladen wird.

<div id="install">
  ## Installation
</div>

<div id="option-a-install-from-npm-recommended">
  ### Option A: Installation mit npm (empfohlen)
</div>

```bash
openclaw plugins install @openclaw/voice-call
```

Starte das Gateway anschließend neu.


<div id="option-b-install-from-a-local-folder-dev-no-copying">
  ### Option B: Installation aus einem lokalen Verzeichnis (dev, ohne Kopieren)
</div>

```bash
openclaw plugins install ./extensions/voice-call
cd ./extensions/voice-call && pnpm install
```

Starte das Gateway anschließend neu.


<div id="config">
  ## Konfiguration
</div>

Lege die Konfiguration unter `plugins.entries.voice-call.config` fest:

```json5
{
  plugins: {
    entries: {
      "voice-call": {
        enabled: true,
        config: {
          provider: "twilio", // or "telnyx" | "plivo" | "mock"
          fromNumber: "+15550001234",
          toNumber: "+15550005678",

          twilio: {
            accountSid: "ACxxxxxxxx",
            authToken: "..."
          },

          plivo: {
            authId: "MAxxxxxxxxxxxxxxxxxxxx",
            authToken: "..."
          },

          // Webhook server
          serve: {
            port: 3334,
            path: "/voice/webhook"
          },

          // Öffentliche Erreichbarkeit (eine auswählen)
          // publicUrl: "https://example.ngrok.app/voice/webhook",
          // tunnel: { provider: "ngrok" },
          // tailscale: { mode: "funnel", path: "/voice/webhook" }

          outbound: {
            defaultMode: "notify" // notify | conversation
          },

          streaming: {
            enabled: true,
            streamPath: "/voice/stream"
          }
        }
      }
    }
  }
}
```

Hinweise:

* Twilio/Telnyx erfordern eine **öffentlich erreichbare** Webhook-URL.
* Plivo erfordert eine **öffentlich erreichbare** Webhook-URL.
* `mock` ist ein lokaler Dev-Anbieter (keine Netzwerkaufrufe).
* `skipSignatureVerification` ist nur für lokale Tests.
* Wenn du den kostenlosen ngrok-Free-Tier-Plan verwendest, setze `publicUrl` auf die exakte ngrok-URL; die Signaturprüfung wird immer erzwungen.
* `tunnel.allowNgrokFreeTierLoopbackBypass: true` erlaubt Twilio-Webhooks mit ungültigen Signaturen **nur**, wenn `tunnel.provider="ngrok"` und `serve.bind` auf Loopback gebunden ist (lokaler ngrok-Agent). Nur für lokale Entwicklung verwenden.
* ngrok-Free-Tier-URLs können sich ändern oder Zwischenseiten einfügen; wenn `publicUrl` abweicht, schlagen Twilio-Signaturen fehl. Für Produktionsumgebungen verwende bevorzugt eine stabile Domain oder Tailscale Funnel.


<div id="tts-for-calls">
  ## TTS für Anrufe
</div>

Voice Call verwendet die zentrale `messages.tts`‑Konfiguration (OpenAI oder ElevenLabs) für
gestreamte Sprachausgabe in Anrufen. Du kannst sie in der Plugin‑Konfiguration mit derselben
**Struktur** überschreiben — sie wird per Deep‑Merge mit `messages.tts` zusammengeführt.

```json5
{
  tts: {
    provider: "elevenlabs",
    elevenlabs: {
      voiceId: "pMsXgVXv3BLzUgSXRplE",
      modelId: "eleven_multilingual_v2"
    }
  }
}
```

Hinweise:

* **Edge TTS wird für Sprach­anrufe ignoriert** (Telefonieaudio benötigt PCM; die Edge-TTS-Ausgabe ist unzuverlässig).
* Core TTS wird verwendet, wenn Twilio Media Streaming aktiviert ist; andernfalls greifen Anrufe auf die nativen Stimmen des Anbieters zurück.


<div id="more-examples">
  ### Weitere Beispiele
</div>

Nur das Core-TTS verwenden (kein Override):

```json5
{
  messages: {
    tts: {
      provider: "openai",
      openai: { voice: "alloy" }
    }
  }
}
```

Override nur für Anrufe auf ElevenLabs umstellen (Kern-Standardkonfiguration sonst beibehalten):

```json5
{
  plugins: {
    entries: {
      "voice-call": {
        config: {
          tts: {
            provider: "elevenlabs",
            elevenlabs: {
              apiKey: "elevenlabs_key",
              voiceId: "pMsXgVXv3BLzUgSXRplE",
              modelId: "eleven_multilingual_v2"
            }
          }
        }
      }
    }
  }
}
```

Nur das OpenAI‑Modell für Aufrufe überschreiben (Beispiel für Deep‑Merge):

```json5
{
  plugins: {
    entries: {
      "voice-call": {
        config: {
          tts: {
            openai: {
              model: "gpt-4o-mini-tts",
              voice: "marin"
            }
          }
        }
      }
    }
  }
}
```


<div id="inbound-calls">
  ## Eingehende Anrufe
</div>

Standardmäßig ist die Richtlinie für eingehende Anrufe auf `disabled` gesetzt. Um eingehende Anrufe zu aktivieren, setze:

```json5
{
  inboundPolicy: "allowlist",
  allowFrom: ["+15550001234"],
  inboundGreeting: "Hallo! Wie kann ich helfen?"
}
```

Automatische Antworten verwenden das agent-System. Konfiguriere sie mit:

* `responseModel`
* `responseSystemPrompt`
* `responseTimeoutMs`


<div id="cli">
  ## CLI
</div>

```bash
openclaw voicecall call --to "+15555550123" --message "Hello from OpenClaw"
openclaw voicecall continue --call-id <id> --message "Any questions?"
openclaw voicecall speak --call-id <id> --message "One moment"
openclaw voicecall end --call-id <id>
openclaw voicecall status --call-id <id>
openclaw voicecall tail
openclaw voicecall expose --mode funnel
```


<div id="agent-tool">
  ## Agent-Tool
</div>

Toolname: `voice_call`

Aktionen:

- `initiate_call` (message, to?, mode?)
- `continue_call` (callId, message)
- `speak_to_user` (callId, message)
- `end_call` (callId)
- `get_status` (callId)

Dieses Repository enthält eine passende Skill-Dokumentation unter `skills/voice-call/SKILL.md`.

<div id="gateway-rpc">
  ## Gateway RPC
</div>

- `voicecall.initiate` (`to?`, `message`, `mode?`)
- `voicecall.continue` (`callId`, `message`)
- `voicecall.speak` (`callId`, `message`)
- `voicecall.end` (`callId`)
- `voicecall.status` (`callId`)