---
title: Webchat
summary: "Come l'app macOS incorpora il Webchat del Gateway e come eseguirne il debug"
read_when:
  - Debug della vista Webchat su macOS o della porta di loopback
---

<div id="webchat-macos-app">
  # WebChat (app macOS)
</div>

L'app della barra dei menu di macOS incorpora la UI di WebChat come vista SwiftUI nativa. Si
connette al Gateway e per impostazione predefinita utilizza la **sessione principale** per l'agente
selezionato (con un selettore di sessioni per le altre sessioni).

- **Modalità locale**: si connette direttamente al WebSocket locale del Gateway.
- **Modalità remota**: inoltra la porta di controllo del Gateway tramite SSH e utilizza quel
  tunnel come piano dati.

<div id="launch-debugging">
  ## Avvio e debug
</div>

- Avvio manuale: menu Lobster → “Open Chat”.
- Apertura automatica per test:
  ```bash
  dist/OpenClaw.app/Contents/MacOS/OpenClaw --webchat
  ```
- Log: `./scripts/clawlog.sh` (sottosistema `bot.molt`, categoria `WebChatSwiftUI`).

<div id="how-its-wired">
  ## Com'è cablato
</div>

- Piano dati: metodi WS del Gateway `chat.history`, `chat.send`, `chat.abort`,
  `chat.inject` ed eventi `chat`, `agent`, `presence`, `tick`, `health`.
- Sessione: per impostazione predefinita usa la sessione primaria (`main`, o
  `global` quando lo scope è globale). La UI può passare da una sessione all’altra.
- La procedura di onboarding utilizza una sessione dedicata per mantenere separata la configurazione al primo avvio.

<div id="security-surface">
  ## Superficie di attacco
</div>

- La modalità remota inoltra esclusivamente la porta di controllo WebSocket del Gateway tramite SSH.

<div id="known-limitations">
  ## Limitazioni note
</div>

- La UI è ottimizzata per le sessioni di chat (non è una sandbox di browser completa).