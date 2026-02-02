---
title: Lokale Modelle
summary: "OpenClaw auf lokalen LLMs ausführen (LM Studio, vLLM, LiteLLM, benutzerdefinierte OpenAI-Endpunkte)"
read_when:
  - Du möchtest Modelle auf deinem eigenen GPU-Rechner bereitstellen
  - Du bindest LM Studio oder einen OpenAI-kompatiblen Proxy an
  - Du benötigst die sicherste Anleitung zum Einsatz lokaler Modelle
---

<div id="local-models">
  # Lokale Modelle
</div>

Lokale Ausführung ist machbar, aber OpenClaw erwartet großen Kontext + starke Schutzmechanismen gegen Prompt-Injection. Kleine GPUs kürzen den Kontext und schwächen die Sicherheit. Setze hoch an: **≥2 maximal ausgebaute Mac Studios oder ein äquivalentes GPU-Rig (~30.000 $+)**. Eine einzelne **24‑GB‑GPU** reicht nur für leichtere Prompts mit höherer Latenz. Verwende die **größte bzw. vollwertige Modellvariante, die du ausführen kannst**; stark quantisierte oder „kleine“ Checkpoints erhöhen das Prompt-Injection-Risiko (siehe [Security](/de/gateway/security)).

<div id="recommended-lm-studio-minimax-m21-responses-api-full-size">
  ## Empfohlen: LM Studio + MiniMax M2.1 (Responses API, vollständiges Modell)
</div>

Dies ist derzeit der beste lokale Stack. Lade MiniMax M2.1 in LM Studio, aktiviere den lokalen Server (Standard: `http://127.0.0.1:1234`) und verwende die Responses API, um das Reasoning vom endgültigen Text zu trennen.

```json5
{
  agents: {
    defaults: {
      model: { primary: "lmstudio/minimax-m2.1-gs32" },
      models: {
        "anthropic/claude-opus-4-5": { alias: "Opus" },
        "lmstudio/minimax-m2.1-gs32": { alias: "Minimax" }
      }
    }
  },
  models: {
    mode: "merge",
    providers: {
      lmstudio: {
        baseUrl: "http://127.0.0.1:1234/v1",
        apiKey: "lmstudio",
        api: "openai-responses",
        models: [
          {
            id: "minimax-m2.1-gs32",
            name: "MiniMax M2.1 GS32",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 196608,
            maxTokens: 8192
          }
        ]
      }
    }
  }
}
```

**Setup-Checkliste**

* Installiere LM Studio: https://lmstudio.ai
* Lade in LM Studio den **größten verfügbaren MiniMax M2.1-Build** herunter (vermeide „small“-/stark quantisierte Varianten), starte den Server und bestätige, dass `http://127.0.0.1:1234/v1/models` ihn auflistet.
* Lass das Modell geladen; ein Kaltstart erhöht die Startlatenz.
* Passe `contextWindow`/`maxTokens` an, falls sich dein LM Studio-Build unterscheidet.
* Verwende für WhatsApp weiterhin die Responses API, damit nur der finale Text gesendet wird.

Lass gehostete Modelle auch bei lokalem Betrieb konfiguriert; verwende `models.mode: "merge"`, damit Fallbacks weiterhin verfügbar bleiben.

<div id="hybrid-config-hosted-primary-local-fallback">
  ### Hybrid-Konfiguration: gehostetes Primärsystem, lokaler Fallback
</div>

```json5
{
  agents: {
    defaults: {
      model: {
        primary: "anthropic/claude-sonnet-4-5",
        fallbacks: ["lmstudio/minimax-m2.1-gs32", "anthropic/claude-opus-4-5"]
      },
      models: {
        "anthropic/claude-sonnet-4-5": { alias: "Sonnet" },
        "lmstudio/minimax-m2.1-gs32": { alias: "MiniMax Local" },
        "anthropic/claude-opus-4-5": { alias: "Opus" }
      }
    }
  },
  models: {
    mode: "merge",
    providers: {
      lmstudio: {
        baseUrl: "http://127.0.0.1:1234/v1",
        apiKey: "lmstudio",
        api: "openai-responses",
        models: [
          {
            id: "minimax-m2.1-gs32",
            name: "MiniMax M2.1 GS32",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 196608,
            maxTokens: 8192
          }
        ]
      }
    }
  }
}
```

<div id="local-first-with-hosted-safety-net">
  ### Lokal zuerst mit gehostetem Sicherheitsnetz
</div>

Tausche die Reihenfolge von primärem und Fallback-Anbieter; behalte denselben Anbieterblock und `models.mode: "merge"` bei, damit du auf Sonnet oder Opus zurückfallen kannst, wenn der lokale Rechner ausfällt.

<div id="regional-hosting-data-routing">
  ### Regionales Hosting / Datenrouting
</div>

* Gehostete MiniMax/Kimi/GLM-Varianten sind ebenfalls auf OpenRouter mit regional gebundenen Endpunkten verfügbar (z. B. in den USA gehostet). Wähle dort die regionale Variante, um den Datenverkehr in deiner gewählten Rechtsordnung zu halten, während du weiterhin `models.mode: "merge"` für Anthropic-/OpenAI-Fallbacks verwendest.
* Rein lokal bleibt die datenschutzfreundlichste Variante; gehostetes regionales Routing ist der Mittelweg, wenn du Anbieterfunktionen benötigst, aber die Datenflüsse kontrollieren willst.

<div id="other-openai-compatible-local-proxies">
  ## Andere OpenAI-kompatible lokale Proxys
</div>

vLLM, LiteLLM, OAI-proxy oder benutzerdefinierte Gateways funktionieren, wenn sie einen OpenAI-kompatiblen `/v1`-Endpunkt bereitstellen. Ersetze den obigen `provider`-Block durch deinen Endpunkt und die Modell-ID:

```json5
{
  models: {
    mode: "merge",
    providers: {
      local: {
        baseUrl: "http://127.0.0.1:8000/v1",
        apiKey: "sk-local",
        api: "openai-responses",
        models: [
          {
            id: "my-local-model",
            name: "Local Model",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 120000,
            maxTokens: 8192
          }
        ]
      }
    }
  }
}
```

Belasse `models.mode: "merge"`, damit gehostete Modelle weiterhin als Fallback zur Verfügung stehen.

<div id="troubleshooting">
  ## Fehlerbehebung
</div>

* Kann das Gateway den Proxy erreichen? `curl http://127.0.0.1:1234/v1/models`.
* LM-Studio-Modell entladen? Neu laden; Kaltstart ist eine häufige Ursache für „Hänger“.
* Kontextfehler? `contextWindow` verkleinern oder dein Server-Limit erhöhen.
* Sicherheit: Lokale Modelle umgehen Filter auf Anbieterseite; halte Agenten eng gefasst und Kompaktierung aktiviert, um den Explosionsradius von Prompt-Injection zu begrenzen.