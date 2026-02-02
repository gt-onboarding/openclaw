---
title: Fähigkeiten
summary: "UI der macOS-Fähigkeiten-Einstellungen und Gateway-gestützter Status"
read_when:
  - Aktualisierung der macOS-Einstellungs-UI für Fähigkeiten
  - Änderung der Freigaberegeln oder des Installationsverhaltens für Fähigkeiten
---

<div id="skills-macos">
  # Fähigkeiten (macOS)
</div>

Die macOS-App stellt OpenClaw-Fähigkeiten über das Gateway zur Verfügung; sie führt Fähigkeiten nicht lokal aus.

<div id="data-source">
  ## Datenquelle
</div>

- `skills.status` (Gateway) gibt alle Fähigkeiten sowie deren Berechtigung und fehlende Voraussetzungen zurück
  (einschließlich Allowlist-Sperren für mitgelieferte Fähigkeiten).
- Voraussetzungen werden aus `metadata.openclaw.requires` in jeder `SKILL.md` abgeleitet.

<div id="install-actions">
  ## Installationsaktionen
</div>

- `metadata.openclaw.install` definiert Installationsoptionen (brew/node/go/uv).
- Die App ruft `skills.install` auf, um Installationsprogramme auf dem Gateway-Host auszuführen.
- Das Gateway bietet nur einen bevorzugten Installer an, wenn mehrere bereitgestellt werden
  (brew, wenn verfügbar, andernfalls der Node-Manager aus `skills.install`, standardmäßig npm).

<div id="envapi-keys">
  ## Env-/API-Schlüssel
</div>

- Die App speichert Schlüssel in `~/.openclaw/openclaw.json` unter `skills.entries.<skillKey>`.
- `skills.update` aktualisiert `enabled`, `apiKey` und `env`.

<div id="remote-mode">
  ## Remote-Modus
</div>

- Installations- und Konfigurationsupdates erfolgen auf dem Gateway-Host (nicht auf dem lokalen Mac).