---
title: Mattermost
summary: "Einrichtung des Mattermost-Bots und OpenClaw-Konfiguration"
read_when:
  - Mattermost einrichten
  - Fehlerbehebung beim Mattermost-Routing
---

<div id="mattermost-plugin">
  # Mattermost (Plugin)
</div>

Status: unterstützt über Plugin (Bot-Token + WebSocket-Events). Kanäle, Gruppen und Direktnachrichten werden unterstützt.
Mattermost ist eine selbstgehostete Team-Messaging-Plattform; siehe die offizielle Website
[mattermost.com](https://mattermost.com) für Produktdetails und Downloads.

<div id="plugin-required">
  ## Plugin erforderlich
</div>

Mattermost wird als Plugin bereitgestellt und ist nicht Teil der Kerninstallation.

Installation über die CLI (npm-Registry):

```bash
openclaw plugins install @openclaw/mattermost
```

Lokaler Checkout (bei Ausführung aus einem Git-Repo):

```bash
openclaw plugins install ./extensions/mattermost
```

Wenn du Mattermost während der Konfiguration bzw. beim Onboarding auswählst und ein Git-Checkout erkannt wird,
schlägt OpenClaw den lokalen Installationspfad automatisch vor.

Details: [Plugins](/de/plugin)

<div id="quick-setup">
  ## Schnelle Einrichtung
</div>

1. Installiere das Mattermost-Plugin.
2. Erstelle einen Mattermost-Bot-Account und kopiere das **Bot-Token**.
3. Kopiere die Mattermost-**Basis-URL** (z. B. `https://chat.example.com`).
4. Konfiguriere OpenClaw und starte das Gateway.

Minimale Konfiguration:

```json5
{
  channels: {
    mattermost: {
      enabled: true,
      botToken: "mm-token",
      baseUrl: "https://chat.example.com",
      dmPolicy: "pairing"
    }
  }
}
```

<div id="environment-variables-default-account">
  ## Umgebungsvariablen (Standardkonto)
</div>

Setze diese auf dem Gateway-Host, wenn du Umgebungsvariablen bevorzugst:

* `MATTERMOST_BOT_TOKEN=...`
* `MATTERMOST_URL=https://chat.example.com`

Umgebungsvariablen gelten nur für das **Standardkonto** (`default`). Für andere Konten musst du Konfigurationswerte verwenden.

<div id="chat-modes">
  ## Chat-Modi
</div>

Mattermost antwortet automatisch auf DMs. Das Verhalten in Kanälen wird über `chatmode` gesteuert:

* `oncall` (Standard): antwortet nur, wenn du in Kanälen mit @ erwähnt wirst.
* `onmessage`: antwortet auf jede Kanalnachricht.
* `onchar`: antwortet, wenn eine Nachricht mit einem Triggerpräfix beginnt.

Konfigurationsbeispiel:

```json5
{
  channels: {
    mattermost: {
      chatmode: "onchar",
      oncharPrefixes: [">", "!"]
    }
  }
}
```

Hinweise:

* `onchar` reagiert weiterhin auf explizite @-Erwähnungen.
* `channels.mattermost.requireMention` wird für Legacy-Konfigurationen weiterhin unterstützt, aber `chatmode` wird bevorzugt.

<div id="access-control-dms">
  ## Zugriffskontrolle (DMs)
</div>

* Standardwert: `channels.mattermost.dmPolicy = "pairing"` (unbekannte Absender erhalten einen Kopplungscode).
* Genehmigung über:
  * `openclaw pairing list mattermost`
  * `openclaw pairing approve mattermost <CODE>`
* Öffentliche DMs: `channels.mattermost.dmPolicy="open"` (erlaubt uneingeschränkte Nachrichtenannahme von beliebigen Absendern) plus `channels.mattermost.allowFrom=["*"]`.

<div id="channels-groups">
  ## Kanäle (Gruppen)
</div>

* Standard: `channels.mattermost.groupPolicy = "allowlist"` (erfordert Erwähnung, „mention-gated“).
* Absender über `channels.mattermost.groupAllowFrom` auf die Allowlist setzen (Benutzer-IDs oder `@username`).
* Offene Kanäle: `channels.mattermost.groupPolicy="open"` (Einstellung „open“ = erlaubt uneingeschränkte Nachrichtenannahme von allen Nutzern; erfordert Erwähnung, „mention-gated“).

<div id="targets-for-outbound-delivery">
  ## Ziele für ausgehenden Versand
</div>

Verwende diese Zielformate mit `openclaw message send` oder cron/Webhooks:

* `channel:<id>` für einen Channel
* `user:<id>` für eine Direktnachricht
* `@username` für eine Direktnachricht (aufgelöst über die Mattermost-API)

IDs ohne Präfix werden als Channels behandelt.

<div id="multi-account">
  ## Mehrere Konten
</div>

Mattermost unterstützt mehrere Konten unter `channels.mattermost.accounts`:

```json5
{
  channels: {
    mattermost: {
      accounts: {
        default: { name: "Primary", botToken: "mm-token", baseUrl: "https://chat.example.com" },
        alerts: { name: "Alerts", botToken: "mm-token-2", baseUrl: "https://alerts.example.com" }
      }
    }
  }
}
```

<div id="troubleshooting">
  ## Fehlerbehebung
</div>

* Keine Antworten in Kanälen: Stelle sicher, dass der Bot im Kanal ist und erwähnt wird (oncall), verwende ein Trigger-Präfix (onchar) oder setze `chatmode: "onmessage"`.
* Authentifizierungsfehler: Überprüfe den Bot-Token, die Basis-URL und ob der Account aktiviert ist.
* Probleme mit mehreren Accounts: Umgebungsvariablen gelten nur für den `default`-Account.