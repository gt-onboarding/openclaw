---
title: Gcp
summary: "OpenClaw Gateway im Dauerbetrieb (24/7) auf einer GCP Compute Engine VM (Docker) mit dauerhaftem Zustand betreiben"
read_when:
  - Du möchtest OpenClaw im Dauerbetrieb (24/7) auf GCP betreiben
  - Du möchtest ein produktionsreifes, stets verfügbares Gateway auf deiner eigenen VM
  - Du möchtest die volle Kontrolle über Persistenz, Binärdateien und Neustartverhalten
---

<div id="openclaw-on-gcp-compute-engine-docker-production-vps-guide">
  # OpenClaw auf GCP Compute Engine (Docker, Produktions-VPS-Leitfaden)
</div>

<div id="goal">
  ## Ziel
</div>

Ein persistent laufendes OpenClaw Gateway auf einer GCP Compute Engine-VM mit Docker betreiben, mit dauerhaftem Zustand, eingebetteten Binaries und sicherem Neustartverhalten.

Wenn du „OpenClaw 24/7 für ~5–12 $ pro Monat“ möchtest, ist dies ein zuverlässiges Setup auf Google Cloud.
Die Preise variieren je nach Maschinentyp und Region; wähle die kleinste VM, die zu deiner Last passt, und skaliere hoch, wenn du OOM-Fehler bekommst.

<div id="what-are-we-doing-simple-terms">
  ## Was machen wir hier (einfach erklärt)?
</div>

* Ein GCP-Projekt erstellen und Abrechnung aktivieren
* Eine Compute Engine VM erstellen
* Docker installieren (isolierte Anwendungslaufzeitumgebung)
* Das OpenClaw Gateway in Docker starten
* `~/.openclaw` + `~/.openclaw/workspace` auf dem Host persistent halten (übersteht Neustarts/Rebuilds)
* Vom Laptop aus über einen SSH-Tunnel auf die Control UI zugreifen

Auf das Gateway kannst du zugreifen über:

* SSH-Port-Forwarding von deinem Laptop
* Direkte Portfreigabe, wenn du Firewalling und Tokens selbst verwaltest

Diese Anleitung verwendet Debian auf GCP Compute Engine.
Ubuntu funktioniert ebenfalls; passe die Paketnamen entsprechend an.
Für den generischen Docker-Flow siehe [Docker](/de/install/docker).

***

<div id="quick-path-experienced-operators">
  ## Schnellstart (für erfahrene Operator:innen)
</div>

1. GCP-Projekt erstellen + Compute Engine API aktivieren
2. Compute Engine-VM erstellen (e2-small, Debian 12, 20 GB)
3. Per SSH auf die VM zugreifen
4. Docker installieren
5. OpenClaw-Repository klonen
6. Persistente Verzeichnisse auf dem Host anlegen
7. `.env` und `docker-compose.yml` konfigurieren
8. Erforderliche Binaries erzeugen, Build ausführen und starten

***

<div id="what-you-need">
  ## Was du brauchst
</div>

* GCP-Konto (Free Tier, geeignet für e2-micro)
* installierte gcloud CLI (oder die Cloud Console verwenden)
* SSH-Zugriff von deinem Laptop
* Grundkenntnisse im Umgang mit SSH und Copy/Paste
* ~20–30 Minuten
* Docker und Docker Compose
* Authentifizierungsdaten für Modelle
* Optionale Anbieter-Zugangsdaten
  * WhatsApp-QR
  * Telegram-Bot-Token
  * Gmail-OAuth

***

<div id="1-install-gcloud-cli-or-use-console">
  ## 1) gcloud CLI installieren (oder Konsole verwenden)
</div>

**Option A: gcloud CLI** (für Automatisierung empfohlen)

Installiere sie über https://cloud.google.com/sdk/docs/install

Initialisieren und authentifizieren:

```bash
gcloud init
gcloud auth login
```

**Option B: Cloud Console**

Alle Schritte können in der Web-UI unter https://console.cloud.google.com durchgeführt werden.

***

<div id="2-create-a-gcp-project">
  ## 2) Erstellen Sie ein GCP-Projekt
</div>

**CLI:**

```bash
gcloud projects create my-openclaw-project --name="OpenClaw Gateway"
gcloud config set project my-openclaw-project
```

Aktivieren Sie die Abrechnung unter https://console.cloud.google.com/billing (erforderlich für Compute Engine).

Aktivieren Sie die Compute Engine API:

```bash
gcloud services enable compute.googleapis.com
```

**Konsole:**

1. Gehe zu IAM &amp; Admin &gt; Projekt erstellen
2. Benenne das Projekt und erstelle es
3. Aktiviere die Abrechnung für dieses Projekt
4. Navigiere zu APIs &amp; Services &gt; APIs aktivieren &gt; suche nach „Compute Engine API“ &gt; Aktivieren

***

<div id="3-create-the-vm">
  ## 3) Erstellen Sie die VM
</div>

**Maschinentypen:**

| Typ      | Spezifikationen           | Kosten                     | Hinweise                                           |
| -------- | ------------------------- | -------------------------- | -------------------------------------------------- |
| e2-small | 2 vCPU, 2GB RAM           | ~12 $/Monat                | Empfohlen                                          |
| e2-micro | 2 vCPU (geteilt), 1GB RAM | Für das Free Tier geeignet | Kann unter Last aufgrund von OOM-Fehlern abstürzen |

**CLI:**

```bash
gcloud compute instances create openclaw-gateway \
  --zone=us-central1-a \
  --machine-type=e2-small \
  --boot-disk-size=20GB \
  --image-family=debian-12 \
  --image-project=debian-cloud
```

**Konsole:**

1. Navigiere zu Compute Engine &gt; VM-Instanzen &gt; Instanz erstellen
2. Name: `openclaw-gateway`
3. Region: `us-central1`, Zone: `us-central1-a`
4. Maschinentyp: `e2-small`
5. Startlaufwerk: Debian 12, 20GB
6. Erstellen

***

<div id="4-ssh-into-the-vm">
  ## 4) Per SSH auf der VM anmelden
</div>

**CLI:**

```bash
gcloud compute ssh openclaw-gateway --zone=us-central1-a
```

**Konsole:**

Klicken Sie auf die Schaltfläche „SSH“ neben Ihrer VM im Compute-Engine-Dashboard.

Hinweis: Die Verteilung des SSH-Schlüssels kann 1–2 Minuten nach dem Erstellen der VM dauern. Wenn die Verbindung abgelehnt wird, warten Sie und versuchen Sie es erneut.

***

<div id="5-install-docker-on-the-vm">
  ## 5) Docker installieren (auf der VM)
</div>

```bash
sudo apt-get update
sudo apt-get install -y git curl ca-certificates
curl -fsSL https://get.docker.com | sudo sh
sudo usermod -aG docker $USER
```

Melde dich ab und wieder an, damit die Gruppenänderung übernommen wird:

```bash
exit
```

Dann per SSH wieder einloggen:

```bash
gcloud compute ssh openclaw-gateway --zone=us-central1-a
```

Prüfen:

```bash
docker --version
docker compose version
```

***

<div id="6-clone-the-openclaw-repository">
  ## 6) Repository von OpenClaw klonen
</div>

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
```

Dieser Leitfaden geht davon aus, dass du ein eigenes Image erstellst, um die dauerhafte Verfügbarkeit der Binärdateien sicherzustellen.

***

<div id="7-create-persistent-host-directories">
  ## 7) Persistente Host-Verzeichnisse erstellen
</div>

Docker-Container sind kurzlebig.
Sämtlicher persistenter Zustand muss auf dem Host liegen.

```bash
mkdir -p ~/.openclaw
mkdir -p ~/.openclaw/workspace
```

***

<div id="8-configure-environment-variables">
  ## 8) Umgebungsvariablen konfigurieren
</div>

Lege die Datei `.env` im Stammverzeichnis des Repositorys an.

```bash
OPENCLAW_IMAGE=openclaw:latest
OPENCLAW_GATEWAY_TOKEN=change-me-now
OPENCLAW_GATEWAY_BIND=lan
OPENCLAW_GATEWAY_PORT=18789

OPENCLAW_CONFIG_DIR=/home/$USER/.openclaw
OPENCLAW_WORKSPACE_DIR=/home/$USER/.openclaw/workspace

GOG_KEYRING_PASSWORD=change-me-now
XDG_CONFIG_HOME=/home/node/.openclaw
```

Starke Secrets generieren:

```bash
openssl rand -hex 32
```

**Diese Datei nicht committen.**

***

<div id="9-docker-compose-configuration">
  ## 9) Docker-Compose-Konfiguration
</div>

Erstellen oder aktualisieren Sie `docker-compose.yml`.

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
      # Recommended: keep the Gateway loopback-only on the VM; access via SSH tunnel.
      # To expose it publicly, remove the `127.0.0.1:` prefix and firewall accordingly.
      - "127.0.0.1:${OPENCLAW_GATEWAY_PORT}:18789"

      # Optional: nur wenn Sie iOS/Android-Knoten gegen diese VM betreiben und einen Canvas-Host benötigen.
      # Wenn Sie dies öffentlich zugänglich machen, lesen Sie /gateway/security und konfigurieren Sie die Firewall entsprechend.
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

<div id="10-bake-required-binaries-into-the-image-critical">
  ## 10) Erforderliche Binärdateien ins Image einbacken (kritisch)
</div>

Das Installieren von Binärdateien in einem laufenden Container ist eine Falle.
Alles, was zur Laufzeit installiert wird, geht beim Neustart verloren.

Alle externen Binärdateien, die von Fähigkeiten benötigt werden, müssen beim Bauen des Images installiert werden.

Die folgenden Beispiele zeigen nur drei häufig verwendete Binärdateien:

* `gog` für Gmail-Zugriff
* `goplaces` für Google Places
* `wacli` für WhatsApp

Dies sind Beispiele, keine vollständige Liste.
Du kannst bei Bedarf beliebig viele Binärdateien nach demselben Muster installieren.

Wenn du später neue Fähigkeiten hinzufügst, die von zusätzlichen Binärdateien abhängen, musst du:

1. Die Dockerfile aktualisieren
2. Das Image neu erstellen
3. Die Container neu starten

**Beispiel-Dockerfile**

```dockerfile
FROM node:22-bookworm

RUN apt-get update && apt-get install -y socat && rm -rf /var/lib/apt/lists/*

# Example binary 1: Gmail CLI
RUN curl -L https://github.com/steipete/gog/releases/latest/download/gog_Linux_x86_64.tar.gz \
  | tar -xz -C /usr/local/bin && chmod +x /usr/local/bin/gog

# Example binary 2: Google Places CLI
RUN curl -L https://github.com/steipete/goplaces/releases/latest/download/goplaces_Linux_x86_64.tar.gz \
  | tar -xz -C /usr/local/bin && chmod +x /usr/local/bin/goplaces

# Example binary 3: WhatsApp CLI
RUN curl -L https://github.com/steipete/wacli/releases/latest/download/wacli_Linux_x86_64.tar.gz \
  | tar -xz -C /usr/local/bin && chmod +x /usr/local/bin/wacli

# Weitere Binaries nach demselben Muster hinzufügen

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

<div id="11-build-and-launch">
  ## 11) Build ausführen und starten
</div>

```bash
docker compose build
docker compose up -d openclaw-gateway
```

Binaries verifizieren:

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

<div id="12-verify-gateway">
  ## 12) Gateway überprüfen
</div>

```bash
docker compose logs -f openclaw-gateway
```

Erfolg:

```
[gateway] listening on ws://0.0.0.0:18789
```

***

<div id="13-access-from-your-laptop">
  ## 13) Zugriff von deinem Laptop aus
</div>

Erstelle einen SSH-Tunnel, um den Port des Gateways weiterzuleiten:

```bash
gcloud compute ssh openclaw-gateway --zone=us-central1-a -- -L 18789:127.0.0.1:18789
```

Öffne in deinem Browser:

`http://127.0.0.1:18789/`

Füge dein Gateway-Token ein.

***

<div id="what-persists-where-source-of-truth">
  ## Was wo persistiert (maßgebliche Quelle)
</div>

OpenClaw läuft in Docker, aber Docker ist nicht die maßgebliche Quelle.
Alle langlebigen Zustände müssen Neustarts, Rebuilds und Reboots überstehen.

| Komponente | Speicherort | Persistenzmechanismus | Hinweise |
|---|---|---|---|
| Gateway-Konfiguration | `/home/node/.openclaw/` | Host-Volume-Mount | Enthält `openclaw.json`, Tokens |
| Modell-Auth-Profile | `/home/node/.openclaw/` | Host-Volume-Mount | OAuth-Tokens, API-Keys |
| Skill-Konfigurationen | `/home/node/.openclaw/skills/` | Host-Volume-Mount | Skill-spezifischer Zustand |
| Agent-Arbeitsbereich | `/home/node/.openclaw/workspace/` | Host-Volume-Mount | Code und Agent-Artefakte |
| WhatsApp-Sitzung | `/home/node/.openclaw/` | Host-Volume-Mount | Bewahrt QR-Login |
| Gmail-Keyring | `/home/node/.openclaw/` | Host-Volume + Passwort | Benötigt `GOG_KEYRING_PASSWORD` |
| Externe Binaries | `/usr/local/bin/` | Docker-Image | Muss zur Build-Zeit eingebunden werden |
| Node.js-Laufzeitumgebung | Container-Dateisystem | Docker-Image | Bei jedem Image-Build neu erstellt |
| OS-Pakete | Container-Dateisystem | Docker-Image | Nicht zur Laufzeit installieren |
| Docker-Container | Flüchtig | Neustartfähig | Kann gefahrlos entfernt werden |

***

<div id="updates">
  ## Aktualisierungen
</div>

So aktualisieren Sie OpenClaw auf der VM:

```bash
cd ~/openclaw
git pull
docker compose build
docker compose up -d
```

***

<div id="troubleshooting">
  ## Fehlerbehebung
</div>

**SSH-Verbindung abgelehnt**

Die Bereitstellung des SSH-Schlüssels kann nach dem Erstellen der VM 1–2 Minuten dauern. Warten Sie und versuchen Sie es erneut.

**OS-Login-Probleme**

Überprüfen Sie Ihr OS-Login-Profil:

```bash
gcloud compute os-login describe-profile
```

Stellen Sie sicher, dass Ihr Account über die erforderlichen IAM-Berechtigungen verfügt (`Compute OS Login` oder `Compute OS Admin Login`).

**Out of memory (OOM)**

Wenn Sie `e2-micro` verwenden und auf OOM-Probleme stoßen, aktualisieren Sie auf `e2-small` oder `e2-medium`:

```bash
# Zuerst die VM stoppen
gcloud compute instances stop openclaw-gateway --zone=us-central1-a

# Maschinentyp ändern
gcloud compute instances set-machine-type openclaw-gateway \
  --zone=us-central1-a \
  --machine-type=e2-small

# VM starten
gcloud compute instances start openclaw-gateway --zone=us-central1-a
```

***

<div id="service-accounts-security-best-practice">
  ## Servicekonten (Security Best Practice)
</div>

Für die private Nutzung reicht dein Standardbenutzerkonto aus.

Für Automatisierung oder CI/CD-Pipelines solltest du ein dediziertes Servicekonto mit minimalen Berechtigungen erstellen:

1. Erstelle ein Servicekonto:
   ```bash
   gcloud iam service-accounts create openclaw-deploy \
     --display-name="OpenClaw Deployment"
   ```

2. Weise die Rolle Compute Instance Admin (oder eine enger gefasste benutzerdefinierte Rolle) zu:
   ```bash
   gcloud projects add-iam-policy-binding my-openclaw-project \
     --member="serviceAccount:openclaw-deploy@my-openclaw-project.iam.gserviceaccount.com" \
     --role="roles/compute.instanceAdmin.v1"
   ```

Vermeide die Verwendung der Rolle Owner für Automatisierung. Folge dem Prinzip der geringsten erforderlichen Berechtigungen.

Siehe https://cloud.google.com/iam/docs/understanding-roles für Details zu IAM-Rollen.

***

<div id="next-steps">
  ## Nächste Schritte
</div>

* Richte Nachrichtenkanäle ein: [Kanäle](/de/channels)
* Kopple lokale Geräte als Knoten: [Knoten](/de/nodes)
* Konfiguriere das Gateway: [Gateway-Konfiguration](/de/gateway/configuration)