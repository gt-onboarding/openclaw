---
title: Provider di modelli
summary: "Panoramica dei provider di modelli con esempi di configurazione e flussi tramite CLI"
read_when:
  - Ti serve un riferimento alla configurazione dei modelli per ciascun provider
  - Vuoi esempi di configurazioni o comandi per l'onboarding tramite CLI dei provider di modelli
---

<div id="model-providers">
  # Provider di modelli
</div>

Questa pagina riguarda i **LLM/provider di modelli** (non i canali di chat come WhatsApp/Telegram).
Per le regole di selezione dei modelli, vedi [/concepts/models](/it/concepts/models).

<div id="quick-rules">
  ## Regole rapide
</div>

* I riferimenti ai modelli usano `provider/model` (esempio: `opencode/claude-opus-4-5`).
* Se imposti `agents.defaults.models`, questo diventa la lista di autorizzati.
* Utilità CLI: `openclaw onboard`, `openclaw models list`, `openclaw models set <provider/model>`.

<div id="built-in-providers-pi-ai-catalog">
  ## Provider integrati (catalogo pi-ai)
</div>

OpenClaw include il catalogo pi‑ai. Questi provider **non richiedono**
alcuna configurazione `models.providers`; devi solo impostare l’autenticazione e scegliere un modello.

<div id="openai">
  ### OpenAI
</div>

* Provider: `openai`
* Autenticazione: `OPENAI_API_KEY`
* Modello di esempio: `openai/gpt-5.2`
* CLI: `openclaw onboard --auth-choice openai-api-key`

```json5
{
  agents: { defaults: { model: { primary: "openai/gpt-5.2" } } }
}
```

<div id="anthropic">
  ### Anthropic
</div>

* Provider: `anthropic`
* Auth: `ANTHROPIC_API_KEY` oppure `claude setup-token`
* Modello di esempio: `anthropic/claude-opus-4-5`
* CLI: `openclaw onboard --auth-choice token` (incolla il setup-token) oppure `openclaw models auth paste-token --provider anthropic`

```json5
{
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-5" } } }
}
```

<div id="openai-code-codex">
  ### OpenAI Code (Codex)
</div>

* Provider: `openai-codex`
* Autenticazione: OAuth (ChatGPT)
* Esempio di modello: `openai-codex/gpt-5.2`
* CLI: `openclaw onboard --auth-choice openai-codex` oppure `openclaw models auth login --provider openai-codex`

```json5
{
  agents: { defaults: { model: { primary: "openai-codex/gpt-5.2" } } }
}
```

<div id="opencode-zen">
  ### OpenCode Zen
</div>

* Provider: `opencode`
* Autenticazione: `OPENCODE_API_KEY` (oppure `OPENCODE_ZEN_API_KEY`)
* Modello di esempio: `opencode/claude-opus-4-5`
* CLI: `openclaw onboard --auth-choice opencode-zen`

```json5
{
  agents: { defaults: { model: { primary: "opencode/claude-opus-4-5" } } }
}
```

<div id="google-gemini-api-key">
  ### Google Gemini (chiave API)
</div>

* Provider: `google`
* Autenticazione: `GEMINI_API_KEY`
* Modello di esempio: `google/gemini-3-pro-preview`
* CLI: `openclaw onboard --auth-choice gemini-api-key`

<div id="google-vertex-antigravity-gemini-cli">
  ### Google Vertex / Antigravity / Gemini CLI
</div>

* Provider: `google-vertex`, `google-antigravity`, `google-gemini-cli`
* Autenticazione: Vertex usa gcloud ADC; Antigravity/Gemini CLI usano i rispettivi flussi di autenticazione
* L’autenticazione OAuth di Antigravity è fornita come plugin in bundle (`google-antigravity-auth`, disabilitato per impostazione predefinita).
  * Abilita: `openclaw plugins enable google-antigravity-auth`
  * Login: `openclaw models auth login --provider google-antigravity --set-default`
* L’autenticazione OAuth di Gemini CLI è fornita come plugin in bundle (`google-gemini-cli-auth`, disabilitato per impostazione predefinita).
  * Abilita: `openclaw plugins enable google-gemini-cli-auth`
  * Login: `openclaw models auth login --provider google-gemini-cli --set-default`
  * Nota: **non** devi incollare un client ID o secret in `openclaw.json`. Il flusso di login della CLI memorizza
    i token nei profili di autenticazione sull’host del Gateway.

<div id="zai-glm">
  ### Z.AI (GLM)
</div>

* Provider: `zai`
* Autenticazione: `ZAI_API_KEY`
* Esempio di modello: `zai/glm-4.7`
* CLI: `openclaw onboard --auth-choice zai-api-key`
  * Alias: `z.ai/*` e `z-ai/*` si normalizzano in `zai/*`

<div id="vercel-ai-gateway">
  ### Vercel AI Gateway
</div>

* Provider: `vercel-ai-gateway`
* Autenticazione: `AI_GATEWAY_API_KEY`
* Modello di esempio: `vercel-ai-gateway/anthropic/claude-opus-4.5`
* CLI: `openclaw onboard --auth-choice ai-gateway-api-key`

<div id="other-built-in-providers">
  ### Altri provider integrati
</div>

* OpenRouter: `openrouter` (`OPENROUTER_API_KEY`)
* Modello di esempio: `openrouter/anthropic/claude-sonnet-4-5`
* xAI: `xai` (`XAI_API_KEY`)
* Groq: `groq` (`GROQ_API_KEY`)
* Cerebras: `cerebras` (`CEREBRAS_API_KEY`)
  * I modelli GLM su Cerebras usano gli ID `zai-glm-4.7` e `zai-glm-4.6`.
  * URL di base compatibile con OpenAI: `https://api.cerebras.ai/v1`.
* Mistral: `mistral` (`MISTRAL_API_KEY`)
* GitHub Copilot: `github-copilot` (`COPILOT_GITHUB_TOKEN` / `GH_TOKEN` / `GITHUB_TOKEN`)

<div id="providers-via-modelsproviders-custombase-url">
  ## Provider tramite `models.providers` (URL di base/personalizzato)
</div>

Usa `models.providers` (o `models.json`) per aggiungere provider **personalizzati** o
proxy compatibili con OpenAI/Anthropic.

<div id="moonshot-ai-kimi">
  ### Moonshot AI (Kimi)
</div>

Moonshot utilizza endpoint compatibili con OpenAI, quindi configuralo come un provider personalizzato:

* Provider: `moonshot`
* Auth: `MOONSHOT_API_KEY`
* Esempio di modello: `moonshot/kimi-k2.5`
* ID dei modelli Kimi K2:
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

Kimi Code usa un endpoint e una chiave dedicati (separati da quelli di Moonshot):

* Provider: `kimi-code`
* Auth: `KIMICODE_API_KEY`
* Modello di esempio: `kimi-code/kimi-for-coding`

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
  ### Qwen OAuth (piano gratuito)
</div>

Qwen fornisce accesso OAuth a Qwen Coder + Vision tramite un flusso device code.
Abilita il plugin integrato, quindi accedi:

```bash
openclaw plugins enable qwen-portal-auth
openclaw models auth login --provider qwen-portal --set-default
```

Riferimenti ai modelli:

* `qwen-portal/coder-model`
* `qwen-portal/vision-model`

Vedi [/providers/qwen](/it/providers/qwen) per i dettagli sulla configurazione e le note.

<div id="synthetic">
  ### Synthetic
</div>

Synthetic fornisce modelli compatibili con Anthropic tramite il provider `synthetic`:

* Provider: `synthetic`
* Auth: `SYNTHETIC_API_KEY`
* Example model: `synthetic/hf:MiniMaxAI/MiniMax-M2.1`
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

MiniMax è configurato tramite `models.providers` perché utilizza endpoint personalizzati:

* MiniMax (compatibile con Anthropic): `--auth-choice minimax-api`
* Autenticazione: `MINIMAX_API_KEY`

Vedi [/providers/minimax](/it/providers/minimax) per i dettagli di configurazione, le opzioni dei modelli e gli snippet di configurazione.

<div id="ollama">
  ### Ollama
</div>

Ollama è un runtime LLM locale che espone un&#39;API compatibile con OpenAI:

* Provider: `ollama`
* Auth: Nessuna autenticazione richiesta (server locale)
* Esempio di modello: `ollama/llama3.3`
* Installazione: https://ollama.ai

```bash
# Installa Ollama, quindi esegui il pull di un modello:
ollama pull llama3.3
```

```json5
{
  agents: {
    defaults: { model: { primary: "ollama/llama3.3" } }
  }
}
```

Ollama viene rilevato automaticamente quando viene eseguito in locale all&#39;indirizzo `http://127.0.0.1:11434/v1`. Consulta [/providers/ollama](/it/providers/ollama) per raccomandazioni sui modelli e per la configurazione personalizzata.

<div id="local-proxies-lm-studio-vllm-litellm-etc">
  ### Proxy locali (LM Studio, vLLM, LiteLLM, ecc.)
</div>

Esempio (compatibile con OpenAI):

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

Note:

* Per i provider personalizzati, `reasoning`, `input`, `cost`, `contextWindow` e `maxTokens` sono opzionali.
  Se non specificati, OpenClaw applica i seguenti valori predefiniti:
  * `reasoning: false`
  * `input: ["text"]`
  * `cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 }`
  * `contextWindow: 200000`
  * `maxTokens: 8192`
* Consigliato: imposta valori espliciti che siano in linea con i limiti del tuo proxy/modello.

<div id="cli-examples">
  ## Esempi di utilizzo della CLI
</div>

```bash
openclaw onboard --auth-choice opencode-zen
openclaw models set opencode/claude-opus-4-5
openclaw models list
```

Consulta anche: [/gateway/configuration](/it/gateway/configuration) per esempi completi di configurazione.
