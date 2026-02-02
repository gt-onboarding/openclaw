---
title: Kanäle
summary: "CLI-Referenz für `openclaw channels` (Konten, Status, An- und Abmelden, Logs)"
read_when:
  - Du möchtest Kanal-Konten hinzufügen oder entfernen (WhatsApp/Telegram/Discord/Google Chat/Slack/Mattermost (Plugin)/Signal/iMessage)
  - Du möchtest den Kanalstatus überprüfen oder Kanal-Logs verfolgen
---

<div id="openclaw-channels">
  # `openclaw channels`
</div>

Verwalte Chatkanalkonten und deren Laufzeitstatus im Gateway.

Zugehörige Dokumentation:

* Kanal-Anleitungen: [Channels](/de/channels/index)
* Gateway-Konfiguration: [Configuration](/de/gateway/configuration)

<div id="common-commands">
  ## Häufig verwendete Befehle
</div>

```bash
openclaw channels list
openclaw channels status
openclaw channels capabilities
openclaw channels capabilities --channel discord --target channel:123
openclaw channels resolve --channel slack "#general" "@jane"
openclaw channels logs --channel all
```

<div id="add-remove-accounts">
  ## Accounts hinzufügen/entfernen
</div>

```bash
openclaw channels add --channel telegram --token <bot-token>
openclaw channels remove --channel telegram --delete
```

Tipp: `openclaw channels add --help` zeigt kanalspezifische Flags (Token, App-Token, signal-cli-Pfade usw.) an.

<div id="login-logout-interactive">
  ## Anmeldung / Abmeldung (interaktiv)
</div>

```bash
openclaw channels login --channel whatsapp
openclaw channels logout --channel whatsapp
```

<div id="troubleshooting">
  ## Fehlerbehebung
</div>

* Führe `openclaw status --deep` für eine umfassende Prüfung aus.
* Verwende `openclaw doctor` für geführte Korrekturen.
* `openclaw channels list` gibt `Claude: HTTP 403 ... user:profile` aus → für den Nutzungssnapshot wird der Scope `user:profile` benötigt. Verwende `--no-usage`, gib alternativ einen claude.ai-Sitzungsschlüssel an (`CLAUDE_WEB_SESSION_KEY` / `CLAUDE_WEB_COOKIE`), oder authentifiziere dich erneut über die Claude Code CLI.

<div id="capabilities-probe">
  ## Fähigkeitenabfrage
</div>

Ruft Hinweise zu anbieterfähigkeiten (Intents/Scopes, sofern verfügbar) sowie zur Unterstützung statischer Funktionen ab:

```bash
openclaw channels capabilities
openclaw channels capabilities --channel discord --target channel:123
```

Hinweise:

* `--channel` ist optional; lass sie weg, um alle Kanäle (einschließlich Erweiterungen) aufzulisten.
* `--target` akzeptiert `channel:<id>` oder eine numerische Channel-ID und gilt nur für Discord.
* Probes sind anbieterspezifisch: Discord-Intents + optionale Channel-Berechtigungen; Slack-Bot + Benutzer-Scopes; Telegram-Bot-Flags + Webhook; Signal-Daemon-Version; MS-Teams-App-Token + Graph-Rollen/Scopes (entsprechend vermerkt, sofern bekannt). Kanäle ohne Probes geben `Probe: unavailable` aus.

<div id="resolve-names-to-ids">
  ## Namen in IDs auflösen
</div>

Löse Channel- und Benutzernamen mithilfe des Anbieterverzeichnisses in IDs auf:

```bash
openclaw channels resolve --channel slack "#general" "@jane"
openclaw channels resolve --channel discord "My Server/#support" "@someone"
openclaw channels resolve --channel matrix "Project Room"
```

Hinweise:

* Verwende `--kind user|group|auto`, um den Zieltyp zu erzwingen.
* Die Namensauflösung bevorzugt aktive Einträge, wenn mehrere Einträge denselben Namen haben.
