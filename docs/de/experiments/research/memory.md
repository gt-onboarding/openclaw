---
title: Speicher
summary: "Forschungsnotizen: Offline-Speichersystem für Clawd-arbeitsbereiche (Markdown als maßgebliche Quelle + abgeleiteter Index)"
read_when:
  - Konzeption und Design des arbeitsbereichsspeichers (~/.openclaw/workspace) über tägliche Markdown-Protokolle hinaus
  - Entscheidung: eigenständige CLI vs. tiefe OpenClaw-Integration
  - Hinzufügen von Offline-Erinnerung und -Reflexion (retain/recall/reflect)
---

<div id="workspace-memory-v2-offline-research-notes">
  # Arbeitsbereichs-Speicher v2 (offline): Forschungsnotizen
</div>

Ziel: Clawd-ähnlicher Arbeitsbereich (`agents.defaults.workspace`, Standard `~/.openclaw/workspace`), in dem „Speicher“ als eine Markdown-Datei pro Tag (`memory/YYYY-MM-DD.md`) plus eine kleine Menge stabiler Dateien (z. B. `memory.md`, `SOUL.md`) gespeichert wird.

Dieses Dokument schlägt eine **offline-first** Speicherarchitektur vor, die Markdown als kanonische, überprüfbare Quelle der Wahrheit beibehält, aber über einen abgeleiteten Index einen **strukturierten Abruf** (Suche, Entitätszusammenfassungen, Aktualisierungen der Konfidenzwerte) hinzufügt.

<div id="why-change">
  ## Warum ändern?
</div>

Das aktuelle Setup (eine Datei pro Tag) ist hervorragend für:

- Append-only-Journaling
- manuelle Bearbeitung
- git-gestützte Haltbarkeit + Nachvollziehbarkeit
- niederschwellige Erfassung („einfach aufschreiben“)

Es ist schwach bei:

- Abfragen mit hoher Wiederauffindbarkeit („was haben wir über X entschieden?“, „was war beim letzten Versuch mit Y?“)
- entitätszentrierten Antworten („erzähl mir von Alice / The Castle / warelay“), ohne viele Dateien erneut lesen zu müssen
- Stabilität von Meinungen/Präferenzen (und Belegen, wenn sie sich ändern)
- zeitbezogenen Fragen („was stimmte im November 2025?“) und Konfliktauflösung

<div id="design-goals">
  ## Designziele
</div>

- **Offline**: funktioniert ohne Netzwerkverbindung; kann auf Laptop/Castle laufen; keine Cloud-Abhängigkeit.
- **Nachvollziehbar**: abgerufene Einträge sollten zuordenbar sein (Datei + Position) und klar von der Inferenz getrennt.
- **Geringer Overhead**: tägliches Logging bleibt bei Markdown, kein aufwendiges Schema-Design.
- **Inkrementell**: v1 ist bereits mit nur FTS nützlich; semantische/Vektor-Suche und Graphen sind optionale Upgrades.
- **Agent-freundlich**: macht „Recall innerhalb von Token-Budgets“ einfach (liefert kleine Bündel von Fakten).

<div id="north-star-model-hindsight-letta">
  ## Nordstern-Modell (Hindsight × Letta)
</div>

Zwei Bausteine, die kombiniert werden sollen:

1) **Letta/MemGPT-artige Kontrollschleife**

- ein kleiner „Kern“ bleibt immer im Kontext (Persona + zentrale Nutzerdaten)
- alles andere liegt außerhalb des Kontexts und wird über Tools abgerufen
- Schreibvorgänge in den Speicher erfolgen über explizite Tool-Aufrufe (append/replace/insert), werden persistiert und in der nächsten Runde erneut eingespeist

2) **Hindsight-artiges Speicher-Substrat**

- trenne Beobachtetes von Geglaubtem und Zusammengefasstem
- Unterstützung für retain/recall/reflect
- bewertende Einschätzungen mit Konfidenzgraden, die sich mit neuen Belegen weiterentwickeln können
- entitätsbasiertes Retrieval + zeitliche Abfragen (auch ohne vollständige Wissensgraphen)

<div id="proposed-architecture-markdown-source-of-truth-derived-index">
  ## Vorgeschlagene Architektur (Markdown als Source of Truth + abgeleiteter Index)
</div>

<div id="canonical-store-git-friendly">
  ### Kanonischer Speicher (git-freundlich)
</div>

Verwende `~/.openclaw/workspace` als kanonischen, menschenlesbaren Speicher.

Vorgeschlagene Arbeitsbereichsstruktur:

```
~/.openclaw/workspace/
  memory.md                    # klein: dauerhafte Fakten + Präferenzen (Kern)
  memory/
    YYYY-MM-DD.md              # Tagesprotokoll (anhängen; narrativ)
  bank/                        # „typisierte" Gedächtnisseiten (stabil, überprüfbar)
    world.md                   # objektive Fakten über die Welt
    experience.md              # was der agent getan hat (Ich-Perspektive)
    opinions.md                # subjektive Präferenzen/Bewertungen + Konfidenz + Verweise auf Belege
    entities/
      Peter.md
      The-Castle.md
      warelay.md
      ...
```

Notizen:

* **Tagesprotokoll bleibt Tagesprotokoll**. Es muss nicht in JSON umgewandelt werden.
* Die `bank/`-Dateien sind **kuratierte** Inhalte, werden von Reflection-Jobs erzeugt und können weiterhin von Hand bearbeitet werden.
* `memory.md` bleibt „klein + eher kernnah“: die Dinge, die Clawd in jeder Sitzung sehen soll.


<div id="derived-store-machine-recall">
  ### Abgeleiteter Store (maschineller Abruf)
</div>

Füge einen abgeleiteten Index unterhalb des arbeitsbereichs hinzu (muss nicht zwingend in Git versioniert sein):

```
~/.openclaw/workspace/.memory/index.sqlite
```

Hinterlege das mit:

* SQLite-Schema für Fakten + Entity-Links + Meinungsmetadaten
* SQLite **FTS5** für lexikalen Abruf (schnell, klein, offline)
* optionale Embeddings-Tabelle für semantischen Abruf (ebenfalls offline)

Der Index lässt sich jederzeit **aus Markdown neu aufbauen**.


<div id="retain-recall-reflect-operational-loop">
  ## Speichern / Abrufen / Reflektieren (operativer Zyklus)
</div>

<div id="retain-normalize-daily-logs-into-facts">
  ### Retain: Tages-Logs in „Fakten“ normalisieren
</div>

Die zentrale, hier relevante Erkenntnis aus Hindsight: Speichere **narrative, in sich geschlossene Fakten**, keine winzigen Schnipsel.

Praktische Regel für `memory/YYYY-MM-DD.md`:

* Am Tagesende (oder zwischendurch) fügst du einen `## Retain`‑Abschnitt mit 2–5 Aufzählungspunkten hinzu, die:
  * narrativ sind (Kontext über mehrere Konversationsturns hinweg bleibt erhalten)
  * in sich geschlossen sind (ergeben später eigenständig Sinn)
  * mit Typ + Entity‑Erwähnungen getaggt sind

Beispiel:

```
## Behalten
- W @Peter: Derzeit in Marrakesch (27. Nov.–1. Dez. 2025) für Andys Geburtstag.
- B @warelay: Ich habe den Baileys WS-Absturz behoben, indem ich connection.update-Handler in try/catch eingeschlossen habe (siehe memory/2025-11-27.md).
- O(c=0.95) @Peter: Bevorzugt prägnante Antworten (&lt;1500 Zeichen) auf WhatsApp; lange Inhalte werden in Dateien gespeichert.
```

Minimales Parsing:

* Typpräfix: `W` (Weltwissen), `B` (Erfahrung/biografisch), `O` (Meinung), `S` (Beobachtung/Zusammenfassung; üblicherweise generiert)
* Entitäten: `@Peter`, `@warelay` usw. (Slugs werden auf `bank/entities/*.md` gemappt)
* Konfidenz der Meinung: `O(c=0.0..1.0)` optional

Wenn Autor:innen darüber nicht nachdenken sollen: Der Reflect-Job kann diese Bulletpoints aus dem Rest des Logs ableiten, aber ein expliziter `## Retain`-Abschnitt ist die einfachste „Qualitätsstellschraube“.


<div id="recall-queries-over-the-derived-index">
  ### Recall: Abfragen über den abgeleiteten Index
</div>

Recall sollte Folgendes unterstützen:

- **lexikalisch**: „finde exakte Begriffe / Namen / Befehle“ (FTS5)
- **entitätsbasiert**: „erzähl mir etwas über X“ (Entity-Seiten + entitätsverknüpfte Fakten)
- **zeitlich**: „was ist um den 27. Nov. herum passiert“ / „seit letzter Woche“
- **Meinung**: „was bevorzugt Peter?“ (mit Vertrauensmaß + Belegen)

Das Ausgabeformat sollte agent-freundlich sein und Quellen zitieren:

- `kind` (`world|experience|opinion|observation`)
- `timestamp` (Quelltag bzw. extrahierter Zeitraum, falls vorhanden)
- `entities` (`["Peter","warelay"]`)
- `content` (die narrativ formulierte Tatsache)
- `source` (`memory/2025-11-27.md#L12` etc)

<div id="reflect-produce-stable-pages-update-beliefs">
  ### Reflexion: stabile Seiten erstellen + Überzeugungen aktualisieren
</div>

Reflexion ist ein geplanter Job (täglich oder Herzschlag `ultrathink`), der:

- `bank/entities/*.md` auf Basis aktueller Fakten aktualisiert (Entitätszusammenfassungen)
- die Konfidenz in `bank/opinions.md` basierend auf Verstärkung bzw. Widerspruch aktualisiert
- optional Änderungen an `memory.md` („kernnahe“ dauerhafte Fakten) vorschlägt

Meinungsentwicklung (einfach, nachvollziehbar):

- jede Meinung hat:
  - eine Aussage
  - eine Konfidenz `c ∈ [0,1]`
  - last_updated
  - Beleg-Links (unterstützende und widersprechende Fakt-IDs)
- wenn neue Fakten eintreffen:
  - Kandidatenmeinungen per Entitätsüberlappung + Ähnlichkeit finden (zuerst FTS, später Embeddings)
  - die Konfidenz in kleinen Schritten aktualisieren; große Sprünge erfordern starken Widerspruch + wiederholte Belege

<div id="cli-integration-standalone-vs-deep-integration">
  ## CLI-Integration: Standalone vs. tiefe Integration
</div>

Empfehlung: **tiefe Integration in OpenClaw**, aber mit einer separaten Kernbibliothek.

<div id="why-integrate-into-openclaw">
  ### Warum in OpenClaw integrieren?
</div>

- OpenClaw kennt bereits:
  - den Arbeitsbereichspfad (`agents.defaults.workspace`)
  - das Sitzungsmodell + Herzschläge
  - Logging- und Troubleshooting-Muster
- Du möchtest, dass der Agent selbst die Tools aufruft:
  - `openclaw memory recall "…" --k 25 --since 30d`
  - `openclaw memory reflect --since 7d`

<div id="why-still-split-a-library">
  ### Warum trotzdem eine Bibliothek abtrennen?
</div>

- Memory-Logik ohne Gateway/Runtime testbar halten
- Wiederverwendung in anderen Kontexten (lokale Skripte, zukünftige Desktop-App usw.)

Form:
Das Memory-Tooling ist als kleine CLI- und Bibliotheksebene gedacht, ist derzeit aber noch rein experimentell.

<div id="s-collide-suco-when-to-use-it-research">
  ## „S-Collide“ / SuCo: wann du es einsetzen solltest (Research)
</div>

Wenn sich „S-Collide“ auf **SuCo (Subspace Collision)** bezieht: Es ist ein ANN-Retrieval-Ansatz, der auf gute Recall/Latenz-Trade-offs abzielt, indem er gelernte/strukturierte Kollisionen in Subräumen nutzt (Paper: arXiv 2411.14754, 2024).

Pragmatische Einordnung für `~/.openclaw/workspace`:

- **fang nicht** mit SuCo an.
- starte mit SQLite FTS + (optional) einfachen Embeddings; damit erzielst du sofort die meisten UX-Vorteile.
- ziehe SuCo-/HNSW-/ScaNN-artige Lösungen erst in Betracht, wenn:
  - das Korpus groß ist (Zehntausende/Hunderttausende Chunks)
  - brute-force Embedding-Suche zu langsam wird
  - die Recall-Qualität durch lexikalische Suche merklich limitiert wird

Offline-freundliche Alternativen (in steigender Komplexität):

- SQLite FTS5 + Metadatenfilter (kein ML)
- Embeddings + brute force (funktioniert überraschend weit, wenn die Chunk-Anzahl gering ist)
- HNSW-Index (verbreitet, robust; benötigt eine Bibliotheksanbindung)
- SuCo (forschungsgradig; attraktiv, wenn es eine solide Implementierung gibt, die du einbetten kannst)

Offene Frage:

- was ist das **beste** Offline-Embedding-Modell für „Personal Assistant Memory“ auf deinen Maschinen (Laptop + Desktop)?
  - wenn du bereits Ollama hast: erstelle Embeddings mit einem lokalen Modell; andernfalls packe ein kleines Embedding-Modell in die Toolchain.

<div id="smallest-useful-pilot">
  ## Kleinstes sinnvolles Pilot-Setup
</div>

Wenn du eine minimale, aber trotzdem brauchbare Version möchtest:

- Füge `bank/`-Entity-Seiten und einen Abschnitt `## Retain` in den Tagesprotokollen hinzu.
- Verwende SQLite FTS für den Abruf mit Belegen (Pfad + Zeilennummern).
- Füge Embeddings nur hinzu, wenn die Abrufqualität oder der Umfang es erfordern.

<div id="references">
  ## Referenzen
</div>

- Letta- / MemGPT-Konzepte: „core memory blocks“ + „archival memory“ + toolgesteuerter, selbsteditierender Speicher.
- Hindsight Technical Report: „retain / recall / reflect“, vierteiliges Netzwerk-Speichermodell, narrative Faktenextraktion, Entwicklung der Konfidenz in Einschätzungen.
- SuCo: arXiv 2411.14754 (2024): „Subspace Collision“ Approximate Nearest Neighbor Retrieval.