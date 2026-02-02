---
title: macOS-VM
summary: "Führe OpenClaw in einer isolierten macOS-VM (Sandbox, lokal oder gehostet) aus, wenn du Isolation oder iMessage benötigst"
read_when:
  - Du möchtest OpenClaw von deiner Haupt-macOS-Umgebung isolieren
  - Du möchtest iMessage-Integration (BlueBubbles) in einer Sandbox
  - Du möchtest eine zurücksetzbare macOS-Umgebung, die du klonen kannst
  - Du möchtest lokale mit gehosteten macOS-VM-Optionen vergleichen
---

<div id="openclaw-on-macos-vms-sandboxing">
  # OpenClaw in macOS-VMs (Sandboxing)
</div>

<div id="recommended-default-most-users">
  ## Empfohlene Standardeinstellung (für die meisten Nutzer)
</div>

* **Kleiner Linux-VPS** für ein dauerhaft laufendes Gateway zu geringen Kosten. Siehe [VPS-Hosting](/de/vps).
* **Dedizierte Hardware** (Mac mini oder Linux-Rechner), wenn du vollständige Kontrolle und eine **Residential-IP** für Browser-Automatisierung willst. Viele Seiten blockieren Rechenzentrums-IPs, daher funktioniert lokales Surfen im Browser oft besser.
* **Hybrid:** Lass das Gateway auf einem günstigen VPS laufen und verbinde deinen Mac als **Knoten**, wenn du Browser-/UI-Automatisierung brauchst. Siehe [Nodes](/de/nodes) und [Gateway remote](/de/gateway/remote).

Verwende eine macOS-VM, wenn du speziell macOS-exklusive Funktionen (iMessage/BlueBubbles) benötigst oder eine strikte Isolation von deinem täglichen Mac willst.

<div id="macos-vm-options">
  ## Optionen für macOS-VMs
</div>

<div id="local-vm-on-your-apple-silicon-mac-lume">
  ### Lokale VM auf deinem Apple‑Silicon‑Mac (Lume)
</div>

Betreibe OpenClaw in einer sandbox‑basierten macOS‑VM auf deinem vorhandenen Apple‑Silicon‑Mac mit [Lume](https://cua.ai/docs/lume).

Das bietet dir:

* Vollständige, isolierte macOS‑Umgebung (dein Host bleibt sauber)
* iMessage‑Unterstützung über BlueBubbles (unter Linux/Windows nicht möglich)
* Sofortiges Zurücksetzen durch Klonen von VMs
* Keine zusätzlichen Hardware‑ oder Cloud‑Kosten

<div id="hosted-mac-providers-cloud">
  ### Gehostete Mac-Anbieter (Cloud)
</div>

Wenn du macOS in der Cloud möchtest, sind gehostete Mac-Anbieter ebenfalls geeignet:

* [MacStadium](https://www.macstadium.com/) (gehostete Macs)
* Andere gehostete Mac-Anbieter funktionieren ebenfalls; befolge deren VM- und SSH-Dokumentation

Sobald du SSH-Zugriff auf eine macOS-VM hast, fährst du unten bei Schritt 6 fort.

***

<div id="quick-path-lume-experienced-users">
  ## Schnellstart (Lume, erfahrene Nutzer)
</div>

1. Lume installieren
2. `lume create openclaw --os macos --ipsw latest`
3. Setup-Assistent abschließen, Entfernte Anmeldung (SSH) aktivieren
4. `lume run openclaw --no-display`
5. Per SSH einloggen, OpenClaw installieren, Kanäle konfigurieren
6. Fertig

***

<div id="what-you-need-lume">
  ## Voraussetzungen (Lume)
</div>

* Apple Silicon Mac (M1/M2/M3/M4)
* macOS Sequoia oder neuer auf dem Host
* ca. 60 GB freier Speicherplatz pro VM
* ca. 20 Minuten

***

<div id="1-install-lume">
  ## 1) Lume installieren
</div>

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/trycua/cua/main/libs/lume/scripts/install.sh)"
```

Wenn `~/.local/bin` nicht in deinem `PATH` enthalten ist:

```bash
echo 'export PATH="$PATH:$HOME/.local/bin"' >> ~/.zshrc && source ~/.zshrc
```

Überprüfen:

```bash
lume --version
```

Dokumentation: [Lume-Installation](https://cua.ai/docs/lume/guide/getting-started/installation)

***

<div id="2-create-the-macos-vm">
  ## 2) Erstelle die macOS-VM
</div>

```bash
lume create openclaw --os macos --ipsw latest
```

Damit wird macOS heruntergeladen und die VM erstellt. Ein VNC-Fenster öffnet sich automatisch.

Hinweis: Der Download kann je nach Verbindungsgeschwindigkeit eine Weile dauern.

***

<div id="3-complete-setup-assistant">
  ## 3) Einrichtungsassistenten abschließen
</div>

Im VNC-Fenster:

1. Sprache und Region auswählen
2. Apple-ID überspringen (oder anmelden, wenn du später iMessage verwenden möchtest)
3. Benutzerkonto erstellen (Benutzernamen und Passwort merken)
4. Alle optionalen Funktionen überspringen

Nachdem das Setup abgeschlossen ist, aktiviere SSH:

1. Systemeinstellungen öffnen → Allgemein → Freigaben
2. „Entfernte Anmeldung“ aktivieren

***

<div id="4-get-the-vms-ip-address">
  ## 4) IP-Adresse der VM abrufen
</div>

```bash
lume get openclaw
```

Suche die IP-Adresse (in der Regel `192.168.64.x`).

***

<div id="5-ssh-into-the-vm">
  ## 5) Per SSH auf die VM einloggen
</div>

```bash
ssh youruser@192.168.64.X
```

Ersetze `youruser` durch den Benutzer, den du erstellt hast, und die IP durch die IP-Adresse deiner VM.

***

<div id="6-install-openclaw">
  ## 6) Installiere OpenClaw
</div>

In der VM:

```bash
npm install -g openclaw@latest
openclaw onboard --install-daemon
```

Folge den Onboarding-Anweisungen, um deinen Modellanbieter (Anthropic, OpenAI, usw.) zu konfigurieren.

***

<div id="7-configure-channels">
  ## 7) Kanäle konfigurieren
</div>

Bearbeiten Sie die Konfigurationsdatei:

```bash
nano ~/.openclaw/openclaw.json
```

Füge deine Kanäle hinzu:

```json
{
  "channels": {
    "whatsapp": {
      "dmPolicy": "allowlist",
      "allowFrom": ["+15551234567"]
    },
    "telegram": {
      "botToken": "YOUR_BOT_TOKEN"
    }
  }
}
```

Melde dich anschließend bei WhatsApp an (QR-Code scannen):

```bash
openclaw channels login
```

***

<div id="8-run-the-vm-headlessly">
  ## 8) Die VM im Headless-Modus ausführen
</div>

Stoppe die VM und starte sie ohne Anzeige neu:

```bash
lume stop openclaw
lume run openclaw --no-display
```

Die VM läuft im Hintergrund. Der OpenClaw-Daemon hält das Gateway in Betrieb.

Um den Status zu prüfen:

```bash
ssh youruser@192.168.64.X "openclaw status"
```

***

<div id="bonus-imessage-integration">
  ## Bonus: iMessage-Integration
</div>

Das ist das Killer-Feature beim Betrieb auf macOS. Verwende [BlueBubbles](https://bluebubbles.app), um iMessage zu OpenClaw hinzuzufügen.

In der VM:

1. Lade BlueBubbles von bluebubbles.app herunter
2. Melde dich mit deiner Apple-ID an
3. Aktiviere die Web-API und setze ein Passwort
4. Richte die BlueBubbles-Webhooks auf deinen Gateway aus (Beispiel: `https://your-gateway-host:3000/bluebubbles-webhook?password=<password>`)

Füge Folgendes zu deiner OpenClaw-Konfiguration hinzu:

```json
{
  "channels": {
    "bluebubbles": {
      "serverUrl": "http://localhost:1234",
      "password": "your-api-password",
      "webhookPath": "/bluebubbles-webhook"
    }
  }
}
```

Starte das Gateway neu. Jetzt kann dein Agent iMessages senden und empfangen.

Vollständige Setup-Details: [BlueBubbles-Kanal](/de/channels/bluebubbles)

***

<div id="save-a-golden-image">
  ## Golden Image sichern
</div>

Bevor du weitere Anpassungen vornimmst, erstelle einen Snapshot deines sauberen Ausgangszustands:

```bash
lume stop openclaw
lume clone openclaw openclaw-golden
```

Zurücksetzen jederzeit möglich:

```bash
lume stop openclaw && lume delete openclaw
lume clone openclaw-golden openclaw
lume run openclaw --no-display
```

***

<div id="running-247">
  ## 24/7-Betrieb
</div>

Halte die VM dauerhaft am Laufen, indem du:

* deinen Mac eingesteckt lässt
* den Ruhezustand in Systemeinstellungen → Energie sparen deaktivierst
* bei Bedarf `caffeinate` verwendest

Für einen wirklich durchgängigen Betrieb solltest du einen dedizierten Mac mini oder einen kleinen VPS in Betracht ziehen. Siehe [VPS-Hosting](/de/vps).

***

<div id="troubleshooting">
  ## Fehlerbehebung
</div>

| Problem | Lösung |
|---------|--------|
| SSH-Verbindung zur VM nicht möglich | Überprüfe, ob „Remote Login“ in den Systemeinstellungen der VM aktiviert ist |
| VM-IP wird nicht angezeigt | Warte, bis die VM vollständig gestartet ist, und führe `lume get openclaw` erneut aus |
| Lume-Befehl nicht gefunden | Füge `~/.local/bin` zu deinem PATH hinzu |
| WhatsApp-QR-Code lässt sich nicht scannen | Stelle sicher, dass du in der VM (nicht auf dem Host) angemeldet bist, wenn du `openclaw channels login` ausführst |

***

<div id="related-docs">
  ## Verwandte Dokumente
</div>

* [VPS-Hosting](/de/vps)
* [Knoten](/de/nodes)
* [Gateway Remote](/de/gateway/remote)
* [BlueBubbles-Kanal](/de/channels/bluebubbles)
* [Lume-Schnellstart](https://cua.ai/docs/lume/guide/getting-started/quickstart)
* [Lume-CLI-Referenz](https://cua.ai/docs/lume/reference/cli-reference)
* [Unbeaufsichtigte VM-Einrichtung](https://cua.ai/docs/lume/guide/fundamentals/unattended-setup) (fortgeschritten)
* [Docker-Sandboxing](/de/install/docker) (alternative Isolierungsvariante)