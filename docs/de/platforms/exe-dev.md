---
title: Exe Dev
summary: "OpenClaw Gateway auf exe.dev ausführen (VM + HTTPS-Proxy) für Remotezugriff"
read_when:
  - Du möchtest einen günstigen, dauerhaft laufenden Linux-Host für das Gateway
  - Du möchtest Remotezugriff auf die Control UI, ohne selbst einen VPS zu betreiben
---

<div id="exedev">
  # exe.dev
</div>

Ziel: OpenClaw Gateway läuft auf einer exe.dev-VM und ist von deinem Laptop aus unter `https://<vm-name>.exe.xyz` erreichbar.

Diese Seite geht vom Standard-Image **exeuntu** von exe.dev aus. Wenn du eine andere Distribution gewählt hast, ordne die Pakete entsprechend zu.

<div id="beginner-quick-path">
  ## Schnellstart für Einsteiger:innen
</div>

1. [https://exe.new/openclaw](https://exe.new/openclaw)
2. Gib bei Bedarf deinen Auth‑Key/Token ein
3. Klicke auf „Agent“ neben deiner VM und warte...
4. ???
5. Profit

<div id="what-you-need">
  ## Was Sie benötigen
</div>

* exe.dev-Konto
* Zugriff per `ssh exe.dev` auf die virtuellen Maschinen von [exe.dev](https://exe.dev) (optional)

<div id="automated-install-with-shelley">
  ## Automatische Installation mit Shelley
</div>

Shelley, der Agent von [exe.dev](https://exe.dev), kann OpenClaw mit unserem Prompt
sofort installieren. Der verwendete Prompt lautet wie folgt:

```
Richten Sie OpenClaw (https://docs.openclaw.ai/install) auf dieser VM ein. Verwenden Sie die Flags non-interactive und accept-risk für das openclaw-Onboarding. Fügen Sie die bereitgestellte Authentifizierung oder das Token nach Bedarf hinzu. Konfigurieren Sie nginx so, dass vom Standard-Port 18789 zum Root-Verzeichnis in der standardmäßig aktivierten Site-Konfiguration weitergeleitet wird, und stellen Sie sicher, dass WebSocket-Unterstützung aktiviert ist. Die Kopplung erfolgt über „openclaw devices list" und „openclaw device approve <request id>". Stellen Sie sicher, dass das Dashboard anzeigt, dass der Zustand von OpenClaw OK ist. exe.dev übernimmt für uns die Weiterleitung von Port 8000 zu Port 80/443 und HTTPS, sodass die endgültige erreichbare Adresse <vm-name>.exe.xyz sein sollte, ohne Port-Angabe.
```

<div id="manual-installation">
  ## Manuelle Installation
</div>

<div id="1-create-the-vm">
  ## 1) Erstelle die VM
</div>

Von deinem Gerät aus:

```bash
ssh exe.dev new 
```

Stelle anschließend die Verbindung her:

```bash
ssh <vm-name>.exe.xyz
```

Tipp: Verwende diese VM **zustandsbehaftet**. OpenClaw speichert seinen Zustand unter `~/.openclaw/` und `~/.openclaw/workspace/`.

<div id="2-install-prerequisites-on-the-vm">
  ## 2) Voraussetzungen installieren (auf der VM)
</div>

```bash
sudo apt-get update
sudo apt-get install -y git curl jq ca-certificates openssl
```

<div id="3-install-openclaw">
  ## 3) OpenClaw installieren
</div>

Führen Sie das OpenClaw-Installationsskript aus:

```bash
curl -fsSL https://openclaw.bot/install.sh | bash
```

<div id="4-setup-nginx-to-proxy-openclaw-to-port-8000">
  ## 4) Richte nginx ein, um als Proxy für OpenClaw auf Port 8000 zu dienen
</div>

Bearbeite `/etc/nginx/sites-enabled/default` wie folgt:

```
server {
    listen 80 default_server;
    listen [::]:80 default_server;
    listen 8000;
    listen [::]:8000;

    server_name _;

    location / {
        proxy_pass http://127.0.0.1:18789;
        proxy_http_version 1.1;

        # WebSocket support
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";

        # Standard proxy headers
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # Timeout-Einstellungen für langlebige Verbindungen
        proxy_read_timeout 86400s;
        proxy_send_timeout 86400s;
    }
}
```

<div id="5-access-openclaw-and-grant-privileges">
  ## 5) Auf OpenClaw zugreifen und Berechtigungen erteilen
</div>

Greife auf `https://<vm-name>.exe.xyz/?token=YOUR-TOKEN-FROM-TERMINAL` zu. Bestätige
Geräte mit `openclaw devices list` und `openclaw device approve`. Wenn du dir unsicher bist,
nutze Shelley direkt im Browser!

<div id="remote-access">
  ## Remotezugriff
</div>

Remotezugriff erfolgt über die Authentifizierung von [exe.dev](https://exe.dev). Standardmäßig wird der HTTP-Datenverkehr von Port 8000 mit E-Mail-Authentifizierung an `https://<vm-name>.exe.xyz` weitergeleitet.

<div id="updating">
  ## Aktualisieren
</div>

```bash
npm i -g openclaw@latest
openclaw doctor
openclaw gateway restart
openclaw health
```

Anleitung: [Aktualisierung](/de/install/updating)
