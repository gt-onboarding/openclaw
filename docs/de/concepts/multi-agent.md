---
summary: "Multi-Agent-Routing: isolierte Agenten, Kanal-Konten und Bindings"
title: Multi-Agent-Routing
read_when: "Wenn du mehrere isolierte Agenten (arbeitsbereiche + Authentifizierung) in einem Gateway-Prozess betreiben möchtest."
status: active
---

<div id="multi-agent-routing">
  # Multi-Agent-Routing
</div>

Ziel: mehrere *isolierte* Agenten (jeweils eigener Arbeitsbereich + `agentDir` + Sitzungen) sowie mehrere Kanal-Konten (z. B. zwei WhatsApp-Accounts) in einem laufenden Gateway. Eingehende Nachrichten werden über Bindings an einen Agenten weitergeleitet.

<div id="what-is-one-agent">
  ## Was ist „ein Agent“?
</div>

Ein **Agent** ist ein vollständig eigenständiges „Gehirn“ mit einem eigenen:

* **Arbeitsbereich** (Dateien, AGENTS.md/SOUL.md/USER.md, lokale Notizen, Persona-Regeln).
* **Zustandsverzeichnis** (`agentDir`) für Auth-Profile, Modell-Registry und agent-spezifische Konfiguration.
* **Sitzungsspeicher** (Chat-Verlauf + Routing-Status) unter `~/.openclaw/agents/<agentId>/sessions`.

Auth-Profile sind **agent-spezifisch**. Jeder Agent liest aus seinem eigenen:

```
~/.openclaw/agents/<agentId>/agent/auth-profiles.json
```

Die Haupt-Anmeldeinformationen eines Agents werden **nicht** automatisch gemeinsam genutzt. Verwende `agentDir` niemals für mehrere Agenten (das führt zu Kollisionen bei Authentifizierung/Sitzungen). Wenn du Anmeldeinformationen teilen möchtest, kopiere `auth-profiles.json` in das `agentDir` des anderen Agents.

Fähigkeiten sind pro Agent über den `skills/`-Ordner in jedem Arbeitsbereich definiert, mit gemeinsam genutzten Fähigkeiten unter `~/.openclaw/skills`. Siehe [Skills: per-agent vs shared](/de/tools/skills#per-agent-vs-shared-skills).

Das Gateway kann **einen Agenten** (Standard) oder **viele Agenten** nebeneinander hosten.

**Hinweis zum Arbeitsbereich:** Der Arbeitsbereich jedes Agents ist das **Standard-cwd**, keine harte sandbox. Relative Pfade werden innerhalb des Arbeitsbereichs aufgelöst, aber absolute Pfade können andere Speicherorte des Hosts erreichen, es sei denn, Sandboxing ist aktiviert. Siehe
[Sandboxing](/de/gateway/sandboxing).

<div id="paths-quick-map">
  ## Pfade (Schnellübersicht)
</div>

* Config: `~/.openclaw/openclaw.json` (oder `OPENCLAW_CONFIG_PATH`)
* State-Verzeichnis: `~/.openclaw` (oder `OPENCLAW_STATE_DIR`)
* Arbeitsbereich: `~/.openclaw/workspace` (oder `~/.openclaw/workspace-<agentId>`)
* Agent-Verzeichnis: `~/.openclaw/agents/<agentId>/agent` (oder `agents.list[].agentDir`)
* Sitzungen: `~/.openclaw/agents/<agentId>/sessions`

<div id="single-agent-mode-default">
  ### Einzelagenten-Modus (Standard)
</div>

Wenn du nichts weiter konfigurierst, führt OpenClaw einen einzelnen Agent aus:

* `agentId` ist standardmäßig **`main`**.
* Sitzungen werden mit `agent:main:&lt;mainKey&gt;` gekennzeichnet.
* Der Arbeitsbereich ist standardmäßig `~/.openclaw/workspace` (oder `~/.openclaw/workspace-<profile>`, wenn `OPENCLAW_PROFILE` gesetzt ist).
* Der Status wird standardmäßig unter `~/.openclaw/agents/main/agent` gespeichert.

<div id="agent-helper">
  ## Agent-Helfer
</div>

Verwende den Agent-Assistenten, um einen neuen, isolierten Agent hinzuzufügen:

```bash
openclaw agents add work
```

Füge anschließend `bindings` hinzu (oder lass den Assistenten das übernehmen), um eingehende Nachrichten zu routen.

Überprüfe dies mit:

```bash
openclaw agents list --bindings
```

<div id="multiple-agents-multiple-people-multiple-personalities">
  ## Mehrere Agenten = mehrere Personen, mehrere Persönlichkeiten
</div>

Mit **mehreren Agenten** wird jede `agentId` zu einer **vollständig isolierten Persona**:

* **Unterschiedliche Telefonnummern/Konten** (pro Kanal-`accountId`).
* **Unterschiedliche Persönlichkeiten** (pro Agent eigener Arbeitsbereich mit Dateien wie `AGENTS.md` und `SOUL.md`).
* **Getrennte Authentifizierung und Sitzungen** (keine Übersprechung, außer wenn ausdrücklich aktiviert).

So können sich **mehrere Personen** einen Gateway-Server teilen, während ihre KI-„Gehirne“ und Daten isoliert bleiben.

<div id="one-whatsapp-number-multiple-people-dm-split">
  ## Eine WhatsApp-Nummer, mehrere Personen (DM-Split)
</div>

Du kannst **verschiedene WhatsApp-DMs** an verschiedene Agenten routen, während du bei **einem WhatsApp-Konto** bleibst. Ordne dazu anhand der Absendernummer im E.164-Format (z. B. `+15551234567`) mit `peer.kind: "dm"` zu. Antworten kommen weiterhin von derselben WhatsApp-Nummer (keine Agent-spezifische Absenderidentität).

Wichtiger Hinweis: Direkte Chats werden auf den **Hauptsitzungsschlüssel** des Agents zusammengeführt, daher erfordert echte Isolation **einen Agenten pro Person**.

Beispiel:

```json5
{
  agents: {
    list: [
      { id: "alex", workspace: "~/.openclaw/workspace-alex" },
      { id: "mia", workspace: "~/.openclaw/workspace-mia" }
    ]
  },
  bindings: [
    { agentId: "alex", match: { channel: "whatsapp", peer: { kind: "dm", id: "+15551230001" } } },
    { agentId: "mia",  match: { channel: "whatsapp", peer: { kind: "dm", id: "+15551230002" } } }
  ],
  channels: {
    whatsapp: {
      dmPolicy: "allowlist",
      allowFrom: ["+15551230001", "+15551230002"]
    }
  }
}
```

Hinweise:

* Die DM-Zugriffskontrolle ist **global pro WhatsApp-Konto** (kopplung/Allowlist), nicht pro agent.
* Für gemeinsame Gruppen binde die Gruppe an einen agent oder verwende [Broadcast-Gruppen](/de/broadcast-groups).

<div id="routing-rules-how-messages-pick-an-agent">
  ## Routing-Regeln (wie Nachrichten einen agent auswählen)
</div>

Bindings sind **deterministisch** und **die spezifischste Regel hat Vorrang**:

1. `peer`-Übereinstimmung (exakte DM-/Gruppen-/Channel-ID)
2. `guildId` (Discord)
3. `teamId` (Slack)
4. `accountId`-Übereinstimmung für einen Channel
5. Übereinstimmung auf Channel-Ebene (`accountId: "*"`)
6. Fallback auf den Standard-agent (`agents.list[].default`, sonst erster Eintrag in der Liste, Standard: `main`)

<div id="multiple-accounts-phone-numbers">
  ## Mehrere Accounts / Telefonnummern
</div>

Kanäle, die **mehrere Accounts** unterstützen (z. B. WhatsApp), verwenden `accountId`, um
jede Anmeldung zu identifizieren. Jede `accountId` kann an einen anderen agent geroutet werden, sodass ein Server
mehrere Telefonnummern hosten kann, ohne Sitzungen zu vermischen.

<div id="concepts">
  ## Konzepte
</div>

* `agentId`: ein „Gehirn“ (Arbeitsbereich, pro-Agent-Authentifizierung, pro-Agent-Sitzungsspeicher).
* `accountId`: eine Channel-Konto-Instanz (z. B. WhatsApp-Konto `"personal"` vs. `"biz"`).
* `binding`: leitet eingehende Nachrichten anhand von `(channel, accountId, peer)` und optionalen Guild-/Team-IDs an eine `agentId` weiter.
* Direkte Chats werden auf `agent:<agentId>:<mainKey>` abgebildet (pro Agent ein „main“; `session.mainKey`).

<div id="example-two-whatsapps-two-agents">
  ## Beispiel: zwei WhatsApps → zwei Agenten
</div>

`~/.openclaw/openclaw.json` (JSON5):

```js
{
  agents: {
    list: [
      {
        id: "home",
        default: true,
        name: "Home",
        workspace: "~/.openclaw/workspace-home",
        agentDir: "~/.openclaw/agents/home/agent",
      },
      {
        id: "work",
        name: "Work",
        workspace: "~/.openclaw/workspace-work",
        agentDir: "~/.openclaw/agents/work/agent",
      },
    ],
  },

  // Deterministic routing: first match wins (most-specific first).
  bindings: [
    { agentId: "home", match: { channel: "whatsapp", accountId: "personal" } },
    { agentId: "work", match: { channel: "whatsapp", accountId: "biz" } },

    // Optional per-peer override (example: send a specific group to work agent).
    {
      agentId: "work",
      match: {
        channel: "whatsapp",
        accountId: "personal",
        peer: { kind: "group", id: "1203630...@g.us" },
      },
    },
  ],

  // Standardmäßig deaktiviert: Agent-zu-Agent-Messaging muss explizit aktiviert und auf die Allowlist gesetzt werden.
  tools: {
    agentToAgent: {
      enabled: false,
      allow: ["home", "work"],
    },
  },

  channels: {
    whatsapp: {
      accounts: {
        personal: {
          // Optional override. Default: ~/.openclaw/credentials/whatsapp/personal
          // authDir: "~/.openclaw/credentials/whatsapp/personal",
        },
        biz: {
          // Optional override. Default: ~/.openclaw/credentials/whatsapp/biz
          // authDir: "~/.openclaw/credentials/whatsapp/biz",
        },
      },
    },
  },
}
```

<div id="example-whatsapp-daily-chat-telegram-deep-work">
  ## Beispiel: WhatsApp-Alltagschat + Telegram-Deep-Work
</div>

Aufteilung nach Kanal: Leite WhatsApp an einen schnellen Alltags-agent und Telegram an einen Opus-agent weiter.

```json5
{
  agents: {
    list: [
      {
        id: "chat",
        name: "Everyday",
        workspace: "~/.openclaw/workspace-chat",
        model: "anthropic/claude-sonnet-4-5"
      },
      {
        id: "opus",
        name: "Deep Work",
        workspace: "~/.openclaw/workspace-opus",
        model: "anthropic/claude-opus-4-5"
      }
    ]
  },
  bindings: [
    { agentId: "chat", match: { channel: "whatsapp" } },
    { agentId: "opus", match: { channel: "telegram" } }
  ]
}
```

Hinweise:

* Wenn du mehrere Konten für einen Channel hast, füge `accountId` zur Bindung hinzu (zum Beispiel `{ channel: "whatsapp", accountId: "personal" }`).
* Um eine einzelne DM oder Gruppe zu Opus zu routen, während der Rest im Chat bleibt, füge eine `match.peer`-Bindung für diesen Peer hinzu; Peer-Matches haben immer Vorrang vor channelweiten Regeln.

<div id="example-same-channel-one-peer-to-opus">
  ## Beispiel: gleicher Kanal, ein Peer zu Opus
</div>

Belasse WhatsApp beim schnellen agent, leite aber eine Direktnachricht an Opus weiter:

```json5
{
  agents: {
    list: [
      { id: "chat", name: "Everyday", workspace: "~/.openclaw/workspace-chat", model: "anthropic/claude-sonnet-4-5" },
      { id: "opus", name: "Deep Work", workspace: "~/.openclaw/workspace-opus", model: "anthropic/claude-opus-4-5" }
    ]
  },
  bindings: [
    { agentId: "opus", match: { channel: "whatsapp", peer: { kind: "dm", id: "+15551234567" } } },
    { agentId: "chat", match: { channel: "whatsapp" } }
  ]
}
```

Peer-Bindings haben immer Vorrang, platziere sie daher oberhalb der kanalweiten Regel.

<div id="family-agent-bound-to-a-whatsapp-group">
  ## Familien-Agent an eine WhatsApp-Gruppe gebunden
</div>

Einen dedizierten Familien-Agent mit Mention-Gating und einer strengeren Tool-Policy an eine einzelne WhatsApp-Gruppe binden:

```json5
{
  agents: {
    list: [
      {
        id: "family",
        name: "Family",
        workspace: "~/.openclaw/workspace-family",
        identity: { name: "Family Bot" },
        groupChat: {
          mentionPatterns: ["@family", "@familybot", "@Family Bot"]
        },
        sandbox: {
          mode: "all",
          scope: "agent"
        },
        tools: {
          allow: ["exec", "read", "sessions_list", "sessions_history", "sessions_send", "sessions_spawn", "session_status"],
          deny: ["write", "edit", "apply_patch", "browser", "canvas", "nodes", "cron"]
        }
      }
    ]
  },
  bindings: [
    {
      agentId: "family",
      match: {
        channel: "whatsapp",
        peer: { kind: "group", id: "120363999999999999@g.us" }
      }
    }
  ]
}
```

Hinweise:

* Allow-/Deny-Listen für Tools sind **Tools**, keine Fähigkeiten. Wenn eine Fähigkeit eine
  Binärdatei ausführen muss, stelle sicher, dass `exec` erlaubt ist und die Binärdatei in der Sandbox vorhanden ist.
* Für strengere Zugriffskontrolle setze `agents.list[].groupChat.mentionPatterns` und lass die Gruppen-Allowlists für den Kanal aktiviert.

<div id="per-agent-sandbox-and-tool-configuration">
  ## Sandbox- und Tool-Konfiguration pro Agent
</div>

Seit v2026.1.6 kann jeder Agent seine eigene Sandbox und eigene Tool-Einschränkungen haben:

```js
{
  agents: {
    list: [
      {
        id: "personal",
        workspace: "~/.openclaw/workspace-personal",
        sandbox: {
          mode: "off",  // No sandbox for personal agent
        },
        // No tool restrictions - all tools available
      },
      {
        id: "family",
        workspace: "~/.openclaw/workspace-family",
        sandbox: {
          mode: "all",     // Immer in Sandbox
          scope: "agent",  // Ein Container pro Agent
          docker: {
            // Optionale einmalige Einrichtung nach Container-Erstellung
            setupCommand: "apt-get update && apt-get install -y git curl",
          },
        },
        tools: {
          allow: ["read"],                    // Only read tool
          deny: ["exec", "write", "edit", "apply_patch"],    // Deny others
        },
      },
    ],
  },
}
```

Hinweis: `setupCommand` befindet sich unter `sandbox.docker` und wird einmalig bei der Erstellung des Containers ausgeführt.
Pro-Agent-Overrides von `sandbox.docker.*` werden ignoriert, wenn die ermittelte Scope `"shared"` ist.

**Vorteile:**

* **Sicherheitsisolation**: Werkzeuge für nicht vertrauenswürdige Agenten einschränken
* **Ressourcenkontrolle**: Bestimmte Agenten in der sandbox ausführen, während andere auf dem Host laufen
* **Flexible Richtlinien**: Unterschiedliche Berechtigungen pro Agent

Hinweis: `tools.elevated` ist **global** und senderspezifisch; es ist nicht pro Agent konfigurierbar.
Wenn Sie Grenzen pro Agent benötigen, verwenden Sie `agents.list[].tools`, um `exec` zu verweigern.
Für gruppenbasierte Zieladressierung verwenden Sie `agents.list[].groupChat.mentionPatterns`, damit @mentions eindeutig dem vorgesehenen Agent zugeordnet werden.

Siehe [Multi-Agent-Sandbox &amp; Tools](/de/multi-agent-sandbox-tools) für ausführliche Beispiele.
