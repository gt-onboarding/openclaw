---
title: OpenAI-HTTP-API
summary: "Einen OpenAI-kompatiblen /v1/chat/completions-HTTP-Endpunkt über das Gateway bereitstellen"
read_when:
  - Beim Integrieren von Tools, die OpenAI-Chat-Completions erwarten
---

<div id="openai-chat-completions-http">
  # OpenAI Chat Completions (HTTP)
</div>

Das Gateway von OpenClaw kann einen kleinen, OpenAI-kompatiblen Chat-Completions-Endpoint bereitstellen.

Dieser Endpoint ist **standardmäßig deaktiviert**. Aktiviere ihn zunächst in der Konfiguration.

- `POST /v1/chat/completions`
- Gleicher Port wie das Gateway (WS + HTTP-Multiplex): `http://<gateway-host>:<port>/v1/chat/completions`

Im Hintergrund werden Anfragen als normaler Lauf eines Gateway-Agents ausgeführt (derselbe Codepfad wie `openclaw agent`), sodass Routing, Berechtigungen und Konfiguration mit deinem Gateway übereinstimmen.

<div id="authentication">
  ## Authentifizierung
</div>

Verwendet die Gateway-Auth-Konfiguration. Sende ein Bearer-Token:

- `Authorization: Bearer <token>`

Hinweise:

- Wenn `gateway.auth.mode="token"`, verwende `gateway.auth.token` (oder `OPENCLAW_GATEWAY_TOKEN`).
- Wenn `gateway.auth.mode="password"`, verwende `gateway.auth.password` (oder `OPENCLAW_GATEWAY_PASSWORD`).

<div id="choosing-an-agent">
  ## Auswahl eines Agents
</div>

Keine benutzerdefinierten Header erforderlich: Codieren Sie die Agent-ID im OpenAI-`model`-Feld:

- `model: "openclaw:<agentId>"` (Beispiel: `"openclaw:main"`, `"openclaw:beta"`)
- `model: "agent:<agentId>"` (Alias)

Oder Sie adressieren einen bestimmten OpenClaw-Agent per Header:

- `x-openclaw-agent-id: <agentId>` (Standardwert: `main`)

Erweitert:

- `x-openclaw-session-key: <sessionKey>` zur vollständigen Kontrolle des Sitzungsroutings.

<div id="enabling-the-endpoint">
  ## Aktivieren des Endpunkts
</div>

Setzen Sie `gateway.http.endpoints.chatCompletions.enabled` auf `true`:

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
  ## Endpunkt deaktivieren
</div>

Setze `gateway.http.endpoints.chatCompletions.enabled` auf `false`:

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
  ## Sitzungsverhalten
</div>

Standardmäßig ist der Endpunkt **zustandslos pro Anfrage** (für jeden Aufruf wird ein neuer Sitzungsschlüssel generiert).

Wenn die Anfrage eine OpenAI-`user`-Zeichenkette enthält, leitet das Gateway daraus einen stabilen Sitzungsschlüssel ab, sodass wiederholte Aufrufe dieselbe Agent-Sitzung teilen können.

<div id="streaming-sse">
  ## Streaming (SSE)
</div>

Setze `stream: true`, um Server-Sent Events (SSE) zu empfangen:

- `Content-Type: text/event-stream`
- Jede Ereigniszeile hat die Form `data: <json>`
- Der Stream endet mit `data: [DONE]`

<div id="examples">
  ## Beispiele
</div>

Ohne Streaming:

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

Streaming:

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
