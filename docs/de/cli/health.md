---
title: Health-Check
summary: "CLI-Referenz für `openclaw health` (Gateway-Health-Endpoint per RPC)"
read_when:
  - Wenn du den Zustand des laufenden Gateways schnell prüfen möchtest
---

<div id="openclaw-health">
  # `openclaw health`
</div>

Ruft den Health-Status des laufenden Gateways ab.

```bash
openclaw health
openclaw health --json
openclaw health --verbose
```

Hinweise:

* `--verbose` führt Live-Checks aus und gibt Zeitmessungen für jedes Konto aus, wenn mehrere Konten konfiguriert sind.
* Die Ausgabe umfasst die Sitzungsspeicher der einzelnen Agenten, wenn mehrere Agenten konfiguriert sind.
