---
title: Webhook
summary: "Webhook-Eingang für Wake- und isolierte Agent-Ausführungen"
read_when:
  - Beim Hinzufügen oder Ändern von Webhook-Endpunkten
  - Beim Anbinden externer Systeme an OpenClaw
---

<div id="webhooks">
  # Webhooks
</div>

Gateway kann einen einfachen HTTP-Webhook-Endpunkt für externe Auslöser bereitstellen.

<div id="enable">
  ## Aktivieren
</div>

```json5
{
  hooks: {
    enabled: true,
    token: "shared-secret",
    path: "/hooks"
  }
}
```

Hinweise:

* `hooks.token` ist erforderlich, wenn `hooks.enabled=true`.
* `hooks.path` ist standardmäßig auf `/hooks` gesetzt.

<div id="auth">
  ## Auth
</div>

Jede Anfrage muss das Hook-Token enthalten. Verwende bevorzugt Header:

* `Authorization: Bearer <token>` (empfohlen)
* `x-openclaw-token: <token>`
* `?token=<token>` (veraltet; gibt eine Warnung im Log aus und wird in einer zukünftigen Major-Version entfernt)

<div id="endpoints">
  ## Endpunkte
</div>

<div id="post-hookswake">
  ### `POST /hooks/wake`
</div>

Payload:

```json
{ "text": "System line", "mode": "now" }
```

* `text` **erforderlich** (string): Die Beschreibung des Ereignisses (z. B. „Neue E-Mail empfangen“).
* `mode` optional (`now` | `next-heartbeat`): Gibt an, ob ein sofortiger Herzschlag ausgelöst werden soll (Standard `now`) oder auf den nächsten periodischen Herzschlag gewartet wird.

Effekt:

* Reiht ein Systemereignis für die **Haupt**-Sitzung in die Warteschlange ein
* Wenn `mode=now`, wird ein sofortiger Herzschlag ausgelöst

<div id="post-hooksagent">
  ### `POST /hooks/agent`
</div>

Payload:

```json
{
  "message": "Run this",
  "name": "Email",
  "sessionKey": "hook:email:msg-123",
  "wakeMode": "now",
  "deliver": true,
  "channel": "last",
  "to": "+15551234567",
  "model": "openai/gpt-5.2-mini",
  "thinking": "low",
  "timeoutSeconds": 120
}
```

* `message` **erforderlich** (string): Der Prompt oder die Nachricht, die der agent verarbeiten soll.
* `name` optional (string): Menschenlesbarer Name für den Hook (z. B. „GitHub“), der als Präfix in Sitzungszusammenfassungen verwendet wird.
* `sessionKey` optional (string): Der Schlüssel zur Identifikation der Sitzung des agents. Standardmäßig ein zufälliges `hook:<uuid>`. Die Verwendung eines konsistenten Schlüssels ermöglicht eine mehrschrittige Unterhaltung im Hook-Kontext.
* `wakeMode` optional (`now` | `next-heartbeat`): Legt fest, ob ein sofortiger Herzschlag ausgelöst werden soll (Standard `now`) oder auf die nächste periodische Prüfung gewartet wird.
* `deliver` optional (boolean): Wenn `true`, wird die Antwort des agents an den Messaging-Kanal gesendet. Standard ist `true`. Antworten, die nur Herzschlag-Bestätigungen sind, werden automatisch übersprungen.
* `channel` optional (string): Der Messaging-Kanal für die Zustellung. Einer von: `last`, `whatsapp`, `telegram`, `discord`, `slack`, `mattermost` (Plugin), `signal`, `imessage`, `msteams`. Standard ist `last`.
* `to` optional (string): Die Empfänger-ID für den Kanal (z. B. Telefonnummer für WhatsApp/Signal, Chat-ID für Telegram, Channel-ID für Discord/Slack/Mattermost (Plugin), Konversations-ID für MS Teams). Standard ist der letzte Empfänger in der Hauptsitzung.
* `model` optional (string): Modell-Override (z. B. `anthropic/claude-3-5-sonnet` oder ein Alias). Muss in der Liste der erlaubten Modelle sein, falls eingeschränkt.
* `thinking` optional (string): Override für das Thinking-Level (z. B. `low`, `medium`, `high`).
* `timeoutSeconds` optional (number): Maximale Dauer für den Lauf des agents in Sekunden.

Effect:

* Führt einen **isolierten** agent-Turn aus (eigener Sitzungsschlüssel)
* Schreibt immer eine Zusammenfassung in die **Haupt**sitzung
* Wenn `wakeMode=now`, wird ein sofortiger Herzschlag ausgelöst

<div id="post-hooksname-mapped">
  ### `POST /hooks/<name>` (zugeordnet)
</div>

Benutzerdefinierte Hook-Namen werden über `hooks.mappings` aufgelöst (siehe Konfiguration). Ein Mapping kann beliebige Payloads in `wake`- oder `agent`-Aktionen umwandeln, optional mit Templates oder Code-Transforms.

Mapping-Optionen (Übersicht):

* `hooks.presets: ["gmail"]` aktiviert das eingebaute Gmail-Mapping.
* `hooks.mappings` ermöglicht dir, `match`, `action` und Templates in der Konfiguration zu definieren.
* `hooks.transformsDir` + `transform.module` lädt ein JS/TS-Modul für benutzerdefinierte Logik.
* Verwende `match.source`, um einen generischen Ingest-Endpunkt beizubehalten (payload-gesteuertes Routing).
* TS-Transforms erfordern einen TS-Loader (z. B. `bun` oder `tsx`) oder vorkompilierte `.js`-Dateien zur Laufzeit.
* Setze `deliver: true` + `channel`/`to` in Mappings, um Antworten an eine Chat-Oberfläche zu routen
  (`channel` hat standardmäßig den Wert `last` und fällt als Fallback auf WhatsApp zurück).
* `allowUnsafeExternalContent: true` deaktiviert den Sicherheits-Wrapper für externe Inhalte für diesen Hook
  (gefährlich; nur für vertrauenswürdige interne Quellen).
* `openclaw webhooks gmail setup` schreibt die `hooks.gmail`-Konfiguration für `openclaw webhooks gmail run`.
  Siehe [Gmail Pub/Sub](/de/automation/gmail-pubsub) für den kompletten Gmail-Watch-Flow.

<div id="responses">
  ## Antworten
</div>

* `200` für `/hooks/wake`
* `202` für `/hooks/agent` (asynchroner Lauf gestartet)
* `401` bei einem Authentifizierungsfehler
* `400` bei ungültiger Payload
* `413` bei zu großer Payload

<div id="examples">
  ## Beispiele
</div>

```bash
curl -X POST http://127.0.0.1:18789/hooks/wake \
  -H 'Authorization: Bearer SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"text":"Neue E-Mail empfangen","mode":"now"}'
```

```bash
curl -X POST http://127.0.0.1:18789/hooks/agent \
  -H 'x-openclaw-token: SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"message":"Summarize inbox","name":"Email","wakeMode":"next-heartbeat"}'
```

<div id="use-a-different-model">
  ### Anderes Modell verwenden
</div>

Füge `model` zur agent-Payload (oder zum Mapping) hinzu, um das Modell für diesen Durchlauf zu überschreiben:

```bash
curl -X POST http://127.0.0.1:18789/hooks/agent \
  -H 'x-openclaw-token: SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"message":"Summarize inbox","name":"Email","model":"openai/gpt-5.2-mini"}'
```

Wenn du `agents.defaults.models` erzwingst, stelle sicher, dass das überschreibende Modell dort enthalten ist.

```bash
curl -X POST http://127.0.0.1:18789/hooks/gmail \
  -H 'Authorization: Bearer SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"source":"gmail","messages":[{"from":"Ada","subject":"Hello","snippet":"Hi"}]}'
```

<div id="security">
  ## Sicherheit
</div>

* Betreibe Hook-Endpunkte nur über Loopback, Tailnet oder einen vertrauenswürdigen Reverse Proxy.
* Verwende ein dediziertes Hook-Token; verwende Gateway-Auth-Tokens nicht mehrfach.
* Vermeide es, sensible rohe Payloads in Webhook-Logs zu protokollieren.
* Hook-Payloads werden standardmäßig als nicht vertrauenswürdig behandelt und durch Sicherheitsgrenzen abgeschottet.
  Wenn du dies für einen bestimmten Hook deaktivieren musst, setze `allowUnsafeExternalContent: true`
  im Mapping dieses Hooks (gefährlich).