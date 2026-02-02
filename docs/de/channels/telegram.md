---
title: Telegram
summary: "Status, Funktionsumfang und Konfiguration der Telegram-Bot-Integration"
read_when:
  - Beim Arbeiten an Telegram-Funktionen oder Webhooks
---

<div id="telegram-bot-api">
  # Telegram (Bot API)
</div>

Status: produktionsreif für Bot-DMs und Gruppen über grammY. Long Polling ist standardmäßig aktiviert; Webhooks optional.

<div id="quick-setup-beginner">
  ## Schnelleinrichtung (Einsteiger)
</div>

1. Erstelle mit **@BotFather** einen Bot und kopiere das Token.
2. Setze das Token:
   * Env: `TELEGRAM_BOT_TOKEN=...`
   * Oder Config: `channels.telegram.botToken: "..."`.
   * Wenn beides gesetzt ist, hat die Config Vorrang (Env-Fallback gilt nur für das Standardkonto).
3. Starte das Gateway.
4. DM-Zugriff erfolgt standardmäßig über Kopplung; bestätige den Kopplungscode beim ersten Kontakt.

Minimale Konfiguration:

```json5
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "123:abc",
      dmPolicy: "pairing"
    }
  }
}
```

<div id="what-it-is">
  ## Was es ist
</div>

* Ein Telegram Bot API-Kanal, der zum Gateway gehört.
* Deterministisches Routing: Antworten gehen zurück an Telegram; das Modell wählt niemals Kanäle.
* DMs teilen sich die Hauptsitzung des Agents; Gruppen bleiben isoliert (`agent:<agentId>:telegram:group:<chatId>`).

<div id="setup-fast-path">
  ## Einrichtung (Schnellstart)
</div>

<div id="1-create-a-bot-token-botfather">
  ### 1) Einen Bot-Token erstellen (BotFather)
</div>

1. Öffne Telegram und starte einen Chat mit **@BotFather**.
2. Führe `/newbot` aus und folge dann den Anweisungen (Name + Benutzername, der auf `bot` endet).
3. Kopiere den Token und bewahre ihn sicher auf.

Optionale BotFather-Einstellungen:

* `/setjoingroups` — steuern, ob der Bot zu Gruppen hinzugefügt werden darf.
* `/setprivacy` — steuern, ob der Bot alle Gruppennachrichten sehen kann.

<div id="2-configure-the-token-env-or-config">
  ### 2) Token konfigurieren (env oder Konfiguration)
</div>

Beispiel:

```json5
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "123:abc",
      dmPolicy: "pairing",
      groups: { "*": { requireMention: true } }
    }
  }
}
```

Env-Option: `TELEGRAM_BOT_TOKEN=...` (funktioniert für das Standardkonto).
Wenn sowohl Env als auch Config gesetzt sind, hat die Config Vorrang.

Multi-Account-Support: Verwende `channels.telegram.accounts` mit kontospezifischen Token und optionalem `name`. Siehe [`gateway/configuration`](/de/gateway/configuration#telegramaccounts--discordaccounts--slackaccounts--signalaccounts--imessageaccounts) für das gemeinsame Muster.

3. Starte das Gateway. Telegram startet, sobald ein Token ermittelt werden kann (zuerst Config, dann Env als Fallback).
4. DM-Zugriff ist standardmäßig auf Kopplung eingestellt. Genehmige den Code, wenn der Bot zum ersten Mal kontaktiert wird.
5. Für Gruppen: Füge den Bot hinzu, lege das Privacy-/Admin-Verhalten fest (unten) und setze dann `channels.telegram.groups`, um Mention-Gating + Allowlists zu steuern.

<div id="token-privacy-permissions-telegram-side">
  ## Token + Datenschutz + Berechtigungen (Telegram-seitig)
</div>

<div id="token-creation-botfather">
  ### Token-Erstellung (BotFather)
</div>

* `/newbot` erstellt den Bot und gibt das Token aus (halte es geheim).
* Wenn ein Token nach außen gelangt ist, widerrufe bzw. regeneriere es über @BotFather und aktualisiere deine Konfiguration.

<div id="group-message-visibility-privacy-mode">
  ### Sichtbarkeit von Gruppennachrichten (Privacy Mode)
</div>

Telegram-Bots verwenden standardmäßig den **Privacy Mode**, der einschränkt, welche Gruppennachrichten sie empfangen.
Wenn dein Bot *alle* Gruppennachrichten sehen muss, hast du zwei Optionen:

* Deaktiviere den Privacy Mode mit `/setprivacy` **oder**
* Füge den Bot als Gruppen‑**Admin** hinzu (Admin-Bots erhalten alle Nachrichten).

**Hinweis:** Wenn du den Privacy Mode änderst, verlangt Telegram, dass du den Bot
aus jeder Gruppe entfernst und anschließend erneut hinzufügst, damit die Änderung wirksam wird.

<div id="group-permissions-admin-rights">
  ### Gruppenberechtigungen (Admin-Rechte)
</div>

Der Admin-Status wird in der Gruppe (Telegram UI) festgelegt. Admin-Bots erhalten immer alle
Nachrichten aus der Gruppe, verwende also Admin-Rechte, wenn du vollständige Sichtbarkeit benötigst.

<div id="how-it-works-behavior">
  ## Funktionsweise (Verhalten)
</div>

* Eingehende Nachrichten werden in den gemeinsamen Channel-Umschlag mit Antwortkontext und Medienplatzhaltern normalisiert.
* Antworten in Gruppen erfordern standardmäßig eine Erwähnung (native @mention oder `agents.list[].groupChat.mentionPatterns` / `messages.groupChat.mentionPatterns`).
* Multi-Agent-Override: Lege pro agent Muster in `agents.list[].groupChat.mentionPatterns` fest.
* Antworten werden immer zurück in denselben Telegram-Chat geroutet.
* Long-Polling verwendet grammY runner mit Sequenzierung pro Chat; die Gesamtparallelität wird durch `agents.defaults.maxConcurrent` begrenzt.
* Die Telegram Bot API unterstützt keine Lesebestätigungen; es gibt keine `sendReadReceipts`-Option.

<div id="formatting-telegram-html">
  ## Formatierung (Telegram HTML)
</div>

* Ausgehende Telegram-Nachrichten verwenden `parse_mode: "HTML"` (Telegrams unterstützte Teilmenge der HTML-Tags).
* Markdown-ähnliche Eingaben werden in **Telegram-sicheres HTML** gerendert (fett/kursiv/durchgestrichen/Code/Links); Blockelemente werden zu Text mit Zeilenumbrüchen/Aufzählungspunkten „abgeflacht“.
* Rohes HTML aus Modellen wird escaped, um Parsefehler bei Telegram zu vermeiden.
* Wenn Telegram die HTML-Nutzlast ablehnt, sendet OpenClaw dieselbe Nachricht erneut als Klartext.

<div id="commands-native-custom">
  ## Befehle (eingebaute + benutzerdefinierte)
</div>

OpenClaw registriert eingebaute Befehle (wie `/status`, `/reset`, `/model`) beim Start im Bot-Menü von Telegram.
Du kannst dem Menü über die Konfiguration eigene Befehle hinzufügen:

```json5
{
  channels: {
    telegram: {
      customCommands: [
        { command: "backup", description: "Git backup" },
        { command: "generate", description: "Create an image" }
      ]
    }
  }
}
```

<div id="troubleshooting">
  ## Fehlerbehebung
</div>

* `setMyCommands failed` in den Logs/Protokollen bedeutet in der Regel, dass ausgehender HTTPS-/DNS-Verkehr zu `api.telegram.org` blockiert ist.
* Wenn du `sendMessage`- oder `sendChatAction`-Fehler siehst, überprüfe IPv6-Routing und DNS.

Weitere Hilfe: [Channel-Fehlerbehebung](/de/channels/troubleshooting).

Hinweise:

* Benutzerdefinierte Befehle sind **nur Menüeinträge**; OpenClaw führt sie nicht aus, es sei denn, du verarbeitest sie an anderer Stelle.
* Befehlsnamen werden normalisiert (führender `/` entfernt, in Kleinbuchstaben umgewandelt) und müssen `a-z`, `0-9`, `_` entsprechen (1–32 Zeichen lang).
* Benutzerdefinierte Befehle **können native Befehle nicht überschreiben**. Konflikte werden ignoriert und protokolliert.
* Wenn `commands.native` deaktiviert ist, werden nur benutzerdefinierte Befehle registriert (oder gelöscht, falls keine vorhanden sind).

<div id="limits">
  ## Limits
</div>

* Ausgehender Text wird in Blöcke von `channels.telegram.textChunkLimit` aufgeteilt (Standardwert 4000).
* Optionale Segmentierung nach Zeilenumbrüchen: Setze `channels.telegram.chunkMode="newline"`, um vor der Längen-Segmentierung an Leerzeilen (Absatzgrenzen) zu splitten.
* Medien-Downloads/-Uploads werden durch `channels.telegram.mediaMaxMb` begrenzt (Standardwert 5).
* Telegram-Bot-API-Anfragen laufen nach `channels.telegram.timeoutSeconds` ab (Standardwert 500 über grammY). Setze einen niedrigeren Wert, um lange Hänger zu vermeiden.
* Gruppenverlaufs-Kontext verwendet `channels.telegram.historyLimit` (oder `channels.telegram.accounts.*.historyLimit`) und fällt zurück auf `messages.groupChat.historyLimit`. Setze `0`, um zu deaktivieren (Standardwert 50).
* DM-Verlauf kann mit `channels.telegram.dmHistoryLimit` (Benutzer-Turns) begrenzt werden. Benutzerspezifische Overrides: `channels.telegram.dms["<user_id>"].historyLimit`.

<div id="group-activation-modes">
  ## Aktivierungsmodi für Gruppen
</div>

Standardmäßig reagiert der Bot in Gruppen nur auf Erwähnungen (`@botname` oder Muster in `agents.list[].groupChat.mentionPatterns`). Um dieses Verhalten zu ändern:

<div id="via-config-recommended">
  ### Über die Konfiguration (empfohlen)
</div>

```json5
{
  channels: {
    telegram: {
      groups: {
        "-1001234567890": { requireMention: false }  // immer in dieser Gruppe antworten
      }
    }
  }
}
```

**Wichtig:** Das Setzen von `channels.telegram.groups` definiert eine **Allowlist** – nur aufgeführte Gruppen (oder `"*"`) werden akzeptiert.
Forum-Themen erben die Konfiguration ihrer übergeordneten Gruppe (allowFrom, requireMention, skills, prompts), sofern du keine themenspezifischen Overrides unter `channels.telegram.groups.<groupId>.topics.<topicId>` definierst.

Um alle Gruppen mit `always-respond` zuzulassen:

```json5
{
  channels: {
    telegram: {
      groups: {
        "*": { requireMention: false }  // alle Gruppen, immer antworten
      }
    }
  }
}
```

Um für alle Gruppen im Modus „nur bei Erwähnung“ zu bleiben (Standardverhalten):

```json5
{
  channels: {
    telegram: {
      groups: {
        "*": { requireMention: true }  // oder groups ganz weglassen
      }
    }
  }
}
```

<div id="via-command-session-level">
  ### Per Befehl (Sitzungsebene)
</div>

Sende in der Gruppe:

* `/activation always` - auf alle Nachrichten antworten
* `/activation mention` - erfordert Erwähnungen (Standard)

**Hinweis:** Befehle ändern nur den Zustand der Sitzung. Für dauerhaftes Verhalten über Neustarts hinweg verwende die Konfiguration.

<div id="getting-the-group-chat-id">
  ### Abrufen der Gruppenchat-ID
</div>

Leite eine beliebige Nachricht aus der Gruppe an `@userinfobot` oder `@getidsbot` auf Telegram weiter, um die Chat-ID zu sehen (negative Zahl wie `-1001234567890`).

**Tipp:** Für deine eigene User-ID sende dem Bot eine Direktnachricht (DM), und er antwortet mit deiner User-ID (Kopplungsnachricht), oder verwende `/whoami`, sobald Befehle aktiviert sind.

**Hinweis zum Datenschutz:** `@userinfobot` ist ein Bot eines Drittanbieters. Falls dir das lieber ist, füge den Bot zur Gruppe hinzu, sende eine Nachricht und verwende `openclaw logs --follow`, um `chat.id` auszulesen, oder verwende die Bot API `getUpdates`.

<div id="config-writes">
  ## Config-Schreibzugriffe
</div>

Standardmäßig darf Telegram Config-Updates schreiben, die durch Channel-Ereignisse oder `/config set|unset` ausgelöst werden.

Das geschieht, wenn:

* Eine Gruppe zu einer Supergroup hochgestuft wird und Telegram `migrate_to_chat_id` sendet (Chat-ID ändert sich). OpenClaw kann `channels.telegram.groups` automatisch migrieren.
* Du `/config set` oder `/config unset` in einem Telegram-Chat ausführst (erfordert `commands.config: true`).

Deaktiviere dies mit:

```json5
{
  channels: { telegram: { configWrites: false } }
}
```

<div id="topics-forum-supergroups">
  ## Themen (Forum-Supergroups)
</div>

Telegram-Forumthemen enthalten eine `message_thread_id` pro Nachricht. OpenClaw:

* Hängt `:topic:<threadId>` an den Telegram-Gruppen-Sitzungsschlüssel an, sodass jedes Thema isoliert ist.
* Sendet Schreibindikatoren und Antworten mit `message_thread_id`, damit Antworten im Thema bleiben.
* Das allgemeine Thema (Thread-ID `1`) ist speziell: Beim Senden von Nachrichten wird `message_thread_id` weggelassen (Telegram lehnt sie sonst ab), Schreibindikatoren enthalten sie aber weiterhin.
* Stellt `MessageThreadId` + `IsForum` im Template-Kontext für Routing/Templating zur Verfügung.
* Themenspezifische Konfiguration ist unter `channels.telegram.groups.<chatId>.topics.<threadId>` verfügbar (Fähigkeiten, Allowlists, Auto-Reply, System-Prompts, Deaktivieren).
* Themenkonfigurationen erben die Gruppeneinstellungen (requireMention, Allowlists, Fähigkeiten, Prompts, enabled), sofern sie nicht pro Thema überschrieben werden.

Private Chats können in einigen Randfällen `message_thread_id` enthalten. OpenClaw lässt den DM-Sitzungsschlüssel unverändert, verwendet die Thread-ID aber weiterhin für Antworten und Entwurfs-Streaming, wenn sie vorhanden ist.

<div id="inline-buttons">
  ## Inline-Buttons
</div>

Telegram unterstützt Inline-Keyboards mit Callback-Buttons.

```json5
{
  "channels": {
    "telegram": {
      "capabilities": {
        "inlineButtons": "allowlist"
      }
    }
  }
}
```

Für die kontospezifische Konfiguration:

```json5
{
  "channels": {
    "telegram": {
      "accounts": {
        "main": {
          "capabilities": {
            "inlineButtons": "allowlist"
          }
        }
      }
    }
  }
}
```

Scopes:

* `off` — Inline-Buttons deaktiviert
* `dm` — nur DMs (Gruppenziele blockiert)
* `group` — nur Gruppen (DM-Ziele blockiert)
* `all` — DMs + Gruppen
* `allowlist` — DMs + Gruppen, aber nur Absender, die von `allowFrom`/`groupAllowFrom` erlaubt sind (dieselben Regeln wie für Steuerbefehle)

Standardwert: `allowlist`.
Alte Syntax: `capabilities: ["inlineButtons"]` = `inlineButtons: "all"`.

<div id="sending-buttons">
  ### Senden von Buttons
</div>

Verwende das Message-Tool mit dem Parameter `buttons`:

```json5
{
  "action": "send",
  "channel": "telegram",
  "to": "123456789",
  "message": "Choose an option:",
  "buttons": [
    [
      {"text": "Yes", "callback_data": "yes"},
      {"text": "No", "callback_data": "no"}
    ],
    [
      {"text": "Cancel", "callback_data": "cancel"}
    ]
  ]
}
```

Wenn ein Benutzer auf einen Button klickt, werden die Callback-Daten als Nachricht im folgenden Format an den agent zurückgesendet:
`callback_data: value`

<div id="configuration-options">
  ### Konfigurationsoptionen
</div>

Telegram-Capabilities können auf zwei Ebenen konfiguriert werden (Objektform oben gezeigt; ältere String-Arrays werden weiterhin unterstützt):

* `channels.telegram.capabilities`: Globale Standardkonfiguration, die auf alle Telegram-Konten angewendet wird, sofern sie nicht überschrieben wird.
* `channels.telegram.accounts.<account>.capabilities`: Kontospezifische Capabilities, die die globalen Standardwerte für dieses bestimmte Konto überschreiben.

Verwende die globale Einstellung, wenn sich alle Telegram-Bots und -Konten gleich verhalten sollen. Verwende die kontospezifische Konfiguration, wenn unterschiedliche Bots unterschiedliches Verhalten benötigen (zum Beispiel, wenn ein Konto nur DMs verarbeitet, während ein anderes in Gruppen verwendet werden darf).

<div id="access-control-dms-groups">
  ## Zugriffskontrolle (Direktnachrichten/DMs + Gruppen)
</div>

<div id="dm-access">
  ### DM-Zugriff
</div>

* Standard: `channels.telegram.dmPolicy = "pairing"`. Unbekannte Absender erhalten einen Kopplungscode; ihre Nachrichten werden ignoriert, bis sie genehmigt werden (Codes laufen nach 1 Stunde ab).
* Genehmigen über:
  * `openclaw pairing list telegram`
  * `openclaw pairing approve telegram <CODE>`
* Kopplung ist der standardmäßige Token-Austauschmechanismus, der für Telegram-DMs verwendet wird. Details: [Pairing](/de/start/pairing)
* `channels.telegram.allowFrom` akzeptiert numerische Benutzer-IDs (empfohlen) oder `@username`-Einträge. Es ist **nicht** der Bot-Benutzername; verwende die ID des menschlichen Absenders. Der Einrichtungsassistent akzeptiert `@username` und löst ihn nach Möglichkeit in die numerische ID auf.

<div id="finding-your-telegram-user-id">
  #### Deine Telegram-Benutzer-ID finden
</div>

Sicherer (kein Drittanbieter-Bot):

1. Starte das Gateway und schreib deinem Bot eine Direktnachricht (DM).
2. Führe `openclaw logs --follow` aus und suche nach `from.id`.

Alternative (offizielle Bot API):

1. Schick deinem Bot eine Direktnachricht (DM).
2. Rufe Updates mit deinem Bot-Token ab und lies `message.from.id`:
   ```bash
   curl "https://api.telegram.org/bot<bot_token>/getUpdates"
   ```

Drittanbieter (weniger privat):

* Schick eine Direktnachricht (DM) an `@userinfobot` oder `@getidsbot` und verwende die zurückgegebene Benutzer-ID.

<div id="group-access">
  ### Gruppenzugriff
</div>

Zwei unabhängige Kontrollen:

**1. Welche Gruppen erlaubt sind** (Gruppen-Allowlist über `channels.telegram.groups`):

* Keine `groups`-Konfiguration = alle Gruppen erlaubt
* Mit `groups`-Konfiguration = nur aufgelistete Gruppen oder `"*"` sind erlaubt
* Beispiel: `"groups": { "-1001234567890": {}, "*": {} }` erlaubt alle Gruppen

**2. Welche Absender erlaubt sind** (Absender-Filterung über `channels.telegram.groupPolicy`):

* `"open"` = alle Absender in erlaubten Gruppen können Nachrichten senden (diese Einstellung erlaubt uneingeschränkte Nachrichtenannahme von allen Nutzern in diesen Gruppen)
* `"allowlist"` = nur Absender in `channels.telegram.groupAllowFrom` können Nachrichten senden
* `"disabled"` = es werden keinerlei Gruppennachrichten akzeptiert
  Standard ist `groupPolicy: "allowlist"` (alles blockiert, bis du `groupAllowFrom` hinzufügst).

Die meisten Nutzer wollen: `groupPolicy: "allowlist"` + `groupAllowFrom` + spezifische Gruppen, die in `channels.telegram.groups` aufgeführt sind

<div id="long-polling-vs-webhook">
  ## Long-Polling vs. Webhook
</div>

* Standard: Long-Polling (keine öffentliche URL erforderlich).
* Webhook-Modus: Setze `channels.telegram.webhookUrl` (optional `channels.telegram.webhookSecret` + `channels.telegram.webhookPath`).
  * Der lokale Listener bindet an `0.0.0.0:8787` und bedient standardmäßig `POST /telegram-webhook`.
  * Wenn deine öffentliche URL abweicht, verwende einen Reverse-Proxy und setze `channels.telegram.webhookUrl` auf den öffentlichen Endpunkt.

<div id="reply-threading">
  ## Antwort-Threads
</div>

Telegram unterstützt optionale Antworten in Threads über Tags:

* `[[reply_to_current]]` -- Antwort auf die auslösende Nachricht.
* `[[reply_to:<id>]]` -- Antwort auf eine bestimmte Nachrichten-ID.

Gesteuert über `channels.telegram.replyToMode`:

* `first` (Standard), `all`, `off`.

<div id="audio-messages-voice-vs-file">
  ## Audio-Nachrichten (Sprachnachricht vs. Datei)
</div>

Telegram unterscheidet **Sprachnachrichten** (runde Blase) von **Audiodateien** (Metadatenkarte).
OpenClaw verwendet standardmäßig Audiodateien für die Abwärtskompatibilität.

Um in Agent-Antworten eine Sprachnachrichten-Blase zu erzwingen, füge dieses Tag an beliebiger Stelle in die Antwort ein:

* `[[audio_as_voice]]` — Audio als Sprachnachricht statt als Datei senden.

Das Tag wird aus dem ausgelieferten Text entfernt. Andere Kanäle ignorieren dieses Tag.

Für Senden über das Nachrichten-Tool setze `asVoice: true` mit einer für Sprachnachrichten geeigneten Audio-`media`-URL
(`message` ist optional, wenn `media` vorhanden ist):

```json5
{
  "action": "send",
  "channel": "telegram",
  "to": "123456789",
  "media": "https://example.com/voice.ogg",
  "asVoice": true
}
```

<div id="stickers">
  ## Sticker
</div>

OpenClaw unterstützt das Empfangen und Versenden von Telegram-Stickern mit intelligentem Caching.

<div id="receiving-stickers">
  ### Empfang von Stickern
</div>

Wenn ein Benutzer einen Sticker sendet, verarbeitet OpenClaw ihn abhängig vom Sticker-Typ:

* **Statische Sticker (WEBP):** Wird heruntergeladen und von Vision verarbeitet. Der Sticker erscheint als `<media:sticker>`-Platzhalter im Nachrichteninhalt.
* **Animierte Sticker (TGS):** Übersprungen (Lottie-Format wird für die Verarbeitung nicht unterstützt).
* **Video-Sticker (WEBM):** Übersprungen (Videoformat wird für die Verarbeitung nicht unterstützt).

Template-Kontextfeld, das beim Empfang von Stickern verfügbar ist:

* `Sticker` — Objekt mit:
  * `emoji` — Emoji, das dem Sticker zugeordnet ist
  * `setName` — Name des Sticker-Sets
  * `fileId` — Telegram-Datei-ID (denselben Sticker zurücksenden)
  * `fileUniqueId` — stabile ID für Cache-Suchen
  * `cachedDescription` — zwischengespeicherte Vision-Beschreibung, falls verfügbar

<div id="sticker-cache">
  ### Sticker-Cache
</div>

Sticker werden über die Bildverarbeitungsfunktionen der KI verarbeitet, um Beschreibungen zu erzeugen. Da dieselben Sticker häufig wiederholt gesendet werden, speichert OpenClaw diese Beschreibungen im Cache, um redundante API-Aufrufe zu vermeiden.

**Funktionsweise:**

1. **Erste Begegnung:** Das Stickerbild wird zur Bildanalyse an die KI gesendet. Die KI erzeugt eine Beschreibung (z. B. &quot;A cartoon cat waving enthusiastically&quot;).
2. **Cache-Speicherung:** Die Beschreibung wird zusammen mit der Datei-ID des Stickers, dem Emoji und dem Set-Namen gespeichert.
3. **Weitere Vorkommen:** Wenn derselbe Sticker erneut erkannt wird, wird direkt die zwischengespeicherte Beschreibung verwendet. Das Bild wird nicht erneut an die KI gesendet.

**Cache-Speicherort:** `~/.openclaw/telegram/sticker-cache.json`

**Format eines Cache-Eintrags:**

```json
{
  "fileId": "CAACAgIAAxkBAAI...",
  "fileUniqueId": "AgADBAADb6cxG2Y",
  "emoji": "👋",
  "setName": "CoolCats",
  "description": "Eine Cartoon-Katze, die begeistert winkt",
  "cachedAt": "2026-01-15T10:30:00.000Z"
}
```

**Vorteile:**

* Reduziert API-Kosten, indem wiederholte Vision-Aufrufe für denselben Sticker vermieden werden
* Schnellere Antwortzeiten für zwischengespeicherte Sticker (keine Vision-Verarbeitungsverzögerung)
* Ermöglicht eine Sticker-Suchfunktion auf Basis der zwischengespeicherten Beschreibungen

Der Cache wird automatisch aufgebaut, sobald Sticker empfangen werden. Es ist keine manuelle Cache-Verwaltung erforderlich.

<div id="sending-stickers">
  ### Senden von Stickern
</div>

Der agent kann Sticker mit den Aktionen `sticker` und `sticker-search` senden und suchen. Diese sind standardmäßig deaktiviert und müssen in der Konfiguration aktiviert werden:

```json5
{
  channels: {
    telegram: {
      actions: {
        sticker: true
      }
    }
  }
}
```

**Einen Sticker senden:**

```json5
{
  "action": "sticker",
  "channel": "telegram",
  "to": "123456789",
  "fileId": "CAACAgIAAxkBAAI..."
}
```

Parameter:

* `fileId` (erforderlich) — die Telegram-Datei-ID des Stickers. Diese erhältst du aus `Sticker.fileId`, wenn du einen Sticker empfängst, oder aus einem `sticker-search`-Ergebnis.
* `replyTo` (optional) — Nachrichten-ID, auf die geantwortet werden soll.
* `threadId` (optional) — Nachrichtenthread-ID für Forenthemen.

**Nach Stickern suchen:**

Der Agent kann zwischengespeicherte Sticker nach Beschreibung, Emoji oder Setnamen durchsuchen:

```json5
{
  "action": "sticker-search",
  "channel": "telegram",
  "query": "cat waving",
  "limit": 5
}
```

Gibt übereinstimmende Sticker aus dem Cache zurück:

```json5
{
  "ok": true,
  "count": 2,
  "stickers": [
    {
      "fileId": "CAACAgIAAxkBAAI...",
      "emoji": "👋",
      "description": "Eine Cartoon-Katze, die begeistert winkt",
      "setName": "CoolCats"
    }
  ]
}
```

Die Suche verwendet Fuzzy-Matching für Beschreibungstexte, Emojis und Set-Namen.

**Beispiel mit Threading:**

```json5
{
  "action": "sticker",
  "channel": "telegram",
  "to": "-1001234567890",
  "fileId": "CAACAgIAAxkBAAI...",
  "replyTo": 42,
  "threadId": 123
}
```

<div id="streaming-drafts">
  ## Streaming (Entwürfe)
</div>

Telegram kann **Entwurfsblasen** streamen, während der Agent eine Antwort generiert.
OpenClaw verwendet die Bot API `sendMessageDraft` (keine echten Nachrichten) und sendet
dann die endgültige Antwort als normale Nachricht.

Voraussetzungen (Telegram Bot API 9.3+):

* **Private Chats mit aktivierten Themen** (Forum-Themenmodus für den Bot).
* Eingehende Nachrichten müssen `message_thread_id` enthalten (privater Themen-Thread).
* Streaming wird für Gruppen/Supergruppen/Kanäle ignoriert.

Konfiguration:

* `channels.telegram.streamMode: "off" | "partial" | "block"` (Standard: `partial`)
  * `partial`: aktualisiert die Entwurfsblase mit dem neuesten Streaming-Text.
  * `block`: aktualisiert die Entwurfsblase in größeren Blöcken (chunked).
  * `off`: deaktiviert Entwurfs-Streaming.
* Optional (nur für `streamMode: "block"`):
  * `channels.telegram.draftChunk: { minChars?, maxChars?, breakPreference? }`
    * Standardwerte: `minChars: 200`, `maxChars: 800`, `breakPreference: "paragraph"` (begrenzt durch `channels.telegram.textChunkLimit`).

Hinweis: Entwurfs-Streaming ist getrennt von **Block-Streaming** (Kanal-Nachrichten).
Block-Streaming ist standardmäßig deaktiviert und erfordert `channels.telegram.blockStreaming: true`,
wenn du frühe Telegram-Nachrichten anstelle von Entwurfs-Updates erhalten möchtest.

Reasoning-Stream (nur Telegram):

* `/reasoning stream` streamt das Reasoning in die Entwurfsblase, während die Antwort
  generiert wird, und sendet dann die endgültige Antwort ohne Reasoning.
* Wenn `channels.telegram.streamMode` auf `off` steht, ist der Reasoning-Stream deaktiviert.
  Mehr Kontext: [Streaming + chunking](/de/concepts/streaming).

<div id="retry-policy">
  ## Retry-Richtlinie
</div>

Ausgehende Aufrufe der Telegram-API werden bei vorübergehenden Netzwerk- bzw. 429-Fehlern mit exponentiellem Backoff und Jitter automatisch wiederholt. Konfiguriere dies über `channels.telegram.retry`. Siehe [Retry-Richtlinie](/de/concepts/retry).

<div id="agent-tool-messages-reactions">
  ## Agent-Tool (Nachrichten + Reaktionen)
</div>

* Tool: `telegram` mit `sendMessage`-Aktion (`to`, `content`, optional `mediaUrl`, `replyToMessageId`, `messageThreadId`).
* Tool: `telegram` mit `react`-Aktion (`chatId`, `messageId`, `emoji`).
* Tool: `telegram` mit `deleteMessage`-Aktion (`chatId`, `messageId`).
* Semantik der Entfernung von Reaktionen: siehe [/tools/reactions](/de/tools/reactions).
* Tool-Gating: `channels.telegram.actions.reactions`, `channels.telegram.actions.sendMessage`, `channels.telegram.actions.deleteMessage` (Standard: aktiviert) und `channels.telegram.actions.sticker` (Standard: deaktiviert).

<div id="reaction-notifications">
  ## Reaktionsbenachrichtigungen
</div>

**So funktionieren Reaktionen:**
Telegram-Reaktionen werden als **separate `message_reaction`-Events** übermittelt, nicht als Eigenschaften in Nachrichten-Payloads. Wenn ein Nutzer eine Reaktion hinzufügt, führt OpenClaw Folgendes aus:

1. Empfängt das `message_reaction`-Update von der Telegram API
2. Wandelt es in ein **System-Event** mit folgendem Format um: `"Telegram reaction added: {emoji} by {user} on msg {id}"`
3. Stellt das System-Event mit demselben **Sitzungsschlüssel** wie reguläre Nachrichten in die Warteschlange
4. Wenn die nächste Nachricht in dieser Unterhaltung eintrifft, werden die System-Events aus der Warteschlange entnommen und dem Kontext des agents vorangestellt

Der agent sieht Reaktionen als **Systembenachrichtigungen** im Gesprächsverlauf, nicht als Nachrichten-Metadaten.

**Konfiguration:**

* `channels.telegram.reactionNotifications`: Steuert, welche Reaktionen Benachrichtigungen auslösen
  * `"off"` — alle Reaktionen ignorieren
  * `"own"` — benachrichtigen, wenn Nutzer auf Bot-Nachrichten reagieren (Best-Effort; im Speicher) (Standard)
  * `"all"` — für alle Reaktionen benachrichtigen

* `channels.telegram.reactionLevel`: Steuert die Reaktionsfähigkeit des agents
  * `"off"` — agent kann nicht auf Nachrichten reagieren
  * `"ack"` — Bot sendet Bestätigungsreaktionen (👀 während der Verarbeitung) (Standard)
  * `"minimal"` — agent kann sparsam reagieren (Richtwert: 1 pro 5–10 Austausche)
  * `"extensive"` — agent kann großzügig reagieren, wenn es sinnvoll ist

**Forengruppen:** Reaktionen in Forengruppen enthalten `message_thread_id` und verwenden Sitzungsschlüssel wie `agent:main:telegram:group:{chatId}:topic:{threadId}`. Dadurch bleiben Reaktionen und Nachrichten im selben Thema zusammen.

**Beispielkonfiguration:**

```json5
{
  channels: {
    telegram: {
      reactionNotifications: "all",  // Alle Reaktionen sehen
      reactionLevel: "minimal"        // Agent kann sparsam reagieren
    }
  }
}
```

**Voraussetzungen:**

* Telegram-Bots müssen `message_reaction` explizit in `allowed_updates` angeben (wird von OpenClaw automatisch konfiguriert)
* Im Webhook-Modus sind Reaktionen in den Webhook-`allowed_updates` enthalten
* Im Polling-Modus sind Reaktionen in den `getUpdates`-`allowed_updates` enthalten

<div id="delivery-targets-clicron">
  ## Zustellziele (CLI/cron)
</div>

* Verwende eine Chat-ID (`123456789`) oder einen Benutzernamen (`@name`) als Ziel.
* Beispiel: `openclaw message send --channel telegram --target 123456789 --message "hi"`.

<div id="troubleshooting">
  ## Fehlerbehebung
</div>

**Bot reagiert nicht auf Nachrichten ohne Erwähnung in einer Gruppe:**

* Wenn du `channels.telegram.groups.*.requireMention=false` gesetzt hast, muss der **Privacy Mode** der Telegram Bot API deaktiviert sein.
  * BotFather: `/setprivacy` → **Disable** (dann den Bot aus der Gruppe entfernen und erneut hinzufügen)
* `openclaw channels status` zeigt eine Warnung an, wenn die Konfiguration Nachrichten in Gruppen ohne Erwähnung erwartet.
* `openclaw channels status --probe` kann zusätzlich die Mitgliedschaft für explizite numerische Gruppen-IDs prüfen (Wildcard-Regeln `"*"` können nicht geprüft werden).
* Schneller Test: `/activation always` (nur für die Sitzung; für persistentes Verhalten Konfiguration verwenden)

**Bot sieht überhaupt keine Gruppennachrichten:**

* Wenn `channels.telegram.groups` gesetzt ist, muss die Gruppe aufgeführt sein oder `"*"` verwenden.
* Überprüfe die Datenschutzeinstellungen in @BotFather → „Group Privacy“ sollte **OFF** sein.
* Prüfe, ob der Bot tatsächlich Mitglied ist (nicht nur ein Admin ohne Lesezugriff).
* Prüfe die Gateway-Logs: `openclaw logs --follow` (suche nach „skipping group message“).

**Bot reagiert auf Erwähnungen, aber nicht auf `/activation always`:**

* Der Befehl `/activation` aktualisiert den Sitzungszustand, wird aber nicht in der Konfiguration persistiert.
* Für dauerhaftes Verhalten füge die Gruppe zu `channels.telegram.groups` mit `requireMention: false` hinzu.

**Befehle wie `/status` funktionieren nicht:**

* Stelle sicher, dass deine Telegram-User-ID autorisiert ist (per Kopplung oder `channels.telegram.allowFrom`).
* Befehle erfordern Autorisierung, selbst in Gruppen mit `groupPolicy: "open"`.

**Long-Polling bricht auf Node 22+ sofort ab (oft mit Proxies/custom fetch):**

* Node 22+ ist strenger mit `AbortSignal`-Instanzen; externe Signale können `fetch`-Aufrufe sofort abbrechen.
* Aktualisiere auf ein OpenClaw-Build, das Abort-Signale normalisiert, oder betreibe das Gateway auf Node 20, bis du aktualisieren kannst.

**Bot startet und hört dann stillschweigend auf zu reagieren (oder loggt `HttpError: Network request ... failed`):**

* Manche Hosts lösen `api.telegram.org` zuerst zu IPv6 auf. Wenn dein Server keinen funktionierenden IPv6-Egress hat, kann grammY bei ausschließlich IPv6-Anfragen hängen bleiben.
* Behebe das, indem du entweder IPv6-Egress aktivierst **oder** IPv4-Auflösung für `api.telegram.org` erzwingst (zum Beispiel durch einen Eintrag in `/etc/hosts` mit dem IPv4-A-Record oder indem du in deinem OS-DNS-Stack IPv4 bevorzugst), und starte dann das Gateway neu.
* Schnelle Prüfung: `dig +short api.telegram.org A` und `dig +short api.telegram.org AAAA`, um zu prüfen, was die DNS-Abfrage zurückgibt.

<div id="configuration-reference-telegram">
  ## Konfigurationsreferenz (Telegram)
</div>

Vollständige Konfiguration: [Konfiguration](/de/gateway/configuration)

Anbieteroptionen:

* `channels.telegram.enabled`: Aktivierung/Deaktivierung des Kanal-Starts.
* `channels.telegram.botToken`: Bot-Token (BotFather).
* `channels.telegram.tokenFile`: Token aus Dateipfad lesen.
* `channels.telegram.dmPolicy`: `pairing | allowlist | open | disabled` (Standard: pairing).
* `channels.telegram.allowFrom`: DM-Allowlist (IDs/Usernames). `open` erfordert `"*"`.
* `channels.telegram.groupPolicy`: `open | allowlist | disabled` (Standard: allowlist).
* `channels.telegram.groupAllowFrom`: Allowlist für Gruppensender (IDs/Usernames).
* `channels.telegram.groups`: gruppenbezogene Defaults + Allowlist (verwende `"*"` für globale Defaults).
  * `channels.telegram.groups.<id>.requireMention`: Standard für Mention-Gating.
  * `channels.telegram.groups.<id>.skills`: Fähigkeiten-Filter (weglassen = alle Fähigkeiten, leer = keine).
  * `channels.telegram.groups.<id>.allowFrom`: gruppenspezifischer Override der Sender-Allowlist.
  * `channels.telegram.groups.<id>.systemPrompt`: zusätzlicher Systemprompt für die Gruppe.
  * `channels.telegram.groups.<id>.enabled`: deaktiviert die Gruppe, wenn `false`.
  * `channels.telegram.groups.<id>.topics.<threadId>.*`: themenbezogene Overrides (gleiche Felder wie Gruppe).
  * `channels.telegram.groups.<id>.topics.<threadId>.requireMention`: themenbezogener Override für Mention-Gating.
* `channels.telegram.capabilities.inlineButtons`: `off | dm | group | all | allowlist` (Standard: allowlist).
* `channels.telegram.accounts.<account>.capabilities.inlineButtons`: kontospezifischer Override.
* `channels.telegram.replyToMode`: `off | first | all` (Standard: `first`).
* `channels.telegram.textChunkLimit`: Größe ausgehender Chunks (Zeichen).
* `channels.telegram.chunkMode`: `length` (Standard) oder `newline`, um vor dem Längen-Chunken an Leerzeilen (Absatzgrenzen) zu splitten.
* `channels.telegram.linkPreview`: Link-Vorschauen für ausgehende Nachrichten umschalten (Standard: true).
* `channels.telegram.streamMode`: `off | partial | block` (Entwurfs-Streaming).
* `channels.telegram.mediaMaxMb`: Limit für ein- und ausgehende Medien (MB).
* `channels.telegram.retry`: Retry-Richtlinie für ausgehende Telegram-API-Aufrufe (attempts, minDelayMs, maxDelayMs, jitter).
* `channels.telegram.network.autoSelectFamily`: Override von Node autoSelectFamily (true=aktivieren, false=deaktivieren). Standardmäßig auf Node 22 deaktiviert, um Happy-Eyeballs-Timeouts zu vermeiden.
* `channels.telegram.proxy`: Proxy-URL für Bot-API-Aufrufe (SOCKS/HTTP).
* `channels.telegram.webhookUrl`: Webhook-Modus aktivieren.
* `channels.telegram.webhookSecret`: Webhook-Secret (optional).
* `channels.telegram.webhookPath`: lokaler Webhook-Pfad (Standard `/telegram-webhook`).
* `channels.telegram.actions.reactions`: Steuerung von Telegram-Tool-Reaktionen.
* `channels.telegram.actions.sendMessage`: Steuerung von Telegram-Tool-Nachrichtensendungen.
* `channels.telegram.actions.deleteMessage`: Steuerung von Telegram-Tool-Nachrichtenlöschungen.
* `channels.telegram.actions.sticker`: Steuerung von Telegram-Sticker-Aktionen — Senden und Suchen (Standard: false).
* `channels.telegram.reactionNotifications`: `off | own | all` — steuert, welche Reaktionen Systemereignisse auslösen (Standard: `own`, wenn nicht gesetzt).
* `channels.telegram.reactionLevel`: `off | ack | minimal | extensive` — steuert die Reaktionsfähigkeit des Agents (Standard: `minimal`, wenn nicht gesetzt).

Zugehörige globale Optionen:

* `agents.list[].groupChat.mentionPatterns` (Mention-Gating-Muster).
* `messages.groupChat.mentionPatterns` (globaler Fallback).
* `commands.native` (Standard ist `"auto"` → an für Telegram/Discord, aus für Slack), `commands.text`, `commands.useAccessGroups` (Befehlsverhalten). Override mit `channels.telegram.commands.native`.
* `messages.responsePrefix`, `messages.ackReaction`, `messages.ackReactionScope`, `messages.removeAckAfterReply`.