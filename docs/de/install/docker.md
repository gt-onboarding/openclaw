---
title: Docker
summary: "Optionale Docker-basierte Einrichtung und Inbetriebnahme für OpenClaw"
read_when:
  - Du möchtest ein containerisiertes Gateway statt lokaler Installationen verwenden
  - Du überprüfst den Docker-Ablauf
---

<div id="docker-optional">
  # Docker (optional)
</div>

Docker ist **optional**. Verwende es nur, wenn du ein containerisiertes Gateway betreiben oder den Docker-Workflow validieren möchtest.

<div id="is-docker-right-for-me">
  ## Ist Docker das Richtige für mich?
</div>

* **Ja**: Du möchtest eine isolierte, temporäre Gateway-Umgebung oder OpenClaw auf einem Host ohne lokale Installation ausführen.
* **Nein**: Du betreibst OpenClaw auf deinem eigenen Rechner und willst einfach nur den schnellsten Dev-Loop. Verwende stattdessen den normalen Installationsablauf.
* **Hinweis zur Sandbox**: Agent-Sandboxing verwendet ebenfalls Docker, erfordert aber **nicht**, dass das gesamte Gateway in Docker läuft. Siehe [Sandboxing](/de/gateway/sandboxing).

In dieser Anleitung geht es um:

* Containerbasiertes Gateway (vollständiges OpenClaw in Docker)
* Agent-Sandbox pro Sitzung (Gateway auf dem Host + Docker-isolierte Agent-Tools)

Details zum Sandboxing: [Sandboxing](/de/gateway/sandboxing)

<div id="requirements">
  ## Anforderungen
</div>

* Docker Desktop (oder Docker Engine) + Docker Compose v2
* Ausreichend Festplattenspeicher für Images + Logs

<div id="containerized-gateway-docker-compose">
  ## Containerisiertes Gateway (Docker Compose)
</div>

<div id="quick-start-recommended">
  ### Schnellstart (empfohlen)
</div>

Im Repository-Root-Verzeichnis:

```bash
./docker-setup.sh
```

Dieses Skript:

* baut das Gateway-Image
* führt den Onboarding-Assistenten aus
* gibt optionale Hinweise zur Einrichtung von Anbietern aus
* startet das Gateway über Docker Compose
* erzeugt ein Gateway-Token und schreibt es in `.env`

Optionale Umgebungsvariablen:

* `OPENCLAW_DOCKER_APT_PACKAGES` — zusätzliche APT-Pakete während des Builds installieren
* `OPENCLAW_EXTRA_MOUNTS` — zusätzliche Host-Bind-Mounts hinzufügen
* `OPENCLAW_HOME_VOLUME` — `/home/node` in einem benannten Volume persistent speichern

Nachdem das Skript abgeschlossen ist:

* Öffne `http://127.0.0.1:18789/` in deinem Browser.
* Füge das Token in die Control UI ein (Settings → token).

Es schreibt Konfigurations- und Arbeitsbereichsdaten auf dem Host:

* `~/.openclaw/`
* `~/.openclaw/workspace`

Läuft das auf einem VPS? Siehe [Hetzner (Docker VPS)](/de/platforms/hetzner).

<div id="manual-flow-compose">
  ### Manueller Ablauf (compose)
</div>

```bash
docker build -t openclaw:local -f Dockerfile .
docker compose run --rm openclaw-cli onboard
docker compose up -d openclaw-gateway
```

<div id="extra-mounts-optional">
  ### Zusätzliche Mounts (optional)
</div>

Wenn du zusätzliche Host-Verzeichnisse in die Container einbinden möchtest, setze
`OPENCLAW_EXTRA_MOUNTS`, bevor du `docker-setup.sh` ausführst. Diese Variable akzeptiert eine
durch Kommas getrennte Liste von Docker-Bind-Mounts und wendet sie sowohl auf
`openclaw-gateway` als auch auf `openclaw-cli` an, wobei `docker-compose.extra.yml` generiert wird.

Beispiel:

```bash
export OPENCLAW_EXTRA_MOUNTS="$HOME/.codex:/home/node/.codex:ro,$HOME/github:/home/node/github:rw"
./docker-setup.sh
```

Hinweise:

* Pfade müssen unter macOS/Windows in Docker Desktop freigegeben werden.
* Wenn du `OPENCLAW_EXTRA_MOUNTS` änderst, führe `docker-setup.sh` erneut aus,
  um die zusätzliche Compose-Datei neu zu generieren.
* `docker-compose.extra.yml` wird generiert. Bearbeite sie nicht von Hand.

<div id="persist-the-entire-container-home-optional">
  ### Gesamtes Home-Verzeichnis des Containers persistent machen (optional)
</div>

Wenn `/home/node` auch nach dem Neuaufsetzen des Containers erhalten bleiben soll, setze ein benanntes Volume über `OPENCLAW_HOME_VOLUME`. Dadurch wird ein Docker-Volume erstellt und unter `/home/node` gemountet, während die Standard-Bind-Mounts für Konfiguration/Arbeitsbereich beibehalten werden. Verwende hier ein benanntes Volume (keinen Bind-Pfad); für Bind-Mounts verwende
`OPENCLAW_EXTRA_MOUNTS`.

Beispiel:

```bash
export OPENCLAW_HOME_VOLUME="openclaw_home"
./docker-setup.sh
```

Du kannst das mit zusätzlichen Volume-Mounts kombinieren:

```bash
export OPENCLAW_HOME_VOLUME="openclaw_home"
export OPENCLAW_EXTRA_MOUNTS="$HOME/.codex:/home/node/.codex:ro,$HOME/github:/home/node/github:rw"
./docker-setup.sh
```

Hinweise:

* Wenn du `OPENCLAW_HOME_VOLUME` änderst, führe `docker-setup.sh` erneut aus, um die zusätzliche Compose-Datei neu zu generieren.
* Das benannte Volume bleibt bestehen, bis es mit `docker volume rm <name>` entfernt wird.

<div id="install-extra-apt-packages-optional">
  ### Zusätzliche apt-Pakete installieren (optional)
</div>

Wenn du Systempakete im Image benötigst (zum Beispiel Build-Tools
oder Medienbibliotheken), setze `OPENCLAW_DOCKER_APT_PACKAGES`, bevor du
`docker-setup.sh` ausführst. Dadurch werden die Pakete während des Image-Builds
installiert, sodass sie erhalten bleiben, selbst wenn der Container gelöscht wird.

Beispiel:

```bash
export OPENCLAW_DOCKER_APT_PACKAGES="ffmpeg build-essential"
./docker-setup.sh
```

Hinweise:

* Dies nimmt eine durch Leerzeichen getrennte Liste von APT-Paketnamen entgegen.
* Wenn du `OPENCLAW_DOCKER_APT_PACKAGES` änderst, führe `docker-setup.sh` erneut aus,
  um das Image neu zu erstellen.

<div id="faster-rebuilds-recommended">
  ### Schnellere Rebuilds (empfohlen)
</div>

Um Rebuilds zu beschleunigen, ordnest du deine Dockerfile so an, dass Dependency-Layer gecached werden.
So vermeidest du, `pnpm install` erneut auszuführen, solange sich die Lockfiles nicht ändern:

```dockerfile
FROM node:22-bookworm

# Bun installieren (erforderlich für Build-Skripte)
RUN curl -fsSL https://bun.sh/install | bash
ENV PATH="/root/.bun/bin:${PATH}"

RUN corepack enable

WORKDIR /app

# Abhängigkeiten cachen, sofern sich Paket-Metadaten nicht ändern
COPY package.json pnpm-lock.yaml pnpm-workspace.yaml .npmrc ./
COPY ui/package.json ./ui/package.json
COPY scripts ./scripts

RUN pnpm install --frozen-lockfile

COPY . .
RUN pnpm build
RUN pnpm ui:install
RUN pnpm ui:build

ENV NODE_ENV=production

CMD ["node","dist/index.js"]
```

<div id="channel-setup-optional">
  ### Channel-Einrichtung (optional)
</div>

Verwende den CLI-Container, um Kanäle zu konfigurieren, und starte anschließend bei Bedarf das Gateway neu.

WhatsApp (QR):

```bash
docker compose run --rm openclaw-cli channels login
```

Telegram (Bot-Token):

```bash
docker compose run --rm openclaw-cli channels add --channel telegram --token "<token>"
```

Discord (Bot-Token):

```bash
docker compose run --rm openclaw-cli channels add --channel discord --token "<token>"
```

Dokumentation: [WhatsApp](/de/channels/whatsapp), [Telegram](/de/channels/telegram), [Discord](/de/channels/discord)

<div id="health-check">
  ### Healthcheck
</div>

```bash
docker compose exec openclaw-gateway node dist/index.js health --token "$OPENCLAW_GATEWAY_TOKEN"
```

<div id="e2e-smoke-test-docker">
  ### E2E-Smoke-Test (Docker)
</div>

```bash
scripts/e2e/onboard-docker.sh
```

<div id="qr-import-smoke-test-docker">
  ### Smoke-Test für QR-Import (Docker)
</div>

```bash
pnpm test:docker:qr
```

<div id="notes">
  ### Hinweise
</div>

* Standardmäßig wird das Gateway im Containerbetrieb an `lan` gebunden.
* Der Gateway-Container ist die maßgebliche Quelle für Sitzungen (`~/.openclaw/agents/<agentId>/sessions/`).

<div id="agent-sandbox-host-gateway-docker-tools">
  ## Agent-Sandbox (Host-Gateway + Docker-Tools)
</div>

Ausführliche Details: [Sandboxing](/de/gateway/sandboxing)

<div id="what-it-does">
  ### Was es macht
</div>

Wenn `agents.defaults.sandbox` aktiviert ist, führen **Nicht-Haupt-Sitzungen** Tools in einem Docker-
Container aus. Das Gateway bleibt auf deinem Host, aber die Tool-Ausführung ist isoliert:

* scope: `"agent"` standardmäßig (ein Container + Arbeitsbereich pro agent)
* scope: `"session"` für Isolierung pro Sitzung
* Arbeitsbereichsordner je Scope wird unter `/workspace` eingehängt
* optionaler Zugriff auf den Agent-Arbeitsbereich (`agents.defaults.sandbox.workspaceAccess`)
* Allow/Deny-Tool-Policy (Deny hat Vorrang)
* eingehende Medien werden in den aktiven Sandbox-Arbeitsbereich kopiert (`media/inbound/*`), damit Tools sie lesen können (mit `workspaceAccess: "rw"` landet dies im Agent-Arbeitsbereich)

Warnung: `scope: "shared"` deaktiviert die Isolierung zwischen Sitzungen. Alle Sitzungen teilen sich
einen Container und einen Arbeitsbereich.

<div id="per-agent-sandbox-profiles-multi-agent">
  ### Sandbox-Profile pro Agent (Multi-Agent)
</div>

Wenn du Multi-Agent-Routing verwendest, kann jeder Agent sandbox- und Tool-Einstellungen übersteuern:
`agents.list[].sandbox` und `agents.list[].tools` (plus `agents.list[].tools.sandbox.tools`). Dadurch kannst du
unterschiedliche Zugriffsebenen innerhalb eines Gateways betreiben:

* Vollzugriff (persönlicher Agent)
* Nur-Lese-Tools + Nur-Lese-Arbeitsbereich (Familien-/Arbeitsagent)
* Keine Dateisystem-/Shell-Tools (öffentlicher Agent)

Siehe [Multi-Agent Sandbox &amp; Tools](/de/multi-agent-sandbox-tools) für Beispiele,
Vorrangregeln und Fehlerbehebung.

<div id="default-behavior">
  ### Standardverhalten
</div>

* Image: `openclaw-sandbox:bookworm-slim`
* Ein Container pro Agent
* Zugriff auf den Agent-Arbeitsbereich: `workspaceAccess: "none"` (Standard) verwendet `~/.openclaw/sandboxes`
  * `"ro"` behält den Sandbox-Arbeitsbereich unter `/workspace` und bindet den Agent-Arbeitsbereich schreibgeschützt unter `/agent` ein (deaktiviert `write`/`edit`/`apply_patch`)
  * `"rw"` bindet den Agent-Arbeitsbereich mit Lese-/Schreibzugriff unter `/workspace` ein
* Automatisches Aufräumen: Leerlauf &gt; 24 h ODER Alter &gt; 7 Tage
* Netzwerk: standardmäßig `none` (explizit aktivieren, wenn du ausgehenden Netzwerkzugriff benötigst)
* Standardmäßig erlaubt: `exec`, `process`, `read`, `write`, `edit`, `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status`
* Standardmäßig verweigert: `browser`, `canvas`, `nodes`, `cron`, `discord`, `gateway`

<div id="enable-sandboxing">
  ### Sandboxing aktivieren
</div>

Wenn du planst, Pakete in `setupCommand` zu installieren, beachte:

* Standard-`docker.network` ist `"none"` (kein ausgehender Netzwerkverkehr).
* `readOnlyRoot: true` verhindert Paketinstallationen.
* `user` muss für `apt-get` root sein (lasse `user` weg oder setze `user: "0:0"`).
  OpenClaw erstellt Container automatisch neu, wenn sich `setupCommand` (oder die Docker-Konfiguration) ändert,
  es sei denn, der Container wurde **kürzlich verwendet** (innerhalb von ca. 5 Minuten). „Hot“-Container
  protokollieren eine Warnung mit dem exakten `openclaw sandbox recreate ...`-Befehl.

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main", // off | non-main | all
        scope: "agent", // session | agent | shared (agent ist Standard)
        workspaceAccess: "none", // none | ro | rw
        workspaceRoot: "~/.openclaw/sandboxes",
        docker: {
          image: "openclaw-sandbox:bookworm-slim",
          workdir: "/workspace",
          readOnlyRoot: true,
          tmpfs: ["/tmp", "/var/tmp", "/run"],
          network: "none",
          user: "1000:1000",
          capDrop: ["ALL"],
          env: { LANG: "C.UTF-8" },
          setupCommand: "apt-get update && apt-get install -y git curl jq",
          pidsLimit: 256,
          memory: "1g",
          memorySwap: "2g",
          cpus: 1,
          ulimits: {
            nofile: { soft: 1024, hard: 2048 },
            nproc: 256
          },
          seccompProfile: "/path/to/seccomp.json",
          apparmorProfile: "openclaw-sandbox",
          dns: ["1.1.1.1", "8.8.8.8"],
          extraHosts: ["internal.service:10.0.0.5"]
        },
        prune: {
          idleHours: 24, // 0 disables idle pruning
          maxAgeDays: 7  // 0 disables max-age pruning
        }
      }
    }
  },
  tools: {
    sandbox: {
      tools: {
        allow: ["exec", "process", "read", "write", "edit", "sessions_list", "sessions_history", "sessions_send", "sessions_spawn", "session_status"],
        deny: ["browser", "canvas", "nodes", "cron", "discord", "gateway"]
      }
    }
  }
}
```

Hardening-Optionen befinden sich unter `agents.defaults.sandbox.docker`:
`network`, `user`, `pidsLimit`, `memory`, `memorySwap`, `cpus`, `ulimits`,
`seccompProfile`, `apparmorProfile`, `dns`, `extraHosts`.

Multi-agent: Du kannst `agents.defaults.sandbox.{docker,browser,prune}.*` pro agent über `agents.list[].sandbox.{docker,browser,prune}.*` überschreiben
(wird ignoriert, wenn `agents.defaults.sandbox.scope` / `agents.list[].sandbox.scope` auf `"shared"` gesetzt ist).

<div id="build-the-default-sandbox-image">
  ### Standard-Sandbox-Image bauen
</div>

```bash
scripts/sandbox-setup.sh
```

Damit wird `openclaw-sandbox:bookworm-slim` mit `Dockerfile.sandbox` gebaut.

<div id="sandbox-common-image-optional">
  ### Sandbox-Common-Image (optional)
</div>

Wenn du ein Sandbox-Image mit gängigen Build-Tools (Node, Go, Rust usw.) verwenden möchtest, erstelle das Common-Image:

```bash
scripts/sandbox-common-setup.sh
```

Damit wird `openclaw-sandbox-common:bookworm-slim` erstellt. So verwendest du es:

```json5
{
  agents: { defaults: { sandbox: { docker: { image: "openclaw-sandbox-common:bookworm-slim" } } } }
}
```

<div id="sandbox-browser-image">
  ### Sandbox-Browser-Image
</div>

Um das Browser-Tool innerhalb der sandbox auszuführen, erstellen Sie das Browser-Image:

```bash
scripts/sandbox-browser-setup.sh
```

Dies erstellt `openclaw-sandbox-browser:bookworm-slim` mit
`Dockerfile.sandbox-browser`. Der Container führt Chromium mit aktiviertem CDP
und einem optionalen noVNC-Observer aus (headful via Xvfb).

Hinweise:

* Headful (Xvfb) reduziert Bot-Blocking im Vergleich zu Headless.
* Headless kann weiterhin verwendet werden, indem `agents.defaults.sandbox.browser.headless=true` gesetzt wird.
* Es wird keine vollständige Desktop-Umgebung (GNOME) benötigt; Xvfb stellt das Display bereit.

Verwenden Sie die folgende Konfiguration:

```json5
{
  agents: {
    defaults: {
      sandbox: {
        browser: { enabled: true }
      }
    }
  }
}
```

Benutzerdefiniertes Browser-Image:

```json5
{
  agents: {
    defaults: {
      sandbox: { browser: { image: "my-openclaw-browser" } }
    }
  }
}
```

Wenn aktiviert, erhält der Agent:

* eine Sandbox-Browser-Steuerungs-URL (für das `browser`-Tool)
* eine noVNC-URL (falls aktiviert und `headless=false`)

Beachte: Wenn du eine Allowlist für Tools verwendest, füge `browser` hinzu
(und entferne es aus `deny`), sonst bleibt das Tool blockiert.
Prune-Regeln (`agents.defaults.sandbox.prune`) gelten auch für Browser-Container.

<div id="custom-sandbox-image">
  ### Eigenes sandbox-Image
</div>

Erstelle ein eigenes Image und verweise in der Konfiguration darauf:

```bash
docker build -t my-openclaw-sbx -f Dockerfile.sandbox .
```

```json5
{
  agents: {
    defaults: {
      sandbox: { docker: { image: "my-openclaw-sbx" } }
    }
  }
}
```

<div id="tool-policy-allowdeny">
  ### Tool-Richtlinie (allow/deny)
</div>

* `deny` hat Vorrang vor `allow`.
* Wenn `allow` leer ist, sind alle Tools (außer `deny`) verfügbar.
* Wenn `allow` nicht leer ist, sind nur die Tools in `allow` verfügbar (abzüglich `deny`).

<div id="pruning-strategy">
  ### Bereinigungsstrategie
</div>

Zwei Parameter:

* `prune.idleHours`: Container entfernen, die in den letzten X Stunden nicht verwendet wurden (0 = deaktivieren)
* `prune.maxAgeDays`: Container entfernen, die älter als X Tage sind (0 = deaktivieren)

Beispiel:

* Aktive Sitzungen beibehalten, aber Lebensdauer begrenzen:
  `idleHours: 24`, `maxAgeDays: 7`
* Nie bereinigen:
  `idleHours: 0`, `maxAgeDays: 0`

<div id="security-notes">
  ### Sicherheitshinweise
</div>

* Hard wall gilt nur für **Tools** (exec/read/write/edit/apply&#95;patch).
* Host-only-Tools wie browser/camera/canvas sind standardmäßig gesperrt.
* Das Zulassen von `browser` in der sandbox **hebt die Isolation auf** (`browser` läuft auf dem Host).

<div id="troubleshooting">
  ## Fehlerbehebung
</div>

* Image fehlt: baue das Image mit [`scripts/sandbox-setup.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/sandbox-setup.sh) oder setze `agents.defaults.sandbox.docker.image`.
* Container läuft nicht: er wird bei Bedarf automatisch pro Sitzung erstellt.
* Berechtigungsfehler in der sandbox: setze `docker.user` auf eine UID:GID, die den
  Besitzverhältnissen deines eingebundenen Arbeitsbereichs entspricht (oder führe `chown` auf dem Arbeitsbereichsordner aus).
* Benutzerdefinierte Tools nicht gefunden: OpenClaw führt Befehle mit `sh -lc` (Login-Shell) aus, was
  `/etc/profile` einliest und PATH zurücksetzen kann. Setze `docker.env.PATH`, um deine
  benutzerdefinierten Tool-Pfade voranzustellen (z. B. `/custom/bin:/usr/local/share/npm-global/bin`), oder füge
  ein Skript unter `/etc/profile.d/` in deinem Dockerfile hinzu.