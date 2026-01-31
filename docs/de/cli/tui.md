---
title: Tui
summary: "CLI-Referenz für `openclaw tui` (Terminal-UI, verbunden mit dem Gateway)"
read_when:
  - Du möchtest eine Terminal-UI für das Gateway (gut für Remote-Zugriff)
  - Du möchtest URL/Token/Sitzung über Skripte übergeben
---

<div id="openclaw-tui">
  # `openclaw tui`
</div>

Öffnet die mit dem Gateway verbundene Terminal-UI.

Verwandte Themen:

* TUI-Anleitung: [TUI](/de/tui)

<div id="examples">
  ## Beispiele
</div>

```bash
openclaw tui
openclaw tui --url ws://127.0.0.1:18789 --token <token>
openclaw tui --session main --deliver
```
