---
title: Authentifizierung
summary: "Modell-Authentifizierung: OAuth, API-Schlüssel und setup-token"
read_when:
  - Fehlerbehebung bei Modell-Authentifizierung oder Ablauf von OAuth-Tokens
  - Dokumentation von Authentifizierung oder der Speicherung von Zugangsdaten
---

<div id="authentication">
  # Authentifizierung
</div>

OpenClaw unterstützt OAuth und API-Schlüssel für Modellanbieter. Für Anthropic-Konten empfehlen wir, einen **API-Schlüssel** zu verwenden. Für den Zugriff über ein Claude-Abonnement verwendest du das dauerhafte Token, das mit `claude setup-token` erstellt wird.

Details zum vollständigen OAuth-Ablauf und zum Speicherlayout findest du unter [/concepts/oauth](/de/concepts/oauth).

<div id="recommended-anthropic-setup-api-key">
  ## Empfohlene Anthropic-Einrichtung (API-Schlüssel)
</div>

Wenn du Anthropic direkt verwendest, verwende einen API-Schlüssel.

1. Erstelle einen API-Schlüssel in der Anthropic Console.
2. Hinterlege ihn auf dem **Gateway-Host** (der Maschine, auf der `openclaw gateway` läuft).

```bash
export ANTHROPIC_API_KEY="..."
openclaw models status
```

3. Wenn das Gateway unter systemd/launchd läuft, lege den Schlüssel nach Möglichkeit in
   `~/.openclaw/.env` ab, damit der Daemon ihn lesen kann:

```bash
cat >> ~/.openclaw/.env <<'EOF'
ANTHROPIC_API_KEY=...
EOF
```

Starte dann den Daemon neu (oder starte den Gateway-Prozess neu) und überprüfe es erneut:

```bash
openclaw models status
openclaw doctor
```

Wenn du Umgebungsvariablen nicht selbst verwalten möchtest, kann der Onboarding-Assistent
API-Schlüssel für die Verwendung durch den Daemon speichern: `openclaw onboard`.

Siehe [Hilfe](/de/help) für Details zur Vererbung von Umgebungsvariablen (`env.shellEnv`,
`~/.openclaw/.env`, systemd/launchd).

<div id="anthropic-setup-token-subscription-auth">
  ## Anthropic: setup-token (Abonnement-Authentifizierung)
</div>

Für Anthropic ist der empfohlene Ansatz ein **API-Schlüssel**. Wenn du ein Claude‑Abonnement
verwendest, wird der setup-token-Flow ebenfalls unterstützt. Führe ihn auf dem **Gateway-Host** aus:

```bash
claude setup-token
```

Füge ihn anschließend in OpenClaw ein:

```bash
openclaw models auth setup-token --provider anthropic
```

Wenn das Token auf einem anderen Rechner erzeugt wurde, füge es manuell ein:

```bash
openclaw models auth paste-token --provider anthropic
```

Wenn du eine Anthropic-Fehlermeldung wie die folgende siehst:

```
This credential is only authorized for use with Claude Code and cannot be used for other API requests.
```

…verwende stattdessen einen Anthropic-API-Schlüssel.

Manuelle Token-Eingabe (beliebiger Anbieter; schreibt `auth-profiles.json` und aktualisiert die Konfiguration):

```bash
openclaw models auth paste-token --provider anthropic
openclaw models auth paste-token --provider openrouter
```

Automatisierungsfreundliche Prüfung (Exit-Code `1`, wenn abgelaufen/fehlt, `2`, wenn kurz vor Ablauf):

```bash
openclaw models status --check
```

Optionale Operations-Skripte (systemd/Termux) sind hier dokumentiert:
[/automation/auth-monitoring](/de/automation/auth-monitoring)

> `claude setup-token` erfordert ein interaktives TTY.

<div id="checking-model-auth-status">
  ## Authentifizierungsstatus des Modells prüfen
</div>

```bash
openclaw models status
openclaw doctor
```

<div id="controlling-which-credential-is-used">
  ## Steuern, welche Anmeldedaten verwendet werden
</div>

<div id="per-session-chat-command">
  ### Pro Sitzung (Chat-Befehl)
</div>

Verwende `/model <alias-or-id>@<profileId>`, um bestimmte Anbieter-Zugangsdaten für die aktuelle Sitzung festzulegen (Beispiel-Profile-IDs: `anthropic:default`, `anthropic:work`).

Verwende `/model` (oder `/model list`) für eine kompakte Auswahl; verwende `/model status` für die Vollansicht (Kandidaten + nächstes Auth-Profil sowie Anbieter-Endpunktdetails, falls konfiguriert).

<div id="per-agent-cli-override">
  ### Pro agent (CLI-Override)
</div>

Lege eine explizite Überschreibung der Reihenfolge der Auth-Profile für einen agent fest (gespeichert in der `auth-profiles.json` dieses agent):

```bash
openclaw models auth order get --provider anthropic
openclaw models auth order set --provider anthropic anthropic:default
openclaw models auth order clear --provider anthropic
```

Verwende `--agent <id>`, um einen bestimmten Agenten anzusprechen; lass die Option weg, um den konfigurierten Standardagenten zu verwenden.

<div id="troubleshooting">
  ## Fehlerbehebung
</div>

<div id="no-credentials-found">
  ### „Keine Zugangsdaten gefunden“
</div>

Wenn das Anthropic-Token-Profil nicht vorhanden ist, führe `claude setup-token` auf dem
**Gateway-Host** aus und überprüfe anschließend erneut:

```bash
openclaw models status
```

<div id="token-expiringexpired">
  ### Token läuft ab/ist abgelaufen
</div>

Führe `openclaw models status` aus, um zu prüfen, welches Profil abläuft. Wenn das Profil nicht aufgeführt ist, führe `claude setup-token` erneut aus und füge das Token wieder ein.

<div id="requirements">
  ## Anforderungen
</div>

* Claude Max- oder Pro-Abonnement (für `claude setup-token`)
* Claude Code-CLI installiert (`claude`-Befehl verfügbar)