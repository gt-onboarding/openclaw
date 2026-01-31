---
title: Browser-Fehlerbehebung unter Linux
summary: "Behebung von Chrome/Brave/Edge/Chromium-CDP-Startproblemen für die OpenClaw-Browsersteuerung unter Linux"
read_when: "Browsersteuerung funktioniert unter Linux nicht, insbesondere bei Snap-Chromium"
---

<div id="browser-troubleshooting-linux">
  # Fehlerbehebung für Browser (Linux)
</div>

<div id="problem-failed-to-start-chrome-cdp-on-port-18800">
  ## Problem: &quot;Failed to start Chrome CDP on port 18800&quot;
</div>

Der Browsersteuerungsserver von OpenClaw kann Chrome/Brave/Edge/Chromium nicht starten und bricht mit folgendem Fehler ab:

```
{"error":"Error: Failed to start Chrome CDP on port 18800 for profile \"openclaw\"."}
```


<div id="root-cause">
  ### Hauptursache
</div>

Unter Ubuntu (und vielen anderen Linux-Distributionen) wird Chromium standardmäßig als **Snap-Paket** installiert. Die AppArmor-Restriktionen von Snap beeinträchtigen die Art und Weise, wie OpenClaw den Browser-Prozess startet und überwacht.

Der Befehl `apt install chromium` installiert ein Stub-Paket, das nur auf Snap umleitet:

```
Note, selecting 'chromium-browser' instead of 'chromium'
chromium-browser is already the newest version (2:1snap1-0ubuntu2).
```

Dies ist KEIN vollwertiger Browser — es ist nur ein Wrapper.


<div id="solution-1-install-google-chrome-recommended">
  ### Lösung 1: Google Chrome installieren (empfohlen)
</div>

Installiere das offizielle `.deb`-Paket von Google Chrome, das nicht in einer Snap-sandbox ausgeführt wird:

```bash
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
sudo dpkg -i google-chrome-stable_current_amd64.deb
sudo apt --fix-broken install -y  # falls Abhängigkeitsfehler auftreten
```

Aktualisiere anschließend deine OpenClaw-Konfiguration (`~/.openclaw/openclaw.json`):

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
  ### Lösung 2: Snap-Chromium mit Attach-Only-Modus verwenden
</div>

Wenn Sie zwingend Snap-Chromium verwenden müssen, konfigurieren Sie OpenClaw so, dass es sich mit einem manuell gestarteten Browser verbindet:

1. Konfiguration aktualisieren:

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

2. Starte Chromium manuell:

```bash
chromium-browser --headless --no-sandbox --disable-gpu \
  --remote-debugging-port=18800 \
  --user-data-dir=$HOME/.openclaw/browser/openclaw/user-data \
  about:blank &
```

3. Optional: Richte einen systemd-User-Service ein, um Chrome automatisch zu starten:

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

Aktiviere den Dienst mit: `systemctl --user enable --now openclaw-browser.service`


<div id="verifying-the-browser-works">
  ### Überprüfen, ob der Browser läuft
</div>

Status prüfen:

```bash
curl -s http://127.0.0.1:18791/ | jq '{running, pid, chosenBrowser}'
```

Browserzugriff testen:

```bash
curl -s -X POST http://127.0.0.1:18791/start
curl -s http://127.0.0.1:18791/tabs
```


<div id="config-reference">
  ### Konfigurationsreferenz
</div>

| Option | Beschreibung | Standardwert |
|--------|-------------|---------|
| `browser.enabled` | Browsersteuerung aktivieren | `true` |
| `browser.executablePath` | Pfad zu einer Chromium-basierten ausführbaren Browser-Datei (Chrome/Brave/Edge/Chromium) | automatisch erkannt (bevorzugt den Standardbrowser, wenn dieser Chromium-basiert ist) |
| `browser.headless` | Ohne GUI ausführen | `false` |
| `browser.noSandbox` | `--no-sandbox`-Flag hinzufügen (für einige Linux-Setups erforderlich) | `false` |
| `browser.attachOnly` | Browser nicht starten, sondern nur an einen bereits laufenden anhängen | `false` |
| `browser.cdpPort` | Chrome DevTools Protocol-Port | `18800` |

<div id="problem-chrome-extension-relay-is-running-but-no-tab-is-connected">
  ### Problem: „Chrome-Erweiterungs-Relay läuft, aber kein Tab ist verbunden“
</div>

Du verwendest das `chrome`-Profil (Extension-Relay). Es erwartet, dass die
OpenClaw-Browsererweiterung mit einem aktiven Tab verbunden ist.

Mögliche Lösungen:

1. **Verwende den verwalteten Browser:** `openclaw browser start --browser-profile openclaw`
   (oder setze `browser.defaultProfile: "openclaw"`).
2. **Verwende das Extension-Relay:** Installiere die Erweiterung, öffne einen Tab und klicke auf das
   OpenClaw-Erweiterungssymbol, um sie zu verbinden.

Hinweise:

- Das `chrome`-Profil verwendet nach Möglichkeit deinen **systemweiten Standard-Chromium-Browser**.
- Lokale `openclaw`-Profile weisen `cdpPort`/`cdpUrl` automatisch zu; setze diese nur für ein Remote-CDP.