---
title: API HTTP OpenAI
summary: "Exposer depuis le Gateway un point de terminaison HTTP /v1/chat/completions compatible OpenAI"
read_when:
  - Intégrer des outils conçus pour OpenAI Chat Completions
---

<div id="openai-chat-completions-http">
  # Complétions de chat OpenAI (HTTP)
</div>

Le Gateway d’OpenClaw peut exposer un petit point de terminaison de complétions de chat compatible OpenAI.

Ce point de terminaison est **désactivé par défaut**. Activez-le d’abord dans la configuration.

- `POST /v1/chat/completions`
- Même port que le Gateway (multiplexage WS + HTTP) : `http://<gateway-host>:<port>/v1/chat/completions`

En interne, les requêtes sont exécutées comme une exécution d’agent Gateway normale (même chemin de code que `openclaw agent`), de sorte que le routage, les permissions et la configuration sont alignés sur ceux de votre Gateway.

<div id="authentication">
  ## Authentification
</div>

Utilise la configuration d’authentification du Gateway. Envoyez un jeton Bearer :

- `Authorization: Bearer <token>`

Remarques :

- Lorsque `gateway.auth.mode="token"`, utilisez `gateway.auth.token` (ou `OPENCLAW_GATEWAY_TOKEN`).
- Lorsque `gateway.auth.mode="password"`, utilisez `gateway.auth.password` (ou `OPENCLAW_GATEWAY_PASSWORD`).

<div id="choosing-an-agent">
  ## Choisir un agent
</div>

Aucun en-tête personnalisé requis : encodez l’identifiant de l’agent dans le champ `model` d’OpenAI :

- `model: "openclaw:&lt;agentId&gt;"` (exemple : `"openclaw:main"`, `"openclaw:beta"`)
- `model: "agent:&lt;agentId&gt;"` (alias)

Ou ciblez un agent OpenClaw spécifique via un en-tête :

- `x-openclaw-agent-id: &lt;agentId&gt;` (par défaut : `main`)

Avancé :

- `x-openclaw-session-key: &lt;sessionKey&gt;` pour contrôler entièrement le routage des sessions.

<div id="enabling-the-endpoint">
  ## Activation du point de terminaison
</div>

Définissez `gateway.http.endpoints.chatCompletions.enabled` sur `true` :

```json5
{
  gateway: {
    http: {
      endpoints: {
        chatCompletions: { enabled: true }
      }
    }
  }
}
```


<div id="disabling-the-endpoint">
  ## Désactivation du point de terminaison
</div>

Définissez `gateway.http.endpoints.chatCompletions.enabled` sur `false` :

```json5
{
  gateway: {
    http: {
      endpoints: {
        chatCompletions: { enabled: false }
      }
    }
  }
}
```


<div id="session-behavior">
  ## Comportement de session
</div>

Par défaut, le point de terminaison est **sans état pour chaque requête** (une nouvelle clé de session est générée à chaque appel).

Si la requête inclut une chaîne `user` OpenAI, le Gateway dérive une clé de session stable à partir de cette valeur, afin que des appels répétés partagent la même session d’agent.

<div id="streaming-sse">
  ## Diffusion en continu (SSE)
</div>

Définissez `stream: true` pour recevoir des événements envoyés par le serveur (Server-Sent Events, SSE) :

- `Content-Type: text/event-stream`
- Chaque événement est sur une ligne `data: <json>`
- Le flux se termine par `data: [DONE]`

<div id="examples">
  ## Exemples
</div>

Sans streaming :

```bash
curl -sS http://127.0.0.1:18789/v1/chat/completions \
  -H 'Authorization: Bearer YOUR_TOKEN' \
  -H 'Content-Type: application/json' \
  -H 'x-openclaw-agent-id: main' \
  -d '{
    "model": "openclaw",
    "messages": [{"role":"user","content":"hi"}]
  }'
```

Streaming :

```bash
curl -N http://127.0.0.1:18789/v1/chat/completions \
  -H 'Authorization: Bearer YOUR_TOKEN' \
  -H 'Content-Type: application/json' \
  -H 'x-openclaw-agent-id: main' \
  -d '{
    "model": "openclaw",
    "stream": true,
    "messages": [{"role":"user","content":"hi"}]
  }'
```
