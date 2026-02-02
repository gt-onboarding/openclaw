---
title: Nachrichten
summary: "Nachrichtenfluss, Sitzungen, Queueing und Sichtbarkeit des Reasonings"
read_when:
  - Zum Erklären, wie eingehende Nachrichten zu Antworten werden
  - Zur Klärung von Sitzungen, Queueing-Modi oder des Streaming-Verhaltens
  - Zum Dokumentieren der Sichtbarkeit des Reasonings und ihrer Auswirkungen auf die Nutzung
---

<div id="messages">
  # Nachrichten
</div>

Diese Seite erläutert, wie OpenClaw den Umgang mit eingehenden Nachrichten, Sitzungen, Queueing (Warteschlangen),
Streaming und der Sichtbarkeit des Reasonings zusammenführt.

<div id="message-flow-high-level">
  ## Nachrichtenfluss (grober Überblick)
</div>

```
Inbound message
  -> routing/bindings -> session key
  -> queue (if a run is active)
  -> agent run (streaming + tools)
  -> outbound replies (channel limits + chunking)
```

Zentrale Stellschrauben befinden sich in der Konfiguration:

* `messages.*` für Präfixe, Warteschlangenverhalten und Gruppenlogik.
* `agents.defaults.*` für Standardwerte zu Block-Streaming und Chunking.
* Kanalbezogene Overrides (`channels.whatsapp.*`, `channels.telegram.*`, etc.) für Limits und Streaming-Umschalter.

Siehe [Konfiguration](/de/gateway/configuration) für das vollständige Schema.

<div id="inbound-dedupe">
  ## Eingehende Deduplizierung
</div>

Kanäle können nach einem erneuten Verbindungsaufbau dieselbe Nachricht noch einmal zustellen. OpenClaw
hält dafür einen kurzlebigen Cache vor, der anhand von Kanal/Konto/Peer/Sitzung/Nachrichten-ID
indiziert wird, sodass doppelte Zustellungen keine weitere agent-Ausführung auslösen.

<div id="inbound-debouncing">
  ## Eingehendes Debouncing
</div>

Schnell aufeinanderfolgende Nachrichten vom **gleichen Absender** können über
`messages.inbound` zu einem einzigen Agent-Turn gebündelt werden. Debouncing gilt
pro Kanal und Konversation und verwendet die neueste Nachricht für
Reply-Threading/IDs.

Konfiguration (globale Standardeinstellung + kanalspezifische Overrides):

```json5
{
  messages: {
    inbound: {
      debounceMs: 2000,
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

* Debounce gilt nur für **rein textbasierte** Nachrichten; Medien und Anhänge werden sofort durchgestellt.
* Steuerbefehle werden nicht gedebounced, damit sie eigenständig bleiben.

<div id="sessions-and-devices">
  ## Sitzungen und Geräte
</div>

Sitzungen gehören dem Gateway, nicht den Clients.

* Direkte Chats werden in den Hauptsitzungsschlüssel des Agents zusammengefasst.
* Gruppen/Kanäle erhalten eigene Sitzungsschlüssel.
* Der Sitzungsspeicher und die Transkripte liegen auf dem Gateway-Host.

Mehrere Geräte/Kanäle können derselben Sitzung zugeordnet sein, aber der Verlauf
wird nicht vollständig zu jedem Client zurücksynchronisiert. Empfehlung: Verwenden
Sie ein primäres Gerät für lange Unterhaltungen, um divergierenden Kontext zu vermeiden.
Die Control UI und die TUI zeigen immer das vom Gateway hinterlegte
Sitzungstranskript, sie sind also die maßgebliche Quelle.

Details: [Sitzungsverwaltung](/de/concepts/session).

<div id="inbound-bodies-and-history-context">
  ## Eingehende Bodies und Verlaufskontext
</div>

OpenClaw trennt den **Prompt-Body** vom **Command-Body**:

* `Body`: Prompt-Text, der an den agent gesendet wird. Dies kann Channel-Wrapper und
  optionale Verlaufs-Wrapper enthalten.
* `CommandBody`: roher Benutzertext für Direktiven-/Befehlsparsing.
* `RawBody`: veraltetes Alias für `CommandBody` (zur Kompatibilität beibehalten).

Wenn ein Channel Verlauf bereitstellt, verwendet er einen gemeinsamen Wrapper:

* `[Chat messages since your last reply - for context]`
* `[Current message - respond to this]`

Bei **nicht-direkten Chats** (Gruppen/Channels/Räumen) wird der **aktuelle Nachrichten-Body** mit dem
Absenderlabel vorangestellt (im selben Stil wie bei Verlaufs­einträgen). So bleiben Echtzeit- und
Warteschlangen-/Verlaufsnachrichten im agent-Prompt konsistent.

Verlaufspuffer enthalten **nur ausstehende Nachrichten**: Sie enthalten Gruppennachrichten, die *keinen*
Run ausgelöst haben (z. B. mention-gated Nachrichten), und **schließen** Nachrichten aus,
die bereits im Sitzungsprotokoll enthalten sind.

Das Entfernen von Direktiven wird nur auf den Abschnitt der **aktuellen Nachricht** angewendet, damit der Verlauf
unverändert bleibt. Channels, die Verlauf wrappen, sollten `CommandBody` (oder
`RawBody`) auf den ursprünglichen Nachrichtentext setzen und `Body` als kombinierten Prompt beibehalten.
Verlaufspuffer sind konfigurierbar über `messages.groupChat.historyLimit` (globaler
Standard) und kanal­spezifische Overrides wie `channels.slack.historyLimit` oder
`channels.telegram.accounts.<id>.historyLimit` (auf `0` setzen, um zu deaktivieren).

<div id="queueing-and-followups">
  ## Warteschlangen und Follow-ups
</div>

Wenn ein Run bereits aktiv ist, können eingehende Nachrichten in die Warteschlange aufgenommen, in den aktuellen Run eingesteuert oder für einen späteren Turn gesammelt werden.

* Konfigurieren über `messages.queue` (und `messages.queue.byChannel`).
* Modi: `interrupt`, `steer`, `followup`, `collect` sowie Backlog-Varianten.

Details: [Warteschlangen](/de/concepts/queue).

<div id="streaming-chunking-and-batching">
  ## Streaming, Chunking und Batching
</div>

Block-Streaming sendet Teilantworten, während das Modell Textblöcke erzeugt.
Chunking respektiert die Textlimits der Kanäle und verhindert das Auftrennen von Fenced-Codeblöcken.

Wichtige Einstellungen:

* `agents.defaults.blockStreamingDefault` (`on|off`, Standard: off)
* `agents.defaults.blockStreamingBreak` (`text_end|message_end`)
* `agents.defaults.blockStreamingChunk` (`minChars|maxChars|breakPreference`)
* `agents.defaults.blockStreamingCoalesce` (leerlaufbasiertes Batching)
* `agents.defaults.humanDelay` (menschlich wirkende Pause zwischen Blockantworten)
* Kanal-Overrides: `*.blockStreaming` und `*.blockStreamingCoalesce` (Nicht-Telegram-Kanäle erfordern explizit `*.blockStreaming: true`)

Details: [Streaming + Chunking](/de/concepts/streaming).

<div id="reasoning-visibility-and-tokens">
  ## Sichtbarkeit von Reasoning und Tokens
</div>

OpenClaw kann das Reasoning des Modells ein- oder ausblenden:

* `/reasoning on|off|stream` steuert die Sichtbarkeit.
* Reasoning-Inhalte zählen weiterhin zum Tokenverbrauch, wenn sie vom Modell erzeugt werden.
* Telegram unterstützt Reasoning-Streams im Entwurfsfeld.

Details: [Thinking- und Reasoning-Direktiven](/de/tools/thinking) und [Tokenverbrauch](/de/token-use).

<div id="prefixes-threading-and-replies">
  ## Präfixe, Threading und Antworten
</div>

Die Formatierung ausgehender Nachrichten wird in `messages` zentral gesteuert:

* `messages.responsePrefix` (ausgehendes Präfix) und `channels.whatsapp.messagePrefix` (eingehendes WhatsApp-Präfix)
* Antwort-Threading über `replyToMode` und kanalbezogene Standardwerte

Details: [Konfiguration](/de/gateway/configuration#messages) und Kanal-Dokumentation.