---
title: Incorporación
summary: "Referencia de la CLI para `openclaw onboard` (asistente interactivo de incorporación)"
read_when:
  - Quieres una configuración asistida de Gateway, espacio de trabajo, autenticación, canales y habilidades
---

<div id="openclaw-onboard">
  # `openclaw onboard`
</div>

Asistente interactivo de configuración inicial (para un Gateway local o remoto).

Relacionado:

* Guía del asistente: [Incorporación](/es/start/onboarding)

<div id="examples">
  ## Ejemplos
</div>

```bash
openclaw onboard
openclaw onboard --flow quickstart
openclaw onboard --flow manual
openclaw onboard --mode remote --remote-url ws://gateway-host:18789
```

Notas sobre el flujo:

* `quickstart`: preguntas mínimas, genera automáticamente un token del Gateway.
* `manual`: preguntas completas para puerto/bind/auth (alias de `advanced`).
* Primer chat más rápido: `openclaw dashboard` (Control UI, sin necesidad de configurar canales).
