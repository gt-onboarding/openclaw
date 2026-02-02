---
title: Heartbeat
summary: "Messaggi di polling dell'heartbeat e regole di notifica"
read_when:
  - Regolare la frequenza o i messaggi di heartbeat
  - Decidere tra heartbeat e cron per le attività pianificate
---

<div id="heartbeat-gateway">
  # Heartbeat (Gateway)
</div>

> **Heartbeat vs Cron?** Vedi [Cron vs Heartbeat](/it/automation/cron-vs-heartbeat) per indicazioni su quando usare ciascuno.

Heartbeat esegue **turni periodici dell&#39;agente** nella sessione principale, così il modello può far emergere tutto ciò che richiede attenzione senza tempestarti di messaggi.

<div id="quick-start-beginner">
  ## Avvio rapido (principiante)
</div>

1. Lascia gli heartbeat attivi (il valore predefinito è `30m`, o `1h` per Anthropic OAuth/setup-token) oppure imposta una cadenza personalizzata.
2. Crea una piccola checklist `HEARTBEAT.md` nello spazio di lavoro dell&#39;agente (opzionale ma consigliato).
3. Decidi dove devono essere recapitati i messaggi di heartbeat (`target: "last"` è il valore predefinito).
4. Opzionale: abilita l&#39;invio del reasoning dell&#39;heartbeat per maggiore trasparenza.
5. Opzionale: limita gli heartbeat alle ore di attività (ora locale).

Esempio di configurazione:

```json5
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m",
        target: "last",
        // activeHours: { start: "08:00", end: "24:00" },
        // includeReasoning: true, // opzionale: invia anche un messaggio separato `Reasoning:`
      }
    }
  }
}
```

<div id="defaults">
  ## Valori predefiniti
</div>

* Intervallo: `30m` (oppure `1h` quando Anthropic OAuth/setup-token è la modalità di autenticazione rilevata). Imposta `agents.defaults.heartbeat.every` o, per singolo agente, `agents.list[].heartbeat.every`; usa `0m` per disabilitare.
* Corpo del prompt (configurabile tramite `agents.defaults.heartbeat.prompt`):
  `Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.`
* Il prompt di heartbeat viene inviato **letteralmente** come messaggio dell’utente. Il prompt
  di sistema include una sezione “Heartbeat” e l’esecuzione è contrassegnata internamente.
* Le ore attive (`heartbeat.activeHours`) vengono verificate nel fuso orario configurato.
  Al di fuori della finestra, gli heartbeat vengono ignorati fino al successivo tick all’interno della finestra.

<div id="what-the-heartbeat-prompt-is-for">
  ## A cosa serve il prompt di heartbeat
</div>

Il prompt predefinito è volutamente ampio:

* **Attività in background**: “Consider outstanding tasks” invita l’agente a rivedere
  i follow-up (posta in arrivo, calendario, promemoria, lavoro in coda) e a far emergere eventuali elementi urgenti.
* **Check-in con l’utente**: “Checkup sometimes on your human during day time” porta
  a un messaggio occasionale e leggero del tipo “ti serve qualcosa?”, ma evita lo spam notturno
  usando il fuso orario locale configurato (vedi [/concepts/timezone](/it/concepts/timezone)).

Se vuoi che l’heartbeat faccia qualcosa di molto specifico (ad es. “check Gmail PubSub
stats” o “verify gateway health”), imposta `agents.defaults.heartbeat.prompt` (oppure
`agents.list[].heartbeat.prompt`) su un corpo personalizzato (inviato così com’è).

<div id="response-contract">
  ## Contratto di risposta
</div>

* Se non è necessaria alcuna azione, rispondi con **`HEARTBEAT_OK`**.
* Durante i cicli di heartbeat, OpenClaw tratta `HEARTBEAT_OK` come un ack quando compare
  all&#39;**inizio o alla fine** della risposta. Il token viene rimosso e la risposta viene
  scartata se il contenuto rimanente è **≤ `ackMaxChars`** (valore predefinito: 300).
* Se `HEARTBEAT_OK` compare **in mezzo** a una risposta, non viene trattato
  in modo speciale.
* In caso di avvisi, **non** includere `HEARTBEAT_OK`; restituisci solo il testo dell&#39;avviso.

Al di fuori dei cicli di heartbeat, un `HEARTBEAT_OK` fuori posto all&#39;inizio/fine di un messaggio viene rimosso
e registrato nei log; un messaggio che contiene solo `HEARTBEAT_OK` viene scartato.

<div id="config">
  ## Configurazione
</div>

```json5
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m",           // default: 30m (0m disables)
        model: "anthropic/claude-opus-4-5",
        includeReasoning: false, // default: false (deliver separate Reasoning: message when available)
        target: "last",         // last | none | <channel id> (core or plugin, e.g. "bluebubbles")
        to: "+15551234567",     // optional channel-specific override
        prompt: "Leggi HEARTBEAT.md se esiste (contesto dello spazio di lavoro). Seguilo rigorosamente. Non dedurre né ripetere vecchie attività da conversazioni precedenti. Se nulla richiede attenzione, rispondi HEARTBEAT_OK.",
        ackMaxChars: 300         // max chars allowed after HEARTBEAT_OK
      }
    }
  }
}
```

<div id="scope-and-precedence">
  ### Scope e precedenza
</div>

* `agents.defaults.heartbeat` definisce il comportamento globale dell&#39;heartbeat.
* `agents.list[].heartbeat` si applica in sovrascrittura; se un qualsiasi agente ha un blocco `heartbeat`, **solo quegli agenti** eseguono gli heartbeat.
* `channels.defaults.heartbeat` definisce i valori di default di visibilità per tutti i canali.
* `channels.<channel>.heartbeat` sovrascrive i valori di default del canale.
* `channels.<channel>.accounts.<id>.heartbeat` (canali multi-account) sovrascrive le impostazioni per canale.

<div id="per-agent-heartbeats">
  ### Heartbeat per agente
</div>

Se una qualsiasi voce in `agents.list[]` include un blocco `heartbeat`, **solo quegli agenti**
eseguono l’heartbeat. Il blocco per-agente viene applicato sopra `agents.defaults.heartbeat`
(in questo modo puoi impostare una sola volta le impostazioni predefinite condivise e sovrascriverle per singolo agente).

Esempio: due agenti, solo il secondo agente esegue l’heartbeat.

```json5
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m",
        target: "last"
      }
    },
    list: [
      { id: "main", default: true },
      {
        id: "ops",
        heartbeat: {
          every: "1h",
          target: "whatsapp",
          to: "+15551234567",
          prompt: "Esegui read su HEARTBEAT.md se esiste (contesto dello spazio di lavoro). Seguilo rigorosamente. Non dedurre né ripetere vecchie attività da chat precedenti. Se nulla richiede attenzione, rispondi HEARTBEAT_OK."
        }
      }
    ]
  }
}
```

<div id="field-notes">
  ### Note operative
</div>

* `every`: intervallo di heartbeat (stringa di durata; unità predefinita = minuti).
* `model`: override facoltativo del modello per le esecuzioni di heartbeat (`provider/model`).
* `includeReasoning`: quando abilitato, recapita anche il messaggio separato `Reasoning:` quando disponibile (stessa struttura di `/reasoning on`).
* `session`: chiave di sessione facoltativa per le esecuzioni di heartbeat.
  * `main` (predefinita): sessione principale dell&#39;agente.
  * Chiave di sessione esplicita (copiata da `openclaw sessions --json` o dalla [CLI delle sessioni](/it/cli/sessions)).
  * Formati della chiave di sessione: vedi [Sessioni](/it/concepts/session) e [Gruppi](/it/concepts/groups).
* `target`:
  * `last` (predefinito): recapita all&#39;ultimo canale esterno utilizzato.
  * canale esplicito: `whatsapp` / `telegram` / `discord` / `googlechat` / `slack` / `msteams` / `signal` / `imessage`.
  * `none`: esegui l&#39;heartbeat ma **non recapitare** all&#39;esterno.
* `to`: override facoltativo del destinatario (ID specifico del canale, ad es. E.164 per WhatsApp o un ID chat di Telegram).
* `prompt`: sostituisce il corpo del prompt predefinito (non viene unito a quello predefinito).
* `ackMaxChars`: numero massimo di caratteri consentiti dopo `HEARTBEAT_OK` prima della consegna.

<div id="delivery-behavior">
  ## Comportamento di consegna
</div>

* Gli heartbeat vengono eseguiti nella sessione principale dell’agente per impostazione predefinita (`agent:<id>:<mainKey>`),
  oppure in `global` quando `session.scope = "global"`. Imposta `session` per sovrascrivere
  con una specifica sessione del canale (Discord/WhatsApp/etc.).
* `session` influisce solo sul contesto di esecuzione; la consegna è controllata da `target` e `to`.
* Per consegnare a uno specifico canale/destinatario, imposta `target` + `to`. Con
  `target: "last"`, la consegna utilizza l’ultimo canale esterno per quella sessione.
* Se la coda principale è occupata, l’heartbeat viene ignorato e ritentato più tardi.
* Se `target` non si risolve in alcuna destinazione esterna, l’esecuzione avviene comunque ma
  non viene inviato alcun messaggio in uscita.
* Le risposte generate solo dall’heartbeat **non** mantengono viva la sessione; l’ultimo `updatedAt`
  viene ripristinato in modo che la scadenza per inattività si comporti normalmente.

<div id="visibility-controls">
  ## Controlli di visibilità
</div>

Per impostazione predefinita, le conferme `HEARTBEAT_OK` vengono disattivate mentre viene recapitato il contenuto degli avvisi. Puoi modificare questo comportamento per singolo canale o per singolo account:

```yaml
channels:
  defaults:
    heartbeat:
      showOk: false      # Hide HEARTBEAT_OK (default)
      showAlerts: true   # Show alert messages (default)
      useIndicator: true # Emit indicator events (default)
  telegram:
    heartbeat:
      showOk: true       # Show OK acknowledgments on Telegram
  whatsapp:
    accounts:
      work:
        heartbeat:
          showAlerts: false # Sopprimi consegna allerte per questo account
```

Precedenza: per-account → per-canale → impostazioni predefinite del canale → impostazioni predefinite integrate.

<div id="what-each-flag-does">
  ### Cosa fa ciascun flag
</div>

* `showOk`: invia una conferma `HEARTBEAT_OK` quando il modello restituisce una risposta che contiene solo OK.
* `showAlerts`: invia il contenuto dell’avviso quando il modello restituisce una risposta non OK.
* `useIndicator`: emette eventi indicatore per le superfici di stato della UI.

Se **tutti e tre** sono false, OpenClaw salta completamente l’esecuzione dell’heartbeat (nessuna chiamata al modello).

<div id="per-channel-vs-per-account-examples">
  ### Esempi per canale e per account
</div>

```yaml
channels:
  defaults:
    heartbeat:
      showOk: false
      showAlerts: true
      useIndicator: true
  slack:
    heartbeat:
      showOk: true # tutti gli account Slack
    accounts:
      ops:
        heartbeat:
          showAlerts: false # sopprime gli alert solo per l'account ops
  telegram:
    heartbeat:
      showOk: true
```

<div id="common-patterns">
  ### Pattern comuni
</div>

| Obiettivo | Configurazione |
| --- | --- |
| Comportamento predefinito (OK silenziosi, avvisi attivi) | *(nessuna configurazione necessaria)* |
| Completamente silenzioso (nessun messaggio, nessun indicatore) | `channels.defaults.heartbeat: { showOk: false, showAlerts: false, useIndicator: false }` |
| Solo indicatore (nessun messaggio) | `channels.defaults.heartbeat: { showOk: false, showAlerts: false, useIndicator: true }` |
| OK in un solo canale | `channels.telegram.heartbeat: { showOk: true }` |

<div id="heartbeatmd-optional">
  ## HEARTBEAT.md (opzionale)
</div>

Se esiste un file `HEARTBEAT.md` nello spazio di lavoro, il prompt predefinito
indica all&#39;agente di leggerlo. Pensa a questo file come alla tua “checklist di heartbeat”:
piccola, stabile e sicura da includere ogni 30 minuti.

Se `HEARTBEAT.md` esiste ma è di fatto vuoto (solo righe vuote e intestazioni
markdown come `# Heading`), OpenClaw salta l&#39;esecuzione dell&#39;heartbeat per
risparmiare chiamate api. Se il file manca, l&#39;heartbeat viene comunque eseguito
e il modello decide cosa fare.

Tienilo molto piccolo (breve checklist o promemoria) per evitare di gonfiare il prompt.

Esempio di `HEARTBEAT.md`:

```md
# Checklist heartbeat

- Scansione rapida: c'è qualcosa di urgente nelle inbox?
- Se è giorno, effettua un check-in leggero se non ci sono altre attività in sospeso.
- Se un task è bloccato, annota *cosa manca* e chiedi a Peter la prossima volta.
```

<div id="can-the-agent-update-heartbeatmd">
  ### L&#39;agente può aggiornare HEARTBEAT.md?
</div>

Sì — se glielo chiedi.

`HEARTBEAT.md` è solo un normale file nello spazio di lavoro dell&#39;agente, quindi
puoi dire all&#39;agente (in una normale chat) qualcosa come:

* &quot;Aggiorna `HEARTBEAT.md` per aggiungere un controllo quotidiano del calendario.&quot;
* &quot;Riscrivi `HEARTBEAT.md` in modo che sia più corto e focalizzato sui follow-up della casella di posta.&quot;

Se vuoi che questo avvenga in modo proattivo, puoi anche includere una riga
esplicita nel tuo prompt di heartbeat, ad esempio: &quot;Se la checklist diventa
obsoleta, aggiorna HEARTBEAT.md con una versione migliore.&quot;

Nota sulla sicurezza: non inserire segreti (chiavi API, numeri di telefono,
token privati) in `HEARTBEAT.md` — diventa parte del contesto del prompt.

<div id="manual-wake-on-demand">
  ## Attivazione manuale (on-demand)
</div>

Puoi mettere in coda un evento di sistema e attivare immediatamente un heartbeat con:

```bash
openclaw system event --text "Verifica follow-up urgenti" --mode now
```

Se più agenti hanno `heartbeat` configurato, un risveglio manuale esegue
immediatamente l&#39;heartbeat di ciascuno di quegli agenti.

Usa `--mode next-heartbeat` per attendere il prossimo tick programmato.

<div id="reasoning-delivery-optional">
  ## Consegna del ragionamento (opzionale)
</div>

Per impostazione predefinita, gli heartbeat inviano solo il payload finale della “risposta”.

Se vuoi maggiore trasparenza, abilita:

* `agents.defaults.heartbeat.includeReasoning: true`

Quando è abilitato, gli heartbeat inviano anche un messaggio separato con prefisso
`Reasoning:` (stessa struttura di `/reasoning on`). Questo può essere utile quando l&#39;agente
gestisce più sessioni/codex e vuoi vedere perché ha deciso di pingarti,
ma può anche far trapelare più dettagli interni di quanto desideri. È preferibile lasciarlo
disattivato nelle chat di gruppo.

<div id="cost-awareness">
  ## Consapevolezza dei costi
</div>

Gli heartbeat eseguono turni completi dell&#39;agente. Intervalli più brevi consumano più token. Mantieni
`HEARTBEAT.md` ridotto e valuta un `model` più economico o `target: "none"` se
ti servono solo aggiornamenti dello stato interno.