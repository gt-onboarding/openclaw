---
title: Mehrere Gateways
summary: "Mehrere OpenClaw-Gateways auf einem Host betreiben (Isolation, Ports und Profile)"
read_when:
  - Du betreibst mehr als ein Gateway auf demselben Host
  - Du benötigst isolierte Konfiguration/Zustand/Ports pro Gateway
---

<div id="multiple-gateways-same-host">
  # Mehrere Gateways (gleicher Host)
</div>

Die meisten Setups sollten ein einzelnes Gateway verwenden, da ein Gateway mehrere Messaging-Verbindungen und Agenten verarbeiten kann. Wenn du stärkere Isolation oder Redundanz benötigst (z. B. für einen Rescue-Bot), betreibe separate Gateways mit isolierten Profilen/Ports.

<div id="isolation-checklist-required">
  ## Isolations-Checkliste (erforderlich)
</div>

- `OPENCLAW_CONFIG_PATH` — instanzspezifische Konfigurationsdatei
- `OPENCLAW_STATE_DIR` — instanzspezifische Sitzungen, Zugangsdaten, Caches
- `agents.defaults.workspace` — instanzspezifisches Arbeitsbereichs-Wurzelverzeichnis
- `gateway.port` (oder `--port`) — eindeutig pro Instanz
- Abgeleitete Ports (Browser/Canvas) dürfen sich nicht überlappen

Wenn diese gemeinsam genutzt werden, kommt es zu Konfigurations-Race-Conditions und Portkonflikten.

<div id="recommended-profiles-profile">
  ## Empfohlen: Profile (`--profile`)
</div>

Profile setzen automatisch den Scope für `OPENCLAW_STATE_DIR` und `OPENCLAW_CONFIG_PATH` und hängen an Dienstnamen ein Suffix an.

```bash
# main
openclaw --profile main setup
openclaw --profile main gateway --port 18789

# rescue
openclaw --profile rescue setup
openclaw --profile rescue gateway --port 19001
```

Profilbezogene Dienste:

```bash
openclaw --profile main gateway install
openclaw --profile rescue gateway install
```


<div id="rescue-bot-guide">
  ## Rescue-Bot-Anleitung
</div>

Führe ein zweites Gateway auf demselben Host aus, mit jeweils eigener:

- Profil/Konfiguration
- State-Verzeichnis
- Arbeitsbereich
- Basisport (plus abgeleitete Ports)

So bleibt der Rescue-Bot vom Hauptbot isoliert und kann debuggen oder Konfigurationsänderungen anwenden, wenn der primäre Bot ausgefallen ist.

Abstand der Ports: Lass mindestens 20 Ports zwischen den Basisports, damit die abgeleiteten Browser/Canvas/CDP-Ports niemals kollidieren.

<div id="how-to-install-rescue-bot">
  ### Installation des Rescue-Bots
</div>

```bash
# Main bot (existing or fresh, without --profile param)
# Runs on port 18789 + Chrome CDC/Canvas/... Ports 
openclaw onboard
openclaw gateway install

# Rescue bot (isolated profile + ports)
openclaw --profile rescue onboard
# Hinweise: 
# - Arbeitsbereich-Name wird standardmäßig mit -rescue als Suffix versehen
# - Port sollte mindestens 18789 + 20 Ports betragen, 
#   besser einen völlig anderen Basis-Port wählen, z. B. 19789,
# - der Rest des Onboardings verläuft wie gewohnt

# To install the service (if not happened automatically during onboarding)
openclaw --profile rescue gateway install
```


<div id="port-mapping-derived">
  ## Portzuordnung (abgeleitet)
</div>

Basisport = `gateway.port` (oder `OPENCLAW_GATEWAY_PORT` / `--port`).

- Port des Browser-Steuerdienstes = Basis + 2 (nur Loopback)
- `canvasHost.port = Basis + 4`
- CDP-Ports für Browser-Profile werden automatisch aus `browser.controlPort + 9 .. + 108` zugewiesen

Wenn du einen dieser Werte in Konfiguration oder Umgebungsvariablen überschreibst, musst du sicherstellen, dass sie pro Instanz eindeutig sind.

<div id="browsercdp-notes-common-footgun">
  ## Browser/CDP-Hinweise (häufige Stolperfalle)
</div>

- Pinne `browser.cdpUrl` **nicht** auf denselben Wert in mehreren Instanzen.
- Jede Instanz benötigt ihren eigenen Browser-Steuerport und eigenen CDP-Bereich (abgeleitet vom jeweiligen Gateway-Port).
- Wenn du explizite CDP-Ports brauchst, setze `browser.profiles.<name>.cdpPort` pro Instanz.
- Remote-Chrome: verwende `browser.profiles.<name>.cdpUrl` (pro Profil, pro Instanz).

<div id="manual-env-example">
  ## Beispiel für manuelle Env-Konfiguration
</div>

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/main.json \
OPENCLAW_STATE_DIR=~/.openclaw-main \
openclaw gateway --port 18789

OPENCLAW_CONFIG_PATH=~/.openclaw/rescue.json \
OPENCLAW_STATE_DIR=~/.openclaw-rescue \
openclaw gateway --port 19001
```


<div id="quick-checks">
  ## Schnellchecks
</div>

```bash
openclaw --profile main status
openclaw --profile rescue status
openclaw --profile rescue browser status
```
