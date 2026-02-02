---
title: Onboard
summary: "Riferimento CLI per `openclaw onboard` (procedura guidata interattiva per la configurazione iniziale)"
read_when:
  - Vuoi configurare in modo guidato Gateway, spazio di lavoro, autenticazione, canali e abilità
---

<div id="openclaw-onboard">
  # `openclaw onboard`
</div>

Procedura guidata interattiva di onboarding (configurazione del Gateway locale o remoto).

Correlato:

* Guida alla procedura guidata di onboarding: [Onboarding](/it/start/onboarding)

<div id="examples">
  ## Esempi
</div>

```bash
openclaw onboard
openclaw onboard --flow quickstart
openclaw onboard --flow manual
openclaw onboard --mode remote --remote-url ws://gateway-host:18789
```

Note sul flusso:

* `quickstart`: prompt minimi, genera automaticamente un token del Gateway.
* `manual`: prompt completi per porta/bind/auth (alias di `advanced`).
* Chat iniziale più veloce: `openclaw dashboard` (Control UI, senza configurare alcun canale).
