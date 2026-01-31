---
title: iMessage
summary: "iMessage-Unterstützung über imsg (JSON-RPC über stdio), Einrichtung sowie chat_id-Routing"
read_when:
  - Einrichten der iMessage-Unterstützung
  - Fehlerbehebung beim Senden/Empfangen von iMessage
---

<div id="imessage-imsg">
  # iMessage (imsg)
</div>

Status: Externe CLI-Integration. Gateway startet `imsg rpc` (JSON-RPC über stdio).

<div id="quick-setup-beginner">
  ## Schnelleinrichtung (Anfänger)
</div>

1. Stelle sicher, dass du in Nachrichten auf diesem Mac angemeldet bist.
2. Installiere `imsg`:
   * `brew install steipete/tap/imsg`
3. Konfiguriere OpenClaw mit `channels.imessage.cliPath` und `channels.imessage.dbPath`.
4. Starte das Gateway und bestätige alle macOS‑Aufforderungen (Automation + Vollzugriff auf die Festplatte).

Minimale Konfiguration:

```json5
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "/usr/local/bin/imsg",
      dbPath: "/Users/<you>/Library/Messages/chat.db"
    }
  }
}
```

<div id="what-it-is">
  ## Was es ist
</div>

* iMessage-Kanal, der auf `imsg` unter macOS basiert.
* Deterministisches Routing: Antworten gehen immer zurück zu iMessage.
* DMs teilen sich die Hauptsitzung des Agents; Gruppen sind isoliert (`agent:<agentId>:imessage:group:<chat_id>`).
* Wenn ein Thread mit mehreren Teilnehmern mit `is_group=false` ankommt, kannst du ihn trotzdem per `chat_id` über `channels.imessage.groups` isolieren (siehe „Gruppenartige Threads“ unten).

<div id="config-writes">
  ## Config-Schreibvorgänge
</div>

Standardmäßig ist iMessage berechtigt, Konfigurationsänderungen zu schreiben, die durch `/config set|unset` ausgelöst werden (erfordert `commands.config: true`).

Deaktiviere das mit:

```json5
{
  channels: { imessage: { configWrites: false } }
}
```

<div id="requirements">
  ## Anforderungen
</div>

* macOS mit angemeldetem Messages-Account.
* Vollzugriff auf die Festplatte für OpenClaw + `imsg` (Zugriff auf die Messages-Datenbank).
* Automatisierungsberechtigung zum Senden.
* `channels.imessage.cliPath` kann auf jeden beliebigen Befehl zeigen, der stdin/stdout durchreicht (zum Beispiel ein Wrapper-Skript, das per SSH auf einen anderen Mac zugreift und `imsg rpc` ausführt).

<div id="setup-fast-path">
  ## Einrichtung (Schnellstart)
</div>

1. Stelle sicher, dass du auf diesem Mac in „Nachrichten“ angemeldet bist.
2. Konfiguriere iMessage und starte das Gateway.

<div id="dedicated-bot-macos-user-for-isolated-identity">
  ### Dedizierter Bot-macOS-Benutzer (für isolierte Identität)
</div>

Wenn du möchtest, dass der Bot von einer **separaten iMessage-Identität** sendet (und deine persönlichen Nachrichten sauber bleiben sollen), verwende eine eigene Apple-ID plus einen dedizierten macOS-Benutzer.

1. Erstelle eine dedizierte Apple-ID (Beispiel: `my-cool-bot@icloud.com`).
   * Apple kann für die Verifizierung/2FA eine Telefonnummer verlangen.
2. Erstelle einen macOS-Benutzer (Beispiel: `openclawhome`) und melde dich damit an.
3. Öffne Nachrichten in diesem macOS-Benutzer und melde dich mit der Bot-Apple-ID bei iMessage an.
4. Aktiviere Remote Login (Systemeinstellungen → Allgemein → Freigaben → Entfernte Anmeldung/Remote Login).
5. Installiere `imsg`:
   * `brew install steipete/tap/imsg`
6. Richte SSH so ein, dass `ssh <bot-macos-user>@localhost true` ohne Passwort funktioniert.
7. Setze `channels.imessage.accounts.bot.cliPath` auf einen SSH-Wrapper, der `imsg` als Bot-Benutzer ausführt.

Hinweis zum ersten Start: Das Senden und Empfangen kann GUI-Bestätigungen (Automation + Vollzugriff auf die Festplatte) im *Bot-macOS-Benutzer* erfordern. Wenn `imsg rpc` hängen bleibt oder beendet wird, melde dich bei diesem Benutzer an (Screen Sharing hilft), führe einmalig `imsg chats --limit 1` / `imsg send ...` aus, bestätige alle Dialoge und versuche es dann erneut.

Beispiel-Wrapper (`chmod +x`). Ersetze `<bot-macos-user>` durch deinen tatsächlichen macOS-Benutzernamen:

```bash
#!/usr/bin/env bash
set -euo pipefail

# Führen Sie zunächst einmal interaktiv SSH aus, um Host-Schlüssel zu akzeptieren:
#   ssh <bot-macos-user>@localhost true
exec /usr/bin/ssh -o BatchMode=yes -o ConnectTimeout=5 -T <bot-macos-user>@localhost \
  "/usr/local/bin/imsg" "$@"
```

Beispielkonfiguration:

```json5
{
  channels: {
    imessage: {
      enabled: true,
      accounts: {
        bot: {
          name: "Bot",
          enabled: true,
          cliPath: "/path/to/imsg-bot",
          dbPath: "/Users/<bot-macos-user>/Library/Messages/chat.db"
        }
      }
    }
  }
}
```

Für Single-Account-Setups verwendest du flache Optionen (`channels.imessage.cliPath`, `channels.imessage.dbPath`) statt der `accounts`-Map.

<div id="remotessh-variant-optional">
  ### Remote/SSH-Variante (optional)
</div>

Wenn du iMessage auf einem anderen Mac verwenden möchtest, setze `channels.imessage.cliPath` auf einen Wrapper, der `imsg` auf dem entfernten macOS-Host über SSH ausführt. OpenClaw benötigt lediglich die Standard-Ein-/Ausgabe (stdio).

Beispiel-Wrapper:

```bash
#!/usr/bin/env bash
exec ssh -T gateway-host imsg "$@"
```

**Remote-Anhänge:** Wenn `cliPath` über SSH auf einen Remote-Host verweist, zeigen die Anhangspfade in der Messages-Datenbank auf Dateien auf dem Remote-Computer. OpenClaw kann diese automatisch per SCP abrufen, indem du `channels.imessage.remoteHost` setzt:

```json5
{
  channels: {
    imessage: {
      cliPath: "~/imsg-ssh",                     // SSH-Wrapper für entfernten Mac
      remoteHost: "user@gateway-host",           // for SCP file transfer
      includeAttachments: true
    }
  }
}
```

Wenn `remoteHost` nicht gesetzt ist, versucht OpenClaw, ihn automatisch zu ermitteln, indem der SSH-Befehl in Ihrem Wrapper-Skript geparst wird. Für ein zuverlässiges Verhalten wird eine explizite Konfiguration empfohlen.

<div id="remote-mac-via-tailscale-example">
  #### Remote-Mac über Tailscale (Beispiel)
</div>

Wenn das Gateway auf einem Linux-Host bzw. in einer VM läuft, iMessage aber auf einem Mac ausgeführt werden muss, ist Tailscale die einfachste Brücke: Das Gateway kommuniziert über das Tailnet mit dem Mac, führt `imsg` per SSH aus und kopiert Anhänge per SCP zurück.

Architektur:

```
┌──────────────────────────────┐          SSH (imsg rpc)          ┌──────────────────────────┐
│ Gateway host (Linux/VM)      │──────────────────────────────────▶│ Mac with Messages + imsg │
│ - openclaw gateway           │          SCP (attachments)        │ - Messages signed in     │
│ - channels.imessage.cliPath  │◀──────────────────────────────────│ - Remote Login enabled   │
└──────────────────────────────┘                                   └──────────────────────────┘
              ▲
              │ Tailscale tailnet (hostname or 100.x.y.z)
              ▼
        user@gateway-host
```

Konkretes Beispiel für eine Konfiguration (Tailscale-Hostname):

```json5
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "~/.openclaw/scripts/imsg-ssh",
      remoteHost: "bot@mac-mini.tailnet-1234.ts.net",
      includeAttachments: true,
      dbPath: "/Users/bot/Library/Messages/chat.db"
    }
  }
}
```

Beispiel-Wrapper (`~/.openclaw/scripts/imsg-ssh`):

```bash
#!/usr/bin/env bash
exec ssh -T bot@mac-mini.tailnet-1234.ts.net imsg "$@"
```

Hinweise:

* Stelle sicher, dass der Mac in der Nachrichten-App angemeldet ist und „Remote Login“ aktiviert ist.
* Verwende SSH-Schlüssel, damit `ssh bot@mac-mini.tailnet-1234.ts.net` ohne Eingabeaufforderungen funktioniert.
* `remoteHost` sollte dem SSH-Ziel entsprechen, damit `scp` Anhänge abrufen kann.

Unterstützung für mehrere Accounts: Verwende `channels.imessage.accounts` mit Account-spezifischer Konfiguration und optionalem `name`. Siehe [`gateway/configuration`](/de/gateway/configuration#telegramaccounts--discordaccounts--slackaccounts--signalaccounts--imessageaccounts) für das gemeinsame Muster. Checke `~/.openclaw/openclaw.json` nicht ein (die Datei enthält häufig Token).

<div id="access-control-dms-groups">
  ## Zugriffskontrolle (DMs + Gruppen)
</div>

DMs:

* Standard: `channels.imessage.dmPolicy = "pairing"`.
* Unbekannte Absender erhalten einen Kopplungscode; Nachrichten werden bis zur Freigabe ignoriert (Codes verfallen nach 1 Stunde).
* Freigabe über:
  * `openclaw pairing list imessage`
  * `openclaw pairing approve imessage <CODE>`
* Kopplung ist der Standard-Tokenaustausch für iMessage-DMs. Details: [Kopplung](/de/start/pairing)

Gruppen:

* `channels.imessage.groupPolicy = open | allowlist | disabled`. `open` erlaubt die uneingeschränkte Annahme von Nachrichten von beliebigen Nutzern.
* `channels.imessage.groupAllowFrom` steuert, wer in Gruppen triggern darf, wenn `allowlist` gesetzt ist.
* Mention-Gating verwendet `agents.list[].groupChat.mentionPatterns` (oder `messages.groupChat.mentionPatterns`), da iMessage keine nativen Mention-Metadaten hat.
* Multi-Agent-Override: Lege agent-spezifische Muster in `agents.list[].groupChat.mentionPatterns` fest.

<div id="how-it-works-behavior">
  ## Funktionsweise (Verhalten)
</div>

* `imsg` streamt Nachrichten-Events; das Gateway normalisiert sie in ein gemeinsames Channel-Envelope-Format.
* Antworten werden immer an dieselbe Chat-ID bzw. denselben Handle zurückgeroutet.

<div id="group-ish-threads-is_groupfalse">
  ## Gruppenähnliche Threads (`is_group=false`)
</div>

Einige iMessage-Threads können mehrere Teilnehmer haben, gehen aber trotzdem mit `is_group=false` ein, abhängig davon, wie Messages die Chat-ID speichert.

Wenn du explizit eine `chat_id` unter `channels.imessage.groups` konfigurierst, behandelt OpenClaw diesen Thread als „Gruppe“ für:

* Sitzungsisolierung (separater Sitzungsschlüssel `agent:<agentId>:imessage:group:<chat_id>`)
* Gruppen-Allowlisting / Mention-Gating-Verhalten

Beispiel:

```json5
{
  channels: {
    imessage: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15555550123"],
      groups: {
        "42": { "requireMention": false }
      }
    }
  }
}
```

Das ist hilfreich, wenn du für einen bestimmten Thread eine isolierte Persönlichkeit bzw. ein isoliertes Modell benötigst (siehe [Multi-Agent-Routing](/de/concepts/multi-agent)). Für eine Isolierung des Dateisystems siehe [Sandboxing](/de/gateway/sandboxing).

<div id="media-limits">
  ## Medien + Grenzwerte
</div>

* Optionale Verarbeitung von Anhängen über `channels.imessage.includeAttachments`.
* Medienobergrenze über `channels.imessage.mediaMaxMb`.

<div id="limits">
  ## Grenzwerte
</div>

* Ausgehender Text wird in Blöcke von `channels.imessage.textChunkLimit` (Standard 4000) aufgeteilt.
* Optionales Chunking nach Zeilenumbrüchen: Setze `channels.imessage.chunkMode="newline"`, um an Leerzeilen (Absatzgrenzen) aufzuteilen, bevor eine längenbasierte Aufteilung erfolgt.
* Medien-Uploads sind durch `channels.imessage.mediaMaxMb` (Standard 16) begrenzt.

<div id="addressing-delivery-targets">
  ## Adressierung / Zieladressen
</div>

Verwende `chat_id` für stabiles Routing:

* `chat_id:123` (bevorzugt)
* `chat_guid:...`
* `chat_identifier:...`
* direkte Handles: `imessage:+1555` / `sms:+1555` / `user@example.com`

Chats auflisten:

```
imsg chats --limit 20
```

<div id="configuration-reference-imessage">
  ## Konfigurationsreferenz (iMessage)
</div>

Vollständige Konfiguration: [Konfiguration](/de/gateway/configuration)

Anbieteroptionen:

* `channels.imessage.enabled`: Kanalstart aktivieren/deaktivieren.
* `channels.imessage.cliPath`: Pfad zu `imsg`.
* `channels.imessage.dbPath`: Nachrichten-DB-Pfad.
* `channels.imessage.remoteHost`: SSH-Host für die Übertragung von Anhängen per SCP, wenn `cliPath` auf einen entfernten Mac zeigt (z. B. `user@gateway-host`). Wird aus dem SSH-Wrapper automatisch erkannt, falls nicht gesetzt.
* `channels.imessage.service`: `imessage | sms | auto`.
* `channels.imessage.region`: SMS-Region.
* `channels.imessage.dmPolicy`: `pairing | allowlist | open | disabled` (Standard: pairing).
* `channels.imessage.allowFrom`: DM-Allowlist (Handles, E-Mail-Adressen, E.164-Nummern oder `chat_id:*`). `open` erfordert `"*"`. iMessage hat keine Benutzernamen; verwenden Sie Handles oder Chat-Ziele.
* `channels.imessage.groupPolicy`: `open | allowlist | disabled` (Standard: allowlist).
* `channels.imessage.groupAllowFrom`: Allowlist für Gruppen-Absender.
* `channels.imessage.historyLimit` / `channels.imessage.accounts.*.historyLimit`: maximale Anzahl von Gruppennachrichten, die als Kontext einbezogen werden (0 deaktiviert).
* `channels.imessage.dmHistoryLimit`: Limit für die DM-Historie in Benutzer-Turns. Überschreibungen pro Benutzer: `channels.imessage.dms["<handle>"].historyLimit`.
* `channels.imessage.groups`: gruppenbezogene Defaults + Allowlist (verwenden Sie `"*"` für globale Defaults).
* `channels.imessage.includeAttachments`: Anhänge in den Kontext aufnehmen.
* `channels.imessage.mediaMaxMb`: maximale Größe eingehender/ausgehender Medien (MB).
* `channels.imessage.textChunkLimit`: Größe ausgehender Chunks (Zeichen).
* `channels.imessage.chunkMode`: `length` (Standard) oder `newline`, um vor der Aufteilung nach Länge an Leerzeilen (Absatzgrenzen) zu trennen.

Zugehörige globale Optionen:

* `agents.list[].groupChat.mentionPatterns` (oder `messages.groupChat.mentionPatterns`).
* `messages.responsePrefix`.