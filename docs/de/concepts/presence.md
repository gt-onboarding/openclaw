---
title: Präsenz
summary: "Wie OpenClaw-Präsenz-Einträge erzeugt, zusammengeführt und angezeigt werden"
read_when:
  - Beim Debuggen des Tabs „Instances“
  - Beim Untersuchen doppelter oder veralteter Instanzzeilen
  - Beim Ändern der Gateway-WS-Verbindung oder der System-Event-Beacons
---

<div id="presence">
  # Präsenz
</div>

Die OpenClaw‑„Präsenz“ ist eine leichtgewichtige Best‑Effort‑Ansicht von:

- dem **Gateway** selbst und
- **Clients, die mit dem Gateway verbunden sind** (Mac‑App, WebChat, CLI usw.)

Präsenz wird hauptsächlich verwendet, um den **Instances**‑Tab der macOS‑App zu rendern und
Betreibern eine schnelle Übersicht zu geben.

<div id="presence-fields-what-shows-up">
  ## Presence-Felder (was angezeigt wird)
</div>

Presence-Einträge sind strukturierte Objekte mit Feldern wie:

- `instanceId` (optional, aber nachdrücklich empfohlen): stabile Client-Identität (in der Regel `connect.client.instanceId`)
- `host`: menschenlesbarer Hostname
- `ip`: Best-Effort-IP-Adresse
- `version`: Client-Versionsstring
- `deviceFamily` / `modelIdentifier`: Hardware-Hinweise
- `mode`: `ui`, `webchat`, `cli`, `backend`, `probe`, `test`, `node`, ...
- `lastInputSeconds`: „Sekunden seit der letzten Benutzereingabe“ (falls bekannt)
- `reason`: `self`, `connect`, `node-connected`, `periodic`, ...
- `ts`: Zeitstempel der letzten Aktualisierung (ms seit Unix-Epoche)

<div id="producers-where-presence-comes-from">
  ## Quellen (woher die Präsenz stammt)
</div>

Präsenz-Einträge werden aus mehreren Quellen erzeugt und **zusammengeführt**.

<div id="1-gateway-self-entry">
  ### 1) Gateway-Selbsteintrag
</div>

Das Gateway legt beim Start immer einen „self“-Eintrag an, damit UIs den Gateway-Host anzeigen, noch bevor sich Clients verbinden.

<div id="2-websocket-connect">
  ### 2) WebSocket-Connect
</div>

Jeder WS-Client beginnt mit einer `connect`-Anfrage. Bei einem erfolgreichen Handshake legt das Gateway einen Presence-Eintrag für diese Verbindung an oder aktualisiert einen bestehenden.

<div id="why-oneoff-cli-commands-dont-show-up">
  #### Warum einmalige CLI‑Befehle nicht angezeigt werden
</div>

Die CLI stellt häufig nur kurz eine Verbindung für einmalige Befehle her. Um die
Instanzenliste nicht zu überfluten, wird `client.mode === "cli"` **nicht** in einen Presence‑Eintrag umgewandelt.

<div id="3-system-event-beacons">
  ### 3) `system-event`-Beacons
</div>

Clients können über die `system-event`-Methode reichhaltigere periodische Beacons senden. Die macOS-App nutzt diese Methode, um Hostname, IP-Adresse und `lastInputSeconds` zu melden.

<div id="4-node-connects-role-node">
  ### 4) Knoten verbindet sich (role: node)
</div>

Wenn sich ein Knoten über den Gateway-WebSocket mit `role: node` verbindet, legt das Gateway
einen Presence-Eintrag für diesen Knoten an oder aktualisiert ihn (gleicher Ablauf wie bei anderen WS-Clients).

<div id="merge-dedupe-rules-why-instanceid-matters">
  ## Merge- und Deduplizierungsregeln (warum `instanceId` wichtig ist)
</div>

Presence-Einträge werden in einer einzelnen In‑Memory-Map gespeichert:

- Einträge werden über einen **Presence-Schlüssel** adressiert.
- Der beste Schlüssel ist eine stabile `instanceId` (von `connect.client.instanceId`), die Neustarts übersteht.
- Schlüssel sind case-insensitiv (Groß-/Kleinschreibung wird ignoriert).

Wenn sich ein Client ohne stabile `instanceId` erneut verbindet, kann er als
**doppelte** Zeile erscheinen.

<div id="ttl-and-bounded-size">
  ## TTL und begrenzte Größe
</div>

Presence ist bewusst kurzlebig:

- **TTL:** Einträge, die älter als 5 Minuten sind, werden verworfen
- **Maximale Anzahl Einträge:** 200 (die ältesten werden zuerst verworfen)

Dadurch bleibt die Liste aktuell und ein unbegrenztes Speicherwachstum wird vermieden.

<div id="remotetunnel-caveat-loopback-ips">
  ## Hinweis zu Remote/Tunnel (Loopback-IPs)
</div>

Wenn sich ein Client über einen SSH-Tunnel bzw. eine lokale Portweiterleitung verbindet, kann das Gateway
die Remote-Adresse als `127.0.0.1` sehen. Um zu vermeiden, dass eine korrekte, vom Client gemeldete IP
überschrieben wird, werden Loopback-Remote-Adressen ignoriert.

<div id="consumers">
  ## Consumer
</div>

<div id="macos-instances-tab">
  ### Tab „Instanzen“ unter macOS
</div>

Die macOS‑App rendert die Ausgabe von `system-presence` und setzt anhand des Alters der letzten Aktualisierung einen kleinen Statusindikator (Active/Idle/Stale).

<div id="debugging-tips">
  ## Debugging-Tipps
</div>

- Um die rohe Liste anzuzeigen, rufe `system-presence` gegen das Gateway auf.
- Wenn du Duplikate siehst:
  - bestätige, dass Clients im Handshake eine stabile `client.instanceId` senden
  - bestätige, dass periodische Beacons dieselbe `instanceId` verwenden
  - prüfe, ob dem verbindungsabgeleiteten Eintrag die `instanceId` fehlt (Duplikate sind dann zu erwarten)