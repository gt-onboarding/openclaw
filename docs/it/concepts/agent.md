---
title: Agente
summary: "Runtime dell'agente (pi-mono incorporato), contratto dello spazio di lavoro e bootstrap della sessione"
read_when:
  - Modifica del runtime dell'agente, del bootstrap dello spazio di lavoro o del comportamento della sessione
---

<div id="agent-runtime">
  # Runtime Agente 🤖
</div>

OpenClaw esegue un unico runtime di agente integrato derivato da **pi-mono**.

<div id="workspace-required">
  ## Spazio di lavoro (obbligatorio)
</div>

OpenClaw usa una singola directory di spazio di lavoro per l&#39;agente (`agents.defaults.workspace`) come **unica** directory di lavoro (`cwd`) dell&#39;agente per strumenti e contesto.

Consigliato: usa `openclaw setup` per creare `~/.openclaw/openclaw.json` se non esiste e inizializzare i file dello spazio di lavoro.

Struttura completa dello spazio di lavoro + guida al backup: [Spazio di lavoro dell&#39;agente](/it/concepts/agent-workspace)

Se `agents.defaults.sandbox` è abilitato, le sessioni non principali possono sovrascrivere questa impostazione con
spazi di lavoro per sessione sotto `agents.defaults.sandbox.workspaceRoot` (vedi
[Configurazione del Gateway](/it/gateway/configuration)).

<div id="bootstrap-files-injected">
  ## File di bootstrap (iniettati)
</div>

All&#39;interno di `agents.defaults.workspace`, OpenClaw richiede questi file modificabili dall&#39;utente:

* `AGENTS.md` — istruzioni operative + “memoria”
* `SOUL.md` — persona, limiti, tono
* `TOOLS.md` — note sugli strumenti mantenute dall&#39;utente (ad es. `imsg`, `sag`, convenzioni)
* `BOOTSTRAP.md` — rituale di primo avvio “one‑time” (eliminato dopo il completamento)
* `IDENTITY.md` — nome dell&#39;agente / “vibe” / emoji
* `USER.md` — profilo utente + modo di rivolgersi preferito

Al primo turno di una nuova sessione, OpenClaw inietta il contenuto di questi file direttamente nel contesto dell&#39;agente.

I file vuoti vengono ignorati. I file di grandi dimensioni vengono ridotti e troncati con un marcatore in modo che i prompt restino compatti (leggi il file per il contenuto completo).

Se un file manca, OpenClaw inietta una singola riga con il marcatore “missing file” (e `openclaw setup` creerà un modello predefinito sicuro).

`BOOTSTRAP.md` viene creato solo per **uno spazio di lavoro completamente nuovo** (nessun altro file di bootstrap presente). Se lo elimini dopo aver completato il rituale, non dovrebbe essere ricreato ai riavvii successivi.

Per disabilitare completamente la creazione dei file di bootstrap (per gli spazi di lavoro pre‑popolati), imposta:

```json5
{ agent: { skipBootstrap: true } }
```

<div id="built-in-tools">
  ## Strumenti integrati
</div>

Gli strumenti di base (`read`/`exec`/`edit`/`write` e i relativi strumenti di sistema) sono sempre disponibili,
nel rispetto della tool policy. `apply_patch` è facoltativo ed è subordinato a
`tools.exec.applyPatch`. `TOOLS.md` **non** controlla quali strumenti esistono; è una
guida su come *tu* vuoi che vengano utilizzati.

<div id="skills">
  ## Abilità
</div>

OpenClaw carica le abilità da tre percorsi (lo spazio di lavoro ha la precedenza in caso di conflitto di nomi):

* Incluse (fornite con l’installazione)
* Gestite/locali: `~/.openclaw/skills`
* Spazio di lavoro: `<workspace>/skills`

Le abilità possono essere abilitate o limitate tramite configurazione/variabili d’ambiente (vedi `skills` in [Gateway configuration](/it/gateway/configuration)).

<div id="pi-mono-integration">
  ## integrazione pi-mono
</div>

OpenClaw riutilizza parti della codebase di pi-mono (modelli/tool), ma **la gestione delle sessioni, la discovery e il wiring dei tool sono gestiti da OpenClaw**.

* Nessun runtime dell&#39;agente pi-coding.
* Nessuna impostazione `~/.pi/agent` o `<workspace>/.pi` viene consultata.

<div id="sessions">
  ## Sessioni
</div>

Le trascrizioni delle sessioni vengono memorizzate come JSONL in:

* `~/.openclaw/agents/<agentId>/sessions/<SessionId>.jsonl`

L&#39;ID della sessione è stabile ed è assegnato da OpenClaw.
Le cartelle delle sessioni legacy Pi/Tau **non** vengono lette.

<div id="steering-while-streaming">
  ## Steering durante lo streaming
</div>

Quando la modalità di coda è `steer`, i messaggi in ingresso vengono iniettati
nell&#39;esecuzione corrente.
La coda viene controllata **dopo ogni chiamata a uno strumento**; se è presente
un messaggio in coda, le restanti chiamate a strumenti del messaggio
dell&#39;assistente corrente vengono saltate (risultati di errore dello strumento con &quot;Skipped due to queued user message.&quot;), quindi il messaggio
dell&#39;utente in coda viene iniettato prima della risposta successiva
dell&#39;assistente.

Quando la modalità di coda è `followup` o `collect`, i messaggi in ingresso
vengono trattenuti fino alla fine del turno corrente, quindi un nuovo turno
dell&#39;agente inizia con i payload in coda. Vedi [Queue](/it/concepts/queue) per
dettagli su modalità + comportamento di debounce/cap.

Lo streaming a blocchi invia i blocchi completati dell&#39;assistente non appena
sono pronti; è **disattivato per impostazione predefinita**
(`agents.defaults.blockStreamingDefault: "off"`).
Regola il boundary tramite `agents.defaults.blockStreamingBreak` (`text_end` vs `message_end`; il valore predefinito è text&#95;end).
Controlla il chunking dei blocchi soft con `agents.defaults.blockStreamingChunk`
(predefinito 800–1200 caratteri; privilegia le interruzioni di paragrafo, poi
le nuove righe, e per ultime le frasi).
Aggrega i chunk in streaming con `agents.defaults.blockStreamingCoalesce` per
ridurre lo spam su singola riga (unione basata sull&#39;inattività prima dell&#39;invio).
I canali non-Telegram richiedono `*.blockStreaming: true` esplicito per
abilitare le risposte a blocchi.
I riepiloghi dettagliati degli strumenti vengono emessi all&#39;avvio dello
strumento (nessun debounce); la Control UI esegue lo streaming dell&#39;output
degli strumenti tramite eventi dell&#39;agente quando disponibile.
Ulteriori dettagli: [Streaming + chunking](/it/concepts/streaming).

<div id="model-refs">
  ## Riferimenti ai modelli
</div>

I riferimenti ai modelli nella configurazione (per esempio `agents.defaults.model` e `agents.defaults.models`) vengono analizzati suddividendo in corrispondenza della **prima** `/`.

* Usa `provider/model` quando configuri i modelli.
* Se l&#39;ID del modello stesso contiene `/` (stile OpenRouter), includi il prefisso del provider (esempio: `openrouter/moonshotai/kimi-k2`).
* Se ometti il provider, OpenClaw tratta l&#39;input come un alias o un modello per il **provider predefinito** (funziona solo quando non c&#39;è `/` nell&#39;ID del modello).

<div id="configuration-minimal">
  ## Configurazione minima
</div>

Configura almeno:

* `agents.defaults.workspace`
* `channels.whatsapp.allowFrom` (fortemente consigliato)

***

*Successivo: [Chat di gruppo](/it/concepts/group-messages)* 🦞