---
title: Zurücksetzen
summary: "CLI-Referenz für `openclaw reset` (lokalen Zustand/Konfiguration zurücksetzen)"
read_when:
  - Du möchtest den lokalen Zustand löschen, während die CLI installiert bleibt
  - Du möchtest einen Dry-Run sehen, welche Daten entfernt würden
---

<div id="openclaw-reset">
  # `openclaw reset`
</div>

Setzt die lokale Konfiguration und den lokalen Zustand zurück (die CLI bleibt installiert).

```bash
openclaw reset
openclaw reset --dry-run
openclaw reset --scope config+creds+sessions --yes --non-interactive
```
