---
title: Transkript-Hygiene
summary: "Referenz: anbieterspezifische Regeln zur Bereinigung und Korrektur von Transkripten"
read_when:
  - Du untersuchst abgelehnte Anfragen von Anbietern, die auf die Struktur des Transkripts zurückzuführen sind
  - Du änderst die Logik zur Transkript-Bereinigung oder zur Reparatur von Tool-Calls
  - Du untersuchst Inkonsistenzen bei Tool-Call-IDs zwischen verschiedenen Anbietern
---

<div id="transcript-hygiene-provider-fixups">
  # Transkripthygiene (Anbieter-Korrekturen)
</div>

Dieses Dokument beschreibt **anbieterspezifische Korrekturen**, die auf Transkripte angewendet werden, bevor ein Lauf startet
(Erstellung des Modellkontexts). Dies sind **In-Memory**-Anpassungen, die dazu dienen, strikte
Anbieteranforderungen zu erfüllen. Sie **schreiben nicht** das gespeicherte JSONL-Transkript auf dem Datenträger um.

Der Umfang umfasst:

* Bereinigung von Tool-Call-IDs
* Reparatur der Tool-Resultat-Kopplung
* Validierung/Reihenfolge der Dialogrunden
* Bereinigung von Thought-Signaturen
* Bereinigung von Bild-Payloads

Wenn Sie Details zur Transkript-Speicherung benötigen, siehe:

* [/reference/session-management-compaction](/de/reference/session-management-compaction)

***

<div id="where-this-runs">
  ## Wo das läuft
</div>

Die gesamte Transkripthygiene ist im Embedded Runner zentralisiert:

* Richtlinienauswahl: `src/agents/transcript-policy.ts`
* Anwendung von Bereinigung/Reparatur: `sanitizeSessionHistory` in `src/agents/pi-embedded-runner/google.ts`

Die Richtlinie verwendet `provider`, `modelApi` und `modelId`, um zu entscheiden, welche Maßnahmen angewendet werden.

***

<div id="global-rule-image-sanitization">
  ## Globale Regel: Bildbereinigung
</div>

Bild-Payloads werden immer bereinigt, um eine Ablehnung durch den Anbieter aufgrund von Größenbeschränkungen zu verhindern (Verkleinern/Neukomprimieren übergroßer Base64-Bilder).

Implementierung:

* `sanitizeSessionMessagesImages` in `src/agents/pi-embedded-helpers/images.ts`
* `sanitizeContentBlocksImages` in `src/agents/tool-images.ts`

***

<div id="provider-matrix-current-behavior">
  ## Anbietermatrix (aktuelles Verhalten)
</div>

**OpenAI / OpenAI Codex**

* Nur Bildbereinigung.
* Beim Modellwechsel zu OpenAI Responses/Codex verwaiste Reasoning-Signaturen verwerfen (eigenständige Reasoning-Elemente ohne nachfolgenden Inhaltsblock).
* Keine Bereinigung von Tool-Call-IDs.
* Keine Reparatur der Zuordnung von Tool-Ergebnissen.
* Keine Turn-Validierung oder -Neuordnung.
* Keine synthetischen Tool-Ergebnisse.
* Kein Entfernen von Thought-Signaturen.

**Google (Generative AI / Gemini CLI / Antigravity)**

* Bereinigung von Tool-Call-IDs: strikt alphanumerisch.
* Reparatur der Zuordnung von Tool-Ergebnissen und synthetische Tool-Ergebnisse.
* Turn-Validierung (Gemini-typische Turn-Alternierung).
* Google-Turn-Ordering-Fixup (einen kleinen User-Bootstrap voranstellen, wenn der Verlauf mit Assistant beginnt).
* Antigravity Claude: Thinking-Signaturen normalisieren; nicht signierte Thinking-Blöcke verwerfen.

**Anthropic / Minimax (Anthropic-kompatibel)**

* Reparatur der Zuordnung von Tool-Ergebnissen und synthetische Tool-Ergebnisse.
* Turn-Validierung (aufeinanderfolgende User-Turns zusammenführen, um strikte Alternierung sicherzustellen).

**Mistral (einschließlich modell-id-basierter Erkennung)**

* Bereinigung von Tool-Call-IDs: strict9 (alphanumerische Länge 9).

**OpenRouter Gemini**

* Thought-Signature-Bereinigung: nicht-Base64-`thought_signature`-Werte entfernen (Base64 beibehalten).

**Alles andere**

* Nur Bildbereinigung.

***

<div id="historical-behavior-pre-2026122">
  ## Historisches Verhalten (vor 2026.1.22)
</div>

Vor dem Release 2026.1.22 wendete OpenClaw mehrere Ebenen der Transkripthygiene an:

* Eine **transcript-sanitize extension** lief bei jedem Kontextaufbau und konnte:
  * die Kopplung von Tool-Nutzung und Ergebnissen reparieren.
  * Tool-Call-IDs bereinigen (einschließlich eines nicht-strikten Modus, der `_`/`-` beibehielt).
* Der Runner führte außerdem anbieter­spezifische Bereinigung durch, was Arbeit duplizierte.
* Zusätzliche Änderungen traten außerhalb der Anbieter-Policy auf, darunter:
  * Entfernen von `<final>`-Tags aus Assistant-Text vor der Speicherung.
  * Verwerfen leerer Assistant-Error-Turns.
  * Kürzen von Assistant-Content nach Tool-Calls.

Diese Komplexität verursachte anbieterübergreifende Regressionen (insbesondere die `openai-responses`
`call_id|fc_id`-Kopplung). Die Bereinigung in 2026.1.22 entfernte die Extension, zentralisierte
die Logik im Runner und ließ OpenAI **no-touch**, abgesehen von der Bereinigung von Bildern.