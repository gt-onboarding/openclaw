---
title: Sistema
summary: "Riferimento CLI per `openclaw system` (eventi di sistema, heartbeat, presenza)"
read_when:
  - Vuoi mettere in coda un evento di sistema senza creare un cron job
  - Devi abilitare o disabilitare gli heartbeat
  - Vuoi esaminare le voci di presenza del sistema
---

<div id="openclaw-system">
  # `openclaw system`
</div>

Strumenti a livello di sistema per il Gateway: accoda eventi di sistema, gestisci gli heartbeat
e visualizza lo stato di presenza.

<div id="common-commands">
  ## Comandi più comuni
</div>

```bash
openclaw system event --text "Check for urgent follow-ups" --mode now
openclaw system heartbeat enable
openclaw system heartbeat last
openclaw system presence
```


<div id="system-event">
  ## `system event`
</div>

Metti in coda un evento di sistema nella sessione **principale**. Il prossimo heartbeat lo inserirà
come riga `System:` nel prompt. Usa `--mode now` per attivare subito l'heartbeat;
`next-heartbeat` attende il prossimo tick pianificato.

Flag:

- `--text <text>`: testo obbligatorio dell'evento di sistema.
- `--mode <mode>`: `now` o `next-heartbeat` (predefinito).
- `--json`: output in formato macchina.

<div id="system-heartbeat-lastenabledisable">
  ## `system heartbeat last|enable|disable`
</div>

Controlli di heartbeat:

- `last`: mostra l'ultimo evento di heartbeat.
- `enable`: riattiva gli heartbeat (usa questo se erano stati disattivati).
- `disable`: mette in pausa gli heartbeat.

Flags:

- `--json`: output leggibile dalla macchina.

<div id="system-presence">
  ## `system presence`
</div>

Elenca le voci di presenza di sistema attuali note al Gateway (nodi,
istanze e righe di stato simili).

Flag:

- `--json`: output in formato macchina.

<div id="notes">
  ## Note
</div>

- Richiede un Gateway in esecuzione, raggiungibile con la configurazione attuale (locale o remota).
- Gli eventi di sistema sono effimeri e non vengono persisiti tra i riavvii.