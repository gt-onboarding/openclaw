---
title: Child-Prozess
summary: "Gateway-Lebenszyklus unter macOS (launchd)"
read_when:
  - Integration der Mac-App in den Gateway-Lebenszyklus
---

<div id="gateway-lifecycle-on-macos">
  # Gateway-Lebenszyklus unter macOS
</div>

Die macOS-App **verwaltet das Gateway standardmäßig über launchd** und startet
das Gateway nicht als Child-Prozess. Sie versucht zunächst, sich mit einem bereits
laufenden Gateway auf dem konfigurierten Port zu verbinden; ist keines erreichbar,
aktiviert sie den launchd-Dienst über die externe `openclaw` CLI (keine eingebettete
Laufzeitumgebung). Das ermöglicht einen zuverlässigen automatischen Start bei der Anmeldung und
einen Neustart nach Abstürzen.

Der Child-Prozess-Modus (Gateway wird direkt von der App gestartet) ist **derzeit nicht im Einsatz**.
Wenn du eine engere Kopplung an die UI benötigst, starte das Gateway manuell im Terminal.

<div id="default-behavior-launchd">
  ## Standardverhalten (launchd)
</div>

* Die App installiert einen benutzerspezifischen LaunchAgent mit der Bezeichnung `bot.molt.gateway`
  (oder `bot.molt.<profile>` bei Verwendung von `--profile`/`OPENCLAW_PROFILE`; das ältere Schema `com.openclaw.*` wird weiterhin unterstützt).
* Wenn der lokale Modus aktiviert ist, stellt die App sicher, dass der LaunchAgent geladen ist und
  startet bei Bedarf das Gateway.
* Logs werden in den launchd-Logpfad des Gateways geschrieben (sichtbar in den Debug-Einstellungen).

Häufig verwendete Befehle:

```bash
launchctl kickstart -k gui/$UID/bot.molt.gateway
launchctl bootout gui/$UID/bot.molt.gateway
```

Ersetze das Label durch `bot.molt.&lt;profile&gt;`, wenn du ein benanntes Profil verwendest.


<div id="unsigned-dev-builds">
  ## Unsigned-Dev-Builds
</div>

`scripts/restart-mac.sh --no-sign` ist für schnelle lokale Builds, wenn du
keine Signierschlüssel hast. Um zu verhindern, dass launchd auf ein unsigniertes Relay-Binary verweist, wird:

* die Datei `~/.openclaw/disable-launchagent` geschrieben.

Signierte Ausführungen von `scripts/restart-mac.sh` heben dieses Override auf, falls der Marker
vorhanden ist. Um dies manuell zurückzusetzen:

```bash
rm ~/.openclaw/disable-launchagent
```


<div id="attach-only-mode">
  ## Nur-Anbindemodus
</div>

Um zu erzwingen, dass die macOS-App **niemals launchd installiert oder verwaltet**, startest du sie mit
`--attach-only` (oder `--no-launchd`). Dadurch wird `~/.openclaw/disable-launchagent`
gesetzt, sodass sich die App nur an ein bereits laufendes Gateway anbindet. Dasselbe
Verhalten kannst du in den Debug-Einstellungen umschalten.

<div id="remote-mode">
  ## Remote-Modus
</div>

Der Remote-Modus startet nie ein lokales Gateway. Die App verwendet einen SSH-Tunnel zum Remote-Host und verbindet sich über diesen Tunnel.

<div id="why-we-prefer-launchd">
  ## Warum wir launchd bevorzugen
</div>

- Automatischer Start nach der Anmeldung.
- Eingebaute Neustart-/KeepAlive-Semantik.
- Vorhersehbare Logs und Überwachung.

Falls jemals wieder ein echter Child‑Prozess‑Modus benötigt wird, sollte er als
separater, expliziter reiner Dev‑Modus dokumentiert werden.