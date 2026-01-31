---
title: Linux
summary: "Linux-Unterstützung + Status der Companion-App"
read_when:
  - Wenn du den Status der Linux-Companion-App suchst
  - Wenn du Plattformabdeckung oder Beiträge planst
---

<div id="linux-app">
  # Linux-App
</div>

Das Gateway wird unter Linux vollständig unterstützt. **Node.js ist die empfohlene Laufzeitumgebung**.
Bun wird für das Gateway nicht empfohlen (WhatsApp-/Telegram-Bugs).

Native Linux-Companion-Apps sind geplant. Beiträge sind willkommen, wenn du beim Entwickeln einer solchen App mithelfen möchtest.

<div id="beginner-quick-path-vps">
  ## Schnellstart für Einsteiger (VPS)
</div>

1. Installiere Node 22 oder höher
2. `npm i -g openclaw@latest`
3. `openclaw onboard --install-daemon`
4. Von deinem Laptop: `ssh -N -L 18789:127.0.0.1:18789 <user>@<host>`
5. Öffne `http://127.0.0.1:18789/` und füge dein Token ein

Schritt-für-Schritt-VPS-Anleitung: [exe.dev](/de/platforms/exe-dev)

<div id="install">
  ## Installation
</div>

* [Erste Schritte](/de/start/getting-started)
* [Installation &amp; Updates](/de/install/updating)
* Optionale Installationsvarianten: [Bun (experimentell)](/de/install/bun), [Nix](/de/install/nix), [Docker](/de/install/docker)

<div id="gateway">
  ## Gateway
</div>

* [Gateway-Runbook](/de/gateway)
* [Konfiguration](/de/gateway/configuration)

<div id="gateway-service-install-cli">
  ## Installation des Gateway-Service (CLI)
</div>

Führen Sie einen der folgenden Schritte aus:

```
openclaw onboard --install-daemon
```

Alternativ:

```
openclaw gateway install
```

Oder:

```
openclaw configure
```

Wählen Sie bei der Aufforderung **Gateway service** aus.

Reparieren/Migrieren:

```
openclaw doctor
```

<div id="system-control-systemd-user-unit">
  ## Systemsteuerung (systemd User-Unit)
</div>

OpenClaw installiert standardmäßig einen systemd-**User**-Dienst. Verwende einen **System**-Dienst für gemeinsam genutzte oder dauerhaft laufende Server. Das vollständige Unit-Beispiel und die Anleitung findest du im [Gateway-Runbook](/de/gateway).

Minimale Einrichtung:

Erstelle `~/.config/systemd/user/openclaw-gateway[-<profile>].service`:

```
[Unit]
Description=OpenClaw Gateway (profile: <profile>, v<version>)
After=network-online.target
Wants=network-online.target

[Service]
ExecStart=/usr/local/bin/openclaw gateway --port 18789
Restart=always
RestartSec=5

[Install]
WantedBy=default.target
```

Aktivieren Sie sie:

```
systemctl --user enable --now openclaw-gateway[-<profile>].service
```
