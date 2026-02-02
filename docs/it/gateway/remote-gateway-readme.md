---
title: Readme del Gateway remoto
summary: "Configurazione di un tunnel SSH per OpenClaw.app verso un Gateway remoto"
read_when: "Collegamento dell'app macOS a un Gateway remoto tramite SSH"
---

<div id="running-openclawapp-with-a-remote-gateway">
  # Eseguire OpenClaw.app con un Gateway remoto
</div>

OpenClaw.app utilizza il tunneling SSH per connettersi a un Gateway remoto. Questa guida spiega come configurarla.

<div id="overview">
  ## Panoramica
</div>

```
┌─────────────────────────────────────────────────────────────┐
│                        Client Machine                          │
│                                                              │
│  OpenClaw.app ──► ws://127.0.0.1:18789 (local port)           │
│                     │                                        │
│                     ▼                                        │
│  SSH Tunnel ────────────────────────────────────────────────│
│                     │                                        │
└─────────────────────┼──────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────┐
│                         Remote Machine                        │
│                                                              │
│  Gateway WebSocket ──► ws://127.0.0.1:18789 ──►              │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```


<div id="quick-setup">
  ## Configurazione rapida
</div>

<div id="step-1-add-ssh-config">
  ### Passaggio 1: Aggiungi la configurazione SSH
</div>

Modifica il file `~/.ssh/config` e aggiungi:

```ssh
Host remote-gateway
    HostName <REMOTE_IP>          # es. 172.27.187.184
    User <REMOTE_USER>            # es. jefferson
    LocalForward 18789 127.0.0.1:18789
    IdentityFile ~/.ssh/id_rsa
```

Sostituisci `<REMOTE_IP>` e `<REMOTE_USER>` con i valori corretti.


<div id="step-2-copy-ssh-key">
  ### Passaggio 2: Copia della chiave SSH
</div>

Copia la chiave pubblica sulla macchina remota (ti verrà richiesta la password una sola volta):

```bash
ssh-copy-id -i ~/.ssh/id_rsa <REMOTE_USER>@<REMOTE_IP>
```


<div id="step-3-set-gateway-token">
  ### Passaggio 3: Imposta il token del Gateway
</div>

```bash
launchctl setenv OPENCLAW_GATEWAY_TOKEN "<your-token>"
```


<div id="step-4-start-ssh-tunnel">
  ### Passaggio 4: Avviare il tunnel SSH
</div>

```bash
ssh -N remote-gateway &
```


<div id="step-5-restart-openclawapp">
  ### Passaggio 5: Riavvia OpenClaw.app
</div>

```bash
# Chiudi OpenClaw.app (⌘Q), quindi riaprila:
open /path/to/OpenClaw.app
```

L&#39;app si connetterà ora al Gateway remoto tramite il tunnel SSH.

***


<div id="auto-start-tunnel-on-login">
  ## Avvio automatico del tunnel all'accesso
</div>

Per fare in modo che il tunnel SSH si avvii automaticamente quando esegui l'accesso, crea un Launch Agent.

<div id="create-the-plist-file">
  ### Crea il file PLIST
</div>

Salva il file come `~/Library/LaunchAgents/bot.molt.ssh-tunnel.plist`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>bot.molt.ssh-tunnel</string>
    <key>ProgramArguments</key>
    <array>
        <string>/usr/bin/ssh</string>
        <string>-N</string>
        <string>remote-gateway</string>
    </array>
    <key>KeepAlive</key>
    <true/>
    <key>RunAtLoad</key>
    <true/>
</dict>
</plist>
```


<div id="load-the-launch-agent">
  ### Caricare il Launch Agent
</div>

```bash
launchctl bootstrap gui/$UID ~/Library/LaunchAgents/bot.molt.ssh-tunnel.plist
```

Il tunnel ora:

* Si avvierà automaticamente quando effettui l&#39;accesso
* Si riavvierà in caso di arresto anomalo
* Continuerà a rimanere in esecuzione in background

Nota legacy: rimuovi eventuali LaunchAgent `com.openclaw.ssh-tunnel` residui, se presenti.

***


<div id="troubleshooting">
  ## Risoluzione dei problemi
</div>

**Verifica che il tunnel sia in esecuzione:**

```bash
ps aux | grep "ssh -N remote-gateway" | grep -v grep
lsof -i :18789
```

**Riavvia il tunnel:**

```bash
launchctl kickstart -k gui/$UID/bot.molt.ssh-tunnel
```

**Interrompi il tunnel:**

```bash
launchctl bootout gui/$UID/bot.molt.ssh-tunnel
```

***


<div id="how-it-works">
  ## Come funziona
</div>

| Componente | Funzione |
|-----------|----------|
| `LocalForward 18789 127.0.0.1:18789` | Inoltra la porta locale 18789 alla porta remota 18789 |
| `ssh -N` | SSH senza eseguire comandi remoti (solo inoltro di porte) |
| `KeepAlive` | Riavvia automaticamente il tunnel in caso di arresto/crash |
| `RunAtLoad` | Avvia il tunnel quando l'agente viene caricato |

OpenClaw.app si connette a `ws://127.0.0.1:18789` sulla tua macchina client. Il tunnel SSH inoltra quella connessione verso la porta 18789 sulla macchina remota dove è in esecuzione il Gateway.