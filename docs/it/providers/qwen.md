---
title: Qwen
summary: "Usa Qwen OAuth (piano gratuito) in OpenClaw"
read_when:
  - Vuoi usare Qwen con OpenClaw
  - Vuoi ottenere l'accesso OAuth al piano gratuito di Qwen Coder
---

<div id="qwen">
  # Qwen
</div>

Qwen offre un flusso OAuth per accedere al livello gratuito dei modelli Qwen Coder e Qwen Vision
(2.000 richieste al giorno, soggetto ai limiti di frequenza delle richieste imposti da Qwen).

<div id="enable-the-plugin">
  ## Abilita il plugin
</div>

```bash
openclaw plugins enable qwen-portal-auth
```

Riavvia il Gateway dopo averlo abilitato.

<div id="authenticate">
  ## Autenticazione
</div>

```bash
openclaw models auth login --provider qwen-portal --set-default
```

Questo avvia il flusso OAuth con codice dispositivo di Qwen e aggiunge una voce di provider al tuo
`models.json` (oltre a un alias `qwen` per passare rapidamente).

<div id="model-ids">
  ## ID dei modelli
</div>

* `qwen-portal/coder-model`
* `qwen-portal/vision-model`

Cambia modello con:

```bash
openclaw models set qwen-portal/coder-model
```

<div id="reuse-qwen-code-cli-login">
  ## Riutilizzare il login della Qwen Code CLI
</div>

Se hai già effettuato il login con la Qwen Code CLI, OpenClaw sincronizzerà le credenziali
da `~/.qwen/oauth_creds.json` quando carica l&#39;auth store. Ti serve comunque una
voce `models.providers.qwen-portal` (usa il comando di login sopra per crearne una).

<div id="notes">
  ## Note
</div>

* I token vengono aggiornati automaticamente; riesegui il comando di login se il rinnovo non riesce o se l&#39;accesso viene revocato.
* URL di base predefinito: `https://portal.qwen.ai/v1` (puoi sovrascriverlo con
  `models.providers.qwen-portal.baseUrl` se Qwen fornisce un endpoint diverso).
* Consulta [Model providers](/it/concepts/model-providers) per le regole a livello di provider.