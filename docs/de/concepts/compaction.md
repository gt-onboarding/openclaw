---
title: Verdichtung
summary: "Kontextfenster + Verdichtung: wie OpenClaw Sitzungen innerhalb der Modellgrenzen hält"
read_when:
  - Du verstehen möchtest, wie automatische Verdichtung und /compact funktionieren
  - Du lange Sitzungen debuggen willst, die an die Kontextgrenzen stoßen
---

<div id="context-window-compaction">
  # Kontextfenster &amp; Verdichtung
</div>

Jedes Modell hat ein **Kontextfenster** (maximale Anzahl von Tokens, die es verarbeiten kann). Länger laufende Chats sammeln Nachrichten und Tool-Ergebnisse an; sobald das Fenster knapp wird, **verdichtet** OpenClaw ältere Verlaufseinträge, um innerhalb dieser Grenze zu bleiben.

<div id="what-compaction-is">
  ## Was Komprimierung ist
</div>

Komprimierung **fasst ältere Teile der Unterhaltung** zu einem kompakten Zusammenfassungseintrag zusammen und lässt aktuelle Nachrichten unverändert. Die Zusammenfassung wird im Sitzungsverlauf gespeichert, sodass zukünftige Anfragen Folgendes verwenden:

* Die durch die Komprimierung erzeugte Zusammenfassung
* Aktuelle Nachrichten nach dem Komprimierungspunkt

Die Komprimierung wird in der JSONL-Historie der Sitzung **persistiert**.

<div id="configuration">
  ## Konfiguration
</div>

Siehe [Compaction-Konfiguration &amp; Modi](/de/concepts/compaction) für die Konfiguration von `agents.defaults.compaction`.

<div id="auto-compaction-default-on">
  ## Auto-Kompaktierung (standardmäßig aktiviert)
</div>

Wenn eine Sitzung sich dem Kontextfenster des Modells nähert oder es überschreitet, löst OpenClaw die Auto-Kompaktierung aus und kann die ursprüngliche Anfrage mit dem kompaktierten Kontext erneut ausführen.

Du siehst dann:

* `🧹 Auto-compaction complete` im ausführlichen Modus
* `/status` mit `🧹 Compactions: <count>`

Vor der Kompaktierung kann OpenClaw eine **stille Memory-Flush-Interaktion** ausführen, um dauerhafte Notizen auf die Festplatte zu schreiben. Siehe [Memory](/de/concepts/memory) für Details und Konfiguration.

<div id="manual-compaction">
  ## Manuelle Verdichtung
</div>

Verwende `/compact` (optional mit Anweisungen), um einen Verdichtungslauf zu erzwingen:

```
/compact Focus on decisions and open questions
```

<div id="context-window-source">
  ## Quelle des Kontextfensters
</div>

Das Kontextfenster ist modellabhängig. OpenClaw verwendet die Modelldefinition aus dem konfigurierten Anbieter-Katalog, um die Grenzwerte zu bestimmen.

<div id="compaction-vs-pruning">
  ## Komprimierung vs. Pruning
</div>

* **Komprimierung**: fasst zusammen und speichert dauerhaft als JSONL.
* **Sitzungs-Pruning**: entfernt alte **Tool-Ergebnisse** nur **im Speicher**, pro Anfrage.

Siehe [/concepts/session-pruning](/de/concepts/session-pruning) für Details zum Pruning.

<div id="tips">
  ## Tipps
</div>

* Verwende `/compact`, wenn Sitzungen veraltet wirken oder der Kontext aufgebläht ist.
* Große Tool-Ausgaben werden bereits gekürzt; weiteres Bereinigen kann den Aufbau von Tool-Ergebnissen zusätzlich reduzieren.
* Wenn du ein komplett leeres Blatt brauchst, starten `/new` oder `/reset` eine neue Sitzungs-ID.