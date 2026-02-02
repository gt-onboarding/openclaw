---
title: Remote-Gateway-Readme
summary: "Einrichtung eines SSH-Tunnels für OpenClaw.app zur Verbindung mit einem Remote-Gateway"
read_when: "Beim Verbinden der macOS-App mit einem Remote-Gateway über SSH"
---

<div id="running-openclawapp-with-a-remote-gateway">
  # Ausführen von OpenClaw.app mit einem Remote-Gateway
</div>

OpenClaw.app verwendet SSH-Tunneling, um eine Verbindung zu einem Remote-Gateway herzustellen. In dieser Anleitung erfährst du, wie du das einrichtest.

<div id="overview">
  ## Übersicht
</div>

```
┌─────────────────────────────────────────────────────────────┐
│                        Client-Rechner                          │
│                                                              │
│  OpenClaw.app ──► ws://127.0.0.1:18789 (lokaler Port)         │
│                     │                                        │
│                     ▼                                        │
│  SSH Tunnel ────────────────────────────────────────────────│
│                     │                                        │
└─────────────────────┼──────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────┐
│                         Remote-Rechner                        │
│                                                              │
│  Gateway WebSocket ──► ws://127.0.0.1:18789 ──►              │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```


<div id="quick-setup">
  ## Schnellstart
</div>

<div id="step-1-add-ssh-config">
  ### Schritt 1: SSH-Config hinzufügen
</div>

Bearbeite `~/.ssh/config` und füge Folgendes hinzu:

```ssh
Host remote-gateway
    HostName <REMOTE_IP>          # z. B. 172.27.187.184
    User <REMOTE_USER>            # z. B. jefferson
    LocalForward 18789 127.0.0.1:18789
    IdentityFile ~/.ssh/id_rsa
```

Ersetzen Sie `<REMOTE_IP>` und `<REMOTE_USER>` durch Ihre Werte.


<div id="step-2-copy-ssh-key">
  ### Schritt 2: SSH-Schlüssel kopieren
</div>

Kopieren Sie Ihren öffentlichen SSH-Schlüssel auf das Remote-System (Passwort einmalig eingeben):

```bash
ssh-copy-id -i ~/.ssh/id_rsa <REMOTE_USER>@<REMOTE_IP>
```


<div id="step-3-set-gateway-token">
  ### Schritt 3: Gateway-Token festlegen
</div>

```bash
launchctl setenv OPENCLAW_GATEWAY_TOKEN "<your-token>"
```


<div id="step-4-start-ssh-tunnel">
  ### Schritt 4: SSH-Tunnel starten
</div>

```bash
ssh -N remote-gateway &
```


<div id="step-5-restart-openclawapp">
  ### Schritt 5: Starte OpenClaw.app neu
</div>

```bash
# OpenClaw.app beenden (⌘Q), dann erneut öffnen:
open /path/to/OpenClaw.app
```

Die App verbindet sich nun über den SSH-Tunnel mit dem Remote-Gateway.

***


<div id="auto-start-tunnel-on-login">
  ## Tunnel beim Anmelden automatisch starten
</div>

Damit der SSH-Tunnel automatisch startet, wenn du dich anmeldest, erstelle einen Launch Agent.

<div id="create-the-plist-file">
  ### Erstelle die PLIST-Datei
</div>

Speichere diese Datei unter `~/Library/LaunchAgents/bot.molt.ssh-tunnel.plist`:

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
  ### Launch-Agent laden
</div>

```bash
launchctl bootstrap gui/$UID ~/Library/LaunchAgents/bot.molt.ssh-tunnel.plist
```

Der Tunnel wird nun:

* Beim Anmelden automatisch starten
* Bei einem Absturz automatisch neu starten
* Im Hintergrund weiterlaufen

Hinweis zu Altversionen: Entferne gegebenenfalls einen vorhandenen `com.openclaw.ssh-tunnel`-LaunchAgent.

***


<div id="troubleshooting">
  ## Fehlerbehebung
</div>

**Überprüfen, ob der Tunnel läuft:**

```bash
ps aux | grep "ssh -N remote-gateway" | grep -v grep
lsof -i :18789
```

**Tunnel neu starten:**

```bash
launchctl kickstart -k gui/$UID/bot.molt.ssh-tunnel
```

**Tunnel beenden:**

```bash
launchctl bootout gui/$UID/bot.molt.ssh-tunnel
```

***


<div id="how-it-works">
  ## Funktionsweise
</div>

| Komponente | Aufgabe |
|-----------|---------|
| `LocalForward 18789 127.0.0.1:18789` | Leitet den lokalen Port 18789 an den Remote-Port 18789 weiter |
| `ssh -N` | SSH ohne Ausführung von Remote-Befehlen (nur Portweiterleitung) |
| `KeepAlive` | Startet den Tunnel automatisch neu, wenn er abstürzt |
| `RunAtLoad` | Startet den Tunnel, wenn der Agent geladen wird |

OpenClaw.app stellt auf dem Client-System eine Verbindung zu `ws://127.0.0.1:18789` her. Der SSH-Tunnel leitet diese Verbindung an Port 18789 auf dem Remote-System weiter, auf dem das Gateway läuft.