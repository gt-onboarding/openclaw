---
title: Sondage
summary: "Envoi de sondages via Gateway et la CLI"
read_when:
  - Ajout ou modification de la prise en charge des sondages
  - Débogage des envois de sondages depuis la CLI ou Gateway
---

<div id="polls">
  # Sondages
</div>

<div id="supported-channels">
  ## Canaux pris en charge
</div>

- WhatsApp (canal Web)
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

Options :

* `--channel` : `whatsapp` (par défaut), `discord` ou `msteams`
* `--poll-multi` : permet de sélectionner plusieurs options
* `--poll-duration-hours` : uniquement pour Discord (24 par défaut si ce paramètre est omis)


<div id="gateway-rpc">
  ## RPC du Gateway
</div>

Méthode : `poll`

Paramètres :

- `to` (string, obligatoire)
- `question` (string, obligatoire)
- `options` (string[], obligatoire)
- `maxSelections` (number, facultatif)
- `durationHours` (number, facultatif)
- `channel` (string, facultatif, par défaut : `whatsapp`)
- `idempotencyKey` (string, obligatoire)

<div id="channel-differences">
  ## Différences selon les canaux
</div>

- WhatsApp : 2 à 12 options, `maxSelections` doit être inférieur ou égal au nombre d’options, ignore `durationHours`.
- Discord : 2 à 10 options, `durationHours` est borné entre 1 et 768 heures (24 par défaut). `maxSelections > 1` active la sélection multiple ; Discord ne prend pas en charge un nombre de sélections strict.
- MS Teams : sondages Adaptive Card (gérés par OpenClaw). Pas d’API de sondage native ; `durationHours` est ignoré.

<div id="agent-tool-message">
  ## Outil d’Agent (Message)
</div>

Utilisez l’outil `message` avec l’action `poll` (`to`, `pollQuestion`, `pollOption`, `pollMulti`, `pollDurationHours`, `channel` facultatifs).

Remarque : Discord n’a pas de mode « choisir exactement N » ; `pollMulti` correspond à une sélection multiple.
Les sondages Teams sont affichés sous forme de cartes Adaptive Cards et nécessitent que le Gateway reste en ligne
pour enregistrer les votes dans `~/.openclaw/msteams-polls.json`.