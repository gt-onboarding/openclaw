---
title: Modelli
summary: "CLI modelli: elenco, impostazione, alias, fallback, scansione, stato"
read_when:
  - Aggiunta o modifica della CLI dei modelli (models list/set/scan/aliases/fallbacks)
  - Modifica del comportamento di fallback dei modelli o della UX di selezione
  - Aggiornamento delle sonde di scansione dei modelli (tools/images)
---

<div id="models-cli">
  # CLI dei modelli
</div>

Consulta [/concepts/model-failover](/it/concepts/model-failover) per la rotazione dei profili di autenticazione, i periodi di cooldown e il modo in cui questi interagiscono con i fallback.
Panoramica rapida dei provider + esempi: [/concepts/model-providers](/it/concepts/model-providers).

<div id="how-model-selection-works">
  ## Come funziona la selezione dei modelli
</div>

OpenClaw seleziona i modelli in questo ordine:

1. Modello **primario** (`agents.defaults.model.primary` o `agents.defaults.model`).
2. **Modelli di fallback** in `agents.defaults.model.fallbacks` (nell’ordine indicato).
3. Il **failover di autenticazione del provider** avviene all’interno di un provider prima di passare al
   modello successivo.

Correlati:

* `agents.defaults.models` è la lista di autorizzati/catalogo dei modelli che OpenClaw può usare (più alias).
* `agents.defaults.imageModel` viene usato **solo quando** il modello primario non può accettare immagini.
* Per singolo agente, i valori predefiniti possono sovrascrivere `agents.defaults.model` tramite `agents.list[].model` più binding (vedi [/concepts/multi-agent](/it/concepts/multi-agent)).

<div id="quick-model-picks-anecdotal">
  ## Scelte rapide di modelli (impressioni aneddotiche)
</div>

* **GLM**: un po&#39; migliore per la programmazione e le chiamate agli strumenti.
* **MiniMax**: migliore per la scrittura e il tono/feeling.

<div id="setup-wizard-recommended">
  ## Procedura guidata di configurazione (consigliata)
</div>

Se non vuoi modificare la configurazione manualmente, esegui la procedura guidata di onboarding:

```bash
openclaw onboard
```

Può configurare modello e autenticazione per i provider più comuni, inclusi l&#39;**abbonamento OpenAI Code (Codex)**
(OAuth) e **Anthropic** (è consigliato l&#39;uso della chiave API; è supportato anche `claude
setup-token`).

<div id="config-keys-overview">
  ## Chiavi di configurazione (panoramica)
</div>

* `agents.defaults.model.primary` e `agents.defaults.model.fallbacks`
* `agents.defaults.imageModel.primary` e `agents.defaults.imageModel.fallbacks`
* `agents.defaults.models` (lista di autorizzati + alias + parametri del provider)
* `models.providers` (provider personalizzati definiti in `models.json`)

I riferimenti ai modelli vengono normalizzati in minuscolo. Gli alias dei provider come `z.ai/*` vengono normalizzati in
`zai/*`.

Esempi di configurazione dei provider (incluso OpenCode Zen) sono disponibili in
[/gateway/configuration](/it/gateway/configuration#opencode-zen-multi-model-proxy).

<div id="model-is-not-allowed-and-why-replies-stop">
  ## “Il modello non è consentito” (e perché le risposte si interrompono)
</div>

Se `agents.defaults.models` è impostato, diventa la **lista di autorizzati** per `/model` e per
gli override di sessione. Quando un utente seleziona un modello che non è presente in quella lista di autorizzati,
OpenClaw restituisce:

```
Il modello "provider/model" non è consentito. Usa /model per elencare i modelli disponibili.
```

Questo accade **prima** che venga generata una normale risposta, quindi il messaggio può dare l’impressione
che “non abbia risposto”. Per risolvere, puoi:

* Aggiungere il modello a `agents.defaults.models`, oppure
* Svuotare la lista di autorizzati (rimuovere `agents.defaults.models`), oppure
* Selezionare un modello da `/model list`.

Esempio di configurazione della lista di autorizzati:

```json5
{
  agent: {
    model: { primary: "anthropic/claude-sonnet-4-5" },
    models: {
      "anthropic/claude-sonnet-4-5": { alias: "Sonnet" },
      "anthropic/claude-opus-4-5": { alias: "Opus" }
    }
  }
}
```

<div id="switching-models-in-chat-model">
  ## Cambiare modello in chat (`/model`)
</div>

Puoi cambiare il modello della sessione corrente senza riavviarla:

```
/model
/model list
/model 3
/model openai/gpt-5.2
/model status
```

Note:

* `/model` (e `/model list`) è un selettore compatto, numerato (famiglia di modelli + provider disponibili).
* `/model <#>` seleziona da quel selettore.
* `/model status` è la vista dettagliata (candidati di autenticazione e, quando configurato, `baseUrl` dell&#39;endpoint del provider + modalità `api`).
* I riferimenti ai modelli vengono interpretati suddividendo sulla **prima** `/`. Usa `provider/model` quando digiti `/model <ref>`.
* Se l&#39;ID del modello contiene a sua volta `/` (stile OpenRouter), devi includere il prefisso del provider (esempio: `/model openrouter/moonshotai/kimi-k2`).
* Se ometti il provider, OpenClaw tratta l&#39;input come un alias o un modello per il **provider predefinito** (funziona solo quando non c&#39;è `/` nell&#39;ID del modello).

Comportamento/configurazione completa del comando: [Slash commands](/it/tools/slash-commands).

<div id="cli-commands">
  ## Comandi CLI
</div>

```bash
openclaw models list
openclaw models status
openclaw models set <provider/model>
openclaw models set-image <provider/model>

openclaw models aliases list
openclaw models aliases add <alias> <provider/model>
openclaw models aliases remove <alias>

openclaw models fallbacks list
openclaw models fallbacks add <provider/model>
openclaw models fallbacks remove <provider/model>
openclaw models fallbacks clear

openclaw models image-fallbacks list
openclaw models image-fallbacks add <provider/model>
openclaw models image-fallbacks remove <provider/model>
openclaw models image-fallbacks clear
```

`openclaw models` (senza sottocomando) è un alias di `models status`.

<div id="models-list">
  ### `models list`
</div>

Mostra per impostazione predefinita i modelli configurati. Opzioni utili:

* `--all`: catalogo completo
* `--local`: solo provider locali
* `--provider <name>`: filtra per provider
* `--plain`: un modello per riga
* `--json`: output in formato JSON, adatto all&#39;elaborazione automatica

<div id="models-status">
  ### `models status`
</div>

Mostra il modello primario risolto, i fallback, il modello per le immagini e una panoramica dell&#39;autenticazione
dei provider configurati. Inoltre mostra lo stato di scadenza OAuth per i profili trovati
nell&#39;auth store (avvisa entro 24 ore per impostazione predefinita). `--plain` stampa solo il
modello primario risolto.
Lo stato OAuth è sempre mostrato (e incluso nell&#39;output `--json`). Se un provider configurato
non ha credenziali, `models status` stampa una sezione **Missing auth**.
Il JSON include `auth.oauth` (finestra di avviso + profili) e `auth.providers`
(autenticazione effettiva per provider).
Usa `--check` per l&#39;automazione (termina con codice di uscita `1` quando mancano/sono scadute, `2` quando sono in scadenza).

L&#39;autenticazione Anthropic preferita è il token di configurazione Claude Code CLI (esegui ovunque; incolla sull&#39;host del Gateway se necessario):

```bash
claude setup-token
openclaw models status
```

<div id="scanning-openrouter-free-models">
  ## Scansione (modelli gratuiti OpenRouter)
</div>

`openclaw models scan` ispeziona il **catalogo dei modelli gratuiti** di OpenRouter e può
facoltativamente verificare se i modelli supportano tool e immagini.

Flag principali:

* `--no-probe`: salta i probe live (solo metadati)
* `--min-params <b>`: dimensione minima dei parametri (in miliardi)
* `--max-age-days <days>`: salta i modelli più vecchi
* `--provider <name>`: filtro sul prefisso del provider
* `--max-candidates <n>`: dimensione della lista di fallback
* `--set-default`: imposta `agents.defaults.model.primary` sulla prima selezione
* `--set-image`: imposta `agents.defaults.imageModel.primary` sulla prima selezione di immagine

Il probing richiede una API key OpenRouter (da profili di autenticazione o
`OPENROUTER_API_KEY`). Senza una chiave, usa `--no-probe` per elencare solo i candidati.

I risultati della scansione sono ordinati in base a:

1. Supporto per le immagini
2. Latenza dei tool
3. Dimensione del contesto
4. Numero di parametri

Input

* Lista `/models` di OpenRouter (filtro `:free`)
* Richiede una API key OpenRouter da profili di autenticazione o `OPENROUTER_API_KEY` (vedi [/environment](/it/environment))
* Filtri opzionali: `--max-age-days`, `--min-params`, `--provider`, `--max-candidates`
* Controlli di probing: `--timeout`, `--concurrency`

Quando viene eseguito in un TTY, puoi selezionare i fallback in modo interattivo. In modalità
non interattiva, passa `--yes` per accettare i valori predefiniti.

<div id="models-registry-modelsjson">
  ## Registro dei modelli (`models.json`)
</div>

I provider personalizzati definiti in `models.providers` vengono scritti nel file `models.json` all&#39;interno della directory dell&#39;agente (per impostazione predefinita `~/.openclaw/agents/<agentId>/models.json`). Per impostazione predefinita questo file viene unito, a meno che `models.mode` non sia impostato su `replace`.