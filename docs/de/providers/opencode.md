---
title: Opencode
summary: "Verwende OpenCode Zen (kuratierte Modelle) mit OpenClaw"
read_when:
  - Du möchtest OpenCode Zen für den Modellzugriff verwenden
  - Du möchtest eine kuratierte Liste programmierfreundlicher Modelle
---

<div id="opencode-zen">
  # OpenCode Zen
</div>

OpenCode Zen ist eine **kuratierte Liste von Modellen**, die vom OpenCode-Team für Coding-Agents empfohlen wird.
Es ist ein optionaler, gehosteter Zugriffsweg auf Modelle, der einen API-Schlüssel und den Anbieter `opencode` verwendet.
Zen befindet sich derzeit in der Beta-Phase.

<div id="cli-setup">
  ## CLI-Setup
</div>

```bash
openclaw onboard --auth-choice opencode-zen
# oder nicht interaktiv
openclaw onboard --opencode-zen-api-key "$OPENCODE_API_KEY"
```


<div id="config-snippet">
  ## Konfigurationsbeispiel
</div>

```json5
{
  env: { OPENCODE_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "opencode/claude-opus-4-5" } } }
}
```


<div id="notes">
  ## Hinweise
</div>

- `OPENCODE_ZEN_API_KEY` wird ebenfalls unterstützt.
- Melde dich bei Zen an, hinterlege deine Rechnungsdaten und kopiere deinen API-Schlüssel.
- OpenCode Zen rechnet pro Anfrage ab; Details findest du im OpenCode-Dashboard.