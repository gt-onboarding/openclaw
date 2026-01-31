---
title: Speicher
summary: "Wie der OpenClaw-Speicher funktioniert (Arbeitsbereichsdateien + automatischer Speicher-Flush)"
read_when:
  - Du möchtest das Layout der Speicherdateien und den Workflow verstehen
  - Du möchtest den automatischen Speicher-Flush vor der Kompaktierung anpassen
---

<div id="memory">
  # Speicher
</div>

OpenClaw‑Speicher besteht aus **purem Markdown im Agent‑Arbeitsbereich**. Die Dateien sind
die maßgebliche Quelle; das Modell „erinnert“ sich nur an das, was auf das Dateisystem geschrieben wird.

Suchwerkzeuge für den Speicher werden vom aktiven Memory‑Plugin bereitgestellt (Standard:
`memory-core`). Deaktiviere Memory‑Plugins mit `plugins.slots.memory = "none"`.

<div id="memory-files-markdown">
  ## Memory-Dateien (Markdown)
</div>

Das Standardlayout des Arbeitsbereichs verwendet zwei Memory-Ebenen:

* `memory/YYYY-MM-DD.md`
  * Tägliches Protokoll (nur anhängen).
  * Lies die Einträge von heute und gestern beim Start der Sitzung.
* `MEMORY.md` (optional)
  * Kuratierte Langzeit-Memory.
  * **Nur in der Hauptsitzung (privat) laden** (niemals in Gruppenkontexten).

Diese Dateien liegen im Arbeitsbereich (`agents.defaults.workspace`, Standard
`~/.openclaw/workspace`). Die vollständige Struktur findest du unter [Agent-Arbeitsbereich](/de/concepts/agent-workspace).

<div id="when-to-write-memory">
  ## Wann du Speicher schreiben solltest
</div>

* Entscheidungen, Präferenzen und dauerhafte Fakten kommen in `MEMORY.md`.
* Alltägliche Notizen und laufender Kontext kommen in `memory/YYYY-MM-DD.md`.
* Wenn jemand sagt „Merk dir das“, schreib es auf (nicht im RAM behalten).
* Dieser Bereich ist noch in Entwicklung. Es hilft, das Modell daran zu erinnern, Dinge als Speicher zu speichern; es weiß dann, was zu tun ist.
* Wenn du möchtest, dass etwas wirklich hängenbleibt, **bitte den Bot, es in den Speicher zu schreiben**.

<div id="automatic-memory-flush-pre-compaction-ping">
  ## Automatischer Speicher-Flush (Pre-Compaction-Ping)
</div>

Wenn eine Sitzung **kurz vor der automatischen Kompaktierung** steht, löst OpenClaw eine **stille, agentische Runde** aus, die das Modell daran erinnert, dauerhafte Memory **zu schreiben, bevor** der Kontext komprimiert wird. Die Standard-Prompts geben explizit an, dass das Modell *antworten darf*, aber in der Regel ist `NO_REPLY` die korrekte Antwort, sodass der Benutzer diese Runde nie zu sehen bekommt.

Dies wird über `agents.defaults.compaction.memoryFlush` gesteuert:

```json5
{
  agents: {
    defaults: {
      compaction: {
        reserveTokensFloor: 20000,
        memoryFlush: {
          enabled: true,
          softThresholdTokens: 4000,
          systemPrompt: "Sitzung nähert sich der Kompaktierung. Speichere jetzt dauerhafte Erinnerungen.",
          prompt: "Schreibe dauerhafte Notizen nach memory/YYYY-MM-DD.md; antworte mit NO_REPLY, falls nichts zu speichern ist."
        }
      }
    }
  }
}
```

Details:

* **Weiche Schwelle**: Ein Flush wird ausgelöst, wenn die Sitzungs-Token-Schätzung
  `contextWindow - reserveTokensFloor - softThresholdTokens` überschreitet.
* **Standardmäßig stumm**: Prompts enthalten `NO_REPLY`, sodass nichts zugestellt wird.
* **Zwei Prompts**: Ein Benutzer-Prompt plus ein System-Prompt fügen die Erinnerung an.
* **Ein Flush pro Kompaktierungszyklus** (in `sessions.json` nachverfolgt).
* **Arbeitsbereich muss schreibbar sein**: Wenn die Sitzung in einer Sandbox mit
  `workspaceAccess: "ro"` oder `"none"` läuft, wird der Flush übersprungen.

Den vollständigen Kompaktierungslebenszyklus findest du unter
[Sitzungsverwaltung + Kompaktierung](/de/reference/session-management-compaction).

<div id="vector-memory-search">
  ## Vektorsuche im Speicher
</div>

OpenClaw kann einen kleinen Vektorindex für `MEMORY.md` und `memory/*.md`
(plus beliebige zusätzliche Verzeichnisse oder Dateien, die du aktivierst)
aufbauen, damit semantische Abfragen verwandte Notizen finden, selbst wenn sich
die Formulierungen unterscheiden.

Standardverhalten:

* Standardmäßig aktiviert.
* Überwacht Memory-Dateien auf Änderungen (mit Entprellung).
* Verwendet standardmäßig Remote-Embeddings. Wenn `memorySearch.provider` nicht gesetzt ist, wählt OpenClaw automatisch:
  1. `local`, wenn ein `memorySearch.local.modelPath` konfiguriert ist und die Datei existiert.
  2. `openai`, wenn ein OpenAI-Schlüssel ermittelt werden kann.
  3. `gemini`, wenn ein Gemini-Schlüssel ermittelt werden kann.
  4. Andernfalls bleibt die Speichersuche deaktiviert, bis sie konfiguriert ist.
* Der lokale Modus verwendet node-llama-cpp und kann `pnpm approve-builds` erfordern.
* Verwendet sqlite-vec (falls verfügbar), um die Vektorsuche innerhalb von SQLite zu beschleunigen.

Remote-Embeddings **erfordern** einen API-Schlüssel für den Embedding-Anbieter.
OpenClaw ermittelt Schlüssel aus Auth-Profilen, `models.providers.*.apiKey` oder
Umgebungsvariablen. Codex OAuth deckt nur Chat/Completions ab und erfüllt
**nicht** die Anforderungen für Embeddings in der Speichersuche. Für Gemini
verwende `GEMINI_API_KEY` oder `models.providers.google.apiKey`. Wenn du einen
benutzerdefinierten OpenAI-kompatiblen Endpunkt verwendest, setze
`memorySearch.remote.apiKey` (und optional `memorySearch.remote.headers`).

<div id="additional-memory-paths">
  ### Zusätzliche Memory-Pfade
</div>

Wenn du Markdown-Dateien außerhalb des Standard-Arbeitsbereich-Layouts indizieren möchtest, füge
explizite Pfade hinzu:

```json5
agents: {
  defaults: {
    memorySearch: {
      extraPaths: ["../team-docs", "/srv/shared-notes/overview.md"]
    }
  }
}
```

Hinweise:

* Pfade können absolut oder relativ zum Arbeitsbereich sein.
* Verzeichnisse werden rekursiv nach `.md`-Dateien durchsucht.
* Es werden nur Markdown-Dateien indexiert.
* Symlinks (Dateien oder Verzeichnisse) werden ignoriert.

<div id="gemini-embeddings-native">
  ### Gemini-Embeddings (native)
</div>

Setze den Anbieter auf `gemini`, um die Gemini-Embeddings-API direkt zu verwenden:

```json5
agents: {
  defaults: {
    memorySearch: {
      provider: "gemini",
      model: "gemini-embedding-001",
      remote: {
        apiKey: "YOUR_GEMINI_API_KEY"
      }
    }
  }
}
```

Hinweise:

* `remote.baseUrl` ist optional (standardmäßig die Gemini-API-Basis-URL).
* Mit `remote.headers` kannst du bei Bedarf zusätzliche Header hinzufügen.
* Standardmodell: `gemini-embedding-001`.

Wenn du einen **benutzerdefinierten OpenAI-kompatiblen Endpoint** (OpenRouter, vLLM oder einen Proxy) verwenden möchtest,
kannst du die `remote`-Konfiguration mit dem OpenAI-Anbieter verwenden:

```json5
agents: {
  defaults: {
    memorySearch: {
      provider: "openai",
      model: "text-embedding-3-small",
      remote: {
        baseUrl: "https://api.example.com/v1/",
        apiKey: "YOUR_OPENAI_COMPAT_API_KEY",
        headers: { "X-Custom-Header": "value" }
      }
    }
  }
}
```

Wenn du keinen API-Schlüssel konfigurieren möchtest, verwende `memorySearch.provider = "local"` oder setze
`memorySearch.fallback = "none"`.

Fallbacks:

* `memorySearch.fallback` kann `openai`, `gemini`, `local` oder `none` sein.
* Der Fallback-Anbieter wird nur verwendet, wenn der primäre Embedding-Anbieter fehlschlägt.

Batch-Indexierung (OpenAI + Gemini):

* Standardmäßig aktiviert für OpenAI- und Gemini-Embeddings. Setze `agents.defaults.memorySearch.remote.batch.enabled = false`, um sie zu deaktivieren.
* Das Standardverhalten wartet auf die Fertigstellung des Batches; passe bei Bedarf `remote.batch.wait`, `remote.batch.pollIntervalMs` und `remote.batch.timeoutMinutes` an.
* Setze `remote.batch.concurrency`, um zu steuern, wie viele Batch-Jobs parallel eingereicht werden (Standardwert: 2).
* Der Batch-Modus gilt, wenn `memorySearch.provider = "openai"` oder `"gemini"` ist und verwendet den entsprechenden API-Schlüssel.
* Gemini-Batch-Jobs verwenden den asynchronen Embeddings-Batch-Endpunkt und erfordern die Verfügbarkeit der Gemini Batch API.

Warum OpenAI-Batch schnell und günstig ist:

* Für große Backfills ist OpenAI typischerweise die schnellste von uns unterstützte Option, weil wir viele Embedding-Anfragen in einem einzigen Batch-Job einreichen und OpenAI sie asynchron verarbeiten lassen können.
* OpenAI bietet rabattierte Preise für Batch-API-Workloads, daher sind große Indexierungsläufe in der Regel günstiger, als dieselben Anfragen synchron zu senden.
* Siehe die OpenAI Batch API-Dokumentation und die Preise für Details:
  * https://platform.openai.com/docs/api-reference/batch
  * https://platform.openai.com/pricing

Konfigurationsbeispiel:

```json5
agents: {
  defaults: {
    memorySearch: {
      provider: "openai",
      model: "text-embedding-3-small",
      fallback: "openai",
      remote: {
        batch: { enabled: true, concurrency: 2 }
      },
      sync: { watch: true }
    }
  }
}
```

Tools:

* `memory_search` — gibt Ausschnitte mit Datei- und Zeilenbereichen zurück.
* `memory_get` — liest den Inhalt einer Memory-Datei anhand des Pfads.

Lokaler Modus:

* Setze `agents.defaults.memorySearch.provider = "local"`.
* Gib `agents.defaults.memorySearch.local.modelPath` an (GGUF oder `hf:`-URI).
* Optional: Setze `agents.defaults.memorySearch.fallback = "none"`, um Remote-Fallback zu vermeiden.

<div id="how-the-memory-tools-work">
  ### Wie die Memory-Tools funktionieren
</div>

* `memory_search` durchsucht semantisch Markdown‑Blöcke (~400 Token Zielgröße, 80‑Token‑Überlappung) aus `MEMORY.md` + `memory/**/*.md`. Es gibt Snippet‑Text (begrenzt auf ~700 Zeichen), Dateipfad, Zeilenbereich, Score, Anbieter/Modell und die Information zurück, ob ein Fallback von lokalen auf entfernte Embeddings stattgefunden hat. Es wird kein vollständiger Dateiinhalt zurückgegeben.
* `memory_get` liest eine bestimmte Memory‑Markdown‑Datei (relativ zum Arbeitsbereich), optional ab einer Startzeile und für N Zeilen. Pfade außerhalb von `MEMORY.md` / `memory/` sind nur erlaubt, wenn sie ausdrücklich in `memorySearch.extraPaths` aufgeführt sind.
* Beide Tools sind nur aktiv, wenn `memorySearch.enabled` für den agent zu `true` ausgewertet wird.

<div id="what-gets-indexed-and-when">
  ### Was wann indiziert wird
</div>

* Dateityp: nur Markdown (`MEMORY.md`, `memory/**/*.md` sowie alle `.md`‑Dateien unter `memorySearch.extraPaths`).
* Indexspeicher: pro Agent eine SQLite-Datenbank unter `~/.openclaw/memory/<agentId>.sqlite` (konfigurierbar über `agents.defaults.memorySearch.store.path`, unterstützt das Token `{agentId}`).
* Aktualität: Ein Watcher auf `MEMORY.md`, `memory/` und `memorySearch.extraPaths` markiert den Index als „dirty“ (Entprellzeit 1,5 s). Die Synchronisierung wird beim Start einer Sitzung, bei einer Suche oder in Intervallen geplant und läuft asynchron. Sitzungsprotokolle verwenden Delta-Schwellenwerte, um eine Hintergrundsynchronisierung auszulösen.
* Neuindizierungs-Auslöser: Der Index speichert den **Anbieter/Modell + Endpoint-Fingerprint + Chunking-Parameter**. Wenn sich einer dieser Werte ändert, setzt OpenClaw den gesamten Speicher automatisch zurück und indiziert ihn neu.

<div id="hybrid-search-bm25-vector">
  ### Hybride Suche (BM25 + Vektor)
</div>

Wenn diese Option aktiviert ist, kombiniert OpenClaw:

* **Vektorähnlichkeit** (semantische Übereinstimmung, Formulierungen können abweichen)
* **BM25-Schlüsselwortrelevanz** (exakte Tokens wie IDs, Umgebungsvariablen, Codesymbole)

Wenn auf deiner Plattform keine Volltextsuche verfügbar ist, fällt OpenClaw auf eine reine Vektorsuche zurück.

<div id="why-hybrid">
  #### Warum hybrid?
</div>

Vektorsuche ist großartig für „das bedeutet dasselbe“:

* „Mac Studio Gateway-Host“ vs. „die Maschine, auf der das Gateway läuft“
* „debounce file updates“ vs. „Vermeide eine Indexierung bei jedem Schreibvorgang“

Aber sie kann schwach sein bei exakten, sehr aussagekräftigen Token:

* IDs (`a828e60`, `b3b9895a…`)
* Code-Symbole (`memorySearch.query.hybrid`)
* Fehlermeldungen („sqlite-vec unavailable“)

BM25 (volltextbasierte Suche) ist das Gegenteil: stark bei exakten Token, schwächer bei Paraphrasen.
Hybridsuche ist der pragmatische Mittelweg: **verwende beide Suchsignale**, damit du
gute Ergebnisse sowohl für Anfragen in natürlicher Sprache als auch für „Nadel im Heuhaufen“-Anfragen bekommst.

<div id="how-we-merge-results-the-current-design">
  #### Wie wir Ergebnisse zusammenführen (aktuelles Design)
</div>

Implementierungsskizze:

1. Kandidatenpool von beiden Seiten abrufen:

* **Vektor**: Top `maxResults * candidateMultiplier` nach Kosinus-Ähnlichkeit.
* **BM25**: Top `maxResults * candidateMultiplier` nach FTS5-BM25-Rang (niedriger ist besser).

2. BM25-Rang in einen Score im Bereich etwa 0..1 umwandeln:

* `textScore = 1 / (1 + max(0, bm25Rank))`

3. Kandidaten per Chunk-ID vereinigen und einen gewichteten Score berechnen:

* `finalScore = vectorWeight * vectorScore + textWeight * textScore`

Hinweise:

* `vectorWeight` + `textWeight` wird bei der Auflösung der Konfiguration auf 1,0 normalisiert, sodass sich die Gewichte wie Prozentsätze verhalten.
* Wenn Embeddings nicht verfügbar sind (oder der anbieter einen Nullvektor zurückgibt), führen wir trotzdem BM25 aus und liefern Schlüsselworttreffer zurück.
* Wenn FTS5 nicht erzeugt werden kann, behalten wir eine reine Vektorsuche bei (kein harter Fehler).

Das ist zwar nicht „IR-theoretisch perfekt“, aber es ist einfach, schnell und verbessert in der Praxis tendenziell Recall/Precision auf realen Notizen.
Wenn wir es später ausgefeilter machen wollen, sind typische nächste Schritte Reciprocal Rank Fusion (RRF) oder Score-Normalisierung
(Min/Max oder z-Score) vor dem Mischen.

Konfiguration:

```json5
agents: {
  defaults: {
    memorySearch: {
      query: {
        hybrid: {
          enabled: true,
          vectorWeight: 0.7,
          textWeight: 0.3,
          candidateMultiplier: 4
        }
      }
    }
  }
}
```

<div id="embedding-cache">
  ### Embedding-Cache
</div>

OpenClaw kann **Chunk-Embeddings** in SQLite cachen, damit beim Reindizieren und bei häufigen Updates (insbesondere Sitzungstranskripten) unveränderter Text nicht erneut eingebettet werden muss.

Konfiguration:

```json5
agents: {
  defaults: {
    memorySearch: {
      cache: {
        enabled: true,
        maxEntries: 50000
      }
    }
  }
}
```

<div id="session-memory-search-experimental">
  ### Sitzungsspeicher-Suche (experimentell)
</div>

Du kannst optional **Sitzungsprotokolle** indexieren und sie über `memory_search` verfügbar machen.
Dies wird über ein experimentelles Flag gesteuert.

```json5
agents: {
  defaults: {
    memorySearch: {
      experimental: { sessionMemory: true },
      sources: ["memory", "sessions"]
    }
  }
}
```

Hinweise:

* Die Sitzungsindizierung ist **opt-in** (standardmäßig deaktiviert).
* Sitzungsaktualisierungen werden entprellt und **asynchron indiziert**, sobald sie Delta-Schwellenwerte überschreiten (Best-effort).
* `memory_search` wartet niemals auf die Indizierung; Ergebnisse können leicht veraltet sein, bis die Hintergrundsynchronisierung abgeschlossen ist.
* Ergebnisse enthalten weiterhin nur Snippets; `memory_get` bleibt auf Memory-Dateien beschränkt.
* Die Sitzungsindizierung ist pro Agent isoliert (nur die Sitzungsprotokolle dieses Agents werden indiziert).
* Sitzungsprotokolle liegen auf der Festplatte (`~/.openclaw/agents/<agentId>/sessions/*.jsonl`). Jeder Prozess/Jeder Benutzer mit Dateisystemzugriff kann sie read, behandle daher den Dateisystemzugriff als Vertrauensgrenze. Für strengere Isolierung führe Agenten unter separaten OS-Benutzern oder auf separaten Hosts aus.

Delta-Schwellenwerte (Standardwerte angegeben):

```json5
agents: {
  defaults: {
    memorySearch: {
      sync: {
        sessions: {
          deltaBytes: 100000,   // ~100 KB
          deltaMessages: 50     // JSONL-Zeilen
        }
      }
    }
  }
}
```

<div id="sqlite-vector-acceleration-sqlite-vec">
  ### SQLite-Vektorbeschleunigung (sqlite-vec)
</div>

Wenn die sqlite-vec-Erweiterung verfügbar ist, speichert OpenClaw Embeddings in
einer virtuellen SQLite-Tabelle (`vec0`) und führt Vektorabstandsabfragen
direkt in der Datenbank aus. Dadurch bleibt die Suche schnell, ohne jedes
Embedding in JS laden zu müssen.

Konfiguration (optional):

```json5
agents: {
  defaults: {
    memorySearch: {
      store: {
        vector: {
          enabled: true,
          extensionPath: "/path/to/sqlite-vec"
        }
      }
    }
  }
}
```

Hinweise:

* `enabled` ist standardmäßig auf `true` gesetzt; wenn deaktiviert, fällt die Suche auf prozessinterne
  Cosine-Ähnlichkeit über gespeicherte Embeddings zurück.
* Wenn die sqlite-vec-Erweiterung fehlt oder nicht geladen werden kann, protokolliert OpenClaw den
  Fehler und fährt mit dem JS-Fallback fort (keine Vektortabelle).
* `extensionPath` überschreibt den mitgelieferten sqlite-vec-Pfad (nützlich für benutzerdefinierte Builds
  oder nicht standardmäßige Installationspfade).

<div id="local-embedding-auto-download">
  ### Automatischer Download lokaler Embeddings
</div>

* Standardmodell für lokale Embeddings: `hf:ggml-org/embeddinggemma-300M-GGUF/embeddinggemma-300M-Q8_0.gguf` (~0,6 GB).
* Wenn `memorySearch.provider = "local"` ist, ermittelt `node-llama-cpp` den `modelPath`. Falls die GGUF-Datei fehlt, wird sie **automatisch** in den Cache (oder in `local.modelCacheDir`, falls gesetzt) heruntergeladen und anschließend geladen. Unterbrochene Downloads werden bei einem erneuten Versuch fortgesetzt.
* Anforderung für nativen Build: `pnpm approve-builds` ausführen, `node-llama-cpp` auswählen, dann `pnpm rebuild node-llama-cpp`.
* Fallback: Wenn das lokale Setup fehlschlägt und `memorySearch.fallback = "openai"` ist, wird automatisch auf Remote-Embeddings (`openai/text-embedding-3-small`, sofern nicht anders konfiguriert) umgestellt und der Grund protokolliert.

<div id="custom-openai-compatible-endpoint-example">
  ### Beispiel für einen benutzerdefinierten, OpenAI-kompatiblen Endpunkt
</div>

```json5
agents: {
  defaults: {
    memorySearch: {
      provider: "openai",
      model: "text-embedding-3-small",
      remote: {
        baseUrl: "https://api.example.com/v1/",
        apiKey: "YOUR_REMOTE_API_KEY",
        headers: {
          "X-Organization": "org-id",
          "X-Project": "project-id"
        }
      }
    }
  }
}
```

Hinweise:

* `remote.*` hat Vorrang vor `models.providers.openai.*`.
* `remote.headers` werden mit den OpenAI-Headern zusammengeführt; bei Schlüsselkonflikten gewinnt `remote`. Lassen Sie `remote.headers` weg, um die OpenAI-Standardwerte zu verwenden.
