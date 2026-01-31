---
title: Status
summary: "Wie die macOS-App Gateway-/Baileys-Health-Status meldet"
read_when:
  - Debugging der Health-Indikatoren der macOS-App
---

<div id="health-checks-on-macos">
  # Health Checks unter macOS
</div>

So prüfst du in der Menüleisten-App, ob der verknüpfte Kanal ordnungsgemäß funktioniert.

<div id="menu-bar">
  ## Menüleiste
</div>

* Statuspunkt spiegelt jetzt den Status von Baileys wider:
  * Grün: verbunden + Socket wurde kürzlich geöffnet.
  * Orange: Verbindung wird aufgebaut/erneuter Verbindungsversuch.
  * Rot: abgemeldet oder Prüfung fehlgeschlagen.
* Zweite Zeile zeigt „linked · auth 12m“ oder den Fehlergrund an.
* Der Menüeintrag „Run Health Check“ löst eine Prüfung bei Bedarf aus.

<div id="settings">
  ## Einstellungen
</div>

* Der Tab „Allgemein“ enthält eine Health-Karte, die Folgendes anzeigt: Alter der verknüpften Authentifizierung, Sitzungsspeicher-Pfad/-Anzahl, Zeitpunkt der letzten Prüfung, letzter Fehler/Statuscode sowie Schaltflächen für Run Health Check / Reveal Logs.
* Verwendet einen zwischengespeicherten Snapshot, sodass die UI sofort lädt und im Offline-Fall sauber zurückfällt.
* Im **Channels-Tab** werden Kanalstatus und Steuerelemente für WhatsApp/Telegram angezeigt (Login-QR, Abmeldung, Probe, letzte Trennung/Fehler).

<div id="how-the-probe-works">
  ## Wie der Health-Check funktioniert
</div>

* Die App führt etwa alle 60 Sekunden sowie bei Bedarf `openclaw health --json` über `ShellExecutor` aus. Die Prüfroutine lädt Anmeldedaten und meldet den Status, ohne Nachrichten zu senden.
* Die App speichert den letzten gültigen Snapshot und den letzten Fehler jeweils separat zwischen, um Flackern zu vermeiden, und zeigt jeweils den Zeitstempel an.

<div id="when-in-doubt">
  ## Im Zweifel
</div>

* Du kannst weiterhin den CLI-Workflow im Abschnitt [Gateway health](/de/gateway/health) nutzen (`openclaw status`, `openclaw status --deep`, `openclaw health --json`) und die Datei `/tmp/openclaw/openclaw-*.log` mit `tail` auf `web-heartbeat` bzw. `web-reconnect` überwachen.