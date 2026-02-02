---
title: OpenClaw
summary: "Guida completa end-to-end per usare OpenClaw come assistente personale, con avvertenze sulla sicurezza"
read_when:
  - Durante l'onboarding di una nuova istanza di assistente
  - Durante la revisione delle implicazioni relative a sicurezza e permessi
---

<div id="building-a-personal-assistant-with-openclaw">
  # Creare un assistente personale con OpenClaw
</div>

OpenClaw è un Gateway WhatsApp + Telegram + Discord + iMessage per agenti **Pi**. I plugin permettono di integrare anche Mattermost. Questa guida illustra la configurazione di un &quot;assistente personale&quot;: un numero WhatsApp dedicato che si comporta come il tuo agente sempre attivo.

<div id="safety-first">
  ## ⚠️ Prima la sicurezza
</div>

Stai mettendo un agente nella condizione di:

* eseguire comandi sulla tua macchina (a seconda della configurazione degli strumenti su Pi)
* leggere/scrivere file nel tuo spazio di lavoro
* inviare messaggi verso l’esterno tramite WhatsApp/Telegram/Discord/Mattermost (plugin)

Parti in modo conservativo:

* Imposta sempre `channels.whatsapp.allowFrom` (non eseguire mai una configurazione aperta a chiunque sul tuo Mac personale).
* Usa un numero WhatsApp dedicato per l’assistente.
* Il valore predefinito per gli heartbeat è ora 30 minuti. Disabilitali finché non ti fidi della configurazione impostando `agents.defaults.heartbeat.every: "0m"`.

<div id="prerequisites">
  ## Prerequisiti
</div>

* Node **22+**
* OpenClaw disponibile nel PATH (consigliata l&#39;installazione globale)
* Un secondo numero di telefono (SIM/eSIM/ricaricabile) per l&#39;assistente

```bash
npm install -g openclaw@latest
# oppure: pnpm add -g openclaw@latest
```

Dal telefono sorgente (sviluppo):

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm ui:build # installa automaticamente le dipendenze della UI alla prima esecuzione
pnpm build
pnpm link --global
```

<div id="the-two-phone-setup-recommended">
  ## La configurazione a due telefoni (consigliata)
</div>

Questo è ciò che vuoi ottenere:

```
Your Phone (personal)          Second Phone (assistant)
┌─────────────────┐           ┌─────────────────┐
│  Your WhatsApp  │  ──────▶  │  Assistant WA   │
│  +1-555-YOU     │  message  │  +1-555-ASSIST  │
└─────────────────┘           └────────┬────────┘
                                       │ linked via QR
                                       ▼
                              ┌─────────────────┐
                              │  Your Mac       │
                              │  (openclaw)      │
                              │    Pi agent     │
                              └─────────────────┘
```

Se colleghi il tuo account WhatsApp personale a OpenClaw, ogni messaggio che ti arriva diventa un “input per l’agente”. Di solito non è quello che vuoi.

<div id="5-minute-quick-start">
  ## Avvio rapido di 5 minuti
</div>

1. Abbina WhatsApp Web (verrà mostrato un codice QR; scansionalo con il telefono dell&#39;assistente):

```bash
openclaw channels login
```

2. Avvia il Gateway (lascialo attivo):

```bash
openclaw gateway --port 18789
```

3. Aggiungi una configurazione minima in `~/.openclaw/openclaw.json`:

```json5
{
  channels: { whatsapp: { allowFrom: ["+15555550123"] } }
}
```

Ora invia un messaggio al numero dell&#39;assistente dal tuo telefono presente nella lista di autorizzati.

Al termine dell&#39;onboarding, apriamo automaticamente la dashboard con il tuo token del Gateway e mostriamo il link tokenizzato. Per riaprirla in seguito: `openclaw dashboard`.

<div id="give-the-agent-a-workspace-agents">
  ## Dai all&#39;agente uno spazio di lavoro (AGENTS)
</div>

OpenClaw legge le istruzioni operative e la “memoria” dalla sua directory dello spazio di lavoro.

Per impostazione predefinita, OpenClaw usa `~/.openclaw/workspace` come spazio di lavoro dell&#39;agente e lo creerà (insieme ai file iniziali `AGENTS.md`, `SOUL.md`, `TOOLS.md`, `IDENTITY.md`, `USER.md`) automaticamente durante la configurazione/prima esecuzione dell&#39;agente. `BOOTSTRAP.md` viene creato solo quando lo spazio di lavoro è appena stato creato (non dovrebbe ricomparire dopo che lo hai eliminato).

Suggerimento: tratta questa cartella come la “memoria” di OpenClaw e trasformala in un repository git (idealmente privato) in modo che i tuoi file `AGENTS.md` + file di memoria vengano sottoposti a backup. Se git è installato, gli spazi di lavoro appena creati vengono inizializzati automaticamente.

```bash
openclaw setup
```

Layout completo dello spazio di lavoro + guida al backup: [Spazio di lavoro dell&#39;Agente](/it/concepts/agent-workspace)
Flusso di lavoro della memoria: [Memoria](/it/concepts/memory)

Facoltativo: puoi scegliere uno spazio di lavoro diverso con `agents.defaults.workspace` (supporta `~`).

```json5
{
  agent: {
    workspace: "~/.openclaw/workspace"
  }
}
```

Se gestisci già i file del tuo spazio di lavoro da un repository, puoi disabilitare del tutto la creazione dei file di bootstrap:

```json5
{
  agent: {
    skipBootstrap: true
  }
}
```

<div id="the-config-that-turns-it-into-an-assistant">
  ## La configurazione che lo trasforma in “un assistente”
</div>

OpenClaw è preconfigurato come un buon assistente, ma in genere vorrai personalizzare:

* persona/istruzioni in `SOUL.md`
* impostazioni predefinite del “thinking” (se vuoi)
* heartbeat (una volta che ti fidi dell’assistente)

Esempio:

```json5
{
  logging: { level: "info" },
  agent: {
    model: "anthropic/claude-opus-4-5",
    workspace: "~/.openclaw/workspace",
    thinkingDefault: "high",
    timeoutSeconds: 1800,
    // Inizia con 0; abilita in seguito.
    heartbeat: { every: "0m" }
  },
  channels: {
    whatsapp: {
      allowFrom: ["+15555550123"],
      groups: {
        "*": { requireMention: true }
      }
    }
  },
  routing: {
    groupChat: {
      mentionPatterns: ["@openclaw", "openclaw"]
    }
  },
  session: {
    scope: "per-sender",
    resetTriggers: ["/new", "/reset"],
    reset: {
      mode: "daily",
      atHour: 4,
      idleMinutes: 10080
    }
  }
}
```

<div id="sessions-and-memory">
  ## Sessioni e memoria
</div>

* File di sessione: `~/.openclaw/agents/<agentId>/sessions/{{SessionId}}.jsonl`
* Metadati della sessione (utilizzo dei token, ultimo instradamento, ecc.): `~/.openclaw/agents/<agentId>/sessions/sessions.json` (legacy: `~/.openclaw/sessions/sessions.json`)
* `/new` o `/reset` avvia una nuova sessione per quella chat (configurabile tramite `resetTriggers`). Se inviato da solo, l&#39;agente risponde con un breve messaggio di saluto per confermare il reset.
* `/compact [instructions]` compatta il contesto della sessione e indica il budget di contesto ancora disponibile.

<div id="heartbeats-proactive-mode">
  ## Heartbeats (modalità proattiva)
</div>

Per impostazione predefinita, OpenClaw esegue un heartbeat ogni 30 minuti con il prompt:
`Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.`
Imposta `agents.defaults.heartbeat.every: "0m"` per disattivarlo.

* Se `HEARTBEAT.md` esiste ma è sostanzialmente vuoto (solo righe vuote e intestazioni markdown come `# Heading`), OpenClaw salta l&#39;esecuzione del heartbeat per risparmiare chiamate API.
* Se il file manca, l&#39;heartbeat viene comunque eseguito e il modello decide cosa fare.
* Se l&#39;agente risponde con `HEARTBEAT_OK` (facoltativamente con un breve padding; vedi `agents.defaults.heartbeat.ackMaxChars`), OpenClaw sopprime l&#39;invio in uscita per quell&#39;heartbeat.
* I heartbeat eseguono turni completi dell&#39;agente — intervalli più brevi consumano più token.

```json5
{
  agent: {
    heartbeat: { every: "30m" }
  }
}
```

<div id="media-in-and-out">
  ## Media in ingresso e in uscita
</div>

Gli allegati in ingresso (immagini/audio/documenti) possono essere passati al tuo comando tramite i template:

* `{{MediaPath}}` (percorso del file temporaneo locale)
* `{{MediaUrl}}` (pseudo-URL)
* `{{Transcript}}` (se la trascrizione audio è abilitata)

Allegati in uscita dall&#39;agente: includi `MEDIA:<path-or-url>` su una riga dedicata (senza spazi). Esempio:

```
Ecco lo screenshot.
MEDIA:/tmp/screenshot.png
```

OpenClaw li estrae e li invia come contenuti multimediali insieme al testo.

<div id="operations-checklist">
  ## Checklist delle operazioni
</div>

```bash
openclaw status          # stato locale (credenziali, sessioni, eventi in coda)
openclaw status --all    # diagnosi completa (sola lettura, incollabile)
openclaw status --deep   # aggiunge probe di salute del Gateway (Telegram + Discord)
openclaw health --json   # snapshot di salute del Gateway (WS)
```

I log si trovano in `/tmp/openclaw/` (nome file predefinito: `openclaw-YYYY-MM-DD.log`).

<div id="next-steps">
  ## Prossimi passi
</div>

* WebChat: [WebChat](/it/web/webchat)
* Operazioni del Gateway: [Gateway runbook](/it/gateway)
* Cron + riattivazioni: [Cron jobs](/it/automation/cron-jobs)
* Companion nella barra dei menu di macOS: [OpenClaw macOS app](/it/platforms/macos)
* App nodo per iOS: [iOS app](/it/platforms/ios)
* App nodo per Android: [Android app](/it/platforms/android)
* Stato di Windows: [Windows (WSL2)](/it/platforms/windows)
* Stato di Linux: [Linux app](/it/platforms/linux)
* Sicurezza: [Sicurezza](/it/gateway/security)