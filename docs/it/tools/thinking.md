---
title: Ragionamento
summary: "Sintassi delle direttive /think + /verbose e come influenzano il ragionamento del modello"
read_when:
  - Regolazione del parsing o dei valori predefiniti delle direttive /think o /verbose
---

<div id="thinking-levels-think-directives">
  # Livelli di pensiero (direttive /think)
</div>

<div id="what-it-does">
  ## Cosa fa
</div>

* Direttiva inline in qualsiasi body in ingresso: `/t <level>`, `/think:<level>`, oppure `/thinking <level>`.
* Livelli (alias): `off | minimal | low | medium | high | xhigh` (solo modelli GPT-5.2 + Codex)
  * minimal → “think”
  * low → “think hard”
  * medium → “think harder”
  * high → “ultrathink” (budget massimo)
  * xhigh → “ultrathink+” (solo modelli GPT-5.2 + Codex)
  * `highest`, `max` vengono mappati su `high`.
* Note sul provider:
  * Z.AI (`zai/*`) supporta solo il pensiero binario (`on`/`off`). Qualsiasi livello diverso da `off` è trattato come `on` (mappato su `low`).

<div id="resolution-order">
  ## Ordine di risoluzione
</div>

1. Direttiva inline nel messaggio (si applica solo a quel messaggio).
2. Override della sessione (impostato inviando un messaggio che contiene solo la direttiva).
3. Valore predefinito globale (`agents.defaults.thinkingDefault` nella configurazione).
4. Fallback: basso per i modelli con capacità di ragionamento; disattivato negli altri casi.

<div id="setting-a-session-default">
  ## Impostare un valore predefinito di sessione
</div>

* Invia un messaggio che sia **solo** la direttiva (spazi consentiti), ad es. `/think:medium` o `/t high`.
* Questa impostazione resta valida per la sessione corrente (per mittente per impostazione predefinita); viene azzerata da `/think:off` o dal reset per inattività della sessione.
* Viene inviata una risposta di conferma (`Livello di riflessione impostato su alto.` / `Riflessione disabilitata.`). Se il livello non è valido (ad es. `/thinking big`), il comando viene rifiutato con un suggerimento e lo stato della sessione rimane invariato.
* Invia `/think` (o `/think:`) senza argomenti per visualizzare il livello di riflessione corrente.

<div id="application-by-agent">
  ## Applicazione per agente
</div>

* **Pi incorporato**: il livello risolto viene passato al runtime in-process dell&#39;agente Pi.

<div id="verbose-directives-verbose-or-v">
  ## Direttive verbose (/verbose o /v)
</div>

* Livelli: `on` (minimo) | `full` | `off` (predefinito).
* Un messaggio che contiene solo la direttiva attiva/disattiva la modalità verbose per la sessione e risponde con `Verbose logging enabled.` / `Verbose logging disabled.`; livelli non validi restituiscono un suggerimento senza modificare lo stato.
* `/verbose off` memorizza un override esplicito per la sessione; rimuovilo dalla Sessions UI selezionando `inherit`.
* Una direttiva inline influisce solo su quel messaggio; in tutti gli altri casi si applicano i valori predefiniti di sessione/globali.
* Invia `/verbose` (o `/verbose:`) senza argomenti per visualizzare il livello verbose corrente.
* Quando verbose è attivo, gli agenti che emettono risultati di strumenti strutturati (Pi, altri agenti JSON) inviano ogni chiamata di strumento come un proprio messaggio solo-metadata, con prefisso `<emoji> <tool-name>: <arg>` quando disponibile (percorso/comando). Questi riepiloghi degli strumenti vengono inviati non appena ogni strumento viene avviato (bolle separate), non come delta in streaming.
* Quando verbose è `full`, anche gli output degli strumenti vengono inoltrati dopo il completamento (bolla separata, troncata a una lunghezza sicura). Se attivi/disattivi `/verbose on|full|off` mentre un&#39;esecuzione è in corso, le bolle degli strumenti successive rispettano la nuova impostazione.

<div id="reasoning-visibility-reasoning">
  ## Visibilità del ragionamento (/reasoning)
</div>

* Livelli: `on|off|stream`.
* Un messaggio di sola direttiva controlla se i blocchi di ragionamento vengono mostrati nelle risposte.
* Quando è abilitata, la modalità di ragionamento viene inviata come **messaggio separato** con prefisso `Reasoning:`.
* `stream` (solo Telegram): trasmette il ragionamento nel riquadro di bozza di Telegram mentre la risposta viene generata, quindi invia la risposta finale senza ragionamento.
* Alias: `/reason`.
* Invia `/reasoning` (o `/reasoning:`) senza argomento per vedere il livello di ragionamento attuale.

<div id="related">
  ## Contenuti correlati
</div>

* La documentazione sulla modalità Elevated si trova in [Modalità Elevated](/it/tools/elevated).

<div id="heartbeats">
  ## Heartbeat
</div>

* Il corpo della sonda di heartbeat è il prompt di heartbeat configurato (predefinito: `Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.`). Le direttive in linea in un messaggio di heartbeat si applicano come di consueto (ma evita di modificare i valori predefiniti della sessione tramite gli heartbeat).
* L&#39;invio degli heartbeat, per impostazione predefinita, include solo il payload finale. Per inviare anche il messaggio separato `Reasoning:` (quando disponibile), imposta `agents.defaults.heartbeat.includeReasoning: true` oppure, per singolo agente, `agents.list[].heartbeat.includeReasoning: true`.

<div id="web-chat-ui">
  ## Web chat UI
</div>

* Quando la pagina viene caricata, il selettore del livello di thinking della web chat replica il livello della sessione memorizzato nello store/config della sessione in ingresso.
* Se scegli un altro livello, questo si applica solo al messaggio successivo (`thinkingOnce`); dopo l&#39;invio, il selettore torna al livello di sessione memorizzato.
* Per modificare il livello predefinito della sessione, invia la direttiva `/think:&lt;level&gt;` (come in precedenza); il selettore la rifletterà dopo il successivo ricaricamento della pagina.