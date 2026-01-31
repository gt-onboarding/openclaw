---
title: Modellanbieter
summary: "Übersicht über Modellanbieter mit Beispielkonfigurationen + CLI-Flows"
read_when:
  - Du benötigst eine nach Anbietern gegliederte Referenz zur Modelleinrichtung
  - Du möchtest Beispielkonfigurationen oder CLI-Onboarding-Befehle für Modellanbieter
---

<div id="model-providers">
  # Modellanbieter
</div>

Diese Seite behandelt **LLM-/Modellanbieter** (nicht Chat-Kanäle wie WhatsApp/Telegram).
Regeln zur Modellauswahl findest du unter [/concepts/models](/de/concepts/models).

<div id="quick-rules">
  ## Kurzregeln
</div>

* Modell-Referenzen verwenden das Format `provider/model` (Beispiel: `opencode/claude-opus-4-5`).
* Wenn du `agents.defaults.models` setzt, dient es als Allowlist.
* CLI-Helfer: `openclaw onboard`, `openclaw models list`, `openclaw models set <provider/model>`.

<div id="built-in-providers-pi-ai-catalog">
  ## Eingebaute Anbieter (pi-ai-Katalog)
</div>

OpenClaw wird mit dem pi‑ai-Katalog ausgeliefert. Diese Anbieter benötigen **keine**
`models.providers`-Konfiguration; du musst nur die Authentifizierung einrichten und ein Modell auswählen.

<div id="openai">
  ### OpenAI
</div>

* Anbieter: `openai`
* Auth: `OPENAI_API_KEY`
* Beispielmodell: `openai/gpt-5.2`
* CLI: `openclaw onboard --auth-choice openai-api-key`

```json5
{
  agents: { defaults: { model: { primary: "openai/gpt-5.2" } } }
}
```

<div id="anthropic">
  ### Anthropic
</div>

* Anbieter: `anthropic`
* Auth: `ANTHROPIC_API_KEY` oder `claude setup-token`
* Beispielmodell: `anthropic/claude-opus-4-5`
* CLI: `openclaw onboard --auth-choice token` (Setup-Token einfügen) oder `openclaw models auth paste-token --provider anthropic`

```json5
{
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-5" } } }
}
```

<div id="openai-code-codex">
  ### OpenAI Code (Codex)
</div>

* Anbieter: `openai-codex`
* Auth: OAuth (ChatGPT)
* Beispielmodell: `openai-codex/gpt-5.2`
* CLI: `openclaw onboard --auth-choice openai-codex` oder `openclaw models auth login --provider openai-codex`

```json5
{
  agents: { defaults: { model: { primary: "openai-codex/gpt-5.2" } } }
}
```

<div id="opencode-zen">
  ### OpenCode Zen
</div>

* Anbieter: `opencode`
* Auth: `OPENCODE_API_KEY` (oder `OPENCODE_ZEN_API_KEY`)
* Beispielmodell: `opencode/claude-opus-4-5`
* CLI: `openclaw onboard --auth-choice opencode-zen`

```json5
{
  agents: { defaults: { model: { primary: "opencode/claude-opus-4-5" } } }
}
```

<div id="google-gemini-api-key">
  ### Google Gemini (API-Schlüssel)
</div>

* Anbieter: `google`
* Auth: `GEMINI_API_KEY`
* Beispielmodell: `google/gemini-3-pro-preview`
* CLI: `openclaw onboard --auth-choice gemini-api-key`

<div id="google-vertex-antigravity-gemini-cli">
  ### Google Vertex / Antigravity / Gemini CLI
</div>

* Anbieter: `google-vertex`, `google-antigravity`, `google-gemini-cli`
* Authentifizierung: Vertex verwendet gcloud ADC; Antigravity/Gemini CLI verwenden ihre jeweiligen Authentifizierungsabläufe
* Antigravity OAuth wird als gebündeltes Plugin ausgeliefert (`google-antigravity-auth`, standardmäßig deaktiviert).
  * Aktivieren: `openclaw plugins enable google-antigravity-auth`
  * Login: `openclaw models auth login --provider google-antigravity --set-default`
* Gemini CLI OAuth wird als gebündeltes Plugin ausgeliefert (`google-gemini-cli-auth`, standardmäßig deaktiviert).
  * Aktivieren: `openclaw plugins enable google-gemini-cli-auth`
  * Login: `openclaw models auth login --provider google-gemini-cli --set-default`
  * Hinweis: Sie fügen **keine** Client-ID und kein Secret in `openclaw.json` ein. Der CLI-Login-Flow speichert
    Token in Auth-Profilen auf dem Gateway-Host.

<div id="zai-glm">
  ### Z.AI (GLM)
</div>

* Anbieter: `zai`
* Auth: `ZAI_API_KEY`
* Beispielmodell: `zai/glm-4.7`
* CLI: `openclaw onboard --auth-choice zai-api-key`
  * Aliasse: `z.ai/*` und `z-ai/*` werden zu `zai/*` normalisiert

<div id="vercel-ai-gateway">
  ### Vercel AI Gateway
</div>

* Anbieter: `vercel-ai-gateway`
* Authentifizierung: `AI_GATEWAY_API_KEY`
* Beispielmodell: `vercel-ai-gateway/anthropic/claude-opus-4.5`
* CLI: `openclaw onboard --auth-choice ai-gateway-api-key`

<div id="other-built-in-providers">
  ### Weitere integrierte Anbieter
</div>

* OpenRouter: `openrouter` (`OPENROUTER_API_KEY`)
* Beispielmodell: `openrouter/anthropic/claude-sonnet-4-5`
* xAI: `xai` (`XAI_API_KEY`)
* Groq: `groq` (`GROQ_API_KEY`)
* Cerebras: `cerebras` (`CEREBRAS_API_KEY`)
  * GLM-Modelle auf Cerebras verwenden die IDs `zai-glm-4.7` und `zai-glm-4.6`.
  * OpenAI-kompatible Basis-URL: `https://api.cerebras.ai/v1`.
* Mistral: `mistral` (`MISTRAL_API_KEY`)
* GitHub Copilot: `github-copilot` (`COPILOT_GITHUB_TOKEN` / `GH_TOKEN` / `GITHUB_TOKEN`)

<div id="providers-via-modelsproviders-custombase-url">
  ## Anbieter über `models.providers` (benutzerdefinierte/Basis‑URL)
</div>

Verwende `models.providers` (oder `models.json`), um **benutzerdefinierte** Anbieter oder
OpenAI-/Anthropic‑kompatible Proxys hinzuzufügen.

<div id="moonshot-ai-kimi">
  ### Moonshot AI (Kimi)
</div>

Moonshot verwendet OpenAI-kompatible Endpunkte, daher konfigurierst du Moonshot als benutzerdefinierten Anbieter:

* Anbieter: `moonshot`
* Auth: `MOONSHOT_API_KEY`
* Beispielmodell: `moonshot/kimi-k2.5`
* Kimi-K2-Modell-IDs:
  {/* moonshot-kimi-k2-model-refs:start */}
  * `moonshot/kimi-k2.5`
  * `moonshot/kimi-k2-0905-preview`
  * `moonshot/kimi-k2-turbo-preview`
  * `moonshot/kimi-k2-thinking`
  * `moonshot/kimi-k2-thinking-turbo`
  {/* moonshot-kimi-k2-model-refs:end */}

```json5
{
  agents: {
    defaults: { model: { primary: "moonshot/kimi-k2.5" } }
  },
  models: {
    mode: "merge",
    providers: {
      moonshot: {
        baseUrl: "https://api.moonshot.ai/v1",
        apiKey: "${MOONSHOT_API_KEY}",
        api: "openai-completions",
        models: [{ id: "kimi-k2.5", name: "Kimi K2.5" }]
      }
    }
  }
}
```

<div id="kimi-code">
  ### Kimi Code
</div>

Kimi Code verwendet einen eigenen Endpoint und Schlüssel (getrennt von Moonshot):

* Anbieter: `kimi-code`
* Auth: `KIMICODE_API_KEY`
* Beispielmodell: `kimi-code/kimi-for-coding`

```json5
{
  env: { KIMICODE_API_KEY: "sk-..." },
  agents: {
    defaults: { model: { primary: "kimi-code/kimi-for-coding" } }
  },
  models: {
    mode: "merge",
    providers: {
      "kimi-code": {
        baseUrl: "https://api.kimi.com/coding/v1",
        apiKey: "${KIMICODE_API_KEY}",
        api: "openai-completions",
        models: [{ id: "kimi-for-coding", name: "Kimi For Coding" }]
      }
    }
  }
}
```

<div id="qwen-oauth-free-tier">
  ### Qwen OAuth (Free-Tarif)
</div>

Qwen stellt über einen Device-Code-Flow OAuth-Zugriff auf Qwen Coder + Vision bereit.
Aktiviere das mitgelieferte Plugin und melde dich anschließend an:

```bash
openclaw plugins enable qwen-portal-auth
openclaw models auth login --provider qwen-portal --set-default
```

Modell-Referenzen:

* `qwen-portal/coder-model`
* `qwen-portal/vision-model`

Siehe [/providers/qwen](/de/providers/qwen) für Details zur Einrichtung und Hinweise.

<div id="synthetic">
  ### Synthetic
</div>

Synthetic stellt Anthropic-kompatible Modelle über den Anbieter `synthetic` bereit:

* Provider: `synthetic`
* Auth: `SYNTHETIC_API_KEY`
* Beispielmodell: `synthetic/hf:MiniMaxAI/MiniMax-M2.1`
* CLI: `openclaw onboard --auth-choice synthetic-api-key`

```json5
{
  agents: {
    defaults: { model: { primary: "synthetic/hf:MiniMaxAI/MiniMax-M2.1" } }
  },
  models: {
    mode: "merge",
    providers: {
      synthetic: {
        baseUrl: "https://api.synthetic.new/anthropic",
        apiKey: "${SYNTHETIC_API_KEY}",
        api: "anthropic-messages",
        models: [{ id: "hf:MiniMaxAI/MiniMax-M2.1", name: "MiniMax M2.1" }]
      }
    }
  }
}
```

<div id="minimax">
  ### MiniMax
</div>

MiniMax wird über `models.providers` konfiguriert, da es benutzerdefinierte Endpunkte verwendet:

* MiniMax (Anthropic‑kompatibel): `--auth-choice minimax-api`
* Auth: `MINIMAX_API_KEY`

Siehe [/providers/minimax](/de/providers/minimax) für Details zur Einrichtung, zu Modelloptionen und Konfigurationsbeispielen.

<div id="ollama">
  ### Ollama
</div>

Ollama ist eine lokale LLM-Laufzeitumgebung, die eine OpenAI-kompatible API bereitstellt:

* Anbieter: `ollama`
* Auth: Keine Authentifizierung erforderlich (lokaler Server)
* Beispielmodell: `ollama/llama3.3`
* Installation: https://ollama.ai

```bash
# Installieren Sie Ollama, dann laden Sie ein Modell herunter:
ollama pull llama3.3
```

```json5
{
  agents: {
    defaults: { model: { primary: "ollama/llama3.3" } }
  }
}
```

Ollama wird automatisch erkannt, wenn es lokal unter `http://127.0.0.1:11434/v1` ausgeführt wird. Siehe [/providers/ollama](/de/providers/ollama) für Empfehlungen zu Modellen und zur individuellen Konfiguration.

<div id="local-proxies-lm-studio-vllm-litellm-etc">
  ### Lokale Proxys (LM Studio, vLLM, LiteLLM usw.)
</div>

Beispiel (OpenAI‑kompatibel):

```json5
{
  agents: {
    defaults: {
      model: { primary: "lmstudio/minimax-m2.1-gs32" },
      models: { "lmstudio/minimax-m2.1-gs32": { alias: "Minimax" } }
    }
  },
  models: {
    providers: {
      lmstudio: {
        baseUrl: "http://localhost:1234/v1",
        apiKey: "LMSTUDIO_KEY",
        api: "openai-completions",
        models: [
          {
            id: "minimax-m2.1-gs32",
            name: "MiniMax M2.1",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 200000,
            maxTokens: 8192
          }
        ]
      }
    }
  }
}
```

Hinweise:

* Für benutzerdefinierte Anbieter sind `reasoning`, `input`, `cost`, `contextWindow` und `maxTokens` optional.
  Wenn diese weggelassen werden, verwendet OpenClaw standardmäßig:
  * `reasoning: false`
  * `input: ["text"]`
  * `cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 }`
  * `contextWindow: 200000`
  * `maxTokens: 8192`
* Empfehlung: Lege explizite Werte fest, die zu den Limits deines Proxy/Modells passen.

<div id="cli-examples">
  ## CLI-Beispiele
</div>

```bash
openclaw onboard --auth-choice opencode-zen
openclaw models set opencode/claude-opus-4-5
openclaw models list
```

Siehe auch: [/gateway/configuration](/de/gateway/configuration) für vollständige Konfigurationsbeispiele.
