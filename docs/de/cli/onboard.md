---
title: Onboarding
summary: "CLI-Referenz für `openclaw onboard` (interaktiver Onboarding-Assistent)"
read_when:
  - du eine geführte Einrichtung für Gateway, Arbeitsbereich, Authentifizierung, Kanäle und Fähigkeiten möchtest
---

<div id="openclaw-onboard">
  # `openclaw onboard`
</div>

Interaktiver Onboarding-Assistent (Einrichtung eines lokalen oder Remote-Gateways).

Verwandt:

* Assistentenleitfaden: [Onboarding](/de/start/onboarding)

<div id="examples">
  ## Beispiele
</div>

```bash
openclaw onboard
openclaw onboard --flow quickstart
openclaw onboard --flow manual
openclaw onboard --mode remote --remote-url ws://gateway-host:18789
```

Ablaufhinweise:

* `quickstart`: minimale Prompts, erzeugt automatisch ein Gateway-Token.
* `manual`: vollständige Prompts für Port/Bind/Auth (Alias für `advanced`).
* Schnellster Weg zum ersten Chat: `openclaw dashboard` (Control UI, keine Channel-Konfiguration erforderlich).
