---
title: Stato di salute
summary: "Passaggi per verificare lo stato di salute della connettività del canale"
read_when:
  - Diagnosi dello stato di salute del canale WhatsApp
---

<div id="health-checks-cli">
  # Verifiche di funzionamento (CLI)
</div>

Breve guida per verificare la connettività dei canali senza dover andare a tentativi.

<div id="quick-checks">
  ## Verifiche rapide
</div>

- `openclaw status` — riepilogo locale: raggiungibilità/modalità del Gateway, suggerimento di aggiornamento, anzianità dell'autenticazione dei canali collegati, sessioni + attività recente.
- `openclaw status --all` — diagnosi locale completa (sola lettura, con colori, output che puoi incollare in sicurezza per il debug).
- `openclaw status --deep` — effettua anche verifiche sul Gateway in esecuzione (per canale, quando supportato).
- `openclaw health --json` — richiede al Gateway in esecuzione uno snapshot completo dello stato di salute (solo WS; nessun socket Baileys diretto).
- Invia `/status` come messaggio separato in WhatsApp/WebChat per ottenere una risposta di stato senza invocare l'agente.
- Log: esegui `tail` su `/tmp/openclaw/openclaw-*.log` e filtra per `web-heartbeat`, `web-reconnect`, `web-auto-reply`, `web-inbound`.

<div id="deep-diagnostics">
  ## Diagnostica approfondita
</div>

- Credenziali su disco: `ls -l ~/.openclaw/credentials/whatsapp/<accountId>/creds.json` (il campo `mtime` dovrebbe essere recente).
- Archivio delle sessioni: `ls -l ~/.openclaw/agents/<agentId>/sessions/sessions.json` (il percorso può essere sovrascritto nella configurazione). Il conteggio e i destinatari recenti sono esposti tramite `status`.
- Flusso di ricollegamento: `openclaw channels logout && openclaw channels login --verbose` quando nei log compaiono i codici di stato 409–515 o `loggedOut`. (Nota: il flusso di login con QR si riavvia automaticamente una volta per lo stato 515 dopo l'abbinamento.)

<div id="when-something-fails">
  ## Quando qualcosa va storto
</div>

- `logged out` o codice di stato 409–515 → ricollega eseguendo `openclaw channels logout` e poi `openclaw channels login`.
- Gateway non raggiungibile → avvialo: `openclaw gateway --port 18789` (usa `--force` se la porta è occupata).
- Nessun messaggio in ingresso → verifica che il telefono collegato sia online e che il mittente sia autorizzato (`channels.whatsapp.allowFrom`); per le chat di gruppo, assicurati che la lista di autorizzati e le regole di menzione siano coerenti (`channels.whatsapp.groups`, `agents.list[].groupChat.mentionPatterns`).

<div id="dedicated-health-command">
  ## Comando dedicato "health"
</div>

`openclaw health --json` richiede al Gateway in esecuzione uno snapshot di stato (senza aprire socket di canale diretti dalla CLI). Riporta, quando disponibili, l’anzianità delle credenziali/autenticazioni collegate, i riepiloghi dei controlli (probe) per canale, il riepilogo dell’archivio delle sessioni e la durata del controllo. Termina con un codice di uscita diverso da zero se il Gateway non è raggiungibile o se il controllo fallisce/va in timeout. Usa `--timeout <ms>` per modificare il valore predefinito di 10s.