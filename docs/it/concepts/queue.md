---
title: Coda
summary: "Progettazione della coda di comandi che serializza le esecuzioni in entrata delle risposte automatiche"
read_when:
  - Modifica della modalità di esecuzione o della concorrenza delle risposte automatiche
---

<div id="command-queue-2026-01-16">
  # Coda dei comandi (2026-01-16)
</div>

Serializziamo le esecuzioni di risposte automatiche in arrivo (su tutti i canali) tramite una piccola coda interna al processo, per evitare che più esecuzioni di agenti si sovrappongano, pur consentendo un parallelismo sicuro tra sessioni.

<div id="why">
  ## Perché
</div>

- Le esecuzioni di risposta automatica possono essere costose (chiamate LLM) e possono entrare in conflitto quando più messaggi in ingresso arrivano ravvicinati nel tempo.
- La serializzazione evita la concorrenza su risorse condivise (file di sessione, log, stdin della CLI) e riduce la probabilità di incorrere in limiti di rate a monte.

<div id="how-it-works">
  ## Come funziona
</div>

- Una coda FIFO "lane-aware" svuota ciascuna lane con un limite di concorrenza configurabile (predefinito 1 per le lane non configurate; `main` predefinita a 4, `subagent` a 8).
- `runEmbeddedPiAgent` mette in coda per **session key** (lane `session:&lt;key&gt;`) per garantire una sola esecuzione attiva per sessione.
- Ogni esecuzione di sessione viene quindi messa in coda in una **lane globale** (`main` per impostazione predefinita) in modo che il parallelismo complessivo sia limitato da `agents.defaults.maxConcurrent`.
- Quando il logging dettagliato è abilitato, le esecuzioni in coda emettono un breve avviso se hanno atteso più di ~2s prima di iniziare.
- Gli indicatori di digitazione vengono comunque attivati immediatamente al momento della messa in coda (quando supportati dal canale), quindi l'esperienza utente rimane invariata mentre l'esecuzione attende il proprio turno.

<div id="queue-modes-per-channel">
  ## Modalità di coda (per canale)
</div>

I messaggi in ingresso possono pilotare l&#39;esecuzione corrente, attendere un turno successivo o fare entrambe le cose:

* `steer`: viene iniettato immediatamente nell&#39;esecuzione corrente (annulla le chiamate agli strumenti in sospeso dopo il successivo boundary del tool). Se non c&#39;è streaming, passa al followup.
* `followup`: messo in coda per il prossimo turno dell&#39;agente dopo la fine dell&#39;esecuzione corrente.
* `collect`: raggruppa tutti i messaggi in coda in un **singolo** turno di followup (predefinito). Se i messaggi sono destinati a canali/thread diversi, vengono svuotati singolarmente per preservare il routing.
* `steer-backlog` (alias `steer+backlog`): pilota ora **e** preserva il messaggio per un turno di followup.
* `interrupt` (legacy): interrompe l&#39;esecuzione attiva per quella sessione, quindi esegue il messaggio più recente.
* `queue` (alias legacy): equivalente a `steer`.

Steer-backlog significa che puoi ottenere una risposta di followup dopo l&#39;esecuzione pilotata, quindi
le superfici di streaming possono sembrare dei duplicati. Preferisci `collect`/`steer` se vuoi
una risposta per ogni messaggio in ingresso.
Invia `/queue collect` come comando autonomo (per sessione) oppure imposta `messages.queue.byChannel.discord: "collect"`.

Valori predefiniti (quando non impostati nella configurazione):

* Tutte le superfici → `collect`

Configura globalmente o per canale tramite `messages.queue`:

```json5
{
  messages: {
    queue: {
      mode: "collect",
      debounceMs: 1000,
      cap: 20,
      drop: "summarize",
      byChannel: { discord: "collect" }
    }
  }
}
```


<div id="queue-options">
  ## Opzioni della coda
</div>

Le opzioni si applicano a `followup`, `collect` e `steer-backlog` (e a `steer` quando effettua fallback su `followup`):

- `debounceMs`: attende un intervallo di silenzio prima di iniziare un turno di followup (evita “continua, continua”).
- `cap`: numero massimo di messaggi in coda per sessione.
- `drop`: criterio di gestione dell’overflow (`old`, `new`, `summarize`).

`summarize` mantiene un breve elenco puntato dei messaggi scartati e lo inserisce come prompt di followup sintetico.
Valori predefiniti: `debounceMs: 1000`, `cap: 20`, `drop: summarize`.

<div id="per-session-overrides">
  ## Override per sessione
</div>

- Invia `/queue <mode>` come comando a sé stante per salvare la modalità per la sessione corrente.
- Le opzioni possono essere combinate: `/queue collect debounce:2s cap:25 drop:summarize`
- `/queue default` o `/queue reset` cancella l'override della sessione.

<div id="scope-and-guarantees">
  ## Ambito e garanzie
</div>

- Si applica alle esecuzioni dell’agente di risposta automatica su tutti i canali in ingresso che usano la pipeline di risposta del Gateway (WhatsApp web, Telegram, Slack, Discord, Signal, iMessage, webchat, ecc.).
- La corsia predefinita (`main`) è a livello di processo per il traffico in ingresso e per gli heartbeat principali; imposta `agents.defaults.maxConcurrent` per consentire più sessioni in parallelo.
- Possono esistere corsie aggiuntive (ad es. `cron`, `subagent`) in modo che i job in background possano essere eseguiti in parallelo senza bloccare le risposte in ingresso.
- Le corsie per sessione garantiscono che solo un’esecuzione di un agente alla volta acceda a una determinata sessione.
- Nessuna dipendenza esterna o thread worker in background; solo TypeScript + Promise.

<div id="troubleshooting">
  ## Risoluzione dei problemi
</div>

- Se i comandi sembrano bloccati, abilita i log verbosi e cerca le righe “queued for …ms” per verificare che la coda si stia effettivamente svuotando.
- Se ti serve conoscere la profondità della coda, abilita i log verbosi e osserva le righe relative ai tempi della coda.