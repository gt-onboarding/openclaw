---
title: "Vercel AI Gateway"
summary: "Einrichtung von Vercel AI Gateway (Authentifizierung + Modellauswahl)"
read_when:
  - Du möchtest Vercel AI Gateway mit OpenClaw verwenden
  - Du benötigst die Umgebungsvariable für den API-Key oder eine Authentifizierungsoption in der CLI
---

<div id="vercel-ai-gateway">
  # Vercel AI Gateway
</div>

Das [Vercel AI Gateway](https://vercel.com/ai-gateway) stellt eine einheitliche API bereit, um über einen einzigen Endpunkt auf Hunderte von Modellen zuzugreifen. 

- Anbieter: `vercel-ai-gateway`
- Auth: `AI_GATEWAY_API_KEY`
- API: Anthropic-Messages-kompatibel

<div id="quick-start">
  ## Schnellstart
</div>

1. Setze den API-Schlüssel (empfohlen: im Gateway speichern):

```bash
openclaw onboard --auth-choice ai-gateway-api-key
```

2. Setze ein Standardmodell:

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
  ## Nicht-interaktives Beispiel
</div>

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice ai-gateway-api-key \
  --ai-gateway-api-key "$AI_GATEWAY_API_KEY"
```


<div id="environment-note">
  ## Hinweis zur Umgebung
</div>

Wenn das Gateway als Daemon (launchd/systemd) läuft, stelle sicher, dass `AI_GATEWAY_API_KEY`
für diesen Prozess verfügbar ist (zum Beispiel in `~/.openclaw/.env` oder über
`env.shellEnv`).