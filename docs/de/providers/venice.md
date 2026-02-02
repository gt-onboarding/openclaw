---
title: Venice
summary: "Verwende datenschutzorientierte Modelle von Venice AI in OpenClaw"
read_when:
  - Du möchtest datenschutzorientierte Inferenz in OpenClaw
  - Du möchtest eine Anleitung zur Einrichtung von Venice AI
---

<div id="venice-ai-venice-highlight">
  # Venice AI (Venice-Highlight)
</div>

**Venice** ist unser Highlight-Setup für Privacy-First-Inferenz mit optional anonymisiertem Zugriff auf proprietäre Modelle.

Venice AI bietet datenschutzfokussierte KI-Inferenz mit Unterstützung für unzensierte Modelle und Zugriff auf führende proprietäre Modelle über deren anonymisierte Proxys. Alle Inferenzvorgänge sind standardmäßig privat – kein Training mit Ihren Daten, kein Logging.

<div id="why-venice-in-openclaw">
  ## Warum Venice in OpenClaw
</div>

- **Private Inferenz** für Open-Source-Modelle (ohne Protokollierung).
- **Unzensierte Modelle**, wenn du sie brauchst.
- **Anonymisierter Zugriff** auf proprietäre Modelle (Opus/GPT/Gemini), wenn Qualität zählt.
- OpenAI-kompatible `/v1`-Endpunkte.

<div id="privacy-modes">
  ## Datenschutzmodi
</div>

Venice bietet zwei Datenschutzstufen – diese zu verstehen ist entscheidend für die Wahl deines Modells:

| Modus | Beschreibung | Modelle |
|-------|--------------|---------|
| **Privat** | Vollständig privat. Prompts und Antworten werden **niemals gespeichert oder protokolliert**. Flüchtig. | Llama, Qwen, DeepSeek, Venice Uncensored, etc. |
| **Anonymisiert** | Über Venice als Proxy mit entfernten Metadaten. Der zugrunde liegende Anbieter (OpenAI, Anthropic) sieht anonymisierte Anfragen. | Claude, GPT, Gemini, Grok, Kimi, MiniMax |

<div id="features">
  ## Funktionen
</div>

- **Datenschutzorientiert**: Wähle zwischen „private“ (vollständig privat) und „anonymized“ (über einen Proxy) Modi
- **Unzensierte Modelle**: Zugriff auf Modelle ohne Inhaltsbeschränkungen
- **Zugriff auf führende Modelle**: Nutze Claude, GPT-5.2, Gemini, Grok über Venices anonymisierten Proxy
- **OpenAI-kompatible API**: Standard-`/v1`-Endpoints für eine einfache Integration
- **Streaming**: ✅ Wird von allen Modellen unterstützt
- **Function calling**: ✅ Wird von ausgewählten Modellen unterstützt (siehe Modellfähigkeiten)
- **Vision**: ✅ Wird von Modellen mit Vision-Fähigkeit unterstützt
- **Keine harten Rate-Limits**: Fair-Use-Drosselung kann bei extrem intensiver Nutzung greifen

<div id="setup">
  ## Einrichtung
</div>

<div id="1-get-api-key">
  ### 1. API-Schlüssel erhalten
</div>

1. Erstelle ein Konto bei [venice.ai](https://venice.ai)
2. Wechsle zu **Settings → API Keys → Create new key**
3. Kopiere deinen API-Schlüssel (Format: `vapi_xxxxxxxxxxxx`)

<div id="2-configure-openclaw">
  ### 2. OpenClaw konfigurieren
</div>

**Option A: Umgebungsvariable**

```bash
export VENICE_API_KEY="vapi_xxxxxxxxxxxx"
```

**Option B: Interaktive Einrichtung (empfohlen)**

```bash
openclaw onboard --auth-choice venice-api-key
```

Dies wird:

1. Nach deinem API-Schlüssel fragen (oder den vorhandenen `VENICE_API_KEY` verwenden)
2. Alle verfügbaren Venice-Modelle anzeigen
3. Dich dein Standardmodell auswählen lassen
4. Den Anbieter automatisch konfigurieren

**Option C: Nicht-interaktiv**

```bash
openclaw onboard --non-interactive \
  --auth-choice venice-api-key \
  --venice-api-key "vapi_xxxxxxxxxxxx"
```


<div id="3-verify-setup">
  ### 3. Einrichtung überprüfen
</div>

```bash
openclaw chat --model venice/llama-3.3-70b "Hello, are you working?"
```


<div id="model-selection">
  ## Modellauswahl
</div>

Nach der Einrichtung zeigt OpenClaw alle verfügbaren Venice-Modelle an. Wähle je nach Bedarf:

* **Standard (unsere Empfehlung)**: `venice/llama-3.3-70b` für private, ausgewogene Leistung.
* **Beste Gesamtqualität**: `venice/claude-opus-45` für anspruchsvolle Aufgaben (Opus bleibt am stärksten).
* **Privatsphäre**: Wähle „private“-Modelle für vollständig private Inferenz.
* **Funktionalität**: Wähle „anonymized“-Modelle, um Claude, GPT und Gemini über den Proxy von Venice zu nutzen.

Du kannst dein Standardmodell jederzeit ändern:

```bash
openclaw models set venice/claude-opus-45
openclaw models set venice/llama-3.3-70b
```

Alle verfügbaren Modelle auflisten:

```bash
openclaw models list | grep venice
```


<div id="configure-via-openclaw-configure">
  ## Konfiguration mit `openclaw configure`
</div>

1. Führen Sie `openclaw configure` aus
2. Wählen Sie **Model/auth**
3. Wählen Sie **Venice AI**

<div id="which-model-should-i-use">
  ## Welches Modell sollte ich verwenden?
</div>

| Anwendungsfall | Empfohlenes Modell | Warum |
|----------------|--------------------|-------|
| **Allgemeiner Chat** | `llama-3.3-70b` | Gutes Allround-Modell, vollständig privat |
| **Beste Gesamtqualität** | `claude-opus-45` | Opus bleibt das stärkste Modell für schwierige Aufgaben |
| **Privatsphäre + Claude-Qualität** | `claude-opus-45` | Bestes Reasoning über anonymisierten Proxy |
| **Programmierung** | `qwen3-coder-480b-a35b-instruct` | Für Code optimiert, 262k Kontextfenster |
| **Vision-Aufgaben** | `qwen3-vl-235b-a22b` | Bestes privates Vision-Modell |
| **Unzensiert** | `venice-uncensored` | Keine Inhaltsbeschränkungen |
| **Schnell + günstig** | `qwen3-4b` | Leichtgewichtig, trotzdem leistungsfähig |
| **Komplexes Reasoning** | `deepseek-v3.2` | Starkes Reasoning, privat |

<div id="available-models-25-total">
  ## Verfügbare Modelle (insgesamt 25)
</div>

<div id="private-models-15-fully-private-no-logging">
  ### Private Modelle (15) — Vollständig privat, kein Logging
</div>

| Model ID | Name | Kontext (Tokens) | Features |
|----------|------|------------------|----------|
| `llama-3.3-70b` | Llama 3.3 70B | 131k | Allgemein |
| `llama-3.2-3b` | Llama 3.2 3B | 131k | Schnell, schlank |
| `hermes-3-llama-3.1-405b` | Hermes 3 Llama 3.1 405B | 131k | Komplexe Aufgaben |
| `qwen3-235b-a22b-thinking-2507` | Qwen3 235B Thinking | 131k | Reasoning |
| `qwen3-235b-a22b-instruct-2507` | Qwen3 235B Instruct | 131k | Allgemein |
| `qwen3-coder-480b-a35b-instruct` | Qwen3 Coder 480B | 262k | Code |
| `qwen3-next-80b` | Qwen3 Next 80B | 262k | Allgemein |
| `qwen3-vl-235b-a22b` | Qwen3 VL 235B | 262k | Vision |
| `qwen3-4b` | Venice Small (Qwen3 4B) | 32k | Schnell, Reasoning |
| `deepseek-v3.2` | DeepSeek V3.2 | 163k | Reasoning |
| `venice-uncensored` | Venice Uncensored | 32k | Unzensiert |
| `mistral-31-24b` | Venice Medium (Mistral) | 131k | Vision |
| `google-gemma-3-27b-it` | Gemma 3 27B Instruct | 202k | Vision |
| `openai-gpt-oss-120b` | OpenAI GPT OSS 120B | 131k | Allgemein |
| `zai-org-glm-4.7` | GLM 4.7 | 202k | Reasoning, mehrsprachig |

<div id="anonymized-models-10-via-venice-proxy">
  ### Anonymisierte Modelle (10) — über den Venice-Proxy
</div>

| Model ID | Original | Kontext (Token) | Fähigkeiten |
|----------|----------|------------------|-------------|
| `claude-opus-45` | Claude Opus 4.5 | 202k | Reasoning, Vision |
| `claude-sonnet-45` | Claude Sonnet 4.5 | 202k | Reasoning, Vision |
| `openai-gpt-52` | GPT-5.2 | 262k | Reasoning |
| `openai-gpt-52-codex` | GPT-5.2 Codex | 262k | Reasoning, Vision |
| `gemini-3-pro-preview` | Gemini 3 Pro | 202k | Reasoning, Vision |
| `gemini-3-flash-preview` | Gemini 3 Flash | 262k | Reasoning, Vision |
| `grok-41-fast` | Grok 4.1 Fast | 262k | Reasoning, Vision |
| `grok-code-fast-1` | Grok Code Fast 1 | 262k | Reasoning, Code |
| `kimi-k2-thinking` | Kimi K2 Thinking | 262k | Reasoning |
| `minimax-m21` | MiniMax M2.1 | 202k | Reasoning |

<div id="model-discovery">
  ## Modellerkennung
</div>

OpenClaw erkennt Modelle automatisch über die Venice API, wenn `VENICE_API_KEY` konfiguriert ist. Wenn die API nicht erreichbar ist, wird auf einen statischen Katalog zurückgegriffen.

Der `/models`-Endpunkt ist öffentlich (keine Authentifizierung für das Auflisten erforderlich), aber für Inferenzanfragen ist ein gültiger API-Schlüssel erforderlich.

<div id="streaming-tool-support">
  ## Streaming- und Tool-Unterstützung
</div>

| Feature | Unterstützung |
|---------|---------|
| **Streaming** | ✅ Alle Modelle |
| **Function calling** | ✅ Die meisten Modelle (siehe `supportsFunctionCalling` in der api) |
| **Vision/Images** | ✅ Modelle mit dem Feature „Vision“ |
| **JSON mode** | ✅ Unterstützt über `response_format` |

<div id="pricing">
  ## Preise
</div>

Venice verwendet ein guthabenbasiertes System. Aktuelle Tarife findest du unter [venice.ai/pricing](https://venice.ai/pricing):

- **Private Modelle**: In der Regel günstiger
- **Anonymisierte Modelle**: Ähnlich wie direkte API-Preise zzgl. einer kleinen Venice-Gebühr

<div id="comparison-venice-vs-direct-api">
  ## Vergleich: Venice vs direkte API
</div>

| Aspekt | Venice (anonymisiert) | Direkte API |
|--------|-----------------------|-------------|
| **Datenschutz** | Metadaten entfernt und anonymisiert | Ihr Konto verknüpft |
| **Latenz** | +10–50 ms (Proxy) | Direkt |
| **Funktionen** | Die meisten Funktionen werden unterstützt | Vollständiger Funktionsumfang |
| **Abrechnung** | Venice-Guthaben | Abrechnung über Anbieter |

<div id="usage-examples">
  ## Anwendungsbeispiele
</div>

```bash
# Use default private model
openclaw chat --model venice/llama-3.3-70b

# Claude über Venice verwenden (anonymisiert)
openclaw chat --model venice/claude-opus-45

# Use uncensored model
openclaw chat --model venice/venice-uncensored

# Use vision model with image
openclaw chat --model venice/qwen3-vl-235b-a22b

# Use coding model
openclaw chat --model venice/qwen3-coder-480b-a35b-instruct
```


<div id="troubleshooting">
  ## Fehlerbehebung
</div>

<div id="api-key-not-recognized">
  ### API-Schlüssel wird nicht erkannt
</div>

```bash
echo $VENICE_API_KEY
openclaw models list | grep venice
```

Stellen Sie sicher, dass der Schlüssel mit `vapi_` beginnt.


<div id="model-not-available">
  ### Modell nicht verfügbar
</div>

Der Venice-Modellkatalog wird dynamisch aktualisiert. Führen Sie `openclaw models list` aus, um die aktuell verfügbaren Modelle anzuzeigen. Einige Modelle sind möglicherweise vorübergehend nicht verfügbar.

<div id="connection-issues">
  ### Verbindungsprobleme
</div>

Die Venice-API ist unter `https://api.venice.ai/api/v1` verfügbar. Stelle sicher, dass dein Netzwerk HTTPS-Verbindungen zulässt.

<div id="config-file-example">
  ## Beispiel-Konfigurationsdatei
</div>

```json5
{
  env: { VENICE_API_KEY: "vapi_..." },
  agents: { defaults: { model: { primary: "venice/llama-3.3-70b" } } },
  models: {
    mode: "merge",
    providers: {
      venice: {
        baseUrl: "https://api.venice.ai/api/v1",
        apiKey: "${VENICE_API_KEY}",
        api: "openai-completions",
        models: [
          {
            id: "llama-3.3-70b",
            name: "Llama 3.3 70B",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 131072,
            maxTokens: 8192
          }
        ]
      }
    }
  }
}
```


<div id="links">
  ## Links
</div>

- [Venice AI](https://venice.ai)
- [API-Dokumentation](https://docs.venice.ai)
- [Preisübersicht](https://venice.ai/pricing)
- [Status](https://status.venice.ai)