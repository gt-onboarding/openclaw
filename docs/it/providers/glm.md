---
title: GLM
summary: "Panoramica della famiglia di modelli GLM + come usarla in OpenClaw"
read_when:
  - Vuoi usare i modelli GLM in OpenClaw
  - Hai bisogno della convenzione di denominazione e della configurazione dei modelli
---

<div id="glm-models">
  # Modelli GLM
</div>

GLM è una **famiglia di modelli** (non un&#39;azienda) disponibile sulla piattaforma Z.AI. In OpenClaw, i modelli GLM
sono accessibili tramite il provider `zai` e ID di modello come `zai/glm-4.7`.

<div id="cli-setup">
  ## Configurazione della CLI
</div>

```bash
openclaw onboard --auth-choice zai-api-key
```

<div id="config-snippet">
  ## Esempio di configurazione
</div>

```json5
{
  env: { ZAI_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "zai/glm-4.7" } } }
}
```

<div id="notes">
  ## Note
</div>

* Le versioni e la disponibilità dei modelli GLM possono cambiare; consulta la documentazione di Z.AI per gli aggiornamenti più recenti.
* Esempi di ID modello includono `glm-4.7` e `glm-4.6`.
* Per maggiori dettagli sui provider, vedi [/providers/zai](/it/providers/zai).