---
title: Authentifizierungsüberwachung
summary: "OAuth-Ablaufzeiten für Modellanbieter überwachen"
read_when:
  - Einrichten von Monitoring oder Alerts für OAuth-Ablaufzeiten
  - Automatisierung von OAuth-Refresh-Prüfungen für Claude Code / Codex
---

<div id="auth-monitoring">
  # Auth-Überwachung
</div>

OpenClaw stellt den OAuth-Ablaufzustand über `openclaw models status` bereit. Verwende das für
Automatisierung und Benachrichtigungen; Skripte sind optionale Ergänzungen für Telefon-Workflows.

<div id="preferred-cli-check-portable">
  ## Empfohlen: CLI-Check (portabel)
</div>

```bash
openclaw models status --check
```

Exit-Codes:

* `0`: OK
* `1`: abgelaufene oder fehlende Zugangsdaten
* `2`: laufen in Kürze ab (innerhalb von 24 Stunden)

Das funktioniert mit cron/systemd und benötigt keine zusätzlichen Skripte.


<div id="optional-scripts-ops-phone-workflows">
  ## Optionale Skripte (Ops- / Telefon-Workflows)
</div>

Diese liegen unter `scripts/` und sind **optional**. Sie setzen SSH-Zugriff auf den
Gateway-Host voraus und sind auf systemd + Termux abgestimmt.

- `scripts/claude-auth-status.sh` verwendet jetzt `openclaw models status --json` als
  maßgebliche Quelle (greift auf direktes Lesen von Dateien zurück, wenn die CLI nicht verfügbar ist),
  halte daher `openclaw` für Timer im `PATH` verfügbar.
- `scripts/auth-monitor.sh`: cron-/systemd-Timer-Target; sendet Alarme (ntfy oder Telefon).
- `scripts/systemd/openclaw-auth-monitor.{service,timer}`: systemd-User-Timer.
- `scripts/claude-auth-status.sh`: Claude Code + OpenClaw Auth-Checker (full/json/simple).
- `scripts/mobile-reauth.sh`: geführter Re-Auth-Flow über SSH.
- `scripts/termux-quick-auth.sh`: One-Tap-Widget-Status + Auth-URL öffnen.
- `scripts/termux-auth-widget.sh`: vollständiger geführter Widget-Flow.
- `scripts/termux-sync-widget.sh`: Claude-Code-Creds → OpenClaw synchronisieren.

Wenn du keine Telefonautomatisierung oder systemd-Timer benötigst, kannst du diese Skripte weglassen.