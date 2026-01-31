---
title: Status
summary: "Schritte zur Überprüfung der Kanalverbindung"
read_when:
  - Diagnose der WhatsApp-Kanalverbindung
---

<div id="health-checks-cli">
  # Health Checks (CLI)
</div>

Kurzanleitung, um Kanalverbindungen zu überprüfen, ohne herumraten zu müssen.

<div id="quick-checks">
  ## Schnelle Checks
</div>

- `openclaw status` — lokale Zusammenfassung: Gateway-Erreichbarkeit/Modus, Update-Hinweis, Alter der verknüpften Channel-Authentifizierung, Sitzungen + letzte Aktivität.
- `openclaw status --all` — vollständige lokale Diagnose (read-only, farbig, gefahrlos zum Einfügen in Debug-/Fehlermeldungen).
- `openclaw status --deep` — prüft zusätzlich das laufende Gateway (channel-spezifische Prüfungen, sofern unterstützt).
- `openclaw health --json` — fordert vom laufenden Gateway einen vollständigen Health-Snapshot an (nur WS; keine direkte Baileys-Socket-Verbindung).
- Sende `/status` als eigenständige Nachricht in WhatsApp/WebChat, um eine Statusantwort zu erhalten, ohne den Agent aufzurufen.
- Logs: `tail /tmp/openclaw/openclaw-*.log` und nach `web-heartbeat`, `web-reconnect`, `web-auto-reply`, `web-inbound` filtern.

<div id="deep-diagnostics">
  ## Detaillierte Diagnose
</div>

- Zugangsdaten auf dem Datenträger: `ls -l ~/.openclaw/credentials/whatsapp/<accountId>/creds.json` (mtime sollte aktuell sein).
- Sitzungsspeicher: `ls -l ~/.openclaw/agents/<agentId>/sessions/sessions.json` (Pfad kann in der Konfiguration überschrieben werden). Anzahl und letzte Empfänger werden über `status` angezeigt.
- Relink-Flow: `openclaw channels logout && openclaw channels login --verbose`, wenn Statuscodes 409–515 oder `loggedOut` in den Logs erscheinen. (Hinweis: Der QR-Anmeldeflow startet nach der Kopplung einmal automatisch neu für Status 515.)

<div id="when-something-fails">
  ## Wenn etwas fehlschlägt
</div>

- `logged out` oder Status 409–515 → erneut verknüpfen mit `openclaw channels logout` und dann `openclaw channels login`.
- Gateway nicht erreichbar → starte ihn: `openclaw gateway --port 18789` (verwende `--force`, wenn der Port belegt ist).
- Keine eingehenden Nachrichten → prüfe, ob das verknüpfte Telefon online ist und der Absender zugelassen ist (`channels.whatsapp.allowFrom`); für Gruppenchats stelle sicher, dass Allowlist- und Mention-Regeln übereinstimmen (`channels.whatsapp.groups`, `agents.list[].groupChat.mentionPatterns`).

<div id="dedicated-health-command">
  ## Dedizierter Befehl „health“
</div>

`openclaw health --json` fragt den laufenden Gateway nach seinem Health-Snapshot ab (keine direkten Channel-Sockets von der CLI aus). Der Befehl meldet, sofern verfügbar, das Alter verknüpfter Zugangsdaten/Auth-Daten, zusammenfassende Prüfungen pro Channel, eine Zusammenfassung des Sitzungsspeichers und die Dauer der Prüfung. Er beendet sich mit einem von Null verschiedenen Exit-Code, wenn der Gateway nicht erreichbar ist oder die Prüfung fehlschlägt bzw. ein Timeout auftritt. Verwende `--timeout <ms>`, um den Standardwert von 10 s zu überschreiben.