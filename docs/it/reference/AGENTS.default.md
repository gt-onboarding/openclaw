---
title: AGENTS.default
summary: "Istruzioni predefinite per l'agente OpenClaw ed elenco delle abilità per la configurazione dell'assistente personale"
read_when:
  - Avvio di una nuova sessione dell'agente OpenClaw
  - Abilitazione o verifica delle abilità predefinite
---

<div id="agentsmd-openclaw-personal-assistant-default">
  # AGENTS.md — Assistente personale OpenClaw (impostazione predefinita)
</div>

<div id="first-run-recommended">
  ## Primo avvio (consigliato)
</div>

OpenClaw utilizza un’apposita directory di spazio di lavoro per l&#39;agente. Valore predefinito: `~/.openclaw/workspace` (configurabile tramite `agents.defaults.workspace`).

1. Crea lo spazio di lavoro (se non esiste ancora):

```bash
mkdir -p ~/.openclaw/workspace
```

2. Copia i modelli di spazio di lavoro predefiniti nello spazio di lavoro:

```bash
cp docs/reference/templates/AGENTS.md ~/.openclaw/workspace/AGENTS.md
cp docs/reference/templates/SOUL.md ~/.openclaw/workspace/SOUL.md
cp docs/reference/templates/TOOLS.md ~/.openclaw/workspace/TOOLS.md
```

3. Facoltativo: se vuoi usare l&#39;elenco delle skill dell&#39;assistente personale, sostituisci AGENTS.md con questo file:

```bash
cp docs/reference/AGENTS.default.md ~/.openclaw/workspace/AGENTS.md
```

4. Facoltativo: scegli uno spazio di lavoro diverso impostando `agents.defaults.workspace` (supporta `~`):

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } }
}
```


<div id="safety-defaults">
  ## Impostazioni di sicurezza predefinite
</div>

- Non inserire o elencare directory o segreti nella chat.
- Non eseguire comandi distruttivi a meno che non ti venga chiesto esplicitamente.
- Non inviare risposte parziali o in streaming a canali di messaggistica esterni (solo risposte finali).

<div id="session-start-required">
  ## Avvio della sessione (obbligatorio)
</div>

- Leggi `SOUL.md`, `USER.md`, `memory.md` e i contenuti di oggi e ieri in `memory/`.
- Esegui questa operazione prima di rispondere.

<div id="soul-required">
  ## Soul (obbligatorio)
</div>

- `SOUL.md` definisce identità, tono e limiti. Mantienilo aggiornato.
- Se modifichi `SOUL.md`, informa l'utente.
- Sei una nuova istanza a ogni sessione; la continuità risiede in questi file.

<div id="shared-spaces-recommended">
  ## Spazi condivisi (consigliato)
</div>

- Non sei la voce dell’utente; fai attenzione nelle chat di gruppo o nei canali pubblici.
- Non condividere dati personali, informazioni di contatto o note interne.

<div id="memory-system-recommended">
  ## Sistema di memoria (consigliato)
</div>

- Registro giornaliero: `memory/YYYY-MM-DD.md` (crea `memory/` se necessario).
- Memoria a lungo termine: `memory.md` per fatti, preferenze e decisioni durature.
- All'avvio della sessione, read oggi + ieri + `memory.md` se presente.
- Raccogli: decisioni, preferenze, vincoli, loop aperti (attività in sospeso).
- Evita segreti, a meno che non siano esplicitamente richiesti.

<div id="tools-skills">
  ## Strumenti e abilità
</div>

- Gli strumenti fanno parte delle abilità; quando ti serve, segui il file `SKILL.md` di ciascuna abilità.
- Tieni le note specifiche dell'ambiente in `TOOLS.md` (Note per le abilità).

<div id="backup-tip-recommended">
  ## Suggerimento per il backup (consigliato)
</div>

Se consideri questo spazio di lavoro come la “memoria” di Clawd, trasformalo in un repository Git (preferibilmente privato) così `AGENTS.md` e i tuoi file di memoria vengono salvati in backup.

```bash
cd ~/.openclaw/workspace
git init
git add AGENTS.md
git commit -m "Add Clawd workspace"
# Opzionale: aggiungi un remote privato e fai push
```


<div id="what-openclaw-does">
  ## Cosa fa OpenClaw
</div>

- Esegue il gateway WhatsApp + l'agente di coding su Pi in modo che l'assistente possa leggere/scrivere chat, recuperare contesto ed eseguire abilità tramite il Mac host.
- L'app macOS gestisce le autorizzazioni (registrazione dello schermo, notifiche, microfono) ed espone la CLI `openclaw` tramite il binario incluso.
- Le chat dirette confluiscono per impostazione predefinita nella sessione `main` dell'agente; i gruppi restano isolati come `agent:<agentId>:<channel>:group:<id>` (stanze/canali: `agent:<agentId>:<channel>:channel:<id>`); gli heartbeat mantengono attive le attività in background.

<div id="core-skills-enable-in-settings-skills">
  ## Abilità principali (attivale in Impostazioni → Abilità)
</div>

- **mcporter** — Runtime/CLI del server di strumenti per gestire backend di abilità esterne.
- **Peekaboo** — Screenshot veloci su macOS con analisi visiva opzionale tramite AI.
- **camsnap** — Acquisisci frame, clip o avvisi di movimento da telecamere di sicurezza RTSP/ONVIF.
- **oracle** — CLI di Agente compatibile con OpenAI, con replay delle sessioni e controllo del browser.
- **eightctl** — Controlla il tuo sonno dal terminale.
- **imsg** — Invia, read e trasmetti in streaming iMessage e SMS.
- **wacli** — CLI di WhatsApp: sincronizza, cerca, invia.
- **discord** — Azioni Discord: reazioni, sticker, sondaggi. Usa come destinazione `user:<id>` o `channel:<id>` (i semplici ID numerici sono ambigui).
- **gog** — CLI Suite Google: Gmail, Calendar, Drive, Contacts.
- **spotify-player** — Client Spotify da terminale per cercare/mettere in coda/controllare la riproduzione.
- **sag** — Voce ElevenLabs con UX in stile `say` su macOS; trasmette in streaming sugli altoparlanti per impostazione predefinita.
- **Sonos CLI** — Controlla gli altoparlanti Sonos (scoperta/stato/riproduzione/volume/raggruppamento) tramite script.
- **blucli** — Riproduci, raggruppa e automatizza i lettori BluOS tramite script.
- **OpenHue CLI** — Controllo dell’illuminazione Philips Hue per scene e automazioni.
- **OpenAI Whisper** — Riconoscimento vocale locale (speech-to-text) per dettatura rapida e trascrizioni di segreteria telefonica.
- **Gemini CLI** — Modelli Google Gemini dal terminale per Q&A veloci.
- **bird** — CLI X/Twitter per twittare, rispondere, leggere thread e cercare senza usare il browser.
- **agent-tools** — Toolkit di utilità per automazioni e script di supporto.

<div id="usage-notes">
  ## Note d'uso
</div>

- Preferisci la CLI `openclaw` per lo scripting; l'app macOS gestisce le autorizzazioni.
- Esegui le installazioni dalla scheda Skills; nasconde il pulsante se un binario è già presente.
- Mantieni gli heartbeat abilitati così l'assistente può programmare promemoria, monitorare le caselle di posta e attivare le acquisizioni dalla fotocamera.
- La Canvas UI funziona a schermo intero con overlay nativi. Evita di posizionare controlli critici negli angoli in alto a sinistra/destra o lungo il bordo inferiore; aggiungi margini espliciti nel layout e non fare affidamento sui safe-area insets.
- Per la verifica guidata dal browser, usa `openclaw browser` (tabs/status/screenshot) con il profilo Chrome gestito da OpenClaw.
- Per l'ispezione del DOM, usa `openclaw browser eval|query|dom|snapshot` (e `--json`/`--out` quando ti serve output adatto all'elaborazione automatica).
- Per le interazioni, usa `openclaw browser click|type|hover|drag|select|upload|press|wait|navigate|back|evaluate|run` (click/type richiedono riferimenti a snapshot; usa `evaluate` per i selettori CSS).