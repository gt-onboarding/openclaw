---
title: Markdown-Formatierung
summary: "Pipeline für die Markdown-Formatierung für ausgehende Kanäle"
read_when:
  - Du änderst die Markdown-Formatierung oder das Chunking für ausgehende Kanäle
  - Du fügst einen neuen Kanal-Formatter oder eine neue Stilzuordnung hinzu
  - Du debuggst kanalübergreifende Formatierungsregressionen
---

<div id="markdown-formatting">
  # Markdown-Formatierung
</div>

OpenClaw formatiert ausgehendes Markdown, indem es dieses zunächst in eine gemeinsame Intermediate Representation (IR) überführt, bevor kanalspezifische Ausgaben gerendert werden. Die IR lässt den Quelltext unverändert und enthält Stil-/Link-Spans, sodass Chunking und Rendering kanalübergreifend konsistent bleiben können.

<div id="goals">
  ## Ziele
</div>

* **Konsistenz:** ein Parsing-Schritt, mehrere Renderer.
* **Sicheres Chunking:** Text vor dem Rendern aufteilen, damit Inline-Formatierung
  niemals über Chunk-Grenzen hinweg kaputtgeht.
* **Kanal-Fit:** dieselbe IR auf Slack mrkdwn, Telegram HTML und Signal-Style-Ranges
  abbilden, ohne Markdown erneut zu parsen.

<div id="pipeline">
  ## Pipeline
</div>

1. **Parse Markdown -&gt; IR**
   * IR ist Klartext plus Stilbereiche (fett/kursiv/durchgestrichen/code/spoiler) und Link-Bereiche.
   * Offsets sind UTF-16-Codeeinheiten, sodass Signal-Stilbereiche mit seiner api ausgerichtet sind.
   * Tabellen werden nur geparst, wenn ein Kanal die Tabellenkonvertierung explizit aktiviert.
2. **Chunk IR (format-first)**
   * Chunking erfolgt auf dem IR-Text vor dem Rendering.
   * Inline-Formatierung wird nicht über Chunk-Grenzen hinweg aufgetrennt; Spans werden pro Chunk zugeschnitten.
3. **Render per channel**
   * **Slack:** mrkdwn-Tokens (fett/kursiv/durchgestrichen/code), Links als `<url|label>`.
   * **Telegram:** HTML-Tags (`<b>`, `<i>`, `<s>`, `<code>`, `<pre><code>`, `<a href>`).
   * **Signal:** Klartext + `text-style`-Bereiche; Links werden zu `label (url)`, wenn sich das Label unterscheidet.

<div id="ir-example">
  ## IR-Beispiel
</div>

Markdown-Eingabe:

```markdown
Hello **world** — see [docs](https://docs.openclaw.ai).
```

IR (Schema):

```json
{
  "text": "Hello world — see docs.",
  "styles": [
    { "start": 6, "end": 11, "style": "bold" }
  ],
  "links": [
    { "start": 19, "end": 23, "href": "https://docs.openclaw.ai" }
  ]
}
```

<div id="where-it-is-used">
  ## Wo es verwendet wird
</div>

* Slack-, Telegram- und Signal-Outbound-Adapter rendern aus der IR.
* Andere Kanäle (WhatsApp, iMessage, MS Teams, Discord) verwenden weiterhin
  Klartext oder ihre eigenen Formatierungsregeln, wobei die Markdown-Tabellenkonvertierung vor
  dem Chunking angewendet wird, wenn diese aktiviert ist.

<div id="table-handling">
  ## Tabellenverarbeitung
</div>

Markdown-Tabellen werden von Chat-Clients nicht einheitlich unterstützt. Verwende
`markdown.tables`, um die Konvertierung pro Kanal (und pro Account) zu steuern.

* `code`: Tabellen als Codeblöcke rendern (Standard für die meisten Kanäle).
* `bullets`: jede Zeile in Aufzählungspunkte umwandeln (Standard für Signal + WhatsApp).
* `off`: Tabellen-Parsing und -Konvertierung deaktivieren; der rohe Tabellentext wird unverändert durchgereicht.

Konfigurationsschlüssel:

```yaml
channels:
  discord:
    markdown:
      tables: code
    accounts:
      work:
        markdown:
          tables: off
```

<div id="chunking-rules">
  ## Chunking-Regeln
</div>

* Chunk-Grenzen stammen aus Channel-Adaptern/-Konfigurationen und werden auf den IR-Text angewendet.
* Code-Fences werden als einzelner Block mit einem abschließenden Zeilenumbruch beibehalten, damit Kanäle
  sie korrekt rendern.
* Listenpräfixe und Blockzitat-Präfixe sind Teil des IR-Texts, daher werden Chunks
  nicht mitten in einem Präfix geteilt.
* Inline-Formatierungen (fett/kursiv/durchgestrichen/Inline-Code/Spoiler) werden nie über
  Chunks hinweg aufgeteilt; der Renderer setzt die Formatierungen in jedem Chunk erneut.

Wenn du mehr zum Chunking-Verhalten über verschiedene Kanäle hinweg brauchst, siehe
[Streaming + chunking](/de/concepts/streaming).

<div id="link-policy">
  ## Link-Richtlinie
</div>

* **Slack:** `[label](url)` -&gt; `<url|label>`; nackte URLs bleiben unverändert. Autolink
  ist beim Parsen deaktiviert, um doppeltes Verlinken zu vermeiden.
* **Telegram:** `[label](url)` -&gt; `<a href="url">label</a>` (HTML-Parse-Modus).
* **Signal:** `[label](url)` -&gt; `label (url)`; es sei denn, das Label entspricht der URL.

<div id="spoilers">
  ## Spoiler
</div>

Spoiler-Markierungen (`||spoiler||`) werden nur in Signal interpretiert; dort werden sie in SPOILER-Stilbereiche umgesetzt. Andere Kanäle behandeln sie als einfachen Text.

<div id="how-to-add-or-update-a-channel-formatter">
  ## So fügst du einen Channel-Formatter hinzu oder aktualisierst ihn
</div>

1. **Einmal parsen:** Verwende die gemeinsame Hilfsfunktion `markdownToIR(...)` mit zum Channel passenden
   Optionen (Autolink, Überschriftenstil, Blockquote-Präfix).
2. **Rendern:** Implementiere einen Renderer mit `renderMarkdownWithMarkers(...)` und einer
   Style-Marker-Map (oder Signal-Style-Ranges).
3. **Chunking:** Rufe `chunkMarkdownIR(...)` vor dem Rendern auf und rendere jeden Chunk.
4. **Adapter anbinden:** Aktualisiere den Outbound-Adapter des Channels, damit er den neuen Chunker
   und Renderer verwendet.
5. **Testen:** Füge Format-Tests hinzu oder aktualisiere sie und füge einen Outbound-Versandtest hinzu, wenn der
   Channel Chunking verwendet.

<div id="common-gotchas">
  ## Häufige Stolperfallen
</div>

* Slack-Token mit spitzen Klammern (`<@U123>`, `<#C123>`, `<https://...>`) müssen
  beibehalten werden; rohes HTML sicher escapen/maskieren.
* Telegram-HTML erfordert das Escapen von Text außerhalb von Tags, um fehlerhaftes Markup zu vermeiden.
* Ranges im Signal-Stil hängen von UTF-16-Offsets ab; verwende keine Codepoint-Offsets.
* Bewahre nachgestellte Zeilenumbrüche für fenced Codeblöcke, damit schließende Markierungen in
  einer eigenen Zeile stehen.