---
title: Piattaforme
summary: "Panoramica del supporto per le piattaforme (Gateway + app companion)"
read_when:
  - Quando cerchi il supporto per un sistema operativo o i percorsi di installazione
  - Quando decidi dove eseguire il Gateway
---

<div id="platforms">
  # Piattaforme
</div>

Il core di OpenClaw è scritto in TypeScript. **Node.js è il runtime consigliato**.
Bun non è consigliato per il Gateway (bug con WhatsApp/Telegram).

Esistono app companion per macOS (app nella barra dei menu) e nodi mobili (iOS/Android). Le app companion per Windows e
Linux sono in programma, ma il Gateway è già pienamente supportato.
Sono inoltre previste app companion native per Windows; per il Gateway è consigliato l&#39;uso tramite WSL2.

<div id="choose-your-os">
  ## Scegli il tuo sistema operativo
</div>

* macOS: [macOS](/it/platforms/macos)
* iOS: [iOS](/it/platforms/ios)
* Android: [Android](/it/platforms/android)
* Windows: [Windows](/it/platforms/windows)
* Linux: [Linux](/it/platforms/linux)

<div id="vps-hosting">
  ## VPS e hosting
</div>

* Hub VPS: [Hosting VPS](/it/vps)
* Fly.io: [Fly.io](/it/platforms/fly)
* Hetzner (Docker): [Hetzner](/it/platforms/hetzner)
* GCP (Compute Engine): [GCP](/it/platforms/gcp)
* exe.dev (VM + proxy HTTPS): [exe.dev](/it/platforms/exe-dev)

<div id="common-links">
  ## Link comuni
</div>

* Guida all&#39;installazione: [Guida introduttiva](/it/start/getting-started)
* Runbook del Gateway: [Gateway](/it/gateway)
* Configurazione del Gateway: [Configurazione](/it/gateway/configuration)
* Stato del servizio: `openclaw gateway status`

<div id="gateway-service-install-cli">
  ## Installazione del servizio Gateway (CLI)
</div>

Utilizza una di queste opzioni (tutte supportate):

* Procedura guidata (consigliata): `openclaw onboard --install-daemon`
* Diretta: `openclaw gateway install`
* Flusso di configurazione: `openclaw configure` → seleziona **Gateway service**
* Riparazione/migrazione: `openclaw doctor` (propone di installare o correggere il servizio)

Il target del servizio varia in base al sistema operativo:

* macOS: LaunchAgent (`bot.molt.gateway` o `bot.molt.<profile>`; legacy `com.openclaw.*`)
* Linux/WSL2: unità utente systemd (`openclaw-gateway[-<profile>].service`)