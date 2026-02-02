---
title: Ciclo dell'agente
summary: "Ciclo di vita dell'agente, stream e semantiche di attesa"
read_when:
  - Hai bisogno di una procedura dettagliata del ciclo dell'agente o degli eventi del suo ciclo di vita
---

<div id="agent-loop-openclaw">
  # Agent Loop (OpenClaw)
</div>

Un loop agentico è l’esecuzione completa “reale” di un agente: acquisizione dell’input → assemblaggio del contesto → inferenza del modello → esecuzione dei tool → streaming delle risposte → persistenza. È il percorso canonico che trasforma un messaggio in azioni e in una risposta finale, mantenendo coerente lo stato della sessione.

In OpenClaw, un loop è una singola esecuzione serializzata per sessione che emette eventi di ciclo di vita e di streaming mentre il modello ragiona, chiama i tool e invia in streaming l’output. Questo documento spiega come quel loop effettivo è strutturato end-to-end.

<div id="entry-points">
  ## Punti di ingresso
</div>

* RPC del Gateway: `agent` e `agent.wait`.
* CLI: comando `agent`.

<div id="how-it-works-high-level">
  ## Come funziona (panoramica ad alto livello)
</div>

1. L&#39;RPC `agent` convalida i parametri, individua la sessione (sessionKey/sessionId), persiste i metadati della sessione, restituisce immediatamente `{ runId, acceptedAt }`.
2. `agentCommand` esegue l&#39;agente:
   * risolve il modello e i valori predefiniti per thinking/verbose
   * carica lo snapshot delle abilità
   * chiama `runEmbeddedPiAgent` (runtime pi-agent-core)
   * emette **lifecycle end/error** se il loop incorporato non ne emette uno
3. `runEmbeddedPiAgent`:
   * serializza le esecuzioni tramite code per sessione + globali
   * risolve modello + profilo di autenticazione e costruisce la sessione pi
   * si sottoscrive agli eventi pi e trasmette in streaming i delta di assistant/tool
   * applica un timeout -&gt; annulla l&#39;esecuzione se superato
   * restituisce payload + metadati di utilizzo
4. `subscribeEmbeddedPiSession` collega gli eventi di pi-agent-core allo stream `agent` di OpenClaw:
   * eventi tool =&gt; `stream: "tool"`
   * delta dell&#39;assistant =&gt; `stream: "assistant"`
   * eventi lifecycle =&gt; `stream: "lifecycle"` (`phase: "start" | "end" | "error"`)
5. `agent.wait` usa `waitForAgentJob`:
   * attende **lifecycle end/error** per `runId`
   * restituisce `{ status: ok|error|timeout, startedAt, endedAt, error? }`

<div id="queueing-concurrency">
  ## Accodamento e concorrenza
</div>

* Le esecuzioni sono serializzate in base alla chiave di sessione (corsia di sessione) e, facoltativamente, tramite una corsia globale.
* Questo evita race condition tra strumenti e sessioni e mantiene coerente la cronologia della sessione.
* I canali di messaggistica possono scegliere modalità di accodamento (collect/steer/followup) che si appoggiano a questo sistema di corsie.
  Vedi [Coda di comandi](/it/concepts/queue).

<div id="session-workspace-preparation">
  ## Preparazione di sessione e spazio di lavoro
</div>

* Lo spazio di lavoro viene individuato e creato; le esecuzioni in sandbox possono essere reindirizzate alla directory radice dello spazio di lavoro sandbox.
* Le abilità vengono caricate (o riutilizzate da uno snapshot) e iniettate nell&#39;env e nel prompt.
* I file di bootstrap/contesto vengono individuati e iniettati nel report del prompt di sistema.
* Viene acquisito un lock di scrittura sulla sessione; `SessionManager` viene aperto e preparato prima dello streaming.

<div id="prompt-assembly-system-prompt">
  ## Assemblaggio del prompt + prompt di sistema
</div>

* Il prompt di sistema è costruito a partire dal prompt base di OpenClaw, dal prompt delle abilità, dal contesto di bootstrap e dagli override specifici di ogni esecuzione.
* Vengono applicati i limiti specifici del modello e i token di riserva per la compattazione.
* Vedi [Prompt di sistema](/it/concepts/system-prompt) per ciò che il modello vede.

<div id="hook-points-where-you-can-intercept">
  ## Punti di hook (dove puoi intercettare)
</div>

OpenClaw dispone di due sistemi di hook:

* **Hook interni** (Gateway hooks): script orientati agli eventi per comandi ed eventi di ciclo di vita.
* **Hook dei plugin**: punti di estensione nel ciclo di vita di agenti/strumenti e nella pipeline del Gateway.

<div id="internal-hooks-gateway-hooks">
  ### Hook interni (hook del Gateway)
</div>

* **`agent:bootstrap`**: viene eseguito durante la creazione dei file di bootstrap, prima che il system prompt sia finalizzato.
  Usalo per aggiungere o rimuovere file di contesto di bootstrap.
* **Hook dei comandi**: `/new`, `/reset`, `/stop` e altri eventi di comando (vedi documentazione degli hook).

Vedi [Hooks](/it/hooks) per la configurazione e gli esempi.

<div id="plugin-hooks-agent-gateway-lifecycle">
  ### Hook dei plugin (ciclo di vita di agente e Gateway)
</div>

Questi hook vengono eseguiti all&#39;interno del loop dell&#39;agente o della pipeline del Gateway:

* **`before_agent_start`**: iniettare contesto o sovrascrivere il prompt di sistema prima che l&#39;esecuzione inizi.
* **`agent_end`**: ispezionare l&#39;elenco finale dei messaggi e i metadati di esecuzione dopo il completamento.
* **`before_compaction` / `after_compaction`**: osservare o annotare i cicli di compattazione.
* **`before_tool_call` / `after_tool_call`**: intercettare i parametri e i risultati degli strumenti.
* **`tool_result_persist`**: trasformare in modo sincrono i risultati degli strumenti prima che vengano scritti nella trascrizione della sessione.
* **`message_received` / `message_sending` / `message_sent`**: hook per messaggi in entrata e in uscita.
* **`session_start` / `session_end`**: delimitare il ciclo di vita della sessione.
* **`gateway_start` / `gateway_stop`**: eventi del ciclo di vita del Gateway.

Consulta [Plugin](/it/plugin#plugin-hooks) per i dettagli sull&#39;API degli hook e sulla registrazione.

<div id="streaming-partial-replies">
  ## Streaming + risposte parziali
</div>

* I delta dell&#39;assistente sono trasmessi in streaming da pi-agent-core ed emessi come eventi `assistant`.
* Lo streaming a blocchi può emettere risposte parziali sia su `text_end` che su `message_end`.
* Lo streaming del ragionamento può essere fornito come flusso separato o come risposte a blocchi.
* Consulta [Streaming](/it/concepts/streaming) per il comportamento di suddivisione in chunk e di risposta a blocchi.

<div id="tool-execution-messaging-tools">
  ## Esecuzione degli strumenti + strumenti di messaggistica
</div>

* Gli eventi di avvio/aggiornamento/termine degli strumenti sono emessi sullo stream `tool`.
* I risultati degli strumenti sono normalizzati per dimensione e payload di immagini prima del logging/emissione.
* Le operazioni di invio degli strumenti di messaggistica sono tracciate per evitare conferme duplicate dell&#39;assistente.

<div id="reply-shaping-suppression">
  ## Modellazione e soppressione delle risposte
</div>

* I payload finali sono assemblati a partire da:
  * testo dell&#39;assistente (ed eventuale ragionamento)
  * riepiloghi inline degli strumenti (quando in modalità verbose e consentiti)
  * testo di errore dell&#39;assistente quando il modello genera un errore
* `NO_REPLY` è trattato come un token silenzioso e viene filtrato dai payload in uscita.
* I duplicati degli strumenti di messaggistica sono rimossi dall&#39;elenco finale dei payload.
* Se non rimane alcun payload renderizzabile e uno strumento è andato in errore, viene emessa una risposta di fallback con l&#39;errore dello strumento
  (a meno che uno strumento di messaggistica non abbia già inviato una risposta visibile all&#39;utente).

<div id="compaction-retries">
  ## Compattazione + ritentativi
</div>

* La compattazione automatica emette eventi di stream `compaction` e può attivare un ritentativo.
* Al ritentativo, i buffer in memoria e i riepiloghi degli strumenti vengono azzerati per evitare output duplicati.
* Consulta [Compaction](/it/concepts/compaction) per la pipeline di compattazione.

<div id="event-streams-today">
  ## Flussi di eventi (oggi)
</div>

* `lifecycle`: generato da `subscribeEmbeddedPiSession` (e come fallback da `agentCommand`)
* `assistant`: delta in streaming da pi-agent-core
* `tool`: eventi dei tool in streaming da pi-agent-core

<div id="chat-channel-handling">
  ## Gestione del canale chat
</div>

* I delta dell&#39;assistente vengono accumulati in messaggi di chat `delta`.
* Un messaggio di chat `final` viene emesso alla **terminazione del ciclo di vita o in caso di errore**.

<div id="timeouts">
  ## Timeout
</div>

* `agent.wait` predefinito: 30s (solo l&#39;attesa). Puoi sovrascriverlo con il parametro `timeoutMs`.
* Runtime dell&#39;agente: `agents.defaults.timeoutSeconds` predefinito 600s; applicato nel timer di interruzione in `runEmbeddedPiAgent`.

<div id="where-things-can-end-early">
  ## Dove l&#39;esecuzione può terminare in anticipo
</div>

* timeout dell&#39;agente (interruzione)
* AbortSignal (annullamento)
* disconnessione del Gateway o timeout RPC
* timeout di `agent.wait` (solo attesa, non interrompe l&#39;agente)