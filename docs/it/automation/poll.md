---
title: Sondaggio
summary: "Invio di sondaggi tramite Gateway + CLI"
read_when:
  - Aggiunta o modifica del supporto per i sondaggi
  - Debug dell'invio di sondaggi tramite CLI o Gateway
---

<div id="polls">
  # Sondaggi
</div>

<div id="supported-channels">
  ## Canali supportati
</div>

- WhatsApp (canale web)
- Discord
- MS Teams (Adaptive Cards)

<div id="cli">
  ## CLI
</div>

```bash
# WhatsApp
openclaw message poll --target +15555550123 \
  --poll-question "Pranzo oggi?" --poll-option "Sì" --poll-option "No" --poll-option "Forse"
openclaw message poll --target 123456789@g.us \
  --poll-question "Orario della riunione?" --poll-option "10am" --poll-option "2pm" --poll-option "4pm" --poll-multi

# Discord
openclaw message poll --channel discord --target channel:123456789 \
  --poll-question "Spuntino?" --poll-option "Pizza" --poll-option "Sushi"
openclaw message poll --channel discord --target channel:123456789 \
  --poll-question "Piano?" --poll-option "A" --poll-option "B" --poll-duration-hours 48

# MS Teams
openclaw message poll --channel msteams --target conversation:19:abc@thread.tacv2 \
  --poll-question "Pranzo?" --poll-option "Pizza" --poll-option "Sushi"
```

Opzioni:

* `--channel`: `whatsapp` (predefinito), `discord` o `msteams`
* `--poll-multi`: consente la selezione di più opzioni
* `--poll-duration-hours`: solo per Discord (predefinito 24 se omesso)


<div id="gateway-rpc">
  ## RPC del Gateway
</div>

Metodo: `poll`

Parametri:

- `to` (string, obbligatorio)
- `question` (string, obbligatorio)
- `options` (string[], obbligatorio)
- `maxSelections` (number, opzionale)
- `durationHours` (number, opzionale)
- `channel` (string, opzionale, valore predefinito: `whatsapp`)
- `idempotencyKey` (string, obbligatorio)

<div id="channel-differences">
  ## Differenze tra i canali
</div>

- WhatsApp: 2-12 opzioni, `maxSelections` deve essere compreso nel numero di opzioni, ignora `durationHours`.
- Discord: 2-10 opzioni, `durationHours` è limitato a 1-768 ore (valore predefinito 24). `maxSelections > 1` abilita la selezione multipla; Discord non supporta un numero esatto di selezioni.
- MS Teams: sondaggi con Adaptive Card (gestiti da OpenClaw). Nessuna API nativa per i sondaggi; `durationHours` viene ignorato.

<div id="agent-tool-message">
  ## Strumento Agente (Messaggio)
</div>

Usa lo strumento `message` con l'azione `poll` (`to`, `pollQuestion`, `pollOption`, opzionali `pollMulti`, `pollDurationHours`, `channel`).

Nota: Discord non ha una modalità “scegli esattamente N”; `pollMulti` corrisponde alla selezione multipla.
I sondaggi di Teams vengono resi come Adaptive Cards e richiedono che il Gateway rimanga online
per registrare i voti in `~/.openclaw/msteams-polls.json`.