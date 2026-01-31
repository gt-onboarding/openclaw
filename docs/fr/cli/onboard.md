---
title: Onboarding
summary: "Référence CLI pour `openclaw onboard` (assistant interactif d'onboarding)"
read_when:
  - Vous souhaitez une configuration guidée de Gateway, de l'espace de travail, de l'authentification, des canaux et des compétences
---

<div id="openclaw-onboard">
  # `openclaw onboard`
</div>

Assistant d’onboarding interactif (mise en place du Gateway en local ou à distance).

Liens associés :

* Guide de l’assistant : [Onboarding](/fr/start/onboarding)

<div id="examples">
  ## Exemples
</div>

```bash
openclaw onboard
openclaw onboard --flow quickstart
openclaw onboard --flow manual
openclaw onboard --mode remote --remote-url ws://gateway-host:18789
```

Notes sur le déroulement :

* `quickstart` : questions minimales, génère automatiquement un jeton Gateway.
* `manual` : questions complètes pour port/bind/auth (alias de `advanced`).
* Pour démarrer une première conversation le plus rapidement possible : `openclaw dashboard` (Control UI, aucune configuration de canal nécessaire).
