---
title: Zai
summary: "Z.AI (GLM-Modelle) mit OpenClaw verwenden"
read_when:
  - Du möchtest Z.AI-/GLM-Modelle in OpenClaw einsetzen
  - Du brauchst ein einfaches ZAI_API_KEY-Setup
---

<div id="zai">
  # Z.AI
</div>

Z.AI ist die API-Plattform für **GLM**-Modelle. Sie stellt REST-APIs für GLM bereit und verwendet API-Schlüssel
zur Authentifizierung. Erstellen Sie Ihren API-Schlüssel in der Z.AI-Konsole. OpenClaw verwendet den `zai` anbieter
mit einem Z.AI-API-Schlüssel.

<div id="cli-setup">
  ## CLI-Setup
</div>

```bash
openclaw onboard --auth-choice zai-api-key
# oder nicht-interaktiv
openclaw onboard --zai-api-key "$ZAI_API_KEY"
```

<div id="config-snippet">
  ## Config-Snippet
</div>

```json5
{
  env: { ZAI_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "zai/glm-4.7" } } }
}
```

<div id="notes">
  ## Hinweise
</div>

* GLM-Modelle sind unter `zai/<model>` verfügbar (z. B. `zai/glm-4.7`).
* Siehe [/providers/glm](/de/providers/glm) für eine Übersicht der Modellfamilie.
* Z.AI verwendet Bearer-Authentifizierung mit deinem API-Schlüssel.