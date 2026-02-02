---
title: Hetzner
summary: "OpenClaw Gateway 24/7 auf einem günstigen Hetzner-VPS (Docker) mit dauerhaftem Zustand und eingebauten Binaries betreiben"
read_when:
  - Du möchtest OpenClaw 24/7 auf einem Cloud-VPS ausführen (nicht auf deinem Laptop)
  - Du möchtest ein produktionsreifes, durchgehend verfügbares Gateway auf deinem eigenen VPS
  - Du möchtest die vollständige Kontrolle über Persistenz, Binaries und Neustartverhalten
  - Du betreibst OpenClaw in Docker auf Hetzner oder einem ähnlichen Anbieter
---

<div id="openclaw-on-hetzner-docker-production-vps-guide">
  # OpenClaw auf Hetzner (Docker, Leitfaden für Produktions-VPS)
</div>

<div id="goal">
  ## Ziel
</div>

Einen persistenten OpenClaw Gateway auf einem Hetzner-VPS mit Docker betreiben, mit dauerhaftem Zustand, eingebetteten Binaries und sicherem Neustartverhalten.

Wenn du „OpenClaw 24/7 für ~5 $“ möchtest, ist dies das einfachste und zuverlässigste Setup.
Die Hetzner-Preise ändern sich; wähle den kleinsten Debian/Ubuntu-VPS und skaliere nach oben, wenn du in OOMs läufst.

<div id="what-are-we-doing-simple-terms">
  ## Was machen wir (in einfachen Worten)?
</div>

* Einen kleinen Linux-Server mieten (Hetzner VPS)
* Docker installieren (isolierte Anwendungs-Runtime)
* Das OpenClaw Gateway in Docker starten
* `~/.openclaw` + `~/.openclaw/workspace` dauerhaft auf dem Host speichern (bleibt über Neustarts/Rebuilds erhalten)
* Von deinem Laptop aus über einen SSH-Tunnel auf die Control UI zugreifen

Auf das Gateway kannst du zugreifen über:

* SSH-Port-Forwarding von deinem Laptop
* Direkte Port-Freigabe, wenn du Firewalling und Tokens selbst verwaltest

Diese Anleitung geht von Ubuntu oder Debian auf Hetzner aus.\
Wenn du einen anderen Linux-VPS nutzt, ordne die Pakete entsprechend zu.
Für den generischen Docker-Flow siehe [Docker](/de/install/docker).

***

<div id="quick-path-experienced-operators">
  ## Schnellpfad (erfahrene Operatoren)
</div>

1. Hetzner-VPS bereitstellen
2. Docker installieren
3. OpenClaw-Repository klonen
4. Persistente Host-Verzeichnisse erstellen
5. `.env` und `docker-compose.yml` konfigurieren
6. Benötigte Binaries in das Image einbacken
7. `docker compose up -d`
8. Persistenz und Gateway-Zugriff überprüfen

***

<div id="what-you-need">
  ## Was du benötigst
</div>

* Hetzner-VPS mit Root-Zugriff
* SSH-Zugriff von deinem Laptop
* Grundkenntnisse in SSH + Copy/Paste
* ca. 20 Minuten
* Docker und Docker Compose
* Auth-Zugangsdaten für Modelle
* Optionale Anbieter-Zugangsdaten
  * WhatsApp-QR-Code
  * Telegram-Bot-Token
  * Gmail-OAuth

***

<div id="1-provision-the-vps">
  ## 1) VPS bereitstellen
</div>

Stelle einen Ubuntu- oder Debian-VPS-Server bei Hetzner bereit.

Verbinde dich als root-Benutzer:

```bash
ssh root@IHRE_VPS_IP
```

Diese Anleitung geht davon aus, dass dein VPS zustandsbehaftet ist.
Behandle ihn nicht als Wegwerf-Infrastruktur.

***

<div id="2-install-docker-on-the-vps">
  ## 2) Docker installieren (auf dem VPS)
</div>

```bash
apt-get update
apt-get install -y git curl ca-certificates
curl -fsSL https://get.docker.com | sh
```

Prüfen:

```bash
docker --version
docker compose version
```

***

<div id="3-clone-the-openclaw-repository">
  ## 3) OpenClaw-Repository klonen
</div>

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
```

Dieses Handbuch geht davon aus, dass du ein benutzerdefiniertes Image erstellst, um die Persistenz der Binaries sicherzustellen.

***

<div id="4-create-persistent-host-directories">
  ## 4) Persistente Host-Verzeichnisse erstellen
</div>

Docker-Container sind kurzlebig.
Jeglicher persistente Zustand muss auf dem Host liegen.

```bash
mkdir -p /root/.openclaw
mkdir -p /root/.openclaw/workspace

# Eigentümerschaft auf den Container-Benutzer setzen (UID 1000):
chown -R 1000:1000 /root/.openclaw
chown -R 1000:1000 /root/.openclaw/workspace
```

***

<div id="5-configure-environment-variables">
  ## 5) Umgebungsvariablen konfigurieren
</div>

Lege die Datei `.env` im Stammverzeichnis des Repositories an.

```bash
OPENCLAW_IMAGE=openclaw:latest
OPENCLAW_GATEWAY_TOKEN=change-me-now
OPENCLAW_GATEWAY_BIND=lan
OPENCLAW_GATEWAY_PORT=18789

OPENCLAW_CONFIG_DIR=/root/.openclaw
OPENCLAW_WORKSPACE_DIR=/root/.openclaw/workspace

GOG_KEYRING_PASSWORD=change-me-now
XDG_CONFIG_HOME=/home/node/.openclaw
```

Starke Secrets erzeugen:

```bash
openssl rand -hex 32
```

**Diese Datei nicht committen.**

***

<div id="6-docker-compose-configuration">
  ## 6) Docker-Compose-Konfiguration
</div>

Erstelle oder aktualisiere die Datei `docker-compose.yml`.

```yaml
services:
  openclaw-gateway:
    image: ${OPENCLAW_IMAGE}
    build: .
    restart: unless-stopped
    env_file:
      - .env
    environment:
      - HOME=/home/node
      - NODE_ENV=production
      - TERM=xterm-256color
      - OPENCLAW_GATEWAY_BIND=${OPENCLAW_GATEWAY_BIND}
      - OPENCLAW_GATEWAY_PORT=${OPENCLAW_GATEWAY_PORT}
      - OPENCLAW_GATEWAY_TOKEN=${OPENCLAW_GATEWAY_TOKEN}
      - GOG_KEYRING_PASSWORD=${GOG_KEYRING_PASSWORD}
      - XDG_CONFIG_HOME=${XDG_CONFIG_HOME}
      - PATH=/home/linuxbrew/.linuxbrew/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
    volumes:
      - ${OPENCLAW_CONFIG_DIR}:/home/node/.openclaw
      - ${OPENCLAW_WORKSPACE_DIR}:/home/node/.openclaw/workspace
    ports:
      # Empfohlen: Gateway nur auf Loopback auf dem VPS belassen; Zugriff über SSH-Tunnel.
      # Um es öffentlich freizugeben, entfernen Sie das Präfix `127.0.0.1:` und konfigurieren Sie die Firewall entsprechend.
      - "127.0.0.1:${OPENCLAW_GATEWAY_PORT}:18789"

      # Optional: nur wenn Sie iOS/Android-Knoten gegen diesen VPS betreiben und einen Canvas-Host benötigen.
      # Wenn Sie dies öffentlich freigeben, lesen Sie /gateway/security und konfigurieren Sie die Firewall entsprechend.
      # - "18793:18793"
    command:
      [
        "node",
        "dist/index.js",
        "gateway",
        "--bind",
        "${OPENCLAW_GATEWAY_BIND}",
        "--port",
        "${OPENCLAW_GATEWAY_PORT}"
      ]
```

***

<div id="7-bake-required-binaries-into-the-image-critical">
  ## 7) Erforderliche Binärprogramme ins Image einbacken (kritisch)
</div>

Binärprogramme in einem laufenden Container zu installieren, ist eine Falle.
Alles, was zur Laufzeit installiert wird, geht beim Neustart verloren.

Alle externen Binärprogramme, die von Fähigkeiten benötigt werden, müssen beim Bauen des Images installiert werden.

Die folgenden Beispiele zeigen nur drei gängige Binärprogramme:

* `gog` für Gmail-Zugriff
* `goplaces` für Google Places
* `wacli` für WhatsApp

Dies sind Beispiele, keine vollständige Liste.
Sie können beliebig viele Binärprogramme nach demselben Muster installieren.

Wenn Sie später neue Fähigkeiten hinzufügen, die von zusätzlichen Binärprogrammen abhängen, müssen Sie:

1. Die Dockerfile aktualisieren
2. Das Image neu bauen
3. Die Container neu starten

**Beispiel-Dockerfile**

```dockerfile
FROM node:22-bookworm

RUN apt-get update && apt-get install -y socat && rm -rf /var/lib/apt/lists/*

# Beispiel-Binary 1: Gmail CLI
RUN curl -L https://github.com/steipete/gog/releases/latest/download/gog_Linux_x86_64.tar.gz \
  | tar -xz -C /usr/local/bin && chmod +x /usr/local/bin/gog

# Beispiel-Binary 2: Google Places CLI
RUN curl -L https://github.com/steipete/goplaces/releases/latest/download/goplaces_Linux_x86_64.tar.gz \
  | tar -xz -C /usr/local/bin && chmod +x /usr/local/bin/goplaces

# Beispiel-Binary 3: WhatsApp CLI
RUN curl -L https://github.com/steipete/wacli/releases/latest/download/wacli_Linux_x86_64.tar.gz \
  | tar -xz -C /usr/local/bin && chmod +x /usr/local/bin/wacli

# Weitere Binaries nach demselben Muster unten hinzufügen

WORKDIR /app
COPY package.json pnpm-lock.yaml pnpm-workspace.yaml .npmrc ./
COPY ui/package.json ./ui/package.json
COPY scripts ./scripts

RUN corepack enable
RUN pnpm install --frozen-lockfile

COPY . .
RUN pnpm build
RUN pnpm ui:install
RUN pnpm ui:build

ENV NODE_ENV=production

CMD ["node","dist/index.js"]
```

***

<div id="8-build-and-launch">
  ## 8) Build erstellen und starten
</div>

```bash
docker compose build
docker compose up -d openclaw-gateway
```

Binärdateien überprüfen:

```bash
docker compose exec openclaw-gateway which gog
docker compose exec openclaw-gateway which goplaces
docker compose exec openclaw-gateway which wacli
```

Erwartete Ausgabe:

```
/usr/local/bin/gog
/usr/local/bin/goplaces
/usr/local/bin/wacli
```

***

<div id="9-verify-gateway">
  ## 9) Gateway überprüfen
</div>

```bash
docker compose logs -f openclaw-gateway
```

Erfolg:

```
[gateway] listening on ws://0.0.0.0:18789
```

Auf deinem Laptop:

```bash
ssh -N -L 18789:127.0.0.1:18789 root@YOUR_VPS_IP
```

Öffne in deinem Browser:

`http://127.0.0.1:18789/`

Füge dort dein Gateway-Token ein.

***

<div id="what-persists-where-source-of-truth">
  ## Was bleibt wo bestehen (Source of Truth)
</div>

OpenClaw läuft in Docker, aber Docker ist nicht die maßgebliche Quelle (Source of Truth).
Alle langlebigen Zustände müssen Neustarts, Neubuilds und Reboots überstehen.

| Komponente | Speicherort | Persistenzmechanismus | Hinweise |
|---|---|---|---|
| Gateway-Konfiguration | `/home/node/.openclaw/` | Host-Volume-Einbindung | Enthält `openclaw.json`, Tokens |
| Modell-Auth-Profile | `/home/node/.openclaw/` | Host-Volume-Einbindung | OAuth-Tokens, API-Keys |
| Fähigkeiten-Konfigurationen | `/home/node/.openclaw/skills/` | Host-Volume-Einbindung | Zustand auf Fähigkeiten-Ebene |
| Agent-Arbeitsbereich | `/home/node/.openclaw/workspace/` | Host-Volume-Einbindung | Code und Agent-Artefakte |
| WhatsApp-Sitzung | `/home/node/.openclaw/` | Host-Volume-Einbindung | Erhält QR-Login |
| Gmail-Schlüsselbund | `/home/node/.openclaw/` | Host-Volume + Passwort | Erfordert `GOG_KEYRING_PASSWORD` |
| Externe Binärdateien | `/usr/local/bin/` | Docker-Image | Muss zum Build-Zeitpunkt ins Image gepackt werden |
| Node.js-Laufzeitumgebung | Container-Dateisystem | Docker-Image | Bei jedem Image-Build neu erstellt |
| OS-Pakete | Container-Dateisystem | Docker-Image | Nicht zur Laufzeit installieren |
| Docker-Container | Flüchtig | Neustartbar | Gefahrlos löschbar |