---
title: Openai
summary: "Nutze OpenAI in OpenClaw über API-Schlüssel oder ein Codex-Abonnement"
read_when:
  - Du möchtest OpenAI-Modelle in OpenClaw nutzen
  - Du möchtest dich mit einem Codex-Abonnement statt mit API-Schlüsseln authentifizieren
---

<div id="openai">
  # OpenAI
</div>

OpenAI stellt Entwickler-APIs für GPT-Modelle bereit. Codex unterstützt die Anmeldung über **ChatGPT-Anmeldung** für abonnementbasierten Zugriff oder die Anmeldung über **API-Schlüssel** für nutzungsbasierten Zugriff. Codex Cloud erfordert die Anmeldung über ChatGPT.

<div id="option-a-openai-api-key-openai-platform">
  ## Option A: OpenAI-API-Schlüssel (OpenAI-Plattform)
</div>

**Am besten geeignet für:** direkten API-Zugriff und nutzungsbasierte Abrechnung.
Rufe deinen API-Schlüssel im OpenAI-Dashboard ab.

### CLI-Setup

```bash
openclaw onboard --auth-choice openai-api-key
# oder nicht interaktiv
openclaw onboard --openai-api-key "$OPENAI_API_KEY"
```

<div id="config-snippet">
  ### Konfigurationsbeispiel
</div>

```json5
{
  env: { OPENAI_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "openai/gpt-5.2" } } }
}
```

<div id="option-b-openai-code-codex-subscription">
  ## Option B: Abonnement für OpenAI Code (Codex)
</div>

**Am besten geeignet für:** Verwendung des ChatGPT/Codex-Abonnements anstelle eines API-Schlüssels.
Die Codex-Cloud erfordert eine Anmeldung mit ChatGPT, während die Codex CLI sowohl die Anmeldung mit ChatGPT als auch mit einem API-Schlüssel unterstützt.

<div id="cli-setup">
  ### CLI-Einrichtung
</div>

```bash
# Codex-OAuth im Assistenten ausführen
openclaw onboard --auth-choice openai-codex

# Oder OAuth direkt ausführen
openclaw models auth login --provider openai-codex
```

### Konfigurations-Snippet

```json5
{
  agents: { defaults: { model: { primary: "openai-codex/gpt-5.2" } } }
}
```

<div id="notes">
  ## Hinweise
</div>

* Modell-Referenzen verwenden immer `provider/model` (siehe [/concepts/models](/de/concepts/models)).
* Details zur Authentifizierung und Regeln zur Wiederverwendung findest du unter [/concepts/oauth](/de/concepts/oauth).