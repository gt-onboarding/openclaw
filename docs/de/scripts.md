---
title: Skripte
summary: "Repository-Skripte: Zweck, Umfang und Sicherheitshinweise"
read_when:
  - Ausführen von Skripten aus dem Repository
  - Hinzufügen oder Ändern von Skripten unter ./scripts
---

<div id="scripts">
  # Skripte
</div>

Das Verzeichnis `scripts/` enthält Hilfsskripte für lokale Workflows und Betriebs-/Ops-Aufgaben.
Verwende diese, wenn eine Aufgabe eindeutig an ein Skript gekoppelt ist; ansonsten bevorzuge die CLI.

<div id="conventions">
  ## Konventionen
</div>

* Skripte sind **optional**, sofern sie nicht in der Dokumentation oder in Release-Checklisten aufgeführt sind.
* Bevorzuge nach Möglichkeit CLI-Oberflächen, wo sie verfügbar sind (Beispiel: Auth-Monitoring verwendet `openclaw models status --check`).
* Gehe davon aus, dass Skripte host­spezifisch sind; lies sie, bevor du sie auf einem neuen Rechner ausführst.

<div id="git-hooks">
  ## Git-Hooks
</div>

* `scripts/setup-git-hooks.js`: Best-Effort-Einrichtung von `core.hooksPath`, wenn du dich in einem Git-Repository befindest.
* `scripts/format-staged.js`: Pre-Commit-Formatter für zum Commit vorgemerkte Dateien in `src/` und `test/`.

<div id="auth-monitoring-scripts">
  ## Auth-Monitoring-Skripte
</div>

Die Auth-Monitoring-Skripte sind hier dokumentiert:
[/automation/auth-monitoring](/de/automation/auth-monitoring)

<div id="when-adding-scripts">
  ## Beim Hinzufügen von Skripten
</div>

* Halte Skripte übersichtlich und gut dokumentiert.
* Füge einen kurzen Eintrag in der entsprechenden Dokumentation hinzu (oder lege sie an, falls sie noch nicht existiert).