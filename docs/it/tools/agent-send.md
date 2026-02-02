---
title: Invio Agente
summary: "Esecuzioni dirette del comando CLI `openclaw agent` (con invio opzionale)"
read_when:
  - Aggiunta o modifica dell'entrypoint CLI dell'agente
---

<div id="openclaw-agent-direct-agent-runs">
  # `openclaw agent` (esecuzioni dirette dell'agente)
</div>

`openclaw agent` esegue un singolo turno di un agente senza richiedere un messaggio di chat in ingresso.
Per impostazione predefinita passa **attraverso il Gateway**; aggiungi `--local` per forzare il runtime integrato sulla macchina corrente.

<div id="behavior">
  ## Comportamento
</div>

- Obbligatorio: `--message <text>`
- Selezione della sessione:
  - `--to <dest>` ricava la chiave di sessione (i target gruppo/canale preservano l'isolamento; le chat dirette confluiscono in `main`), **oppure**
  - `--session-id <id>` riutilizza una sessione esistente per ID, **oppure**
  - `--agent <id>` indirizza direttamente a un agente configurato (usa la chiave di sessione `main` di quell'agente)
- Esegue lo stesso runtime di agente incorporato usato per le normali risposte in ingresso.
- I flag thinking/verbose persistono nell'archivio delle sessioni.
- Output:
  - predefinito: stampa il testo della risposta (più le righe `MEDIA:<url>`)
  - `--json`: stampa il payload strutturato + metadati
- Consegna opzionale di risposta a un canale con `--deliver` + `--channel` (i formati di destinazione corrispondono a `openclaw message --target`).
- Usa `--reply-channel`/`--reply-to`/`--reply-account` per forzare la consegna senza modificare la sessione.

Se il Gateway non è raggiungibile, la CLI **ripiega** sull'esecuzione locale incorporata.

<div id="examples">
  ## Esempi
</div>

```bash
openclaw agent --to +15555550123 --message "status update"
openclaw agent --agent ops --message "Summarize logs"
openclaw agent --session-id 1234 --message "Summarize inbox" --thinking medium
openclaw agent --to +15555550123 --message "Trace logs" --verbose on --json
openclaw agent --to +15555550123 --message "Summon reply" --deliver
openclaw agent --agent ops --message "Generate report" --deliver --reply-channel slack --reply-to "#reports"
```


<div id="flags">
  ## Flag
</div>

- `--local`: esegue in locale (richiede che nella shell siano impostate le chiavi API del provider del modello)
- `--deliver`: invia la risposta al canale scelto
- `--channel`: canale di consegna (`whatsapp|telegram|discord|googlechat|slack|signal|imessage`, predefinito: `whatsapp`)
- `--reply-to`: sovrascrive la destinazione di consegna
- `--reply-channel`: sovrascrive il canale di consegna
- `--reply-account`: sovrascrive l'ID dell'account di consegna
- `--thinking <off|minimal|low|medium|high|xhigh>`: mantiene il livello di thinking (solo modelli GPT-5.2 + Codex)
- `--verbose <on|full|off>`: mantiene il livello di verbosità
- `--timeout <seconds>`: sovrascrive il timeout dell'agente
- `--json`: produce JSON strutturato