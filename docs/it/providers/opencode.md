---
title: Opencode
summary: "Usa OpenCode Zen (modelli selezionati) con OpenClaw"
read_when:
  - Vuoi usare OpenCode Zen per l'accesso ai modelli
  - Vuoi un elenco selezionato di modelli adatti alla programmazione
---

<div id="opencode-zen">
  # OpenCode Zen
</div>

OpenCode Zen è un **elenco selezionato di modelli** consigliato dal team OpenCode per agenti di programmazione.
È un canale opzionale di accesso a modelli ospitati che utilizza una chiave API e il provider `opencode`.
Zen è attualmente in beta.

<div id="cli-setup">
  ## Configurazione CLI
</div>

```bash
openclaw onboard --auth-choice opencode-zen
# oppure in modalità non interattiva
openclaw onboard --opencode-zen-api-key "$OPENCODE_API_KEY"
```


<div id="config-snippet">
  ## Esempio di configurazione
</div>

```json5
{
  env: { OPENCODE_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "opencode/claude-opus-4-5" } } }
}
```


<div id="notes">
  ## Note
</div>

- È supportata anche `OPENCODE_ZEN_API_KEY`.
- Accedi a Zen, aggiungi i dati di fatturazione e copia la tua chiave API.
- OpenCode Zen fattura per singola richiesta; controlla la dashboard di OpenCode per i dettagli.