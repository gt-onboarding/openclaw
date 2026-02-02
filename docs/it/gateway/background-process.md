---
title: Processo in background
summary: "Esecuzione di exec in background e gestione dei processi"
read_when:
  - Aggiunta o modifica del comportamento di exec in background
  - Debug di attività exec di lunga durata
---

<div id="background-exec-process-tool">
  # Exec in background + strumento Process
</div>

OpenClaw esegue comandi di shell tramite lo strumento `exec` e mantiene in memoria le attività di lunga esecuzione. Lo strumento `process` gestisce quelle sessioni in background.

<div id="exec-tool">
  ## strumento exec
</div>

Parametri principali:

- `command` (obbligatorio)
- `yieldMs` (predefinito 10000): passa automaticamente in background dopo questo intervallo
- `background` (bool): va immediatamente in background
- `timeout` (secondi, predefinito 1800): termina il processo dopo questo timeout
- `elevated` (bool): esegui sull'host se la modalità elevata è abilitata/consentita
- Ti serve un TTY reale? Imposta `pty: true`.
- `workdir`, `env`

Comportamento:

- Le esecuzioni in foreground restituiscono direttamente l'output.
- Quando viene messo in background (esplicitamente o per timeout), lo strumento restituisce `status: "running"` + `sessionId` e un breve spezzone finale dell'output.
- L'output viene mantenuto in memoria finché la sessione non viene interrogata (poll) o svuotata.
- Se lo strumento `process` non è consentito, `exec` viene eseguito in modo sincrono e ignora `yieldMs`/`background`.

<div id="child-process-bridging">
  ## Collegamento dei processi figli
</div>

Quando avvii processi figli di lunga durata al di fuori degli strumenti exec/process (ad esempio, riavvii della CLI o helper del Gateway), collega l'helper di bridge per il processo figlio in modo che i segnali di terminazione vengano inoltrati e i listener vengano scollegati in caso di uscita o errore. Questo evita processi orfani su systemd e mantiene il comportamento di arresto coerente tra le diverse piattaforme.

Override tramite variabili d'ambiente:

- `PI_BASH_YIELD_MS`: intervallo di yield predefinito (ms)
- `PI_BASH_MAX_OUTPUT_CHARS`: limite di output in memoria (caratteri)
- `OPENCLAW_BASH_PENDING_MAX_OUTPUT_CHARS`: limite di stdout/stderr in sospeso per flusso (caratteri)
- `PI_BASH_JOB_TTL_MS`: TTL per le sessioni terminate (ms, limitato a 1m–3h)

Configurazione (consigliata):

- `tools.exec.backgroundMs` (predefinito 10000)
- `tools.exec.timeoutSec` (predefinito 1800)
- `tools.exec.cleanupMs` (predefinito 1800000)
- `tools.exec.notifyOnExit` (predefinito true): accoda un evento di sistema e richiede un heartbeat quando un exec in background termina.

<div id="process-tool">
  ## process tool
</div>

Azioni:

- `list`: sessioni in esecuzione + terminate
- `poll`: estrae il nuovo output per una sessione (riporta anche lo stato di uscita)
- `log`: legge l'output aggregato (supporta `offset` + `limit`)
- `write`: invia su stdin (`data`, `eof` opzionale)
- `kill`: termina una sessione in background
- `clear`: rimuove una sessione terminata dalla memoria
- `remove`: esegue `kill` se in esecuzione, altrimenti `clear` se terminata

Note:

- Solo le sessioni in background sono elencate e mantenute in memoria.
- Le sessioni vengono perse al riavvio del processo (nessuna persistenza su disco).
- I log delle sessioni vengono salvati nella cronologia della chat solo se esegui `process poll/log` e il risultato dello strumento viene registrato.
- `process` ha scope per agente; vede solo le sessioni avviate da quell'agente.
- `process list` include un `name` derivato (verbo del comando + target) per consultazioni rapide.
- `process log` usa `offset`/`limit` basati sulle righe (ometti `offset` per prendere le ultime N righe).

<div id="examples">
  ## Esempi
</div>

Esegui un&#39;attività di lunga durata ed effettua il polling più tardi:

```json
{"tool": "exec", "command": "sleep 5 && echo done", "yieldMs": 1000}
```

```json
{"tool": "process", "action": "poll", "sessionId": "<id>"}
```

Avvia subito in background:

```json
{"tool": "exec", "command": "npm run build", "background": true}
```

Invia lo stdin:

```json
{"tool": "process", "action": "write", "sessionId": "<id>", "data": "y\n"}
```
