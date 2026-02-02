---
title: Doctor
summary: "CLI-Referenz für `openclaw doctor` (Health Checks + geführte Reparaturen)"
read_when:
  - Du hast Verbindungs- oder Authentifizierungsprobleme und möchtest geführte Korrekturen
  - Du hast ein Update durchgeführt und möchtest einen schnellen Check
---

<div id="openclaw-doctor">
  # `openclaw doctor`
</div>

Health-Checks + schnelle Korrekturen für das Gateway und die Kanäle.

Verwandte Themen:

* Fehlerbehebung: [Fehlerbehebung](/de/gateway/troubleshooting)
* Sicherheitsprüfung: [Sicherheit](/de/gateway/security)

<div id="examples">
  ## Beispiele
</div>

```bash
openclaw doctor
openclaw doctor --repair
openclaw doctor --deep
```

Hinweise:

* Interaktive Prompts (z. B. für Keychain-/OAuth-Fixes) laufen nur, wenn stdin ein TTY ist und `--non-interactive` **nicht** gesetzt ist. Headless-Runs (Cron-Jobs, Telegram, kein Terminal) überspringen alle Prompts.
* `--fix` (Alias für `--repair`) legt ein Backup unter `~/.openclaw/openclaw.json.bak` an und entfernt unbekannte Konfigurationsschlüssel, wobei jeder entfernte Eintrag aufgelistet wird.

<div id="macos-launchctl-env-overrides">
  ## macOS: `launchctl`-Umgebungsvariablen-Overrides
</div>

Wenn du zuvor `launchctl setenv OPENCLAW_GATEWAY_TOKEN ...` (oder `...PASSWORD`) ausgeführt hast, hat dieser Wert Vorrang vor deiner Konfigurationsdatei und kann zu anhaltenden „unauthorized“-Fehlern führen.

```bash
launchctl getenv OPENCLAW_GATEWAY_TOKEN
launchctl getenv OPENCLAW_GATEWAY_PASSWORD

launchctl unsetenv OPENCLAW_GATEWAY_TOKEN
launchctl unsetenv OPENCLAW_GATEWAY_PASSWORD
```
