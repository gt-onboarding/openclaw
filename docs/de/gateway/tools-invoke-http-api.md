---
title: Tools Invoke Http Api
summary: "Ein einzelnes Tool direkt über den Gateway-HTTP-Endpunkt aufrufen"
read_when:
  - Tools aufrufen, ohne einen vollständigen Agent-Durchlauf auszuführen
  - Automatisierungen erstellen, die Tool-Richtlinien durchsetzen müssen
---

<div id="tools-invoke-http">
  # Tools Invoke (HTTP)
</div>

Das Gateway von OpenClaw stellt einen einfachen HTTP-Endpunkt bereit, um ein einzelnes Tool direkt aufzurufen. Er ist immer aktiv, aber durch Gateway-Authentifizierung und Tool-Policy abgesichert.

- `POST /tools/invoke`
- Gleicher Port wie das Gateway (WS + HTTP-Multiplexing): `http://<gateway-host>:<port>/tools/invoke`

Die Standard-Maximalgröße der Payload beträgt 2 MB.

<div id="authentication">
  ## Authentifizierung
</div>

Verwendet die Auth-Konfiguration des Gateways. Sende ein Bearer-Token:

- `Authorization: Bearer <token>`

Hinweise:

- Wenn `gateway.auth.mode="token"` ist, verwende `gateway.auth.token` (oder `OPENCLAW_GATEWAY_TOKEN`).
- Wenn `gateway.auth.mode="password"` ist, verwende `gateway.auth.password` (oder `OPENCLAW_GATEWAY_PASSWORD`).

<div id="request-body">
  ## Request Body
</div>

```json
{
  "tool": "sessions_list",
  "action": "json",
  "args": {},
  "sessionKey": "main",
  "dryRun": false
}
```

Felder:

* `tool` (string, erforderlich): Name des aufzurufenden Tools.
* `action` (string, optional): wird in `args` übernommen, falls das Toolschema `action` unterstützt und die `args`-Nutzlast es nicht enthält.
* `args` (object, optional): toolspezifische Argumente.
* `sessionKey` (string, optional): Ziel-Sitzungsschlüssel. Wenn weggelassen oder `"main"`, verwendet das Gateway den konfigurierten Haupt-Sitzungsschlüssel (beachtet `session.mainKey` und den Standard-agent oder `global` im globalen Scope).
* `dryRun` (boolean, optional): für zukünftige Verwendung reserviert; wird derzeit ignoriert.


<div id="policy-routing-behavior">
  ## Richtlinien- und Routing-Verhalten
</div>

Die Verfügbarkeit von Tools wird durch dieselbe Richtlinienkette gefiltert, die auch von Gateway-Agents verwendet wird:

- `tools.profile` / `tools.byProvider.profile`
- `tools.allow` / `tools.byProvider.allow`
- `agents.<id>.tools.allow` / `agents.<id>.tools.byProvider.allow`
- Gruppenrichtlinien (wenn der Sitzungsschlüssel einer Gruppe oder einem Kanal zugeordnet ist)
- Subagenten-Richtlinie (bei Aufruf mit einem Subagenten-Sitzungsschlüssel)

Wenn ein Tool durch die Richtlinien nicht zugelassen ist, gibt der Endpunkt **404** zurück.

Damit Gruppenrichtlinien den Kontext besser auflösen können, kannst du optional Folgendes setzen:

- `x-openclaw-message-channel: <channel>` (Beispiel: `slack`, `telegram`)
- `x-openclaw-account-id: <accountId>` (wenn mehrere Konten existieren)

<div id="responses">
  ## Antworten
</div>

- `200` → `{ ok: true, result }`
- `400` → `{ ok: false, error: { type, message } }` (ungültige Anfrage oder Toolfehler)
- `401` → nicht autorisiert
- `404` → Tool nicht verfügbar (nicht gefunden oder nicht in der Allowlist)
- `405` → Methode nicht zulässig

<div id="example">
  ## Beispiel
</div>

```bash
curl -sS http://127.0.0.1:18789/tools/invoke \
  -H 'Authorization: Bearer YOUR_TOKEN' \
  -H 'Content-Type: application/json' \
  -d '{
    "tool": "sessions_list",
    "action": "json",
    "args": {}
  }'
```
