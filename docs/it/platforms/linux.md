---
title: Linux
summary: "Supporto Linux e stato dell'app companion"
read_when:
  - Quando vuoi conoscere lo stato dell'app companion per Linux
  - Quando stai pianificando la copertura delle piattaforme o i contributi
---

<div id="linux-app">
  # App Linux
</div>

Il Gateway è completamente supportato su Linux. **Node è il runtime consigliato**.
Bun non è consigliato per il Gateway (bug con WhatsApp/Telegram).

Sono previste app companion native per Linux. I contributi sono benvenuti se vuoi aiutarci a svilupparne una.

<div id="beginner-quick-path-vps">
  ## Percorso rapido per principianti (VPS)
</div>

1. Installa Node 22 o superiore
2. `npm i -g openclaw@latest`
3. `openclaw onboard --install-daemon`
4. Dal tuo laptop: `ssh -N -L 18789:127.0.0.1:18789 <user>@<host>`
5. Apri `http://127.0.0.1:18789/` e incolla il tuo token

Guida VPS passo-passo: [exe.dev](/it/platforms/exe-dev)

<div id="install">
  ## Installazione
</div>

* [Guida introduttiva](/it/start/getting-started)
* [Installazione e aggiornamenti](/it/install/updating)
* Procedure opzionali: [Bun (sperimentale)](/it/install/bun), [Nix](/it/install/nix), [Docker](/it/install/docker)

<div id="gateway">
  ## Gateway
</div>

* [Runbook del Gateway](/it/gateway)
* [Configurazione](/it/gateway/configuration)

<div id="gateway-service-install-cli">
  ## Installazione del servizio Gateway (CLI)
</div>

Usa uno dei seguenti metodi:

```
openclaw onboard --install-daemon
```

In alternativa:

```
openclaw gateway install
```

In alternativa:

```
openclaw configure
```

Quando richiesto, seleziona **Gateway service**.

Riparare/migrare:

```
openclaw doctor
```

<div id="system-control-systemd-user-unit">
  ## Controllo del sistema (unità utente systemd)
</div>

OpenClaw installa per impostazione predefinita un servizio systemd **utente**. Usa un servizio **di sistema** per server condivisi o sempre attivi. L&#39;esempio completo dell&#39;unità e le relative linee guida
sono disponibili nel [runbook del Gateway](/it/gateway).

Configurazione minima:

Crea il file `~/.config/systemd/user/openclaw-gateway[-<profile>].service`:

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

Abilita il servizio:

```
systemctl --user enable --now openclaw-gateway[-<profile>].service
```
