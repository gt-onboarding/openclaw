---
title: Logs
summary: "CLI-Referenz für `openclaw logs` (Gateway-Logs per RPC im Tail-Modus anzeigen)"
read_when:
  - Du musst Gateway-Logs aus der Ferne verfolgen (ohne SSH)
  - Du möchtest JSON-Logzeilen für deine Tools
---

<div id="openclaw-logs">
  # `openclaw logs`
</div>

Logdateien des Gateway per RPC live anzeigen (funktioniert im Remote-Modus).

Verwandtes:

* Logging-Übersicht: [Logging](/de/logging)

<div id="examples">
  ## Beispiele
</div>

```bash
openclaw logs
openclaw logs --follow
openclaw logs --json
openclaw logs --limit 500
```
