---
title: Ansible
summary: "Automatisierte, gehärtete OpenClaw-Installation mit Ansible, Tailscale-VPN und Firewall-Isolierung"
read_when:
  - Sie möchten Server automatisiert bereitstellen, inklusive Sicherheitshärtung
  - Sie benötigen ein firewall-isoliertes Setup mit VPN-Zugriff
  - Sie stellen auf entfernten Debian-/Ubuntu-Servern bereit
---

<div id="ansible-installation">
  # Ansible-Installation
</div>

Die empfohlene Methode, um OpenClaw auf Produktionsservern bereitzustellen, ist über **[openclaw-ansible](https://github.com/openclaw/openclaw-ansible)** — einen automatisierten Installer mit Security-first-Architektur.

<div id="quick-start">
  ## Schnellstart
</div>

Installation mit einem einzigen Befehl:

```bash
curl -fsSL https://raw.githubusercontent.com/openclaw/openclaw-ansible/main/install.sh | bash
```

> **📦 Umfassende Anleitung: [github.com/openclaw/openclaw-ansible](https://github.com/openclaw/openclaw-ansible)**
>
> Das Repository openclaw-ansible ist die zentrale Referenz für das Ansible-Deployment. Diese Seite gibt einen kurzen Überblick.

<div id="what-you-get">
  ## Was du bekommst
</div>

* 🔒 **Firewall-zentrierte Sicherheit**: UFW + Docker-Isolierung (nur SSH + Tailscale erreichbar)
* 🔐 **Tailscale-VPN**: Sicherer Remote-Zugriff, ohne Dienste öffentlich freizugeben
* 🐳 **Docker**: Isolierte Sandbox-Container, nur localhost-Bindings
* 🛡️ **Mehrschichtige Verteidigung**: Vierstufige Sicherheitsarchitektur
* 🚀 **Setup mit einem Befehl**: Vollständiges Deployment in wenigen Minuten
* 🔧 **Systemd-Integration**: Automatischer Start beim Booten mit zusätzlicher Härtung

<div id="requirements">
  ## Anforderungen
</div>

* **OS**: Debian 11+ oder Ubuntu 20.04+
* **Zugriff**: Root- oder sudo-Rechte
* **Netzwerk**: Internetverbindung für die Paketinstallation
* **Ansible**: 2.14+ (wird automatisch vom Quickstart-Skript installiert)

<div id="what-gets-installed">
  ## Was installiert wird
</div>

Das Ansible-Playbook installiert und konfiguriert:

1. **Tailscale** (Mesh-VPN für sicheren Remote-Zugriff)
2. **UFW-Firewall** (nur SSH- und Tailscale-Ports)
3. **Docker CE + Compose V2** (für Agent-Sandboxes)
4. **Node.js 22.x + pnpm** (Laufzeitabhängigkeiten)
5. **OpenClaw** (hostbasiert, nicht containerisiert)
6. **systemd-Service** (Autostart mit Sicherheits-Härtung)

Hinweis: Das Gateway läuft **direkt auf dem Host** (nicht in Docker), aber Agent-Sandboxes verwenden Docker zur Isolation. Details findest du unter [Sandboxing](/de/gateway/sandboxing).

<div id="post-install-setup">
  ## Einrichtung nach der Installation
</div>

Nach Abschluss der Installation wechseln Sie zum Benutzer openclaw:

```bash
sudo -i -u openclaw
```

Das Postinstallationsskript führt dich durch:

1. **Onboarding-Assistent**: Konfiguration der OpenClaw-Einstellungen
2. **Provider-Login**: Verbindung von WhatsApp/Telegram/Discord/Signal
3. **Gateway-Test**: Überprüfung der Installation
4. **Tailscale-Einrichtung**: Verbindung mit deinem VPN-Mesh

<div id="quick-commands">
  ### Kurzbefehle
</div>

```bash
# Dienststatus überprüfen
sudo systemctl status openclaw

# Live-Logs anzeigen
sudo journalctl -u openclaw -f

# Gateway neu starten
sudo systemctl restart openclaw

# Anbieter-Login (als openclaw-Benutzer ausführen)
sudo -i -u openclaw
openclaw channels login
```

<div id="security-architecture">
  ## Sicherheitsarchitektur
</div>

<div id="4-layer-defense">
  ### Verteidigung auf 4 Ebenen
</div>

1. **Firewall (UFW)**: Nur SSH (22) + Tailscale (41641/udp) öffentlich erreichbar
2. **VPN (Tailscale)**: Gateway nur über VPN-Mesh erreichbar
3. **Docker-Isolierung**: DOCKER-USER-iptables-Chain verhindert die Freigabe externer Ports
4. **Systemd-Härtung**: NoNewPrivileges, PrivateTmp, nicht privilegierter Benutzer

<div id="verification">
  ### Überprüfung
</div>

Prüfe die externe Angriffsfläche:

```bash
nmap -p- YOUR_SERVER_IP
```

Es sollte **nur Port 22** (SSH) offen sein. Alle anderen Dienste (Gateway, Docker) müssen strikt abgeschottet sein.

<div id="docker-availability">
  ### Docker-Verfügbarkeit
</div>

Docker wird für **agent sandboxes** (isolierte Tool-Ausführung) installiert, nicht, um das Gateway selbst auszuführen. Das Gateway bindet nur an localhost und ist über das Tailscale-VPN erreichbar.

Siehe [Multi-Agent Sandbox &amp; Tools](/de/multi-agent-sandbox-tools) für die Sandbox-Konfiguration.

<div id="manual-installation">
  ## Manuelle Installation
</div>

Wenn Sie die Automatisierung lieber manuell steuern möchten:

```bash
# 1. Install prerequisites
sudo apt update && sudo apt install -y ansible git

# 2. Clone repository
git clone https://github.com/openclaw/openclaw-ansible.git
cd openclaw-ansible

# 3. Install Ansible collections
ansible-galaxy collection install -r requirements.yml

# 4. Run playbook
./run-playbook.sh

# Oder direkt ausführen (danach /tmp/openclaw-setup.sh manuell ausführen)
# ansible-playbook playbook.yml --ask-become-pass
```

<div id="updating-openclaw">
  ## OpenClaw aktualisieren
</div>

Der Ansible-Installer richtet OpenClaw für manuelle Updates ein. Siehe [Aktualisierung](/de/install/updating) für den Standard-Update-Prozess.

So führst du das Ansible-Playbook erneut aus (z. B. für Konfigurationsänderungen):

```bash
cd openclaw-ansible
./run-playbook.sh
```

Hinweis: Dies ist idempotent und kann gefahrlos mehrmals ausgeführt werden.

<div id="troubleshooting">
  ## Fehlerbehebung
</div>

<div id="firewall-blocks-my-connection">
  ### Firewall blockiert meine Verbindung
</div>

Wenn du keinen Zugriff mehr hast:

* Stelle zuerst sicher, dass du über das Tailscale-VPN zugreifen kannst
* SSH-Zugriff (Port 22) ist immer erlaubt
* Das Gateway ist **ausschließlich** über Tailscale erreichbar – das ist so vorgesehen

<div id="service-wont-start">
  ### Dienst startet nicht
</div>

```bash
# Check logs
sudo journalctl -u openclaw -n 100

# Verify permissions
sudo ls -la /opt/openclaw

# Manuellen Start testen
sudo -i -u openclaw
cd ~/openclaw
pnpm start
```

<div id="docker-sandbox-issues">
  ### Probleme mit der Docker-sandbox
</div>

```bash
# Prüfen, ob Docker läuft
sudo systemctl status docker

# Sandbox-Image prüfen
sudo docker images | grep openclaw-sandbox

# Sandbox-Image erstellen, falls fehlend
cd /opt/openclaw/openclaw
sudo -u openclaw ./scripts/sandbox-setup.sh
```

<div id="provider-login-fails">
  ### Anmeldung beim Anbieter schlägt fehl
</div>

Stelle sicher, dass du als Benutzer `openclaw` angemeldet bist:

```bash
sudo -i -u openclaw
openclaw channels login
```

<div id="advanced-configuration">
  ## Erweiterte Konfiguration
</div>

Für detaillierte Informationen zur Sicherheitsarchitektur und Fehlerbehebung:

* [Sicherheitsarchitektur](https://github.com/openclaw/openclaw-ansible/blob/main/docs/security.md)
* [Technische Details](https://github.com/openclaw/openclaw-ansible/blob/main/docs/architecture.md)
* [Leitfaden zur Fehlerbehebung](https://github.com/openclaw/openclaw-ansible/blob/main/docs/troubleshooting.md)

<div id="related">
  ## Verwandte Inhalte
</div>

* [openclaw-ansible](https://github.com/openclaw/openclaw-ansible) — vollständige Deployment-Anleitung
* [Docker](/de/install/docker) — containerisierte Gateway-Einrichtung
* [Sandboxing](/de/gateway/sandboxing) — Agent-Sandbox-Konfiguration
* [Multi-Agent Sandbox &amp; Tools](/de/multi-agent-sandbox-tools) — Isolierung auf Agent-Ebene