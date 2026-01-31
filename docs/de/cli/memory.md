---
title: Speicher
summary: "CLI-Referenz zu `openclaw memory` (status/index/search)"
read_when:
  - Du möchtest semantischen Speicher indizieren oder durchsuchen
  - Du debuggst Probleme mit der Speicherverfügbarkeit oder Indexierung
---

<div id="openclaw-memory">
  # `openclaw memory`
</div>

Verwalte semantische Speicherindizierung und -Suche.
Bereitgestellt durch das aktive Memory-Plugin (Standard: `memory-core`; setze `plugins.slots.memory = "none"`, um es zu deaktivieren).

Verwandte Themen:

* Konzept: [Memory](/de/concepts/memory)
* Plugins: [Plugins](/de/plugins)

<div id="examples">
  ## Beispiele
</div>

```bash
openclaw memory status
openclaw memory status --deep
openclaw memory status --deep --index
openclaw memory status --deep --index --verbose
openclaw memory index
openclaw memory index --verbose
openclaw memory search "release checklist"
openclaw memory status --agent main
openclaw memory index --agent main --verbose
```

<div id="options">
  ## Optionen
</div>

Allgemein:

* `--agent <id>`: auf einen einzelnen agent beschränken (Standard: alle konfigurierten Agenten).
* `--verbose`: während der Prüfungen und beim Indexieren detaillierte Logs ausgeben.

Hinweise:

* `memory status --deep` prüft Vektor- und Embedding-Verfügbarkeit.
* `memory status --deep --index` führt ein Reindex aus, wenn der Store „dirty“ ist.
* `memory index --verbose` gibt Details pro Phase aus (anbieter, Modell, Quellen, Batch-Aktivität).
* `memory status` umfasst alle zusätzlichen Pfade, die über `memorySearch.extraPaths` konfiguriert sind.