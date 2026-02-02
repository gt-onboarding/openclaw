---
title: Raspberry Pi
summary: "OpenClaw auf dem Raspberry Pi (kostengünstiges Self-Hosting-Setup)"
read_when:
  - Einrichtung von OpenClaw auf einem Raspberry Pi
  - Betrieb von OpenClaw auf ARM-Geräten
  - Aufbau einer günstigen, dauerhaft laufenden persönlichen KI
---

<div id="openclaw-on-raspberry-pi">
  # OpenClaw auf dem Raspberry Pi
</div>

<div id="goal">
  ## Ziel
</div>

Einen dauerhaft laufenden OpenClaw Gateway auf einem Raspberry Pi mit einmaligen Kosten von **ca. 35–80 US‑$** (keine monatlichen Gebühren) betreiben.

Perfekt für:

* 24/7 persönlichen KI-Assistenten
* Smart-Home-Automatisierungszentrale
* stromsparenden, stets verfügbaren Telegram-/WhatsApp-Bot

<div id="hardware-requirements">
  ## Hardware-Anforderungen
</div>

| Pi-Modell | RAM | Funktioniert? | Hinweise |
|----------|-----|--------|-------|
| **Pi 5** | 4GB/8GB | ✅ Am besten | Am schnellsten, empfohlen |
| **Pi 4** | 4GB | ✅ Gut | Guter Kompromiss für die meisten Nutzer |
| **Pi 4** | 2GB | ✅ OK | Funktioniert, Swap hinzufügen |
| **Pi 4** | 1GB | ⚠️ Knapp | Möglich mit Swap und Minimal-Konfiguration |
| **Pi 3B+** | 1GB | ⚠️ Langsam | Funktioniert, aber träge |
| **Pi Zero 2 W** | 512MB | ❌ | Nicht empfohlen |

**Mindestanforderungen:** 1GB RAM, 1 Kern, 500MB Speicherplatz\
**Empfohlen:** 2GB+ RAM, 64-Bit-OS, 16GB+ SD-Karte (oder USB-SSD)

<div id="what-youll-need">
  ## Was Sie benötigen
</div>

* Raspberry Pi 4 oder 5 (2 GB oder mehr empfohlen)
* MicroSD-Karte (16 GB oder größer) oder USB-SSD (bessere Leistung)
* Stromversorgung (offizielles Pi-Netzteil empfohlen)
* Netzwerkverbindung (Ethernet oder WLAN)
* ~30 Minuten

<div id="1-flash-the-os">
  ## 1) Betriebssystem flashen
</div>

Verwende **Raspberry Pi OS Lite (64-bit)** – kein Desktop nötig für einen Headless-Server.

1. Lade den [Raspberry Pi Imager](https://www.raspberrypi.com/software/) herunter
2. Wähle als OS: **Raspberry Pi OS Lite (64-bit)**
3. Klicke auf das Zahnrad-Symbol (⚙️), um vorzukonfigurieren:
   * Hostname festlegen: `gateway-host`
   * SSH aktivieren
   * Benutzername/Passwort setzen
   * WLAN konfigurieren (falls du kein Ethernet verwendest)
4. Auf deine SD-Karte / dein USB-Laufwerk flashen
5. SD-Karte/USB-Laufwerk einlegen und den Pi starten

<div id="2-connect-via-ssh">
  ## 2) Über SSH verbinden
</div>

```bash
ssh user@gateway-host
# oder verwende die IP-Adresse
ssh user@192.168.x.x
```

<div id="3-system-setup">
  ## 3) Systemeinrichtung
</div>

```bash
# System aktualisieren
sudo apt update && sudo apt upgrade -y

# Erforderliche Pakete installieren
sudo apt install -y git curl build-essential

# Zeitzone setzen (wichtig für cron/Erinnerungen)
sudo timedatectl set-timezone America/Chicago  # An Ihre Zeitzone anpassen
```

<div id="4-install-nodejs-22-arm64">
  ## 4) Node.js 22 installieren (ARM64)
</div>

```bash
# Node.js über NodeSource installieren
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
sudo apt install -y nodejs

# Überprüfen
node --version  # Sollte v22.x.x anzeigen
npm --version
```

<div id="5-add-swap-important-for-2gb-or-less">
  ## 5) Swap hinzufügen (wichtig für Systeme mit 2 GB RAM oder weniger)
</div>

Swap hilft, Abstürze durch Speichermangel (Out-of-Memory) zu vermeiden:

```bash
# Create 2GB swap file
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Make permanent
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab

# Für wenig RAM optimieren (Swappiness reduzieren)
echo 'vm.swappiness=10' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

<div id="6-install-openclaw">
  ## 6) OpenClaw installieren
</div>

<div id="option-a-standard-install-recommended">
  ### Option A: Standard-Installation (empfohlen)
</div>

```bash
curl -fsSL https://openclaw.bot/install.sh | bash
```

<div id="option-b-hackable-install-for-tinkering">
  ### Option B: Hackbare Installation (zum Tüfteln)
</div>

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
npm install
npm run build
npm link
```

Die hackbare Installation verschafft dir direkten Zugriff auf Logs und Quellcode – hilfreich beim Debuggen von ARM-spezifischen Problemen.

<div id="7-run-onboarding">
  ## 7) Onboarding durchführen
</div>

```bash
openclaw onboard --install-daemon
```

Folge dem Einrichtungsassistenten:

1. **Gateway-Modus:** Lokal
2. **Auth:** API-Schlüssel werden empfohlen (OAuth kann auf einem headless-Raspberry-Pi unzuverlässig sein)
3. **Kanäle:** Telegram ist am einfachsten für den Einstieg
4. **Daemon:** Ja (systemd)

<div id="8-verify-installation">
  ## 8) Installation überprüfen
</div>

```bash
# Status überprüfen
openclaw status

# Dienst überprüfen
sudo systemctl status openclaw

# Logs anzeigen
journalctl -u openclaw -f
```

<div id="9-access-the-dashboard">
  ## 9) Auf das Dashboard zugreifen
</div>

Da der Pi headless läuft, greife über einen SSH-Tunnel darauf zu:

```bash
# Von Ihrem Laptop/Desktop
ssh -L 18789:localhost:18789 user@gateway-host

# Then open in browser
open http://localhost:18789
```

Oder verwende Tailscale für dauerhaften Zugriff:

```bash
# On the Pi
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up

# Konfiguration aktualisieren
openclaw config set gateway.bind tailnet
sudo systemctl restart openclaw
```

***

<div id="performance-optimizations">
  ## Leistungsoptimierungen
</div>

<div id="use-a-usb-ssd-huge-improvement">
  ### Verwende eine USB-SSD (großer Leistungsgewinn)
</div>

SD-Karten sind langsam und verschleißen. Eine USB-SSD verbessert die Leistung deutlich:

```bash
# Prüfen, ob von USB gebootet wird
lsblk
```

Siehe die [Pi-USB-Boot-Anleitung](https://www.raspberrypi.com/documentation/computers/raspberry-pi.html#usb-mass-storage-boot) zur Einrichtung.

<div id="reduce-memory-usage">
  ### Speichernutzung reduzieren
</div>

```bash
# GPU-Speicherzuweisung deaktivieren (Headless-Betrieb)
echo 'gpu_mem=16' | sudo tee -a /boot/config.txt

# Disable Bluetooth if not needed
sudo systemctl disable bluetooth
```

<div id="monitor-resources">
  ### Systemressourcen überwachen
</div>

```bash
# Check memory
free -h

# CPU-Temperatur prüfen
vcgencmd measure_temp

# Live monitoring
htop
```

***

<div id="arm-specific-notes">
  ## ARM-spezifische Hinweise
</div>

<div id="binary-compatibility">
  ### Binärkompatibilität
</div>

Die meisten OpenClaw-Funktionen laufen auf ARM64, aber einige externe Binärdateien benötigen möglicherweise eigene ARM-Builds:

| Tool | ARM64-Status | Hinweise |
|------|--------------|---------|
| Node.js | ✅ | Funktioniert hervorragend |
| WhatsApp (Baileys) | ✅ | Reines JS, keine Probleme |
| Telegram | ✅ | Reines JS, keine Probleme |
| gog (Gmail CLI) | ⚠️ | Auf ARM-Release prüfen |
| Chromium (browser) | ✅ | `sudo apt install chromium-browser` |

Wenn ein Skill fehlschlägt, prüfe, ob seine Binärdatei einen ARM-Build hat. Viele Go-/Rust-Tools haben einen; einige nicht.

<div id="32-bit-vs-64-bit">
  ### 32-Bit vs. 64-Bit
</div>

**Verwende immer ein 64-Bit-Betriebssystem.** Node.js und viele moderne Tools setzen dies voraus. Überprüfe das mit:

```bash
uname -m
# Sollte aarch64 (64-bit) anzeigen, nicht armv7l (32-bit)
```

***

<div id="recommended-model-setup">
  ## Empfohlene Modellkonfiguration
</div>

Da der Pi nur als Gateway fungiert (die Modelle laufen in der Cloud), solltest du API-basierte Modelle verwenden:

```json
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "anthropic/claude-sonnet-4-20250514",
        "fallbacks": ["openai/gpt-4o-mini"]
      }
    }
  }
}
```

**Versuche gar nicht erst, lokale LLMs auf einem Raspberry Pi auszuführen** — selbst kleine Modelle sind zu langsam. Überlass Claude/GPT die Schwerstarbeit.

***

<div id="auto-start-on-boot">
  ## Automatischer Start beim Systemstart
</div>

Der Onboarding-Assistent richtet das ein, aber um es zu überprüfen:

```bash
# Prüfen, ob der Dienst aktiviert ist
sudo systemctl is-enabled openclaw

# Falls nicht, aktivieren
sudo systemctl enable openclaw

# Beim Booten starten
sudo systemctl start openclaw
```

***

<div id="troubleshooting">
  ## Fehlerbehebung
</div>

<div id="out-of-memory-oom">
  ### Out-of-Memory-Fehler (OOM)
</div>

```bash
# Speicher überprüfen
free -h

# Mehr Swap hinzufügen (siehe Schritt 5)
# Oder die auf dem Pi laufenden Dienste reduzieren
```

<div id="slow-performance">
  ### Schlechte Performance
</div>

* Verwende eine USB-SSD anstelle einer SD-Karte
* Deaktiviere nicht benötigte Dienste: `sudo systemctl disable cups bluetooth avahi-daemon`
* Prüfe CPU-Throttling: `vcgencmd get_throttled` (sollte `0x0` zurückgeben)

<div id="service-wont-start">
  ### Dienst lässt sich nicht starten
</div>

```bash
# Logs prüfen
journalctl -u openclaw --no-pager -n 100

# Häufige Lösung: Rebuild
cd ~/openclaw  # bei hackable install
npm run build
sudo systemctl restart openclaw
```

<div id="arm-binary-issues">
  ### ARM-Binary-Probleme
</div>

Wenn ein Skill mit „exec format error“ fehlschlägt:

1. Prüfe, ob das Binary einen ARM64-Build besitzt
2. Versuche, aus dem Quellcode zu bauen
3. Oder verwende einen Docker-Container mit ARM-Unterstützung

<div id="wifi-drops">
  ### WLAN-Abbrüche
</div>

Für headless Raspberry Pis mit WLAN:

```bash
# WiFi-Energieverwaltung deaktivieren
sudo iwconfig wlan0 power off

# Dauerhaft einrichten
echo 'wireless-power off' | sudo tee -a /etc/network/interfaces
```

***

<div id="cost-comparison">
  ## Kostenvergleich
</div>

| Setup | Einmalkosten | Monatliche Kosten | Hinweise |
|-------|---------------|--------------|-------|
| **Pi 4 (2GB)** | ~45 $ | 0 $ | + Stromkosten (~5 $/Jahr) |
| **Pi 4 (4GB)** | ~55 $ | 0 $ | Empfohlen |
| **Pi 5 (4GB)** | ~60 $ | 0 $ | Beste Leistung |
| **Pi 5 (8GB)** | ~80 $ | 0 $ | Overkill, aber zukunftssicher |
| DigitalOcean | 0 $ | 6 $/Monat | 72 $/Jahr |
| Hetzner | 0 $ | 3,79 €/Monat | ~50 $/Jahr |

**Amortisation:** Ein Pi amortisiert sich nach ca. 6–12 Monaten im Vergleich zu einem Cloud-VPS.

<div id="see-also">
  ## Siehe auch
</div>

* [Linux-Anleitung](/de/platforms/linux) — allgemeine Linux-Konfiguration
* [DigitalOcean-Anleitung](/de/platforms/digitalocean) — Cloud-Alternative
* [Hetzner-Anleitung](/de/platforms/hetzner) — Docker-Konfiguration
* [Tailscale](/de/gateway/tailscale) — Remotezugriff
* [Knoten](/de/nodes) — verbinde deinen Laptop oder dein Smartphone mit dem Pi-Gateway