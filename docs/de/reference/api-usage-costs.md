---
title: API-Nutzungskosten
summary: "Nachvollziehen, was Kosten verursachen kann, welche Keys verwendet werden und wie du die Nutzung einsehen kannst"
read_when:
  - Du verstehen willst, welche Funktionen kostenpflichtige APIs aufrufen können
  - Du Keys, Kosten und Sichtbarkeit der Nutzung prüfen musst
  - Du die Kostenberichte unter /status oder /usage erklärst
---

<div id="api-usage-costs">
  # API-Nutzung &amp; Kosten
</div>

In diesem Dokument sind **Funktionen aufgeführt, die API-Schlüssel verwenden können**, und es wird erläutert, wo deren Kosten angezeigt werden. Der Fokus liegt auf OpenClaw-Funktionen, die anbieterseitige Nutzung oder kostenpflichtige API-Aufrufe erzeugen können.

<div id="where-costs-show-up-chat-cli">
  ## Wo Kosten sichtbar werden (Chat + CLI)
</div>

**Kosten-Snapshot pro Sitzung**

* `/status` zeigt das aktuelle Sitzungsmodell, die Kontextnutzung und die Tokenanzahl der letzten Antwort.
* Wenn das Modell **API-Key-Auth** verwendet, zeigt `/status` auch die **geschätzten Kosten** für die letzte Antwort.

**Kosten-Fußzeile pro Nachricht**

* `/usage full` hängt jeder Antwort eine Nutzungs-Fußzeile an, einschließlich **geschätzter Kosten** (nur bei API-Key).
* `/usage tokens` zeigt nur Token; OAuth-Flows blenden die Kosten in Dollar aus.

**CLI-Nutzungsfenster (Anbieter-Kontingente)**

* `openclaw status --usage` und `openclaw channels list` zeigen **Nutzungsfenster** des Anbieters
  (Kontingent-Snapshots, keine Kosten pro Nachricht).

Siehe [Tokenverbrauch &amp; Kosten](/de/token-use) für Details und Beispiele.

<div id="how-keys-are-discovered">
  ## Wie Schlüssel ermittelt werden
</div>

OpenClaw kann Zugangsdaten/API-Schlüssel aus folgenden Quellen übernehmen:

* **Auth-Profile** (pro Agent, gespeichert in `auth-profiles.json`).
* **Umgebungsvariablen** (z. B. `OPENAI_API_KEY`, `BRAVE_API_KEY`, `FIRECRAWL_API_KEY`).
* **Konfiguration** (`models.providers.*.apiKey`, `tools.web.search.*`, `tools.web.fetch.firecrawl.*`,
  `memorySearch.*`, `talk.apiKey`).
* **Fähigkeiten** (`skills.entries.<name>.apiKey`), die Schlüssel in die Umgebungsvariablen des Fähigkeiten-Prozesses exportieren können.

<div id="features-that-can-spend-keys">
  ## Funktionen, die Schlüsselguthaben verbrauchen
</div>

<div id="1-core-model-responses-chat-tools">
  ### 1) Antworten des Kernmodells (Chat + Tools)
</div>

Jede Antwort oder jeder Toolaufruf verwendet den **aktuellen Modellanbieter** (OpenAI, Anthropic, etc.). Dies ist die
primäre Quelle für Nutzung und Kosten.

Siehe [Modelle](/de/providers/models) für Preiskonfiguration und [Token-Nutzung &amp; -Kosten](/de/token-use) zur Darstellung.

<div id="2-media-understanding-audioimagevideo">
  ### 2) Medienverständnis (Audio/Bild/Video)
</div>

Eingehende Medien können zusammengefasst oder transkribiert werden, bevor die Antwort generiert wird. Dafür werden Modell-/Anbieter-APIs verwendet.

* Audio: OpenAI / Groq / Deepgram (jetzt **automatisch aktiviert**, wenn API-Schlüssel vorhanden sind).
* Bild: OpenAI / Anthropic / Google.
* Video: Google.

Siehe [Medienverständnis](/de/nodes/media-understanding).

<div id="3-memory-embeddings-semantic-search">
  ### 3) Memory-Embeddings + semantische Suche
</div>

Semantische Memory-Suche verwendet **Embedding-APIs**, wenn sie für Remote-Anbieter konfiguriert ist:

* `memorySearch.provider = "openai"` → OpenAI-Embeddings
* `memorySearch.provider = "gemini"` → Gemini-Embeddings
* Optionaler Fallback auf OpenAI, wenn lokale Embeddings fehlschlagen

Du kannst alles lokal halten mit `memorySearch.provider = "local"` (keine API-Nutzung).

Siehe [Memory](/de/concepts/memory).

<div id="4-web-search-tool-brave-perplexity-via-openrouter">
  ### 4) Web-Suchtool (Brave / Perplexity via OpenRouter)
</div>

`web_search` verwendet API-Schlüssel und kann Nutzungskosten verursachen:

* **Brave Search API**: `BRAVE_API_KEY` oder `tools.web.search.apiKey`
* **Perplexity** (über OpenRouter): `PERPLEXITY_API_KEY` oder `OPENROUTER_API_KEY`

**Kostenloses Brave-Kontingent (großzügig):**

* **2.000 Anfragen/Monat**
* **1 Anfrage/Sekunde**
* **Kreditkarte erforderlich** zur Verifizierung (keine Belastung, solange du kein Upgrade durchführst)

Siehe [Web-Tools](/de/tools/web).

<div id="5-web-fetch-tool-firecrawl">
  ### 5) Web-Fetch-Tool (Firecrawl)
</div>

`web_fetch` kann **Firecrawl** aufrufen, wenn ein API-Schlüssel gesetzt ist:

* `FIRECRAWL_API_KEY` oder `tools.web.fetch.firecrawl.apiKey`

Wenn Firecrawl nicht konfiguriert ist, fällt das Tool auf direkten Fetch + Readability zurück (kein kostenpflichtiger API-Dienst).

Siehe [Web-Tools](/de/tools/web).

<div id="6-provider-usage-snapshots-statushealth">
  ### 6) Anbieter-Nutzungs-Snapshots (Status/Gesundheit)
</div>

Einige Statusbefehle rufen **Anbieter-Nutzungsendpunkte** auf, um Kontingentfenster oder den Zustand der Authentifizierung anzuzeigen.
Dies sind typischerweise Aufrufe mit geringem Umfang, belasten aber dennoch die APIs der Anbieter:

* `openclaw status --usage`
* `openclaw models status --json`

Siehe [Models CLI](/de/cli/models).

<div id="7-compaction-safeguard-summarization">
  ### 7) Zusammenfassung durch Kompaktionsschutz
</div>

Der Kompaktionsschutz kann den Sitzungsverlauf mithilfe des **aktuellen Modells** zusammenfassen; dabei werden die APIs des Anbieters aufgerufen.

Siehe [Sitzungsverwaltung + Kompaktierung](/de/reference/session-management-compaction).

<div id="8-model-scan-probe">
  ### 8) Modellscan/-abfrage
</div>

`openclaw models scan` kann OpenRouter-Modelle abfragen und verwendet `OPENROUTER_API_KEY`, wenn
das Scannen aktiviert ist.

Siehe [Models CLI](/de/cli/models).

<div id="9-talk-speech">
  ### 9) Talk (Sprache)
</div>

Der Talk-Modus kann **ElevenLabs** verwenden, wenn er konfiguriert ist:

* `ELEVENLABS_API_KEY` oder `talk.apiKey`

Siehe [Talk-Modus](/de/nodes/talk).

<div id="10-skills-third-party-apis">
  ### 10) Fähigkeiten (Drittanbieter-APIs)
</div>

Fähigkeiten können `apiKey` in `skills.entries.<name>.apiKey` ablegen. Wenn eine Fähigkeit diesen Schlüssel für externe
APIs verwendet, können dabei Kosten gemäß dem jeweiligen anbieter der Fähigkeit anfallen.

Siehe [Fähigkeiten](/de/tools/skills).