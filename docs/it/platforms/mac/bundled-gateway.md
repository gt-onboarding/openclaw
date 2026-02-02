---
title: Gateway in bundle
summary: "Runtime del Gateway su macOS (servizio launchd esterno)"
read_when:
  - Creazione del pacchetto di OpenClaw.app
  - Debug del servizio launchd del Gateway su macOS
  - Installazione della CLI del Gateway per macOS
---

<div id="gateway-on-macos-external-launchd">
  # Gateway su macOS (launchd esterno)
</div>

OpenClaw.app non include più Node/Bun né il runtime del Gateway. L'app per macOS
richiede un'installazione **esterna** della CLI `openclaw`, non avvia il Gateway come
processo figlio e gestisce un servizio launchd per‑utente per mantenere il Gateway
in esecuzione (oppure si connette a un Gateway locale esistente se ce n'è già uno in esecuzione).

<div id="install-the-cli-required-for-local-mode">
  ## Installa la CLI (necessaria per la modalità locale)
</div>

Ti serve Node.js 22 o superiore sul Mac, poi installa `openclaw` a livello globale:

```bash
npm install -g openclaw@<version>
```

Il pulsante **Install CLI** dell’app macOS esegue lo stesso processo tramite npm/pnpm (bun non è consigliato come runtime del Gateway).


<div id="launchd-gateway-as-launchagent">
  ## Launchd (Gateway come LaunchAgent)
</div>

Etichetta:

- `bot.molt.gateway` (oppure `bot.molt.<profile>`; i vecchi `com.openclaw.*` possono rimanere)

Percorso del plist (per utente):

- `~/Library/LaunchAgents/bot.molt.gateway.plist`
  (oppure `~/Library/LaunchAgents/bot.molt.<profile>.plist`)

Gestore:

- L'app macOS gestisce l'installazione/l'aggiornamento del LaunchAgent in modalità locale.
- Anche la CLI può installarlo: `openclaw gateway install`.

Comportamento:

- "OpenClaw Active" abilita/disabilita il LaunchAgent.
- La chiusura dell'app **non** arresta il Gateway (launchd lo mantiene attivo).
- Se il Gateway è già in esecuzione sulla porta configurata, l'app vi si collega
  invece di avviarne uno nuovo.

Logging:

- stdout/err di launchd: `/tmp/openclaw/openclaw-gateway.log`

<div id="version-compatibility">
  ## Compatibilità delle versioni
</div>

L'app macOS confronta la versione del Gateway con la propria. Se risultano incompatibili, aggiorna la CLI globale per allinearla alla versione dell'app.

<div id="smoke-check">
  ## Verifica di funzionamento
</div>

```bash
openclaw --version

OPENCLAW_SKIP_CHANNELS=1 \
OPENCLAW_SKIP_CANVAS_HOST=1 \
openclaw gateway --port 18999 --bind loopback
```

Poi:

```bash
openclaw gateway call health --url ws://127.0.0.1:18999 --timeout 3000
```
