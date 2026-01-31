---
title: Oracle
summary: "OpenClaw in Oracle Cloud (Always Free-ARM)"
read_when:
  - OpenClaw in Oracle Cloud einrichten
  - Nach günstigem VPS-Hosting für OpenClaw suchen
  - OpenClaw rund um die Uhr auf einem kleinen Server betreiben möchtest
---

<div id="openclaw-on-oracle-cloud-oci">
  # OpenClaw in Oracle Cloud (OCI)
</div>

<div id="goal">
  ## Ziel
</div>

Einen dauerhaft laufenden OpenClaw Gateway auf Oracles **Always Free** ARM-Tarif betreiben.

Oracles kostenloser Free-Tarif kann sehr gut zu OpenClaw passen (insbesondere, wenn du bereits ein OCI-Konto hast), bringt aber einige Nachteile mit sich:

* ARM-Architektur (das meiste funktioniert, aber einige Binärdateien sind eventuell nur für x86 verfügbar)
* Kapazität und Anmeldung können etwas heikel sein

<div id="cost-comparison-2026">
  ## Kostenvergleich (2026)
</div>

| Anbieter | Plan | Spezifikationen | Preis/Monat | Hinweise |
|----------|------|-----------------|-------------|----------|
| Oracle Cloud | Always Free ARM | bis zu 4 OCPU, 24GB RAM | $0 | ARM, begrenzte Kapazität |
| Hetzner | CX22 | 2 vCPU, 4GB RAM | ~ $4 | Günstigste kostenpflichtige Option |
| DigitalOcean | Basic | 1 vCPU, 1GB RAM | $6 | Einfache UI, gute Dokumentation |
| Vultr | Cloud Compute | 1 vCPU, 1GB RAM | $6 | Viele Regionen |
| Linode | Nanode | 1 vCPU, 1GB RAM | $5 | Jetzt Teil von Akamai |

***

<div id="prerequisites">
  ## Voraussetzungen
</div>

* Oracle-Cloud-Konto ([Registrierung](https://www.oracle.com/cloud/free/)) – siehe [Community-Anleitung zur Registrierung](https://gist.github.com/rssnyder/51e3cfedd730e7dd5f4a816143b25dbd), falls Probleme auftreten
* Tailscale-Konto (kostenlos unter [tailscale.com](https://tailscale.com))
* etwa 30 Minuten

<div id="1-create-an-oci-instance">
  ## 1) OCI-Instanz erstellen
</div>

1. Melde dich bei der [Oracle Cloud Console](https://cloud.oracle.com/) an
2. Navigiere zu **Compute → Instances → Create Instance**
3. Konfiguriere:
   * **Name:** `openclaw`
   * **Image:** Ubuntu 24.04 (aarch64)
   * **Shape:** `VM.Standard.A1.Flex` (Ampere ARM)
   * **OCPUs:** 2 (oder bis zu 4)
   * **Memory:** 12 GB (oder bis zu 24 GB)
   * **Boot-Volume:** 50 GB (bis zu 200 GB frei)
   * **SSH-Schlüssel:** Füge deinen öffentlichen Schlüssel hinzu
4. Klicke auf **Create**
5. Notiere dir die öffentliche IP-Adresse

**Tipp:** Wenn das Erstellen der Instanz mit „Out of capacity“ fehlschlägt, versuche eine andere Availability Domain oder probiere es später erneut. Die Kapazität des Free Tiers ist begrenzt.

<div id="2-connect-and-update">
  ## 2) Verbinden und aktualisieren
</div>

```bash
# Verbindung über öffentliche IP herstellen
ssh ubuntu@YOUR_PUBLIC_IP

# Update system
sudo apt update && sudo apt upgrade -y
sudo apt install -y build-essential
```

**Hinweis:** `build-essential` ist für die ARM-spezifische Kompilierung einiger Abhängigkeiten erforderlich.

<div id="3-configure-user-and-hostname">
  ## 3) Benutzer und Hostname konfigurieren
</div>

```bash
# Hostname festlegen
sudo hostnamectl set-hostname openclaw

# Passwort für ubuntu-Benutzer festlegen
sudo passwd ubuntu

# Lingering aktivieren (hält Benutzerdienste nach dem Abmelden am Laufen)
sudo loginctl enable-linger ubuntu
```

<div id="4-install-tailscale">
  ## 4) Tailscale installieren
</div>

```bash
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up --ssh --hostname=openclaw
```

Dadurch wird Tailscale SSH aktiviert, sodass du dich von jedem Gerät in deinem Tailnet mit `ssh openclaw` verbinden kannst – eine öffentliche IP-Adresse ist nicht nötig.

Prüfen:

```bash
tailscale status
```

**Ab jetzt stellst du die Verbindung über Tailscale her:** `ssh ubuntu@openclaw` (oder nutze die Tailscale-IP).

<div id="5-install-openclaw">
  ## 5) OpenClaw installieren
</div>

```bash
curl -fsSL https://openclaw.bot/install.sh | bash
source ~/.bashrc
```

Wenn du die Aufforderung „How do you want to hatch your bot?“ siehst, wähle **„Do this later“**.

> Hinweis: Wenn du auf ARM-native Build-Probleme stößt, installiere zuerst Systempakete (z. B. `sudo apt install -y build-essential`), bevor du Homebrew verwendest.

<div id="6-configure-gateway-loopback-token-auth-and-enable-tailscale-serve">
  ## 6) Gateway konfigurieren (Loopback + Token-Authentifizierung) und Tailscale Serve aktivieren
</div>

Verwende standardmäßig die Token-Authentifizierung. Sie ist vorhersehbar und erspart dir, irgendwelche „insecure auth“-Flags in der Control UI setzen zu müssen.

```bash
# Keep the Gateway private on the VM
openclaw config set gateway.bind loopback

# Require auth for the Gateway + Control UI
openclaw config set gateway.auth.mode token
openclaw doctor --generate-gateway-token

# Über Tailscale Serve bereitstellen (HTTPS + Tailnet-Zugriff)
openclaw config set gateway.tailscale.mode serve
openclaw config set gateway.trustedProxies '["127.0.0.1"]'

systemctl --user restart openclaw-gateway
```

<div id="7-verify">
  ## 7) Prüfen
</div>

```bash
# Check version
openclaw --version

# Check daemon status
systemctl --user status openclaw-gateway

# Tailscale Serve prüfen
tailscale serve status

# Test local response
curl http://localhost:18789
```

<div id="8-lock-down-vcn-security">
  ## 8) VCN-Sicherheit absichern
</div>

Nachdem jetzt alles funktioniert, sicherst du das VCN so ab, dass jeglicher Datenverkehr außer Tailscale blockiert wird. Das Virtual Cloud Network von OCI fungiert als Firewall an der Netzwerkgrenze — Datenverkehr wird blockiert, bevor er deine Instanz erreicht.

1. Wechsle in der OCI-Konsole zu **Networking → Virtual Cloud Networks**
2. Klicke auf dein VCN → **Security Lists** → Default Security List
3. **Entferne** alle Ingress-Regeln außer:
   * `0.0.0.0/0 UDP 41641` (Tailscale)
4. Belasse die Standard-Egress-Regeln unverändert (erlaube sämtlichen ausgehenden Datenverkehr)

Dadurch werden SSH auf Port 22, HTTP, HTTPS und sämtlicher weiterer Datenverkehr an der Netzwerkgrenze blockiert. Ab jetzt kannst du nur noch über Tailscale eine Verbindung herstellen.

***

<div id="access-the-control-ui">
  ## Auf die Control UI zugreifen
</div>

Von jedem Gerät in Ihrem Tailscale-Netzwerk aus:

```
https://openclaw.<tailnet-name>.ts.net/
```

Ersetze `<tailnet-name>` durch deinen Tailnet-Namen (sichtbar in `tailscale status`).

Kein SSH-Tunnel erforderlich. Tailscale bietet:

* HTTPS-Verschlüsselung (automatische Zertifikate)
* Authentifizierung über die Tailscale-Identität
* Zugriff von jedem Gerät in deinem Tailnet (Laptop, Telefon, usw.)

***

<div id="security-vcn-tailscale-recommended-baseline">
  ## Sicherheit: VCN + Tailscale (empfohlene Baseline)
</div>

Wenn die VCN abgesichert ist (nur UDP 41641 ist offen) und der Gateway an Loopback gebunden ist, erhältst du eine starke Defense-in-Depth-Strategie: Öffentlicher Traffic wird an der Netzgrenze blockiert, und Admin-Zugriff erfolgt über dein Tailnet.

Dieses Setup eliminiert häufig die *Notwendigkeit* zusätzlicher hostbasierter Firewall-Regeln nur zum Abblocken internetweiter SSH-Brute-Force-Angriffe – aber du solltest das Betriebssystem trotzdem aktuell halten, `openclaw security audit` ausführen und sicherstellen, dass du nicht versehentlich auf öffentlichen Interfaces lauscht.

<div id="whats-already-protected">
  ### Was bereits geschützt ist
</div>

| Traditioneller Schritt | Erforderlich? | Warum |
|------------------|---------|-----|
| UFW-Firewall | Nein | VCN blockiert den Datenverkehr, bevor er die Instanz erreicht |
| fail2ban | Nein | Keine Brute-Force-Angriffe, wenn Port 22 auf VCN-Ebene blockiert ist |
| sshd-Hardening | Nein | Tailscale SSH verwendet sshd nicht |
| Root-Login deaktivieren | Nein | Tailscale verwendet Tailscale-Identitäten statt Systembenutzern |
| SSH-Authentifizierung nur mit Schlüssel | Nein | Tailscale authentifiziert über das Tailnet |
| IPv6-Hardening | In der Regel nicht | Hängt von deinen VCN-/Subnetz-Einstellungen ab; prüfe, was tatsächlich zugewiesen bzw. exponiert ist |

<div id="still-recommended">
  ### Nach wie vor empfohlen
</div>

* **Berechtigungen für Zugangsdaten:** `chmod 700 ~/.openclaw`
* **Sicherheitsaudit:** `openclaw security audit`
* **Systemupdates:** `sudo apt update && sudo apt upgrade` regelmäßig ausführen
* **Tailscale überwachen:** Geräte in der [Tailscale Admin-Konsole](https://login.tailscale.com/admin) prüfen

<div id="verify-security-posture">
  ### Sicherheitslage überprüfen
</div>

```bash
# Bestätigen, dass keine öffentlichen Ports horchen
sudo ss -tlnp | grep -v '127.0.0.1\|::1'

# Überprüfen, ob Tailscale SSH aktiv ist
tailscale status | grep -q 'offers: ssh' && echo "Tailscale SSH active"

# Optional: sshd vollständig deaktivieren
sudo systemctl disable --now ssh
```

***

<div id="fallback-ssh-tunnel">
  ## Fallback: SSH-Tunnel
</div>

Wenn Tailscale Serve nicht funktioniert, verwenden Sie einen SSH-Tunnel:

```bash
# Von Ihrem lokalen Rechner (über Tailscale)
ssh -L 18789:127.0.0.1:18789 ubuntu@openclaw
```

Öffne anschließend `http://localhost:18789`.

***

<div id="troubleshooting">
  ## Fehlerbehebung
</div>

<div id="instance-creation-fails-out-of-capacity">
  ### Instanzerstellung schlägt fehl („Out of capacity“)
</div>

ARM-Instanzen im Free-Tier sind stark nachgefragt. Versuche Folgendes:

* Andere Availability Domain wählen
* Erneut während verkehrsarmer Zeiten (früher Morgen) versuchen
* Beim Auswählen der Shape den Filter „Always Free“ verwenden

<div id="tailscale-wont-connect">
  ### Tailscale verbindet sich nicht
</div>

```bash
# Status überprüfen
sudo tailscale status

# Neu authentifizieren
sudo tailscale up --ssh --hostname=openclaw --reset
```

<div id="gateway-wont-start">
  ### Gateway startet nicht
</div>

```bash
openclaw gateway status
openclaw doctor --non-interactive
journalctl --user -u openclaw-gateway -n 50
```

<div id="cant-reach-control-ui">
  ### Control UI ist nicht erreichbar
</div>

```bash
# Überprüfen, ob Tailscale Serve läuft
tailscale serve status

# Check gateway is listening
curl http://localhost:18789

# Restart if needed
systemctl --user restart openclaw-gateway
```

<div id="arm-binary-issues">
  ### ARM-Binary-Probleme
</div>

Einige Tools werden möglicherweise nicht als ARM-Builds bereitgestellt. Überprüfe:

```bash
uname -m  # Sollte aarch64 ausgeben
```

Die meisten npm-Pakete funktionieren einwandfrei. Bei Binaries solltest du nach Releases für `linux-arm64` oder `aarch64` suchen.

***

<div id="persistence">
  ## Persistenz
</div>

Der gesamte Zustand wird gespeichert in:

* `~/.openclaw/` — Konfiguration, Zugangsdaten, Sitzungsdaten
* `~/.openclaw/workspace/` — arbeitsbereich (SOUL.md, Memory, Artefakte)

Sichere diese Verzeichnisse regelmäßig:

```bash
tar -czvf openclaw-backup.tar.gz ~/.openclaw ~/.openclaw/workspace
```

***

<div id="see-also">
  ## Siehe auch
</div>

* [Gateway-Fernzugriff](/de/gateway/remote) — weitere Remote-Zugriffsmuster
* [Tailscale-Integration](/de/gateway/tailscale) — vollständige Tailscale-Dokumentation
* [Gateway-Konfiguration](/de/gateway/configuration) — alle Konfigurationsoptionen
* [DigitalOcean-Anleitung](/de/platforms/digitalocean) — falls Sie einen kostenpflichtigen, aber einfacheren Einstieg bevorzugen
* [Hetzner-Anleitung](/de/platforms/hetzner) — Docker-basierte Alternative