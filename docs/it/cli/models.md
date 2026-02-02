---
title: Modelli
summary: "Riferimento CLI per `openclaw models` (status/list/set/scan, alias, fallback, autenticazione)"
read_when:
  - Vuoi modificare i modelli predefiniti o verificare lo stato di autenticazione dei provider
  - Vuoi eseguire la scansione dei modelli/provider disponibili ed eseguire il debug dei profili di autenticazione
---

<div id="openclaw-models">
  # `openclaw models`
</div>

Rilevamento, scansione e configurazione dei modelli (modello predefinito, fallback, profili di autenticazione).

Contenuti correlati:

* Provider e modelli: [Modelli](/it/providers/models)
* Configurazione dell&#39;autenticazione dei provider: [Guida introduttiva](/it/start/getting-started)

<div id="common-commands">
  ## Comandi principali
</div>

```bash
openclaw models status
openclaw models list
openclaw models set <model-or-alias>
openclaw models scan
```

`openclaw models status` mostra i valori predefiniti/di fallback risolti più una panoramica dell&#39;autenticazione.
Quando sono disponibili istantanee di utilizzo dei provider, la sezione sullo stato OAuth/token include
le intestazioni di utilizzo dei provider.
Aggiungi `--probe` per eseguire sonde di autenticazione in tempo reale su ciascun profilo di provider configurato.
Le sonde effettuano richieste reali (possono consumare token e attivare limiti di frequenza).
Usa `--agent <id>` per ispezionare lo stato modello/auth di un agente configurato. Se omesso,
il comando usa `OPENCLAW_AGENT_DIR`/`PI_CODING_AGENT_DIR` se impostate, altrimenti l&#39;agente predefinito configurato.

Note:

* `models set <model-or-alias>` accetta `provider/model` o un alias.
* I riferimenti ai modelli sono analizzati dividendo sulla **prima** `/`. Se l&#39;ID del modello include `/` (stile OpenRouter), includi il prefisso del provider (esempio: `openrouter/moonshotai/kimi-k2`).
* Se ometti il provider, OpenClaw tratta l&#39;input come un alias o un modello per il **provider predefinito** (funziona solo quando non c&#39;è `/` nell&#39;ID del modello).

<div id="models-status">
  ### `models status`
</div>

Opzioni:

* `--json`
* `--plain`
* `--check` (exit 1=scaduto/mancante, 2=in scadenza)
* `--probe` (sonda live dei profili di autenticazione configurati)
* `--probe-provider <name>` (sonda un singolo provider)
* `--probe-profile <id>` (ripeti o elenco di id profilo separati da virgola)
* `--probe-timeout <ms>`
* `--probe-concurrency <n>`
* `--probe-max-tokens <n>`
* `--agent <id>` (id agente configurato; ha precedenza su `OPENCLAW_AGENT_DIR`/`PI_CODING_AGENT_DIR`)

<div id="aliases-fallbacks">
  ## Alias e fallback
</div>

```bash
openclaw models aliases list
openclaw models fallbacks list
```

<div id="auth-profiles">
  ## Profili di autenticazione
</div>

```bash
openclaw models auth add
openclaw models auth login --provider <id>
openclaw models auth setup-token
openclaw models auth paste-token
```

`models auth login` esegue il flusso di autenticazione (OAuth/API key) di un plugin di un provider. Usa
`openclaw plugins list` per vedere quali provider sono installati.

Note:

* `setup-token` chiede di inserire un valore di setup-token (generalo con `claude setup-token` su qualsiasi macchina).
* `paste-token` accetta una stringa di token generata altrove o tramite automazione.
