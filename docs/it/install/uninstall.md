---
title: Disinstallazione
summary: "Disinstalla completamente OpenClaw (CLI, servizio, stato, spazio di lavoro)"
read_when:
  - Vuoi rimuovere OpenClaw da una macchina
  - Il servizio gateway è ancora in esecuzione dopo la disinstallazione
---

<div id="uninstall">
  # Disinstallazione
</div>

Due opzioni:

- **Opzione semplice** se `openclaw` è ancora installato.
- **Rimozione manuale del servizio** se la CLI non è più disponibile ma il servizio è ancora in esecuzione.

<div id="easy-path-cli-still-installed">
  ## Procedura semplificata (CLI ancora installata)
</div>

Consigliato: usa il programma di disinstallazione integrato:

```bash
openclaw uninstall
```

Modalità non interattiva (automazione / npx):

```bash
openclaw uninstall --all --yes --non-interactive
npx -y openclaw uninstall --all --yes --non-interactive
```

Procedura manuale (stesso risultato):

1. Arresta il servizio Gateway:

```bash
openclaw gateway stop
```

2. Disinstalla il servizio del Gateway (launchd/systemd/schtasks):

```bash
openclaw gateway uninstall
```

3. Elimina stato e configurazione:

```bash
rm -rf "${OPENCLAW_STATE_DIR:-$HOME/.openclaw}"
```

Se hai impostato `OPENCLAW_CONFIG_PATH` su un percorso personalizzato al di fuori della directory di stato, elimina anche quel file.

4. Elimina lo spazio di lavoro (opzionale; rimuove i file dell&#39;agente):

```bash
rm -rf ~/.openclaw/workspace
```

5. Rimuovi l&#39;installazione della CLI (seleziona quella che hai usato):

```bash
npm rm -g openclaw
pnpm remove -g openclaw
bun remove -g openclaw
```

6. Se hai installato l&#39;app per macOS:

```bash
rm -rf /Applications/OpenClaw.app
```

Note:

* Se hai usato i profili (`--profile` / `OPENCLAW_PROFILE`), ripeti il passaggio 3 per ogni directory di stato (per impostazione predefinita è `~/.openclaw-<profile>`).
* In modalità remota, la directory di stato risiede sull’**host del Gateway**, quindi esegui i passaggi 1-4 anche su quell’host.


<div id="manual-service-removal-cli-not-installed">
  ## Rimozione manuale del servizio (CLI non installata)
</div>

Usa questa procedura se il servizio Gateway continua a essere in esecuzione ma `openclaw` manca.

<div id="macos-launchd">
  ### macOS (launchd)
</div>

L&#39;etichetta predefinita è `bot.molt.gateway` (oppure `bot.molt.<profile>`; le etichette legacy `com.openclaw.*` potrebbero essere ancora presenti):

```bash
launchctl bootout gui/$UID/bot.molt.gateway
rm -f ~/Library/LaunchAgents/bot.molt.gateway.plist
```

Se hai usato un profilo, sostituisci l&#39;etichetta e il nome del plist con `bot.molt.<profile>`. Rimuovi eventuali plist legacy `com.openclaw.*` se presenti.


<div id="linux-systemd-user-unit">
  ### Linux (unità utente systemd)
</div>

Il nome dell&#39;unità predefinita è `openclaw-gateway.service` (oppure `openclaw-gateway-<profile>.service`):

```bash
systemctl --user disable --now openclaw-gateway.service
rm -f ~/.config/systemd/user/openclaw-gateway.service
systemctl --user daemon-reload
```


<div id="windows-scheduled-task">
  ### Windows (attività pianificata)
</div>

Il nome predefinito dell&#39;attività è `OpenClaw Gateway` (oppure `OpenClaw Gateway (<profile>)`).
Lo script dell&#39;attività si trova nella tua directory dello stato.

```powershell
schtasks /Delete /F /TN "OpenClaw Gateway"
Remove-Item -Force "$env:USERPROFILE\.openclaw\gateway.cmd"
```

Se hai utilizzato un profilo, elimina il nome dell&#39;attività corrispondente e `~\.openclaw-<profile>\gateway.cmd`.


<div id="normal-install-vs-source-checkout">
  ## Installazione normale vs checkout del codice sorgente
</div>

<div id="normal-install-installsh-npm-pnpm-bun">
  ### Installazione standard (install.sh / npm / pnpm / bun)
</div>

Se hai utilizzato `https://openclaw.bot/install.sh` o `install.ps1`, la CLI è stata installata con `npm install -g openclaw@latest`.
Rimuovila con `npm rm -g openclaw` (oppure `pnpm remove -g` / `bun remove -g` se l'hai installata in questo modo).

<div id="source-checkout-git-clone">
  ### Checkout del sorgente (git clone)
</div>

Se esegui da un checkout del repo (`git clone` + `openclaw ...` / `bun run openclaw ...`):

1) Disinstalla il servizio Gateway **prima** di eliminare il repo (usa la procedura semplice indicata sopra o rimuovi il servizio manualmente).
2) Elimina la directory del repo.
3) Rimuovi lo state + lo spazio di lavoro come mostrato sopra.