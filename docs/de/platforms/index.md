---
title: Plattformen
summary: "Übersicht zur Plattformunterstützung (Gateway + Begleit‑Apps)"
read_when:
  - Wenn du nach Informationen zur Betriebssystemunterstützung oder zu Installationspfaden suchst
  - Wenn du entscheiden willst, wo das Gateway ausgeführt werden soll
---

<div id="platforms">
  # Plattformen
</div>

Der OpenClaw‑Kern ist in TypeScript geschrieben. **Node ist die empfohlene Runtime**.
Bun wird für das Gateway nicht empfohlen (WhatsApp/Telegram‑Bugs).

Companion‑Apps existieren für macOS (Menüleisten‑App) und mobile Knoten (iOS/Android). Windows‑ und
Linux‑Companion‑Apps sind geplant, aber das Gateway wird bereits heute vollständig unterstützt.
Native Companion‑Apps für Windows sind ebenfalls geplant; die Nutzung des Gateways über WSL2 wird empfohlen.

<div id="choose-your-os">
  ## Wähle dein Betriebssystem
</div>

* macOS: [macOS](/de/platforms/macos)
* iOS: [iOS](/de/platforms/ios)
* Android: [Android](/de/platforms/android)
* Windows: [Windows](/de/platforms/windows)
* Linux: [Linux](/de/platforms/linux)

<div id="vps-hosting">
  ## VPS &amp; Hosting
</div>

* VPS-Hub: [VPS-Hosting](/de/vps)
* Fly.io: [Fly.io](/de/platforms/fly)
* Hetzner (Docker): [Hetzner](/de/platforms/hetzner)
* GCP (Compute Engine): [GCP](/de/platforms/gcp)
* exe.dev (VM + HTTPS-Proxy): [exe.dev](/de/platforms/exe-dev)

<div id="common-links">
  ## Häufig verwendete Links
</div>

* Installationsanleitung: [Erste Schritte](/de/start/getting-started)
* Gateway-Betriebshandbuch: [Gateway](/de/gateway)
* Gateway-Konfiguration: [Konfiguration](/de/gateway/configuration)
* Dienststatus: `openclaw gateway status`

<div id="gateway-service-install-cli">
  ## Installation des Gateway-Dienstes (CLI)
</div>

Verwenden Sie eine der folgenden Optionen (alle werden unterstützt):

* Assistent (empfohlen): `openclaw onboard --install-daemon`
* Direkt: `openclaw gateway install`
* Konfigurations-Workflow: `openclaw configure` → **Gateway service** auswählen
* Reparieren/Migrieren: `openclaw doctor` (bietet an, den Dienst zu installieren oder zu reparieren)

Das Ziel für den Dienst hängt vom Betriebssystem ab:

* macOS: LaunchAgent (`bot.molt.gateway` oder `bot.molt.<profile>`; veraltet: `com.openclaw.*`)
* Linux/WSL2: systemd-Benutzerdienst (`openclaw-gateway[-<profile>].service`)