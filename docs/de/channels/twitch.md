---
title: Twitch
summary: "Konfiguration und Einrichtung des Twitch-Chatbots"
read_when:
  - Einrichten der Twitch-Chat-Integration für OpenClaw
---

<div id="twitch-plugin">
  # Twitch (Plugin)
</div>

Twitch-Chat-Unterstützung über eine IRC-Verbindung. OpenClaw stellt als Twitch-Benutzerkonto (Bot) eine Verbindung her, um Nachrichten in Kanälen zu empfangen und zu senden.

<div id="plugin-required">
  ## Plugin erforderlich
</div>

Twitch wird als Plugin ausgeliefert und ist nicht in der Kerninstallation enthalten.

Über die CLI installieren (npm-Registry):

```bash
openclaw plugins install @openclaw/twitch
```

Lokaler Checkout (bei Ausführung aus einem Git-Repository):

```bash
openclaw plugins install ./extensions/twitch
```

Weitere Informationen: [Plugins](/de/plugin)

<div id="quick-setup-beginner">
  ## Schnelleinrichtung (für Einsteiger)
</div>

1. Erstelle ein dediziertes Twitch-Konto für den Bot (oder verwende ein bestehendes Konto).
2. Generiere Zugangsdaten: [Twitch Token Generator](https://twitchtokengenerator.com/)
   * Wähle **Bot Token**
   * Stelle sicher, dass die Scopes `chat:read` und `chat:write` ausgewählt sind
   * Kopiere die **Client-ID** und das **Access Token**
3. Ermittle deine Twitch-User-ID: https://www.streamweasels.com/tools/convert-twitch-username-to-user-id/
4. Konfiguriere das Token:
   * Umgebung: `OPENCLAW_TWITCH_ACCESS_TOKEN=...` (nur Standardkonto)
   * Oder Konfiguration: `channels.twitch.accessToken`
   * Wenn beides gesetzt ist, hat die Konfiguration Vorrang (die Umgebungsvariable dient nur als Fallback für das Standardkonto).
5. Starte das Gateway.

**⚠️ Wichtig:** Füge Zugriffskontrollen (`allowFrom` oder `allowedRoles`) hinzu, um zu verhindern, dass nicht autorisierte Benutzer den Bot triggern können. `requireMention` ist standardmäßig auf `true` gesetzt.

Minimale Konfiguration:

```json5
{
  channels: {
    twitch: {
      enabled: true,
      username: "openclaw",              // Bot's Twitch account
      accessToken: "oauth:abc123...",    // OAuth Access Token (or use OPENCLAW_TWITCH_ACCESS_TOKEN env var)
      clientId: "xyz789...",             // Client ID from Token Generator
      channel: "vevisk",                 // Which Twitch channel's chat to join (required)
      allowFrom: ["123456789"]           // (empfohlen) Nur Ihre Twitch-Benutzer-ID – erhalten Sie diese unter https://www.streamweasels.com/tools/convert-twitch-username-to-user-id/
    }
  }
}
```

<div id="what-it-is">
  ## Was es ist
</div>

* Ein Twitch-Kanal, der dem Gateway gehört.
* Deterministisches Routing: Antworten gehen immer zurück zu Twitch.
* Jedem Account wird ein isolierter Sitzungsschlüssel `agent:<agentId>:twitch:<accountName>` zugeordnet.
* `username` ist das Konto des Bots (das sich authentifiziert), `channel` ist der Chatraum, dem beigetreten werden soll.

<div id="setup-detailed">
  ## Ausführliche Einrichtung
</div>

<div id="generate-credentials">
  ### Zugangsdaten erzeugen
</div>

Verwende den [Twitch Token Generator](https://twitchtokengenerator.com/):

* Wähle **Bot Token**
* Stelle sicher, dass die Scopes `chat:read` und `chat:write` ausgewählt sind
* Kopiere die **Client ID** und das **Access Token**

Keine manuelle App-Registrierung notwendig. Die Token laufen nach einigen Stunden ab.

<div id="configure-the-bot">
  ### Bot konfigurieren
</div>

**Umgebungsvariable (nur Standardkonto):**

```bash
OPENCLAW_TWITCH_ACCESS_TOKEN=oauth:abc123...
```

**Oder per Konfiguration:**

```json5
{
  channels: {
    twitch: {
      enabled: true,
      username: "openclaw",
      accessToken: "oauth:abc123...",
      clientId: "xyz789...",
      channel: "vevisk"
    }
  }
}
```

Wenn sowohl env als auch config gesetzt sind, hat config Vorrang.

<div id="access-control-recommended">
  ### Zugriffskontrolle (empfohlen)
</div>

```json5
{
  channels: {
    twitch: {
      allowFrom: ["123456789"],       // (empfohlen) Nur Ihre Twitch-Benutzer-ID
      allowedRoles: ["moderator"]     // Oder Beschränkung auf Rollen
    }
  }
}
```

**Verfügbare Rollen:** `"moderator"`, `"owner"`, `"vip"`, `"subscriber"`, `"all"`.

**Warum User-IDs?** Benutzernamen können sich ändern, sodass sich andere als dich ausgeben können. User-IDs sind dauerhaft.

Finde deine Twitch-User-ID: https://www.streamweasels.com/tools/convert-twitch-username-%20to-user-id/ (Konvertiere deinen Twitch-Benutzernamen in eine ID)

<div id="token-refresh-optional">
  ## Token-Aktualisierung (optional)
</div>

Token vom [Twitch Token Generator](https://twitchtokengenerator.com/) können nicht automatisch aktualisiert werden – generiere sie nach Ablauf neu.

Für eine automatische Token-Aktualisierung erstelle in der [Twitch Developer Console](https://dev.twitch.tv/console) eine eigene Twitch-Anwendung und füge sie der Konfiguration hinzu:

```json5
{
  channels: {
    twitch: {
      clientSecret: "your_client_secret",
      refreshToken: "your_refresh_token"
    }
  }
}
```

Der Bot erneuert die Token automatisch, bevor sie ablaufen, und protokolliert die jeweiligen Aktualisierungsvorgänge.

<div id="multi-account-support">
  ## Multi-Account-Unterstützung
</div>

Verwende `channels.twitch.accounts` mit accountspezifischen Tokens. Siehe [`gateway/configuration`](/de/gateway/configuration) für das allgemeine Muster.

Beispiel (ein Bot-Account in zwei Kanälen):

```json5
{
  channels: {
    twitch: {
      accounts: {
        channel1: {
          username: "openclaw",
          accessToken: "oauth:abc123...",
          clientId: "xyz789...",
          channel: "vevisk"
        },
        channel2: {
          username: "openclaw",
          accessToken: "oauth:def456...",
          clientId: "uvw012...",
          channel: "secondchannel"
        }
      }
    }
  }
}
```

**Hinweis:** Für jedes Konto wird ein eigenes Token benötigt (ein Token pro Kanal).

<div id="access-control">
  ## Zugriffskontrolle
</div>

<div id="role-based-restrictions">
  ### Rollenbasierte Beschränkungen
</div>

```json5
{
  channels: {
    twitch: {
      accounts: {
        default: {
          allowedRoles: ["moderator", "vip"]
        }
      }
    }
  }
}
```

<div id="allowlist-by-user-id-most-secure">
  ### Allowlist anhand der Benutzer-ID (am sichersten)
</div>

```json5
{
  channels: {
    twitch: {
      accounts: {
        default: {
          allowFrom: ["123456789", "987654321"]
        }
      }
    }
  }
}
```

<div id="combined-allowlist-roles">
  ### Kombinierte Allowlist + Rollen
</div>

Benutzer in `allowFrom` umgehen die Rollenkontrolle:

```json5
{
  channels: {
    twitch: {
      accounts: {
        default: {
          allowFrom: ["123456789"],
          allowedRoles: ["moderator"]
        }
      }
    }
  }
}
```

<div id="disable-mention-requirement">
  ### Erfordernis einer @mention deaktivieren
</div>

Standardmäßig ist `requireMention` auf `true` gesetzt. Um das zu deaktivieren und auf alle Nachrichten zu antworten:

```json5
{
  channels: {
    twitch: {
      accounts: {
        default: {
          requireMention: false
        }
      }
    }
  }
}
```

<div id="troubleshooting">
  ## Fehlerbehebung
</div>

Führe zunächst Diagnosebefehle aus:

```bash
openclaw doctor
openclaw channels status --probe
```

<div id="bot-doesnt-respond-to-messages">
  ### Bot antwortet nicht auf Nachrichten
</div>

**Zugriffskontrolle prüfen:** Setze vorübergehend `allowedRoles: ["all"]`, um zu testen.

**Prüfen, ob der Bot im Kanal ist:** Der Bot muss dem in `channel` angegebenen Kanal beitreten.

<div id="token-issues">
  ### Tokenprobleme
</div>

**„Verbindung fehlgeschlagen“ oder Authentifizierungsfehler:**

* Überprüfe, dass `accessToken` der OAuth-Access-Token-Wert ist (beginnt in der Regel mit dem Präfix `oauth:`)
* Überprüfe, ob das Token die `chat:read`- und `chat:write`-Scopes hat
* Wenn Token-Refresh verwendet wird, stelle sicher, dass `clientSecret` und `refreshToken` gesetzt sind

<div id="token-refresh-not-working">
  ### Tokenaktualisierung funktioniert nicht
</div>

**Überprüfe die Logs auf Aktualisierungsereignisse:**

```
Using env token source for mybot
Access token refreshed for user 123456 (expires in 14400s)
```

Wenn dir „token refresh disabled (no refresh token)“ angezeigt wird:

* Stelle sicher, dass ein `clientSecret` angegeben ist
* Stelle sicher, dass ein `refreshToken` angegeben ist

<div id="config">
  ## Konfiguration
</div>

**Konto-Konfiguration:**

* `username` - Bot-Benutzername
* `accessToken` - OAuth-Zugriffstoken mit `chat:read` und `chat:write`
* `clientId` - Twitch-Client-ID (vom Token-Generator oder deiner App)
* `channel` - Channel, dem der Bot beitritt (erforderlich)
* `enabled` - Dieses Konto aktivieren (Standard: `true`)
* `clientSecret` - Optional: Für automatische Token-Erneuerung
* `refreshToken` - Optional: Für automatische Token-Erneuerung
* `expiresIn` - Token-Ablauf in Sekunden
* `obtainmentTimestamp` - Zeitstempel, zu dem das Token erhalten wurde
* `allowFrom` - Benutzer-ID-Allowlist
* `allowedRoles` - Rollenbasierte Zugriffskontrolle (`"moderator" | "owner" | "vip" | "subscriber" | "all"`)
* `requireMention` - @Mention erforderlich (Standard: `true`)

**Anbieteroptionen:**

* `channels.twitch.enabled` - Channel-Start aktivieren/deaktivieren
* `channels.twitch.username` - Bot-Benutzername (vereinfachte Einzelkonto-Konfiguration)
* `channels.twitch.accessToken` - OAuth-Zugriffstoken (vereinfachte Einzelkonto-Konfiguration)
* `channels.twitch.clientId` - Twitch-Client-ID (vereinfachte Einzelkonto-Konfiguration)
* `channels.twitch.channel` - Channel, dem der Bot beitritt (vereinfachte Einzelkonto-Konfiguration)
* `channels.twitch.accounts.<accountName>` - Mehrkonto-Konfiguration (alle obigen Konto-Felder)

Vollständiges Beispiel:

```json5
{
  channels: {
    twitch: {
      enabled: true,
      username: "openclaw",
      accessToken: "oauth:abc123...",
      clientId: "xyz789...",
      channel: "vevisk",
      clientSecret: "secret123...",
      refreshToken: "refresh456...",
      allowFrom: ["123456789"],
      allowedRoles: ["moderator", "vip"],
      accounts: {
        default: {
          username: "mybot",
          accessToken: "oauth:abc123...",
          clientId: "xyz789...",
          channel: "your_channel",
          enabled: true,
          clientSecret: "secret123...",
          refreshToken: "refresh456...",
          expiresIn: 14400,
          obtainmentTimestamp: 1706092800000,
          allowFrom: ["123456789", "987654321"],
          allowedRoles: ["moderator"]
        }
      }
    }
  }
}
```

<div id="tool-actions">
  ## Tool-Aktionen
</div>

Der agent kann `twitch` mit der folgenden Aktion aufrufen:

* `send` - sendet eine Nachricht an einen Kanal

Beispiel:

```json5
{
  "action": "twitch",
  "params": {
    "message": "Hello Twitch!",
    "to": "#mychannel"
  }
}
```

<div id="safety-ops">
  ## Sicherheit &amp; Betrieb
</div>

* **Behandle Token wie Passwörter** – Checke niemals Token in Git ein
* **Verwende automatische Token-Erneuerung** für länger laufende Bots
* **Verwende Allowlists für Benutzer-IDs** statt Benutzernamen zur Zugriffskontrolle
* **Überwache Logs** auf Token-Erneuerungen und Verbindungsstatus
* **Schränke Token-Berechtigungen auf das Nötigste ein** – fordere nur `chat:read` und `chat:write` an
* **Wenn du nicht weiterkommst**: Starte den Gateway neu, nachdem du bestätigt hast, dass kein anderer Prozess die Sitzung besitzt

<div id="limits">
  ## Limits
</div>

* **500 Zeichen** pro Nachricht (automatisch an Wortgrenzen in Chunks aufgeteilt)
* Markdown wird vor der Aufteilung entfernt
* Kein Rate-Limiting (verwendet die integrierten Rate-Limits von Twitch)