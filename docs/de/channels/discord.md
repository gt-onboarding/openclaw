---
title: Discord
summary: "Status, Fähigkeiten und Konfiguration der Discord-Bot-Integration"
read_when:
  - Arbeit an Funktionen des Discord-Kanals
---

<div id="discord-bot-api">
  # Discord (Bot API)
</div>

Status: Bereit für Direktnachrichten (DMs) und Textkanäle auf Servern über das offizielle Discord-Bot-Gateway.

<div id="quick-setup-beginner">
  ## Schnelleinrichtung (Einsteiger)
</div>

1. Erstelle einen Discord-Bot und kopiere den Bot-Token.
2. Aktiviere in den Discord-App-Einstellungen **Message Content Intent** (und **Server Members Intent**, wenn du Allowlists oder Namensabfragen verwenden möchtest).
3. Setze den Token für OpenClaw:
   * Env: `DISCORD_BOT_TOKEN=...`
   * Oder Config: `channels.discord.token: "..."`.
   * Wenn beides gesetzt ist, hat die Config Vorrang (Env-Fallback gilt nur für das Standardkonto).
4. Lade den Bot mit Nachrichtenberechtigungen auf deinen Server ein (erstelle einen privaten Server, wenn du nur DMs möchtest).
5. Starte das Gateway.
6. DM-Zugriff erfolgt standardmäßig per Kopplung; bestätige den Kopplungscode beim ersten Kontakt.

Minimale Konfiguration:

```json5
{
  channels: {
    discord: {
      enabled: true,
      token: "DEIN_BOT_TOKEN"
    }
  }
}
```

<div id="goals">
  ## Ziele
</div>

* Über Discord-DMs oder Guild-Kanäle mit OpenClaw kommunizieren.
* Direkte Chats werden in die Hauptsitzung des Agents zusammengeführt (Standard: `agent:main:main`); Guild-Kanäle bleiben isoliert als `agent:<agentId>:discord:channel:<channelId>` (Anzeigenamen verwenden `discord:<guildSlug>#<channelSlug>`).
* Gruppen-DMs werden standardmäßig ignoriert; aktiviere sie über `channels.discord.dm.groupEnabled` und beschränke sie optional über `channels.discord.dm.groupChannels`.
* Halte das Routing deterministisch: Antworten gehen immer in den Kanal zurück, in dem sie eingetroffen sind.

<div id="how-it-works">
  ## Funktionsweise
</div>

1. Erstelle eine Discord-Anwendung → Bot, aktiviere die benötigten Intents (DMs + Guild-Nachrichten + Nachrichteninhalt) und hole dir das Bot-Token.
2. Lade den Bot auf deinen Server mit den Berechtigungen ein, die erforderlich sind, um in den Kanälen, in denen du ihn verwenden willst, Nachrichten zu lesen/senden.
3. Konfiguriere OpenClaw mit `channels.discord.token` (oder `DISCORD_BOT_TOKEN` als Fallback).
4. Starte das Gateway; es startet den Discord-Kanal automatisch, sobald ein Token verfügbar ist (zuerst Konfiguration, dann Env-Fallback) und `channels.discord.enabled` nicht `false` ist.
   * Wenn du Umgebungsvariablen bevorzugst, setze `DISCORD_BOT_TOKEN` (ein Konfigurationsblock ist optional).
5. Direktchats: verwende `user:<id>` (oder eine `<@id>`-Erwähnung) bei der Zustellung; alle Dialogrunden landen in der gemeinsamen Sitzung `main`. Reine numerische IDs sind mehrdeutig und werden abgelehnt.
6. Guild-Kanäle: verwende `channel:<channelId>` für die Zustellung. Erwähnungen sind standardmäßig erforderlich und können pro Guild oder pro Kanal konfiguriert werden.
7. Direktchats: standardmäßig abgesichert über `channels.discord.dm.policy` (Standard: `"pairing"`). Unbekannte Absender erhalten einen Kopplungscode (läuft nach 1 Stunde ab); freigeben via `openclaw pairing approve discord <code>`.
   * Um das frühere Verhalten „für alle offen“ beizubehalten: setze `channels.discord.dm.policy="open"` (Einstellung `open` erlaubt uneingeschränkte Nachrichtenannahme von beliebigen Nutzern) und `channels.discord.dm.allowFrom=["*"]`.
   * Für eine strikte Allowlist: setze `channels.discord.dm.policy="allowlist"` und liste Absender in `channels.discord.dm.allowFrom` auf.
   * Um alle DMs zu ignorieren: setze `channels.discord.dm.enabled=false` oder `channels.discord.dm.policy="disabled"`.
8. Gruppen-DMs werden standardmäßig ignoriert; aktiviere sie über `channels.discord.dm.groupEnabled` und schränke sie optional mit `channels.discord.dm.groupChannels` ein.
9. Optionale Guild-Regeln: setze `channels.discord.guilds`, indiziert nach Guild-ID (bevorzugt) oder Slug, mit kanalbezogenen Regeln.
10. Optionale native Commands: `commands.native` ist standardmäßig `"auto"` (an für Discord/Telegram, aus für Slack). Überschreibe dies mit `channels.discord.commands.native: true|false|"auto"`; `false` entfernt zuvor registrierte Commands. Text-Commands werden über `commands.text` gesteuert und müssen als eigenständige `/...`-Nachrichten gesendet werden. Verwende `commands.useAccessGroups: false`, um Access-Group-Prüfungen für Commands zu umgehen.
    * Vollständige Command-Liste + Konfiguration: [Slash commands](/de/tools/slash-commands)
11. Optionale Guild-Kontexthistorie: setze `channels.discord.historyLimit` (Standard 20, fällt zurück auf `messages.groupChat.historyLimit`), um die letzten N Guild-Nachrichten als Kontext einzubinden, wenn auf eine Erwähnung geantwortet wird. Setze `0`, um dies zu deaktivieren.
12. Reaktionen: der Agent kann Reaktionen über das `discord`-Tool auslösen (gesteuert durch `channels.discord.actions.*`).
    * Semantik für das Entfernen von Reaktionen: siehe [/tools/reactions](/de/tools/reactions).
    * Das `discord`-Tool wird nur bereitgestellt, wenn der aktuelle Kanal Discord ist.
13. Native Commands verwenden isolierte Sitzungsschlüssel (`agent:<agentId>:discord:slash:<userId>`) statt der gemeinsamen Sitzung `main`.

Hinweis: Namens-→ID-Auflösung verwendet die Guild-Mitgliedersuche und erfordert den Servermitglieder-Intent; wenn der Bot keine Mitglieder suchen kann, verwende IDs oder `<@id>`-Erwähnungen.
Hinweis: Slugs sind kleingeschrieben, mit durch `-` ersetzten Leerzeichen. Kanalnamen werden ohne führendes `#` zu Slugs umgewandelt.
Hinweis: Guild-Kontextzeilen `[from:]` enthalten `author.tag` + `id`, um antwortfertige Pings zu erleichtern.

<div id="config-writes">
  ## Config-Schreibvorgänge
</div>

Standardmäßig ist es Discord erlaubt, Konfigurations-Updates zu schreiben, die durch `/config set|unset` ausgelöst werden (erfordert `commands.config: true`).

Deaktiviere dies mit:

```json5
{
  channels: { discord: { configWrites: false } }
}
```

<div id="how-to-create-your-own-bot">
  ## So erstellst du deinen eigenen Bot
</div>

Dies ist die Konfiguration im „Discord Developer Portal“, um OpenClaw in einem Serverkanal (Guild) wie `#help` auszuführen.

<div id="1-create-the-discord-app-bot-user">
  ### 1) Erstelle die Discord-App + den Bot-Benutzer
</div>

1. Discord Developer Portal → **Applications** → **New Application**
2. In deiner App:
   * **Bot** → **Add Bot**
   * Kopiere den **Bot-Token** (diesen trägst du in `DISCORD_BOT_TOKEN` ein)

<div id="2-enable-the-gateway-intents-openclaw-needs">
  ### 2) Aktiviere die Gateway-Intents, die OpenClaw benötigt
</div>

Discord blockiert „privilegierte Intents“, solange du sie nicht explizit aktivierst.

Unter **Bot** → **Privileged Gateway Intents** aktivierst du:

* **Message Content Intent** (erforderlich, um Nachrichtentext in den meisten Servern zu lesen; ohne diese Berechtigung siehst du „Used disallowed intents“ oder der Bot verbindet sich, reagiert aber nicht auf Nachrichten)
* **Server Members Intent** (empfohlen; erforderlich für bestimmte Mitglieds-/Benutzerabfragen und Allowlist-Abgleiche in Servern)

In der Regel benötigst du **Presence Intent** nicht.

<div id="3-generate-an-invite-url-oauth2-url-generator">
  ### 3) Eine Einladungs-URL generieren (OAuth2 URL Generator)
</div>

In deiner App: **OAuth2** → **URL Generator**

**Scopes**

* ✅ `bot`
* ✅ `applications.commands` (erforderlich für native Commands)

**Bot-Berechtigungen** (minimales Basis-Set)

* ✅ Kanäle anzeigen
* ✅ Nachrichten senden
* ✅ Nachrichtenverlauf lesen
* ✅ Links einbetten
* ✅ Dateien anhängen
* ✅ Reaktionen hinzufügen (optional, aber empfohlen)
* ✅ Externe Emojis / Sticker verwenden (optional; nur falls gewünscht)

Vermeide **Administrator**, außer wenn du debuggen musst und dem Bot vollständig vertraust.

Kopiere die generierte URL, öffne sie, wähle deinen Server aus und installiere den Bot.

<div id="4-get-the-ids-guilduserchannel">
  ### 4) IDs abrufen (Server/Benutzer/Channel)
</div>

Discord verwendet überall numerische IDs; die OpenClaw-Konfiguration bevorzugt IDs.

1. Discord (Desktop/Web) → **User Settings** → **Advanced** → **Developer Mode** aktivieren
2. Rechtsklick:
   * Servername → **Copy Server ID** (Guild-ID)
   * Channel (z. B. `#help`) → **Copy Channel ID**
   * Dein Benutzerkonto → **Copy User ID**

<div id="5-configure-openclaw">
  ### 5) OpenClaw konfigurieren
</div>

<div id="token">
  #### Token
</div>

Lege das Bot-Token als Umgebungsvariable fest (auf Servern empfohlen):

* `DISCORD_BOT_TOKEN=...`

Oder über die Konfiguration:

```json5
{
  channels: {
    discord: {
      enabled: true,
      token: "YOUR_BOT_TOKEN"
    }
  }
}
```

Unterstützung mehrerer Konten: Verwende `channels.discord.accounts` mit je einem Token pro Konto und optionalem `name`. Siehe [`gateway/configuration`](/de/gateway/configuration#telegramaccounts--discordaccounts--slackaccounts--signalaccounts--imessageaccounts) für das gemeinsame Muster.

<div id="allowlist-channel-routing">
  #### Allowlist + Channel-Routing
</div>

Beispiel „Einzelner Server, nur ich zugelassen, nur #help zugelassen“:

```json5
{
  channels: {
    discord: {
      enabled: true,
      dm: { enabled: false },
      guilds: {
        "YOUR_GUILD_ID": {
          users: ["YOUR_USER_ID"],
          requireMention: true,
          channels: {
            help: { allow: true, requireMention: true }
          }
        }
      },
      retry: {
        attempts: 3,
        minDelayMs: 500,
        maxDelayMs: 30000,
        jitter: 0.1
      }
    }
  }
}
```

Notes:

* `requireMention: true` bedeutet, dass der Bot nur bei Erwähnung antwortet (empfohlen für gemeinsam genutzte Channels).
* `agents.list[].groupChat.mentionPatterns` (oder `messages.groupChat.mentionPatterns`) zählen bei Guild-Nachrichten ebenfalls als Erwähnungen.
* Multi-Agent-Override: Lege agent-spezifische Muster in `agents.list[].groupChat.mentionPatterns` fest.
* Wenn `channels` vorhanden ist, wird jeder nicht aufgeführte Channel standardmäßig blockiert.
* Verwende einen `"*"`‑Channel-Eintrag, um Standardwerte auf alle Channels anzuwenden; explizite Channel-Einträge überschreiben das Wildcardsymbol.
* Threads erben die Konfiguration des übergeordneten Channels (Allowlist, `requireMention`, Fähigkeiten, Prompts usw.), sofern du die Thread-Channel-ID nicht explizit hinzufügst.
* Vom Bot verfasste Nachrichten werden standardmäßig ignoriert; setze `channels.discord.allowBots=true`, um sie zu erlauben (eigene Nachrichten bleiben gefiltert).
* Warnung: Wenn du Antworten auf andere Bots erlaubst (`channels.discord.allowBots=true`), verhindere Bot-zu-Bot-Antwortschleifen mit `requireMention`, `channels.discord.guilds.*.channels.<id>.users`‑Allowlists und/oder klar definierten Guardrails in `AGENTS.md` und `SOUL.md`.

<div id="6-verify-it-works">
  ### 6) Überprüfe, ob alles funktioniert
</div>

1. Starte das Gateway.
2. Sende in deinem Server-Channel: `@Krill hello` (oder wie auch immer dein Bot heißt).
3. Falls nichts passiert, siehe **Troubleshooting** weiter unten.

<div id="troubleshooting">
  ### Fehlerbehebung
</div>

* Zuerst: `openclaw doctor` und `openclaw channels status --probe` ausführen (konkrete Warnungen + schnelle Audits).
* **„Used disallowed intents“**: **Message Content Intent** (und vermutlich **Server Members Intent**) im Developer Portal aktivieren und anschließend das Gateway neu starten.
* **Bot verbindet sich, antwortet aber nie in einem Server-/Guild-Channel**:
  * Fehlende **Message Content Intent**, oder
  * Der Bot hat keine Kanalberechtigungen (View/Send/Read History), oder
  * Deine Konfiguration erfordert Erwähnungen und du hast ihn nicht erwähnt, oder
  * Deine Guild-/Channel-Allowlist verweigert dem Channel/User den Zugriff.
* **`requireMention: false`, aber trotzdem keine Antworten**:
* `channels.discord.groupPolicy` ist standardmäßig **allowlist**; setze sie auf `"open"` (unbeschränkte Annahme von Nachrichten von allen Nutzern) oder füge unter `channels.discord.guilds` einen Guild-Eintrag hinzu (optional kannst du unter `channels.discord.guilds.<id>.channels` Channels auflisten, um einzuschränken).
  * Wenn du nur `DISCORD_BOT_TOKEN` setzt und nie einen `channels.discord`-Abschnitt anlegst, setzt die Laufzeit
    `groupPolicy` standardmäßig auf `open` (unbeschränkte Annahme von Nachrichten von allen Nutzern). Füge `channels.discord.groupPolicy`,
    `channels.defaults.groupPolicy` oder eine Guild-/Channel-Allowlist hinzu, um sie abzusichern.
* `requireMention` muss unter `channels.discord.guilds` (oder einem spezifischen Channel) stehen. `channels.discord.requireMention` auf Top-Level-Ebene wird ignoriert.
* **Berechtigungs-Audits** (`channels status --probe`) prüfen nur numerische Channel-IDs. Wenn du Slugs/Namen als `channels.discord.guilds.*.channels`-Keys verwendest, kann das Audit die Berechtigungen nicht verifizieren.
* **DMs funktionieren nicht**: `channels.discord.dm.enabled=false`, `channels.discord.dm.policy="disabled"` oder du wurdest noch nicht freigeschaltet (`channels.discord.dm.policy="pairing"`).

<div id="capabilities-limits">
  ## Fähigkeiten &amp; Einschränkungen
</div>

* DMs und Textkanäle in Guilds (Threads werden als separate Kanäle behandelt; Voice wird nicht unterstützt).
* Tippindikatoren werden nach Best-Effort-Prinzip gesendet; Nachrichten-Chunks verwenden `channels.discord.textChunkLimit` (Standard 2000) und teilen lange Antworten anhand der Zeilenanzahl auf (`channels.discord.maxLinesPerMessage`, Standard 17).
* Optionales Chunking nach Leerzeilen: Setze `channels.discord.chunkMode="newline"`, um vor der Längenaufteilung an Leerzeilen (Absatzgrenzen) zu splitten.
* Datei-Uploads werden bis zur konfigurierten Grenze `channels.discord.mediaMaxMb` unterstützt (Standard 8 MB).
* Antworten in Guilds sind standardmäßig mention-gated, um laute Bots zu vermeiden.
* Antwortkontext wird eingefügt, wenn eine Nachricht auf eine andere Nachricht verweist (zitierter Inhalt + IDs).
* Native Reply-Threading ist **standardmäßig deaktiviert**; aktiviere es mit `channels.discord.replyToMode` und Reply-Tags.

<div id="retry-policy">
  ## Wiederholungsrichtlinie
</div>

Ausgehende Discord-API-Aufrufe werden bei Rate-Limit-Fehlern (429) mithilfe des Discord-Felds `retry_after` (falls verfügbar) mit exponentiellem Backoff und Jitter erneut gesendet. Konfiguriere dies über `channels.discord.retry`. Siehe [Wiederholungsrichtlinie](/de/concepts/retry).

<div id="config">
  ## Konfiguration
</div>

```json5
{
  channels: {
    discord: {
      enabled: true,
      token: "abc.123",
      groupPolicy: "allowlist",
      guilds: {
        "*": {
          channels: {
            general: { allow: true }
          }
        }
      },
      mediaMaxMb: 8,
      actions: {
        reactions: true,
        stickers: true,
        emojiUploads: true,
        stickerUploads: true,
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
        channels: true,
        voiceStatus: true,
        events: true,
        moderation: false
      },
      replyToMode: "off",
      dm: {
        enabled: true,
        policy: "pairing", // pairing | allowlist | open | disabled (Kopplung | Allowlist | offen | deaktiviert)
        allowFrom: ["123456789012345678", "steipete"],
        groupEnabled: false,
        groupChannels: ["openclaw-dm"]
      },
      guilds: {
        "*": { requireMention: true },
        "123456789012345678": {
          slug: "friends-of-openclaw",
          requireMention: false,
          reactionNotifications: "own",
          users: ["987654321098765432", "steipete"],
          channels: {
            general: { allow: true },
            help: {
              allow: true,
              requireMention: true,
              users: ["987654321098765432"],
              skills: ["search", "docs"],
              systemPrompt: "Keep answers short."
            }
          }
        }
      }
    }
  }
}
```

Ack-Reaktionen werden global über `messages.ackReaction` +
`messages.ackReactionScope` gesteuert. Verwende `messages.removeAckAfterReply`, um die
Ack-Reaktion zu entfernen, nachdem der Bot geantwortet hat.

* `dm.enabled`: auf `false` setzen, um alle DMs zu ignorieren (Standard `true`).
* `dm.policy`: DM-Zugriffskontrolle (`pairing` empfohlen). `"open"` erfordert `dm.allowFrom=["*"]`.
* `dm.allowFrom`: DM-Allowlist (User-IDs oder Namen). Wird von `dm.policy="allowlist"` sowie zur Validierung bei `dm.policy="open"` verwendet. Der Assistent akzeptiert Usernames und löst sie in IDs auf, wenn der Bot nach Mitgliedern suchen kann.
* `dm.groupEnabled`: Gruppen-DMs aktivieren (Standard `false`).
* `dm.groupChannels`: optionale Allowlist für Gruppen-DM-Channel-IDs oder -Slugs.
* `groupPolicy`: steuert das Handling von Guild-Channels (`open|disabled|allowlist`); `allowlist` erfordert Channel-Allowlists.
* `guilds`: Regeln pro Guild, indiziert nach Guild-ID (bevorzugt) oder Slug.
* `guilds."*"`: Standardeinstellungen pro Guild, die angewendet werden, wenn kein expliziter Eintrag existiert.
* `guilds.<id>.slug`: optionaler leserlicher Slug, der für Anzeigenamen verwendet wird.
* `guilds.<id>.users`: optionale User-Allowlist pro Guild (IDs oder Namen).
* `guilds.<id>.tools`: optionale Tool-Policy-Overrides pro Guild (`allow`/`deny`/`alsoAllow`), die verwendet werden, wenn der Channel-Override fehlt.
* `guilds.<id>.toolsBySender`: optionale Tool-Policy-Overrides pro Sender auf Guild-Ebene (greift, wenn der Channel-Override fehlt; `"*"`-Wildcard unterstützt).
* `guilds.<id>.channels.<channel>.allow`: Channel zulassen/ablehnen, wenn `groupPolicy="allowlist"`.
* `guilds.<id>.channels.<channel>.requireMention`: Mention-Gating für den Channel.
* `guilds.<id>.channels.<channel>.tools`: optionale Tool-Policy-Overrides pro Channel (`allow`/`deny`/`alsoAllow`).
* `guilds.<id>.channels.<channel>.toolsBySender`: optionale Tool-Policy-Overrides pro Sender innerhalb des Channels (`"*"`-Wildcard unterstützt).
* `guilds.<id>.channels.<channel>.users`: optionale User-Allowlist pro Channel.
* `guilds.<id>.channels.<channel>.skills`: Fähigkeiten-Filter (weglassen = alle Fähigkeiten, leer = keine).
* `guilds.<id>.channels.<channel>.systemPrompt`: zusätzlicher System-Prompt für den Channel (wird mit dem Channel-Thema kombiniert).
* `guilds.<id>.channels.<channel>.enabled`: auf `false` setzen, um den Channel zu deaktivieren.
* `guilds.<id>.channels`: Channel-Regeln (Keys sind Channel-Slugs oder -IDs).
* `guilds.<id>.requireMention`: Mention-Anforderung pro Guild (pro Channel überschreibbar).
* `guilds.<id>.reactionNotifications`: Modus für Reaction-Systemereignisse (`off`, `own`, `all`, `allowlist`).
* `textChunkLimit`: Größe ausgehender Textblöcke (Zeichen). Standard: 2000.
* `chunkMode`: `length` (Standard) teilt nur, wenn `textChunkLimit` überschritten wird; `newline` teilt an Leerzeilen (Absatzgrenzen) vor der Längen-basierten Aufteilung.
* `maxLinesPerMessage`: weiches Maximum an Zeilen pro Nachricht. Standard: 17.
* `mediaMaxMb`: eingehende Medien begrenzen, die auf die Festplatte gespeichert werden.
* `historyLimit`: Anzahl der letzten Guild-Nachrichten, die als Kontext einbezogen werden, wenn auf eine Mention geantwortet wird (Standard 20; Fallback auf `messages.groupChat.historyLimit`; `0` deaktiviert).
* `dmHistoryLimit`: DM-Verlaufsgrenze in Benutzer-Interaktionen. Overrides pro User: `dms["<user_id>"].historyLimit`.
* `retry`: Retry-Policy für ausgehende Discord-API-Aufrufe (attempts, minDelayMs, maxDelayMs, jitter).
* `actions`: Tool-Gates pro Aktion; weglassen, um alle zuzulassen (auf `false` setzen, um zu deaktivieren).
  * `reactions` (deckt Reactions + das Lesen von Reactions ab)
  * `stickers`, `emojiUploads`, `stickerUploads`, `polls`, `permissions`, `messages`, `threads`, `pins`, `search`
  * `memberInfo`, `roleInfo`, `channelInfo`, `voiceStatus`, `events`
  * `channels` (Channels + Kategorien + Berechtigungen erstellen/bearbeiten/löschen)
  * `roles` (Rollen hinzufügen/entfernen, Standard `false`)
  * `moderation` (timeout/kick/ban, Standard `false`)

Reaction-Benachrichtigungen verwenden `guilds.<id>.reactionNotifications`:

* `off`: keine Reaction-Events.
* `own`: Reactions auf die eigenen Nachrichten des Bots (Standard).
* `all`: alle Reactions auf alle Nachrichten.
* `allowlist`: Reactions von `guilds.<id>.users` auf alle Nachrichten (leere Liste deaktiviert).

<div id="tool-action-defaults">
  ### Standardwerte für Tool-Aktionen
</div>

| Aktionsgruppe | Standard | Hinweise |
| --- | --- | --- |
| reactions | enabled | Reagieren + Reaktionen auflisten + emojiList |
| stickers | enabled | Sticker senden |
| emojiUploads | enabled | Emojis hochladen |
| stickerUploads | enabled | Sticker hochladen |
| polls | enabled | Umfragen erstellen |
| permissions | enabled | Snapshot der Kanalberechtigungen |
| messages | enabled | Lesen/Senden/Bearbeiten/Löschen |
| threads | enabled | Erstellen/Auflisten/Antworten |
| pins | enabled | Anheften/Lösen/Auflisten |
| search | enabled | Nachrichtensuche (Vorschaufunktion) |
| memberInfo | enabled | Mitgliederinformationen |
| roleInfo | enabled | Rollenliste |
| channelInfo | enabled | Kanalinfos + Liste |
| channels | enabled | Kanal-/Kategorieverwaltung |
| voiceStatus | enabled | Sprachstatusabfrage |
| events | enabled | Geplante Events auflisten/erstellen |
| roles | disabled | Rollen hinzufügen/entfernen |
| moderation | disabled | Timeout/Kick/Ban |

* `replyToMode`: `off` (Standard), `first` oder `all`. Gilt nur, wenn das Modell einen Reply-Tag enthält.

<div id="reply-tags">
  ## Antwort-Tags
</div>

Um eine Antwort in einem Thread anzufordern, kann das Modell einen Tag in seine Ausgabe aufnehmen:

* `[[reply_to_current]]` — auf die auslösende Discord-Nachricht antworten.
* `[[reply_to:<id>]]` — auf eine bestimmte Nachrichten-ID aus Kontext/Verlauf antworten.
  Aktuelle Nachrichten-IDs werden den Prompts als `[message_id: …]` angehängt; Verlaufs­einträge enthalten IDs bereits.

Das Verhalten wird über `channels.discord.replyToMode` gesteuert:

* `off`: Tags ignorieren.
* `first`: Nur das erste ausgehende Chunk/Attachment ist eine Antwort.
* `all`: Jedes ausgehende Chunk/Attachment ist eine Antwort.

Hinweise zum Allowlist-Matching:

* `allowFrom`/`users`/`groupChannels` akzeptieren IDs, Namen, Tags oder Erwähnungen wie `<@id>`.
* Präfixe wie `discord:`/`user:` (Benutzer) und `channel:` (Gruppen-DMs) werden unterstützt.
* Verwende `*`, um jeden Absender/Kanal zu erlauben.
* Wenn `guilds.<id>.channels` vorhanden ist, werden nicht aufgeführte Kanäle standardmäßig verweigert.
* Wenn `guilds.<id>.channels` weggelassen wird, sind alle Kanäle in der in der Allowlist aufgeführten Guild erlaubt.
* Um **keine Kanäle** zu erlauben, setze `channels.discord.groupPolicy: "disabled"` (oder verwende eine leere Allowlist).
* Der Konfigurationsassistent akzeptiert `Guild/Channel`-Namen (öffentlich + privat) und löst sie nach Möglichkeit in IDs auf.
* Beim Start löst OpenClaw Kanal- und Benutzernamen in Allowlists in IDs auf (wenn der Bot Mitglieder durchsuchen kann)
  und protokolliert das Mapping; nicht auflösbare Einträge bleiben wie eingegeben erhalten.

Hinweise zu nativen Befehlen:

* Die registrierten Befehle spiegeln die Chat-Befehle von OpenClaw wider.
* Native Befehle respektieren dieselben Allowlists wie DM-/Guild-Nachrichten (`channels.discord.dm.allowFrom`, `channels.discord.guilds`, kanalbezogene Regeln).
* Slash-Commands können in der Discord-UI weiterhin für Benutzer sichtbar sein, die nicht in der Allowlist stehen; OpenClaw erzwingt Allowlists bei der Ausführung und antwortet mit „not authorized“.

<div id="tool-actions">
  ## Tool-Aktionen
</div>

Der agent kann `discord` mit folgenden Aktionen aufrufen:

* `react` / `reactions` (Reaktionen hinzufügen oder auflisten)
* `sticker`, `poll`, `permissions`
* `readMessages`, `sendMessage`, `editMessage`, `deleteMessage`
* Lese-/Such-/Pin-Tool-Payloads enthalten normalisierte `timestampMs` (UTC-Epoch-ms) und `timestampUtc` zusätzlich zum rohen Discord-`timestamp`.
* `threadCreate`, `threadList`, `threadReply`
* `pinMessage`, `unpinMessage`, `listPins`
* `searchMessages`, `memberInfo`, `roleInfo`, `roleAdd`, `roleRemove`, `emojiList`
* `channelInfo`, `channelList`, `voiceStatus`, `eventList`, `eventCreate`
* `timeout`, `kick`, `ban`

Discord-Nachrichten-IDs werden im injizierten Kontext bereitgestellt (`[discord message id: …]` und Verlaufszeilen), sodass der agent sie gezielt ansprechen kann.
Emojis können Unicode sein (z. B. `✅`) oder die benutzerdefinierte Emoji-Syntax `<:party_blob:1234567890>`.

<div id="safety-ops">
  ## Sicherheit &amp; Betrieb
</div>

* Behandle das Bot-Token wie ein Passwort; verwende nach Möglichkeit die Umgebungsvariable `DISCORD_BOT_TOKEN` auf verwalteten Hosts oder beschränke die Berechtigungen der Konfigurationsdatei.
* Gewähre dem Bot nur die Berechtigungen, die er benötigt (typischerweise Nachrichten lesen/senden).
* Wenn der Bot hängt oder durch ein Rate-Limit ausgebremst wird, starte das Gateway neu (`openclaw gateway --force`), nachdem du sichergestellt hast, dass keine anderen Prozesse die Discord-Sitzung verwenden.