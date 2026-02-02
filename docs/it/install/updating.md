---
title: Aggiornamento
summary: "Aggiornare OpenClaw in modo sicuro (installazione globale o da sorgente) e strategia di rollback"
read_when:
  - Aggiornamento di OpenClaw
  - Qualcosa non funziona dopo un aggiornamento
---

<div id="updating">
  # Aggiornamento
</div>

OpenClaw si evolve rapidamente (pre “1.0”). Tratta gli aggiornamenti come il deploy dell’infrastruttura: aggiorna → esegui i check → riavvia (oppure usa `openclaw update`, che esegue anche il riavvio) → verifica.

<div id="recommended-re-run-the-website-installer-upgrade-in-place">
  ## Consigliato: riesegui il programma di installazione dal sito web (aggiornamento in-place)
</div>

Il percorso di aggiornamento **preferito** è rieseguire il programma di installazione dal sito web. Rileva le installazioni esistenti, le aggiorna in-place ed esegue `openclaw doctor` quando necessario.

```bash
curl -fsSL https://openclaw.bot/install.sh | bash
```

Note:

* Aggiungi `--no-onboard` se non vuoi eseguire nuovamente la procedura guidata di onboarding.
* Per le **installazioni da sorgente**, usa:
  ```bash
  curl -fsSL https://openclaw.bot/install.sh | bash -s -- --install-method git --no-onboard
  ```
  Il programma di installazione eseguirà `git pull --rebase` **solo** se il repository è pulito.
* Per le **installazioni globali**, lo script usa `npm install -g openclaw@latest` dietro le quinte.
* Nota storica: `openclaw` rimane disponibile come shim di compatibilità.

<div id="before-you-update">
  ## Prima di aggiornare
</div>

* Verifica come hai effettuato l&#39;installazione: **globale** (npm/pnpm) vs **da sorgente** (git clone).
* Verifica come è in esecuzione il tuo Gateway: **in primo piano nel terminale** vs **come servizio supervisionato** (launchd/systemd).
* Crea uno snapshot delle tue personalizzazioni:
  * Configurazione: `~/.openclaw/openclaw.json`
  * Credenziali: `~/.openclaw/credentials/`
  * Spazio di lavoro: `~/.openclaw/workspace`

<div id="update-global-install">
  ## Aggiornamento (installazione globale)
</div>

Installazione globale (scegline uno):

```bash
npm i -g openclaw@latest
```

```bash
pnpm add -g openclaw@latest
```

Sconsigliamo **fortemente** di usare Bun per il runtime del Gateway (bug con WhatsApp/Telegram).

Per passare a un altro canale di aggiornamento (installazioni via git + npm):

```bash
openclaw update --channel beta
openclaw update --channel dev
openclaw update --channel stable
```

Usa `--tag <dist-tag|version>` per un&#39;installazione una tantum con un tag/versione specifico.

Consulta [Canali di sviluppo](/it/install/development-channels) per la semantica dei canali e le note di rilascio.

Nota: nelle installazioni via npm, il Gateway emette nei log un suggerimento di aggiornamento all&#39;avvio (verifica il tag del canale corrente). Disattivalo tramite `update.checkOnStart: false`.

Quindi:

```bash
openclaw doctor
openclaw gateway restart
openclaw health
```

Note:

* Se il tuo Gateway viene eseguito come servizio, `openclaw gateway restart` è preferibile rispetto al terminare i PID manualmente.
* Se sei vincolato a una versione specifica, vedi “Rollback / pinning” più sotto.

<div id="update-openclaw-update">
  ## Aggiornamento (`openclaw update`)
</div>

Per le **installazioni da sorgente** (git checkout), usa preferibilmente:

```bash
openclaw update
```

Esegue un flusso di aggiornamento relativamente sicuro:

* Richiede una worktree pulita.
* Passa al canale selezionato (tag o branch).
* Esegue fetch + rebase sull’upstream configurato (canale dev).
* Installa le dipendenze, esegue il build, compila la Control UI ed esegue `openclaw doctor`.
* Per impostazione predefinita riavvia il Gateway (usa `--no-restart` per non riavviarlo).

Se hai installato tramite **npm/pnpm** (senza metadati git), `openclaw update` proverà ad aggiornare l’installazione tramite il tuo package manager. Se non riesce a rilevare l’installazione, usa “Aggiorna (installazione globale)” al suo posto.

<div id="update-control-ui-rpc">
  ## Aggiornamento (Control UI / RPC)
</div>

La Control UI dispone di **Update &amp; Restart** (RPC: `update.run`). Questa funzione:

1. Esegue lo stesso flusso di aggiornamento del codice sorgente di `openclaw update` (solo git checkout).
2. Scrive una sentinella di riavvio con un report strutturato (parte finale di stdout/stderr).
3. Riavvia il Gateway e invia un ping all&#39;ultima sessione attiva includendo il report.

Se il rebase fallisce, il Gateway interrompe l&#39;operazione e si riavvia senza applicare l&#39;aggiornamento.

<div id="update-from-source">
  ## Aggiornamento (dai sorgenti)
</div>

Dal checkout del repository:

Metodo preferito:

```bash
openclaw update
```

Procedura manuale (più o meno equivalente):

```bash
git pull
pnpm install
pnpm build
pnpm ui:build # installa automaticamente le dipendenze della UI alla prima esecuzione
openclaw doctor
openclaw health
```

Note:

* `pnpm build` è importante quando esegui il binario pacchettizzato `openclaw` ([`openclaw.mjs`](https://github.com/openclaw/openclaw/blob/main/openclaw.mjs)) o usi Node.js per eseguire `dist/`.
* Se esegui da un checkout del repository senza un&#39;installazione globale, usa `pnpm openclaw ...` per i comandi CLI.
* Se esegui direttamente da TypeScript (`pnpm openclaw ...`), una ricompilazione di solito non è necessaria, ma **le migrazioni della configurazione si applicano comunque** → esegui `openclaw doctor`.
* Passare tra installazioni globali e git è semplice: installa l&#39;altra variante, quindi esegui `openclaw doctor` in modo che il punto di ingresso del servizio Gateway venga riscritto sull&#39;installazione corrente.

<div id="always-run-openclaw-doctor">
  ## Esegui sempre: `openclaw doctor`
</div>

`openclaw doctor` è il comando di “aggiornamento sicuro”. È intenzionalmente noioso: ripara + migra + avvisa.

Nota: se stai usando un’**installazione dai sorgenti** (git checkout), `openclaw doctor` ti proporrà prima di eseguire `openclaw update`.

Operazioni tipiche che esegue:

* Migrare chiavi di configurazione deprecate e posizioni legacy dei file di configurazione.
* Controllare le policy per i DM e avvisare in caso di impostazioni “open” rischiose (cioè che permettono di accettare messaggi da chiunque senza restrizioni).
* Verificare lo stato di salute del Gateway e, se necessario, proporre un riavvio.
* Rilevare e migrare vecchi servizi del Gateway (launchd/systemd; schtasks legacy) ai servizi OpenClaw correnti.
* Su Linux, verificare e abilitare il “systemd user lingering” (in modo che il Gateway sopravviva al logout).

Dettagli: [Doctor](/it/gateway/doctor)

<div id="start-stop-restart-the-gateway">
  ## Avvia / arresta / riavvia il Gateway
</div>

CLI (funziona su qualsiasi sistema operativo):

```bash
openclaw gateway status
openclaw gateway stop
openclaw gateway restart
openclaw gateway --port 18789
openclaw logs --follow
```

Se è supervisionato:

* macOS launchd (LaunchAgent incluso nell’app): `launchctl kickstart -k gui/$UID/bot.molt.gateway` (utilizza `bot.molt.<profile>`; il vecchio `com.openclaw.*` funziona ancora)
* Servizio utente systemd su Linux: `systemctl --user restart openclaw-gateway[-<profile>].service`
* Windows (WSL2): `systemctl --user restart openclaw-gateway[-<profile>].service`
  * `launchctl`/`systemctl` funzionano solo se il servizio è installato; in caso contrario esegui `openclaw gateway install`.

Runbook + etichette di servizio esatte: [Gateway runbook](/it/gateway)

<div id="rollback-pinning-when-something-breaks">
  ## Rollback / blocco versione (in caso di problemi)
</div>

<div id="pin-global-install">
  ### Pin (installazione globale)
</div>

Installa una versione nota per funzionare correttamente (sostituisci `<version>` con l&#39;ultima che funzionava correttamente):

```bash
npm i -g openclaw@<version>
```

```bash
pnpm add -g openclaw@<version>
```

Suggerimento: per vedere la versione attualmente pubblicata, esegui `npm view openclaw version`.

Quindi riavvia ed esegui nuovamente doctor:

```bash
openclaw doctor
openclaw gateway restart
```

<div id="pin-source-by-date">
  ### Fissa (sorgente) per data
</div>

Seleziona un commit in base a una data (esempio: &quot;stato di main alla data del 2026-01-01&quot;):

```bash
git fetch origin
git checkout "$(git rev-list -n 1 --before=\"2026-01-01\" origin/main)"
```

Poi reinstalla le dipendenze e riavvia:

```bash
pnpm install
pnpm build
openclaw gateway restart
```

Se più tardi vuoi tornare a usare l&#39;ultima versione disponibile:

```bash
git checkout main
git pull
```

<div id="if-youre-stuck">
  ## Se sei bloccato
</div>

* Esegui di nuovo `openclaw doctor` e leggi attentamente l&#39;output (spesso contiene già la soluzione).
* Controlla: [Risoluzione dei problemi](/it/gateway/troubleshooting)
* Chiedi su Discord: https://channels.discord.gg/clawd