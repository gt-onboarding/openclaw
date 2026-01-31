---
title: Windows
summary: "Windows-(WSL2)-Support + Status der Companion-App"
read_when:
  - OpenClaw unter Windows installieren
  - Status der Windows-Companion-App prüfen
---

<div id="windows-wsl2">
  # Windows (WSL2)
</div>

OpenClaw auf Windows wird **über WSL2** empfohlen (empfohlen: Ubuntu). Die
CLI und das Gateway laufen innerhalb von Linux, was für eine konsistente Laufzeitumgebung sorgt und
die Kompatibilität mit Tools deutlich erhöht (Node/Bun/pnpm, Linux-Binaries, Fähigkeiten). Native
Windows-Installationen sind derzeit ungetestet und eher problematisch.

Native Windows-Companion-Apps sind in Planung.

<div id="install-wsl2">
  ## Installation (WSL2)
</div>

* [Erste Schritte](/de/start/getting-started) (in WSL verwenden)
* [Installation &amp; Updates](/de/install/updating)
* Offizieller WSL2‑Leitfaden (Microsoft): https://learn.microsoft.com/windows/wsl/install

<div id="gateway">
  ## Gateway
</div>

* [Gateway-Runbook](/de/gateway)
* [Konfiguration](/de/gateway/configuration)

<div id="gateway-service-install-cli">
  ## Installation des Gateway-Dienstes (CLI)
</div>

In WSL2:

```
openclaw onboard --install-daemon
```

Alternativ:

```
openclaw gateway install
```

Oder:

```
openclaw configure
```

Wählen Sie bei der Aufforderung den **Gateway-Dienst** aus.

Reparieren/Migrieren:

```
openclaw doctor
```

<div id="advanced-expose-wsl-services-over-lan-portproxy">
  ## Erweitert: WSL-Dienste im LAN verfügbar machen (portproxy)
</div>

WSL verfügt über ein eigenes virtuelles Netzwerk. Wenn ein anderer Rechner einen Dienst
erreichen muss, der **innerhalb von WSL** läuft (SSH, ein lokaler TTS-Server oder das Gateway), musst du
einen Windows-Port an die aktuelle WSL-IP weiterleiten. Die WSL-IP ändert sich nach Neustarts,
daher musst du die Weiterleitungsregel ggf. aktualisieren.

Beispiel (PowerShell **als Administrator**):

```powershell
$Distro = "Ubuntu-24.04"
$ListenPort = 2222
$TargetPort = 22

$WslIp = (wsl -d $Distro -- hostname -I).Trim().Split(" ")[0]
if (-not $WslIp) { throw "WSL-IP nicht gefunden." }

netsh interface portproxy add v4tov4 listenaddress=0.0.0.0 listenport=$ListenPort `
  connectaddress=$WslIp connectport=$TargetPort
```

Öffnen Sie den Port einmalig in der Windows-Firewall:

```powershell
New-NetFirewallRule -DisplayName "WSL SSH $ListenPort" -Direction Inbound `
  -Protocol TCP -LocalPort $ListenPort -Action Allow
```

Aktualisiere die Portproxy-Konfiguration nach einem Neustart von WSL:

```powershell
netsh interface portproxy delete v4tov4 listenport=$ListenPort listenaddress=0.0.0.0 | Out-Null
netsh interface portproxy add v4tov4 listenport=$ListenPort listenaddress=0.0.0.0 `
  connectaddress=$WslIp connectport=$TargetPort | Out-Null
```

Hinweise:

* SSH von einem anderen Rechner muss die **IP des Windows-Hosts** ansprechen (Beispiel: `ssh user@windows-host -p 2222`).
* Remote-Knoten müssen auf eine **erreichbare** Gateway-URL zeigen (nicht `127.0.0.1`); verwende
  `openclaw status --all`, um das zu überprüfen.
* Verwende `listenaddress=0.0.0.0` für LAN-Zugriff; `127.0.0.1` beschränkt den Zugriff auf den lokalen Host.
* Wenn du das automatisieren möchtest, registriere eine geplante Aufgabe (Scheduled Task), die den Aktualisierungsschritt bei der Anmeldung ausführt.

<div id="step-by-step-wsl2-install">
  ## Schritt-für-Schritt-Installation von WSL2
</div>

<div id="1-install-wsl2-ubuntu">
  ### 1) WSL2 + Ubuntu installieren
</div>

Öffnen Sie PowerShell (als Administrator):

```powershell
wsl --install
# Oder wählen Sie explizit eine Distro aus:
wsl --list --online
wsl --install -d Ubuntu-24.04
```

Starte den Computer neu, wenn Windows dazu auffordert.

<div id="2-enable-systemd-required-for-gateway-install">
  ### 2) systemd aktivieren (erforderlich für die Gateway-Installation)
</div>

In deinem WSL-Terminal:

```bash
sudo tee /etc/wsl.conf >/dev/null <<'EOF'
[boot]
systemd=true
EOF
```

Dann in der PowerShell:

```powershell
wsl --shutdown
```

Öffne Ubuntu erneut und überprüfe:

```bash
systemctl --user status
```

<div id="3-install-openclaw-inside-wsl">
  ### 3) OpenClaw installieren (in WSL)
</div>

Folge der Linux-Getting-Started-Anleitung in WSL:

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm ui:build # installiert UI-Abhängigkeiten beim ersten Ausführen automatisch
pnpm build
openclaw onboard
```

Vollständige Anleitung: [Erste Schritte](/de/start/getting-started)

<div id="windows-companion-app">
  ## Windows-Companion-App
</div>

Wir haben derzeit noch keine Windows-Companion-App. Beiträge sind willkommen, wenn du dazu beitragen möchtest, dass es eine gibt.