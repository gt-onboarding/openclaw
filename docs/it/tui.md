---
title: TUI
summary: "Interfaccia utente da terminale (TUI): collegati al Gateway da qualsiasi macchina"
read_when:
  - Vuoi una guida per principianti alla TUI
  - Hai bisogno dell'elenco completo delle funzionalità, dei comandi e delle scorciatoie da tastiera della TUI
---

<div id="tui-terminal-ui">
  # TUI (interfaccia utente da terminale)
</div>

<div id="quick-start">
  ## Guida rapida
</div>

1. Avvia il Gateway.

```bash
openclaw gateway
```

2. Apri l&#39;interfaccia TUI.

```bash
openclaw tui
```

3. Digita un messaggio e premi Invio.

Gateway remoto:

```bash
openclaw tui --url ws://<host>:<port> --token <gateway-token>
```

Utilizza `--password` se il tuo Gateway usa l&#39;autenticazione con password.

<div id="what-you-see">
  ## Cosa vedi
</div>

* Intestazione: URL di connessione, agente corrente, sessione corrente.
* Registro della chat: messaggi utente, risposte dell&#39;assistente, avvisi di sistema, schede degli strumenti.
* Riga di stato: stato di connessione/esecuzione (connessione in corso, in esecuzione, in streaming, inattivo, errore).
* Piè di pagina: stato della connessione + agente + sessione + modello + think/verbose/reasoning + conteggi token + invio.
* Input: editor di testo con completamento automatico.

<div id="mental-model-agents-sessions">
  ## Modello mentale: agenti + sessioni
</div>

* Gli agenti sono slug univoci (ad es. `main`, `research`). Il Gateway ne espone l&#39;elenco.
* Le sessioni appartengono all&#39;agente corrente.
* Le chiavi di sessione sono memorizzate come `agent:<agentId>:<sessionKey>`.
  * Se digiti `/session main`, la TUI lo espande in `agent:<currentAgent>:main`.
  * Se digiti `/session agent:other:main`, passi esplicitamente alla sessione di quell&#39;agente.
* Scope della sessione:
  * `per-sender` (predefinito): ogni agente ha molte sessioni.
  * `global`: la TUI usa sempre la sessione `global` (il selettore può essere vuoto).
* L&#39;agente e la sessione correnti sono sempre visibili nel footer.

<div id="sending-delivery">
  ## Invio + recapito
</div>

* I messaggi vengono inviati al Gateway; il recapito ai provider è disattivato per impostazione predefinita.
* Per attivare il recapito:
  * `/deliver on`
  * oppure dal pannello Impostazioni
  * oppure avvia con `openclaw tui --deliver`

<div id="pickers-overlays">
  ## Selettori + overlay
</div>

* Selettore di modelli: elenca i modelli disponibili e imposta l&#39;override della sessione.
* Selettore di agenti: scegli un altro agente.
* Selettore di sessioni: mostra solo le sessioni dell&#39;agente corrente.
* Impostazioni: attiva/disattiva deliver, l&#39;espansione dell&#39;output degli strumenti e la visibilità del thinking.

<div id="keyboard-shortcuts">
  ## Scorciatoie da tastiera
</div>

* Enter: Invia messaggio
* Esc: interrompi l&#39;esecuzione corrente
* Ctrl+C: cancella l&#39;input (premi due volte per uscire)
* Ctrl+D: esci
* Ctrl+L: selettore modello
* Ctrl+G: selettore agente
* Ctrl+P: selettore sessione
* Ctrl+O: attiva/disattiva espansione output strumenti
* Ctrl+T: attiva/disattiva visibilità del ragionamento (ricarica la cronologia)

<div id="slash-commands">
  ## Comandi slash
</div>

Principali:

* `/help`
* `/status`
* `/agent <id>` (oppure `/agents`)
* `/session <key>` (oppure `/sessions`)
* `/model <provider/model>` (oppure `/models`)

Controlli della sessione:

* `/think <off|minimal|low|medium|high>`
* `/verbose <on|full|off>`
* `/reasoning <on|off|stream>`
* `/usage <off|tokens|full>`
* `/elevated <on|off|ask|full>` (alias: `/elev`)
* `/activation <mention|always>`
* `/deliver <on|off>`

Ciclo di vita della sessione:

* `/new` oppure `/reset` (reimposta la sessione)
* `/abort` (interrompe il run attivo)
* `/settings`
* `/exit`

Altri comandi slash del Gateway (ad esempio, `/context`) vengono inoltrati al Gateway e mostrati come output di sistema. Consulta [Comandi slash](/it/tools/slash-commands).

<div id="local-shell-commands">
  ## Comandi shell locali
</div>

* Anteponi `!` a una riga per eseguire un comando shell locale sull&#39;host della TUI.
* La TUI ti chiede, una sola volta per sessione, se consentire l&#39;esecuzione locale; se rifiuti, `!` rimane disabilitato per la sessione.
* I comandi vengono eseguiti in una nuova shell non interattiva nella directory di lavoro della TUI (nessun `cd`/env persistente).
* Un `!` da solo viene inviato come messaggio normale; gli spazi iniziali non attivano l&#39;esecuzione locale.

<div id="tool-output">
  ## Output degli strumenti
</div>

* Le chiamate agli strumenti vengono visualizzate come schede con argomenti e risultati.
* Ctrl+O passa dalla vista compressa a quella espansa e viceversa.
* Durante l&#39;esecuzione degli strumenti, gli aggiornamenti parziali vengono trasmessi in streaming nella stessa scheda.

<div id="history-streaming">
  ## Cronologia + streaming
</div>

* Al momento della connessione, la TUI carica la cronologia più recente (per impostazione predefinita, 200 messaggi).
* Le risposte in streaming vengono aggiornate in tempo reale finché non vengono finalizzate.
* La TUI ascolta anche gli eventi degli strumenti dell&#39;agente per mostrare schede degli strumenti più dettagliate.

<div id="connection-details">
  ## Dettagli di connessione
</div>

* La TUI si registra al Gateway come `mode: "tui"`.
* Le riconnessioni mostrano un messaggio di sistema; le interruzioni negli eventi vengono riportate nel log.

<div id="options">
  ## Opzioni
</div>

* `--url <url>`: URL WS del Gateway (per impostazione predefinita dal file di configurazione o `ws://127.0.0.1:<port>`)
* `--token <token>`: token del Gateway (se richiesto)
* `--password <password>`: password del Gateway (se richiesta)
* `--session <key>`: chiave di sessione (valore predefinito: `main`, oppure `global` quando lo scope è `global`)
* `--deliver`: Consegna le risposte dell&#39;assistente al provider (disattivato per impostazione predefinita)
* `--thinking <level>`: Sovrascrive il livello di ragionamento per gli invii
* `--timeout-ms <ms>`: timeout dell&#39;agente in ms (predefinito a `agents.defaults.timeoutSeconds`)

<div id="troubleshooting">
  ## Risoluzione dei problemi
</div>

Nessun output dopo aver inviato un messaggio:

* Esegui `/status` nella TUI per confermare che il Gateway è connesso e se è inattivo o occupato.
* Controlla i log del Gateway: `openclaw logs --follow`.
* Verifica che l&#39;agente possa essere eseguito: `openclaw status` e `openclaw models status`.
* Se ti aspetti messaggi in un canale di chat, abilita la consegna dei messaggi (`/deliver on` oppure `--deliver`).
* `--history-limit <n>`: voci di cronologia da caricare (valore predefinito: 200)

<div id="troubleshooting">
  ## Risoluzione dei problemi
</div>

* `disconnected`: assicurati che il Gateway sia in esecuzione e che i tuoi `--url/--token/--password` siano corretti.
* Nessun agente nel selettore: verifica `openclaw agents list` e la tua configurazione di routing.
* Selettore delle sessioni vuoto: potresti essere nello scope globale oppure non avere ancora alcuna sessione.