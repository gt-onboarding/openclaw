---
title: DigitalOcean
summary: "OpenClaw auf DigitalOcean (einfache, kostenpflichtige VPS-Option)"
read_when:
  - Einrichtung von OpenClaw auf DigitalOcean
  - Auf der Suche nach günstigem VPS-Hosting für OpenClaw
---

<div id="openclaw-on-digitalocean">
  # OpenClaw auf DigitalOcean
</div>

<div id="goal">
  ## Ziel
</div>

Einen dauerhaft laufenden OpenClaw-Gateway auf DigitalOcean für **6 $/Monat** (oder 4 $/Monat mit Reserved-Pricing-Tarif) betreiben.

Wenn du eine Option für 0 $/Monat möchtest und dich an ARM + anbieterspezifischem Setup nicht störst, siehe den [Oracle-Cloud-Leitfaden](/de/platforms/oracle).

<div id="cost-comparison-2026">
  ## Kostenvergleich (2026)
</div>

| Anbieter | Tarif | Specs | Preis/Monat | Anmerkungen |
|----------|------|-------|----------|-------|
| Oracle Cloud | Always Free ARM | bis zu 4 OCPU, 24GB RAM | $0 | ARM, begrenzte Kapazität / Besonderheiten bei der Anmeldung |
| Hetzner | CX22 | 2 vCPU, 4GB RAM | €3,79 (~$4) | Günstigste kostenpflichtige Option |
| DigitalOcean | Basic | 1 vCPU, 1GB RAM | $6 | Einfache UI, gute Dokumentation |
| Vultr | Cloud Compute | 1 vCPU, 1GB RAM | $6 | Viele Standorte |
| Linode | Nanode | 1 vCPU, 1GB RAM | $5 | Jetzt Teil von Akamai |

**Einen Anbieter auswählen:**

* DigitalOcean: einfachste UX + vorhersehbares Setup (diese Anleitung)
* Hetzner: gutes Preis-Leistungs-Verhältnis (siehe [Hetzner-Anleitung](/de/platforms/hetzner))
* Oracle Cloud: kann $0/Monat kosten, ist aber komplizierter und nur ARM (siehe [Oracle-Anleitung](/de/platforms/oracle))

***

<div id="prerequisites">
  ## Voraussetzungen
</div>

* DigitalOcean-Konto ([Registrierung mit 200 $ Startguthaben](https://m.do.co/c/signup))
* SSH-Schlüsselpaar (oder Bereitschaft, Passwortauthentifizierung zu verwenden)
* ~20 Minuten

<div id="1-create-a-droplet">
  ## 1) Droplet erstellen
</div>

1. Melde dich bei [DigitalOcean](https://cloud.digitalocean.com/) an
2. Klicke auf **Create → Droplets**
3. Wähle:
   * **Region:** Am nächsten zu dir (oder deinen Nutzern)
   * **Image:** Ubuntu 24.04 LTS
   * **Size:** Basic → Regular → **$6/mo** (1 vCPU, 1GB RAM, 25GB SSD)
   * **Authentication:** SSH-Schlüssel (empfohlen) oder Passwort
4. Klicke auf **Create Droplet**
5. Notiere dir die IP-Adresse

<div id="2-connect-via-ssh">
  ## 2) Über SSH verbinden
</div>

```bash
ssh root@YOUR_DROPLET_IP
```

<div id="3-install-openclaw">
  ## 3) OpenClaw installieren
</div>

```bash
# Update system
apt update && apt upgrade -y

# Node.js 22 installieren
curl -fsSL https://deb.nodesource.com/setup_22.x | bash -
apt install -y nodejs

# Install OpenClaw
curl -fsSL https://openclaw.bot/install.sh | bash

# Verify
openclaw --version
```

<div id="4-run-onboarding">
  ## 4) Onboarding durchführen
</div>

```bash
openclaw onboard --install-daemon
```

Der Assistent führt dich durch:

* Modell-Authentifizierung (API-Schlüssel oder OAuth)
* Channel-Einrichtung (Telegram, WhatsApp, Discord usw.)
* Gateway-Token (automatisch erstellt)
* Daemon-Installation (systemd)

<div id="5-verify-the-gateway">
  ## 5) Gateway überprüfen
</div>

```bash
# Status überprüfen
openclaw status

# Service überprüfen
systemctl --user status openclaw-gateway.service

# Logs anzeigen
journalctl --user -u openclaw-gateway.service -f
```

<div id="6-access-the-dashboard">
  ## 6) Auf das Dashboard zugreifen
</div>

Standardmäßig bindet das Gateway nur an die Loopback-Schnittstelle. Um auf die Control UI zuzugreifen:

**Option A: SSH-Tunnel (empfohlen)**

```bash
# Von Ihrem lokalen Rechner
ssh -L 18789:localhost:18789 root@YOUR_DROPLET_IP

# Dann öffnen: http://localhost:18789
```

**Option B: Tailscale Serve (HTTPS, nur Loopback-Zugriff)**

```bash
# On the droplet
curl -fsSL https://tailscale.com/install.sh | sh
tailscale up

# Gateway für die Verwendung von Tailscale Serve konfigurieren
openclaw config set gateway.tailscale.mode serve
openclaw gateway restart
```

Öffne im Browser: `https://<magicdns>/`

Hinweise:

* Serve hält den Gateway nur über Loopback (localhost) erreichbar und authentifiziert über Tailscale-Identity-Header.
* Um stattdessen Token/Passwort zu erzwingen, setze `gateway.auth.allowTailscale: false` oder verwende `gateway.auth.mode: "password"`.

**Option C: Tailnet-Bind (kein Serve)**

```bash
openclaw config set gateway.bind tailnet
openclaw gateway restart
```

Öffne: `http://<tailscale-ip>:18789` (Token erforderlich).

<div id="7-connect-your-channels">
  ## 7) Kanäle verbinden
</div>

<div id="telegram">
  ### Telegram
</div>

```bash
openclaw pairing list telegram
openclaw pairing approve telegram <CODE>
```

<div id="whatsapp">
  ### WhatsApp
</div>

```bash
openclaw channels login whatsapp
# QR-Code scannen
```

Siehe [Channels](/de/channels) für weitere Anbieter.

***

<div id="optimizations-for-1gb-ram">
  ## Optimierungen für 1 GB RAM
</div>

Das $6-Droplet verfügt nur über 1 GB RAM. Damit alles reibungslos läuft:

<div id="add-swap-recommended">
  ### Swap-Speicher hinzufügen (empfohlen)
</div>

```bash
fallocate -l 2G /swapfile
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
echo '/swapfile none swap sw 0 0' >> /etc/fstab
```

<div id="use-a-lighter-model">
  ### Verwende ein leichteres Modell
</div>

Wenn du OOM-Fehler bekommst, solltest du Folgendes in Betracht ziehen:

* Verwende API-basierte Modelle (Claude, GPT) anstelle lokaler Modelle
* Setze `agents.defaults.model.primary` auf ein kleineres Modell

<div id="monitor-memory">
  ### Arbeitsspeicher überwachen
</div>

```bash
free -h
htop
```

***

<div id="persistence">
  ## Persistenz
</div>

Sämtliche Zustandsdaten liegen in:

* `~/.openclaw/` — Konfiguration, Zugangsdaten, Sitzungsdaten
* `~/.openclaw/workspace/` — Arbeitsbereich (SOUL.md, Memory, etc.)

Diese überstehen Neustarts. Lege regelmäßig Backups davon an:

```bash
tar -czvf openclaw-backup.tar.gz ~/.openclaw ~/.openclaw/workspace
```

***

<div id="oracle-cloud-free-alternative">
  ## Kostenlose Oracle-Cloud-Alternative
</div>

Oracle Cloud bietet **Always Free** ARM-Instanzen, die deutlich leistungsstärker sind als jede hier genannte kostenpflichtige Option — für 0 $/Monat.

| Was du erhältst | Spezifikationen |
|-----------------|-----------------|
| **4 OCPUs** | ARM Ampere A1 |
| **24GB RAM** | Mehr als genug |
| **200GB Speicher** | Block-Volume |
| **Für immer kostenlos** | Keine Kreditkartenabbuchungen |

**Einschränkungen:**

* Registrierung kann etwas hakelig sein (versuche es erneut, wenn sie fehlschlägt)
* ARM-Architektur — das meiste funktioniert, aber einige Binärdateien benötigen ARM-Builds

Die vollständige Setup-Anleitung findest du unter [Oracle Cloud](/de/platforms/oracle). Hinweise zur Registrierung und zur Fehlerbehebung im Anmeldeprozess findest du in diesem [Community-Leitfaden](https://gist.github.com/rssnyder/51e3cfedd730e7dd5f4a816143b25dbd).

***

<div id="troubleshooting">
  ## Fehlerbehebung
</div>

<div id="gateway-wont-start">
  ### Gateway startet nicht
</div>

```bash
openclaw gateway status
openclaw doctor --non-interactive
journalctl -u openclaw --no-pager -n 50
```

<div id="port-already-in-use">
  ### Port wird bereits verwendet
</div>

```bash
lsof -i :18789
kill <PID>
```

<div id="out-of-memory">
  ### Speicher erschöpft
</div>

```bash
# Speicher überprüfen
free -h

# Mehr Swap hinzufügen
# Oder auf $12/Monat-Droplet (2GB RAM) upgraden
```

***

<div id="see-also">
  ## Siehe auch
</div>

* [Hetzner-Anleitung](/de/platforms/hetzner) — günstiger, leistungsfähiger
* [Docker-Installation](/de/install/docker) — Container-basiertes Setup
* [Tailscale](/de/gateway/tailscale) — sicherer Fernzugriff
* [Konfiguration](/de/gateway/configuration) — vollständige Referenz der Konfiguration