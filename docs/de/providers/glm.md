---
title: GLM
summary: "Überblick über die GLM‑Modellfamilie + Verwendung in OpenClaw"
read_when:
  - Du möchtest GLM‑Modelle in OpenClaw verwenden
  - Du benötigst die Konvention zur Modellbenennung und die Einrichtung
---

<div id="glm-models">
  # GLM-Modelle
</div>

GLM ist eine **Modellfamilie** (kein Unternehmen), die über die Z.AI-Plattform verfügbar ist. In OpenClaw greifst du auf GLM-Modelle über den Anbieter `zai` und Modell-IDs wie `zai/glm-4.7` zu.

<div id="cli-setup">
  ## CLI-Einrichtung
</div>

```bash
openclaw onboard --auth-choice zai-api-key
```

<div id="config-snippet">
  ## Konfigurationsbeispiel
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

* GLM-Versionen und -Verfügbarkeit können sich ändern; prüfen Sie die Dokumentation von Z.AI für den aktuellen Stand.
* Beispielhafte Modell-IDs sind `glm-4.7` und `glm-4.6`.
* Details zum Anbieter finden Sie unter [/providers/zai](/de/providers/zai).