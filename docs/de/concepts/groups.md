---
title: Gruppen
summary: "Gruppenchat-Verhalten über verschiedene Plattformen hinweg (WhatsApp/Telegram/Discord/Slack/Signal/iMessage/Microsoft Teams)"
read_when:
  - Änderung des Gruppenchat-Verhaltens oder der Mention-Gating-Regeln
---

<div id="groups">
  # Gruppen
</div>

OpenClaw behandelt Gruppenchats über alle Kanäle hinweg einheitlich: WhatsApp, Telegram, Discord, Slack, Signal, iMessage, Microsoft Teams.

<div id="beginner-intro-2-minutes">
  ## Kurze Einführung für Einsteiger (2 Minuten)
</div>

OpenClaw „lebt“ in deinen eigenen Messaging-Accounts. Es gibt keinen separaten WhatsApp-Bot-User.
Wenn **du** in einer Gruppe bist, kann OpenClaw diese Gruppe sehen und dort antworten.

Standardverhalten:

* Gruppen sind eingeschränkt konfiguriert (`groupPolicy: "allowlist"`).
* Antworten erfordern eine Erwähnung, es sei denn, du deaktivierst Mention-Gating explizit.

Bedeutung: Absender, die in der Allowlist stehen, können OpenClaw durch eine Erwähnung triggern.

> TL;DR
>
> * **DM-Zugriff** wird über `*.allowFrom` gesteuert.
> * **Gruppenzugriff** wird über `*.groupPolicy` + Allowlists (`*.groups`, `*.groupAllowFrom`) gesteuert.
> * **Auslösen von Antworten** wird über Mention-Gating (`requireMention`, `/activation`) gesteuert.

Kurzablauf (was mit einer Gruppennachricht passiert):

```
groupPolicy? disabled -> drop
groupPolicy? allowlist -> group allowed? no -> drop
requireMention? yes -> mentioned? no -> store for context only
otherwise -> reply
```

![Gruppen-Nachrichtenfluss](/images/groups-flow.svg)

Wenn du Folgendes möchtest ...

| Ziel                                                    | Was du konfigurieren musst                                 |
| ------------------------------------------------------- | ---------------------------------------------------------- |
| Alle Gruppen erlauben, aber nur auf @mentions antworten | `groups: { "*": { requireMention: true } }`                |
| Alle Gruppen-Antworten deaktivieren                     | `groupPolicy: "disabled"`                                  |
| Nur bestimmte Gruppen                                   | `groups: { "<group-id>": { ... } }` (kein `"*"`-Schlüssel) |
| Nur du kannst in Gruppen triggern                       | `groupPolicy: "allowlist"`, `groupAllowFrom: ["+1555..."]` |

<div id="session-keys">
  ## Sitzungsschlüssel
</div>

* Gruppensitzungen verwenden Sitzungsschlüssel vom Typ `agent:<agentId>:<channel>:group:<id>` (Räume/Kanäle verwenden `agent:<agentId>:<channel>:channel:<id>`).
* Themen in Telegram-Foren hängen `:topic:<threadId>` an die Gruppen-ID an, sodass jedes Thema seine eigene Sitzung hat.
* Direkte Chats verwenden die Hauptsitzung (oder eine pro Absender, falls entsprechend konfiguriert).
* Herzschlag-Ereignisse werden für Gruppensitzungen ausgelassen.

<div id="pattern-personal-dms-public-groups-single-agent">
  ## Pattern: persönliche DMs + öffentliche Gruppen (einzelner Agent)
</div>

Ja – das funktioniert gut, wenn dein „persönlicher“ Nachrichtenverkehr aus **DMs** und dein „öffentlicher“ Nachrichtenverkehr aus **Gruppen** besteht.

Grund: Im Single-Agent-Modus landen DMs typischerweise im **main**-Sitzungsschlüssel (`agent:main:main`), während Gruppen immer **non-main**-Sitzungsschlüssel verwenden (`agent:main:<channel>:group:<id>`). Wenn du Sandboxing mit `mode: "non-main"` aktivierst, laufen diese Gruppen-Sitzungen in Docker, während deine Haupt-DM-Sitzung auf dem Host bleibt.

So erhältst du ein gemeinsames Agent-„Gehirn“ (geteilter arbeitsbereich + Memory), aber zwei Ausführungsmodi:

* **DMs**: volle Tools (Host)
* **Gruppen**: sandbox + eingeschränkte Tools (Docker)

> Wenn du wirklich getrennte Arbeitsbereiche/Personas brauchst („persönlich“ und „öffentlich“ dürfen sich niemals mischen), verwende einen zweiten Agent + Bindings. Siehe [Multi-Agent Routing](/de/concepts/multi-agent).

Beispiel (DMs auf dem Host, Gruppen in der sandbox + reine Messaging-Tools):

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main", // groups/channels are non-main -> sandboxed
        scope: "session", // strongest isolation (one container per group/channel)
        workspaceAccess: "none"
      }
    }
  },
  tools: {
    sandbox: {
      tools: {
        // Wenn allow nicht leer ist, wird alles andere blockiert (deny hat trotzdem Vorrang).
        allow: ["group:messaging", "group:sessions"],
        deny: ["group:runtime", "group:fs", "group:ui", "nodes", "cron", "gateway"]
      }
    }
  }
}
```

Du möchtest „Gruppen können nur Ordner X sehen“ statt „kein Hostzugriff“? Belasse `workspaceAccess: "none"` und mounte nur Allowlist-Pfade in die sandbox:

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main",
        scope: "session",
        workspaceAccess: "none",
        docker: {
          binds: [
            // hostPfad:containerPfad:modus
            "~/FriendsShared:/data:ro"
          ]
        }
      }
    }
  }
}
```

Verwandte Themen:

* Konfigurationsschlüssel und Standardwerte: [Gateway-Konfiguration](/de/gateway/configuration#agentsdefaultssandbox)
* Debugging, warum ein Tool blockiert wird: [Sandbox vs Tool Policy vs Elevated](/de/gateway/sandbox-vs-tool-policy-vs-elevated)
* Details zu Bind-Mounts: [Sandboxing](/de/gateway/sandboxing#custom-bind-mounts)

<div id="display-labels">
  ## Anzeigelabels
</div>

* UI-Labels verwenden `displayName`, falls verfügbar, formatiert als `<channel>:<token>`.
* `#room` ist für Räume/Kanäle reserviert; Gruppenchats verwenden `g-<slug>` (Kleinbuchstaben, Leerzeichen -&gt; `-`, `#@+._-` beibehalten).

<div id="group-policy">
  ## Gruppenrichtlinie
</div>

Lege fest, wie Gruppen- und Raumnachrichten pro Kanal verarbeitet werden:

```json5
{
  channels: {
    whatsapp: {
      groupPolicy: "disabled", // "open" | "disabled" | "allowlist"
      groupAllowFrom: ["+15551234567"]
    },
    telegram: {
      groupPolicy: "disabled",
      groupAllowFrom: ["123456789", "@username"]
    },
    signal: {
      groupPolicy: "disabled",
      groupAllowFrom: ["+15551234567"]
    },
    imessage: {
      groupPolicy: "disabled",
      groupAllowFrom: ["chat_id:123"]
    },
    msteams: {
      groupPolicy: "disabled",
      groupAllowFrom: ["user@org.com"]
    },
    discord: {
      groupPolicy: "allowlist",
      guilds: {
        "GUILD_ID": { channels: { help: { allow: true } } }
      }
    },
    slack: {
      groupPolicy: "allowlist",
      channels: { "#general": { allow: true } }
    },
    matrix: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["@owner:example.org"],
      groups: {
        "!roomId:example.org": { allow: true },
        "#alias:example.org": { allow: true }
      }
    }
  }
}
```

| Policy        | Verhalten                                                                        |
| ------------- | -------------------------------------------------------------------------------- |
| `"open"`      | Gruppen umgehen die Allowlist; Mention-Gating gilt weiterhin.                    |
| `"disabled"`  | Alle Gruppennachrichten vollständig blockieren.                                  |
| `"allowlist"` | Nur Gruppen/Räume zulassen, die mit der konfigurierten Allowlist übereinstimmen. |

Hinweise:

* `groupPolicy` ist getrennt von Mention-Gating (das @mentions erfordert).
* WhatsApp/Telegram/Signal/iMessage/Microsoft Teams: verwende `groupAllowFrom` (Fallback: explizites `allowFrom`).
* Discord: Allowlist verwendet `channels.discord.guilds.<id>.channels`.
* Slack: Allowlist verwendet `channels.slack.channels`.
* Matrix: Allowlist verwendet `channels.matrix.groups` (Raum-IDs, Aliasse oder Namen). Verwende `channels.matrix.groupAllowFrom`, um Absender einzuschränken; per-Raum-`users`-Allowlists werden ebenfalls unterstützt.
* Gruppen-DMs werden separat gesteuert (`channels.discord.dm.*`, `channels.slack.dm.*`).
* Die Telegram-Allowlist kann mit Benutzer-IDs (`"123456789"`, `"telegram:123456789"`, `"tg:123456789"`) oder Benutzernamen (`"@alice"` oder `"alice"`) übereinstimmen; Präfixe sind nicht groß-/kleinschreibungssensitiv.
* Standard ist `groupPolicy: "allowlist"`; wenn deine Gruppen-Allowlist leer ist, werden Gruppennachrichten blockiert.

Schnelles mentales Modell (Auswertungsreihenfolge für Gruppennachrichten):

1. `groupPolicy` (open/disabled/allowlist)
2. Gruppen-Allowlists (`*.groups`, `*.groupAllowFrom`, kanalspezifische Allowlist)
3. Mention-Gating (`requireMention`, `/activation`)

<div id="mention-gating-default">
  ## Mention-Gating (Standard)
</div>

Gruppennachrichten setzen eine @-Mention voraus, sofern dies nicht pro Gruppe überschrieben wird. Die Standardwerte liegen pro Subsystem unter `*.groups."*"`.

Eine Antwort auf eine Bot-Nachricht zählt als implizite @-Mention (wenn der Kanal Reply-Metadaten unterstützt). Dies gilt für Telegram, WhatsApp, Slack, Discord und Microsoft Teams.

```json5
{
  channels: {
    whatsapp: {
      groups: {
        "*": { requireMention: true },
        "123@g.us": { requireMention: false }
      }
    },
    telegram: {
      groups: {
        "*": { requireMention: true },
        "123456789": { requireMention: false }
      }
    },
    imessage: {
      groups: {
        "*": { requireMention: true },
        "123": { requireMention: false }
      }
    }
  },
  agents: {
    list: [
      {
        id: "main",
        groupChat: {
          mentionPatterns: ["@openclaw", "openclaw", "\\+15555550123"],
          historyLimit: 50
        }
      }
    ]
  }
}
```

Hinweise:

* `mentionPatterns` sind groß-/kleinschreibungsunabhängige reguläre Ausdrücke (Regex).
* Oberflächen/Clients, die explizite Erwähnungen bereitstellen, werden weiterhin akzeptiert; Patterns dienen nur als Fallback.
* Override pro agent: `agents.list[].groupChat.mentionPatterns` (nützlich, wenn sich mehrere Agenten einen Gruppenchat teilen).
* Mention-Gating wird nur erzwungen, wenn Mention-Erkennung möglich ist (native Mentions oder konfigurierte `mentionPatterns`).
* Discord-Standardwerte befinden sich in `channels.discord.guilds."*"` (pro Guild/Channel überschreibbar).
* Der Kontext des Gruppenverlaufs wird kanalübergreifend einheitlich gekapselt und ist **pending-only** (Nachrichten, die aufgrund von Mention-Gating übersprungen wurden); `messages.groupChat.historyLimit` für den globalen Standard verwenden und `channels.<channel>.historyLimit` (oder `channels.<channel>.accounts.*.historyLimit`) für Overrides. `0` setzen, um zu deaktivieren.

<div id="groupchannel-tool-restrictions-optional">
  ## Gruppen-/Channel-Tool-Einschränkungen (optional)
</div>

Einige Channel-Konfigurationen unterstützen die Einschränkung, welche Tools **innerhalb einer bestimmten Gruppe, eines bestimmten Raums oder Channels** verfügbar sind.

* `tools`: Tools für die gesamte Gruppe erlauben oder verbieten.
* `toolsBySender`: Absenderbezogene Overrides innerhalb der Gruppe (Schlüssel sind Absender-IDs/Benutzernamen/E-Mails/Telefonnummern, abhängig vom Channel). Verwende `"*"` als Platzhalter.

Reihenfolge der Auswertung (die spezifischste Regel gewinnt):

1. Gruppen-/Channel-`toolsBySender`-Treffer
2. Gruppen-/Channel-`tools`
3. Standard-`toolsBySender`-Treffer für (`"*"`)
4. Standard-`tools` für (`"*"`)

Beispiel (Telegram):

```json5
{
  channels: {
    telegram: {
      groups: {
        "*": { tools: { deny: ["exec"] } },
        "-1001234567890": {
          tools: { deny: ["exec", "read", "write"] },
          toolsBySender: {
            "123456789": { alsoAllow: ["exec"] }
          }
        }
      }
    }
  }
}
```

Hinweise:

* Gruppen-/Channel-Tool-Einschränkungen werden zusätzlich zur globalen/Agent-Tool-Policy angewendet (deny hat weiterhin Vorrang).
* Einige Channels verwenden eine andere Verschachtelung für Räume/Channels (z. B. Discord `guilds.*.channels.*`, Slack `channels.*`, MS Teams `teams.*.channels.*`).

<div id="group-allowlists">
  ## Gruppen-Allowlists
</div>

Wenn `channels.whatsapp.groups`, `channels.telegram.groups` oder `channels.imessage.groups` konfiguriert ist, fungieren die Keys als Gruppen-Allowlist. Verwende `"*"`, um alle Gruppen zuzulassen, während du trotzdem das Standard-Erwähnungsverhalten festlegst.

Häufige Intents (Copy/paste):

1. Alle Gruppenantworten deaktivieren

```json5
{
  channels: { whatsapp: { groupPolicy: "disabled" } }
}
```

2. Nur bestimmte Gruppen erlauben (WhatsApp)

```json5
{
  channels: {
    whatsapp: {
      groups: {
        "123@g.us": { requireMention: true },
        "456@g.us": { requireMention: false }
      }
    }
  }
}
```

3. Alle Gruppen zulassen, aber Erwähnung erforderlich (explizit)

```json5
{
  channels: {
    whatsapp: {
      groups: { "*": { requireMention: true } }
    }
  }
}
```

4. Nur der Besitzer kann die Aktivierung in Gruppen starten (WhatsApp)

```json5
{
  channels: {
    whatsapp: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15551234567"],
      groups: { "*": { requireMention: true } }
    }
  }
}
```

<div id="activation-owner-only">
  ## Aktivierung (nur Besitzer)
</div>

Gruppenbesitzer können die Aktivierung pro Gruppe umschalten:

* `/activation mention`
* `/activation always`

Der Besitzer wird durch `channels.whatsapp.allowFrom` bestimmt (oder die eigene E.164 des Bots, falls nicht gesetzt). Senden Sie den Befehl als separate Nachricht. Andere Oberflächen ignorieren `/activation` derzeit.

<div id="context-fields">
  ## Kontextfelder
</div>

Bei eingehenden Gruppen-Payloads werden gesetzt:

* `ChatType=group`
* `GroupSubject` (falls bekannt)
* `GroupMembers` (falls bekannt)
* `WasMentioned` (Ergebnis der Mention-Gating-Logik)
* Telegram-Forum-Threads enthalten außerdem `MessageThreadId` und `IsForum`.

Der Agent-Systemprompt enthält im ersten Turn einer neuen Gruppen-Sitzung eine Gruppen-Einführung. Er erinnert das Modell daran, wie ein Mensch zu antworten, Markdown-Tabellen zu vermeiden und keine wörtlichen `\n`-Sequenzen zu tippen.

<div id="imessage-specifics">
  ## iMessage-Spezifika
</div>

* Verwende nach Möglichkeit `chat_id:<id>` beim Routing oder Allowlisting.
* Chats auflisten: `imsg chats --limit 20`.
* Gruppenantworten werden immer an dieselbe `chat_id` zurückgesendet.

<div id="whatsapp-specifics">
  ## WhatsApp-spezifisches
</div>

Siehe [Gruppennachrichten](/de/concepts/group-messages) für WhatsApp-spezifisches Verhalten (Verlaufseinbettung, Details zur Verarbeitung von Erwähnungen).