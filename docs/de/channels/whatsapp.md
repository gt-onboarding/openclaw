---
title: WhatsApp
summary: "WhatsApp-Webkanal-Integration: Anmeldung, Posteingang, Antworten, Medien und Betrieb"
read_when:
  - Wenn du am Verhalten des WhatsApp-Webkanals oder am Inbox-Routing arbeitest
---

<div id="whatsapp-web-channel">
  # WhatsApp (Web-Kanal)
</div>

Status: WhatsApp Web nur über Baileys. Gateway verwaltet die Sitzung(en).

<div id="quick-setup-beginner">
  ## Schnelleinrichtung (Einsteiger)
</div>

1. Verwende wenn möglich eine **separate Telefonnummer** (empfohlen).
2. Konfiguriere WhatsApp in `~/.openclaw/openclaw.json`.
3. Führe `openclaw channels login` aus, um den QR-Code zu scannen (Verknüpfte Geräte).
4. Starte das Gateway.

Minimale Konfiguration:

```json5
{
  channels: {
    whatsapp: {
      dmPolicy: "allowlist",
      allowFrom: ["+15551234567"]
    }
  }
}
```

<div id="goals">
  ## Ziele
</div>

* Mehrere WhatsApp-Konten (Multi-Account) in einem einzigen Gateway-Prozess.
* Deterministisches Routing: Antworten gehen zurück an WhatsApp, kein Model-Routing.
* Das Modell erhält genügend Kontext, um zitierte Antworten zu verstehen.

<div id="config-writes">
  ## Config-Schreibvorgänge
</div>

Standardmäßig darf WhatsApp Config-Updates schreiben, die durch `/config set|unset` ausgelöst werden (erfordert `commands.config: true`).

Deaktivieren mit:

```json5
{
  channels: { whatsapp: { configWrites: false } }
}
```

<div id="architecture-who-owns-what">
  ## Architektur (wem gehört was)
</div>

* **Gateway** hält den Baileys-Socket und die Inbox-Schleife.
* **CLI / macOS-App** kommunizieren mit dem Gateway; keine direkte Baileys-Nutzung.
* Ein **aktiver Listener** ist für ausgehende Send-Vorgänge erforderlich; andernfalls schlagen diese sofort fehl.

<div id="getting-a-phone-number-two-modes">
  ## Eine Telefonnummer beschaffen (zwei Modi)
</div>

WhatsApp erfordert für die Verifizierung eine echte Mobilfunknummer. VoIP- und virtuelle Nummern werden in der Regel blockiert. Es gibt zwei unterstützte Möglichkeiten, OpenClaw mit WhatsApp zu betreiben:

<div id="dedicated-number-recommended">
  ### Eigene Nummer (empfohlen)
</div>

Verwende eine **separate Telefonnummer** für OpenClaw. Bestes UX, sauberes Routing, keine Eigenheiten durch Selbst-Chats. Ideales Setup: **zusätzliches/altes Android‑Telefon + eSIM**. Lass es über WLAN und am Strom dauerhaft laufen und verbinde es per QR-Code.

**WhatsApp Business:** Du kannst WhatsApp Business auf demselben Gerät mit einer anderen Nummer nutzen. Ideal, um dein persönliches WhatsApp getrennt zu halten — installiere WhatsApp Business und registriere dort die OpenClaw-Nummer.

**Beispielkonfiguration (eigene Nummer, Single-User-Allowlist):**

```json5
{
  channels: {
    whatsapp: {
      dmPolicy: "allowlist",
      allowFrom: ["+15551234567"]
    }
  }
}
```

**Kopplungsmodus (optional):**
Wenn du Kopplung statt einer Allowlist verwenden möchtest, setze `channels.whatsapp.dmPolicy` auf `pairing`. Unbekannte Absender erhalten einen Kopplungscode; genehmigst du mit:
`openclaw pairing approve whatsapp <code>`

<div id="personal-number-fallback">
  ### Persönliche Nummer (Fallback)
</div>

Schneller Fallback: Lass OpenClaw auf **deiner eigenen Nummer** laufen. Schreibe dir selbst (WhatsApp „Message yourself“) zu Testzwecken, damit du keine Kontakte zuspammst. Rechne damit, während der Einrichtung und bei Experimenten Verifizierungscodes auf deinem Haupttelefon lesen zu müssen. **Der Selbst-Chat-Modus muss aktiviert sein.**
Wenn der Assistent nach deiner persönlichen WhatsApp-Nummer fragt, gib die Nummer des Telefons ein, von dem du Nachrichten sendest (Besitzer/Absender), nicht die Assistenten-Nummer.

**Beispielkonfiguration (persönliche Nummer, Selbst-Chat):**

```json
{
  "whatsapp": {
    "selfChatMode": true,
    "dmPolicy": "allowlist",
    "allowFrom": ["+15551234567"]
  }
}
```

Antworten in Self-Chats verwenden standardmäßig `[{identity.name}]`, falls gesetzt (ansonsten `[openclaw]`), wenn `messages.responsePrefix` nicht gesetzt ist. Setze `messages.responsePrefix` explizit, um das Präfix anzupassen oder zu deaktivieren (verwende `""`, um es zu entfernen).

<div id="number-sourcing-tips">
  ### Tipps zur Nummernbeschaffung
</div>

* **Lokale eSIM** von einem Mobilfunkanbieter in deinem Land (am zuverlässigsten)
  * Österreich: [hot.at](https://www.hot.at)
  * UK: [giffgaff](https://www.giffgaff.com) — kostenlose SIM-Karte, kein Vertrag
* **Prepaid-SIM-Karte** — günstig, muss nur eine SMS zur Verifizierung empfangen können

**Vermeide:** TextNow, Google Voice, die meisten „Free-SMS“-Dienste — WhatsApp blockiert diese sehr aggressiv.

**Tipp:** Die Nummer muss nur eine Verifizierungs-SMS empfangen. Danach bleiben WhatsApp-Web-Sitzungen über `creds.json` bestehen.

<div id="why-not-twilio">
  ## Warum nicht Twilio?
</div>

* Frühe OpenClaw-Versionen unterstützten Twilios WhatsApp-Business-Integration.
* WhatsApp-Business-Nummern eignen sich schlecht für einen persönlichen Assistenten.
* Meta erzwingt eine 24‑Stunden-Antwortfrist; wenn du in den letzten 24 Stunden nicht geantwortet hast, kann die Business-Nummer keine neuen Nachrichten initiieren.
* Nutzung mit hohem Nachrichtenaufkommen oder besonders „chatty“ Nutzung führt zu aggressiven Sperren, weil Business-Konten nicht dafür gedacht sind, Dutzende von Nachrichten eines persönlichen Assistenten zu senden.
* Ergebnis: unzuverlässige Zustellung und häufige Sperren, daher wurde die Unterstützung entfernt.

<div id="login-credentials">
  ## Login + Zugangsdaten
</div>

* Login-Befehl: `openclaw channels login` (QR-Code über „Verknüpfte Geräte“).
* Multi-Account-Login: `openclaw channels login --account <id>` (`<id>` = `accountId`).
* Standardkonto (wenn `--account` weggelassen wird): `default`, falls vorhanden, sonst die erste konfigurierte Account-ID (sortiert).
* Zugangsdaten werden in `~/.openclaw/credentials/whatsapp/<accountId>/creds.json` gespeichert.
* Sicherungskopie in `creds.json.bak` (wird bei Beschädigung wiederhergestellt).
* Legacy-Kompatibilität: Ältere Installationen speicherten Baileys-Dateien direkt in `~/.openclaw/credentials/`.
* Logout: `openclaw channels logout` (oder `--account <id>`) löscht den WhatsApp-Auth-Status (behält aber die gemeinsame `oauth.json`).
* Abgemeldete Socket-Verbindung =&gt; Fehler mit Aufforderung zur erneuten Verknüpfung.

<div id="inbound-flow-dm-group">
  ## Eingehender Flow (DM + Gruppe)
</div>

* WhatsApp-Events kommen von `messages.upsert` (Baileys).
* Inbox-Listener werden beim Herunterfahren getrennt, um das Ansammeln von Event-Handlern in Tests/Neustarts zu vermeiden.
* Status-/Broadcast-Chats werden ignoriert.
* Direktchats verwenden E.164; Gruppen verwenden die Gruppen-JID.
* **DM-Policy**: `channels.whatsapp.dmPolicy` steuert den Zugriff auf Direktchats (Standard: `pairing`).
  * Kopplung: Unbekannte Absender erhalten einen Kopplungscode (Freigabe per `openclaw pairing approve whatsapp <code>`; Codes laufen nach 1 Stunde ab).
  * open: erfordert, dass `channels.whatsapp.allowFrom` `"*"` enthält (diese Einstellung erlaubt die unbegrenzte Annahme eingehender Nachrichten von beliebigen Nutzern).
  * Deine verknüpfte WhatsApp-Nummer wird implizit als vertrauenswürdig behandelt, daher überspringen eigene Nachrichten die Prüfungen von `channels.whatsapp.dmPolicy` und `channels.whatsapp.allowFrom`.

<div id="personal-number-mode-fallback">
  ### Modus für persönliche Nummer (Fallback)
</div>

Wenn du OpenClaw mit deiner **persönlichen WhatsApp-Nummer** betreibst, aktiviere `channels.whatsapp.selfChatMode` (siehe Beispiel oben).

Verhalten:

* Ausgehende DMs lösen niemals Kopplungsantworten aus (verhindert Spam an Kontakte).
* Eingehende DMs von unbekannten Absendern folgen weiterhin `channels.whatsapp.dmPolicy`.
* Self-Chat-Modus (allowFrom enthält deine Nummer) vermeidet automatische Lesebestätigungen und ignoriert Mention-JIDs.
* Lesebestätigungen werden für DMs außerhalb des Self-Chat-Modus gesendet.

<div id="read-receipts">
  ## Lesebestätigungen
</div>

Standardmäßig markiert das Gateway eingehende WhatsApp-Nachrichten als gelesen (blaue Haken), sobald sie akzeptiert werden.

Global deaktivieren:

```json5
{
  channels: { whatsapp: { sendReadReceipts: false } }
}
```

Für jedes Konto deaktivieren:

```json5
{
  channels: {
    whatsapp: {
      accounts: {
        personal: { sendReadReceipts: false }
      }
    }
  }
}
```

Hinweise:

* Im Self-Chat-Modus werden niemals Lesebestätigungen gesendet.

<div id="whatsapp-faq-sending-messages-pairing">
  ## WhatsApp-FAQ: Nachrichten senden + Kopplung
</div>

**Wird OpenClaw zufällige Kontakte anschreiben, wenn ich WhatsApp verknüpfe?**\
Nein. Die Standard-DM-Richtlinie ist **Kopplung**, daher erhalten unbekannte Absender nur einen Kopplungscode und ihre Nachricht wird **nicht verarbeitet**. OpenClaw antwortet nur auf Chats, die es empfängt, oder auf Send-Operationen, die du explizit auslöst (agent/CLI).

**Wie funktioniert Kopplung auf WhatsApp?**\
Kopplung ist ein DM-Gate für unbekannte Absender:

* Die erste DM von einem neuen Absender liefert einen kurzen Code zurück (die Nachricht wird nicht verarbeitet).
* Genehmige mit: `openclaw pairing approve whatsapp <code>` (auflisten mit `openclaw pairing list whatsapp`).
* Codes laufen nach 1 Stunde ab; ausstehende Anfragen sind auf 3 pro Kanal begrenzt.

**Können mehrere Personen verschiedene OpenClaw-Instanzen auf einer WhatsApp-Nummer verwenden?**\
Ja, indem du jeden Absender über `bindings` zu einem anderen agent routest (`peer kind: "dm"`, Absender-E.164 wie `+15551234567`). Antworten kommen trotzdem vom **gleichen WhatsApp-Konto**, und Direktchats werden in die Hauptsitzung jedes agents zusammengeführt, daher solltest du **einen agent pro Person** verwenden. Die DM-Zugriffskontrolle (`dmPolicy`/`allowFrom`) ist global pro WhatsApp-Konto. Siehe [Multi-Agent Routing](/de/concepts/multi-agent).

**Warum fragt der Wizard nach meiner Telefonnummer?**\
Der Wizard verwendet sie, um deine **Allowlist/Owner** zu setzen, damit deine eigenen DMs zugelassen werden. Sie wird nicht für automatisches Senden verwendet. Wenn du OpenClaw mit deiner persönlichen WhatsApp-Nummer betreibst, verwende dieselbe Nummer und aktiviere `channels.whatsapp.selfChatMode`.

<div id="message-normalization-what-the-model-sees">
  ## Nachrichten-Normalisierung (was das Modell sieht)
</div>

* `Body` ist der aktuelle Nachrichtentext inklusive Envelope.
* Zitierter Antwortkontext wird **immer angehängt**:
  ```
  [Replying to +1555 id:ABC123]
  <quoted text or <media:...>>
  [/Replying]
  ```
* Antwort-Metadaten werden ebenfalls gesetzt:
  * `ReplyToId` = stanzaId
  * `ReplyToBody` = zitierter Text oder Medienplatzhalter
  * `ReplyToSender` = E.164, wenn bekannt
* Eingehende reine Medien-Nachrichten verwenden Platzhalter:
  * `<media:image|video|audio|document|sticker>`

<div id="groups">
  ## Gruppen
</div>

* Gruppen werden `agent:<agentId>:whatsapp:group:<jid>`-Sitzungen zugeordnet.
* Gruppenrichtlinie: `channels.whatsapp.groupPolicy = open|disabled|allowlist` (Standardwert: `allowlist`).
* Aktivierungsmodi:
  * `mention` (Standard): erfordert eine @Erwähnung oder eine Regex-Übereinstimmung.
  * `always`: wird immer ausgelöst.
* `/activation mention|always` ist nur für den Owner verfügbar und muss als eigenständige Nachricht gesendet werden.
* Owner = `channels.whatsapp.allowFrom` (oder eigene E.164, falls nicht gesetzt).
* **Historieneinbettung** (nur für ausstehende Nachrichten):
  * Kürzliche *unverarbeitete* Nachrichten (Standard 50) werden eingefügt unter:
    `[Chat messages since your last reply - for context]` (Nachrichten, die bereits in der Sitzung sind, werden nicht erneut eingebettet)
  * Aktuelle Nachricht unter:
    `[Current message - respond to this]`
  * Absender-Suffix wird angehängt: `[from: Name (+E164)]`
* Gruppen-Metadaten werden 5 Min. im Cache gehalten (Betreff + Teilnehmende).

<div id="reply-delivery-threading">
  ## Antwortzustellung (Threading)
</div>

* WhatsApp Web sendet Standardnachrichten (keine zitierten Antwort-Threads im aktuellen Gateway).
* Antwort-Tags werden auf diesem Kanal ignoriert.

<div id="acknowledgment-reactions-auto-react-on-receipt">
  ## Bestätigungsreaktionen (automatische Reaktion beim Empfang)
</div>

WhatsApp kann automatisch Emoji-Reaktionen auf eingehende Nachrichten direkt beim Empfang senden, noch bevor der Bot eine Antwort generiert. So erhalten Benutzer eine sofortige Rückmeldung, dass ihre Nachricht eingegangen ist.

**Konfiguration:**

```json
{
  "whatsapp": {
    "ackReaction": {
      "emoji": "👀",
      "direct": true,
      "group": "mentions"
    }
  }
}
```

**Optionen:**

* `emoji` (string): Emoji für Bestätigungen (z. B. „👀“, „✅“, „📨“). Leer oder weggelassen = Funktion deaktiviert.
* `direct` (boolean, default: `true`): Reaktionen in Direkt-/DM-Chats senden.
* `group` (string, default: `"mentions"`): Verhalten in Gruppenchats:
  * `"always"`: Auf alle Gruppennachrichten reagieren (auch ohne @-Erwähnung)
  * `"mentions"`: Nur reagieren, wenn der Bot mit @ erwähnt wird
  * `"never"`: Nie in Gruppenchats reagieren

**Kontospezifische Überschreibung:**

```json
{
  "whatsapp": {
    "accounts": {
      "work": {
        "ackReaction": {
          "emoji": "✅",
          "direct": false,
          "group": "always"
        }
      }
    }
  }
}
```

**Hinweise zum Verhalten:**

* Reaktionen werden **sofort** beim Eingang der Nachricht gesendet, noch bevor Tippindikatoren oder Bot-Antworten erscheinen.
* In Gruppen mit `requireMention: false` (Aktivierung: immer) reagiert `group: "mentions"` auf alle Nachrichten (nicht nur auf @Mentions).
* Fire-and-forget: Fehler bei Reaktionen werden protokolliert, verhindern aber nicht, dass der Bot antwortet.
* Die JID des Teilnehmers wird für Gruppenreaktionen automatisch hinzugefügt.
* WhatsApp ignoriert `messages.ackReaction`; verwende stattdessen `channels.whatsapp.ackReaction`.

<div id="agent-tool-reactions">
  ## Agent-Tool (Reaktionen)
</div>

* Tool: `whatsapp` mit `react`-Aktion (`chatJid`, `messageId`, `emoji`, optional `remove`).
* Optional: `participant` (Absender in der Gruppe), `fromMe` (Reaktion auf die eigene Nachricht), `accountId` (Multi-Account).
* Semantik für das Entfernen von Reaktionen: siehe [/tools/reactions](/de/tools/reactions).
* Tool-Gating: `channels.whatsapp.actions.reactions` (Standard: aktiviert).

<div id="limits">
  ## Grenzwerte
</div>

* Ausgehender Text wird in Blöcke der Größe `channels.whatsapp.textChunkLimit` aufgeteilt (Standardwert 4000).
* Optionale Zeilenumbruch-Segmentierung: Setze `channels.whatsapp.chunkMode="newline"`, um vor der Längensegmentierung an Leerzeilen (Absatzgrenzen) zu trennen.
* Das Speichern eingehender Medien ist durch `channels.whatsapp.mediaMaxMb` begrenzt (Standardwert 50 MB).
* Ausgehende Mediendateien sind durch `agents.defaults.mediaMaxMb` begrenzt (Standardwert 5 MB).

<div id="outbound-send-text-media">
  ## Ausgehender Versand (Text + Medien)
</div>

* Verwendet einen aktiven Web-Listener; es tritt ein Fehler auf, wenn das Gateway nicht läuft.
* Text-Chunking: maximal 4k pro Nachricht (konfigurierbar über `channels.whatsapp.textChunkLimit`, optional `channels.whatsapp.chunkMode`).
* Medien:
  * Bild/Video/Audio/Dokument werden unterstützt.
  * Audio wird als PTT gesendet; `audio/ogg` =&gt; `audio/ogg; codecs=opus`.
  * Bildunterschrift nur beim ersten Medienelement.
  * Medienabruf unterstützt HTTP(S) und lokale Pfade.
  * Animierte GIFs: WhatsApp erwartet MP4 mit `gifPlayback: true` für Inline-Wiedergabe in Endlosschleife.
    * CLI: `openclaw message send --media <mp4> --gif-playback`
    * Gateway: `send`-Parameter umfassen `gifPlayback: true`

<div id="voice-notes-ptt-audio">
  ## Sprachnachrichten (PTT-Audio)
</div>

WhatsApp sendet Audio als **Sprachnachricht** (PTT-Bubble).

* Beste Ergebnisse mit: OGG/Opus. OpenClaw schreibt `audio/ogg` in `audio/ogg; codecs=opus` um.
* `[[audio_as_voice]]` wird für WhatsApp ignoriert (Audio wird bereits als Sprachnachricht übertragen).

<div id="media-limits-optimization">
  ## Medienlimits + Optimierung
</div>

* Standardobergrenze für ausgehende Medien: 5 MB (pro Medienelement).
* Override: `agents.defaults.mediaMaxMb`.
* Bilder werden automatisch als JPEG innerhalb der Obergrenze optimiert (Größenanpassung + Qualitätsanpassung).
* Zu große Medien =&gt; Fehler; die Antwort mit Medium fällt auf eine Textwarnung zurück.

<div id="heartbeats">
  ## Herzschläge
</div>

* **Gateway-Herzschlag** protokolliert den Zustand der Verbindung (`web.heartbeatSeconds`, Standard 60s).
* **Agent-Herzschlag** kann pro agent konfiguriert werden (`agents.list[].heartbeat`) oder global
  über `agents.defaults.heartbeat` (Fallback, wenn keine agent-spezifischen Einträge gesetzt sind).
  * Verwendet den konfigurierten Herzschlag-Prompt (Standard: `Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.`) plus `HEARTBEAT_OK`-Überspringverhalten.
  * Standardmäßig wird an den zuletzt verwendeten Kanal (oder das konfigurierte Ziel) zugestellt.

<div id="reconnect-behavior">
  ## Verhalten bei Wiederverbindung
</div>

* Backoff-Richtlinie: `web.reconnect`:
  * `initialMs`, `maxMs`, `factor`, `jitter`, `maxAttempts`.
* Wenn `maxAttempts` erreicht ist, wird das Web-Monitoring beendet (degradierter Zustand).
* Abgemeldet =&gt; beenden und erneute Verknüpfung erforderlich.

<div id="config-quick-map">
  ## Konfigurations-Schnellübersicht
</div>

* `channels.whatsapp.dmPolicy` (DM-Richtlinie: pairing/allowlist/open/disabled).
* `channels.whatsapp.selfChatMode` (Setup mit derselben Telefonnummer; Bot verwendet deine persönliche WhatsApp-Nummer).
* `channels.whatsapp.allowFrom` (DM-Allowlist). WhatsApp verwendet E.164-Telefonnummern (keine Benutzernamen).
* `channels.whatsapp.mediaMaxMb` (Grenzwert für eingehende Medien in MB).
* `channels.whatsapp.ackReaction` (automatische Reaktion beim Nachrichteneingang: `{emoji, direct, group}`).
* `channels.whatsapp.accounts.<accountId>.*` (kontospezifische Einstellungen + optionales `authDir`).
* `channels.whatsapp.accounts.<accountId>.mediaMaxMb` (kontospezifischer Grenzwert für eingehende Medien).
* `channels.whatsapp.accounts.<accountId>.ackReaction` (kontospezifischer Override der Empfangsreaktion).
* `channels.whatsapp.groupAllowFrom` (Allowlist für Gruppensender).
* `channels.whatsapp.groupPolicy` (Gruppenrichtlinie).
* `channels.whatsapp.historyLimit` / `channels.whatsapp.accounts.<accountId>.historyLimit` (Kontext der Gruppenhistorie; `0` deaktiviert).
* `channels.whatsapp.dmHistoryLimit` (DM-Historienlimit in Benutzer-Turns). Benutzerbezogene Overrides: `channels.whatsapp.dms["<phone>"].historyLimit`.
* `channels.whatsapp.groups` (Gruppen-Allowlist + Standardwerte für Mention-Gating; verwende `"*"`, um alle zuzulassen)
* `channels.whatsapp.actions.reactions` (Steuerung von WhatsApp-Tool-Reaktionen).
* `agents.list[].groupChat.mentionPatterns` (oder `messages.groupChat.mentionPatterns`)
* `messages.groupChat.historyLimit`
* `channels.whatsapp.messagePrefix` (eingehender Präfix; pro Konto: `channels.whatsapp.accounts.<accountId>.messagePrefix`; veraltet: `messages.messagePrefix`)
* `messages.responsePrefix` (ausgehender Präfix)
* `agents.defaults.mediaMaxMb`
* `agents.defaults.heartbeat.every`
* `agents.defaults.heartbeat.model` (optionalem Override)
* `agents.defaults.heartbeat.target`
* `agents.defaults.heartbeat.to`
* `agents.defaults.heartbeat.session`
* `agents.list[].heartbeat.*` (agentspezifische Overrides)
* `session.*` (scope, idle, store, mainKey)
* `web.enabled` (deaktiviert Channel-Start, wenn false)
* `web.heartbeatSeconds`
* `web.reconnect.*`

<div id="logs-troubleshooting">
  ## Logs + Fehlerbehebung
</div>

* Subsysteme: `whatsapp/inbound`, `whatsapp/outbound`, `web-heartbeat`, `web-reconnect`.
* Logdatei: `/tmp/openclaw/openclaw-YYYY-MM-DD.log` (konfigurierbar).
* Leitfaden zur Fehlerbehebung: [Gateway-Fehlerbehebung](/de/gateway/troubleshooting).

<div id="troubleshooting-quick">
  ## Fehlerbehebung (kurz)
</div>

**Nicht verknüpft / QR-Login erforderlich**

* Symptom: `channels status` zeigt `linked: false` oder warnt „Not linked“.
* Lösung: Führe `openclaw channels login` auf dem Gateway-Host aus und scanne den QR-Code (WhatsApp → Einstellungen → Verknüpfte Geräte).

**Verknüpft, aber getrennt / Reconnect-Schleife**

* Symptom: `channels status` zeigt `running, disconnected` oder warnt „Linked but disconnected“.
* Lösung: `openclaw doctor` ausführen (oder das Gateway neu starten). Wenn das Problem bestehen bleibt, über `channels login` neu verknüpfen und `openclaw logs --follow` überprüfen.

**Bun-Runtime**

* Bun wird **nicht empfohlen**. WhatsApp (Baileys) und Telegram sind unter Bun unzuverlässig.
  Führe das Gateway mit **Node** aus. (Siehe Hinweis zur Runtime im Abschnitt Getting Started.)