---
title: Sitzungsbereinigung
summary: "Sitzungsbereinigung: Kürzen von Tool-Ergebnissen, um Kontextaufblähung zu reduzieren"
read_when:
  - Du möchtest das Anwachsen des LLM-Kontexts durch Tool-Ausgaben begrenzen
  - Du passt agents.defaults.contextPruning an
---

<div id="session-pruning">
  # Sitzungsbereinigung
</div>

Die Sitzungsbereinigung entfernt **alte Tool-Ergebnisse** aus dem In-Memory-Kontext direkt vor jedem LLM-Aufruf. Dabei wird der Sitzungsverlauf auf dem Datenträger (`*.jsonl`) **nicht** umgeschrieben.

<div id="when-it-runs">
  ## Wann es ausgeführt wird
</div>

* Wenn `mode: "cache-ttl"` aktiviert ist und der letzte Anthropic-Aufruf in der Sitzung älter als `ttl` ist.
* Betrifft nur die Nachrichten, die für diese Anfrage an das Modell gesendet werden.
* Nur aktiv für Anthropic-API-Aufrufe (und OpenRouter-Anthropic-Modelle).
* Für beste Ergebnisse `ttl` an das `cacheControlTtl` deines Modells anpassen.
* Nach einem Prune-Vorgang wird das TTL-Fenster zurückgesetzt, sodass nachfolgende Anfragen den Cache weiterverwenden, bis `ttl` erneut abläuft.

<div id="smart-defaults-anthropic">
  ## Intelligente Standardwerte (Anthropic)
</div>

* **OAuth- oder Setup-Token**-Profile: aktiviere `cache-ttl`-Pruning und setze den Herzschlag auf `1h`.
* **API-Key**-Profile: aktiviere `cache-ttl`-Pruning, setze den Herzschlag auf `30m` und setze standardmäßig `cacheControlTtl` auf `1h` für Anthropic-Modelle.
* Wenn du einen dieser Werte explizit festlegst, überschreibt OpenClaw ihn **nicht**.

<div id="what-this-improves-cost-cache-behavior">
  ## Was sich dadurch verbessert (Kosten + Cache-Verhalten)
</div>

* **Warum kürzen:** Anthropic Prompt-Caching greift nur innerhalb der TTL. Wenn eine Sitzung nach Ablauf der TTL inaktiv wird, speichert die nächste Anfrage den vollständigen Prompt erneut im Cache, sofern du ihn nicht vorher kürzt.
* **Was günstiger wird:** Kürzen reduziert die Größe von **cacheWrite** für die erste Anfrage, nachdem die TTL abgelaufen ist.
* **Warum das Zurücksetzen der TTL wichtig ist:** Sobald das Kürzen ausgeführt wurde, wird das Cache-Fenster zurückgesetzt, sodass nachfolgende Anfragen den frisch im Cache gespeicherten Prompt wiederverwenden können, statt den gesamten Verlauf erneut zu cachen.
* **Was es nicht tut:** Kürzen fügt keine Token hinzu und „verdoppelt“ auch nicht die Kosten, sondern ändert nur, was bei dieser ersten Anfrage nach Ablauf der TTL im Cache landet.

<div id="what-can-be-pruned">
  ## Was gekürzt werden kann
</div>

* Nur `toolResult`-Nachrichten.
* Benutzer- und Assistenten-Nachrichten werden **nie** verändert.
* Die letzten `keepLastAssistants`-Assistenten-Nachrichten sind geschützt; Tool-Ergebnisse nach dieser Schwelle werden nicht gekürzt.
* Wenn es nicht genug Assistenten-Nachrichten gibt, um diese Schwelle festzulegen, entfällt die Kürzung.
* Tool-Ergebnisse, die **Bildblöcke** enthalten, werden übersprungen (niemals gekürzt/gelöscht).

<div id="context-window-estimation">
  ## Schätzung des Kontextfensters
</div>

Pruning verwendet ein geschätztes Kontextfenster (Zeichen ≈ Token × 4). Die Fenstergröße wird in folgender Reihenfolge bestimmt:

1. Modelldefinition `contextWindow` (aus der Modell-Registry).
2. Überschreibung über `models.providers.*.models[].contextWindow`.
3. `agents.defaults.contextTokens`.
4. Standardwert von `200000` Token.

<div id="mode">
  ## Modus
</div>

<div id="cache-ttl">
  ### cache-ttl
</div>

* Pruning wird nur ausgeführt, wenn der letzte Anthropic-Aufruf älter als `ttl` ist (Standardwert `5m`).
* Wenn es ausgeführt wird: dasselbe Soft-Trim- plus Hard-Clear-Verhalten wie zuvor.

<div id="soft-vs-hard-pruning">
  ## Soft- vs. Hard-Pruning
</div>

* **Soft-Trim**: nur für zu große Tool-Ergebnisse.
  * Behält Anfang und Ende, fügt `...` ein und hängt eine Notiz mit der ursprünglichen Größe an.
  * Überspringt Ergebnisse mit Bildblöcken.
* **Hard-Clear**: ersetzt das gesamte Tool-Ergebnis durch `hardClear.placeholder`.

<div id="tool-selection">
  ## Werkzeugauswahl
</div>

* `tools.allow` / `tools.deny` unterstützen `*`-Platzhalter (Wildcards).
* `deny` hat Vorrang.
* Groß-/Kleinschreibung wird bei der Übereinstimmung ignoriert.
* Leere Allow-Liste =&gt; alle Werkzeuge sind erlaubt.

<div id="interaction-with-other-limits">
  ## Interaktion mit anderen Limits
</div>

* Eingebaute Tools kürzen ihre eigene Ausgabe bereits; Sitzungsbereinigung ist eine zusätzliche Ebene, die verhindert, dass sich in lang andauernden Chats zu viel Tool-Ausgabe im Modellkontext ansammelt.
* Kompaktierung ist davon getrennt: Kompaktierung erstellt Zusammenfassungen und speichert sie dauerhaft, Bereinigung ist flüchtig und wirkt pro Anfrage. Siehe [/concepts/compaction](/de/concepts/compaction).

<div id="defaults-when-enabled">
  ## Standardwerte (falls aktiviert)
</div>

* `ttl`: `"5m"`
* `keepLastAssistants`: `3`
* `softTrimRatio`: `0.3`
* `hardClearRatio`: `0.5`
* `minPrunableToolChars`: `50000`
* `softTrim`: `{ maxChars: 4000, headChars: 1500, tailChars: 1500 }`
* `hardClear`: `{ enabled: true, placeholder: "[Alter Tool-Ergebnisinhalt entfernt]" }`

<div id="examples">
  ## Beispiele
</div>

Standard (deaktiviert):

```json5
{
  agent: {
    contextPruning: { mode: "off" }
  }
}
```

TTL-basierte Bereinigung aktivieren:

```json5
{
  agent: {
    contextPruning: { mode: "cache-ttl", ttl: "5m" }
  }
}
```

Beschränken Sie das Pruning auf bestimmte Tools:

```json5
{
  agent: {
    contextPruning: {
      mode: "cache-ttl",
      tools: { allow: ["exec", "read"], deny: ["*image*"] }
    }
  }
}
```

Siehe die Konfigurationsreferenz: [Gateway-Konfiguration](/de/gateway/configuration)
