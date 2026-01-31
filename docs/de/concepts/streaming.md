---
title: Streaming
summary: "Streaming- und Chunking-Verhalten (Block-Antworten, Entwurfs-Streaming, Limits)"
read_when:
  - Erklären, wie Streaming oder Chunking in Kanälen funktioniert
  - Block-Streaming oder Chunking-Verhalten von Kanälen ändern
  - Doppelte/zu frühe Block-Antworten oder Entwurfs-Streaming debuggen
---

<div id="streaming-chunking">
  # Streaming + Chunking
</div>

OpenClaw hat zwei getrennte „Streaming“-Ebenen:

- **Block-Streaming (Channels):** Gibt abgeschlossene **Blöcke** aus, während der Assistant schreibt. Das sind normale Channel-Nachrichten (keine Token-Deltas).
- **Token-artiges Streaming (nur Telegram):** Aktualisiert eine **Entwurfs-Sprechblase** mit Teiltext während der Generierung; die endgültige Nachricht wird am Ende gesendet.

Es gibt derzeit **kein echtes Token-Streaming** zu externen Channel-Nachrichten. Telegram-Entwurfs-Streaming ist die einzige Schnittstelle mit Teil-Streaming.

<div id="block-streaming-channel-messages">
  ## Block-Streaming (Kanalnachrichten)
</div>

Block-Streaming überträgt die Ausgaben des Assistenten in größeren Blöcken, sobald sie vorliegen.

```
Model output
  └─ text_delta/events
       ├─ (blockStreamingBreak=text_end)
       │    └─ chunker emits blocks as buffer grows
       └─ (blockStreamingBreak=message_end)
            └─ chunker flushes at message_end
                   └─ channel send (block replies)
```

Legende:

* `text_delta/events`: Modell-Stream-Events (können bei nicht-streamenden Modellen spärlich sein).
* `chunker`: `EmbeddedBlockChunker`, der Min-/Max-Grenzen plus bevorzugte Trennpunkte anwendet.
* `channel send`: tatsächliche ausgehende Nachrichten (Block-Antworten).

**Konfiguration:**

* `agents.defaults.blockStreamingDefault`: `"on"`/`"off"` (Standard: `"off"`).
* Channel-Overrides: `*.blockStreaming` (und Varianten pro Account), um `"on"`/`"off"` pro Channel zu erzwingen.
* `agents.defaults.blockStreamingBreak`: `"text_end"` oder `"message_end"`.
* `agents.defaults.blockStreamingChunk`: `{ minChars, maxChars, breakPreference? }`.
* `agents.defaults.blockStreamingCoalesce`: `{ minChars?, maxChars?, idleMs? }` (gestreamte Blöcke vor dem Senden zusammenführen).
* Harte Channel-Obergrenze: `*.textChunkLimit` (z. B. `channels.whatsapp.textChunkLimit`).
* Channel-Chunk-Modus: `*.chunkMode` (`length` als Standard, `newline` trennt an Leerzeilen (Absatzgrenzen), bevor nach Länge gechunkt wird).
* Discord-Soft-Limit: `channels.discord.maxLinesPerMessage` (Standard 17) teilt sehr lange Antworten auf, um UI-Clipping zu vermeiden.

**Grenzsemantik:**

* `text_end`: Blöcke werden gestreamt, sobald der Chunker ausgibt; bei jedem `text_end` wird geflusht.
* `message_end`: warten, bis die Assistant-Nachricht fertig ist, dann gepufferte Ausgabe flushen.

`message_end` verwendet den Chunker weiterhin, wenn der gepufferte Text `maxChars` überschreitet, sodass am Ende mehrere Chunks ausgegeben werden können.


<div id="chunking-algorithm-lowhigh-bounds">
  ## Chunking-Algorithmus (untere/obere Grenzen)
</div>

Block-Chunking wird vom `EmbeddedBlockChunker` implementiert:

- **Untere Grenze:** nichts ausgeben, bis der Puffer mindestens `minChars` Zeichen enthält (außer bei erzwungener Ausgabe).
- **Obere Grenze:** Trennungen möglichst vor `maxChars`; wenn erzwungen, genau bei `maxChars` trennen.
- **Bevorzugte Trennstellen:** `paragraph` → `newline` → `sentence` → `whitespace` → harter Umbruch.
- **Code-Fences:** niemals innerhalb von Fences trennen; wenn eine Trennung bei `maxChars` erzwungen wird, das Fence schließen und neu öffnen, um gültiges Markdown beizubehalten.

`maxChars` wird auf das kanalspezifische `textChunkLimit` begrenzt, sodass du die Kanalobergrenzen nicht überschreiten kannst.

<div id="coalescing-merge-streamed-blocks">
  ## Koaleszieren (gestreamte Blöcke zusammenführen)
</div>

Wenn Block-Streaming aktiviert ist, kann OpenClaw **aufeinanderfolgende Block-Chunks zusammenführen**, bevor sie gesendet werden. Dadurch wird „Einzeilen-Spam“ reduziert, während weiterhin progressive Ausgaben bereitgestellt werden.

- Koaleszieren wartet auf **Leerlaufpausen** (`idleMs`), bevor gepuffertes Material ausgegeben (geflusht) wird.
- Puffer sind durch `maxChars` begrenzt und werden geleert, wenn sie diesen Wert überschreiten.
- `minChars` verhindert, dass winzige Fragmente gesendet werden, bis sich genug Text angesammelt hat
  (der abschließende Flush sendet immer den verbleibenden Text).
- Der Joiner wird aus `blockStreamingChunk.breakPreference` abgeleitet
  (`paragraph` → `\n\n`, `newline` → `\n`, `sentence` → Leerzeichen).
- Kanal-Overrides sind über `*.blockStreamingCoalesce` verfügbar (einschließlich kontospezifischer Konfigurationen).
- Der Standardwert für `minChars` beim Koaleszieren wird für Signal/Slack/Discord auf 1500 angehoben, sofern er nicht überschrieben wird.

<div id="human-like-pacing-between-blocks">
  ## Menschliches Tempo zwischen Blöcken
</div>

Wenn Block-Streaming aktiviert ist, kannst du eine **zufällige Pause** zwischen
Block-Antworten (nach dem ersten Block) hinzufügen. Das lässt Antworten mit mehreren Antwortblasen natürlicher wirken.

- Konfiguration: `agents.defaults.humanDelay` (pro Agent überschreibbar über `agents.list[].humanDelay`).
- Modi: `off` (Standard), `natural` (800–2500ms), `custom` (`minMs`/`maxMs`).
- Gilt nur für **Block-Antworten**, nicht für finale Antworten oder Tool-Zusammenfassungen.

<div id="stream-chunks-or-everything">
  ## „Chunks streamen oder alles“
</div>

Dies entspricht Folgendem:

- **Chunks streamen:** `blockStreamingDefault: "on"` + `blockStreamingBreak: "text_end"` (fortlaufend ausgeben). Nicht-Telegram-Kanäle benötigen zusätzlich `*.blockStreaming: true`.
- **Alles am Ende streamen:** `blockStreamingBreak: "message_end"` (einmaliges Flush, ggf. mehrere Chunks bei sehr langen Nachrichten).
- **Kein Block-Streaming:** `blockStreamingDefault: "off"` (nur finale Antwort).

**Hinweis zu Kanälen:** Für Nicht-Telegram-Kanäle ist Block-Streaming **aus, außer**
`*.blockStreaming` ist explizit auf `true` gesetzt. Telegram kann Entwürfe
(`channels.telegram.streamMode`) ohne Block-Antworten streamen.

Erinnerung zum Speicherort in der Config: Die `blockStreaming*`-Defaults liegen unter
`agents.defaults`, nicht in der Root-Konfiguration.

<div id="telegram-draft-streaming-token-ish">
  ## Telegram-Entwurfsstreaming (token-ähnlich)
</div>

Telegram ist der einzige Kanal mit Entwurfsstreaming:

* Verwendet die Bot-API `sendMessageDraft` in **privaten Chats mit Themen**.
* `channels.telegram.streamMode: "partial" | "block" | "off"`.
  * `partial`: Entwurfsaktualisierungen mit dem jeweils neuesten Stream-Text.
  * `block`: Entwurfsaktualisierungen in gechunkten Blöcken (gleiche Chunker-Regeln).
  * `off`: kein Entwurfsstreaming.
* Entwurfs-Chunk-Konfiguration (nur für `streamMode: "block"`): `channels.telegram.draftChunk` (Standardwerte: `minChars: 200`, `maxChars: 800`).
* Entwurfsstreaming ist getrennt von Block-Streaming; Block-Antworten sind standardmäßig deaktiviert und werden nur durch `*.blockStreaming: true` auf Nicht-Telegram-Kanälen aktiviert.
* Die endgültige Antwort ist weiterhin eine normale Nachricht.
* `/reasoning stream` schreibt das Reasoning in die Entwurfsblase (nur Telegram).

Wenn Entwurfsstreaming aktiv ist, deaktiviert OpenClaw für diese Antwort das Block-Streaming, um doppeltes Streaming zu vermeiden.

```
Telegram (privat + Topics)
  └─ sendMessageDraft (Entwurfs-Bubble)
       ├─ streamMode=partial → aktualisiert neuesten Text
       └─ streamMode=block   → Chunker aktualisiert Entwurf
  └─ finale Antwort → normale Nachricht
```

Legende:

* `sendMessageDraft`: Telegram-Entwurfsblase (keine echte Nachricht).
* `final reply`: normal gesendete Telegram-Nachricht.
