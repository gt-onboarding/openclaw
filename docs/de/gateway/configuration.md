---
title: Konfiguration
summary: "Alle Konfigurationsoptionen der Datei ~/.openclaw/openclaw.json mit Beispielen"
read_when:
  - Beim Hinzufügen oder Ändern von Konfigurationsfeldern
---

<div id="configuration">
  # Konfiguration 🔧
</div>

OpenClaw liest eine optionale **JSON5**-Konfiguration aus `~/.openclaw/openclaw.json` (Kommentare und nachgestellte Kommata sind erlaubt).

Falls die Datei fehlt, verwendet OpenClaw relativ sichere Standardwerte (eingebetteter Pi-Agent + Sitzungen pro Absender + Arbeitsbereich `~/.openclaw/workspace`). In der Regel benötigst du eine Konfiguration nur, um:

* einzuschränken, wer den Bot triggern kann (`channels.whatsapp.allowFrom`, `channels.telegram.allowFrom`, etc.)
* Gruppen-Allowlists und Mention-Verhalten zu steuern (`channels.whatsapp.groups`, `channels.telegram.groups`, `channels.discord.guilds`, `agents.list[].groupChat`)
* Nachrichtenpräfixe anzupassen (`messages`)
* den Arbeitsbereich des Agents festzulegen (`agents.defaults.workspace` oder `agents.list[].workspace`)
* die Standardwerte des eingebetteten Agents (`agents.defaults`) und das Sitzungsverhalten (`session`) zu justieren
* die Identität pro Agent festzulegen (`agents.list[].identity`)

> **Neu bei der Konfiguration?** Sieh dir den Leitfaden [Konfigurationsbeispiele](/de/gateway/configuration-examples) für vollständige Beispiele mit ausführlichen Erklärungen an!

<div id="strict-config-validation">
  ## Strikte Validierung der Konfiguration
</div>

OpenClaw akzeptiert nur Konfigurationen, die vollständig dem Schema entsprechen.
Unbekannte Schlüssel, fehlerhafte Typen oder ungültige Werte führen dazu, dass das Gateway aus Sicherheitsgründen den Start **verweigert**.

Wenn die Validierung fehlschlägt:

* Das Gateway startet nicht.
* Es sind nur Diagnosebefehle erlaubt (zum Beispiel: `openclaw doctor`, `openclaw logs`, `openclaw health`, `openclaw status`, `openclaw service`, `openclaw help`).
* Führe `openclaw doctor` aus, um die genauen Probleme zu sehen.
* Führe `openclaw doctor --fix` (oder `--yes`) aus, um Migrationen/Reparaturen anzuwenden.

`openclaw doctor` nimmt niemals Änderungen vor, es sei denn, du stimmst mit `--fix`/`--yes` ausdrücklich zu.

<div id="schema-ui-hints">
  ## Schema + UI-Hinweise
</div>

Das Gateway stellt über `config.schema` eine JSON-Schema-Repräsentation der Konfiguration für UI-Editoren bereit.
Die Control UI rendert aus diesem Schema ein Formular, mit einem **Raw JSON**-Editor als Ausweichmöglichkeit.

Channel-Plugins und -Erweiterungen können Schema- und UI-Hinweise für ihre Konfiguration registrieren, sodass
Channel-Einstellungen über Apps hinweg schemagesteuert bleiben, ohne hartkodierte Formulare.

Hinweise (Labels, Gruppierung, sensible Felder) werden zusammen mit dem Schema ausgeliefert, damit Clients
bessere Formulare rendern können, ohne Konfigurationswissen hartzukodieren.

<div id="apply-restart-rpc">
  ## Anwenden + Neustart (RPC)
</div>

Verwende `config.apply`, um die vollständige Konfiguration in einem Schritt zu validieren, zu schreiben und das Gateway neu zu starten.
Es schreibt ein Neustart-Sentinel und sendet einen Ping an die letzte aktive Sitzung, nachdem das Gateway wieder läuft.

Warnung: `config.apply` ersetzt die **gesamte Konfiguration**. Wenn du nur einige wenige Schlüssel ändern willst,
verwende `config.patch` oder `openclaw config set`. Halte ein Backup von `~/.openclaw/openclaw.json` bereit.

Parameter:

* `raw` (string) — JSON5-Payload für die gesamte Konfiguration
* `baseHash` (optional) — Konfigurations-Hash von `config.get` (erforderlich, wenn bereits eine Konfiguration existiert)
* `sessionKey` (optional) — Schlüssel der letzten aktiven Sitzung für den Wake-up-Ping
* `note` (optional) — Notiz, die im Neustart-Sentinel enthalten sein soll
* `restartDelayMs` (optional) — Verzögerung vor dem Neustart (Standardwert 2000)

Beispiel (über `gateway call`):

```bash
openclaw gateway call config.get --params '{}' # payload.hash erfassen
openclaw gateway call config.apply --params '{
  "raw": "{\\n  agents: { defaults: { workspace: \\"~/.openclaw/workspace\\" } }\\n}\\n",
  "baseHash": "<hash-from-config.get>",
  "sessionKey": "agent:main:whatsapp:dm:+15555550123",
  "restartDelayMs": 1000
}'
```

<div id="partial-updates-rpc">
  ## Partielle Aktualisierungen (RPC)
</div>

Verwende `config.patch`, um eine partielle Aktualisierung in die bestehende Konfiguration einzufügen, ohne
unabhängige Schlüssel zu überschreiben. Es wendet JSON-Merge-Patch-Semantik an:

* Objekte werden rekursiv zusammengeführt
* `null` löscht einen Schlüssel
* Arrays werden ersetzt

Wie `config.apply` validiert es die Konfiguration, schreibt sie, speichert ein Neustart-Sentinel und plant
den Gateway-Neustart (mit optionalem Wecken, wenn `sessionKey` angegeben ist).

Parameter:

* `raw` (string) — JSON5-Payload, die nur die zu ändernden Schlüssel enthält
* `baseHash` (required) — Konfigurations-Hash von `config.get`
* `sessionKey` (optional) — letzter aktiver Session-Key für das Weck-Ping
* `note` (optional) — Notiz, die im Neustart-Sentinel gespeichert wird
* `restartDelayMs` (optional) — Verzögerung vor dem Neustart (Standard: 2000)

Beispiel:

```bash
openclaw gateway call config.get --params '{}' # payload.hash erfassen
openclaw gateway call config.patch --params '{
  "raw": "{\\n  channels: { telegram: { groups: { \\"*\\": { requireMention: false } } } }\\n}\\n",
  "baseHash": "<hash-from-config.get>",
  "sessionKey": "agent:main:whatsapp:dm:+15555550123",
  "restartDelayMs": 1000
}'
```

<div id="minimal-config-recommended-starting-point">
  ## Minimale Konfiguration (empfohlener Ausgangspunkt)
</div>

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
  channels: { whatsapp: { allowFrom: ["+15555550123"] } }
}
```

Erstelle das Standard-Image einmalig mit:

```bash
scripts/sandbox-setup.sh
```

<div id="self-chat-mode-recommended-for-group-control">
  ## Self-Chat-Modus (empfohlen für die Gruppensteuerung)
</div>

Um zu verhindern, dass der Bot in WhatsApp-Gruppen auf @-Erwähnungen reagiert (stattdessen nur auf bestimmte Text-Trigger antwortet):

```json5
{
  agents: {
    defaults: { workspace: "~/.openclaw/workspace" },
    list: [
      {
        id: "main",
        groupChat: { mentionPatterns: ["@openclaw", "reisponde"] }
      }
    ]
  },
  channels: {
    whatsapp: {
      // Allowlist gilt nur für Direktnachrichten; die Aufnahme Ihrer eigenen Nummer aktiviert den Self-Chat-Modus.
      allowFrom: ["+15555550123"],
      groups: { "*": { requireMention: true } }
    }
  }
}
```

<div id="config-includes-include">
  ## Config Includes (`$include`)
</div>

Teile deine Konfiguration in mehrere Dateien auf, indem du die Direktive `$include` verwendest. Dies ist nützlich für:

* Strukturieren großer Konfigurationen (z. B. Agent-Definitionen pro Mandant)
* Wiederverwendung gemeinsamer Einstellungen in mehreren Umgebungen
* Getrennte Verwaltung sensibler Konfigurationen

<div id="basic-usage">
  ### Grundlegende Nutzung
</div>

```json5
// ~/.openclaw/openclaw.json
{
  gateway: { port: 18789 },
  
  // Eine einzelne Datei einbinden (ersetzt den Wert des Schlüssels)
  agents: { "$include": "./agents.json5" },
  
  // Include multiple files (deep-merged in order)
  broadcast: { 
    "$include": [
      "./clients/mueller.json5",
      "./clients/schmidt.json5"
    ]
  }
}
```

```json5
// ~/.openclaw/agents.json5
{
  defaults: { sandbox: { mode: "all", scope: "session" } },
  list: [
    { id: "main", workspace: "~/.openclaw/workspace" }
  ]
}
```

<div id="merge-behavior">
  ### Merge-Verhalten
</div>

* **Einzelne Datei**: Ersetzt das Objekt, das `$include` enthält
* **Array von Dateien**: Führt die Dateien der Reihe nach per Deep-Merge zusammen (spätere Dateien überschreiben frühere)
* **Mit Geschwister-Schlüsseln**: Geschwister-Schlüssel werden nach den Includes zusammengeführt (überschreiben inkludierte Werte)
* **Geschwister-Schlüssel + Arrays/Primitive**: Nicht unterstützt (der inkludierte Inhalt muss ein Objekt sein)

```json5
// Schlüssel auf gleicher Ebene überschreiben eingebundene Werte
{
  "$include": "./base.json5",   // { a: 1, b: 2 }
  b: 99                          // Ergebnis: { a: 1, b: 99 }
}
```

<div id="nested-includes">
  ### Verschachtelte Includes
</div>

Eingebundene Dateien können ihrerseits `$include`-Direktiven enthalten (bis zu 10 Ebenen tief):

```json5
// clients/mueller.json5
{
  agents: { "$include": "./mueller/agents.json5" },
  broadcast: { "$include": "./mueller/broadcast.json5" }
}
```

<div id="path-resolution">
  ### Pfadauflösung
</div>

* **Relative Pfade**: Relativ zur einbindenden Datei aufgelöst
* **Absolute Pfade**: Unverändert verwendet
* **Übergeordnete Verzeichnisse**: `../`-Verweise funktionieren wie erwartet

```json5
{ "$include": "./sub/config.json5" }      // relative
{ "$include": "/etc/openclaw/base.json5" } // absolute
{ "$include": "../shared/common.json5" }   // übergeordnetes Verzeichnis
```

<div id="error-handling">
  ### Fehlerbehandlung
</div>

* **Fehlende Datei**: Klare Fehlermeldung mit aufgelöstem Pfad
* **Parsing-Fehler**: Zeigt an, in welcher eingebundenen Datei der Fehler aufgetreten ist
* **Zirkuläre Includes**: Werden erkannt und zusammen mit der Include-Kette gemeldet

<div id="example-multi-client-legal-setup">
  ### Beispiel: Rechtliche Konfiguration für mehrere Mandanten
</div>

```json5
// ~/.openclaw/openclaw.json
{
  gateway: { port: 18789, auth: { token: "secret" } },
  
  // Common agent defaults
  agents: {
    defaults: {
      sandbox: { mode: "all", scope: "session" }
    },
    // Agent-Listen aller Mandanten zusammenführen
    list: { "$include": [
      "./clients/mueller/agents.json5",
      "./clients/schmidt/agents.json5"
    ]}
  },
  
  // Merge broadcast configs
  broadcast: { "$include": [
    "./clients/mueller/broadcast.json5",
    "./clients/schmidt/broadcast.json5"
  ]},
  
  channels: { whatsapp: { groupPolicy: "allowlist" } }
}
```

```json5
// ~/.openclaw/clients/mueller/agents.json5
[
  { id: "mueller-transcribe", workspace: "~/clients/mueller/transcribe" },
  { id: "mueller-docs", workspace: "~/clients/mueller/docs" }
]
```

```json5
// ~/.openclaw/clients/mueller/broadcast.json5
{
  "120363403215116621@g.us": ["mueller-transcribe", "mueller-docs"]
}
```

<div id="common-options">
  ## Allgemeine Optionen
</div>

<div id="env-vars-env">
  ### Env Vars + `.env`
</div>

OpenClaw liest Env Vars aus dem Elternprozess (Shell, launchd/systemd, CI usw.).

Zusätzlich lädt es:

* `.env` aus dem aktuellen Arbeitsverzeichnis (falls vorhanden)
* ein globales Fallback-`.env` aus `~/.openclaw/.env` (aka `$OPENCLAW_STATE_DIR/.env`)

Keine der `.env`-Dateien überschreibt bestehende Env Vars.

Du kannst auch Inline-Env-Vars in der Konfiguration angeben. Diese werden nur angewendet, wenn
in der Prozessumgebung der Schlüssel fehlt (gleiche Regel: nichts wird überschrieben):

```json5
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: {
      GROQ_API_KEY: "gsk-..."
    }
  }
}
```

Siehe [/environment](/de/environment) für die genaue Reihenfolge und die Quellen.

<div id="envshellenv-optional">
  ### `env.shellEnv` (optional)
</div>

Optionale Opt-in-Komfortfunktion: Wenn aktiviert und noch keiner der erwarteten Keys gesetzt ist, führt OpenClaw deine Login-Shell aus und importiert nur die fehlenden erwarteten Keys (ohne bestehende Werte jemals zu überschreiben).
Das entspricht effektiv einem `source` deines Shell-Profils.

```json5
{
  env: {
    shellEnv: {
      enabled: true,
      timeoutMs: 15000
    }
  }
}
```

Entspricht den folgenden Umgebungsvariablen:

* `OPENCLAW_LOAD_SHELL_ENV=1`
* `OPENCLAW_SHELL_ENV_TIMEOUT_MS=15000`

<div id="env-var-substitution-in-config">
  ### Ersetzung von Env-Variablen in der Konfiguration
</div>

Du kannst Umgebungsvariablen direkt in jedem String-Wert der Konfiguration mit der Syntax
`${VAR_NAME}` referenzieren. Die Variablen werden beim Laden der Konfiguration ersetzt, bevor die Validierung ausgeführt wird.

```json5
{
  models: {
    providers: {
      "vercel-gateway": {
        apiKey: "${VERCEL_GATEWAY_API_KEY}"
      }
    }
  },
  gateway: {
    auth: {
      token: "${OPENCLAW_GATEWAY_TOKEN}"
    }
  }
}
```

**Regeln:**

* Es werden nur großgeschriebene Namen von Umgebungsvariablen erkannt: `[A-Z_][A-Z0-9_]*`
* Fehlende oder leere Umgebungsvariablen führen beim Laden der Konfiguration zu einem Fehler
* Mit `$${VAR}` maskieren, um `${VAR}` wörtlich auszugeben
* Funktioniert mit `$include` (in eingebundenen Dateien wird die Ersetzung ebenfalls vorgenommen)

**Inline-Ersetzung:**

```json5
{
  models: {
    providers: {
      custom: {
        baseUrl: "${CUSTOM_API_BASE}/v1"  // → "https://api.example.com/v1"
      }
    }
  }
}
```

<div id="auth-storage-oauth-api-keys">
  ### Auth-Speicherung (OAuth + API-Schlüssel)
</div>

OpenClaw speichert **pro Agent** Auth-Profile (OAuth + API-Schlüssel) in:

* `<agentDir>/auth-profiles.json` (Standardpfad: `~/.openclaw/agents/<agentId>/agent/auth-profiles.json`)

Siehe auch: [/concepts/oauth](/de/concepts/oauth)

Veraltete OAuth-Importe:

* `~/.openclaw/credentials/oauth.json` (oder `$OPENCLAW_STATE_DIR/credentials/oauth.json`)

Der eingebettete Pi-Agent führt einen Laufzeit-Cache unter:

* `<agentDir>/auth.json` (wird automatisch verwaltet; nicht manuell bearbeiten)

Veraltetes Agent-Verzeichnis (vor Multi-Agent):

* `~/.openclaw/agent/*` (wird von `openclaw doctor` migriert nach `~/.openclaw/agents/<defaultAgentId>/agent/*`)

Überschreibungen (Overrides):

* OAuth-Verzeichnis (nur für veraltete Importe): `OPENCLAW_OAUTH_DIR`
* Agent-Verzeichnis (Override für das Standard-Agent-Root-Verzeichnis): `OPENCLAW_AGENT_DIR` (bevorzugt), `PI_CODING_AGENT_DIR` (veraltet)

Bei der ersten Verwendung importiert OpenClaw die `oauth.json`-Einträge in `auth-profiles.json`.

<div id="auth">
  ### `auth`
</div>

Optionale Metadaten für Auth-Profile. Hier werden **keine** Secrets gespeichert; es werden Profil-IDs einem Anbieter + Modus (und optional einer E-Mail-Adresse) zugeordnet und die Rotationsreihenfolge der Anbieter für Failover definiert.

```json5
{
  auth: {
    profiles: {
      "anthropic:me@example.com": { provider: "anthropic", mode: "oauth", email: "me@example.com" },
      "anthropic:work": { provider: "anthropic", mode: "api_key" }
    },
    order: {
      anthropic: ["anthropic:me@example.com", "anthropic:work"]
    }
  }
}
```

<div id="agentslistidentity">
  ### `agents.list[].identity`
</div>

Optionale agent-spezifische Identität, die für Standardwerte und UX verwendet wird. Diese wird vom macOS-Onboarding-Assistenten geschrieben.

Wenn gesetzt, leitet OpenClaw daraus Standardwerte ab (nur, wenn du sie nicht explizit gesetzt hast):

* `messages.ackReaction` aus `identity.emoji` des **aktiven agent** (Fallback: 👀)
* `agents.list[].groupChat.mentionPatterns` aus `identity.name`/`identity.emoji` des agent (sodass „@Samantha“ in Gruppen über Telegram/Slack/Discord/Google Chat/iMessage/WhatsApp funktioniert)
* `identity.avatar` akzeptiert einen zum arbeitsbereich relativen Bildpfad oder eine Remote-URL/Data-URL. Lokale Dateien müssen sich innerhalb des agent-arbeitsbereichs befinden.

`identity.avatar` akzeptiert:

* arbeitsbereich-relativen Pfad (muss innerhalb des agent-arbeitsbereichs bleiben)
* `http(s)`-URL
* `data:`-URI

```json5
{
  agents: {
    list: [
      {
        id: "main",
        identity: {
          name: "Samantha",
          theme: "hilfsbereites Faultier",
          emoji: "🦥",
          avatar: "avatars/samantha.png"
        }
      }
    ]
  }
}
```

<div id="wizard">
  ### `wizard`
</div>

Von CLI-Wizards (`onboard`, `configure`, `doctor`) erzeugte Metadaten.

```json5
{
  wizard: {
    lastRunAt: "2026-01-01T00:00:00.000Z",
    lastRunVersion: "2026.1.4",
    lastRunCommit: "abc1234",
    lastRunCommand: "configure",
    lastRunMode: "local"
  }
}
```

<div id="logging">
  ### `logging`
</div>

* Standard-Logdatei: `/tmp/openclaw/openclaw-YYYY-MM-DD.log`
* Wenn du einen festen Pfad möchtest, setze `logging.file` auf `/tmp/openclaw/openclaw.log`.
* Die Konsolenausgabe kann separat angepasst werden über:
  * `logging.consoleLevel` (Standard ist `info`, wird auf `debug` angehoben, wenn `--verbose` verwendet wird)
  * `logging.consoleStyle` (`pretty` | `compact` | `json`)
* Tool-Zusammenfassungen können geschwärzt werden, um das Offenlegen von Secrets zu verhindern:
  * `logging.redactSensitive` (`off` | `tools`, Standard: `tools`)
  * `logging.redactPatterns` (Array von Regex-Strings; überschreibt die Standardwerte)

```json5
{
  logging: {
    level: "info",
    file: "/tmp/openclaw/openclaw.log",
    consoleLevel: "info",
    consoleStyle: "pretty",
    redactSensitive: "tools",
    redactPatterns: [
      // Beispiel: Standardwerte mit eigenen Regeln überschreiben.
      "\\bTOKEN\\b\\s*[=:]\\s*([\"']?)([^\\s\"']+)\\1",
      "/\\bsk-[A-Za-z0-9_-]{8,}\\b/gi"
    ]
  }
}
```

<div id="channelswhatsappdmpolicy">
  ### `channels.whatsapp.dmPolicy`
</div>

Steuert, wie WhatsApp‑Direktchats (DMs) behandelt werden:

* `"pairing"` (Standard): Unbekannte Absender erhalten einen Kopplungscode; der Owner muss sie genehmigen
* `"allowlist"`: Nur Absender in `channels.whatsapp.allowFrom` (oder gekoppeltem Allowlist‑Store) zulassen
* `"open"`: Alle eingehenden DMs zulassen (**erfordert**, dass `channels.whatsapp.allowFrom` `"*"` enthält; der Richtwert `open` bedeutet hier, dass Nachrichten von beliebigen Absendern uneingeschränkt akzeptiert werden)
* `"disabled"`: Alle eingehenden DMs ignorieren

Kopplungscodes laufen nach 1 Stunde ab; der Bot sendet einen Kopplungscode nur, wenn eine neue Anfrage erstellt wird. Ausstehende DM‑Kopplungsanfragen sind standardmäßig auf **3 pro Kanal** begrenzt.

Kopplungsfreigaben:

* `openclaw pairing list whatsapp`
* `openclaw pairing approve whatsapp <code>`

<div id="channelswhatsappallowfrom">
  ### `channels.whatsapp.allowFrom`
</div>

Allowlist mit E.164‑Telefonnummern, die automatische WhatsApp‑Antworten auslösen dürfen (**nur DMs**).
Wenn leer und `channels.whatsapp.dmPolicy="pairing"`, erhalten unbekannte Absender einen Kopplungscode.
Für Gruppen verwenden Sie `channels.whatsapp.groupPolicy` + `channels.whatsapp.groupAllowFrom`.

```json5
{
  channels: {
    whatsapp: {
      dmPolicy: "pairing", // pairing | allowlist | open | disabled
      allowFrom: ["+15555550123", "+447700900123"],
      textChunkLimit: 4000, // optional outbound chunk size (chars)
      chunkMode: "length", // optional: Chunking-Modus (length | newline)
      mediaMaxMb: 50 // optional inbound media cap (MB)
    }
  }
}
```

<div id="channelswhatsappsendreadreceipts">
  ### `channels.whatsapp.sendReadReceipts`
</div>

Steuert, ob eingehende WhatsApp-Nachrichten als gelesen markiert werden (blaue Haken). Standardwert: `true`.

Der Self-Chat-Modus sendet niemals Lesebestätigungen, selbst wenn sie aktiviert sind.

Kontospezifische Überschreibung: `channels.whatsapp.accounts.<id>.sendReadReceipts`.

```json5
{
  channels: {
    whatsapp: { sendReadReceipts: false }
  }
}
```

<div id="channelswhatsappaccounts-multi-account">
  ### `channels.whatsapp.accounts` (Multi-Account)
</div>

Mehrere WhatsApp-Konten gleichzeitig in einem Gateway betreiben:

```json5
{
  channels: {
    whatsapp: {
      accounts: {
        default: {}, // optional; keeps the default id stable
        personal: {},
        biz: {
          // Optionale Überschreibung. Standard: ~/.openclaw/credentials/whatsapp/biz
          // authDir: "~/.openclaw/credentials/whatsapp/biz",
        }
      }
    }
  }
}
```

Hinweise:

* Ausgehende Befehle verwenden standardmäßig das Konto `default`, sofern vorhanden; andernfalls die erste konfigurierte Konto-ID in sortierter Reihenfolge.
* Das Legacy-Authentifizierungsverzeichnis der Ein-Konto-Baileys-Konfiguration wird von `openclaw doctor` nach `whatsapp/default` migriert.

<div id="channelstelegramaccounts-channelsdiscordaccounts-channelsgooglechataccounts-channelsslackaccounts-channelsmattermostaccounts-channelssignalaccounts-channelsimessageaccounts">
  ### `channels.telegram.accounts` / `channels.discord.accounts` / `channels.googlechat.accounts` / `channels.slack.accounts` / `channels.mattermost.accounts` / `channels.signal.accounts` / `channels.imessage.accounts`
</div>

Mehrere Accounts pro Kanal verwenden (jeder Account hat seine eigene `accountId` und optional einen `name`):

```json5
{
  channels: {
    telegram: {
      accounts: {
        default: {
          name: "Primary bot",
          botToken: "123456:ABC..."
        },
        alerts: {
          name: "Alerts bot",
          botToken: "987654:XYZ..."
        }
      }
    }
  }
}
```

Hinweise:

* `default` wird verwendet, wenn `accountId` weggelassen wird (CLI + Routing).
* Umgebungs-Token gelten nur für das **default**-Konto.
* Grundlegende Kanaleinstellungen (Gruppenrichtlinien, Mention-Gating usw.) gelten für alle Konten, sofern sie nicht pro Konto überschrieben werden.
* Verwende `bindings[].match.accountId`, um jedes Konto zu einem anderen agents.defaults zu routen.

<div id="group-chat-mention-gating-agentslistgroupchat-messagesgroupchat">
  ### Steuerung von Erwähnungen in Gruppenchats (`agents.list[].groupChat` + `messages.groupChat`)
</div>

Gruppennachrichten sind standardmäßig auf **Erwähnung erforderlich** gesetzt (entweder Metadaten-Erwähnung oder Regex-Muster). Gilt für Gruppenchats in WhatsApp, Telegram, Discord, Google Chat und iMessage.

**Erwähnungstypen:**

* **Metadaten-Erwähnungen**: Native @-Erwähnungen der Plattform (z. B. Tippen-zum-Erwähnen in WhatsApp). Im WhatsApp-Self-Chat-Modus ignoriert (siehe `channels.whatsapp.allowFrom`).
* **Textmuster**: Regex-Muster, definiert in `agents.list[].groupChat.mentionPatterns`. Werden immer geprüft, unabhängig vom Self-Chat-Modus.
* Die Erwähnungssteuerung wird nur erzwungen, wenn eine Erwähnungserkennung möglich ist (native Erwähnungen oder mindestens ein `mentionPattern`).

```json5
{
  messages: {
    groupChat: { historyLimit: 50 }
  },
  agents: {
    list: [
      { id: "main", groupChat: { mentionPatterns: ["@openclaw", "openclaw"] } }
    ]
  }
}
```

`messages.groupChat.historyLimit` legt die globale Standardeinstellung für den Gruppenverlaufs‑Kontext fest. Kanäle können dies mit `channels.<channel>.historyLimit` (oder `channels.<channel>.accounts.*.historyLimit` bei mehreren Accounts) überschreiben. Setze `0`, um das History‑Wrapping zu deaktivieren.

<div id="dm-history-limits">
  #### DM-Verlaufsgrenzen
</div>

DM-Unterhaltungen verwenden einen sitzungsbasierten Verlauf, der vom agent verwaltet wird. Du kannst die Anzahl der Benutzernachrichten begrenzen, die pro DM-Sitzung aufbewahrt werden:

```json5
{
  channels: {
    telegram: {
      dmHistoryLimit: 30,  // DM-Sitzungen auf 30 Benutzer-Turns begrenzen
      dms: {
        "123456789": { historyLimit: 50 }  // benutzerspezifische Überschreibung (Benutzer-ID)
      }
    }
  }
}
```

Reihenfolge der Auflösung:

1. DM-spezifisches Override: `channels.<provider>.dms[userId].historyLimit`
2. Anbieter-Standard: `channels.<provider>.dmHistoryLimit`
3. Kein Limit (gesamter Verlauf wird beibehalten)

Unterstützte Anbieter: `telegram`, `whatsapp`, `discord`, `slack`, `signal`, `imessage`, `msteams`.

Agent-spezifisches Override (hat Vorrang, wenn gesetzt, auch bei `[]`):

```json5
{
  agents: {
    list: [
      { id: "work", groupChat: { mentionPatterns: ["@workbot", "\\+15555550123"] } },
      { id: "personal", groupChat: { mentionPatterns: ["@homebot", "\\+15555550999"] } }
    ]
  }
}
```

Mention-Gating-Standardwerte gelten pro Channel (`channels.whatsapp.groups`, `channels.telegram.groups`, `channels.imessage.groups`, `channels.discord.guilds`). Wenn `*.groups` gesetzt ist, wirkt diese Einstellung auch als Gruppen-Allowlist; füge `"*"` hinzu, um alle Gruppen zuzulassen.

Um **nur** auf bestimmte Text-Trigger zu reagieren (und native @-Erwähnungen zu ignorieren):

```json5
{
  channels: {
    whatsapp: {
      // Fügen Sie Ihre eigene Nummer hinzu, um den Self-Chat-Modus zu aktivieren (native @-Erwähnungen werden ignoriert).
      allowFrom: ["+15555550123"],
      groups: { "*": { requireMention: true } }
    }
  },
  agents: {
    list: [
      {
        id: "main",
        groupChat: {
          // Only these text patterns will trigger responses
          mentionPatterns: ["reisponde", "@openclaw"]
        }
      }
    ]
  }
}
```

<div id="group-policy-per-channel">
  ### Gruppenrichtlinie (pro Kanal)
</div>

Verwende `channels.*.groupPolicy`, um zu steuern, ob Nachrichten aus Gruppen/Räumen überhaupt akzeptiert werden:

```json5
{
  channels: {
    whatsapp: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15551234567"]
    },
    telegram: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["tg:123456789", "@alice"]
    },
    signal: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15551234567"]
    },
    imessage: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["chat_id:123"]
    },
    msteams: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["user@org.com"]
    },
    discord: {
      groupPolicy: "allowlist",
      guilds: {
        "GUILD_ID": {
          channels: { help: { allow: true } }
        }
      }
    },
    slack: {
      groupPolicy: "allowlist",
      channels: { "#general": { allow: true } }
    }
  }
}
```

Hinweise:

* `"open"`: Gruppen umgehen die Allowlist; Mention-Gating gilt weiterhin.
* `"disabled"`: blockiert alle Gruppen-/Raumnachrichten.
* `"allowlist"`: erlaubt nur Gruppen/Räume, die mit der konfigurierten Allowlist übereinstimmen.
* `channels.defaults.groupPolicy` legt den Standardwert fest, wenn die `groupPolicy` eines anbieters nicht gesetzt ist.
* WhatsApp/Telegram/Signal/iMessage/Microsoft Teams verwenden `groupAllowFrom` (Fallback: explizites `allowFrom`).
* Discord/Slack verwenden Kanal-Allowlists (`channels.discord.guilds.*.channels`, `channels.slack.channels`).
* Gruppen-DMs (Discord/Slack) werden weiterhin durch `dm.groupEnabled` + `dm.groupChannels` gesteuert.
* Standard ist `groupPolicy: "allowlist"` (sofern nicht durch `channels.defaults.groupPolicy` überschrieben); wenn keine Allowlist konfiguriert ist, werden Gruppennachrichten blockiert.

<div id="multi-agent-routing-agentslist-bindings">
  ### Multi-agent routing (`agents.list` + `bindings`)
</div>

Führe mehrere isolierte agenten (getrennter Arbeitsbereich, `agentDir`, Sitzungen) in einem Gateway aus.
Eingehende Nachrichten werden über Bindings an einen agent geroutet.

* `agents.list[]`: agent-spezifische Overrides.
  * `id`: stabile agent-ID (erforderlich).
  * `default`: optional; wenn mehrere gesetzt sind, gewinnt der erste, und es wird eine Warnung protokolliert.
    Wenn keiner gesetzt ist, ist der **erste Eintrag** in der Liste der Standard-agent.
  * `name`: Anzeigename für den agent.
  * `workspace`: Standard `~/.openclaw/workspace-<agentId>` (für `main` Fallback auf `agents.defaults.workspace`).
  * `agentDir`: Standard `~/.openclaw/agents/<agentId>/agent`.
  * `model`: agent-spezifisches Standardmodell, überschreibt `agents.defaults.model` für diesen agent.
    * String-Form: `"provider/model"`, überschreibt nur `agents.defaults.model.primary`
    * Objekt-Form: `{ primary, fallbacks }` (Fallbacks überschreiben `agents.defaults.model.fallbacks`; `[]` deaktiviert globale Fallbacks für diesen agent)
  * `identity`: agent-spezifischer Name/Theme/Emoji (wird für Mention-Muster + Ack-Reaktionen verwendet).
  * `groupChat`: agent-spezifisches Mention-Gating (`mentionPatterns`).
  * `sandbox`: agent-spezifische sandbox-Konfiguration (überschreibt `agents.defaults.sandbox`).
    * `mode`: `"off"` | `"non-main"` | `"all"`
    * `workspaceAccess`: `"none"` | `"ro"` | `"rw"`
    * `scope`: `"session"` | `"agent"` | `"shared"`
    * `workspaceRoot`: benutzerdefiniertes sandbox-Arbeitsbereichs-Root
    * `docker`: agent-spezifische Docker-Overrides (z. B. `image`, `network`, `env`, `setupCommand`, Limits; wird ignoriert, wenn `scope: "shared"`)
    * `browser`: agent-spezifische sandboxed-Browser-Overrides (wird ignoriert, wenn `scope: "shared"`)
    * `prune`: agent-spezifische sandbox-Pruning-Overrides (wird ignoriert, wenn `scope: "shared"`)
  * `subagents`: agent-spezifische Sub-agent-Defaults.
    * `allowAgents`: Allowlist von agent-IDs für `sessions_spawn` von diesem agent (`["*"]` = alle erlauben; Standard: nur derselbe agent)
  * `tools`: agent-spezifische Tool-Einschränkungen (werden vor der sandbox-Tool-Policy angewendet).
    * `profile`: Basis-Tool-Profil (wird vor allow/deny angewendet)
    * `allow`: Array erlaubter Tool-Namen
    * `deny`: Array abgelehnter Tool-Namen (deny gewinnt)
* `agents.defaults`: gemeinsame agent-Defaults (Modell, Arbeitsbereich, sandbox usw.).
* `bindings[]`: routet eingehende Nachrichten zu einer `agentId`.
  * `match.channel` (erforderlich)
  * `match.accountId` (optional; `*` = beliebiger Account; weggelassen = Standard-Account)
  * `match.peer` (optional; `{ kind: dm|group|channel, id }`)
  * `match.guildId` / `match.teamId` (optional; kanalspezifisch)

Deterministische Match-Reihenfolge:

1. `match.peer`
2. `match.guildId`
3. `match.teamId`
4. `match.accountId` (exakt, kein peer/guild/team)
5. `match.accountId: "*"` (kanalweit, kein peer/guild/team)
6. Standard-agent (`agents.list[].default`, sonst erster Listeneintrag, sonst `"main"`)

Innerhalb jeder Match-Stufe gewinnt der erste passende Eintrag in `bindings`.

<div id="per-agent-access-profiles-multi-agent">
  #### Agent-spezifische Zugriffsprofile (Multi-Agent)
</div>

Jeder agent kann seine eigene sandbox- und Tool-Richtlinie haben. Verwende dies, um
unterschiedliche Zugriffsebenen in einem Gateway zu mischen:

* **Voller Zugriff** (persönlicher agent)
* **Schreibgeschützte** Tools + arbeitsbereich
* **Kein Dateisystemzugriff** (nur Messaging-/Sitzungs-Tools)

Siehe [Multi-Agent-Sandbox &amp; Tools](/de/multi-agent-sandbox-tools) für Priorität und
weitere Beispiele.

Voller Zugriff (keine sandbox):

```json5
{
  agents: {
    list: [
      {
        id: "personal",
        workspace: "~/.openclaw/workspace-personal",
        sandbox: { mode: "off" }
      }
    ]
  }
}
```

Schreibgeschützte Tools + schreibgeschützter Arbeitsbereich:

```json5
{
  agents: {
    list: [
      {
        id: "family",
        workspace: "~/.openclaw/workspace-family",
        sandbox: {
          mode: "all",
          scope: "agent",
          workspaceAccess: "ro"
        },
        tools: {
          allow: ["read", "sessions_list", "sessions_history", "sessions_send", "sessions_spawn", "session_status"],
          deny: ["write", "edit", "apply_patch", "exec", "process", "browser"]
        }
      }
    ]
  }
}
```

Kein Dateisystemzugriff (Messaging-/Sitzungstools aktiviert):

```json5
{
  agents: {
    list: [
      {
        id: "public",
        workspace: "~/.openclaw/workspace-public",
        sandbox: {
          mode: "all",
          scope: "agent",
          workspaceAccess: "none"
        },
        tools: {
          allow: ["sessions_list", "sessions_history", "sessions_send", "sessions_spawn", "session_status", "whatsapp", "telegram", "slack", "discord", "gateway"],
          deny: ["read", "write", "edit", "apply_patch", "exec", "process", "browser", "canvas", "nodes", "cron", "gateway", "image"]
        }
      }
    ]
  }
}
```

Beispiel: zwei WhatsApp-Konten → zwei Agenten:

```json5
{
  agents: {
    list: [
      { id: "home", default: true, workspace: "~/.openclaw/workspace-home" },
      { id: "work", workspace: "~/.openclaw/workspace-work" }
    ]
  },
  bindings: [
    { agentId: "home", match: { channel: "whatsapp", accountId: "personal" } },
    { agentId: "work", match: { channel: "whatsapp", accountId: "biz" } }
  ],
  channels: {
    whatsapp: {
      accounts: {
        personal: {},
        biz: {},
      }
    }
  }
}
```

<div id="toolsagenttoagent-optional">
  ### `tools.agentToAgent` (optional)
</div>

Agent-zu-Agent-Kommunikation ist optional (Opt-in):

```json5
{
  tools: {
    agentToAgent: {
      enabled: false,
      allow: ["home", "work"]
    }
  }
}
```

<div id="messagesqueue">
  ### `messages.queue`
</div>

Steuert, wie sich eingehende Nachrichten verhalten, wenn bereits eine Agent-Ausführung aktiv ist.

```json5
{
  messages: {
    queue: {
      mode: "collect", // steer | followup | collect | steer-backlog (steer+backlog ok) | interrupt (queue=steer veraltet)
      debounceMs: 1000,
      cap: 20,
      drop: "summarize", // old | new | summarize
      byChannel: {
        whatsapp: "collect",
        telegram: "collect",
        discord: "collect",
        imessage: "collect",
        webchat: "collect"
      }
    }
  }
}
```

<div id="messagesinbound">
  ### `messages.inbound`
</div>

Entprellt schnelle eingehende Nachrichten vom **gleichen Absender**, sodass mehrere direkt aufeinanderfolgende Nachrichten zu einem einzelnen Agent-Turn zusammengefasst werden. Das Debouncing erfolgt getrennt pro Channel und Konversation und verwendet die neueste Nachricht für Antwort-Threading und IDs.

```json5
{
  messages: {
    inbound: {
      debounceMs: 2000, // 0 deaktiviert
      byChannel: {
        whatsapp: 5000,
        slack: 1500,
        discord: 1500
      }
    }
  }
}
```

Hinweise:

* Debounce fasst **nur Text**-Nachrichten zu Batches zusammen; Medien und Anhänge werden sofort gesendet.
* Steuerbefehle (z. B. `/queue`, `/new`) umgehen das Debouncing und werden immer als eigenständige Nachrichten behandelt.

<div id="commands-chat-command-handling">
  ### `commands` (Verarbeitung von Chat-Befehlen)
</div>

Steuert, wie Chat-Befehle über Konnektoren hinweg aktiviert werden.

```json5
{
  commands: {
    native: "auto",         // register native commands when supported (auto)
    text: true,             // parse slash commands in chat messages
    bash: false,            // ! erlauben (Alias: /bash) (nur Host; erfordert tools.elevated-Allowlists)
    bashForegroundMs: 2000, // bash foreground window (0 backgrounds immediately)
    config: false,          // allow /config (writes to disk)
    debug: false,           // allow /debug (runtime-only overrides)
    restart: false,         // allow /restart + gateway restart tool
    useAccessGroups: true   // enforce access-group allowlists/policies for commands
  }
}
```

Hinweise:

* Textbefehle müssen als **eigenständige** Nachricht gesendet werden und mit einem führenden `/` beginnen (keine Klartext-Aliasse).
* `commands.text: false` deaktiviert das Parsen von Chat-Nachrichten nach Befehlen.
* `commands.native: "auto"` (Standard) aktiviert native Befehle für Discord/Telegram und lässt Slack deaktiviert; nicht unterstützte Kanäle bleiben reine Textkanäle.
* Setze `commands.native: true|false`, um alles zu erzwingen, oder überschreibe pro Kanal mit `channels.discord.commands.native`, `channels.telegram.commands.native`, `channels.slack.commands.native` (boolesch oder `"auto"`). `false` entfernt beim Start zuvor registrierte Befehle auf Discord/Telegram; Slack-Befehle werden in der Slack-App verwaltet.
* `channels.telegram.customCommands` fügt zusätzliche Menüeinträge im Telegram-Bot hinzu. Namen werden normalisiert; Konflikte mit nativen Befehlen werden ignoriert.
* `commands.bash: true` aktiviert `! <cmd>`, um Shell-Befehle auf dem Host auszuführen (`/bash <cmd>` funktioniert ebenfalls als Alias). Erfordert `tools.elevated.enabled` und dass der Absender in der Allowlist unter `tools.elevated.allowFrom.<channel>` eingetragen ist.
* `commands.bashForegroundMs` steuert, wie lange bash wartet, bevor der Prozess in den Hintergrund verschoben wird. Während ein bash-Job läuft, werden neue `! <cmd>`-Anfragen abgelehnt (jeweils nur ein Auftrag gleichzeitig).
* `commands.config: true` aktiviert `/config` (liest/schreibt `openclaw.json`).
* `channels.<provider>.configWrites` steuert Konfigurationsänderungen, die von diesem Kanal ausgelöst werden (Standard: true). Dies gilt für `/config set|unset` plus anbieter­spezifische Auto-Migrationen (Telegram-Supergroup-ID-Änderungen, Slack-Channel-ID-Änderungen).
* `commands.debug: true` aktiviert `/debug` (nur Laufzeit-Overrides).
* `commands.restart: true` aktiviert `/restart` und die Neustartaktion des Gateway-Tools.
* `commands.useAccessGroups: false` erlaubt es Befehlen, Allowlists und Richtlinien von Access-Gruppen zu umgehen.
* Slash-Befehle und Direktiven werden nur für **autorisierte Absender** berücksichtigt. Die Autorisierung wird aus
  Kanal-Allowlists/Kopplung plus `commands.useAccessGroups` abgeleitet.

<div id="web-whatsapp-web-channel-runtime">
  ### `web` (WhatsApp-Web-Kanal-Runtime)
</div>

WhatsApp läuft über den Web-Kanal des Gateways (Baileys Web). Dieser wird automatisch gestartet, sobald eine verknüpfte Sitzung vorhanden ist.
Setze `web.enabled: false`, um ihn standardmäßig deaktiviert zu lassen.

```json5
{
  web: {
    enabled: true,
    heartbeatSeconds: 60,
    reconnect: {
      initialMs: 2000,
      maxMs: 120000,
      factor: 1.4,
      jitter: 0.2,
      maxAttempts: 0
    }
  }
}
```

<div id="channelstelegram-bot-transport">
  ### `channels.telegram` (Bot-Transport)
</div>

OpenClaw startet Telegram nur, wenn ein `channels.telegram`-Konfigurationsabschnitt existiert. Das Bot-Token wird aus `channels.telegram.botToken` (oder `channels.telegram.tokenFile`) bezogen, mit `TELEGRAM_BOT_TOKEN` als Fallback für das Standardkonto.
Setze `channels.telegram.enabled: false`, um den automatischen Start zu deaktivieren.
Unterstützung für mehrere Konten befindet sich unter `channels.telegram.accounts` (siehe den Multi-Account-Abschnitt oben). Env-Tokens (Umgebungsvariablen-Tokens) gelten nur für das Standardkonto.
Setze `channels.telegram.configWrites: false`, um von Telegram ausgelöste Schreibzugriffe auf die Konfiguration zu blockieren (einschließlich Supergroup-ID-Migrationen und `/config set|unset`).

```json5
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "your-bot-token",
      dmPolicy: "pairing",                 // pairing | allowlist | open | disabled
      allowFrom: ["tg:123456789"],         // optional; "open" requires ["*"]
      groups: {
        "*": { requireMention: true },
        "-1001234567890": {
          allowFrom: ["@admin"],
          systemPrompt: "Keep answers brief.",
          topics: {
            "99": {
              requireMention: false,
              skills: ["search"],
              systemPrompt: "Stay on topic."
            }
          }
        }
      },
      customCommands: [
        { command: "backup", description: "Git backup" },
        { command: "generate", description: "Create an image" }
      ],
      historyLimit: 50,                     // include last N group messages as context (0 disables)
      replyToMode: "first",                 // off | first | all
      linkPreview: true,                   // toggle outbound link previews
      streamMode: "partial",               // off | partial | block (Entwurfs-Streaming; getrennt von Block-Streaming)
      draftChunk: {                        // optional; only for streamMode=block
        minChars: 200,
        maxChars: 800,
        breakPreference: "paragraph"       // paragraph | newline | sentence
      },
      actions: { reactions: true, sendMessage: true }, // tool action gates (false disables)
      reactionNotifications: "own",   // off | own | all
      mediaMaxMb: 5,
      retry: {                             // outbound retry policy
        attempts: 3,
        minDelayMs: 400,
        maxDelayMs: 30000,
        jitter: 0.1
      },
      network: {                           // transport overrides
        autoSelectFamily: false
      },
      proxy: "socks5://localhost:9050",
      webhookUrl: "https://example.com/telegram-webhook",
      webhookSecret: "secret",
      webhookPath: "/telegram-webhook"
    }
  }
}
```

Hinweise zum Streaming von Entwürfen:

* Verwendet Telegram `sendMessageDraft` (Entwurfs-Sprechblase, keine echte Nachricht).
* Setzt **private Chat-Themen** voraus (`message_thread_id` in DMs; Bot hat Themen aktiviert).
* `/reasoning stream` streamt die Begründung in den Entwurf und sendet anschließend die endgültige Antwort.
  Standardwerte und das Verhalten der Retry-Richtlinie sind in [Retry policy](/de/concepts/retry) dokumentiert.

<div id="channelsdiscord-bot-transport">
  ### `channels.discord` (Bot-Transport)
</div>

Konfiguriere den Discord-Bot, indem du das Bot-Token und optionales Gating (Zugriffsbeschränkung) angibst:
Mehrkontenunterstützung befindet sich unter `channels.discord.accounts` (siehe den Multi-Account-Abschnitt oben). Env-Tokens gelten nur für den Standard-Account.

```json5
{
  channels: {
    discord: {
      enabled: true,
      token: "your-bot-token",
      mediaMaxMb: 8,                          // clamp inbound media size
      allowBots: false,                       // allow bot-authored messages
      actions: {                              // tool action gates (false disables)
        reactions: true,
        stickers: true,
        polls: true,
        permissions: true,
        messages: true,
        threads: true,
        pins: true,
        search: true,
        memberInfo: true,
        roleInfo: true,
        roles: false,
        channelInfo: true,
        voiceStatus: true,
        events: true,
        moderation: false
      },
      replyToMode: "off",                     // off | first | all
      dm: {
        enabled: true,                        // disable all DMs when false
        policy: "pairing",                    // pairing | allowlist | open | disabled
        allowFrom: ["1234567890", "steipete"], // optional DM allowlist ("open" requires ["*"])
        groupEnabled: false,                 // enable group DMs
        groupChannels: ["openclaw-dm"]          // optional group DM allowlist
      },
      guilds: {
        "123456789012345678": {               // guild id (preferred) or slug
          slug: "friends-of-openclaw",
          requireMention: false,              // per-guild default
          reactionNotifications: "own",       // off | own | all | allowlist
          users: ["987654321098765432"],      // optional per-guild user allowlist
          channels: {
            general: { allow: true },
            help: {
              allow: true,
              requireMention: true,
              users: ["987654321098765432"],
              skills: ["docs"],
              systemPrompt: "Short answers only."
            }
          }
        }
      },
      historyLimit: 20,                       // include last N guild messages as context
      textChunkLimit: 2000,                   // optional outbound text chunk size (chars)
      chunkMode: "length",                    // optional chunking mode (length | newline)
      maxLinesPerMessage: 17,                 // weiche maximale Zeilenanzahl pro Nachricht (Discord-UI-Clipping)
      retry: {                                // outbound retry policy
        attempts: 3,
        minDelayMs: 500,
        maxDelayMs: 30000,
        jitter: 0.1
      }
    }
  }
}
```

OpenClaw startet Discord nur, wenn ein `channels.discord`-Konfigurationsabschnitt vorhanden ist. Das Token wird aus `channels.discord.token` ermittelt, mit `DISCORD_BOT_TOKEN` als Fallback für das Standardkonto (sofern `channels.discord.enabled` nicht `false` ist). Verwende `user:<id>` (DM) oder `channel:<id>` (Guild-Channel), wenn du Zustellziele für Cron-/CLI-Befehle angibst; reine numerische IDs sind mehrdeutig und werden abgelehnt.
Guild-Slugs sind kleingeschrieben, Leerzeichen werden durch `-` ersetzt; Channel-Keys verwenden den geslugten Channel-Namen (ohne führendes `#`). Bevorzuge Guild-IDs als Keys, um Mehrdeutigkeiten bei Umbenennungen zu vermeiden.
Von Bots verfasste Nachrichten werden standardmäßig ignoriert. Aktiviere sie mit `channels.discord.allowBots` (eigene Nachrichten werden weiterhin gefiltert, um Selbst-Antwort-Schleifen zu verhindern).
Reaktions-Benachrichtigungsmodi:

* `off`: keine Reaktions-Events.
* `own`: Reaktionen auf die eigenen Nachrichten des Bots (Standard).
* `all`: alle Reaktionen auf alle Nachrichten.
* `allowlist`: Reaktionen von `guilds.<id>.users` auf alle Nachrichten (leere Liste deaktiviert).
  Ausgehender Text wird durch `channels.discord.textChunkLimit` (Standard 2000) in Blöcke aufgeteilt. Setze `channels.discord.chunkMode="newline"`, um zunächst an Leerzeilen (Absatzgrenzen) zu trennen, bevor nach Länge in Blöcke aufgeteilt wird. Discord-Clients können sehr hohe Nachrichten abschneiden, daher teilt `channels.discord.maxLinesPerMessage` (Standard 17) lange, mehrzeilige Antworten sogar dann auf, wenn sie unter 2000 Zeichen bleiben.
  Standardwerte und Verhalten der Retry-Policy sind unter [Retry policy](/de/concepts/retry) dokumentiert.

<div id="channelsgooglechat-chat-api-webhook">
  ### `channels.googlechat` (Chat-API-Webhook)
</div>

Google Chat läuft über HTTP-Webhooks mit App-Authentifizierung auf App-Ebene (Dienstkonto).
Unterstützung für mehrere Konten befindet sich unter `channels.googlechat.accounts` (siehe den Abschnitt zur Mehrkontenunterstützung oben). Umgebungsvariablen gelten nur für das Standardkonto.

```json5
{
  channels: {
    "googlechat": {
      enabled: true,
      serviceAccountFile: "/path/to/service-account.json",
      audienceType: "app-url",             // app-url | project-number
      audience: "https://gateway.example.com/googlechat",
      webhookPath: "/googlechat",
      botUser: "users/1234567890",        // optional; improves mention detection
      dm: {
        enabled: true,
        policy: "pairing",                // pairing (Kopplung) | allowlist | open (offen für alle) | disabled
        allowFrom: ["users/1234567890"]   // optional; "open" requires ["*"]
      },
      groupPolicy: "allowlist",
      groups: {
        "spaces/AAAA": { allow: true, requireMention: true }
      },
      actions: { reactions: true },
      typingIndicator: "message",
      mediaMaxMb: 20
    }
  }
}
```

Hinweise:

* Service-Account-JSON kann inline (`serviceAccount`) oder dateibasiert (`serviceAccountFile`) angegeben werden.
* Umgebungsvariablen-Fallbacks für das Standardkonto: `GOOGLE_CHAT_SERVICE_ACCOUNT` oder `GOOGLE_CHAT_SERVICE_ACCOUNT_FILE`.
* `audienceType` + `audience` müssen mit der Webhook-Auth-Konfiguration der Chat-App übereinstimmen.
* Verwende `spaces/<spaceId>` oder `users/<userId|email>` beim Festlegen von Zustellzielen.

<div id="channelsslack-socket-mode">
  ### `channels.slack` (Socket-Modus)
</div>

Slack läuft im Socket-Modus und benötigt sowohl ein Bot-Token als auch ein App-Token:

```json5
{
  channels: {
    slack: {
      enabled: true,
      botToken: "xoxb-...",
      appToken: "xapp-...",
      dm: {
        enabled: true,
        policy: "pairing", // pairing | allowlist | open | disabled
        allowFrom: ["U123", "U456", "*"], // optional; "open" requires ["*"]
        groupEnabled: false,
        groupChannels: ["G123"]
      },
      channels: {
        C123: { allow: true, requireMention: true, allowBots: false },
        "#general": {
          allow: true,
          requireMention: true,
          allowBots: false,
          users: ["U123"],
          skills: ["docs"],
          systemPrompt: "Short answers only."
        }
      },
      historyLimit: 50,          // letzte N Kanal-/Gruppennachrichten als Kontext einbeziehen (0 deaktiviert)
      allowBots: false,
      reactionNotifications: "own", // off | own | all | allowlist
      reactionAllowlist: ["U123"],
      replyToMode: "off",           // off | first | all
      thread: {
        historyScope: "thread",     // thread | channel
        inheritParent: false
      },
      actions: {
        reactions: true,
        messages: true,
        pins: true,
        memberInfo: true,
        emojiList: true
      },
      slashCommand: {
        enabled: true,
        name: "openclaw",
        sessionPrefix: "slack:slash",
        ephemeral: true
      },
      textChunkLimit: 4000,
      chunkMode: "length",
      mediaMaxMb: 20
    }
  }
}
```

Multi-Account-Unterstützung findest du unter `channels.slack.accounts` (siehe Abschnitt zur Multi-Account-Konfiguration oben). Env-Tokens gelten nur für das Standardkonto.

OpenClaw startet Slack, wenn der Anbieter aktiviert ist und beide Tokens gesetzt sind (über die Konfiguration oder `SLACK_BOT_TOKEN` + `SLACK_APP_TOKEN`). Verwende `user:<id>` (DM) oder `channel:<id>`, wenn du Zustellziele für Cron-/CLI-Befehle angibst.
Setze `channels.slack.configWrites: false`, um von Slack initiierte Konfigurationsschreibvorgänge zu blockieren (einschließlich Channel-ID-Migrationen und `/config set|unset`).

Vom Bot verfasste Nachrichten werden standardmäßig ignoriert. Aktiviere sie mit `channels.slack.allowBots` oder `channels.slack.channels.<id>.allowBots`.

Modi für Reaktionsbenachrichtigungen:

* `off`: keine Reaktionsereignisse.
* `own`: Reaktionen auf die eigenen Nachrichten des Bots (Standard).
* `all`: alle Reaktionen auf alle Nachrichten.
* `allowlist`: Reaktionen aus `channels.slack.reactionAllowlist` auf alle Nachrichten (leere Liste deaktiviert).

Thread-Sitzungsisolation:

* `channels.slack.thread.historyScope` steuert, ob der Thread-Verlauf pro Thread (`thread`, Standard) oder gemeinsam über den Channel (`channel`) geführt wird.
* `channels.slack.thread.inheritParent` steuert, ob neue Thread-Sitzungen das übergeordnete Channel-Transkript erben (Standard: false).

Slack-Aktionsgruppen (begrenzen `slack`-Tool-Aktionen):

| Aktionsgruppe | Standard | Hinweise                         |
| ------------- | -------- | -------------------------------- |
| reactions     | enabled  | Reagieren + Reaktionen auflisten |
| messages      | enabled  | Lesen/Senden/Bearbeiten/Löschen  |
| pins          | enabled  | Anheften/Lösen/Auflisten         |
| memberInfo    | enabled  | Member-Informationen             |
| emojiList     | enabled  | Liste benutzerdefinierter Emojis |

<div id="channelsmattermost-bot-token">
  ### `channels.mattermost` (Bot-Token)
</div>

Mattermost wird als Plugin bereitgestellt und ist nicht im Core-Installationspaket enthalten.
Installiere es zuerst: `openclaw plugins install @openclaw/mattermost` (oder `./extensions/mattermost` aus einem Git-Checkout).

Mattermost erfordert einen Bot-Token plus die Basis-URL deines Servers:

```json5
{
  channels: {
    mattermost: {
      enabled: true,
      botToken: "mm-token",
      baseUrl: "https://chat.example.com",
      dmPolicy: "pairing",
      chatmode: "oncall", // oncall | onmessage | onchar
      oncharPrefixes: [">", "!"],
      textChunkLimit: 4000,
      chunkMode: "length"
    }
  }
}
```

OpenClaw startet Mattermost, wenn das Konto konfiguriert (Bot-Token + Basis-URL) und aktiviert ist. Token und Basis-URL werden aus `channels.mattermost.botToken` + `channels.mattermost.baseUrl` oder `MATTERMOST_BOT_TOKEN` + `MATTERMOST_URL` für das Standardkonto ermittelt (sofern `channels.mattermost.enabled` nicht `false` ist).

Chat-Modi:

* `oncall` (Standard): antwortet auf Kanalnachrichten nur bei @Erwähnung.
* `onmessage`: antwortet auf jede Kanalnachricht.
* `onchar`: antwortet, wenn eine Nachricht mit einem Trigger-Präfix beginnt (`channels.mattermost.oncharPrefixes`, Standard `[">", "!"]`).

Zugriffskontrolle:

* Standard-DMs: `channels.mattermost.dmPolicy="pairing"` (unbekannte Absender erhalten einen Kopplungscode).
* Öffentliche DMs: `channels.mattermost.dmPolicy="open"` plus `channels.mattermost.allowFrom=["*"]` (erlaubt uneingeschränkten Nachrichteneingang von beliebigen Absendern).
* Gruppen: `channels.mattermost.groupPolicy="allowlist"` ist standardmäßig aktiv (durch Erwähnung gesteuert). Verwende `channels.mattermost.groupAllowFrom`, um Absender einzuschränken.

Mehrkonten-Unterstützung befindet sich unter `channels.mattermost.accounts` (siehe den Mehrkonten-Abschnitt oben). Umgebungsvariablen gelten nur für das Standardkonto.
Verwende `channel:<id>` oder `user:<id>` (oder `@username`), wenn du Zustellziele angibst; reine IDs werden als Kanal-IDs behandelt.

<div id="channelssignal-signal-cli">
  ### `channels.signal` (signal-cli)
</div>

Signal-Reaktionen können Systemereignisse auslösen (gemeinsame Reaktionsfunktionen):

```json5
{
  channels: {
    signal: {
      reactionNotifications: "own", // off | own | all | allowlist
      reactionAllowlist: ["+15551234567", "uuid:123e4567-e89b-12d3-a456-426614174000"],
      historyLimit: 50 // letzten N Gruppennachrichten als Kontext einbeziehen (0 deaktiviert)
    }
  }
}
```

Reaktionsbenachrichtigungsmodi:

* `off`: keine Reaktionsereignisse.
* `own`: Reaktionen auf eigene Nachrichten des Bots (Standardwert).
* `all`: alle Reaktionen auf alle Nachrichten.
* `allowlist`: Reaktionen aus `channels.signal.reactionAllowlist` auf alle Nachrichten (leere Liste deaktiviert).

<div id="channelsimessage-imsg-cli">
  ### `channels.imessage` (imsg CLI)
</div>

OpenClaw startet `imsg rpc` (JSON-RPC über stdio). Es wird kein Daemon und kein Port benötigt.

```json5
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "imsg",
      dbPath: "~/Library/Messages/chat.db",
      remoteHost: "user@gateway-host", // SCP für entfernte Anhänge bei Verwendung eines SSH-Wrappers
      dmPolicy: "pairing", // pairing | allowlist | open | disabled
      allowFrom: ["+15555550123", "user@example.com", "chat_id:123"],
      historyLimit: 50,    // die letzten N Gruppennachrichten als Kontext einbeziehen (0 deaktiviert)
      includeAttachments: false,
      mediaMaxMb: 16,
      service: "auto",
      region: "US"
    }
  }
}
```

Unterstützung für mehrere Accounts befindet sich unter `channels.imessage.accounts` (siehe den Abschnitt zu mehreren Accounts oben).

Hinweise:

* Erfordert Vollzugriff auf die Festplatte für die Messages-Datenbank.
* Der erste Sendevorgang fordert eine Berechtigung für Messages-Automatisierung an.
* Verwende bevorzugt `chat_id:<id>`-Ziele. Nutze `imsg chats --limit 20`, um Chats aufzulisten.
* `channels.imessage.cliPath` kann auf ein Wrapper-Skript zeigen (z. B. `ssh` zu einem anderen Mac, der `imsg rpc` ausführt); verwende SSH-Schlüssel, um Passwortabfragen zu vermeiden.
* Für Remote-SSH-Wrapper setze `channels.imessage.remoteHost`, um Anhänge per SCP abzurufen, wenn `includeAttachments` aktiviert ist.

Beispiel-Wrapper:

```bash
#!/usr/bin/env bash
exec ssh -T gateway-host imsg "$@"
```

<div id="agentsdefaultsworkspace">
  ### `agents.defaults.workspace`
</div>

Legt das **einzige globale Arbeitsbereichsverzeichnis** fest, das vom Agent für Dateioperationen verwendet wird.

Standard: `~/.openclaw/workspace`.

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } }
}
```

Wenn `agents.defaults.sandbox` aktiviert ist, können Nicht-Haupt-Sitzungen dies mit eigenen, je Scope getrennten Arbeitsbereichen unter `agents.defaults.sandbox.workspaceRoot` überschreiben.

<div id="agentsdefaultsreporoot">
  ### `agents.defaults.repoRoot`
</div>

Optionales Repository-Root-Verzeichnis, das in der Runtime-Zeile des System-Prompts angezeigt wird. Wenn nicht gesetzt, versucht OpenClaw,
ein `.git`-Verzeichnis zu erkennen, indem es vom Arbeitsbereich (und aktuellen
Arbeitsverzeichnis) aus die Verzeichnisstruktur nach oben durchsucht. Der Pfad muss existieren, um verwendet zu werden.

```json5
{
  agents: { defaults: { repoRoot: "~/Projects/openclaw" } }
}
```

<div id="agentsdefaultsskipbootstrap">
  ### `agents.defaults.skipBootstrap`
</div>

Deaktiviert die automatische Erstellung der Bootstrap-Dateien des Arbeitsbereichs (`AGENTS.md`, `SOUL.md`, `TOOLS.md`, `IDENTITY.md`, `USER.md` und `BOOTSTRAP.md`).

Verwende diese Option für vorkonfigurierte Deployments, bei denen deine Arbeitsbereich-Dateien aus einem Repository stammen.

```json5
{
  agents: { defaults: { skipBootstrap: true } }
}
```

<div id="agentsdefaultsbootstrapmaxchars">
  ### `agents.defaults.bootstrapMaxChars`
</div>

Maximale Anzahl von Zeichen aus jeder Bootstrap-Datei des jeweiligen Arbeitsbereichs, die vor dem Abschneiden in den System-Prompt eingefügt werden. Standardwert: `20000`.

Wenn eine Datei diese Grenze überschreitet, protokolliert OpenClaw eine Warnung
und fügt einen gekürzten Kopf- und Endbereich mit einer Markierung ein.

```json5
{
  agents: { defaults: { bootstrapMaxChars: 20000 } }
}
```

<div id="agentsdefaultsusertimezone">
  ### `agents.defaults.userTimezone`
</div>

Legt die Zeitzone des Benutzers für den **Systemprompt-Kontext** fest (nicht für Zeitstempel in
Nachrichtenhüllen). Falls nicht gesetzt, verwendet OpenClaw zur Laufzeit die Zeitzone des Hostsystems.

```json5
{
  agents: { defaults: { userTimezone: "America/Chicago" } }
}
```

<div id="agentsdefaultstimeformat">
  ### `agents.defaults.timeFormat`
</div>

Steuert das **Zeitformat**, das im Abschnitt „Aktuelles Datum &amp; Uhrzeit“ des System-Prompts angezeigt wird.
Standard: `auto` (OS-Voreinstellung).

```json5
{
  agents: { defaults: { timeFormat: "auto" } } // auto | 12 | 24
}
```

<div id="messages">
  ### `messages`
</div>

Steuert eingehende und ausgehende Präfixe sowie optionale Bestätigungsreaktionen.
Siehe [Messages](/de/concepts/messages) für Informationen zu Warteschlangen, Sitzungen und Streaming-Kontext.

```json5
{
  messages: {
    responsePrefix: "🦞", // oder „auto"
    ackReaction: "👀",
    ackReactionScope: "group-mentions",
    removeAckAfterReply: false
  }
}
```

`responsePrefix` wird auf **alle ausgehenden Antworten** (Tool-Zusammenfassungen,
Block-Streaming, finale Antworten) über alle Kanäle hinweg angewendet, sofern es nicht bereits vorhanden ist.

Wenn `messages.responsePrefix` nicht gesetzt ist, wird standardmäßig kein Präfix angewendet. Antworten in WhatsApp-Selbstchats
sind die Ausnahme: Sie verwenden standardmäßig `[{identity.name}]`, wenn gesetzt, sonst
`[openclaw]`, damit Unterhaltungen auf demselben Telefon lesbar bleiben.
Setze es auf `"auto"`, um `[{identity.name}]` für den gerouteten agent abzuleiten (falls gesetzt).

<div id="template-variables">
  #### Template-Variablen
</div>

Der `responsePrefix`-String kann Template-Variablen enthalten, die dynamisch aufgelöst werden:

| Variable          | Beschreibung                   | Beispiel                    |
| ----------------- | ------------------------------ | --------------------------- |
| `{model}`         | Kurzer Modellname              | `claude-opus-4-5`, `gpt-4o` |
| `{modelFull}`     | Vollständiger Modellbezeichner | `anthropic/claude-opus-4-5` |
| `{provider}`      | Anbietername                   | `anthropic`, `openai`       |
| `{thinkingLevel}` | Aktuelles Denkniveau           | `high`, `low`, `off`        |
| `{identity.name}` | Agent-Identitätsname           | (wie im `"auto"`-Modus)     |

Bei Variablen wird nicht zwischen Groß- und Kleinschreibung unterschieden (`{MODEL}` = `{model}`). `{think}` ist ein Alias für `{thinkingLevel}`.
Nicht aufgelöste Variablen bleiben als Literaltext erhalten.

```json5
{
  messages: {
    responsePrefix: "[{model} | think:{thinkingLevel}]"
  }
}
```

Beispielausgabe: `[claude-opus-4-5 | think:high] Here's my response...`

Das WhatsApp-Eingangspräfix wird über `channels.whatsapp.messagePrefix` konfiguriert (deprecated:
`messages.messagePrefix`). Der Standard bleibt **unverändert**: `"[openclaw]"`, wenn
`channels.whatsapp.allowFrom` leer ist, ansonsten `""` (kein Präfix). Bei Verwendung von
`"[openclaw]"` verwendet OpenClaw stattdessen `[{identity.name}]`, wenn der geroutete
agent `identity.name` gesetzt hat.

`ackReaction` sendet nach Best-Effort-Prinzip eine Emoji-Reaktion, um eingehende Nachrichten auf
Kanälen zu bestätigen, die Reaktionen unterstützen (Slack/Discord/Telegram/Google Chat). Standardwert ist das
`identity.emoji` des aktiven agents, falls gesetzt, sonst `"👀"`. Setz den Wert auf `""`, um zu deaktivieren.

`ackReactionScope` steuert, wann Reaktionen ausgelöst werden:

* `group-mentions` (Standard): nur, wenn in einer Gruppe/einem Raum Erwähnungen erforderlich sind **und** der Bot erwähnt wurde
* `group-all`: alle Gruppen-/Raumnachrichten
* `direct`: nur Direktnachrichten
* `all`: alle Nachrichten

`removeAckAfterReply` entfernt die Ack-Reaktion des Bots, nachdem eine Antwort gesendet wurde
(nur Slack/Discord/Telegram/Google Chat). Standard: `false`.

<div id="messagestts">
  #### `messages.tts`
</div>

Aktiviert Text-zu-Sprache für ausgehende Antworten. Wenn aktiviert, erzeugt OpenClaw Audiodaten
mit ElevenLabs oder OpenAI und hängt sie an Antworten an. Telegram verwendet Sprachnachrichten im
Opus-Format, andere Kanäle senden MP3-Audio.

```json5
{
  messages: {
    tts: {
      auto: "always", // off | always | inbound | tagged
      mode: "final", // final | all (include tool/block replies)
      provider: "elevenlabs",
      summaryModel: "openai/gpt-4.1-mini",
      modelOverrides: {
        enabled: true
      },
      maxTextLength: 4000,
      timeoutMs: 30000,
      prefsPath: "~/.openclaw/settings/tts.json",
      elevenlabs: {
        apiKey: "elevenlabs_api_key",
        baseUrl: "https://api.elevenlabs.io",
        voiceId: "voice_id",
        modelId: "eleven_multilingual_v2",
        seed: 42,
        applyTextNormalization: "auto",
        languageCode: "en",
        voiceSettings: {
          stability: 0.5,
          similarityBoost: 0.75,
          style: 0.0,
          useSpeakerBoost: true,
          speed: 1.0
        }
      },
      openai: {
        apiKey: "openai_api_key",
        model: "gpt-4o-mini-tts",
        voice: "alloy"
      }
    }
  }
}
```

Hinweise:

* `messages.tts.auto` steuert Auto‑TTS (`off`, `always`, `inbound`, `tagged`).
* `/tts off|always|inbound|tagged` setzt den automatischen Modus pro Sitzung (überschreibt die Konfiguration).
* `messages.tts.enabled` ist veraltet; doctor migriert es zu `messages.tts.auto`.
* `prefsPath` speichert lokale Überschreibungen (Anbieter/Limit/Zusammenfassung).
* `maxTextLength` ist eine harte Obergrenze für TTS‑Eingaben; Zusammenfassungen werden entsprechend gekürzt.
* `summaryModel` überschreibt `agents.defaults.model.primary` für automatische Zusammenfassungen.
  * Akzeptiert `provider/model` oder einen Alias aus `agents.defaults.models`.
* `modelOverrides` aktiviert modellbasierte Überschreibungen wie `[[tts:...]]`‑Tags (standardmäßig aktiviert).
* `/tts limit` und `/tts summary` steuern die benutzerspezifischen Einstellungen für Zusammenfassungen.
* `apiKey`‑Werte fallen zurück auf `ELEVENLABS_API_KEY`/`XI_API_KEY` und `OPENAI_API_KEY`.
* `elevenlabs.baseUrl` überschreibt die Basis‑URL der ElevenLabs‑API.
* `elevenlabs.voiceSettings` unterstützt `stability`/`similarityBoost`/`style` (0..1),
  `useSpeakerBoost` und `speed` (0,5..2,0).

<div id="talk">
  ### `talk`
</div>

Voreinstellungen für den Talk-Modus (macOS/iOS/Android). Wenn nicht gesetzt, fallen Voice-IDs auf `ELEVENLABS_VOICE_ID` oder `SAG_VOICE_ID` zurück.
`apiKey` fällt, wenn nicht gesetzt, auf `ELEVENLABS_API_KEY` (oder das Shell-Profil des Gateways) zurück.
`voiceAliases` ermöglicht es Talk-Direktiven, benutzerfreundliche Namen zu verwenden (z. B. `"voice":"Clawd"`).

```json5
{
  talk: {
    voiceId: "elevenlabs_voice_id",
    voiceAliases: {
      Clawd: "EXAVITQu4vr4xnSDxMaL",
      Roger: "CwhRBWXzGAHq8TQ4Fs17"
    },
    modelId: "eleven_v3",
    outputFormat: "mp3_44100_128",
    apiKey: "elevenlabs_api_key",
    interruptOnSpeech: true
  }
}
```

<div id="agentsdefaults">
  ### `agents.defaults`
</div>

Steuert die eingebettete agent-Laufzeitumgebung (Modell/Denken/Verbose/Timeouts).
`agents.defaults.models` definiert den konfigurierten Modellkatalog (und dient als Allowlist für `/model`).
`agents.defaults.model.primary` legt das Standardmodell fest; `agents.defaults.model.fallbacks` sind globale Failover-Modelle.
`agents.defaults.imageModel` ist optional und wird **nur verwendet, wenn das Primärmodell keine Bildeingabe unterstützt**.
Jeder Eintrag in `agents.defaults.models` kann enthalten:

* `alias` (optionale Modellabkürzung, z. B. `/opus`).
* `params` (optionale, anbieterspezifische API-Parameter, die an die Modellanfrage durchgereicht werden).

`params` wird auch auf Streaming-Ausführungen (eingebetteter agent + Verdichtung) angewendet. Derzeit unterstützte Schlüssel: `temperature`, `maxTokens`. Diese werden mit Optionen zur Aufrufzeit zusammengeführt; vom Aufrufer übergebene Werte haben Vorrang. `temperature` ist ein fortgeschrittener Einstellparameter – lasse ihn unverändert, es sei denn, du kennst die Standardwerte des Modells und benötigst eine Anpassung.

Beispiel:

```json5
{
  agents: {
    defaults: {
      models: {
        "anthropic/claude-sonnet-4-5-20250929": {
          params: { temperature: 0.6 }
        },
        "openai/gpt-5.2": {
          params: { maxTokens: 8192 }
        }
      }
    }
  }
}
```

Z.AI GLM-4.x-Modelle aktivieren den Thinking-Modus automatisch, außer du:

* setzt `--thinking off`, oder
* definierst `agents.defaults.models["zai/<model>"].params.thinking` selbst.

OpenClaw liefert außerdem einige eingebaute Alias-Kürzel mit. Standardzuweisungen greifen nur, wenn das Modell
bereits in `agents.defaults.models` vorhanden ist:

* `opus` -&gt; `anthropic/claude-opus-4-5`
* `sonnet` -&gt; `anthropic/claude-sonnet-4-5`
* `gpt` -&gt; `openai/gpt-5.2`
* `gpt-mini` -&gt; `openai/gpt-5-mini`
* `gemini` -&gt; `google/gemini-3-pro-preview`
* `gemini-flash` -&gt; `google/gemini-3-flash-preview`

Wenn du denselben Alias-Namen (Groß-/Kleinschreibung wird ignoriert) selbst konfigurierst, gewinnt deine Einstellung (Standardwerte überschreiben sie niemals).

Beispiel: Opus 4.5 als Primärmodell mit MiniMax M2.1 als Fallback (gehostetes MiniMax):

```json5
{
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-5": { alias: "opus" },
        "minimax/MiniMax-M2.1": { alias: "minimax" }
      },
      model: {
        primary: "anthropic/claude-opus-4-5",
        fallbacks: ["minimax/MiniMax-M2.1"]
      }
    }
  }
}
```

MiniMax-Authentifizierung: Setze die Umgebungsvariable `MINIMAX_API_KEY` oder konfiguriere `models.providers.minimax`.

<div id="agentsdefaultsclibackends-cli-fallback">
  #### `agents.defaults.cliBackends` (CLI-Fallback)
</div>

Optionale CLI-Backends für rein textbasierte Fallback-Ausführungen (keine Tool-Aufrufe). Diese sind nützlich als
Fallback-Pfad, wenn API-Anbieter fehlschlagen. Das Durchreichen von Bildern wird unterstützt, wenn du ein
`imageArg` konfigurierst, das Dateipfade akzeptiert.

Hinweise:

* CLI-Backends sind **text-first** ausgelegt; Tools sind immer deaktiviert.
* Sitzungen werden unterstützt, wenn `sessionArg` gesetzt ist; Sitzungs-IDs werden pro Backend dauerhaft gespeichert.
* Für `claude-cli` sind Standardwerte vorkonfiguriert. Passe den Befehls-Pfad an, wenn PATH minimal ist
  (launchd/systemd).

Beispiel:

```json5
{
  agents: {
    defaults: {
      cliBackends: {
        "claude-cli": {
          command: "/opt/homebrew/bin/claude"
        },
        "my-cli": {
          command: "my-cli",
          args: ["--json"],
          output: "json",
          modelArg: "--model",
          sessionArg: "--session",
          sessionMode: "existing",
          systemPromptArg: "--system",
          systemPromptWhen: "first",
          imageArg: "--image",
          imageMode: "repeat"
        }
      }
    }
  }
}
```

```json5
{
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-5": { alias: "Opus" },
        "anthropic/claude-sonnet-4-1": { alias: "Sonnet" },
        "openrouter/deepseek/deepseek-r1:free": {},
        "zai/glm-4.7": {
          alias: "GLM",
          params: {
            thinking: {
              type: "enabled",
              clear_thinking: false
            }
          }
        }
      },
      model: {
        primary: "anthropic/claude-opus-4-5",
        fallbacks: [
          "openrouter/deepseek/deepseek-r1:free",
          "openrouter/meta-llama/llama-3.3-70b-instruct:free"
        ]
      },
      imageModel: {
        primary: "openrouter/qwen/qwen-2.5-vl-72b-instruct:free",
        fallbacks: [
          "openrouter/google/gemini-2.0-flash-vision:free"
        ]
      },
      thinkingDefault: "low",
      verboseDefault: "off",
      elevatedDefault: "on",
      timeoutSeconds: 600,
      mediaMaxMb: 5,
      heartbeat: {
        every: "30m",
        target: "last"
      },
      maxConcurrent: 3,
      subagents: {
        model: "minimax/MiniMax-M2.1",
        maxConcurrent: 1,
        archiveAfterMinutes: 60
      },
      exec: {
        backgroundMs: 10000,
        timeoutSec: 1800,
        cleanupMs: 1800000
      },
      contextTokens: 200000
    }
  }
}
```

<div id="agentsdefaultscontextpruning-tool-result-pruning">
  #### `agents.defaults.contextPruning` (Tool-Ergebnis-Pruning)
</div>

`agents.defaults.contextPruning` entfernt **alte Tool-Ergebnisse** aus dem In-Memory-Kontext unmittelbar bevor eine Anfrage an das LLM gesendet wird.
Es verändert **nicht** den Sitzungsverlauf auf der Festplatte (`*.jsonl` bleibt vollständig).

Dies soll den Token-Verbrauch für gesprächige Agenten reduzieren, die im Laufe der Zeit große Tool-Ausgaben ansammeln.

Überblick:

* Fasst Benutzer-/Assistenten-Nachrichten niemals an.
* Schützt die letzten `keepLastAssistants` Assistenten-Nachrichten (keine Tool-Ergebnisse nach diesem Punkt werden entfernt).
* Schützt das Bootstrap-Präfix (nichts vor der ersten Benutzer-Nachricht wird entfernt).
* Modi:
  * `adaptive`: schneidet übergroße Tool-Ergebnisse weich zurück (Kopf/Ende bleiben erhalten), sobald das geschätzte Kontextverhältnis `softTrimRatio` überschreitet.
    Danach werden die ältesten berechtigten Tool-Ergebnisse hart gelöscht, wenn das geschätzte Kontextverhältnis `hardClearRatio` überschreitet **und**
    genügend entfernbares Tool-Ergebnis-Volumen vorhanden ist (`minPrunableToolChars`).
  * `aggressive`: ersetzt berechtigte Tool-Ergebnisse vor dem Cutoff immer durch `hardClear.placeholder` (keine Verhältnisprüfungen).

Weiches vs. hartes Pruning (was sich im an das LLM gesendeten Kontext ändert):

* **Weiches Trimmen**: nur für *übergroße* Tool-Ergebnisse. Behält den Anfang + das Ende und fügt `...` in der Mitte ein.
  * Vorher: `toolResult("…very long output…")`
  * Nachher: `toolResult("HEAD…\n...\n…TAIL\n\n[Tool result trimmed: …]")`
* **Hartes Löschen**: ersetzt das gesamte Tool-Ergebnis durch den Platzhalter.
  * Vorher: `toolResult("…very long output…")`
  * Nachher: `toolResult("[Old tool result content cleared]")`

Hinweise / aktuelle Einschränkungen:

* Tool-Ergebnisse, die **Image-Blöcke enthalten, werden derzeit übersprungen** (niemals getrimmt/gelöscht).
* Das geschätzte „Kontextverhältnis“ basiert auf **Zeichen** (näherungsweise), nicht auf exakten Tokens.
* Wenn die Sitzung noch nicht mindestens `keepLastAssistants` Assistenten-Nachrichten enthält, wird kein Pruning durchgeführt.
* Im `aggressive`-Modus wird `hardClear.enabled` ignoriert (berechtigte Tool-Ergebnisse werden immer durch `hardClear.placeholder` ersetzt).

Standard (adaptive):

```json5
{
  agents: { defaults: { contextPruning: { mode: "adaptive" } } }
}
```

Zum Deaktivieren:

```json5
{
  agents: { defaults: { contextPruning: { mode: "off" } } }
}
```

Standardwerte (wenn `mode` auf `"adaptive"` oder `"aggressive"` gesetzt ist):

* `keepLastAssistants`: `3`
* `softTrimRatio`: `0.3` (nur im adaptiven Modus)
* `hardClearRatio`: `0.5` (nur im adaptiven Modus)
* `minPrunableToolChars`: `50000` (nur im adaptiven Modus)
* `softTrim`: `{ maxChars: 4000, headChars: 1500, tailChars: 1500 }` (nur im adaptiven Modus)
* `hardClear`: `{ enabled: true, placeholder: "[Old tool result content cleared]" }`

Beispiel (aggressiv, minimal):

```json5
{
  agents: { defaults: { contextPruning: { mode: "aggressive" } } }
}
```

Beispiel (adaptiv feinabgestimmt):

```json5
{
  agents: {
    defaults: {
      contextPruning: {
        mode: "adaptive",
        keepLastAssistants: 3,
        softTrimRatio: 0.3,
        hardClearRatio: 0.5,
        minPrunableToolChars: 50000,
        softTrim: { maxChars: 4000, headChars: 1500, tailChars: 1500 },
        hardClear: { enabled: true, placeholder: "[Old tool result content cleared]" },
        // Optional: Pruning auf bestimmte Tools beschränken (deny hat Vorrang; unterstützt "*"-Wildcards)
        tools: { deny: ["browser", "canvas"] },
      }
    }
  }
}
```

Details zum Verhalten findest du unter [/concepts/session-pruning](/de/concepts/session-pruning).

<div id="agentsdefaultscompaction-reserve-headroom-memory-flush">
  #### `agents.defaults.compaction` (Headroom reservieren + Memory Flush)
</div>

`agents.defaults.compaction.mode` wählt die Kompaktierungs- und Zusammenfassungsstrategie. Standard ist `default`; setze `safeguard`, um für sehr lange Verläufe eine chunk-basierte Zusammenfassung zu aktivieren. Siehe [/concepts/compaction](/de/concepts/compaction).

`agents.defaults.compaction.reserveTokensFloor` erzwingt einen minimalen `reserveTokens`-
Wert für Pi-Kompaktierung (Standard: `20000`). Setze den Wert auf `0`, um diese Untergrenze zu deaktivieren.

`agents.defaults.compaction.memoryFlush` führt einen **stillen** agentischen Turn aus,
bevor die automatische Kompaktierung läuft, und weist das Modell an, dauerhafte Erinnerungen auf die Festplatte zu schreiben (z. B.
`memory/YYYY-MM-DD.md`). Er wird ausgelöst, wenn die Schätzung der Sitzungs-Token eine
weiche Schwelle knapp unterhalb des Kompaktierungs-Limits überschreitet.

Legacy-Standardwerte:

* `memoryFlush.enabled`: `true`
* `memoryFlush.softThresholdTokens`: `4000`
* `memoryFlush.prompt` / `memoryFlush.systemPrompt`: eingebaute Standardwerte mit `NO_REPLY`
* Hinweis: Memory Flush wird übersprungen, wenn der Sitzungsarbeitsbereich schreibgeschützt ist
  (`agents.defaults.sandbox.workspaceAccess: "ro"` oder `"none"`).

Beispiel (optimiert):

```json5
{
  agents: {
    defaults: {
      compaction: {
        mode: "safeguard",
        reserveTokensFloor: 24000,
        memoryFlush: {
          enabled: true,
          softThresholdTokens: 6000,
          systemPrompt: "Session nearing compaction. Store durable memories now.",
          prompt: "Schreibe alle dauerhaften Notizen nach memory/YYYY-MM-DD.md; antworte mit NO_REPLY, wenn nichts zu speichern ist."
        }
      }
    }
  }
}
```

Block-Streaming:

* `agents.defaults.blockStreamingDefault`: `"on"`/`"off"` (Standard: off).
* Channel-Overrides: `*.blockStreaming` (und Varianten pro Account), um Block-Streaming an/aus zu erzwingen.
  Nicht-Telegram-Kanäle benötigen ein explizites `*.blockStreaming: true`, um Blockantworten zu aktivieren.
* `agents.defaults.blockStreamingBreak`: `"text_end"` oder `"message_end"` (Standard: text&#95;end).
* `agents.defaults.blockStreamingChunk`: weiches Chunking für gestreamte Blöcke. Standard sind
  800–1200 Zeichen, bevorzugt Absatzumbrüche (`\n\n`), dann Zeilenumbrüche, dann Sätze.
  Beispiel:
  ```json5
  {
    agents: { defaults: { blockStreamingChunk: { minChars: 800, maxChars: 1200 } } }
  }
  ```
* `agents.defaults.blockStreamingCoalesce`: gestreamte Blöcke vor dem Senden zusammenführen.
  Standard ist `{ idleMs: 1000 }` und übernimmt `minChars` von `blockStreamingChunk`,
  wobei `maxChars` auf das Textlimit des Channels begrenzt wird. Signal/Slack/Discord/Google Chat verwenden standardmäßig
  `minChars: 1500`, sofern nicht überschrieben.
  Channel-Overrides: `channels.whatsapp.blockStreamingCoalesce`, `channels.telegram.blockStreamingCoalesce`,
  `channels.discord.blockStreamingCoalesce`, `channels.slack.blockStreamingCoalesce`, `channels.mattermost.blockStreamingCoalesce`,
  `channels.signal.blockStreamingCoalesce`, `channels.imessage.blockStreamingCoalesce`, `channels.msteams.blockStreamingCoalesce`,
  `channels.googlechat.blockStreamingCoalesce`
  (und Varianten pro Account).
* `agents.defaults.humanDelay`: zufällige Pause zwischen **Blockantworten** nach der ersten.
  Modi: `off` (Standard), `natural` (800–2500ms), `custom` (verwendet `minMs`/`maxMs`).
  Override pro agent: `agents.list[].humanDelay`.
  Beispiel:
  ```json5
  {
    agents: { defaults: { humanDelay: { mode: "natural" } } }
  }
  ```

Siehe [/concepts/streaming](/de/concepts/streaming) für Details zu Verhalten und Chunking.

Tippindikatoren:

* `agents.defaults.typingMode`: `"never" | "instant" | "thinking" | "message"`. Standard ist
  `instant` für Direktchats/Erwähnungen und `message` für nicht erwähnte Gruppenchats.
* `session.typingMode`: sitzungsspezifisches Override für den Modus.
* `agents.defaults.typingIntervalSeconds`: wie häufig das Tipp-Signal aktualisiert wird (Standard: 6s).
* `session.typingIntervalSeconds`: sitzungsspezifisches Override für das Aktualisierungsintervall.
  Siehe [/concepts/typing-indicators](/de/concepts/typing-indicators) für Verhaltensdetails.

`agents.defaults.model.primary` sollte als `provider/model` gesetzt werden (z. B. `anthropic/claude-opus-4-5`).
Aliasse kommen aus `agents.defaults.models.*.alias` (z. B. `Opus`).
Wenn du den Provider weglässt, nimmt OpenClaw derzeit `anthropic` als temporären
Deprecation-Fallback an.
Z.AI-Modelle sind als `zai/<model>` verfügbar (z. B. `zai/glm-4.7`) und erfordern
`ZAI_API_KEY` (oder legacy `Z_AI_API_KEY`) in der Umgebung.

`agents.defaults.heartbeat` konfiguriert periodische Herzschlag-Läufe:

* `every`: Dauer-String (`ms`, `s`, `m`, `h`); Standard-Einheit Minuten. Standard:
  `30m`. Setze `0m`, um zu deaktivieren.
* `model`: optionales Override-Modell für Herzschlag-Läufe (`provider/model`).
* `includeReasoning`: wenn `true`, liefern Herzschläge zusätzlich die separate `Reasoning:`-Nachricht, sofern verfügbar (gleiche Form wie `/reasoning on`). Standard: `false`.
* `session`: optionaler Sitzungsschlüssel, um zu steuern, in welcher Sitzung der Herzschlag ausgeführt wird. Standard: `main`.
* `to`: optionales Empfänger-Override (kanalspezifische ID, z. B. E.164 für WhatsApp, Chat-ID für Telegram).
* `target`: optionaler Auslieferungskanal (`last`, `whatsapp`, `telegram`, `discord`, `slack`, `msteams`, `signal`, `imessage`, `none`). Standard: `last`.
* `prompt`: optionales Override für den Herzschlag-Inhalt (Standard: `Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.`). Overrides werden unverändert gesendet; füge eine `Read HEARTBEAT.md`-Zeile ein, wenn die Datei weiterhin gelesen werden soll.
* `ackMaxChars`: maximale Anzahl Zeichen nach `HEARTBEAT_OK` vor der Auslieferung (Standard: 300).

Herzschläge pro Agent:

* Setze `agents.list[].heartbeat`, um Herzschlag-Einstellungen für einen bestimmten Agenten zu aktivieren oder zu überschreiben.
* Wenn irgendein Agent-Eintrag `heartbeat` definiert, führen **nur diese Agenten** Herzschläge aus; Defaults
  werden zur gemeinsamen Basis für diese Agenten.

Herzschläge führen vollständige Agent-Turns aus. Kürzere Intervalle verbrauchen mehr Tokens; achte auf
`every`, halte `HEARTBEAT.md` klein und/oder wähle ein günstigeres `model`.

`tools.exec` konfiguriert Hintergrund-Exec-Defaults:

* `backgroundMs`: Zeit bis zum automatischen Verschieben in den Hintergrund (ms, Standard 10000)
* `timeoutSec`: Auto-Kill nach dieser Laufzeit (Sekunden, Standard 1800)
* `cleanupMs`: wie lange abgeschlossene Sitzungen im Speicher gehalten werden (ms, Standard 1800000)
* `notifyOnExit`: System-Event einreihen + Herzschlag anfordern, wenn ein im Hintergrund laufendes Exec beendet wird (Standard true)
* `applyPatch.enabled`: experimentelles `apply_patch` aktivieren (nur OpenAI/OpenAI Codex; Standard false)
* `applyPatch.allowModels`: optionale Allowlist von Modell-IDs (z. B. `gpt-5.2` oder `openai/gpt-5.2`)
  Hinweis: `applyPatch` befindet sich nur unter `tools.exec`.

`tools.web` konfiguriert Websuche + Fetch-Tools:

* `tools.web.search.enabled` (Standard: true, wenn Key vorhanden ist)
* `tools.web.search.apiKey` (empfohlen: setzen über `openclaw configure --section web` oder `BRAVE_API_KEY`-Env-Var verwenden)
* `tools.web.search.maxResults` (1–10, Standard 5)
* `tools.web.search.timeoutSeconds` (Standard 30)
* `tools.web.search.cacheTtlMinutes` (Standard 15)
* `tools.web.fetch.enabled` (Standard true)
* `tools.web.fetch.maxChars` (Standard 50000)
* `tools.web.fetch.timeoutSeconds` (Standard 30)
* `tools.web.fetch.cacheTtlMinutes` (Standard 15)
* `tools.web.fetch.userAgent` (optionalem Override)
* `tools.web.fetch.readability` (Standard true; deaktiviere, um nur grundlegendes HTML-Cleanup zu verwenden)
* `tools.web.fetch.firecrawl.enabled` (Standard true, wenn ein API-Key gesetzt ist)
* `tools.web.fetch.firecrawl.apiKey` (optional; Standard ist `FIRECRAWL_API_KEY`)
* `tools.web.fetch.firecrawl.baseUrl` (Standard https://api.firecrawl.dev)
* `tools.web.fetch.firecrawl.onlyMainContent` (Standard true)
* `tools.web.fetch.firecrawl.maxAgeMs` (optional)
* `tools.web.fetch.firecrawl.timeoutSeconds` (optional)

`tools.media` konfiguriert das Verstehen eingehender Medien (Bild/Audio/Video):

* `tools.media.models`: gemeinsame Modellliste (mit Fähigkeits-Tags; wird nach den fähigkeitsspezifischen Listen verwendet).
* `tools.media.concurrency`: maximale gleichzeitige Capability-Ausführungen (Standard 2).
* `tools.media.image` / `tools.media.audio` / `tools.media.video`:
  * `enabled`: Opt-out-Schalter (Standard `true`, wenn Modelle konfiguriert sind).
  * `prompt`: optionale Prompt-Überschreibung (für Bild/Video wird automatisch ein `maxChars`-Hinweis angehängt).
  * `maxChars`: maximale Anzahl Ausgabezeichen (Standard 500 für Bild/Video; nicht gesetzt für Audio).
  * `maxBytes`: maximale zu sendende Mediengröße (Standards: Bild 10MB, Audio 20MB, Video 50MB).
  * `timeoutSeconds`: Anfrage-Timeout (Standards: Bild 60s, Audio 60s, Video 120s).
  * `language`: optionaler Audio-Hinweis.
  * `attachments`: Attachment-Richtlinie (`mode`, `maxAttachments`, `prefer`).
  * `scope`: optionales Gating (first match wins) mit `match.channel`, `match.chatType` oder `match.keyPrefix`.
  * `models`: geordnete Liste von Modelleinträgen; Fehler oder zu große Medien führen zum Fallback auf den nächsten Eintrag.
* Jeder Eintrag in `models[]`:
  * anbieter-Eintrag (`type: "provider"` oder ausgelassen):
    * `provider`: API-Anbieter-ID (`openai`, `anthropic`, `google`/`gemini`, `groq`, usw.).
    * `model`: Modell-ID-Überschreibung (erforderlich für Bild; Standard ist `gpt-4o-mini-transcribe`/`whisper-large-v3-turbo` für Audio-Anbieter und `gemini-3-flash-preview` für Video).
    * `profile` / `preferredProfile`: Auswahl des Auth-Profils.
  * CLI-Eintrag (`type: "cli"`):
    * `command`: auszuführendes Programm.
    * `args`: vorlagenbasierte Argumente (unterstützt `{{MediaPath}}`, `{{Prompt}}`, `{{MaxChars}}`, usw.).
  * `capabilities`: optionale Liste (`image`, `audio`, `video`), um einen gemeinsamen Eintrag zu begrenzen. Standard, wenn ausgelassen: `openai`/`anthropic`/`minimax` → image, `google` → image+audio+video, `groq` → audio.
  * `prompt`, `maxChars`, `maxBytes`, `timeoutSeconds`, `language` können pro Eintrag überschrieben werden.

Wenn keine Modelle konfiguriert sind (oder `enabled: false`), wird die Verarbeitung übersprungen; das Modell erhält trotzdem die ursprünglichen Attachments.

Die Anbieter-Authentifizierung folgt der Standardreihenfolge für die Modell-Authentifizierung (Auth-Profile, Umgebungsvariablen wie `OPENAI_API_KEY`/`GROQ_API_KEY`/`GEMINI_API_KEY` oder `models.providers.*.apiKey`).

Beispiel:

```json5
{
  tools: {
    media: {
      audio: {
        enabled: true,
        maxBytes: 20971520,
        scope: {
          default: "deny",
          rules: [{ action: "allow", match: { chatType: "direct" } }]
        },
        models: [
          { provider: "openai", model: "gpt-4o-mini-transcribe" },
          { type: "cli", command: "whisper", args: ["--model", "base", "{{MediaPath}}"] }
        ]
      },
      video: {
        enabled: true,
        maxBytes: 52428800,
        models: [{ provider: "google", model: "gemini-3-flash-preview" }]
      }
    }
  }
}
```

`agents.defaults.subagents` konfiguriert Standardwerte für Sub-Agenten:

* `model`: Standardmodell für erzeugte Sub-Agenten (String oder `{ primary, fallbacks }`). Falls ausgelassen, erben Sub-Agenten das Modell des Aufrufers, es sei denn, es wird pro Agent oder pro Aufruf überschrieben.
* `maxConcurrent`: maximale Anzahl gleichzeitiger Sub-Agent-Ausführungen (Standard 1)
* `archiveAfterMinutes`: automatische Archivierung von Sub-Agent-Sitzungen nach N Minuten (Standard 60; `0` zum Deaktivieren setzen)
* Tool-Richtlinie pro Sub-Agent: `tools.subagents.tools.allow` / `tools.subagents.tools.deny` (deny gewinnt)

`tools.profile` setzt eine **Basis-Tool-Allowlist** vor `tools.allow`/`tools.deny`:

* `minimal`: nur `session_status`
* `coding`: `group:fs`, `group:runtime`, `group:sessions`, `group:memory`, `image`
* `messaging`: `group:messaging`, `sessions_list`, `sessions_history`, `sessions_send`, `session_status`
* `full`: keine Einschränkung (entspricht nicht gesetzt)

Agent-spezifische Überschreibung: `agents.list[].tools.profile`.

Beispiel (standardmäßig nur Messaging, zusätzlich auch Slack- und Discord-Tools erlauben):

```json5
{
  tools: {
    profile: "messaging",
    allow: ["slack", "discord"]
  }
}
```

Beispiel (Coding-Profil, aber `exec`/`process` überall unterbinden):

```json5
{
  tools: {
    profile: "coding",
    deny: ["group:runtime"]
  }
}
```

`tools.byProvider` ermöglicht es dir, Tools für bestimmte Anbieter (oder ein einzelnes `provider/model`) **weiter einzuschränken**.
Per-Agent-Überschreibung: `agents.list[].tools.byProvider`.

Reihenfolge: Basisprofil → Anbieterprofil → Allow-/Deny-Richtlinien.
Provider-Schlüssel akzeptieren entweder `provider` (z. B. `google-antigravity`) oder `provider/model`
(z. B. `openai/gpt-5.2`).

Beispiel (globales Coding-Profil beibehalten, aber minimale Tool-Auswahl für Google Antigravity):

```json5
{
  tools: {
    profile: "coding",
    byProvider: {
      "google-antigravity": { profile: "minimal" }
    }
  }
}
```

Beispiel (Anbieter-/modellspezifische Allowlist):

```json5
{
  tools: {
    allow: ["group:fs", "group:runtime", "sessions_list"],
    byProvider: {
      "openai/gpt-5.2": { allow: ["group:fs", "sessions_list"] }
    }
  }
}
```

`tools.allow` / `tools.deny` konfigurieren eine globale Allow/Deny-Policy für Tools (deny gewinnt).
Die Übereinstimmung ist nicht groß-/kleinschreibungssensitiv und unterstützt `*`-Wildcards (`"*"` bedeutet alle Tools).
Dies wird auch angewendet, wenn die Docker-Sandbox **off** ist.

Beispiel (Browser/Canvas überall deaktivieren):

```json5
{
  tools: { deny: ["browser", "canvas"] }
}
```

Werkzeuggruppen (Kurzformen) funktionieren in **globalen** und **agent-spezifischen** Tool-Richtlinien:

* `group:runtime`: `exec`, `bash`, `process`
* `group:fs`: `read`, `write`, `edit`, `apply_patch`
* `group:sessions`: `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status`
* `group:memory`: `memory_search`, `memory_get`
* `group:web`: `web_search`, `web_fetch`
* `group:ui`: `browser`, `canvas`
* `group:automation`: `cron`, `gateway`
* `group:messaging`: `message`
* `group:nodes`: `nodes`
* `group:openclaw`: alle integrierten OpenClaw-Tools (schließt anbieter-Plugins aus)

`tools.elevated` regelt erhöhten Exec-Zugriff auf dem Host:

* `enabled`: den erhöhten Modus erlauben (Standard: true)
* `allowFrom`: Allowlists pro Kanal (leer = deaktiviert)
  * `whatsapp`: E.164-Nummern
  * `telegram`: Chat-IDs oder Benutzernamen
  * `discord`: User-IDs oder Benutzernamen (fällt auf `channels.discord.dm.allowFrom` zurück, falls weggelassen)
  * `signal`: E.164-Nummern
  * `imessage`: Handles/Chat-IDs
  * `webchat`: Sitzungs-IDs oder Benutzernamen

Beispiel:

```json5
{
  tools: {
    elevated: {
      enabled: true,
      allowFrom: {
        whatsapp: ["+15555550123"],
        discord: ["steipete", "1234567890123"]
      }
    }
  }
}
```

Agent-spezifische Überschreibung (zusätzlich einschränken):

```json5
{
  agents: {
    list: [
      {
        id: "family",
        tools: {
          elevated: { enabled: false }
        }
      }
    ]
  }
}
```

Hinweise:

* `tools.elevated` ist der globale Basiswert. `agents.list[].tools.elevated` kann nur weiter einschränken (beide müssen es erlauben).
* `/elevated on|off|ask|full` speichert den Zustand pro Sitzungsschlüssel; Inline-Direktiven gelten für eine einzelne Nachricht.
* Elevated-`exec` läuft auf dem Host und umgeht die Sandbox.
* Die Tool-Richtlinie gilt weiterhin; wenn `exec` verweigert wird, kann „elevated“ nicht verwendet werden.

`agents.defaults.maxConcurrent` legt die maximale Anzahl eingebetteter Agent-Ausführungen fest, die
parallel über Sitzungen hinweg ausgeführt werden können. Jede Sitzung wird weiterhin serialisiert (ein Lauf
pro Sitzungsschlüssel gleichzeitig). Standardwert: 1.

<div id="agentsdefaultssandbox">
  ### `agents.defaults.sandbox`
</div>

Optionale **Docker-Sandboxing** für den eingebetteten Agenten. Gedacht für Nicht-Hauptsitzungen, damit sie nicht auf dein Hostsystem zugreifen können.

Details: [Sandboxing](/de/gateway/sandboxing)

Standardwerte (falls aktiviert):

* scope: `"agent"` (ein Container + Arbeitsbereich pro Agent)
* Debian bookworm-slim-basiertes Image
* Zugriff des Agents auf den Arbeitsbereich: `workspaceAccess: "none"` (Standard)
  * `"none"`: verwende einen Sandbox-Arbeitsbereich pro Scope unter `~/.openclaw/sandboxes`
* `"ro"`: belasse den Sandbox-Arbeitsbereich unter `/workspace` und mounte den Agent-Arbeitsbereich read-only unter `/agent` (deaktiviert `write`/`edit`/`apply_patch`)
  * `"rw"`: mounte den Agent-Arbeitsbereich read/write unter `/workspace`
* automatisches Pruning: Leerlauf &gt; 24h ODER Alter &gt; 7d
* Tool-Richtlinie: erlaube nur `exec`, `process`, `read`, `write`, `edit`, `apply_patch`, `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status` (deny hat Vorrang)
  * konfiguriere über `tools.sandbox.tools`, überschreibe sie pro Agent via `agents.list[].tools.sandbox.tools`
  * Tool-Gruppen-Kürzel, die in der Sandbox-Policy unterstützt werden: `group:runtime`, `group:fs`, `group:sessions`, `group:memory` (siehe [Sandbox vs Tool Policy vs Elevated](/de/gateway/sandbox-vs-tool-policy-vs-elevated#tool-groups-shorthands))
* optionale sandboxed Browser-Umgebung (Chromium + CDP, noVNC-Observer)
* Hardening-Optionen: `network`, `user`, `pidsLimit`, `memory`, `cpus`, `ulimits`, `seccompProfile`, `apparmorProfile`

Warnung: `scope: "shared"` bedeutet einen geteilten Container und einen geteilten Arbeitsbereich. Keine
Isolation zwischen Sitzungen. Verwende `scope: "session"` für Isolation pro Sitzung.

Legacy: `perSession` wird weiterhin unterstützt (`true` → `scope: "session"`,
`false` → `scope: "shared"`).

`setupCommand` läuft **einmal**, nachdem der Container erstellt wurde (im Container via `sh -lc`).
Stelle für Paketinstallationen sicher, dass ausgehender Netzwerkverkehr möglich ist, das Root-FS schreibbar ist und ein Root-User vorhanden ist.

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main", // off | non-main | all
        scope: "agent", // session | agent | shared (agent is default)
        workspaceAccess: "none", // none | ro | rw
        workspaceRoot: "~/.openclaw/sandboxes",
        docker: {
          image: "openclaw-sandbox:bookworm-slim",
          containerPrefix: "openclaw-sbx-",
          workdir: "/workspace",
          readOnlyRoot: true,
          tmpfs: ["/tmp", "/var/tmp", "/run"],
          network: "none",
          user: "1000:1000",
          capDrop: ["ALL"],
          env: { LANG: "C.UTF-8" },
          setupCommand: "apt-get update && apt-get install -y git curl jq",
          // Per-agent override (multi-agent): agents.list[].sandbox.docker.*
          pidsLimit: 256,
          memory: "1g",
          memorySwap: "2g",
          cpus: 1,
          ulimits: {
            nofile: { soft: 1024, hard: 2048 },
            nproc: 256
          },
          seccompProfile: "/path/to/seccomp.json",
          apparmorProfile: "openclaw-sandbox",
          dns: ["1.1.1.1", "8.8.8.8"],
          extraHosts: ["internal.service:10.0.0.5"],
          binds: ["/var/run/docker.sock:/var/run/docker.sock", "/home/user/source:/source:rw"]
        },
        browser: {
          enabled: false,
          image: "openclaw-sandbox-browser:bookworm-slim",
          containerPrefix: "openclaw-sbx-browser-",
          cdpPort: 9222,
          vncPort: 5900,
          noVncPort: 6080,
          headless: false,
          enableNoVnc: true,
          allowHostControl: false,
          allowedControlUrls: ["http://10.0.0.42:18791"],
          allowedControlHosts: ["browser.lab.local", "10.0.0.42"],
          allowedControlPorts: [18791],
          autoStart: true,
          autoStartTimeoutMs: 12000
        },
        prune: {
          idleHours: 24,  // 0 deaktiviert Leerlauf-Bereinigung
          maxAgeDays: 7   // 0 deaktiviert Maximalalter-Bereinigung
        }
      }
    }
  },
  tools: {
    sandbox: {
      tools: {
        allow: ["exec", "process", "read", "write", "edit", "apply_patch", "sessions_list", "sessions_history", "sessions_send", "sessions_spawn", "session_status"],
        deny: ["browser", "canvas", "nodes", "cron", "discord", "gateway"]
      }
    }
  }
}
```

Baue das Standard-sandbox-Image einmalig mit:

```bash
scripts/sandbox-setup.sh
```

Hinweis: sandbox-Container verwenden standardmäßig `network: "none"`; setze `agents.defaults.sandbox.docker.network`
auf `"bridge"` (oder dein benutzerdefiniertes Netzwerk), wenn der agent ausgehenden Netzwerkzugriff benötigt.

Hinweis: eingehende Anhänge werden im aktiven arbeitsbereich unter `media/inbound/*` abgelegt. Mit `workspaceAccess: "rw"` bedeutet das, dass Dateien in den agent-arbeitsbereich geschrieben werden.

Hinweis: `docker.binds` bindet zusätzliche Host-Verzeichnisse ein; globale und per-agent-Binds werden zusammengeführt.

Erstelle das optionale Browser-Image mit:

```bash
scripts/sandbox-browser-setup.sh
```

Wenn `agents.defaults.sandbox.browser.enabled=true` ist, verwendet das Browser-Tool eine
Chromium-Instanz in einer sandbox (CDP). Wenn noVNC aktiviert ist (Standard, wenn headless=false),
wird die noVNC-URL in den System-Prompt injiziert, damit der agent darauf verweisen kann.
Dies erfordert kein `browser.enabled` in der Hauptkonfiguration; die sandbox-Steuerungs-URL
wird pro Sitzung injiziert.

`agents.defaults.sandbox.browser.allowHostControl` (Standard: false) erlaubt
Sitzungen in der sandbox, den **Host**-Browser-Steuerungsserver explizit über das
Browser-Tool (`target: "host"`) anzusprechen. Lassen Sie dies deaktiviert, wenn Sie
strikte sandbox-Isolation möchten.

Allowlists für Fernsteuerung:

* `allowedControlUrls`: exakte Steuerungs-URLs, die für `target: "custom"` erlaubt sind.
* `allowedControlHosts`: erlaubte Hostnames (nur Hostname, kein Port).
* `allowedControlPorts`: erlaubte Ports (Standard: http=80, https=443).
  Standard: Alle Allowlists sind nicht gesetzt (keine Einschränkung). `allowHostControl` ist standardmäßig false.

<div id="models-custom-providers-base-urls">
  ### `models` (benutzerdefinierte Anbieter + Basis-URLs)
</div>

OpenClaw verwendet den **pi-coding-agent**-Modellkatalog. Du kannst benutzerdefinierte Anbieter
(LiteLLM, lokale OpenAI-kompatible Server, Anthropic-Proxys usw.) hinzufügen, indem du
`~/.openclaw/agents/<agentId>/agent/models.json` erstellst oder indem du dasselbe Schema in deiner
OpenClaw-Konfiguration unter `models.providers` definierst.
Anbieterübersicht + Beispiele: [/concepts/model-providers](/de/concepts/model-providers).

Wenn `models.providers` vorhanden ist, schreibt und merged OpenClaw beim Start eine `models.json` nach
`~/.openclaw/agents/<agentId>/agent/`:

* Standardverhalten: **merge** (behält vorhandene Anbieter, überschreibt anhand des Namens)
* setze `models.mode: "replace"`, um den Dateiinhalt zu überschreiben

Wähle das Modell über `agents.defaults.model.primary` (Anbieter/Modell) aus.

```json5
{
  agents: {
    defaults: {
      model: { primary: "custom-proxy/llama-3.1-8b" },
      models: {
        "custom-proxy/llama-3.1-8b": {}
      }
    }
  },
  models: {
    mode: "merge",
    providers: {
      "custom-proxy": {
        baseUrl: "http://localhost:4000/v1",
        apiKey: "LITELLM_KEY",
        api: "openai-completions",
        models: [
          {
            id: "llama-3.1-8b",
            name: "Llama 3.1 8B",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 128000,
            maxTokens: 32000
          }
        ]
      }
    }
  }
}
```

<div id="opencode-zen-multi-model-proxy">
  ### OpenCode Zen (Multi-Modell-Proxy)
</div>

OpenCode Zen ist ein Multi-Modell-Gateway mit Endpunkten pro Modell. OpenClaw verwendet
den integrierten `opencode`-Anbieter von pi-ai; setze `OPENCODE_API_KEY` (oder
`OPENCODE_ZEN_API_KEY`) unter https://opencode.ai/auth.

Hinweise:

* Modellreferenzen verwenden `opencode/<modelId>` (Beispiel: `opencode/claude-opus-4-5`).
* Wenn du eine Allowlist über `agents.defaults.models` aktivierst, füge jedes Modell hinzu, das du verwenden willst.
* Kurzbefehl: `openclaw onboard --auth-choice opencode-zen`.

```json5
{
  agents: {
    defaults: {
      model: { primary: "opencode/claude-opus-4-5" },
      models: { "opencode/claude-opus-4-5": { alias: "Opus" } }
    }
  }
}
```

<div id="zai-glm-47-provider-alias-support">
  ### Z.AI (GLM-4.7) — Unterstützung für Anbieter-Aliasnamen
</div>

Z.AI-Modelle sind über den integrierten Anbieter `zai` verfügbar. Setze `ZAI_API_KEY`
in deiner Umgebung und gib das Modell als provider/model an.

Shortcut: `openclaw onboard --auth-choice zai-api-key`.

```json5
{
  agents: {
    defaults: {
      model: { primary: "zai/glm-4.7" },
      models: { "zai/glm-4.7": {} }
    }
  }
}
```

Hinweise:

* `z.ai/*` und `z-ai/*` sind zulässige Aliasse und werden auf `zai/*` normalisiert.
* Wenn `ZAI_API_KEY` fehlt, schlagen Anfragen an `zai/*` zur Laufzeit mit einem Authentifizierungsfehler fehl.
* Beispiel-Fehlermeldung: `No API key found for provider "zai".`
* Der allgemeine API-Endpunkt von Z.AI ist `https://api.z.ai/api/paas/v4`. GLM-Coding-
  Anfragen verwenden den dedizierten Coding-Endpunkt `https://api.z.ai/api/coding/paas/v4`.
  Der integrierte `zai`-anbieter verwendet den Coding-Endpunkt. Wenn du den allgemeinen
  Endpunkt benötigst, definiere einen benutzerdefinierten anbieter in `models.providers`
  mit einem Override der Basis-URL (siehe den Abschnitt zu benutzerdefinierten anbietern oben).
* Verwende in Dokumentation und Konfigurationen Platzhalter; committe niemals echte API-Schlüssel.

<div id="moonshot-ai-kimi">
  ### Moonshot AI (Kimi)
</div>

Verwende den OpenAI-kompatiblen Endpoint von Moonshot:

```json5
{
  env: { MOONSHOT_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "moonshot/kimi-k2.5" },
      models: { "moonshot/kimi-k2.5": { alias: "Kimi K2.5" } }
    }
  },
  models: {
    mode: "merge",
    providers: {
      moonshot: {
        baseUrl: "https://api.moonshot.ai/v1",
        apiKey: "${MOONSHOT_API_KEY}",
        api: "openai-completions",
        models: [
          {
            id: "kimi-k2.5",
            name: "Kimi K2.5",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 256000,
            maxTokens: 8192
          }
        ]
      }
    }
  }
}
```

Hinweise:

* Setze die Umgebungsvariable `MOONSHOT_API_KEY` oder verwende `openclaw onboard --auth-choice moonshot-api-key`.
* Modellreferenz: `moonshot/kimi-k2.5`.
* Verwende `https://api.moonshot.cn/v1`, wenn du den China-Endpunkt benötigst.

<div id="kimi-code">
  ### Kimi Code
</div>

Verwende den eigenen, OpenAI-kompatiblen Endpunkt von Kimi Code (separat von Moonshot):

```json5
{
  env: { KIMICODE_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "kimi-code/kimi-for-coding" },
      models: { "kimi-code/kimi-for-coding": { alias: "Kimi Code" } }
    }
  },
  models: {
    mode: "merge",
    providers: {
      "kimi-code": {
        baseUrl: "https://api.kimi.com/coding/v1",
        apiKey: "${KIMICODE_API_KEY}",
        api: "openai-completions",
        models: [
          {
            id: "kimi-for-coding",
            name: "Kimi For Coding",
            reasoning: true,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 262144,
            maxTokens: 32768,
            headers: { "User-Agent": "KimiCLI/0.77" },
            compat: { supportsDeveloperRole: false }
          }
        ]
      }
    }
  }
}
```

Hinweise:

* Setze `KIMICODE_API_KEY` als Umgebungsvariable oder führe `openclaw onboard --auth-choice kimi-code-api-key` aus.
* Modellreferenz: `kimi-code/kimi-for-coding`.

<div id="synthetic-anthropic-compatible">
  ### Synthetic (Anthropic-kompatibel)
</div>

Verwende den Anthropic-kompatiblen Endpunkt von Synthetic:

```json5
{
  env: { SYNTHETIC_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "synthetic/hf:MiniMaxAI/MiniMax-M2.1" },
      models: { "synthetic/hf:MiniMaxAI/MiniMax-M2.1": { alias: "MiniMax M2.1" } }
    }
  },
  models: {
    mode: "merge",
    providers: {
      synthetic: {
        baseUrl: "https://api.synthetic.new/anthropic",
        apiKey: "${SYNTHETIC_API_KEY}",
        api: "anthropic-messages",
        models: [
          {
            id: "hf:MiniMaxAI/MiniMax-M2.1",
            name: "MiniMax M2.1",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 192000,
            maxTokens: 65536
          }
        ]
      }
    }
  }
}
```

Hinweise:

* Setze `SYNTHETIC_API_KEY` oder verwende `openclaw onboard --auth-choice synthetic-api-key`.
* Modellreferenz: `synthetic/hf:MiniMaxAI/MiniMax-M2.1`.
* Die Basis-URL sollte `/v1` nicht enthalten, da der Anthropic-Client es automatisch anhängt.

<div id="local-models-lm-studio-recommended-setup">
  ### Lokale Modelle (LM Studio) — empfohlenes Setup
</div>

Siehe [/gateway/local-models](/de/gateway/local-models) für die aktuellen Hinweise zur lokalen Nutzung. TL;DR: MiniMax M2.1 über die LM Studio Responses API auf leistungsfähiger Hardware ausführen; gehostete Modelle konsolidiert als Fallback vorhalten.

<div id="minimax-m21">
  ### MiniMax M2.1
</div>

Verwenden Sie MiniMax M2.1 direkt, ohne LM Studio:

```json5
{
  agent: {
    model: { primary: "minimax/MiniMax-M2.1" },
    models: {
      "anthropic/claude-opus-4-5": { alias: "Opus" },
      "minimax/MiniMax-M2.1": { alias: "Minimax" }
    }
  },
  models: {
    mode: "merge",
    providers: {
      minimax: {
        baseUrl: "https://api.minimax.io/anthropic",
        apiKey: "${MINIMAX_API_KEY}",
        api: "anthropic-messages",
        models: [
          {
            id: "MiniMax-M2.1",
            name: "MiniMax M2.1",
            reasoning: false,
            input: ["text"],
            // Preisgestaltung: Aktualisieren Sie models.json, falls Sie eine exakte Kostenverfolgung benötigen.
            cost: { input: 15, output: 60, cacheRead: 2, cacheWrite: 10 },
            contextWindow: 200000,
            maxTokens: 8192
          }
        ]
      }
    }
  }
}
```

Hinweise:

* Setze die Umgebungsvariable `MINIMAX_API_KEY` oder führe `openclaw onboard --auth-choice minimax-api` aus.
* Verfügbares Modell: `MiniMax-M2.1` (Standardmodell).
* Aktualisiere die Preise in `models.json`, wenn du eine exakte Kostenverfolgung benötigst.

<div id="cerebras-glm-46-47">
  ### Cerebras (GLM 4.6 / 4.7)
</div>

Verwenden Sie Cerebras über den OpenAI-kompatiblen Endpunkt von Cerebras:

```json5
{
  env: { CEREBRAS_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: {
        primary: "cerebras/zai-glm-4.7",
        fallbacks: ["cerebras/zai-glm-4.6"]
      },
      models: {
        "cerebras/zai-glm-4.7": { alias: "GLM 4.7 (Cerebras)" },
        "cerebras/zai-glm-4.6": { alias: "GLM 4.6 (Cerebras)" }
      }
    }
  },
  models: {
    mode: "merge",
    providers: {
      cerebras: {
        baseUrl: "https://api.cerebras.ai/v1",
        apiKey: "${CEREBRAS_API_KEY}",
        api: "openai-completions",
        models: [
          { id: "zai-glm-4.7", name: "GLM 4.7 (Cerebras)" },
          { id: "zai-glm-4.6", name: "GLM 4.6 (Cerebras)" }
        ]
      }
    }
  }
}
```

Hinweise:

* Verwende `cerebras/zai-glm-4.7` für Cerebras; verwende `zai/glm-4.7` für Z.AI direkt.
* Setze `CEREBRAS_API_KEY` als Umgebungsvariable oder in der Konfiguration.

Hinweise:

* Unterstützte APIs: `openai-completions`, `openai-responses`, `anthropic-messages`,
  `google-generative-ai`
* Verwende `authHeader: true` + `headers` für benutzerdefinierte Authentifizierung.
* Überschreibe das Wurzelverzeichnis der Agent-Konfiguration mit `OPENCLAW_AGENT_DIR` (oder `PI_CODING_AGENT_DIR`),
  wenn du möchtest, dass `models.json` an einem anderen Ort gespeichert wird (Standard: `~/.openclaw/agents/main/agent`).

<div id="session">
  ### `session`
</div>

Steuert den Geltungsbereich der Sitzung, die Richtlinie und Auslöser für Zurücksetzungen sowie den Speicherort, an den der Sitzungs-Store geschrieben wird.

```json5
{
  session: {
    scope: "per-sender",
    dmScope: "main",
    identityLinks: {
      alice: ["telegram:123456789", "discord:987654321012345678"]
    },
    reset: {
      mode: "daily",
      atHour: 4,
      idleMinutes: 60
    },
    resetByType: {
      thread: { mode: "daily", atHour: 4 },
      dm: { mode: "idle", idleMinutes: 240 },
      group: { mode: "idle", idleMinutes: 120 }
    },
    resetTriggers: ["/new", "/reset"],
    // Standard ist bereits pro Agent unter ~/.openclaw/agents/<agentId>/sessions/sessions.json
    // Sie können dies mit {agentId}-Templating überschreiben:
    store: "~/.openclaw/agents/{agentId}/sessions/sessions.json",
    // Direct chats collapse to agent:<agentId>:<mainKey> (default: "main").
    mainKey: "main",
    agentToAgent: {
      // Max ping-pong reply turns between requester/target (0–5).
      maxPingPongTurns: 5
    },
    sendPolicy: {
      rules: [
        { action: "deny", match: { channel: "discord", chatType: "group" } }
      ],
      default: "allow"
    }
  }
}
```

Fields:

* `mainKey`: Bucket-Schlüssel für Direct-Chats (Standard: `"main"`). Nützlich, wenn du den primären DM-Thread „umbenennen“ möchtest, ohne `agentId` zu ändern.
  * Sandbox-Hinweis: `agents.defaults.sandbox.mode: "non-main"` verwendet diesen Schlüssel, um die Hauptsitzung zu erkennen. Jeder Sitzungsschlüssel, der nicht mit `mainKey` übereinstimmt (Gruppen/Channels), wird in der Sandbox ausgeführt.
* `dmScope`: wie DM-Sitzungen gruppiert werden (Standard: `"main"`).
  * `main`: alle DMs teilen sich die Hauptsitzung für Kontinuität.
  * `per-peer`: isoliert DMs nach Absender-ID über Channels hinweg.
  * `per-channel-peer`: isoliert DMs je Channel + Absender (empfohlen für Multi-User-Inboxen).
  * `per-account-channel-peer`: isoliert DMs je Account + Channel + Absender (empfohlen für Multi-Account-Inboxen).
* `identityLinks`: ordnet kanonische IDs Peers mit Anbieterpräfix zu, sodass dieselbe Person eine DM-Sitzung über Channels hinweg teilt, wenn `per-peer`, `per-channel-peer` oder `per-account-channel-peer` verwendet wird.
  * Beispiel: `alice: ["telegram:123456789", "discord:987654321012345678"]`.
* `reset`: primäre Reset-Richtlinie. Standard sind tägliche Resets um 4:00 Uhr lokaler Zeit auf dem Gateway-Host.
  * `mode`: `daily` oder `idle` (Standard: `daily`, wenn `reset` gesetzt ist).
  * `atHour`: lokale Stunde (0–23) für die tägliche Reset-Grenze.
  * `idleMinutes`: gleitendes Idle-Fenster in Minuten. Wenn daily + idle beide konfiguriert sind, gewinnt die Einstellung, die zuerst abläuft.
* `resetByType`: sitzungsspezifische Overrides für `dm`, `group` und `thread`.
  * Wenn du nur das Legacy-Feld `session.idleMinutes` ohne `reset`/`resetByType` setzt, bleibt OpenClaw im Idle-only-Modus für Abwärtskompatibilität.
* `heartbeatIdleMinutes`: optionales Idle-Override für Herzschlag-Checks (täglicher Reset gilt weiterhin, wenn aktiviert).
* `agentToAgent.maxPingPongTurns`: maximale Anzahl an Ping-Pong-Runden zwischen Anfragendem und Ziel (0–5, Standard 5).
* `sendPolicy.default`: `allow`- oder `deny`-Fallback, wenn keine Regel greift.
* `sendPolicy.rules[]`: Abgleich nach `channel`, `chatType` (`direct|group|room`) oder `keyPrefix` (z. B. `cron:`). Das erste `deny` gewinnt; ansonsten `allow`.

<div id="skills-skills-config">
  ### `skills` (Fähigkeiten-Konfiguration)
</div>

Steuert die Allowlist für gebündelte Fähigkeiten, Installationspräferenzen, zusätzliche Fähigkeiten-Ordner und
Überschreibungen pro Fähigkeit. Gilt für **gebündelte** Fähigkeiten und `~/.openclaw/skills`
(Fähigkeiten im arbeitsbereich haben bei Namenskonflikten weiterhin Vorrang).

Felder:

* `allowBundled`: optionale Allowlist nur für **gebündelte** Fähigkeiten. Wenn gesetzt, sind nur diese
  gebündelten Fähigkeiten zulässig (verwaltete/Arbeitsbereich-Fähigkeiten bleiben unberührt).
* `load.extraDirs`: zusätzliche Fähigkeiten-Verzeichnisse, die durchsucht werden (niedrigste Priorität).
* `install.preferBrew`: bevorzugt `brew`-Installer, wenn verfügbar (Standard: true).
* `install.nodeManager`: Node.js-Paketmanager-Präferenz (`npm` | `pnpm` | `yarn`, Standard: npm).
* `entries.<skillKey>`: Konfigurationsüberschreibungen pro Fähigkeit.

Pro-Fähigkeit-Felder:

* `enabled`: auf `false` setzen, um eine Fähigkeit zu deaktivieren, selbst wenn sie gebündelt/installiert ist.
* `env`: Umgebungsvariablen, die für die Agent-Ausführung injiziert werden (nur, wenn sie noch nicht gesetzt sind).
* `apiKey`: optionale Komfortangabe für Fähigkeiten, die eine primäre Umgebungsvariable deklarieren (z. B. `nano-banana-pro` → `GEMINI_API_KEY`).

Beispiel:

```json5
{
  skills: {
    allowBundled: ["gemini", "peekaboo"],
    load: {
      extraDirs: [
        "~/Projects/agent-scripts/skills",
        "~/Projects/oss/some-skill-pack/skills"
      ]
    },
    install: {
      preferBrew: true,
      nodeManager: "npm"
    },
    entries: {
      "nano-banana-pro": {
        apiKey: "GEMINI_KEY_HERE",
        env: {
          GEMINI_API_KEY: "GEMINI_KEY_HERE"
        }
      },
      peekaboo: { enabled: true },
      sag: { enabled: false }
    }
  }
}
```

<div id="plugins-extensions">
  ### `plugins` (Erweiterungen)
</div>

Steuert Plugin-Erkennung, Zulassen/Blockieren und Plugin-spezifische Konfiguration. Plugins werden geladen
aus `~/.openclaw/extensions`, `<workspace>/.openclaw/extensions` sowie allen
Einträgen unter `plugins.load.paths`. **Konfigurationsänderungen erfordern einen Neustart des Gateways.**
Siehe [/plugin](/de/plugin) für die vollständige Beschreibung.

Felder:

* `enabled`: Hauptschalter für das Laden von Plugins (Standard: true).
* `allow`: optionale Allowlist von Plugin-IDs; wenn gesetzt, werden nur aufgeführte Plugins geladen.
* `deny`: optionale Denylist von Plugin-IDs (deny gewinnt).
* `load.paths`: zusätzliche Plugin-Dateien oder -Verzeichnisse, die geladen werden (absolut oder `~`).
* `entries.<pluginId>`: Plugin-spezifische Überschreibungen.
  * `enabled`: auf `false` setzen, um zu deaktivieren.
  * `config`: Plugin-spezifisches Konfigurationsobjekt (falls vorhanden vom Plugin validiert).

Beispiel:

```json5
{
  plugins: {
    enabled: true,
    allow: ["voice-call"],
    load: {
      paths: ["~/Projects/oss/voice-call-extension"]
    },
    entries: {
      "voice-call": {
        enabled: true,
        config: {
          provider: "twilio"
        }
      }
    }
  }
}
```

<div id="browser-openclaw-managed-browser">
  ### `browser` (von openclaw verwalteter Browser)
</div>

OpenClaw kann eine **dedizierte, isolierte** Chrome-/Brave-/Edge-/Chromium-Instanz für openclaw starten und einen kleinen Loopback-Steuerdienst bereitstellen.
Profile können über `profiles.<name>.cdpUrl` auf einen **entfernten** Chromium-basierten Browser verweisen. Entfernte
Profile können nur angehängt werden (start/stop/reset sind deaktiviert).

`browser.cdpUrl` bleibt für ältere Single-Profile-Konfigurationen erhalten und dient als Basis-
Scheme/Host für Profile, die nur `cdpPort` setzen.

Standardwerte:

* enabled: `true`
* evaluateEnabled: `true` (auf `false` setzen, um `act:evaluate` und `wait --fn` zu deaktivieren)
* Steuerdienst: nur Loopback (Port abgeleitet aus `gateway.port`, Standard `18791`)
* CDP-URL: `http://127.0.0.1:18792` (Steuerdienst + 1, ältere Single-Profile-Konfiguration)
* Profilfarbe: `#FF4500` (Lobster-Orange)
* Hinweis: Der Steuerdienst wird vom laufenden Gateway gestartet (Menüleiste von OpenClaw.app oder `openclaw gateway`).
* Reihenfolge der Auto-Erkennung: Standardbrowser, falls Chromium-basiert; andernfalls Chrome → Brave → Edge → Chromium → Chrome Canary.

```json5
{
  browser: {
    enabled: true,
    evaluateEnabled: true,
    // cdpUrl: "http://127.0.0.1:18792", // legacy single-profile override
    defaultProfile: "chrome",
    profiles: {
      openclaw: { cdpPort: 18800, color: "#FF4500" },
      work: { cdpPort: 18801, color: "#0066CC" },
      remote: { cdpUrl: "http://10.0.0.42:9222", color: "#00AA00" }
    },
    color: "#FF4500",
    // Advanced:
    // headless: false,
    // noSandbox: false,
    // executablePath: "/Applications/Brave Browser.app/Contents/MacOS/Brave Browser",
    // attachOnly: false, // auf true setzen beim Tunneln eines Remote-CDP zu localhost
  }
}
```

<div id="ui-appearance">
  ### `ui` (Erscheinungsbild)
</div>

Optionale Akzentfarbe, die von den nativen Apps für das UI‑Chrome verwendet wird (z. B. die Tönung der Talk-Mode-Sprechblase).

Wenn nicht gesetzt, fallen Clients auf ein dezentes Hellblau zurück.

```json5
{
  ui: {
    seamColor: "#FF4500", // hex (RRGGBB or #RRGGBB)
    // Optional: Überschreibung der Assistenten-Identität für die Control UI.
    // Falls nicht gesetzt, verwendet die Control UI die aktive Agent-Identität (Config oder IDENTITY.md).
    assistant: {
      name: "OpenClaw",
      avatar: "CB" // emoji, short text, or image URL/data URI
    }
  }
}
```

<div id="gateway-gateway-server-mode-bind">
  ### `gateway` (Gateway-Servermodus + Bind)
</div>

Verwende `gateway.mode`, um explizit festzulegen, ob dieses System das Gateway ausführen soll.

Standardwerte:

* mode: **unset** (als „nicht automatisch starten“ behandelt)
* bind: `loopback`
* port: `18789` (einziger Port für WS + HTTP)

```json5
{
  gateway: {
    mode: "local", // or "remote"
    port: 18789, // WS + HTTP multiplex
    bind: "loopback",
    // controlUi: { enabled: true, basePath: "/openclaw" }
    // auth: { mode: "token", token: "your-token" } // Token steuert WS + Control UI Zugriff
    // tailscale: { mode: "off" | "serve" | "funnel" }
  }
}
```

Control UI-Basispfad:

* `gateway.controlUi.basePath` legt das URL-Präfix fest, unter dem die Control UI ausgeliefert wird.
* Beispiele: `"/ui"`, `"/openclaw"`, `"/apps/openclaw"`.
* Standard: Root-Pfad (`/`) (unverändert).
* `gateway.controlUi.allowInsecureAuth` erlaubt ausschließlich Token-basierte Authentifizierung für die Control UI, wenn
  die Geräteidentität weggelassen wird (typischerweise über HTTP). Standard: `false`. Bevorzuge HTTPS
  (Tailscale Serve) oder `127.0.0.1`.
* `gateway.controlUi.dangerouslyDisableDeviceAuth` deaktiviert Geräteidentitätsprüfungen für die
  Control UI (nur Token/Passwort). Standard: `false`. Nur als Notfallmaßnahme verwenden.

Zugehörige Dokumentation:

* [Control UI](/de/web/control-ui)
* [Web-Überblick](/de/web)
* [Tailscale](/de/gateway/tailscale)
* [Remotezugriff](/de/gateway/remote)

Vertrauenswürdige Proxys:

* `gateway.trustedProxies`: Liste von Reverse-Proxy-IP-Adressen, die TLS vor dem Gateway terminieren.
* Wenn eine Verbindung von einer dieser IPs kommt, verwendet OpenClaw `x-forwarded-for` (oder `x-real-ip`), um die Client-IP für lokale Kopplungsprüfungen und HTTP-/Lokalprüfungen zu bestimmen.
* Liste nur Proxys auf, die du vollständig kontrollierst, und stelle sicher, dass sie eingehende `x-forwarded-for`-Header **überschreiben**.

Hinweise:

* `openclaw gateway` verweigert den Start, sofern `gateway.mode` nicht auf `local` gesetzt ist (oder du das Override-Flag übergibst).
* `gateway.port` steuert einen einzelnen, multiplexenden Port, der für WebSocket + HTTP (Control UI, Hooks, A2UI) verwendet wird.
* OpenAI Chat Completions-Endpunkt: **standardmäßig deaktiviert**; aktiviere ihn mit `gateway.http.endpoints.chatCompletions.enabled: true`.
* Priorität: `--port` &gt; `OPENCLAW_GATEWAY_PORT` &gt; `gateway.port` &gt; Standard `18789`.
* Gateway-Auth ist standardmäßig erforderlich (Token/Passwort oder Tailscale Serve-Identität). Bindungen an Nicht-Loopback-Adressen erfordern ein gemeinsames Token/Passwort.
* Der Onboarding-Assistent generiert standardmäßig ein Gateway-Token (auch auf Loopback).
* `gateway.remote.token` ist **nur** für entfernte CLI-Aufrufe; es aktiviert keine lokale Gateway-Auth. `gateway.token` wird ignoriert.

Auth und Tailscale:

* `gateway.auth.mode` legt die Anforderungen für den Handshake fest (`token` oder `password`). Wenn nicht gesetzt, wird Token-Auth verwendet.
* `gateway.auth.token` speichert das gemeinsame Token für Token-Auth (wird von der CLI auf derselben Maschine verwendet).
* Wenn `gateway.auth.mode` gesetzt ist, wird nur diese Methode akzeptiert (plus optionale Tailscale-Header).
* `gateway.auth.password` kann hier gesetzt werden oder über `OPENCLAW_GATEWAY_PASSWORD` (empfohlen).
* `gateway.auth.allowTailscale` erlaubt Tailscale Serve-Identitätsheader
  (`tailscale-user-login`), die Auth zu erfüllen, wenn die Anfrage über Loopback eintrifft
  mit `x-forwarded-for`, `x-forwarded-proto` und `x-forwarded-host`. OpenClaw
  verifiziert die Identität, indem es die `x-forwarded-for`-Adresse über
  `tailscale whois` auflöst, bevor sie akzeptiert wird. Wenn `true`, benötigen Serve-Anfragen kein
  Token/Passwort; setze `false`, um explizite Zugangsdaten zu erzwingen. Standard ist
  `true`, wenn `tailscale.mode = "serve"` und der Auth-Modus nicht `password` ist.
* `gateway.tailscale.mode: "serve"` verwendet Tailscale Serve (nur Tailnet, Loopback-Bind).
* `gateway.tailscale.mode: "funnel"` macht das Dashboard öffentlich zugänglich; Auth ist erforderlich.
* `gateway.tailscale.resetOnExit` setzt die Serve-/Funnel-Konfiguration beim Herunterfahren zurück.

Standardwerte für Remote-Clients (CLI):

* `gateway.remote.url` legt die Standard-Gateway-WebSocket-URL für CLI-Aufrufe fest, wenn `gateway.mode = "remote"` ist.
* `gateway.remote.transport` wählt den macOS-Remote-Transport aus (`ssh` ist der Standard, `direct` für ws/wss). Bei `direct` muss `gateway.remote.url` mit `ws://` oder `wss://` beginnen. `ws://host` verwendet standardmäßig Port `18789`.
* `gateway.remote.token` stellt das Token für Remote-Aufrufe bereit (leer lassen, um auf Authentifizierung zu verzichten).
* `gateway.remote.password` stellt das Passwort für Remote-Aufrufe bereit (leer lassen, um auf Authentifizierung zu verzichten).

Verhalten der macOS-App:

* OpenClaw.app überwacht `~/.openclaw/openclaw.json` und wechselt den Modus live, wenn sich `gateway.mode` oder `gateway.remote.url` ändert.
* Wenn `gateway.mode` nicht gesetzt ist, aber `gateway.remote.url` gesetzt ist, behandelt die macOS-App dies als Remote-Modus.
* Wenn du den Verbindungsmodus in der macOS-App änderst, schreibt sie `gateway.mode` (sowie `gateway.remote.url` + `gateway.remote.transport` im Remote-Modus) zurück in die Konfigurationsdatei.

```json5
{
  gateway: {
    mode: "remote",
    remote: {
      url: "ws://gateway.tailnet:18789",
      token: "your-token",
      password: "your-password"
    }
  }
}
```

Direkttransport-Beispiel (macOS-App):

```json5
{
  gateway: {
    mode: "remote",
    remote: {
      transport: "direct",
      url: "wss://gateway.example.ts.net",
      token: "your-token"
    }
  }
}
```

<div id="gatewayreload-config-hot-reload">
  ### `gateway.reload` (Config-Hot-Reload)
</div>

Das Gateway überwacht `~/.openclaw/openclaw.json` (oder `OPENCLAW_CONFIG_PATH`) und wendet Änderungen automatisch an.

Modi:

* `hybrid` (Standard): sichere Änderungen per Hot-Reload anwenden; das Gateway bei kritischen Änderungen neu starten.
* `hot`: nur Hot-Reload-sichere Änderungen anwenden; protokollieren, wenn ein Neustart erforderlich ist.
* `restart`: das Gateway bei jeder Konfigurationsänderung neu starten.
* `off`: Hot-Reload deaktivieren.

```json5
{
  gateway: {
    reload: {
      mode: "hybrid",
      debounceMs: 300
    }
  }
}
```

<div id="hot-reload-matrix-files-impact">
  #### Hot-Reload-Matrix (Dateien + Auswirkung)
</div>

Beobachtete Dateien:

* `~/.openclaw/openclaw.json` (oder `OPENCLAW_CONFIG_PATH`)

Hot angewendet (kein vollständiger Gateway-Neustart):

* `hooks` (Webhook-Auth/Pfad/Zuordnungen) + `hooks.gmail` (Gmail-Watcher wird neu gestartet)
* `browser` (Neustart des Browser-Steuerungsservers)
* `cron` (Neustart des Cron-Dienstes + Aktualisierung der Parallelität)
* `agents.defaults.heartbeat` (Neustart des Herzschlag-Runners)
* `web` (Neustart des WhatsApp-Web-Kanals)
* `telegram`, `discord`, `signal`, `imessage` (Neustarts der Kanäle)
* `agent`, `models`, `routing`, `messages`, `session`, `whatsapp`, `logging`, `skills`, `ui`, `talk`, `identity`, `wizard` (dynamische Lesevorgänge)

Erfordert vollständigen Gateway-Neustart:

* `gateway` (Port/Bind/Auth/Control UI/Tailscale)
* `bridge` (Legacy)
* `discovery`
* `canvasHost`
* `plugins`
* Jeder unbekannte/nicht unterstützte Konfigurationspfad (führt aus Sicherheitsgründen standardmäßig zu einem Neustart)

<div id="multi-instance-isolation">
  ### Isolierung mehrerer Instanzen
</div>

Um mehrere Gateways auf einem Host auszuführen (für Redundanz oder einen Rescue‑Bot), isoliere Zustands- und Konfigurationsdaten pro Instanz und verwende eindeutige Ports:

* `OPENCLAW_CONFIG_PATH` (Konfiguration pro Instanz)
* `OPENCLAW_STATE_DIR` (Sitzungen/Creds)
* `agents.defaults.workspace` (Speicher)
* `gateway.port` (eindeutig pro Instanz)

Komfort-Flags (CLI):

* `openclaw --dev …` → verwendet `~/.openclaw-dev` und verschiebt Ports ausgehend vom Basiswert `19001`
* `openclaw --profile <name> …` → verwendet `~/.openclaw-<name>` (Port über Config/Env/Flags)

Siehe [Gateway runbook](/de/gateway) für das abgeleitete Port-Mapping (Gateway/Browser/Canvas).
Siehe [Multiple gateways](/de/gateway/multiple-gateways) für Details zur Isolation der Browser-/CDP-Ports.

Beispiel:

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json \
OPENCLAW_STATE_DIR=~/.openclaw-a \
openclaw gateway --port 19001
```

<div id="hooks-gateway-webhooks">
  ### `hooks` (Gateway-Webhooks)
</div>

Aktiviert einen einfachen HTTP-Webhook-Endpunkt auf dem Gateway-HTTP-Server.

Standardwerte:

* enabled: `false`
* path: `/hooks`
* maxBodyBytes: `262144` (256 KB)

```json5
{
  hooks: {
    enabled: true,
    token: "shared-secret",
    path: "/hooks",
    presets: ["gmail"],
    transformsDir: "~/.openclaw/hooks",
    mappings: [
      {
        match: { path: "gmail" },
        action: "agent",
        wakeMode: "now",
        name: "Gmail",
        sessionKey: "hook:gmail:{{messages[0].id}}",
        messageTemplate:
          "From: {{messages[0].from}}\nSubject: {{messages[0].subject}}\n{{messages[0].snippet}}",
        deliver: true,
        channel: "last",
        model: "openai/gpt-5.2-mini",
      },
    ],
  }
}
```

Anfragen müssen das Hook-Token enthalten:

* `Authorization: Bearer <token>` **oder**
* `x-openclaw-token: <token>` **oder**
* `?token=<token>`

Endpoints:

* `POST /hooks/wake` → `{ text, mode?: "now"|"next-heartbeat" }`
* `POST /hooks/agent` → `{ message, name?, sessionKey?, wakeMode?, deliver?, channel?, to?, model?, thinking?, timeoutSeconds? }`
* `POST /hooks/<name>` → wird über `hooks.mappings` aufgelöst

`/hooks/agent` schreibt immer eine Zusammenfassung in die Hauptsitzung (und kann optional einen sofortigen Herzschlag über `wakeMode: "now"` auslösen).

Mapping-Hinweise:

* `match.path` entspricht dem Unterpfad nach `/hooks` (z. B. `/hooks/gmail` → `gmail`).
* `match.source` entspricht einem Payload-Feld (z. B. `{ source: "gmail" }`), sodass du einen generischen Pfad `/hooks/ingest` verwenden kannst.
* Templates wie `{{messages[0].subject}}` lesen aus der Payload.
* `transform` kann auf ein JS/TS-Modul verweisen, das eine Hook-Aktion zurückgibt.
* `deliver: true` sendet die finale Antwort an einen Kanal; `channel` ist standardmäßig `last` (fällt auf WhatsApp zurück).
* Wenn es keine vorherige Zustellroute gibt, setze `channel` + `to` explizit (erforderlich für Telegram/Discord/Google Chat/Slack/Signal/iMessage/MS Teams).
* `model` überschreibt das LLM für diesen Hook-Lauf (`provider/model` oder Alias; muss erlaubt sein, wenn `agents.defaults.models` gesetzt ist).

Gmail-Hilfskonfiguration (verwendet von `openclaw webhooks gmail setup` / `run`):

```json5
{
  hooks: {
    gmail: {
      account: "openclaw@gmail.com",
      topic: "projects/<project-id>/topics/gog-gmail-watch",
      subscription: "gog-gmail-watch-push",
      pushToken: "shared-push-token",
      hookUrl: "http://127.0.0.1:18789/hooks/gmail",
      includeBody: true,
      maxBytes: 20000,
      renewEveryMinutes: 720,
      serve: { bind: "127.0.0.1", port: 8788, path: "/" },
      tailscale: { mode: "funnel", path: "/gmail-pubsub" },

      // Optional: günstigeres Modell für Gmail-Hook-Verarbeitung verwenden
      // Fällt zurück auf agents.defaults.model.fallbacks, dann primary, bei auth/rate-limit/timeout
      model: "openrouter/meta-llama/llama-3.3-70b-instruct:free",
      // Optional: default thinking level for Gmail hooks
      thinking: "off",
    }
  }
}
```

Modell-Override für Gmail-Hooks:

* `hooks.gmail.model` gibt ein Modell an, das für die Verarbeitung von Gmail-Hooks verwendet werden soll (Standard ist das primäre Modell der Sitzung).
* Akzeptiert `provider/model`-Referenzen oder Aliasse aus `agents.defaults.models`.
* Fällt bei Authentifizierungs-, Rate-Limit- oder Timeout-Fehlern zurück auf `agents.defaults.model.fallbacks`, dann `agents.defaults.model.primary`.
* Wenn `agents.defaults.models` gesetzt ist, nimm das Hook-Modell in die Allowlist auf.
* Warnt beim Start, wenn das konfigurierte Modell nicht im Modellkatalog oder in der Allowlist enthalten ist.
* `hooks.gmail.thinking` setzt die standardmäßige Denktiefe für Gmail-Hooks und wird von `thinking` auf Hook-Ebene überschrieben.

Gateway-Autostart:

* Wenn `hooks.enabled=true` und `hooks.gmail.account` gesetzt ist, startet das Gateway
  `gog gmail watch serve` beim Booten und erneuert das Watch-Abo automatisch.
* Setze `OPENCLAW_SKIP_GMAIL_WATCHER=1`, um den Auto-Start zu deaktivieren (für manuelle Ausführungen).
* Vermeide, einen separaten `gog gmail watch serve` parallel zum Gateway auszuführen; der Befehl
  schlägt mit `listen tcp 127.0.0.1:8788: bind: address already in use` fehl.

Hinweis: Wenn `tailscale.mode` aktiviert ist, setzt OpenClaw standardmäßig `serve.path` auf `/`,
damit Tailscale `/gmail-pubsub` korrekt weiterleiten kann (es entfernt das gesetzte Pfadpräfix).
Wenn das Backend den präfixierten Pfad erhalten soll, setze
`hooks.gmail.tailscale.target` auf eine vollständige URL (und passe `serve.path` an).

<div id="canvashost-lantailnet-canvas-file-server-live-reload">
  ### `canvasHost` (LAN/Tailnet-Canvas-Dateiserver + Live-Reload)
</div>

Das Gateway stellt ein Verzeichnis mit HTML/CSS/JS über HTTP bereit, sodass iOS-/Android-Knoten einfach per `canvas.navigate` darauf zugreifen können.

Standard-Stammverzeichnis: `~/.openclaw/workspace/canvas`
Standardport: `18793` (gewählt, um den openclaw-Browser-CDP-Port `18792` zu vermeiden)
Der Server lauscht auf dem **Gateway-Bind-Host** (LAN oder Tailnet), sodass Knoten ihn erreichen können.

Der Server:

* liefert Dateien unter `canvasHost.root` aus
* injiziert einen kleinen Live-Reload-Client in ausgelieferte HTML-Dateien
* beobachtet das Verzeichnis und sendet Reloads über einen WebSocket-Endpunkt unter `/__openclaw__/ws`
* erstellt automatisch eine Starter-`index.html`, wenn das Verzeichnis leer ist (damit du sofort etwas siehst)
* stellt außerdem A2UI unter `/__openclaw__/a2ui/` bereit und wird Knoten als `canvasHostUrl` bekanntgegeben
  (wird von Knoten immer für Canvas/A2UI verwendet)

Deaktiviere Live-Reload (und Dateiüberwachung), wenn das Verzeichnis groß ist oder du auf einen `EMFILE`-Fehler stößt:

* Konfiguration: `canvasHost: { liveReload: false }`

```json5
{
  canvasHost: {
    root: "~/.openclaw/workspace/canvas",
    port: 18793,
    liveReload: true
  }
}
```

Änderungen an `canvasHost.*` erfordern einen Neustart des Gateways (Konfigurations-Reload startet es neu).

Deaktivieren mit:

* config: `canvasHost: { enabled: false }`
* env: `OPENCLAW_SKIP_CANVAS_HOST=1`

<div id="bridge-legacy-tcp-bridge-removed">
  ### `bridge` (veraltete TCP-Bridge, entfernt)
</div>

Aktuelle Builds enthalten keinen TCP-Bridge-Listener mehr; `bridge.*`-Konfigurationsschlüssel werden ignoriert.
Knoten verbinden sich über den Gateway-WebSocket. Dieser Abschnitt wird aus historischen Gründen beibehalten.

Veraltetes Verhalten:

* Das Gateway konnte eine einfache TCP-Bridge für Knoten (iOS/Android) bereitstellen, typischerweise auf Port `18790`.

Standardwerte:

* enabled: `true`
* port: `18790`
* bind: `lan` (bindet an `0.0.0.0`)

Bind-Modi:

* `lan`: `0.0.0.0` (über jedes Interface erreichbar, inklusive LAN/Wi‑Fi und Tailscale)
* `tailnet`: bindet nur an die Tailscale-IP der Maschine (empfohlen für Wien ⇄ London)
* `loopback`: `127.0.0.1` (nur lokal)
* `auto`: bevorzugt die Tailnet-IP, falls vorhanden, sonst `lan`

TLS:

* `bridge.tls.enabled`: TLS für Bridge-Verbindungen aktivieren (bei Aktivierung nur TLS).
* `bridge.tls.autoGenerate`: ein selbstsigniertes Zertifikat erzeugen, wenn kein Zertifikat/Key vorhanden ist (Standard: true).
* `bridge.tls.certPath` / `bridge.tls.keyPath`: PEM-Pfade für das Bridge-Zertifikat + den privaten Schlüssel.
* `bridge.tls.caPath`: optionales PEM-CA-Bundle (benutzerdefinierte Roots oder zukünftiges mTLS).

Wenn TLS aktiviert ist, kündigt das Gateway `bridgeTls=1` und `bridgeTlsSha256` in Discovery-TXT-
Einträgen an, sodass Knoten das Zertifikat pinnen können. Manuelle Verbindungen verwenden „Trust on First Use“, wenn noch kein
Fingerabdruck gespeichert ist.
Automatisch generierte Zertifikate erfordern `openssl` im PATH; wenn die Generierung fehlschlägt, startet die Bridge nicht.

```json5
{
  bridge: {
    enabled: true,
    port: 18790,
    bind: "tailnet",
    tls: {
      enabled: true,
      // Verwendet ~/.openclaw/bridge/tls/bridge-{cert,key}.pem, falls weggelassen.
      // certPath: "~/.openclaw/bridge/tls/bridge-cert.pem",
      // keyPath: "~/.openclaw/bridge/tls/bridge-key.pem"
    }
  }
}
```

<div id="discoverymdns-bonjour-mdns-broadcast-mode">
  ### `discovery.mdns` (Bonjour- / mDNS-Broadcast-Modus)
</div>

Steuert mDNS-Discovery-Broadcasts im LAN (`_openclaw-gw._tcp`).

* `minimal` (Standard): lässt `cliPath` + `sshPort` in TXT-Records weg
* `full`: nimmt `cliPath` + `sshPort` in TXT-Records auf
* `off`: deaktiviert mDNS-Broadcasts vollständig
* Hostname: Standard ist `openclaw` (bewirbt `openclaw.local`). Kann mit `OPENCLAW_MDNS_HOSTNAME` überschrieben werden.

```json5
{
  discovery: { mdns: { mode: "minimal" } }
}
```

<div id="discoverywidearea-wide-area-bonjour-unicast-dnssd">
  ### `discovery.wideArea` (Wide-Area Bonjour / Unicast-DNS‑SD)
</div>

Wenn aktiviert, schreibt der Gateway eine Unicast-DNS-SD-Zone für `_openclaw-gw._tcp` unter `~/.openclaw/dns/` unter Verwendung der konfigurierten Discovery-Domain (Beispiel: `openclaw.internal.`).

Damit iOS/Android netzwerkübergreifend entdecken können (Wien ⇄ London), kombiniere dies mit:

* einem DNS-Server auf dem Gateway-Host, der deine gewählte Domain ausliefert (CoreDNS wird empfohlen)
* Tailscale-**Split DNS**, sodass Clients diese Domain über den Gateway-DNS-Server auflösen

Einmalige Einrichtungshilfe (Gateway-Host):

```bash
openclaw dns setup --apply
```

```json5
{
  discovery: { wideArea: { enabled: true } }
}
```

## Template-Variablen

Template-Platzhalter werden in `tools.media.*.models[].args` und `tools.media.models[].args` (und allen zukünftigen vorlagenbasierten Argumentfeldern) aufgelöst.

| Variable | Beschreibung |
|----------|-------------|
| `{{Body}}` | Vollständiger eingehender Nachrichtentext |
| `{{RawBody}}` | Roher eingehender Nachrichtentext (keine Verlaufs-/Absender-Wrapper; am besten für Befehls-Parsing) |
| `{{BodyStripped}}` | Text mit entfernten Gruppen-Erwähnungen (beste Standardeinstellung für Agenten) |
| `{{From}}` | Absenderkennung (E.164 für WhatsApp; kann je Kanal abweichen) |
| `{{To}}` | Zielkennung |
| `{{MessageSid}}` | Kanal-Nachrichten-ID (falls verfügbar) |
| `{{SessionId}}` | Aktuelle Sitzungs-UUID |
| `{{IsNewSession}}` | `"true"`, wenn eine neue Sitzung erstellt wurde |
| `{{MediaUrl}}` | Eingehende Medien-Pseudo-URL (falls vorhanden) |
| `{{MediaPath}}` | Lokaler Medienpfad (falls heruntergeladen) |
| `{{MediaType}}` | Medientyp (Bild/Audio/Dokument/…) |
| `{{Transcript}}` | Audio-Transkript (falls aktiviert) |
| `{{Prompt}}` | Aufgelöster Medien-Prompt für CLI-Einträge |
| `{{MaxChars}}` | Aufgelöste maximale Ausgabelänge (Zeichen) für CLI-Einträge |
| `{{ChatType}}` | `"direct"` oder `"group"` |
| `{{GroupSubject}}` | Gruppenbetreff (bestmöglich ermittelt) |
| `{{GroupMembers}}` | Vorschau der Gruppenmitglieder (bestmöglich ermittelt) |
| `{{SenderName}}` | Anzeigename des Absenders (bestmöglich ermittelt) |
| `{{SenderE164}}` | Telefonnummer des Absenders (bestmöglich ermittelt) |
| `{{Provider}}` | Anbieterhinweis (whatsapp|telegram|discord|googlechat|slack|signal|imessage|msteams|webchat|…) |

<div id="cron-gateway-scheduler">
  ## Cron (Gateway-Scheduler)
</div>

Cron ist ein Gateway-eigener Scheduler für Aufweckvorgänge und geplante Jobs. Siehe [Cron-Jobs](/de/automation/cron-jobs) für eine Funktionsübersicht und CLI-Beispiele.

```json5
{
  cron: {
    enabled: true,
    maxConcurrentRuns: 2
  }
}
```

***

*Weiter: [Agent-Runtime](/de/concepts/agent)* 🦞
