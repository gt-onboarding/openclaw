---
title: Cron job
summary: "Cron job e riattivazioni per lo scheduler del Gateway"
read_when:
  - Pianifichi job in background o riattivazioni
  - Configuri automazioni da eseguire con o insieme agli heartbeat
  - Stai decidendo se usare heartbeat o cron per attività pianificate
---

<div id="cron-jobs-gateway-scheduler">
  # Cron jobs (Gateway scheduler)
</div>

> **Cron vs Heartbeat?** Consulta [Cron vs Heartbeat](/it/automation/cron-vs-heartbeat) per indicazioni su quando usare ciascuno.

Cron è lo scheduler integrato del Gateway. Memorizza i job in modo persistente, riattiva l’agente
al momento giusto e può, opzionalmente, recapitare l’output a una chat.

Se vuoi *&quot;eseguire questo ogni mattina&quot;* o *&quot;attivare l’agente tra 20 minuti&quot;*,
cron è il meccanismo da usare.

<div id="tldr">
  ## TL;DR
</div>

* Cron viene eseguito **all&#39;interno del Gateway** (non all&#39;interno del modello).
* I job persistono in `~/.openclaw/cron/`, così i riavvii non fanno perdere le pianificazioni.
* Due stili di esecuzione:
  * **Sessione principale**: accoda un evento di sistema, poi esegue al prossimo heartbeat.
  * **Isolato**: esegue un turno dedicato di un agente in `cron:<jobId>`, con consegna dell&#39;output opzionale.
* I risvegli sono di prima classe: un job può richiedere «sveglia ora» invece di «prossimo heartbeat».

<div id="beginner-friendly-overview">
  ## Panoramica per principianti
</div>

Pensa a un cron job come: **quando** eseguire + **cosa** fare.

1. **Scegli una pianificazione**
   * Promemoria una tantum → `schedule.kind = "at"` (CLI: `--at`)
   * Job ricorrente → `schedule.kind = "every"` oppure `schedule.kind = "cron"`
   * Se il tuo timestamp ISO omette il fuso orario, viene trattato come **UTC**.

2. **Scegli dove viene eseguito**
   * `sessionTarget: "main"` → viene eseguito al prossimo heartbeat nel contesto principale.
   * `sessionTarget: "isolated"` → esegue un turno dedicato dell&#39;agente in `cron:<jobId>`.

3. **Scegli il payload**
   * Sessione principale → `payload.kind = "systemEvent"`
   * Sessione isolata → `payload.kind = "agentTurn"`

Opzionale: `deleteAfterRun: true` rimuove dallo store i job una tantum eseguiti con successo.

<div id="concepts">
  ## Concetti
</div>

<div id="jobs">
  ### Job
</div>

Un cron job è un record memorizzato con:

* una **schedule** (quando deve essere eseguito),
* un **payload** (cosa deve fare),
* una **delivery** opzionale (dove deve essere inviato l&#39;output),
* un **binding dell&#39;agente** opzionale (`agentId`): esegui il job con uno specifico agente; se
  mancante o sconosciuto, il Gateway utilizza l&#39;agente predefinito.

I job sono identificati da un `jobId` stabile (usato dalle API CLI/Gateway).
Nelle chiamate agli strumenti dell&#39;agente, `jobId` è canonico; il vecchio `id` è accettato per compatibilità.
I job possono opzionalmente auto-eliminarsi dopo un&#39;esecuzione one-shot riuscita tramite `deleteAfterRun: true`.

<div id="schedules">
  ### Pianificazioni
</div>

Cron supporta tre tipi di pianificazione:

* `at`: timestamp singolo (ms dall’epoca Unix). Gateway accetta ISO 8601 e converte in UTC.
* `every`: intervallo fisso (ms).
* `cron`: espressione cron a 5 campi con fuso orario IANA opzionale.

Le espressioni cron usano `croner`. Se il fuso orario viene omesso, viene utilizzato
il fuso orario locale dell’host del Gateway.

<div id="main-vs-isolated-execution">
  ### Esecuzione principale vs esecuzione isolata
</div>

<div id="main-session-jobs-system-events">
  #### Job della sessione principale (eventi di sistema)
</div>

I job principali accodano un evento di sistema e, opzionalmente, riattivano il runner di heartbeat.
Devono usare `payload.kind = "systemEvent"`.

* `wakeMode: "next-heartbeat"` (predefinito): l&#39;evento attende il prossimo heartbeat pianificato.
* `wakeMode: "now"`: l&#39;evento attiva un&#39;esecuzione immediata dell&#39;heartbeat.

Questa è la scelta migliore quando ti serve il normale prompt di heartbeat + il contesto della sessione principale.
Vedi [Heartbeat](/it/gateway/heartbeat).

<div id="isolated-jobs-dedicated-cron-sessions">
  #### Job isolati (sessioni cron dedicate)
</div>

I job isolati eseguono un turno dedicato dell’agente nella sessione `cron:<jobId>`.

Comportamenti chiave:

* Il prompt è prefissato con `[cron:<jobId> <job name>]` per la tracciabilità.
* Ogni esecuzione avvia un **nuovo id di sessione** (nessun riutilizzo del contesto di conversazione precedente).
* Un riepilogo viene inviato alla sessione principale (prefisso `Cron`, configurabile).
* `wakeMode: "now"` attiva un heartbeat immediato dopo l’invio del riepilogo.
* Se `payload.deliver: true`, l’output viene recapitato a un canale; altrimenti rimane interno.

Usa job isolati per attività rumorose, frequenti o di &quot;background&quot; che non dovrebbero saturare
la cronologia principale della chat.

<div id="payload-shapes-what-runs">
  ### Formati del payload (cosa viene eseguito)
</div>

Sono supportati due tipi di payload:

* `systemEvent`: solo nella sessione principale, instradato tramite il prompt di heartbeat.
* `agentTurn`: solo nella sessione isolata, esegue un turno dedicato dell&#39;agente.

Campi comuni di `agentTurn`:

* `message`: prompt testuale obbligatorio.
* `model` / `thinking`: override opzionali (vedi sotto).
* `timeoutSeconds`: override opzionale del timeout.
* `deliver`: `true` per inviare l&#39;output a una destinazione di canale.
* `channel`: `last` o uno specifico canale.
* `to`: destinazione specifica del canale (telefono/chat/id canale).
* `bestEffortDeliver`: evita che il job fallisca se la consegna non riesce.

Opzioni di isolamento (solo per `session=isolated`):

* `postToMainPrefix` (CLI: `--post-prefix`): prefisso per l&#39;evento di sistema nella sessione principale.
* `postToMainMode`: `summary` (predefinito) o `full`.
* `postToMainMaxChars`: numero massimo di caratteri quando `postToMainMode=full` (predefinito 8000).

<div id="model-and-thinking-overrides">
  ### Override di modello e livello di ragionamento
</div>

I job isolati (`agentTurn`) possono applicare override del modello e del livello di ragionamento:

* `model`: stringa provider/modello (ad es. `anthropic/claude-sonnet-4-20250514`) o alias (ad es. `opus`)
* `thinking`: livello di ragionamento (`off`, `minimal`, `low`, `medium`, `high`, `xhigh`; solo modelli GPT-5.2 + Codex)

Nota: puoi impostare `model` anche sui job della sessione principale, ma in tal caso modifichi
il modello condiviso della sessione principale. Ti consigliamo di usare override del modello
solo per job isolati, per evitare cambi di contesto imprevisti.

Priorità di risoluzione:

1. Override nel payload del job (massima priorità)
2. Valori predefiniti specifici dell&#39;hook (ad es. `hooks.gmail.model`)
3. Valore predefinito della configurazione dell&#39;agente

<div id="delivery-channel-target">
  ### Consegna (canale + destinazione)
</div>

I job isolati possono recapitare l’output a un canale. Il payload del job può specificare:

* `channel`: `whatsapp` / `telegram` / `discord` / `slack` / `mattermost` (plugin) / `signal` / `imessage` / `last`
* `to`: destinatario specifico del canale

Se `channel` o `to` vengono omessi, cron può ripiegare sulla “last route” della sessione principale
(l’ultimo punto in cui l’agente ha risposto).

Note sulla consegna:

* Se `to` è impostato, cron consegna automaticamente l’output finale dell’agente anche se `deliver` è omesso.
* Usa `deliver: true` quando vuoi la consegna tramite last-route senza un `to` esplicito.
* Usa `deliver: false` per mantenere l’output interno anche se è presente un `to`.

Promemoria sul formato della destinazione:

* Le destinazioni Slack/Discord/Mattermost (plugin) dovrebbero usare prefissi espliciti (ad es. `channel:<id>`, `user:<id>`) per evitare ambiguità.
* I topic di Telegram dovrebbero usare la forma `:topic:` (vedi sotto).

<div id="telegram-delivery-targets-topics-forum-threads">
  #### Destinazioni di invio Telegram (argomenti / thread del forum)
</div>

Telegram supporta gli argomenti del forum tramite `message_thread_id`. Per le consegne pianificate tramite cron, puoi codificare
l&#39;argomento/thread nel campo `to`:

* `-1001234567890` (solo chat ID)
* `-1001234567890:topic:123` (preferito: marcatore di argomento esplicito)
* `-1001234567890:123` (forma abbreviata: suffisso numerico)

Anche le destinazioni con prefisso come `telegram:...` / `telegram:group:...` sono accettate:

* `telegram:group:-1001234567890:topic:123`

<div id="storage-history">
  ## Archiviazione e cronologia
</div>

* Archivio dei job: `~/.openclaw/cron/jobs.json` (JSON gestito dal Gateway).
* Cronologia delle esecuzioni: `~/.openclaw/cron/runs/<jobId>.jsonl` (JSONL, con eliminazione automatica).
* Percorso dell’archivio da sovrascrivere: `cron.store` nella configurazione.

<div id="configuration">
  ## Configurazione
</div>

```json5
{
  cron: {
    enabled: true, // predefinito: true
    store: "~/.openclaw/cron/jobs.json",
    maxConcurrentRuns: 1 // predefinito: 1
  }
}
```

Disabilita completamente cron:

* `cron.enabled: false` (config)
* `OPENCLAW_SKIP_CRON=1` (env)

<div id="cli-quickstart">
  ## Guida rapida alla CLI
</div>

Promemoria una tantum (UTC ISO, eliminazione automatica in caso di esito positivo):

```bash
openclaw cron add \
  --name "Send reminder" \
  --at "2026-01-12T18:00:00Z" \
  --session main \
  --system-event "Reminder: submit expense report." \
  --wake now \
  --delete-after-run
```

Promemoria una tantum (sessione principale, attivare immediatamente):

```bash
openclaw cron add \
  --name "Calendar check" \
  --at "20m" \
  --session main \
  --system-event "Next heartbeat: check calendar." \
  --wake now
```

Job ricorrente isolato (invio a WhatsApp):

```bash
openclaw cron add \
  --name "Morning status" \
  --cron "0 7 * * *" \
  --tz "America/Los_Angeles" \
  --session isolated \
  --message "Summarize inbox + calendar for today." \
  --deliver \
  --channel whatsapp \
  --to "+15551234567"
```

Job ricorrente isolato (consegna a un topic di Telegram):

```bash
openclaw cron add \
  --name "Nightly summary (topic)" \
  --cron "0 22 * * *" \
  --tz "America/Los_Angeles" \
  --session isolated \
  --message "Riepiloga la giornata; invia al topic notturno." \
  --deliver \
  --channel telegram \
  --to "-1001234567890:topic:123"
```

Job isolato con override di modello e thinking:

````bash
openclaw cron add \
  --name "Deep analysis" \
  --cron "0 6 * * 1" \
  --tz "America/Los_Angeles" \
  --session isolated \
  --message "Weekly deep analysis of project progress." \
  --model "opus" \
  --thinking high \
  --deliver \
  --channel whatsapp \
  --to "+15551234567"

Agent selection (multi-agent setups):
```bash
# Pin a job to agent "ops" (falls back to default if that agent is missing)
openclaw cron add --name "Ops sweep" --cron "0 6 * * *" --session isolated --message "Controlla coda ops" --agent ops

# Switch or clear the agent on an existing job
openclaw cron edit <jobId> --agent ops
openclaw cron edit <jobId> --clear-agent
````

````

Esecuzione manuale (debug):
```bash
openclaw cron run <jobId> --force
````

Modifica un job esistente (patch dei campi):

```bash
openclaw cron edit <jobId> \
  --message "Updated prompt" \
  --model "opus" \
  --thinking low
```

Cronologia delle esecuzioni:

```bash
openclaw cron runs --id <jobId> --limit 50
```

Evento di sistema immediato senza creare un job pianificato:

```bash
openclaw system event --mode now --text "Next heartbeat: check battery."
```

<div id="gateway-api-surface">
  ## Endpoint API del Gateway
</div>

* `cron.list`, `cron.status`, `cron.add`, `cron.update`, `cron.remove`
* `cron.run` (forzato o alla scadenza), `cron.runs`
  Per eventi di sistema immediati senza un job pianificato, utilizza [`openclaw system event`](/it/cli/system).

<div id="troubleshooting">
  ## Risoluzione dei problemi
</div>

<div id="nothing-runs">
  ### &quot;Non viene eseguito niente&quot;
</div>

* Verifica che cron sia abilitato: `cron.enabled` e `OPENCLAW_SKIP_CRON`.
* Verifica che il Gateway sia in esecuzione senza interruzioni (cron viene eseguito all&#39;interno del processo del Gateway).
* Per le pianificazioni `cron`: conferma il fuso orario (`--tz`) rispetto al fuso orario dell&#39;host.

<div id="telegram-delivers-to-the-wrong-place">
  ### Telegram recapita alla destinazione sbagliata
</div>

* Per i topic del forum, usa `-100…:topic:<id>` in modo che sia esplicito e non ambiguo.
* Se vedi prefissi `telegram:...` nei log o nei target memorizzati come &quot;ultimo instradamento&quot;, è normale;
  il job cron di consegna li accetta e continua comunque a interpretare correttamente gli ID dei topic.