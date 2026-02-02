---
title: Test
summary: "Kit di test: suite di test unitari/e2e/live, runner Docker e cosa verifica ciascun test"
read_when:
  - Esecuzione dei test in locale o in CI
  - Aggiunta di test di regressione per bug di modello/provider
  - Debugging del comportamento di Gateway e agenti
---

<div id="testing">
  # Testing
</div>

OpenClaw ha tre suite Vitest (unità/integrazione, e2e, live) e un piccolo set di runner Docker.

Questo documento è una guida “come testiamo”:

* Cosa copre ciascuna suite (e cosa *non* copre deliberatamente)
* Quali comandi eseguire per i flussi di lavoro più comuni (locale, pre-push, debugging)
* Come i test live rilevano le credenziali e selezionano modelli/provider
* Come aggiungere test di regressione per problemi reali di modelli/provider

<div id="quick-start">
  ## Avvio rapido
</div>

Di solito:

* Gate completo (previsto prima del push): `pnpm lint && pnpm build && pnpm test`

Quando modifichi i test o vuoi maggiore sicurezza:

* Gate di coverage: `pnpm test:coverage`
* Suite E2E: `pnpm test:e2e`

Quando esegui il debug di provider/modelli reali (richiede credenziali reali):

* Suite live (modelli + sonde degli strumenti/immagini del Gateway): `pnpm test:live`

Suggerimento: quando ti basta un singolo caso che fallisce, è preferibile restringere i test live tramite le variabili d&#39;ambiente della lista di autorizzati descritte di seguito.

<div id="test-suites-what-runs-where">
  ## Suite di test (cosa viene eseguito dove)
</div>

Pensa alle suite come a livelli di “realismo crescente” (e, con esso, di maggiore instabilità/costo):

<div id="unit-integration-default">
  ### Unit / integrazione (predefinito)
</div>

* Comando: `pnpm test`
* Config: `vitest.config.ts`
* File: `src/**/*.test.ts`
* Scope:
  * Test unitari puri
  * Test di integrazione in-process (autenticazione del Gateway, routing, tooling, parsing, config)
  * Test di regressione deterministici per bug noti
* Aspettative:
  * Viene eseguita in CI
  * Non richiede chiavi reali
  * Dovrebbe essere veloce e stabile

<div id="e2e-gateway-smoke">
  ### E2E (smoke test del Gateway)
</div>

* Command: `pnpm test:e2e`
* Config: `vitest.e2e.config.ts`
* Files: `src/**/*.e2e.test.ts`
* Scope:
  * Comportamento end-to-end multi‑istanza del Gateway
  * Superfici WebSocket/HTTP, abbinamento dei nodi e carico di rete più intenso
* Expectations:
  * Viene eseguita in CI (quando abilitata nella pipeline)
  * Non richiede chiavi reali
  * Più componenti in gioco rispetto ai test unitari (può essere più lenta)

<div id="live-real-providers-real-models">
  ### Live (provider reali + modelli reali)
</div>

* Command: `pnpm test:live`
* Config: `vitest.live.config.ts`
* Files: `src/**/*.live.test.ts`
* Default: **abilitata** da `pnpm test:live` (imposta `OPENCLAW_LIVE_TEST=1`)
* Scope:
  * &quot;Questo provider/modello funziona davvero *oggi* con credenziali reali?&quot;
  * Intercetta modifiche al formato del provider, peculiarità del tool-calling, problemi di autenticazione e comportamento dei rate limit
* Expectations:
  * Non stabile per la CI per scelta (reti reali, policy reali dei provider, quote, interruzioni del servizio)
  * Ha un costo / utilizza i rate limit
  * È preferibile eseguire sottoinsiemi ristretti invece di &quot;tutto&quot;
  * Le esecuzioni live caricheranno `~/.profile` per rilevare eventuali chiavi API mancanti
  * Rotazione delle chiavi Anthropic: imposta `OPENCLAW_LIVE_ANTHROPIC_KEYS="sk-...,sk-..."` (oppure `OPENCLAW_LIVE_ANTHROPIC_KEY=sk-...`) o più variabili `ANTHROPIC_API_KEY*`; i test riproveranno in caso di rate limit

<div id="which-suite-should-i-run">
  ## Quale suite dovrei eseguire?
</div>

Usa questa tabella decisionale:

* Se stai modificando la logica o i test: esegui `pnpm test` (ed eventualmente `pnpm test:coverage` se hai cambiato molte cose)
* Se stai modificando il networking del Gateway / il protocollo WS / l’abbinamento: esegui anche `pnpm test:e2e`
* Per eseguire il debug di “il mio bot è down” / errori specifici di un provider / chiamate agli strumenti: esegui una versione ristretta di `pnpm test:live`

<div id="live-model-smoke-profile-keys">
  ## Live: smoke test del modello (chiavi del profilo)
</div>

I test live sono suddivisi in due livelli per poter isolare i problemi:

* Il “modello diretto” ci dice che il provider/modello è in grado di rispondere con la chiave fornita.
* Il “gateway smoke” ci dice che l&#39;intera pipeline Gateway+agente funziona per quel modello (sessioni, cronologia, strumenti, policy della sandbox, ecc.).

<div id="layer-1-direct-model-completion-no-gateway">
  ### Layer 1: completamento diretto del modello (senza Gateway)
</div>

* Test: `src/agents/models.profiles.live.test.ts`
* Obiettivo:
  * Enumerare i modelli individuati
  * Usare `getApiKeyForModel` per selezionare i modelli per cui hai le credenziali
  * Eseguire un piccolo completamento per modello (e regressioni mirate dove necessario)
* Come abilitarlo:
  * `pnpm test:live` (oppure `OPENCLAW_LIVE_TEST=1` se richiami direttamente Vitest)
* Imposta `OPENCLAW_LIVE_MODELS=modern` (oppure `all`, alias di modern) per eseguire effettivamente questa suite; altrimenti viene saltata per mantenere `pnpm test:live` focalizzato sullo smoke test del Gateway
* Come selezionare i modelli:
  * `OPENCLAW_LIVE_MODELS=modern` per eseguire la lista di autorizzati moderna (Opus/Sonnet/Haiku 4.5, GPT-5.x + Codex, Gemini 3, GLM 4.7, MiniMax M2.1, Grok 4)
  * `OPENCLAW_LIVE_MODELS=all` è un alias per la lista di autorizzati moderna
  * oppure `OPENCLAW_LIVE_MODELS="openai/gpt-5.2,anthropic/claude-opus-4-5,..."` (lista di autorizzati separata da virgole)
* Come selezionare i provider:
  * `OPENCLAW_LIVE_PROVIDERS="google,google-antigravity,google-gemini-cli"` (lista di autorizzati separata da virgole)
* Da dove provengono le chiavi:
  * Predefinito: archivio dei profili e variabili d&#39;ambiente come fallback
  * Imposta `OPENCLAW_LIVE_REQUIRE_PROFILE_KEYS=1` per imporre **solo l&#39;archivio dei profili**
* Perché esiste:
  * Separa “la API del provider non funziona / la chiave non è valida” da “la pipeline dell&#39;agente nel Gateway non funziona”
  * Contiene piccole regressioni isolate (esempio: flussi di reasoning replay + tool-call di OpenAI Responses/Codex Responses)

<div id="layer-2-gateway-dev-agent-smoke-what-openclaw-actually-does">
  ### Livello 2: smoke test Gateway + agente di sviluppo (cosa fa davvero “@openclaw”)
</div>

* Test: `src/gateway/gateway-models.profiles.live.test.ts`
* Obiettivo:
  * Avviare un Gateway in-process
  * Creare/patchare una sessione `agent:dev:*` (override del modello per singola esecuzione)
  * Iterare i modelli con chiave e verificare:
    * risposta “significativa” (nessun tool)
    * che una vera invocazione di tool funzioni (probe read)
    * probe opzionali di tool aggiuntivi (probe exec+read)
    * che i percorsi di regressione OpenAI (solo tool-call → follow-up) continuino a funzionare
* Dettagli delle probe (così puoi spiegare rapidamente gli errori):
  * probe `read`: il test scrive un file nonce nello spazio di lavoro e chiede all’agente di eseguire `read` sul file e fare echo del nonce.
  * probe `exec+read`: il test chiede all’agente di `exec`-scrivere un nonce in un file temporaneo, quindi di eseguire `read` sul file.
  * probe immagine: il test allega un PNG generato (gatto + codice randomizzato) e si aspetta che il modello restituisca `cat <CODE>`.
  * Riferimento di implementazione: `src/gateway/gateway-models.profiles.live.test.ts` e `src/gateway/live-image-probe.ts`.
* Come abilitare:
  * `pnpm test:live` (oppure `OPENCLAW_LIVE_TEST=1` se invochi direttamente Vitest)
* Come selezionare i modelli:
  * Default: lista di autorizzati moderna (Opus/Sonnet/Haiku 4.5, GPT-5.x + Codex, Gemini 3, GLM 4.7, MiniMax M2.1, Grok 4)
  * `OPENCLAW_LIVE_GATEWAY_MODELS=all` è un alias per la lista di autorizzati moderna
  * Oppure imposta `OPENCLAW_LIVE_GATEWAY_MODELS="provider/model"` (o una lista separata da virgole) per restringere
* Come selezionare i provider (evita “OpenRouter su tutto”):
  * `OPENCLAW_LIVE_GATEWAY_PROVIDERS="google,google-antigravity,google-gemini-cli,openai,anthropic,zai,minimax"` (lista di autorizzati separata da virgole)
* Le probe di tool + immagine sono sempre attive in questo live test:
  * probe `read` + probe `exec+read` (stress sui tool)
  * la probe immagine viene eseguita quando il modello dichiara il supporto per input immagine
  * Flusso (alto livello):
    * Il test genera un piccolo PNG con “CAT” + codice random (`src/gateway/live-image-probe.ts`)
    * Lo invia tramite `agent` `attachments: [{ mimeType: "image/png", content: "<base64>" }]`
    * Il Gateway esegue il parse degli attachment in `images[]` (`src/gateway/server-methods/agent.ts` + `src/gateway/chat-attachments.ts`)
    * L’agente embedded inoltra un messaggio utente multimodale al modello
    * Verifica: la risposta contiene `cat` + il codice (tolleranza OCR: sono permessi piccoli errori)

Suggerimento: per vedere cosa puoi testare sulla tua macchina (e gli ID esatti `provider/model`), esegui:

```bash
openclaw models list
openclaw models list --json
```

<div id="live-anthropic-setup-token-smoke">
  ## Live: smoke test setup-token Anthropic
</div>

* Test: `src/agents/anthropic.setup-token.live.test.ts`
* Obiettivo: verificare che il comando `setup-token` di Claude Code CLI (o un profilo setup-token incollato) possa completare un prompt Anthropic.
* Per abilitare:
  * `pnpm test:live` (o `OPENCLAW_LIVE_TEST=1` se richiami direttamente Vitest)
  * `OPENCLAW_LIVE_SETUP_TOKEN=1`
* Origine del token (scegline una):
  * Profilo: `OPENCLAW_LIVE_SETUP_TOKEN_PROFILE=anthropic:setup-token-test`
  * Token grezzo: `OPENCLAW_LIVE_SETUP_TOKEN_VALUE=sk-ant-oat01-...`
* Override del modello (opzionale):
  * `OPENCLAW_LIVE_SETUP_TOKEN_MODEL=anthropic/claude-opus-4-5`

Esempio di setup:

```bash
openclaw models auth paste-token --provider anthropic --profile-id anthropic:setup-token-test
OPENCLAW_LIVE_SETUP_TOKEN=1 OPENCLAW_LIVE_SETUP_TOKEN_PROFILE=anthropic:setup-token-test pnpm test:live src/agents/anthropic.setup-token.live.test.ts
```

<div id="live-cli-backend-smoke-claude-code-cli-or-other-local-clis">
  ## Live: CLI backend smoke (Claude Code CLI o altre CLI locali)
</div>

* Test: `src/gateway/gateway-cli-backend.live.test.ts`
* Obiettivo: convalidare la pipeline Gateway + agente usando un backend CLI locale, senza toccare la configurazione predefinita.
* Abilita:
  * `pnpm test:live` (o `OPENCLAW_LIVE_TEST=1` se invochi direttamente Vitest)
  * `OPENCLAW_LIVE_CLI_BACKEND=1`
* Valori predefiniti:
  * Modello: `claude-cli/claude-sonnet-4-5`
  * Comando: `claude`
  * Argomenti: `["-p","--output-format","json","--dangerously-skip-permissions"]`
* Override (facoltativi):
  * `OPENCLAW_LIVE_CLI_BACKEND_MODEL="claude-cli/claude-opus-4-5"`
  * `OPENCLAW_LIVE_CLI_BACKEND_MODEL="codex-cli/gpt-5.2-codex"`
  * `OPENCLAW_LIVE_CLI_BACKEND_COMMAND="/full/path/to/claude"`
  * `OPENCLAW_LIVE_CLI_BACKEND_ARGS='["-p","--output-format","json","--permission-mode","bypassPermissions"]'`
  * `OPENCLAW_LIVE_CLI_BACKEND_CLEAR_ENV='["ANTHROPIC_API_KEY","ANTHROPIC_API_KEY_OLD"]'`
  * `OPENCLAW_LIVE_CLI_BACKEND_IMAGE_PROBE=1` per inviare un vero allegato immagine (i percorsi vengono iniettati nel prompt).
  * `OPENCLAW_LIVE_CLI_BACKEND_IMAGE_ARG="--image"` per passare i percorsi dei file immagine come argomenti CLI invece che tramite injection nel prompt.
  * `OPENCLAW_LIVE_CLI_BACKEND_IMAGE_MODE="repeat"` (o `"list"`) per controllare come vengono passati gli argomenti immagine quando `IMAGE_ARG` è impostato.
  * `OPENCLAW_LIVE_CLI_BACKEND_RESUME_PROBE=1` per inviare un secondo turno e convalidare il flusso di ripresa.
* `OPENCLAW_LIVE_CLI_BACKEND_DISABLE_MCP_CONFIG=0` per mantenere abilitata la configurazione MCP di Claude Code CLI (per impostazione predefinita la configurazione MCP viene disabilitata con un file vuoto temporaneo).

Esempio:

```bash
OPENCLAW_LIVE_CLI_BACKEND=1 \
  OPENCLAW_LIVE_CLI_BACKEND_MODEL="claude-cli/claude-sonnet-4-5" \
  pnpm test:live src/gateway/gateway-cli-backend.live.test.ts
```

<div id="recommended-live-recipes">
  ### Ricette live consigliate
</div>

Le liste di autorizzati ristrette ed esplicite sono le più veloci e le più stabili:

* Singolo modello, diretto (nessun gateway):
  * `OPENCLAW_LIVE_MODELS="openai/gpt-5.2" pnpm test:live src/agents/models.profiles.live.test.ts`

* Singolo modello, smoke test via Gateway:
  * `OPENCLAW_LIVE_GATEWAY_MODELS="openai/gpt-5.2" pnpm test:live src/gateway/gateway-models.profiles.live.test.ts`

* Invocazione di tool su più provider:
  * `OPENCLAW_LIVE_GATEWAY_MODELS="openai/gpt-5.2,anthropic/claude-opus-4-5,google/gemini-3-flash-preview,zai/glm-4.7,minimax/minimax-m2.1" pnpm test:live src/gateway/gateway-models.profiles.live.test.ts`

* Focus Google (chiave API Gemini + Antigravity):
  * Gemini (chiave API): `OPENCLAW_LIVE_GATEWAY_MODELS="google/gemini-3-flash-preview" pnpm test:live src/gateway/gateway-models.profiles.live.test.ts`
  * Antigravity (OAuth): `OPENCLAW_LIVE_GATEWAY_MODELS="google-antigravity/claude-opus-4-5-thinking,google-antigravity/gemini-3-pro-high" pnpm test:live src/gateway/gateway-models.profiles.live.test.ts`

Note:

* `google/...` usa la Gemini API (chiave API).
* `google-antigravity/...` usa il bridge Antigravity OAuth (endpoint di Agente in stile Cloud Code Assist).
* `google-gemini-cli/...` usa la CLI Gemini locale sulla tua macchina (autenticazione separata + particolarità degli strumenti).
* Gemini API vs Gemini CLI:
  * API: OpenClaw chiama la Gemini API ospitata da Google via HTTP (chiave API / profilo di autenticazione); è ciò che la maggior parte degli utenti intende con “Gemini”.
  * CLI: OpenClaw esegue un binario `gemini` locale; ha una propria autenticazione e può comportarsi in modo diverso (supporto per streaming/tool/disallineamenti di versione).

<div id="live-model-matrix-what-we-cover">
  ## Live: matrice dei modelli (cosa copriamo)
</div>

Non esiste un “elenco di modelli CI” fisso (la modalità live è opt-in), ma questi sono i modelli **consigliati** da testare regolarmente su una macchina di sviluppo con chiavi.

<div id="modern-smoke-set-tool-calling-image">
  ### Set moderno di smoke test (tool calling + image)
</div>

Questa è l’esecuzione dei “modelli comuni” che ci aspettiamo continui a funzionare:

* OpenAI (non-Codex): `openai/gpt-5.2` (opzionale: `openai/gpt-5.1`)
* OpenAI Codex: `openai-codex/gpt-5.2` (opzionale: `openai-codex/gpt-5.2-codex`)
* Anthropic: `anthropic/claude-opus-4-5` (oppure `anthropic/claude-sonnet-4-5`)
* Google (Gemini API): `google/gemini-3-pro-preview` e `google/gemini-3-flash-preview` (evita i modelli Gemini 2.x più vecchi)
* Google (Antigravity): `google-antigravity/claude-opus-4-5-thinking` e `google-antigravity/gemini-3-flash`
* Z.AI (GLM): `zai/glm-4.7`
* MiniMax: `minimax/minimax-m2.1`

Esegui lo smoke test del Gateway con tools + image:
`OPENCLAW_LIVE_GATEWAY_MODELS="openai/gpt-5.2,openai-codex/gpt-5.2,anthropic/claude-opus-4-5,google/gemini-3-pro-preview,google/gemini-3-flash-preview,google-antigravity/claude-opus-4-5-thinking,google-antigravity/gemini-3-flash,zai/glm-4.7,minimax/minimax-m2.1" pnpm test:live src/gateway/gateway-models.profiles.live.test.ts`

<div id="baseline-tool-calling-read-optional-exec">
  ### Baseline: chiamata agli strumenti (Read + Exec opzionale)
</div>

Scegli almeno un modello per famiglia di provider:

* OpenAI: `openai/gpt-5.2` (o `openai/gpt-5-mini`)
* Anthropic: `anthropic/claude-opus-4-5` (o `anthropic/claude-sonnet-4-5`)
* Google: `google/gemini-3-flash-preview` (o `google/gemini-3-pro-preview`)
* Z.AI (GLM): `zai/glm-4.7`
* MiniMax: `minimax/minimax-m2.1`

Copertura aggiuntiva facoltativa (consigliata ma non obbligatoria):

* xAI: `xai/grok-4` (o il più recente disponibile)
* Mistral: `mistral/`… (scegli un modello con funzionalità di tool calling che hai abilitato)
* Cerebras: `cerebras/`… (se hai accesso)
* LM Studio: `lmstudio/`… (locale; la chiamata agli strumenti dipende dalla modalità API)

<div id="vision-image-send-attachment-multimodal-message">
  ### Vision: invio di immagini (allegato → messaggio multimodale)
</div>

Includi almeno un modello con capacità di elaborazione di immagini in `OPENCLAW_LIVE_GATEWAY_MODELS` (varianti di Claude/Gemini/OpenAI con supporto alla visione, ecc.) per testare il probe delle immagini.

<div id="aggregators-alternate-gateways">
  ### Aggregatori / gateway alternativi
</div>

Se hai le API key abilitate, supportiamo anche i test tramite:

* OpenRouter: `openrouter/...` (centinaia di modelli; usa `openclaw models scan` per trovare i candidati compatibili con tool+immagini)
* OpenCode Zen: `opencode/...` (autenticazione tramite `OPENCODE_API_KEY` / `OPENCODE_ZEN_API_KEY`)

Altri provider che puoi includere nella matrice live (se hai credenziali/configurazione):

* Integrati: `openai`, `openai-codex`, `anthropic`, `google`, `google-vertex`, `google-antigravity`, `google-gemini-cli`, `zai`, `openrouter`, `opencode`, `xai`, `groq`, `cerebras`, `mistral`, `github-copilot`
* Tramite `models.providers` (endpoint personalizzati): `minimax` (cloud/API), più qualsiasi proxy compatibile OpenAI/Anthropic (LM Studio, vLLM, LiteLLM, ecc.)

Suggerimento: non cercare di codificare in modo rigido “tutti i modelli” nella documentazione. L’elenco di riferimento è dato da ciò che `discoverModels(...)` restituisce sulla tua macchina, più le chiavi disponibili.

<div id="credentials-never-commit">
  ## Credenziali (non eseguire mai il commit)
</div>

I test live individuano le credenziali nello stesso modo in cui lo fa la CLI. Implicazioni pratiche:

* Se la CLI funziona, i test live dovrebbero trovare le stesse chiavi.

* Se un test live indica “no creds”, esegui il debug nello stesso modo in cui eseguiresti il debug di `openclaw models list` / selezione del modello.

* Profile store: `~/.openclaw/credentials/` (preferito; è ciò a cui si riferisce “profile keys” nei test)

* Config: `~/.openclaw/openclaw.json` (oppure `OPENCLAW_CONFIG_PATH`)

Se vuoi basarti su chiavi in variabili d’ambiente (ad es. esportate nel tuo `~/.profile`), esegui i test locali dopo `source ~/.profile`, oppure usa i runner Docker seguenti (possono montare `~/.profile` all’interno del container).

<div id="deepgram-live-audio-transcription">
  ## Deepgram live (trascrizione audio)
</div>

* Test: `src/media-understanding/providers/deepgram/audio.live.test.ts`
* Per abilitarlo: `DEEPGRAM_API_KEY=... DEEPGRAM_LIVE_TEST=1 pnpm test:live src/media-understanding/providers/deepgram/audio.live.test.ts`

<div id="docker-runners-optional-works-in-linux-checks">
  ## Runner Docker (facoltativi, controlli “funziona su Linux”)
</div>

Questi eseguono `pnpm test:live` all&#39;interno dell&#39;immagine Docker della repository, montando la tua directory di configurazione locale e lo spazio di lavoro (ed eseguendo il sourcing di `~/.profile` se montato):

* Modelli diretti: `pnpm test:docker:live-models` (script: `scripts/test-live-models-docker.sh`)
* Gateway + agente di sviluppo: `pnpm test:docker:live-gateway` (script: `scripts/test-live-gateway-models-docker.sh`)
* Onboarding wizard (TTY, scaffolding completo): `pnpm test:docker:onboard` (script: `scripts/e2e/onboard-docker.sh`)
* Networking del Gateway (due container, WS auth + health): `pnpm test:docker:gateway-network` (script: `scripts/e2e/gateway-network-docker.sh`)
* Plugin (caricamento estensioni personalizzate + smoke test del registry): `pnpm test:docker:plugins` (script: `scripts/e2e/plugins-docker.sh`)

Variabili d&#39;ambiente utili:

* `OPENCLAW_CONFIG_DIR=...` (valore predefinito: `~/.openclaw`) montata su `/home/node/.openclaw`
* `OPENCLAW_WORKSPACE_DIR=...` (valore predefinito: `~/.openclaw/workspace`) montata su `/home/node/.openclaw/workspace`
* `OPENCLAW_PROFILE_FILE=...` (valore predefinito: `~/.profile`) montata su `/home/node/.profile` ed eseguita (sourced) prima di lanciare i test
* `OPENCLAW_LIVE_GATEWAY_MODELS=...` / `OPENCLAW_LIVE_MODELS=...` per limitare l&#39;esecuzione
* `OPENCLAW_LIVE_REQUIRE_PROFILE_KEYS=1` per assicurarsi che le credenziali provengano dall&#39;archivio del profilo (non dalle variabili d&#39;ambiente)

<div id="docs-sanity">
  ## Verifica della documentazione
</div>

Esegui i controlli sulla documentazione dopo ogni modifica: `pnpm docs:list`.

<div id="offline-regression-ci-safe">
  ## Regressione offline (compatibile con CI)
</div>

Queste sono regressioni della pipeline reale senza provider reali:

* Chiamata degli strumenti del Gateway (OpenAI simulato, Gateway reale + loop dell’agente): `src/gateway/gateway.tool-calling.mock-openai.test.ts`
* Wizard del Gateway (WS `wizard.start`/`wizard.next`, scrive la configurazione con autenticazione applicata): `src/gateway/gateway.wizard.e2e.test.ts`

<div id="agent-reliability-evals-skills">
  ## Valutazioni di affidabilità degli agenti (abilità)
</div>

Abbiamo già alcuni test sicuri per la CI che si comportano come “valutazioni di affidabilità degli agenti”:

* Mock delle chiamate agli strumenti tramite il vero loop gateway + agente (`src/gateway/gateway.tool-calling.mock-openai.test.ts`).
* Flussi guidati (wizard) end-to-end che verificano il wiring delle sessioni e gli effetti della configurazione (`src/gateway/gateway.wizard.e2e.test.ts`).

Cosa manca ancora per le abilità (vedi [Skills](/it/tools/skills)):

* **Decisione:** quando le abilità sono elencate nel prompt, l’agente sceglie l’abilità corretta (o evita quelle irrilevanti)?
* **Conformità:** l’agente legge `SKILL.md` prima dell’uso e segue i passaggi/argomenti richiesti?
* **Contratti di workflow:** scenari multi-turno che verificano l’ordine degli strumenti, la persistenza della cronologia della sessione e i confini della sandbox.

In futuro, le valutazioni dovrebbero restare innanzitutto deterministiche:

* Un runner di scenari che usa provider simulati (mock) per verificare chiamate agli strumenti + ordine, letture dei file delle abilità e wiring delle sessioni.
* Una piccola suite di scenari focalizzati sulle abilità (uso vs evitamento, gating/controllo di accesso, prompt injection).
* Valutazioni live opzionali (opt-in, controllate tramite variabili d’ambiente) solo dopo che la suite sicura per la CI è in atto.

<div id="adding-regressions-guidance">
  ## Aggiungere regressioni (linee guida)
</div>

Quando risolvi un problema di provider/modello scoperto in live:

* Aggiungi, se possibile, un test di regressione sicuro per la CI (mock/stub del provider o cattura esatta della trasformazione della struttura della richiesta)
* Se è intrinsecamente verificabile solo in live (limiti di frequenza delle richieste, politiche di autenticazione), mantieni il test live ristretto e opzionale tramite variabili d&#39;ambiente
* Preferisci puntare al livello più ristretto che intercetta il bug:
  * bug di conversione/riproduzione della richiesta del provider → test diretto sui modelli
  * bug nella pipeline di sessione/cronologia/strumenti del Gateway → smoke test live del Gateway o test del Gateway con mock sicuro per la CI