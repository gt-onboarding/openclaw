---
title: "Vercel AI Gateway"
summary: "Configuración de Vercel AI Gateway (autenticación + selección de modelo)"
read_when:
  - Quieres usar Vercel AI Gateway con OpenClaw
  - Necesitas la variable de entorno de la clave de API o la opción de autenticación en la CLI
---

<div id="vercel-ai-gateway">
  # Vercel AI Gateway
</div>

[Vercel AI Gateway](https://vercel.com/ai-gateway) proporciona una API unificada para acceder a cientos de modelos mediante un único endpoint. 

- Proveedor: `vercel-ai-gateway`
- Autenticación: `AI_GATEWAY_API_KEY`
- API: compatible con Anthropic Messages

<div id="quick-start">
  ## Inicio rápido
</div>

1. Configura la clave de API (recomendado: guárdala en el Gateway):

```bash
openclaw onboard --auth-choice ai-gateway-api-key
```

2. Establece un modelo predeterminado:

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
  ## Ejemplo no interactivo
</div>

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice ai-gateway-api-key \
  --ai-gateway-api-key "$AI_GATEWAY_API_KEY"
```


<div id="environment-note">
  ## Nota sobre el entorno
</div>

Si el Gateway se ejecuta como un daemon (launchd/systemd), asegúrate de que la variable de entorno `AI_GATEWAY_API_KEY`
esté disponible para ese proceso (por ejemplo, en `~/.openclaw/.env` o mediante
`env.shellEnv`).