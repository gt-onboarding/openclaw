---
title: Nix
summary: "Installa OpenClaw in modo dichiarativo con Nix"
read_when:
  - Vuoi installazioni riproducibili, con possibilità di rollback
  - Stai già usando Nix/NixOS/Home Manager
  - Vuoi che tutto sia fissato a versioni specifiche e gestito in modo dichiarativo
---

<div id="nix-installation">
  # Installazione con Nix
</div>

Il metodo consigliato per eseguire OpenClaw con Nix è usare **[nix-openclaw](https://github.com/openclaw/nix-openclaw)** — un modulo Home Manager completo e pronto all&#39;uso.

<div id="quick-start">
  ## Guida rapida
</div>

Incolla quanto segue nel tuo agente AI (Claude, Cursor, ecc.):

```text
I want to set up nix-openclaw on my Mac.
Repository: github:openclaw/nix-openclaw

What I need you to do:
1. Check if Determinate Nix is installed (if not, install it)
2. Create a local flake at ~/code/openclaw-local using templates/agent-first/flake.nix
3. Help me create a Telegram bot (@BotFather) and get my chat ID (@userinfobot)
4. Set up secrets (bot token, Anthropic key) - plain files at ~/.secrets/ is fine
5. Fill in the template placeholders and run home-manager switch
6. Verify: launchd running, bot responds to messages

Reference the nix-openclaw README for module options.
```

> **📦 Guida completa: [github.com/openclaw/nix-openclaw](https://github.com/openclaw/nix-openclaw)**
>
> Il repository nix-openclaw è il punto di riferimento canonico per l&#39;installazione con Nix. Questa pagina è solo una panoramica rapida.

<div id="what-you-get">
  ## Cosa ottieni
</div>

* Gateway + app macOS + strumenti (whisper, spotify, telecamere) — tutti con versioni bloccate
* Servizio launchd che sopravvive ai riavvii
* Sistema di plugin con configurazione dichiarativa
* Rollback istantaneo: `home-manager switch --rollback`

***

<div id="nix-mode-runtime-behavior">
  ## Comportamento di runtime in modalità Nix
</div>

Quando `OPENCLAW_NIX_MODE=1` è impostato (automatico con nix-openclaw):

OpenClaw supporta una **modalità Nix** che rende la configurazione deterministica e disabilita i flussi di installazione automatica.
Per abilitarla, esporta:

```bash
OPENCLAW_NIX_MODE=1
```

Su macOS, l&#39;app GUI non eredita automaticamente le variabili d&#39;ambiente della shell. Puoi
anche abilitare la modalità Nix tramite il comando `defaults`:

```bash
defaults write bot.molt.mac openclaw.nixMode -bool true
```

<div id="config-state-paths">
  ### Percorsi di configurazione e stato
</div>

OpenClaw legge il file di configurazione JSON5 da `OPENCLAW_CONFIG_PATH` e memorizza i dati mutabili in `OPENCLAW_STATE_DIR`.

* `OPENCLAW_STATE_DIR` (predefinito: `~/.openclaw`)
* `OPENCLAW_CONFIG_PATH` (predefinito: `$OPENCLAW_STATE_DIR/openclaw.json`)

Quando esegui in ambiente Nix, imposta esplicitamente queste variabili su percorsi gestiti da Nix, in modo che lo stato di runtime e la configurazione
rimangano al di fuori dello store immutabile.

<div id="runtime-behavior-in-nix-mode">
  ### Comportamento in fase di esecuzione in modalità Nix
</div>

* Le procedure di auto-installazione e auto-mutazione sono disabilitate
* Le dipendenze mancanti generano messaggi di risoluzione specifici per Nix
* La UI mostra un banner di modalità Nix in sola lettura quando presente

<div id="packaging-note-macos">
  ## Nota sul packaging (macOS)
</div>

Il flusso di packaging di macOS richiede un modello Info.plist stabile in:

```
apps/macos/Sources/OpenClaw/Resources/Info.plist
```

[`scripts/package-mac-app.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/package-mac-app.sh) copia questo template nel bundle dell&#39;app e aggiorna i campi dinamici
(bundle ID, versione/build, Git SHA, chiavi Sparkle). Questo mantiene il file plist deterministico per il packaging con SwiftPM
e per le build Nix (che non si basano su una toolchain completa di Xcode).

<div id="related">
  ## Correlati
</div>

* [nix-openclaw](https://github.com/openclaw/nix-openclaw) — guida completa alla configurazione
* [Wizard](/it/start/wizard) — configurazione della CLI non-Nix
* [Docker](/it/install/docker) — configurazione in container