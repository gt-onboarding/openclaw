---
title: Risoluzione dei problemi del browser su Linux
summary: "Risolvi i problemi di avvio CDP di Chrome/Brave/Edge/Chromium per il controllo del browser OpenClaw su Linux"
read_when: "Il controllo del browser non funziona su Linux, soprattutto con la versione snap di Chromium"
---

<div id="browser-troubleshooting-linux">
  # Risoluzione dei problemi del browser (Linux)
</div>

<div id="problem-failed-to-start-chrome-cdp-on-port-18800">
  ## Problema: &quot;Failed to start Chrome CDP on port 18800&quot;
</div>

Il server di controllo del browser di OpenClaw non riesce ad avviare Chrome/Brave/Edge/Chromium e mostra il seguente errore:

```
{"error":"Error: Failed to start Chrome CDP on port 18800 for profile \"openclaw\"."}
```


<div id="root-cause">
  ### Causa principale
</div>

Su Ubuntu (e molte altre distro Linux), l&#39;installazione predefinita di Chromium è un **pacchetto snap**. Il confinamento AppArmor di snap interferisce con il modo in cui OpenClaw avvia e monitora il processo del browser.

Il comando `apt install chromium` installa un pacchetto segnaposto (stub) che reindirizza a snap:

```
Note, selecting 'chromium-browser' instead of 'chromium'
chromium-browser is already the newest version (2:1snap1-0ubuntu2).
```

Questo NON è un vero browser, ma solo un semplice “wrapper”.


<div id="solution-1-install-google-chrome-recommended">
  ### Soluzione 1: Installa Google Chrome (consigliato)
</div>

Installa il pacchetto `.deb` ufficiale di Google Chrome, che non è eseguito in sandbox tramite Snap:

```bash
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
sudo dpkg -i google-chrome-stable_current_amd64.deb
sudo apt --fix-broken install -y  # in caso di errori di dipendenze
```

Quindi aggiorna il file di configurazione di OpenClaw (`~/.openclaw/openclaw.json`):

```json
{
  "browser": {
    "enabled": true,
    "executablePath": "/usr/bin/google-chrome-stable",
    "headless": true,
    "noSandbox": true
  }
}
```


<div id="solution-2-use-snap-chromium-with-attach-only-mode">
  ### Soluzione 2: Usa Chromium (snap) in modalità solo attach
</div>

Se devi usare Chromium installato tramite snap, configura OpenClaw per collegarsi a un browser avviato manualmente:

1. Aggiorna la configurazione:

```json
{
  "browser": {
    "enabled": true,
    "attachOnly": true,
    "headless": true,
    "noSandbox": true
  }
}
```

2. Avvia Chromium manualmente:

```bash
chromium-browser --headless --no-sandbox --disable-gpu \
  --remote-debugging-port=18800 \
  --user-data-dir=$HOME/.openclaw/browser/openclaw/user-data \
  about:blank &
```

3. Facoltativamente, puoi creare un servizio utente systemd per avviare automaticamente Chrome:

```ini
# ~/.config/systemd/user/openclaw-browser.service
[Unit]
Description=OpenClaw Browser (Chrome CDP)
After=network.target

[Service]
ExecStart=/snap/bin/chromium --headless --no-sandbox --disable-gpu --remote-debugging-port=18800 --user-data-dir=%h/.openclaw/browser/openclaw/user-data about:blank
Restart=on-failure
RestartSec=5

[Install]
WantedBy=default.target
```

Abilita il servizio con: `systemctl --user enable --now openclaw-browser.service`


<div id="verifying-the-browser-works">
  ### Verifica che il browser funzioni
</div>

Verifica lo stato:

```bash
curl -s http://127.0.0.1:18791/ | jq '{running, pid, chosenBrowser}'
```

Verifica la navigazione:

```bash
curl -s -X POST http://127.0.0.1:18791/start
curl -s http://127.0.0.1:18791/tabs
```


<div id="config-reference">
  ### Riferimento configurazione
</div>

| Opzione | Descrizione | Valore predefinito |
|--------|-------------|--------------------|
| `browser.enabled` | Abilita il controllo del browser | `true` |
| `browser.executablePath` | Percorso di un eseguibile di browser basato su Chromium (Chrome/Brave/Edge/Chromium) | rilevato automaticamente (preferisce il browser predefinito se basato su Chromium) |
| `browser.headless` | Esegui senza interfaccia grafica (headless) | `false` |
| `browser.noSandbox` | Aggiungi il flag `--no-sandbox` (necessario per alcune configurazioni Linux) | `false` |
| `browser.attachOnly` | Non avviare il browser, collegati solo a uno già esistente | `false` |
| `browser.cdpPort` | Porta del Chrome DevTools Protocol | `18800` |

<div id="problem-chrome-extension-relay-is-running-but-no-tab-is-connected">
  ### Problema: "Il relay dell’estensione Chrome è in esecuzione, ma nessuna scheda è connessa"
</div>

Stai usando il profilo `chrome` (relay dell’estensione). Questo profilo richiede che l’estensione browser di OpenClaw sia collegata a una scheda attiva.

Possibili soluzioni:

1. **Usa il browser gestito:** `openclaw browser start --browser-profile openclaw`
   (oppure imposta `browser.defaultProfile: "openclaw"`).
2. **Usa il relay dell’estensione:** installa l’estensione, apri una scheda e fai clic
   sull’icona dell’estensione OpenClaw per collegarla.

Note:

- Il profilo `chrome` utilizza, quando possibile, il **browser Chromium predefinito di sistema**.
- I profili `openclaw` locali assegnano automaticamente `cdpPort`/`cdpUrl`; imposta questi valori solo per CDP remoti.