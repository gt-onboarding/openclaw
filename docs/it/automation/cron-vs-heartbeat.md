---
title: Cron vs heartbeat
summary: "Guida alla scelta tra heartbeat e cron job per l'automazione"
read_when:
  - Decidere come pianificare attività ricorrenti
  - Configurare monitoraggio o notifiche in background
  - Ottimizzare l'uso dei token per controlli periodici
---

<div id="cron-vs-heartbeat-when-to-use-each">
  # Cron vs Heartbeat: quando usare l&#39;uno o l&#39;altro
</div>

Sia gli heartbeat che i job cron ti consentono di eseguire attività pianificate. Questa guida ti aiuta a scegliere il meccanismo giusto per il tuo caso d&#39;uso.

<div id="quick-decision-guide">
  ## Guida rapida alle decisioni
</div>

| Caso d&#39;uso | Consigliato | Perché |
|------------|-------------|--------|
| Controllare la posta in arrivo ogni 30 minuti | Heartbeat | Raggruppa con altri controlli, sensibile al contesto |
| Inviare il report giornaliero alle 9:00 in punto | Cron (isolato) | È necessaria una tempistica esatta |
| Monitorare il calendario per eventi imminenti | Heartbeat | Si adatta naturalmente alla consapevolezza periodica |
| Eseguire un&#39;analisi approfondita settimanale | Cron (isolato) | Attività autonoma, può usare un modello diverso |
| Ricordamelo tra 20 minuti | Cron (main, `--at`) | Esecuzione singola con tempistica precisa |
| Controllo in background dello stato di salute del progetto | Heartbeat | Sfrutta il ciclo già esistente |

<div id="heartbeat-periodic-awareness">
  ## Heartbeat: consapevolezza periodica
</div>

Gli heartbeat vengono eseguiti nella **sessione principale** a intervalli regolari (valore predefinito: 30 min). Servono perché l&#39;agente possa verificare la situazione e far emergere qualsiasi elemento importante.

<div id="when-to-use-heartbeat">
  ### Quando utilizzare heartbeat
</div>

* **Controlli periodici multipli**: invece di 5 job cron separati che controllano inbox, calendario, meteo, notifiche e stato dei progetti, un singolo heartbeat può raggruppare tutte queste verifiche.
* **Decisioni basate sul contesto**: l&#39;agente ha a disposizione l&#39;intero contesto della sessione principale, quindi può decidere in modo intelligente cosa è urgente e cosa può aspettare.
* **Continuità conversazionale**: le esecuzioni di heartbeat condividono la stessa sessione, quindi l&#39;agente ricorda le conversazioni recenti e può fare follow-up in modo naturale.
* **Monitoraggio a basso overhead**: un solo heartbeat sostituisce molte piccole attività di polling.

<div id="heartbeat-advantages">
  ### Vantaggi dell&#39;heartbeat
</div>

* **Raggruppa più controlli**: Un singolo turno dell&#39;agente può esaminare insieme inbox, calendario e notifiche.
* **Riduce le chiamate API**: Un singolo heartbeat è più economico di 5 job cron isolati.
* **Consapevole del contesto**: L&#39;agente sa su cosa hai lavorato e può dare le priorità di conseguenza.
* **Soppressione intelligente**: Se nulla richiede attenzione, l&#39;agente risponde `HEARTBEAT_OK` e nessun messaggio viene inviato.
* **Tempistiche naturali**: Varia leggermente in base al carico della coda, il che va bene per la maggior parte dei casi di monitoraggio.

<div id="heartbeat-example-heartbeatmd-checklist">
  ### Esempio di heartbeat: checklist di HEARTBEAT.md
</div>

```md
# Checklist heartbeat

- Controlla l'email per messaggi urgenti
- Controlla il calendario per eventi nelle prossime 2 ore
- Se un'attività in background è terminata, riassumi i risultati
- Se inattivo per oltre 8 ore, invia un breve check-in
```

L&#39;agente legge questa configurazione a ogni heartbeat e gestisce tutti gli elementi in un&#39;unica interazione.

<div id="configuring-heartbeat">
  ### Configurazione di heartbeat
</div>

```json5
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m",        // interval
        target: "last",      // where to deliver alerts
        activeHours: { start: "08:00", end: "22:00" }  // opzionale
      }
    }
  }
}
```

Consulta [Heartbeat](/it/gateway/heartbeat) per la configurazione completa.

<div id="cron-precise-scheduling">
  ## Cron: pianificazione precisa
</div>

I cron job vengono eseguiti a **orari precisi** e possono essere eseguiti in sessioni isolate senza influire sul contesto principale.

<div id="when-to-use-cron">
  ### Quando usare cron
</div>

* **Quando serve una tempistica esatta**: &quot;Invia questo alle 9:00 ogni lunedì&quot; (non &quot;più o meno alle 9&quot;).
* **Attività autonome**: Attività che non richiedono contesto conversazionale.
* **Modello o tipo di ragionamento diverso**: Analisi pesante che richiede un modello più potente.
* **Promemoria una tantum**: &quot;Ricordamelo tra 20 minuti&quot; con `--at`.
* **Attività rumorose/frequenti**: Attività che intaserebbero la cronologia della sessione principale.
* **Trigger esterni**: Attività che devono essere eseguite indipendentemente dal fatto che l&#39;agente sia attivo o meno.

<div id="cron-advantages">
  ### Vantaggi di cron
</div>

* **Pianificazione precisa**: espressioni cron a 5 campi con supporto per il fuso orario.
* **Isolamento delle sessioni**: viene eseguito in `cron:<jobId>` senza sporcare la cronologia principale.
* **Override del modello**: usa un modello più economico o più potente per ogni job.
* **Controllo del recapito**: può recapitare direttamente a un canale; per impostazione predefinita invia comunque un riepilogo alla sessione principale (configurabile).
* **Nessun contesto di agente necessario**: viene eseguito anche se la sessione principale è inattiva o compattata.
* **Supporto one-shot**: `--at` per timestamp futuri precisi.

<div id="cron-example-daily-morning-briefing">
  ### Esempio di Cron: briefing mattutino quotidiano
</div>

```bash
openclaw cron add \
  --name "Morning briefing" \
  --cron "0 7 * * *" \
  --tz "America/New_York" \
  --session isolated \
  --message "Generate today's briefing: weather, calendar, top emails, news summary." \
  --model opus \
  --deliver \
  --channel whatsapp \
  --to "+15551234567"
```

Questo viene eseguito esattamente alle 7:00 del mattino, ora di New York, usa Opus per la qualità e invia direttamente su WhatsApp.

<div id="cron-example-one-shot-reminder">
  ### Esempio di Cron: promemoria una tantum
</div>

```bash
openclaw cron add \
  --name "Meeting reminder" \
  --at "20m" \
  --session main \
  --system-event "Reminder: standup meeting starts in 10 minutes." \
  --wake now \
  --delete-after-run
```

Consulta la sezione [Cron jobs](/it/automation/cron-jobs) per la documentazione completa della CLI.

<div id="decision-flowchart">
  ## Diagramma decisionale
</div>

```
Does the task need to run at an EXACT time?
  YES -> Use cron
  NO  -> Continue...

Does the task need isolation from main session?
  YES -> Use cron (isolated)
  NO  -> Continue...

Can this task be batched with other periodic checks?
  YES -> Use heartbeat (add to HEARTBEAT.md)
  NO  -> Use cron

Is this a one-shot reminder?
  YES -> Use cron with --at
  NO  -> Continue...

Does it need a different model or thinking level?
  YES -> Use cron (isolated) with --model/--thinking
  NO  -> Use heartbeat
```

<div id="combining-both">
  ## Usare entrambi
</div>

La configurazione più efficiente utilizza **entrambi**:

1. **Heartbeat** gestisce il monitoraggio di routine (posta in arrivo, calendario, notifiche) in un&#39;unica esecuzione batch ogni 30 minuti.
2. **Cron** gestisce pianificazioni precise (report giornalieri, revisioni settimanali) e promemoria singoli.

<div id="example-efficient-automation-setup">
  ### Esempio: configurazione efficiente dell&#39;automazione
</div>

**HEARTBEAT.md** (verificato ogni 30 minuti):

```md
# Heartbeat checklist
- Scan inbox for urgent emails
- Check calendar for events in next 2h
- Review any pending tasks
- Light check-in if quiet for 8+ hours
```

**Job cron** (esecuzione con orari precisi):

```bash
# Daily morning briefing at 7am
openclaw cron add --name "Morning brief" --cron "0 7 * * *" --session isolated --message "..." --deliver

# Revisione settimanale del progetto il lunedì alle 9
openclaw cron add --name "Weekly review" --cron "0 9 * * 1" --session isolated --message "..." --model opus

# One-shot reminder
openclaw cron add --name "Call back" --at "2h" --session main --system-event "Call back the client" --wake now
```

<div id="lobster-deterministic-workflows-with-approvals">
  ## Lobster: workflow deterministici con approvazioni
</div>

Lobster è il runtime di workflow per **pipeline di strumenti a più fasi** che richiedono un&#39;esecuzione deterministica e approvazioni esplicite.
Usalo quando l&#39;attività va oltre il singolo turno di un agente e vuoi un workflow riprendibile con punti di controllo umani.

<div id="when-lobster-fits">
  ### Quando usare Lobster
</div>

* **Automazione multi-step**: hai bisogno di una pipeline fissa di chiamate agli strumenti, non di un singolo prompt una tantum.
* **Punti di approvazione**: le azioni con effetti collaterali devono essere sospese finché non approvi, quindi riprendere.
* **Esecuzioni riprendibili**: continua un workflow messo in pausa senza rieseguire i passaggi precedenti.

<div id="how-it-pairs-with-heartbeat-and-cron">
  ### Come si integra con heartbeat e cron
</div>

* **Heartbeat/cron** decidono *quando* viene eseguita un&#39;esecuzione.
* **Lobster** definisce *quali passaggi* vengono eseguiti una volta che l&#39;esecuzione è avviata.

Per i workflow pianificati, usa cron o heartbeat per attivare un turno dell&#39;agente che invoca Lobster.
Per i workflow ad hoc, invoca direttamente Lobster.

<div id="operational-notes-from-the-code">
  ### Note operative (dal codice)
</div>

* Lobster viene eseguito come **sottoprocesso locale** (`lobster` CLI) in modalità strumento e restituisce un **contenitore JSON**.
* Se lo strumento restituisce `needs_approval`, riprendi passando un `resumeToken` e il flag `approve`.
* Lo strumento è un **plugin opzionale**; abilitalo aggiungendolo tramite `tools.alsoAllow: ["lobster"]` (consigliato).
* Se passi `lobsterPath`, deve essere un **percorso assoluto**.

Consulta [Lobster](/it/tools/lobster) per l&#39;utilizzo completo e gli esempi.

<div id="main-session-vs-isolated-session">
  ## Sessione principale vs sessione isolata
</div>

Sia heartbeat che cron possono interagire con la sessione principale, ma in modo diverso:

| | Heartbeat | Cron (principale) | Cron (isolata) |
|---|---|---|---|
| Sessione | Principale | Principale (tramite evento di sistema) | `cron:<jobId>` |
| Cronologia | Condivisa | Condivisa | Nuova a ogni esecuzione |
| Contesto | Completo | Completo | Nessuno (parte da zero) |
| Modello | Modello della sessione principale | Modello della sessione principale | Può essere sovrascritto |
| Output | Recapitato se non è `HEARTBEAT_OK` | Prompt di heartbeat + evento | Riepilogo pubblicato nella sessione principale |

<div id="when-to-use-main-session-cron">
  ### Quando usare il cron della sessione principale
</div>

Usa `--session main` con `--system-event` quando vuoi:

* Che il promemoria/evento compaia nel contesto della sessione principale
* Che l&#39;agente lo gestisca durante il prossimo heartbeat con il contesto completo
* Senza un&#39;esecuzione isolata separata

```bash
openclaw cron add \
  --name "Check project" \
  --every "4h" \
  --session main \
  --system-event "Time for a project health check" \
  --wake now
```

<div id="when-to-use-isolated-cron">
  ### Quando usare cron isolato
</div>

Usa `--session isolated` quando vuoi:

* Un ambiente pulito senza contesto precedente
* Impostazioni diverse per il modello o il ragionamento
* Output inviato direttamente a un canale (il riepilogo viene comunque pubblicato sul canale principale per impostazione predefinita)
* Una cronologia che non intasa la sessione principale

```bash
openclaw cron add \
  --name "Deep analysis" \
  --cron "0 6 * * 0" \
  --session isolated \
  --message "Weekly codebase analysis..." \
  --model opus \
  --thinking high \
  --deliver
```

<div id="cost-considerations">
  ## Considerazioni sui costi
</div>

| Meccanismo | Profilo di costo |
|-----------|------------------|
| Heartbeat | Un turno ogni N minuti; cresce in funzione della dimensione di HEARTBEAT.md |
| Cron (main) | Aggiunge un evento al prossimo heartbeat (nessun turno isolato) |
| Cron (isolated) | Turno completo dell&#39;agente per ogni job; puoi usare un modello più economico |

**Suggerimenti**:

* Mantieni `HEARTBEAT.md` di piccole dimensioni per ridurre al minimo l&#39;overhead di token.
* Accorpa controlli simili nell&#39;heartbeat invece di usare più job cron separati.
* Usa `target: "none"` per l&#39;heartbeat se vuoi solo l&#39;elaborazione interna.
* Usa cron isolato con un modello più economico per le attività di routine.

<div id="related">
  ## Correlati
</div>

* [Heartbeat](/it/gateway/heartbeat) - configurazione completa di heartbeat
* [Cron jobs](/it/automation/cron-jobs) - riferimento completo per CLI e API cron
* [System](/it/cli/system) - eventi di sistema + controlli di heartbeat