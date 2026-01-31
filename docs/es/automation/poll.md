---
title: "Encuesta"
summary: "Envío de encuestas mediante Gateway y CLI"
read_when:
  - Agregar o modificar el soporte para encuestas
  - Depurar el envío de encuestas desde la CLI o el Gateway
---

<div id="polls">
  # Encuestas
</div>

<div id="supported-channels">
  ## Canales compatibles
</div>

- WhatsApp (canal web)
- Discord
- MS Teams (Adaptive Cards)

<div id="cli">
  ## CLI
</div>

```bash
# WhatsApp
openclaw message poll --target +15555550123 \
  --poll-question "Lunch today?" --poll-option "Yes" --poll-option "No" --poll-option "Maybe"
openclaw message poll --target 123456789@g.us \
  --poll-question "Meeting time?" --poll-option "10am" --poll-option "2pm" --poll-option "4pm" --poll-multi

# Discord
openclaw message poll --channel discord --target channel:123456789 \
  --poll-question "Snack?" --poll-option "Pizza" --poll-option "Sushi"
openclaw message poll --channel discord --target channel:123456789 \
  --poll-question "Plan?" --poll-option "A" --poll-option "B" --poll-duration-hours 48

# MS Teams
openclaw message poll --channel msteams --target conversation:19:abc@thread.tacv2 \
  --poll-question "Lunch?" --poll-option "Pizza" --poll-option "Sushi"
```

Opciones:

* `--channel`: `whatsapp` (por defecto), `discord` o `msteams`
* `--poll-multi`: permite seleccionar varias opciones
* `--poll-duration-hours`: solo para Discord (por defecto 24 si se omite)


<div id="gateway-rpc">
  ## RPC del Gateway
</div>

Método: `poll`

Parámetros:

- `to` (string, obligatorio)
- `question` (string, obligatorio)
- `options` (string[], obligatorio)
- `maxSelections` (number, opcional)
- `durationHours` (number, opcional)
- `channel` (string, opcional, por defecto: `whatsapp`)
- `idempotencyKey` (string, obligatorio)

<div id="channel-differences">
  ## Diferencias entre canales
</div>

- WhatsApp: 2-12 opciones, `maxSelections` debe estar dentro del número de opciones disponibles, ignora `durationHours`.
- Discord: 2-10 opciones, `durationHours` se limita a entre 1 y 768 horas (24 por defecto). `maxSelections > 1` habilita la selección múltiple; Discord no admite un recuento estricto de selecciones.
- MS Teams: encuestas con Adaptive Card (administradas por OpenClaw). No hay una API nativa de encuestas; se ignora `durationHours`.

<div id="agent-tool-message">
  ## Herramienta del agente (Mensaje)
</div>

Usa la herramienta `message` con la acción `poll` (`to`, `pollQuestion`, `pollOption`, opcionales `pollMulti`, `pollDurationHours`, `channel`).

Nota: Discord no tiene un modo de “elegir exactamente N”; `pollMulti` corresponde a selección múltiple.
Las encuestas de Teams se renderizan como Adaptive Cards y requieren que el Gateway permanezca en línea
para registrar los votos en `~/.openclaw/msteams-polls.json`.