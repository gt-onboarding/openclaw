---
title: Configurazione
summary: "Guida alla configurazione: mantieni la tua configurazione OpenClaw personalizzata e sempre aggiornata"
read_when:
  - Stai configurando una nuova macchina
  - Vuoi avere sempre l’ultima versione e le funzionalità più recenti senza compromettere la tua configurazione personale
---

<div id="setup">
  # Configurazione
</div>

Ultimo aggiornamento: 2026-01-01

<div id="tldr">
  ## TL;DR
</div>

* **La personalizzazione risiede fuori dal repo:** `~/.openclaw/workspace` (spazio di lavoro) + `~/.openclaw/openclaw.json` (configurazione).
* **Workflow stabile:** installa l&#39;app macOS; lascia che esegua il Gateway incluso.
* **Workflow all&#39;avanguardia:** esegui tu il Gateway tramite `pnpm gateway:watch`, poi lascia che l&#39;app macOS si colleghi in modalità Local.

<div id="prereqs-from-source">
  ## Prerequisiti (dalla sorgente)
</div>

* Node `>=22`
* `pnpm`
* Docker (opzionale; solo per configurazione/e2e containerizzata — vedi [Docker](/it/install/docker))

<div id="tailoring-strategy-so-updates-dont-hurt">
  ## Strategia di personalizzazione (per non soffrire con gli aggiornamenti)
</div>

Se vuoi qualcosa “personalizzato al 100% su di te” *e* aggiornamenti facili, mantieni la personalizzazione in:

* **Config:** `~/.openclaw/openclaw.json` (simile a JSON/JSON5)
* **Spazio di lavoro:** `~/.openclaw/workspace` (abilità, prompt, memorie; rendilo un repository git privato)

Esegui il bootstrap una sola volta:

```bash
openclaw setup
```

Dall&#39;interno di questa repository, usa il punto di ingresso locale della CLI:

```bash
openclaw setup
```

Se non hai ancora effettuato un&#39;installazione globale, esegui `pnpm openclaw setup`.

<div id="stable-workflow-macos-app-first">
  ## Workflow stabile (prima l&#39;app macOS)
</div>

1. Installa e avvia **OpenClaw.app** (icona nella barra dei menu).
2. Completa la checklist iniziale di onboarding/autorizzazioni (prompt TCC).
3. Verifica che il Gateway sia impostato su **Local** e in esecuzione (lo gestisce l&#39;app).
4. Collega le superfici di interazione (ad esempio: WhatsApp):

```bash
openclaw channels login
```

5. Verifica rapida:

```bash
openclaw health
```

Se l&#39;onboarding non è disponibile nella tua build:

* Esegui `openclaw setup`, poi `openclaw channels login` e quindi avvia manualmente il Gateway (`openclaw gateway`).

<div id="bleeding-edge-workflow-gateway-in-a-terminal">
  ## Workflow all&#39;avanguardia (Gateway in un terminale)
</div>

Obiettivo: lavorare sul Gateway scritto in TypeScript, ottenere l&#39;hot reload e mantenere collegata l&#39;UI dell&#39;app macOS.

<div id="0-optional-run-the-macos-app-from-source-too">
  ### 0) (Opzionale) Esegui anche l&#39;app macOS dal codice sorgente
</div>

Se vuoi avere anche l&#39;app macOS nella versione più avanzata (bleeding edge):

```bash
./scripts/restart-mac.sh
```

<div id="1-start-the-dev-gateway">
  ### 1) Avvia il Gateway in modalità sviluppo
</div>

```bash
pnpm install
pnpm gateway:watch
```

`gateway:watch` esegue il Gateway in modalità watch e si ricarica al rilevamento di modifiche ai file TypeScript.

<div id="2-point-the-macos-app-at-your-running-gateway">
  ### 2) Configura l&#39;app macOS perché punti al Gateway in esecuzione
</div>

In **OpenClaw.app**:

* Modalità di connessione: **Locale**
  L&#39;app si collegherà al Gateway in esecuzione sulla porta configurata.

<div id="3-verify">
  ### 3) Verifica
</div>

* Lo stato del Gateway nell’app dovrebbe mostrare **“Using existing gateway …”**
* Oppure tramite CLI:

```bash
openclaw health
```

<div id="common-footguns">
  ### Errori comuni
</div>

* **Porta sbagliata:** il Gateway WS usa per impostazione predefinita `ws://127.0.0.1:18789`; mantieni app e CLI configurate sulla stessa porta.
* **Dove risiede lo stato:**
  * Credenziali: `~/.openclaw/credentials/`
  * Sessioni: `~/.openclaw/agents/<agentId>/sessions/`
  * Log: `/tmp/openclaw/`

<div id="credential-storage-map">
  ## Mappa di archiviazione delle credenziali
</div>

Usa questa sezione quando esegui il debug dell&#39;autenticazione o decidi cosa includere nel backup:

* **WhatsApp**: `~/.openclaw/credentials/whatsapp/<accountId>/creds.json`
* **Token bot Telegram**: config/env oppure `channels.telegram.tokenFile`
* **Token bot Discord**: config/env (file di token non ancora supportato)
* **Token Slack**: config/env (`channels.slack.*`)
* **Liste di autorizzati per l&#39;abbinamento**: `~/.openclaw/credentials/<channel>-allowFrom.json`
* **Profili di autenticazione dei modelli**: `~/.openclaw/agents/<agentId>/agent/auth-profiles.json`
* **Importazione OAuth legacy**: `~/.openclaw/credentials/oauth.json`

Ulteriori dettagli: [Security](/it/gateway/security#credential-storage-map).

<div id="updating-without-wrecking-your-setup">
  ## Aggiornare (senza distruggere la configurazione)
</div>

* Considera `~/.openclaw/workspace` e `~/.openclaw/` come “roba tua”: non inserire prompt/configurazioni personali nella repo `openclaw`.
* Per aggiornare il sorgente: `git pull` + `pnpm install` (quando è cambiato il file di lock) + continua a usare `pnpm gateway:watch`.

<div id="linux-systemd-user-service">
  ## Linux (servizio utente systemd)
</div>

Le installazioni Linux usano un servizio **utente** di systemd. Per impostazione predefinita, systemd ferma i servizi utente al logout o in caso di inattività, interrompendo così il Gateway. La procedura di onboarding tenta di abilitare automaticamente il lingering (potrebbe richiedere sudo). Se è ancora disattivato, esegui:

```bash
sudo loginctl enable-linger $USER
```

Per i server sempre attivi o multi‑utente, valuta un servizio **di sistema** invece di un
servizio utente (non è necessario il lingering). Vedi il [Gateway runbook](/it/gateway) per le note su systemd.

<div id="related-docs">
  ## Documenti correlati
</div>

* [Runbook del Gateway](/it/gateway) (flag, supervisione, porte)
* [Configurazione del Gateway](/it/gateway/configuration) (schema di configurazione ed esempi)
* [Discord](/it/channels/discord) e [Telegram](/it/channels/telegram) (reply tags e impostazioni replyToMode)
* [Configurazione dell&#39;assistente OpenClaw](/it/start/openclaw)
* [App macOS](/it/platforms/macos) (ciclo di vita del Gateway)