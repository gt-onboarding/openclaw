---
title: OpenAI
summary: "Utilizza OpenAI tramite chiavi API o abbonamento Codex in OpenClaw"
read_when:
  - Vuoi utilizzare i modelli OpenAI in OpenClaw
  - Vuoi usare l'autenticazione tramite abbonamento Codex invece delle chiavi API
---

<div id="openai">
  # OpenAI
</div>

OpenAI fornisce API di sviluppo per i modelli GPT. Codex supporta l’**accesso tramite ChatGPT** per l’uso in abbonamento oppure l’**accesso tramite API key** per l’uso a consumo. Il cloud Codex richiede l’accesso tramite ChatGPT.

<div id="option-a-openai-api-key-openai-platform">
  ## Opzione A: chiave API OpenAI (piattaforma OpenAI)
</div>

**Ideale per:** accesso diretto all&#39;API e fatturazione basata sull&#39;utilizzo.
Ottieni la tua chiave API dalla dashboard di OpenAI.

<div id="cli-setup">
  ### Configurazione della CLI
</div>

```bash
openclaw onboard --auth-choice openai-api-key
# oppure in modalità non interattiva
openclaw onboard --openai-api-key "$OPENAI_API_KEY"
```

<div id="config-snippet">
  ### Snippet di configurazione
</div>

```json5
{
  env: { OPENAI_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "openai/gpt-5.2" } } }
}
```

<div id="option-b-openai-code-codex-subscription">
  ## Opzione B: abbonamento a OpenAI Code (Codex)
</div>

**Ideale per:** usare l’accesso in abbonamento a ChatGPT/Codex invece di una chiave API.
Il cloud Codex richiede l’accesso tramite ChatGPT, mentre la CLI Codex supporta l’accesso tramite ChatGPT o con chiave API.

<div id="cli-setup">
  ### Configurazione della CLI
</div>

```bash
# Esegui l'OAuth di Codex nella procedura guidata
openclaw onboard --auth-choice openai-codex

# Oppure esegui l'OAuth direttamente
openclaw models auth login --provider openai-codex
```

### Esempio di configurazione

```json5
{
  agents: { defaults: { model: { primary: "openai-codex/gpt-5.2" } } }
}
```

<div id="notes">
  ## Note
</div>

* I riferimenti ai modelli utilizzano sempre `provider/model` (consulta [/concepts/models](/it/concepts/models)).
* I dettagli di autenticazione e le regole di riutilizzo sono descritti in [/concepts/oauth](/it/concepts/oauth).