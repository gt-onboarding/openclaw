---
title: Kanalrouting
summary: "Routing-Regeln pro Kanal (WhatsApp, Telegram, Discord, Slack) und gemeinsamer Kontext"
read_when:
  - Beim Ändern des Kanalroutings oder des Posteingangsverhaltens
---

<div id="channels-routing">
  # Kanäle &amp; Routing
</div>

OpenClaw leitet Antworten **an den Kanal zurück, aus dem eine Nachricht kam**. Das
Modell wählt keinen Kanal; das Routing ist deterministisch und wird durch die
Hostkonfiguration gesteuert.

<div id="key-terms">
  ## Wichtige Begriffe
</div>

* **Channel**: `whatsapp`, `telegram`, `discord`, `slack`, `signal`, `imessage`, `webchat`.
* **AccountId**: kanalspezifische Konto‑Instanz (falls unterstützt).
* **AgentId**: ein isolierter Arbeitsbereich + Sitzungsspeicher („Gehirn“).
* **SessionKey**: der Bucket‑Key, der zur Speicherung von Kontext und zur Steuerung der Nebenläufigkeit verwendet wird.

<div id="session-key-shapes-examples">
  ## Strukturen von Sitzungsschlüsseln (Beispiele)
</div>

Direktnachrichten werden in die **Haupt**sitzung des Agents zusammengeführt:

* `agent:<agentId>:<mainKey>` (Standardwert: `agent:main:main`)

Gruppen und Kanäle bleiben pro Kanal isoliert:

* Gruppen: `agent:<agentId>:<channel>:group:<id>`
* Kanäle/Räume: `agent:<agentId>:<channel>:channel:<id>`

Threads:

* Slack-/Discord-Threads fügen `:thread:<threadId>` an den Basisschlüssel an.
* Telegram-Forenthemen betten `:topic:<topicId>` in den Gruppenschlüssel ein.

Beispiele:

* `agent:main:telegram:group:-1001234567890:topic:42`
* `agent:main:discord:channel:123456:thread:987654`

<div id="routing-rules-how-an-agent-is-chosen">
  ## Routing-Regeln (wie ein agent ausgewählt wird)
</div>

Routing wählt **einen agent** für jede eingehende Nachricht:

1. **Exakte Peer-Übereinstimmung** (`bindings` mit `peer.kind` + `peer.id`).
2. **Guild-Übereinstimmung** (Discord) über `guildId`.
3. **Team-Übereinstimmung** (Slack) über `teamId`.
4. **Account-Übereinstimmung** (`accountId` auf dem Kanal).
5. **Kanal-Übereinstimmung** (beliebiger Account auf diesem Kanal).
6. **Standard-agent** (`agents.list[].default`, sonst erster Listeneintrag, Fallback auf `main`).

Der ausgewählte agent bestimmt, welcher Arbeitsbereich und welcher Sitzungsspeicher verwendet werden.

<div id="broadcast-groups-run-multiple-agents">
  ## Broadcast-Gruppen (mehrere Agenten ausführen)
</div>

Broadcast-Gruppen ermöglichen dir, **mehrere Agenten** für denselben Peer gleichzeitig auszuführen, **wenn OpenClaw normalerweise antworten würde** (zum Beispiel in WhatsApp-Gruppen nach Mention-/Aktivierungs-Gating).

Konfiguration:

```json5
{
  broadcast: {
    strategy: "parallel",
    "120363403215116621@g.us": ["alfred", "baerbel"],
    "+15555550123": ["support", "logger"]
  }
}
```

Siehe auch: [Broadcast-Gruppen](/de/broadcast-groups).

<div id="config-overview">
  ## Konfigurationsübersicht
</div>

* `agents.list`: benannte Agent-Definitionen (Arbeitsbereich, Modell, etc.).
* `bindings`: ordnet eingehende Kanäle/Konten/Peers Agenten zu.

Beispiel:

```json5
{
  agents: {
    list: [
      { id: "support", name: "Support", workspace: "~/.openclaw/workspace-support" }
    ]
  },
  bindings: [
    { match: { channel: "slack", teamId: "T123" }, agentId: "support" },
    { match: { channel: "telegram", peer: { kind: "group", id: "-100123" } }, agentId: "support" }
  ]
}
```

<div id="session-storage">
  ## Sitzungsspeicher
</div>

Sitzungsspeicher befinden sich im State-Verzeichnis (Standard: `~/.openclaw`):

* `~/.openclaw/agents/<agentId>/sessions/sessions.json`
* JSONL-Transkripte liegen direkt neben dem Speicher

Du kannst den Speicherpfad über `session.store` und `{agentId}`-Templating überschreiben.

<div id="webchat-behavior">
  ## WebChat-Verhalten
</div>

WebChat ist an den **ausgewählten agent** gebunden und verwendet standardmäßig dessen Hauptsitzung. Dadurch kannst du kanalübergreifenden Kontext für diesen agent an einem Ort einsehen.

<div id="reply-context">
  ## Antwortkontext
</div>

Eingehende Antworten enthalten:

* `ReplyToId`, `ReplyToBody` und `ReplyToSender`, sofern verfügbar.
* Der zitierte Kontext wird als `[Replying to ...]`-Block an `Body` angehängt.

Dies ist über alle Kanäle hinweg konsistent.