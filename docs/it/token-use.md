---
title: Utilizzo dei token
summary: "Come OpenClaw costruisce il contesto dei prompt e segnala l'utilizzo dei token e i costi"
read_when:
  - Spiegare l'utilizzo dei token, i costi o le finestre di contesto (context window)
  - Eseguire il debug della crescita o della compattazione del contesto
---

<div id="token-use-costs">
  # Uso di token e costi
</div>

OpenClaw conteggia i **token**, non i caratteri. I token sono specifici del modello, ma la maggior parte dei modelli in stile OpenAI usa in media circa 4 caratteri per token per il testo in inglese.

<div id="how-the-system-prompt-is-built">
  ## Come viene costruito il system prompt
</div>

OpenClaw assembla il proprio system prompt a ogni esecuzione. Include:

* Elenco degli strumenti + brevi descrizioni
* Elenco delle abilità (solo metadati; le istruzioni vengono caricate on demand con `read`)
* Istruzioni di auto-aggiornamento
* Spazio di lavoro + file di bootstrap (`AGENTS.md`, `SOUL.md`, `TOOLS.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md`, `BOOTSTRAP.md` se presenti). I file di grandi dimensioni vengono troncati da `agents.defaults.bootstrapMaxChars` (valore predefinito: 20000).
* Ora (UTC + fuso orario dell&#39;utente)
* Tag di risposta + comportamento dell&#39;heartbeat
* Metadati di runtime (host/OS/modello/thinking)

Per una descrizione completa, vedi [System Prompt](/it/concepts/system-prompt).

<div id="what-counts-in-the-context-window">
  ## Cosa rientra nella finestra di contesto
</div>

Tutto ciò che il modello riceve concorre al limite di contesto:

* Prompt di sistema (tutte le sezioni elencate sopra)
* Cronologia della conversazione (messaggi utente + assistente)
* Chiamate agli strumenti e risultati degli strumenti
* Allegati/trascrizioni (immagini, audio, file)
* Riepiloghi di compattazione e artefatti di pruning
* Wrapper del provider o intestazioni di sicurezza (non visibili, ma comunque conteggiati)

Per una ripartizione pratica (per ogni file iniettato, strumenti, abilità e dimensione del prompt di sistema), usa `/context list` o `/context detail`. Vedi [Contesto](/it/concepts/context).

<div id="how-to-see-current-token-usage">
  ## Come vedere l&#39;utilizzo corrente dei token
</div>

Usa questi comandi in chat:

* `/status` → **scheda di stato ricca di emoji** con il modello della sessione, l&#39;utilizzo del contesto,
  gli ultimi token di input/output dell&#39;ultima risposta e il **costo stimato** (solo chiave API).
* `/usage off|tokens|full` → aggiunge un **footer di utilizzo per risposta** a ogni risposta.
  * Persiste a livello di sessione (archiviato come `responseUsage`).
  * L&#39;autenticazione OAuth **nasconde il costo** (solo token).
* `/usage cost` → mostra un riepilogo locale dei costi dai log delle sessioni OpenClaw.

Altre interfacce:

* **TUI/Web TUI:** `/status` e `/usage` sono supportati.
* **CLI:** `openclaw status --usage` e `openclaw channels list` mostrano
  le finestre di quota del provider (non i costi per risposta).

<div id="cost-estimation-when-shown">
  ## Stima dei costi (quando disponibile)
</div>

I costi vengono stimati in base alla configurazione dei prezzi del modello:

```
models.providers.<provider>.models[].cost
```

Questi sono **USD per 1M token** per `input`, `output`, `cacheRead` e
`cacheWrite`. Se il prezzo non è indicato, OpenClaw mostra solo i token. I token OAuth
non riportano mai un costo in dollari.

<div id="cache-ttl-and-pruning-impact">
  ## Impatto del TTL della cache e del pruning
</div>

La cache dei prompt del provider si applica solo all&#39;interno della finestra di TTL
della cache. OpenClaw può facoltativamente eseguire il **pruning cache-ttl**:
esegue il pruning della sessione una volta che il TTL della cache è scaduto, poi
reimposta la finestra della cache in modo che le richieste successive possano
riutilizzare il contesto appena memorizzato in cache invece di mettere di nuovo
in cache l’intera cronologia. Questo mantiene più bassi i costi di scrittura in
cache quando una sessione rimane inattiva oltre il TTL.

Configura questa opzione nella [Gateway configuration](/it/gateway/configuration) e consulta i
dettagli del comportamento in [Session pruning](/it/concepts/session-pruning).

Heartbeat può mantenere la cache **calda** durante i periodi di inattività.
Se il TTL della cache del modello è `1h`, impostare l’intervallo di heartbeat
poco al di sotto di questo valore (ad esempio, `55m`) può evitare di mettere di
nuovo in cache l’intero prompt, riducendo i costi di scrittura in cache.

Per il pricing dell’api Anthropic, le letture dalla cache sono
significativamente più economiche dei token di input, mentre le scritture in
cache sono fatturate con un moltiplicatore più alto. Consulta il pricing della
prompt caching di Anthropic per le tariffe e i moltiplicatori di TTL più
recenti:
https://docs.anthropic.com/docs/build-with-claude/prompt-caching

<div id="example-keep-1h-cache-warm-with-heartbeat">
  ### Esempio: mantieni la cache calda per 1 ora con heartbeat
</div>

```yaml
agents:
  defaults:
    model:
      primary: "anthropic/claude-opus-4-5"
    models:
      "anthropic/claude-opus-4-5":
        params:
          cacheControlTtl: "1h"
    heartbeat:
      every: "55m"
```

<div id="tips-for-reducing-token-pressure">
  ## Suggerimenti per ridurre la pressione sui token
</div>

* Usa `/compact` per riassumere le sessioni lunghe.
* Riduci gli output molto grandi dei tool nei tuoi workflow.
* Mantieni brevi le descrizioni delle abilità (l&#39;elenco delle abilità viene iniettato nel prompt).
* Preferisci modelli più piccoli per attività esplorative e verbose.

Consulta [Abilità](/it/tools/skills) per la formula esatta del sovraccarico dovuto all&#39;elenco delle abilità.