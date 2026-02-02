---
title: "Vercel AI Gateway"
summary: "Configurazione di Vercel AI Gateway (auth + selezione del modello)"
read_when:
  - Se vuoi utilizzare Vercel AI Gateway con OpenClaw
  - Se ti serve la variabile d'ambiente della chiave API oppure l'opzione di autenticazione tramite CLI
---

<div id="vercel-ai-gateway">
  # Vercel AI Gateway
</div>

Il [Vercel AI Gateway](https://vercel.com/ai-gateway) fornisce un'API unificata per accedere a centinaia di modelli tramite un singolo endpoint. 

- Provider: `vercel-ai-gateway`
- Auth: `AI_GATEWAY_API_KEY`
- API: compatibile con Anthropic Messages

<div id="quick-start">
  ## Guida rapida
</div>

1. Imposta la API key (consigliato: salvala per il Gateway):

```bash
openclaw onboard --auth-choice ai-gateway-api-key
```

2. Imposta il modello predefinito:

```json5
{
  agents: {
    defaults: {
      model: { primary: "vercel-ai-gateway/anthropic/claude-opus-4.5" }
    }
  }
}
```


<div id="non-interactive-example">
  ## Esempio non interattivo
</div>

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice ai-gateway-api-key \
  --ai-gateway-api-key "$AI_GATEWAY_API_KEY"
```


<div id="environment-note">
  ## Nota sull'ambiente
</div>

Se il Gateway viene eseguito come demone (launchd/systemd), assicurati che `AI_GATEWAY_API_KEY`
sia accessibile a tale processo (ad esempio in `~/.openclaw/.env` o tramite
`env.shellEnv`).