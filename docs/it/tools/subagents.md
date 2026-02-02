---
title: Sottoagenti
summary: "Sottoagenti: avvio di esecuzioni di agenti isolate che inviano i risultati alla chat che le ha richieste"
read_when:
  - Vuoi eseguire lavoro in background/parallelo tramite l'agente
  - Stai modificando sessions_spawn o la policy degli strumenti dei sottoagenti
---

<div id="sub-agents">
  # Sottoagenti
</div>

I sottoagenti sono esecuzioni di un agente in background generate da un'esecuzione di agente esistente. Vengono eseguiti nella propria sessione (`agent:<agentId>:subagent:<uuid>`) e, al termine, **annunciano** il loro risultato al canale di chat che li ha richiesti.

<div id="slash-command">
  ## Comando slash
</div>

Usa `/subagents` per ispezionare o controllare le esecuzioni dei sub-agenti per la **sessione corrente**:

- `/subagents list`
- `/subagents stop <id|#|all>`
- `/subagents log <id|#> [limit] [tools]`
- `/subagents info <id|#>`
- `/subagents send <id|#> <message>`

`/subagents info` mostra i metadati dell'esecuzione (stato, timestamp, id di sessione, percorso della trascrizione, operazioni di cleanup).

Obiettivi principali:

- Parallelizzare il lavoro di “ricerca / attività lunghe / strumenti lenti” senza bloccare l'esecuzione principale.
- Mantenere i sub-agenti isolati per impostazione predefinita (separazione delle sessioni + sandbox opzionale).
- Mantenere la superficie dei tool difficile da usare in modo improprio: i sub-agenti **non** ricevono gli strumenti della sessione per impostazione predefinita.
- Evitare il fan-out annidato: i sub-agenti non possono generare altri sub-agenti.

Nota sui costi: ogni sub-agente ha il **proprio** contesto e consumo di token. Per attività pesanti o ripetitive,
imposta un modello più economico per i sub-agenti e mantieni il tuo agente principale su un modello di qualità superiore.
Puoi configurarlo tramite `agents.defaults.subagents.model` o con override specifici per agente.

<div id="tool">
  ## Strumento
</div>

Usa `sessions_spawn`:

- Avvia un'esecuzione di sottoagente (`deliver: false`, corsia globale: `subagent`)
- Poi esegue un passaggio di annuncio e pubblica la risposta di annuncio nel canale chat del richiedente
- Modello predefinito: eredita quello del chiamante, a meno che tu non imposti `agents.defaults.subagents.model` (o per agente `agents.list[].subagents.model`); un `sessions_spawn.model` esplicito ha comunque la precedenza.

Parametri dello strumento:

- `task` (obbligatorio)
- `label?` (opzionale)
- `agentId?` (opzionale; genera sotto un altro id di agente, se consentito)
- `model?` (opzionale; sovrascrive il modello del sottoagente; i valori non validi vengono ignorati e il sottoagente viene eseguito con il modello predefinito, con un avviso nel risultato dello strumento)
- `thinking?` (opzionale; sovrascrive il livello di ragionamento per l'esecuzione del sottoagente)
- `runTimeoutSeconds?` (predefinito `0`; quando impostato, l'esecuzione del sottoagente viene interrotta dopo N secondi)
- `cleanup?` (`delete|keep`, predefinito `keep`)

Lista di autorizzati:

- `agents.list[].subagents.allowAgents`: elenco di id di agente che possono essere selezionati tramite `agentId` (`["*"]` per consentire qualsiasi agente). Predefinito: solo l'agente richiedente.

Discovery:

- Usa `agents_list` per vedere quali id di agente sono attualmente consentiti per `sessions_spawn`.

Auto-archiviazione:

- Le sessioni dei sottoagenti vengono archiviate automaticamente dopo `agents.defaults.subagents.archiveAfterMinutes` (predefinito: 60).
- L'archiviazione usa `sessions.delete` e rinomina la trascrizione in `*.deleted.<timestamp>` (stessa cartella).
- `cleanup: "delete"` archivia immediatamente dopo l'annuncio (mantiene comunque la trascrizione tramite rinomina).
- L'auto-archiviazione è su base best-effort; i timer in sospeso vengono persi se il Gateway si riavvia.
- `runTimeoutSeconds` **non** esegue l'auto-archiviazione; interrompe solo l'esecuzione. La sessione rimane fino all'auto-archiviazione.

<div id="authentication">
  ## Autenticazione
</div>

L'autenticazione dei sotto-agenti è risolta tramite **ID dell'agente**, non in base al tipo di sessione:

- La chiave di sessione del sotto-agente è `agent:<agentId>:subagent:<uuid>`.
- L'archivio di autenticazione (`auth store`) viene caricato dalla `agentDir` di quell'agente.
- I profili di autenticazione dell'agente principale vengono fusi come **fallback**; i profili dell'agente hanno la precedenza su quelli principali in caso di conflitto.

Nota: la fusione è additiva, quindi i profili principali sono sempre disponibili come fallback. Un'autenticazione completamente isolata per singolo agente non è ancora supportata.

<div id="announce">
  ## Announce
</div>

I sotto-agenti riportano i risultati tramite uno step di announce:

- Lo step di announce viene eseguito all'interno della sessione del sotto-agente (non nella sessione del richiedente).
- Se il sotto-agente risponde esattamente con `ANNOUNCE_SKIP`, non viene pubblicato nulla.
- Altrimenti, la risposta di announce viene pubblicata nel canale chat del richiedente tramite una chiamata `agent` di follow-up (`deliver=true`).
- Le risposte di announce preservano l'instradamento thread/argomento quando disponibile (thread Slack, topic Telegram, thread Matrix).
- I messaggi di announce vengono normalizzati in un template stabile:
  - `Status:` derivato dall'esito dell'esecuzione (`success`, `error`, `timeout` o `unknown`).
  - `Result:` il contenuto di riepilogo dello step di announce (oppure `(not available)` se mancante).
  - `Notes:` dettagli di errore e altro contesto utile.
- `Status` non viene dedotto dall'output del modello; deriva dai segnali di esito del runtime.

I payload di announce includono una riga di statistiche alla fine (anche quando sono wrappati):

- Runtime (ad es. `runtime 5m12s`)
- Utilizzo di token (input/output/totale)
- Costo stimato quando è configurato il pricing del modello (`models.providers.*.models[].cost`)
- `sessionKey`, `sessionId` e percorso della trascrizione (così l'agente principale può recuperare la cronologia tramite `sessions_history` o ispezionare il file su disco)

<div id="tool-policy-sub-agent-tools">
  ## Criteri per gli strumenti (strumenti dei sottoagenti)
</div>

Per impostazione predefinita, i sottoagenti hanno accesso a **tutti gli strumenti tranne gli strumenti di sessione**:

* `sessions_list`
* `sessions_history`
* `sessions_send`
* `sessions_spawn`

Puoi eseguire l&#39;override tramite la configurazione:

```json5
{
  agents: {
    defaults: {
      subagents: {
        maxConcurrent: 1
      }
    }
  },
  tools: {
    subagents: {
      tools: {
        // deny wins
        deny: ["gateway", "cron"],
        // se allow è impostato, diventa allow-only (deny ha comunque la precedenza)
        // allow: ["read", "exec", "process"]
      }
    }
  }
}
```


<div id="concurrency">
  ## Concorrenza
</div>

I sottoagenti usano una corsia di coda dedicata in-process:

- Nome corsia: `subagent`
- Concorrenza: `agents.defaults.subagents.maxConcurrent` (valore predefinito `8`)

<div id="stopping">
  ## Arresto
</div>

- Invia `/stop` nella chat del richiedente per interrompere la sessione del richiedente e arrestare tutte le esecuzioni di sotto-agenti attive avviate da quella sessione.

<div id="limitations">
  ## Limitazioni
</div>

- L'annuncio del sub-agente è **best-effort**. Se il Gateway si riavvia, il lavoro “announce back” in sospeso viene perso.
- I sub-agenti condividono comunque le stesse risorse di processo del Gateway; considera `maxConcurrent` una valvola di sicurezza.
- `sessions_spawn` è sempre non bloccante: restituisce immediatamente `{ status: "accepted", runId, childSessionKey }`.
- Il contesto del sub-agente inserisce solo `AGENTS.md` + `TOOLS.md` (nessun `SOUL.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md` o `BOOTSTRAP.md`).