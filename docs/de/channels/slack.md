---
title: Slack
summary: "Slack-Konfiguration für Socket- oder HTTP-Webhook-Modus"
read_when: "Beim Einrichten von Slack oder beim Debuggen des Slack-Socket-/HTTP-Modus"
---

<div id="slack">
  # Slack
</div>

<div id="socket-mode-default">
  ## Socket-Modus (Voreinstellung)
</div>

<div id="quick-setup-beginner">
  ### Schnelle Einrichtung (für Einsteiger)
</div>

1. Erstelle eine Slack-App und aktiviere den **Socket Mode**.
2. Erstelle ein **App-Token** (`xapp-...`) und ein **Bot-Token** (`xoxb-...`).
3. Konfiguriere die Tokens in OpenClaw und starte das Gateway.

Minimale Konfiguration:

```json5
{
  channels: {
    slack: {
      enabled: true,
      appToken: "xapp-...",
      botToken: "xoxb-..."
    }
  }
}
```

<div id="setup">
  ### Einrichtung
</div>

1. Erstelle eine Slack-App („From scratch“) unter https://api.slack.com/apps.
2. **Socket Mode** → aktivieren. Dann gehe zu **Basic Information** → **App-Level Tokens** → **Generate Token and Scopes** mit Scope `connections:write`. Kopiere den **App Token** (`xapp-...`).
3. **OAuth &amp; Permissions** → füge Bot-Token-Scopes hinzu (verwende das Manifest unten). Klicke auf **Install to Workspace**. Kopiere den **Bot User OAuth Token** (`xoxb-...`).
4. Optional: **OAuth &amp; Permissions** → füge **User Token Scopes** hinzu (siehe die schreibgeschützte Liste unten). Installiere die App erneut und kopiere den **User OAuth Token** (`xoxp-...`).
5. **Event Subscriptions** → aktiviere Events und abonniere:
   * `message.*` (einschließlich Bearbeitungen/Löschungen/Thread-Broadcasts)
   * `app_mention`
   * `reaction_added`, `reaction_removed`
   * `member_joined_channel`, `member_left_channel`
   * `channel_rename`
   * `pin_added`, `pin_removed`
6. Lade den Bot in die Channels ein, die er lesen soll.
7. Slash Commands → erstelle `/openclaw`, wenn du `channels.slack.slashCommand` verwendest. Wenn du native Commands aktivierst, füge einen Slash-Command pro eingebautem Command hinzu (gleiche Namen wie `/help`). Native ist für Slack standardmäßig deaktiviert, es sei denn, du setzt `channels.slack.commands.native: true` (globales `commands.native` ist `"auto"`, was Slack deaktiviert lässt).
8. App Home → aktiviere den **Messages-Tab**, damit Nutzer dem Bot DMs schicken können.

Verwende das Manifest unten, damit Scopes und Events konsistent bleiben.

Multi-Account-Unterstützung: Verwende `channels.slack.accounts` mit kontospezifischen Tokens und optionalem `name`. Siehe [`gateway/configuration`](/de/gateway/configuration#telegramaccounts--discordaccounts--slackaccounts--signalaccounts--imessageaccounts) für das gemeinsame Muster.

<div id="openclaw-config-minimal">
  ### OpenClaw-Konfiguration (minimal)
</div>

Setze die Token per Umgebungsvariablen (empfohlen):

* `SLACK_APP_TOKEN=xapp-...`
* `SLACK_BOT_TOKEN=xoxb-...`

Oder über die Konfiguration:

```json5
{
  channels: {
    slack: {
      enabled: true,
      appToken: "xapp-...",
      botToken: "xoxb-..."
    }
  }
}
```

<div id="user-token-optional">
  ### User-Token (optional)
</div>

OpenClaw kann ein Slack-User-Token (`xoxp-...`) für Leseoperationen verwenden
(Verlauf, Pins, Reaktionen, Emoji, Mitgliederinformationen). Standardmäßig
bleibt dies nur mit Leserechten: Lesezugriffe verwenden bevorzugt das
User-Token, wenn vorhanden, und Schreibzugriffe nutzen weiterhin das Bot-Token,
sofern du nicht ausdrücklich etwas anderes konfigurierst. Selbst mit
`userTokenReadOnly: false` bleibt das Bot-Token für Schreibzugriffe
bevorzugt, wenn es verfügbar ist.

User-Tokens werden in der Konfigurationsdatei festgelegt (keine Unterstützung
für Umgebungsvariablen). Für mehrere Accounts setze
`channels.slack.accounts.<id>.userToken`.

Beispiel mit Bot-, App- und User-Tokens:

```json5
{
  channels: {
    slack: {
      enabled: true,
      appToken: "xapp-...",
      botToken: "xoxb-...",
      userToken: "xoxp-..."
    }
  }
}
```

Beispiel mit explizit gesetztem userTokenReadOnly (Schreibzugriffe mit dem User-Token zulassen):

```json5
{
  channels: {
    slack: {
      enabled: true,
      appToken: "xapp-...",
      botToken: "xoxb-...",
      userToken: "xoxp-...",
      userTokenReadOnly: false
    }
  }
}
```

<div id="token-usage">
  #### Token-Verwendung
</div>

* Leseoperationen (Verlauf, Reaktionsliste, Pin-Liste, Emoji-Liste, Mitgliederinformationen,
  Suche) bevorzugen das Benutzer-Token, falls konfiguriert, ansonsten das Bot-Token.
* Schreiboperationen (Senden/Bearbeiten/Löschen von Nachrichten, Hinzufügen/Entfernen von Reaktionen, Anpinnen/Lösen von Nachrichten,
  Datei-Uploads) verwenden standardmäßig das Bot-Token. Wenn `userTokenReadOnly: false` gesetzt ist und
  kein Bot-Token verfügbar ist, fällt OpenClaw auf das Benutzer-Token zurück.

<div id="history-context">
  ### Verlaufskontext
</div>

* `channels.slack.historyLimit` (oder `channels.slack.accounts.*.historyLimit`) steuert, wie viele der jüngsten Channel-/Gruppennachrichten in den Prompt aufgenommen werden.
* Fällt auf `messages.groupChat.historyLimit` zurück. Setze `0`, um zu deaktivieren (Standard: 50).

<div id="http-mode-events-api">
  ## HTTP-Modus (Events API)
</div>

Verwende den HTTP-Webhook-Modus, wenn dein Gateway von Slack aus über HTTPS erreichbar ist (typisch bei Server-Deployments).
Der HTTP-Modus verwendet die Events API + Interactivity + Slash Commands mit einer gemeinsamen Request-URL.

<div id="setup">
  ### Einrichtung
</div>

1. Erstelle eine Slack-App und **deaktiviere Socket Mode** (optional, wenn du nur HTTP verwendest).
2. **Basic Information** → kopiere das **Signing Secret**.
3. **OAuth &amp; Permissions** → installiere die App und kopiere den **Bot User OAuth Token** (`xoxb-...`).
4. **Event Subscriptions** → aktiviere Events und setze die **Request URL** auf deinen Gateway-Webhook-Pfad (Standard: `/slack/events`).
5. **Interactivity &amp; Shortcuts** → aktiviere und setze dieselbe **Request URL**.
6. **Slash Commands** → setze für deine Slash-Commands dieselbe **Request URL**.

Beispiel für eine Request-URL:
`https://gateway-host/slack/events`

### Minimale OpenClaw-Konfiguration

```json5
{
  channels: {
    slack: {
      enabled: true,
      mode: "http",
      botToken: "xoxb-...",
      signingSecret: "your-signing-secret",
      webhookPath: "/slack/events"
    }
  }
}
```

Multi-Account-HTTP-Modus: Setze `channels.slack.accounts.&lt;id&gt;.mode = "http"` und gib für jeden Account einen eindeutigen
`webhookPath` an, damit jede Slack-App auf ihre eigene URL verweisen kann.

<div id="manifest-optional">
  ### Manifest (optional)
</div>

Verwende dieses Slack-App-Manifest, um die App schnell anzulegen (passe Namen/Befehl bei Bedarf an). Füge die
User-Scopes hinzu, wenn du ein User-Token konfigurieren möchtest.

```json
{
  "display_information": {
    "name": "OpenClaw",
    "description": "Slack connector for OpenClaw"
  },
  "features": {
    "bot_user": {
      "display_name": "OpenClaw",
      "always_online": false
    },
    "app_home": {
      "messages_tab_enabled": true,
      "messages_tab_read_only_enabled": false
    },
    "slash_commands": [
      {
        "command": "/openclaw",
        "description": "Send a message to OpenClaw",
        "should_escape": false
      }
    ]
  },
  "oauth_config": {
    "scopes": {
      "bot": [
        "chat:write",
        "channels:history",
        "channels:read",
        "groups:history",
        "groups:read",
        "groups:write",
        "im:history",
        "im:read",
        "im:write",
        "mpim:history",
        "mpim:read",
        "mpim:write",
        "users:read",
        "app_mentions:read",
        "reactions:read",
        "reactions:write",
        "pins:read",
        "pins:write",
        "emoji:read",
        "commands",
        "files:read",
        "files:write"
      ],
      "user": [
        "channels:history",
        "channels:read",
        "groups:history",
        "groups:read",
        "im:history",
        "im:read",
        "mpim:history",
        "mpim:read",
        "users:read",
        "reactions:read",
        "pins:read",
        "emoji:read",
        "search:read"
      ]
    }
  },
  "settings": {
    "socket_mode_enabled": true,
    "event_subscriptions": {
      "bot_events": [
        "app_mention",
        "message.channels",
        "message.groups",
        "message.im",
        "message.mpim",
        "reaction_added",
        "reaction_removed",
        "member_joined_channel",
        "member_left_channel",
        "channel_rename",
        "pin_added",
        "pin_removed"
      ]
    }
  }
}
```

Wenn du native Befehle aktivierst, füge für jeden Befehl, den du verfügbar machen möchtest, einen `slash_commands`-Eintrag hinzu (entsprechend der `/help`-Liste). Du kannst dies mit `channels.slack.commands.native` überschreiben.

<div id="scopes-current-vs-optional">
  ## Scopes (aktuell vs. optional)
</div>

Die Conversations-API von Slack verwendet typspezifische Scopes: Du benötigst nur die Scopes für die Konversationstypen, mit denen du tatsächlich arbeitest (channels, groups, im, mpim). Eine Übersicht findest du unter https://docs.slack.dev/apis/web-api/using-the-conversations-api/.

<div id="bot-token-scopes-required">
  ### Bot-Token-Scopes (erforderlich)
</div>

* `chat:write` (Nachrichten über `chat.postMessage` senden/aktualisieren/löschen)
  https://docs.slack.dev/reference/methods/chat.postMessage
* `im:write` (Benutzer-DMs über `conversations.open` öffnen)
  https://docs.slack.dev/reference/methods/conversations.open
* `channels:history`, `groups:history`, `im:history`, `mpim:history`
  https://docs.slack.dev/reference/methods/conversations.history
* `channels:read`, `groups:read`, `im:read`, `mpim:read`
  https://docs.slack.dev/reference/methods/conversations.info
* `users:read` (Benutzersuche)
  https://docs.slack.dev/reference/methods/users.info
* `reactions:read`, `reactions:write` (`reactions.get` / `reactions.add`)
  https://docs.slack.dev/reference/methods/reactions.get
  https://docs.slack.dev/reference/methods/reactions.add
* `pins:read`, `pins:write` (`pins.list` / `pins.add` / `pins.remove`)
  https://docs.slack.dev/reference/scopes/pins.read
  https://docs.slack.dev/reference/scopes/pins.write
* `emoji:read` (`emoji.list`)
  https://docs.slack.dev/reference/scopes/emoji.read
* `files:write` (Uploads über `files.uploadV2`)
  https://docs.slack.dev/messaging/working-with-files/#upload

<div id="user-token-scopes-optional-read-only-by-default">
  ### User-Token-Scopes (optional, standardmäßig schreibgeschützt)
</div>

Füge diese Berechtigungen unter **User Token Scopes** hinzu, wenn du `channels.slack.userToken` konfigurierst.

* `channels:history`, `groups:history`, `im:history`, `mpim:history`
* `channels:read`, `groups:read`, `im:read`, `mpim:read`
* `users:read`
* `reactions:read`
* `pins:read`
* `emoji:read`
* `search:read`

<div id="not-needed-today-but-likely-future">
  ### Heute nicht erforderlich (aber wahrscheinlich in Zukunft)
</div>

* `mpim:write` (nur, falls wir Gruppen-DMs öffnen/DMs starten über `conversations.open` hinzufügen)
* `groups:write` (nur, falls wir private-Channel-Verwaltung hinzufügen: erstellen/umbenennen/einladen/archivieren)
* `chat:write.public` (nur, falls wir in Channels posten wollen, in denen der Bot nicht Mitglied ist)
  https://docs.slack.dev/reference/scopes/chat.write.public
* `users:read.email` (nur, falls wir E-Mail-Felder aus `users.info` benötigen)
  https://docs.slack.dev/changelog/2017-04-narrowing-email-access
* `files:read` (nur, falls wir anfangen, Datei-Metadaten aufzulisten/zu lesen)

<div id="config">
  ## Konfiguration
</div>

Slack verwendet ausschließlich Socket Mode (kein HTTP-Webhook-Server). Gib beide Tokens an:

```json
{
  "slack": {
    "enabled": true,
    "botToken": "xoxb-...",
    "appToken": "xapp-...",
    "groupPolicy": "allowlist",
    "dm": {
      "enabled": true,
      "policy": "pairing",
      "allowFrom": ["U123", "U456", "*"],
      "groupEnabled": false,
      "groupChannels": ["G123"],
      "replyToMode": "all"
    },
    "channels": {
      "C123": { "allow": true, "requireMention": true },
      "#general": {
        "allow": true,
        "requireMention": true,
        "users": ["U123"],
        "skills": ["search", "docs"],
        "systemPrompt": "Keep answers short."
      }
    },
    "reactionNotifications": "own",
    "reactionAllowlist": ["U123"],
    "replyToMode": "off",
    "actions": {
      "reactions": true,
      "messages": true,
      "pins": true,
      "memberInfo": true,
      "emojiList": true
    },
    "slashCommand": {
      "enabled": true,
      "name": "openclaw",
      "sessionPrefix": "slack:slash",
      "ephemeral": true
    },
    "textChunkLimit": 4000,
    "mediaMaxMb": 20
  }
}
```

Tokens können auch über Umgebungsvariablen bereitgestellt werden:

* `SLACK_BOT_TOKEN`
* `SLACK_APP_TOKEN`

Ack-Reaktionen werden global über `messages.ackReaction` +
`messages.ackReactionScope` gesteuert. Verwende `messages.removeAckAfterReply`, um die
Ack-Reaktion zu löschen, nachdem der Bot geantwortet hat.

<div id="limits">
  ## Limits
</div>

* Ausgehender Text wird in Segmente mit der Länge `channels.slack.textChunkLimit` (Standardwert 4000) aufgeteilt.
* Optionales Zeilenumbruch-Chunking: Setze `channels.slack.chunkMode="newline"`, um vor dem längenbasierten Chunking an Leerzeilen (Absatzgrenzen) zu splitten.
* Medien-Uploads sind durch `channels.slack.mediaMaxMb` begrenzt (Standardwert 20).

<div id="reply-threading">
  ## Antwort-Threading
</div>

Standardmäßig antwortet OpenClaw im Hauptkanal. Verwende `channels.slack.replyToMode`, um das automatische Threading zu steuern:

| Modus | Verhalten |
| --- | --- |
| `off` | **Standard.** Antwort im Hauptkanal. Ein Thread wird nur verwendet, wenn die auslösende Nachricht bereits in einem Thread war. |
| `first` | Die erste Antwort geht in den Thread (unter die auslösende Nachricht), nachfolgende Antworten gehen in den Hauptkanal. Nützlich, um den Kontext sichtbar zu halten und gleichzeitig zu viele Threads zu vermeiden. |
| `all` | Alle Antworten gehen in den Thread. Hält Unterhaltungen kompakt, kann aber die Sichtbarkeit verringern. |

Der Modus gilt sowohl für automatische Antworten als auch für Agent-Tool-Aufrufe (`slack sendMessage`).

<div id="per-chat-type-threading">
  ### Threading je Chat-Typ
</div>

Du kannst das Threading-Verhalten je Chat-Typ konfigurieren, indem du `channels.slack.replyToModeByChatType` setzt:

```json5
{
  channels: {
    slack: {
      replyToMode: "off",        // default for channels
      replyToModeByChatType: {
        direct: "all",           // DMs always thread
        group: "first"           // Gruppen-DMs/MPIM Thread bei erster Antwort
      },
    }
  }
}
```

Unterstützte Chattypen:

* `direct`: 1:1-DMs (Slack `im`)
* `group`: Gruppen-DMs / MPIMs (Slack `mpim`)
* `channel`: Standard-Kanäle (öffentlich/privat)

Prioritätsreihenfolge:

1. `replyToModeByChatType.<chatType>`
2. `replyToMode`
3. anbieter-Standardwert (`off`)

Das veraltete `channels.slack.dm.replyToMode` wird weiterhin als Fallback für `direct` akzeptiert, wenn keine Chattyp-spezifische Einstellung gesetzt ist.

Beispiele:

Nur DMs in Threads:

```json5
{
  channels: {
    slack: {
      replyToMode: "off",
      replyToModeByChatType: { direct: "all" }
    }
  }
}
```

Gruppen-DMs als Threads führen, Channels aber auf der Root-Ebene belassen:

```json5
{
  channels: {
    slack: {
      replyToMode: "off",
      replyToModeByChatType: { group: "first" }
    }
  }
}
```

In Channels Threads verwenden, DMs im Hauptverlauf lassen:

```json5
{
  channels: {
    slack: {
      replyToMode: "first",
      replyToModeByChatType: { direct: "off", group: "off" }
    }
  }
}
```

<div id="manual-threading-tags">
  ### Manuelle Threading-Tags
</div>

Für eine fein abgestimmte Steuerung verwende diese Tags in Agent-Antworten:

* `[[reply_to_current]]` — auf die auslösende Nachricht antworten (Thread starten/fortsetzen).
* `[[reply_to:<id>]]` — auf eine bestimmte Nachrichten-ID antworten.

<div id="sessions-routing">
  ## Sitzungen + Routing
</div>

* DMs teilen sich die `main`-Sitzung (ähnlich wie bei WhatsApp/Telegram).
* Kanäle werden auf `agent:<agentId>:slack:channel:<channelId>`-Sitzungen abgebildet.
* Slash-Commands verwenden `agent:<agentId>:slack:slash:<userId>`-Sitzungen (Präfix konfigurierbar über `channels.slack.slashCommand.sessionPrefix`).
* Wenn Slack keinen `channel_type` liefert, leitet OpenClaw ihn aus dem Kanal-ID-Präfix (`D`, `C`, `G`) ab und fällt zur Stabilisierung der Sitzungsschlüssel standardmäßig auf `channel` zurück.
* Die Registrierung nativer Commands verwendet `commands.native` (globaler Standard `"auto"` → Slack aus) und kann pro Arbeitsbereich mit `channels.slack.commands.native` überschrieben werden. Textbefehle erfordern eigenständige `/...`-Nachrichten und können mit `commands.text: false` deaktiviert werden. Slack-Slash-Commands werden in der Slack-App verwaltet und nicht automatisch entfernt. Verwende `commands.useAccessGroups: false`, um Zugriffsgruppenprüfungen für Befehle zu umgehen.
* Vollständige Befehlsliste + Konfiguration: [Slash-Commands](/de/tools/slash-commands)

<div id="dm-security-pairing">
  ## DM-Sicherheit (Kopplung)
</div>

* Standard: `channels.slack.dm.policy="pairing"` — unbekannte DM-Absender erhalten einen Kopplungscode (verfällt nach 1 Stunde).
* Freigabe über: `openclaw pairing approve slack <code>`.
* Um alle zuzulassen: setze `channels.slack.dm.policy="open"` (Einstellung &quot;open&quot; erlaubt uneingeschränkte Nachrichtenannahme von beliebigen Nutzern) und `channels.slack.dm.allowFrom=["*"]`.
* `channels.slack.dm.allowFrom` akzeptiert Benutzer-IDs, @handles oder E-Mail-Adressen (beim Start aufgelöst, wenn Token dies erlauben). Der Assistent akzeptiert Benutzernamen und löst sie während der Einrichtung in IDs auf, wenn Token dies erlauben.

<div id="group-policy">
  ## Gruppenrichtlinie
</div>

* `channels.slack.groupPolicy` steuert die Kanalbehandlung (`open|disabled|allowlist`).
* `allowlist` erfordert, dass Kanäle in `channels.slack.channels` aufgelistet werden.
* Wenn du nur `SLACK_BOT_TOKEN`/`SLACK_APP_TOKEN` setzt und keinen `channels.slack`-Abschnitt anlegst,
  setzt die Laufzeitumgebung `groupPolicy` standardmäßig auf `open` (d. h. Nachrichtenannahme ohne Einschränkungen von allen Nutzer:innen). Füge `channels.slack.groupPolicy`,
  `channels.defaults.groupPolicy` oder eine Kanal-Allowlist hinzu, um das Verhalten einzuschränken.
* Der Konfigurationsassistent akzeptiert `#channel`-Namen und löst sie, wenn möglich, in IDs auf
  (öffentlich + privat); wenn mehrere Treffer existieren, wird der aktive Kanal bevorzugt.
* Beim Start löst OpenClaw Kanal- und Benutzernamen in Allowlists in IDs auf (wenn Tokens dies erlauben)
  und protokolliert das Mapping; nicht auflösbare Einträge bleiben wie eingegeben.
* Um **keine Kanäle** zuzulassen, setze `channels.slack.groupPolicy: "disabled"` (oder verwende eine leere Allowlist).

Kanaloptionen (`channels.slack.channels.<id>` oder `channels.slack.channels.<name>`):

* `allow`: Kanal erlauben/ablehnen, wenn `groupPolicy="allowlist"` ist.
* `requireMention`: Erwähnungspflicht für den Kanal.
* `tools`: optionale kanalbezogene Überschreibungen der Tool-Richtlinie (`allow`/`deny`/`alsoAllow`).
* `toolsBySender`: optionale senderbezogene Überschreibungen der Tool-Richtlinie innerhalb des Kanals (Keys sind Sender-IDs/@Handles/E-Mails; `"*"`-Wildcard wird unterstützt).
* `allowBots`: botverfasste Nachrichten in diesem Kanal erlauben (Standard: false).
* `users`: optionale kanalbezogene Benutzer-Allowlist.
* `skills`: Fähigkeitenfilter (weglassen = alle Fähigkeiten, leer = keine).
* `systemPrompt`: zusätzlicher System-Prompt für den Kanal (kombiniert mit Thema/Zweck).
* `enabled`: auf `false` setzen, um den Kanal zu deaktivieren.

<div id="delivery-targets">
  ## Zieladressen
</div>

Verwende diese bei cron-/CLI-Sendungen:

* `user:&lt;id&gt;` für DMs
* `channel:&lt;id&gt;` für Kanäle

<div id="tool-actions">
  ## Tool-Aktionen
</div>

Slack-Tool-Aktionen können über `channels.slack.actions.*` gesteuert werden:

| Aktionsgruppe | Standard | Hinweise |
| --- | --- | --- |
| reactions | enabled | Reagieren + Reaktionen auflisten |
| messages | enabled | Lesen/Senden/Bearbeiten/Löschen |
| pins | enabled | Anheften/Lösen/Auflisten |
| memberInfo | enabled | Mitgliederinformationen |
| emojiList | enabled | Liste benutzerdefinierter Emojis |

<div id="security-notes">
  ## Sicherheitshinweise
</div>

* Schreiboperationen verwenden standardmäßig das Bot-Token, sodass zustandsverändernde Aktionen auf die Berechtigungen und die Identität des App-Bots beschränkt bleiben.
* Setzt du `userTokenReadOnly: false`, kann das User-Token für Schreiboperationen verwendet werden, wenn kein Bot-Token verfügbar ist. Das bedeutet, dass Aktionen mit den Rechten des installierenden Nutzers ausgeführt werden. Behandle das User-Token als hoch privilegiert und halte Action-Gates und Allowlists streng.
* Wenn du Schreibzugriffe mit dem User-Token aktivierst, stelle sicher, dass das User-Token die erwarteten Write-Scopes enthält (`chat:write`, `reactions:write`, `pins:write`, `files:write`), andernfalls schlagen diese Operationen fehl.

<div id="notes">
  ## Hinweise
</div>

* Erwähnungs-Gating wird über `channels.slack.channels` gesteuert (setze `requireMention` auf `true`); `agents.list[].groupChat.mentionPatterns` (oder `messages.groupChat.mentionPatterns`) zählen ebenfalls als Erwähnungen.
* Multi-Agent-Override: Setze agent-spezifische Muster auf `agents.list[].groupChat.mentionPatterns`.
* Reaktionsbenachrichtigungen folgen `channels.slack.reactionNotifications` (verwende `reactionAllowlist` mit Modus `allowlist`).
* Von Bots verfasste Nachrichten werden standardmäßig ignoriert; aktiviere sie über `channels.slack.allowBots` oder `channels.slack.channels.<id>.allowBots`.
* Warnung: Wenn du Antworten auf andere Bots zulässt (`channels.slack.allowBots=true` oder `channels.slack.channels.<id>.allowBots=true`), verhindere Bot-zu-Bot-Antwortschleifen mit `requireMention`, `channels.slack.channels.<id>.users`-Allowlists und/oder klar definierten Guardrails in `AGENTS.md` und `SOUL.md`.
* Für das Slack-Tool findest du die Semantik zur Entfernung von Reaktionen unter [/tools/reactions](/de/tools/reactions).
* Attachments werden in den Medienspeicher heruntergeladen, sofern dies erlaubt ist und sie unterhalb der Größengrenze liegen.