---
title: Gateway-Sperre
summary: "Gateway-Singleton-Schutz mittels WebSocket-Listener-Bindung"
read_when:
  - Beim Ausführen oder Debuggen des Gateway-Prozesses
  - Beim Untersuchen der Durchsetzung des Einzelinstanzbetriebs
---

<div id="gateway-lock">
  # Gateway-Sperre
</div>

Zuletzt aktualisiert: 2025-12-11

<div id="why">
  ## Warum
</div>

- Sicherstellen, dass nur eine Gateway-Instanz pro Basisport auf demselben Host läuft; zusätzliche Gateways müssen isolierte Profile und eindeutige Ports verwenden.
- Abstürze/SIGKILL überstehen, ohne veraltete Lock-Dateien zu hinterlassen.
- Schnell mit einer klaren Fehlermeldung fehlschlagen, wenn der Kontrollport bereits belegt ist.

<div id="mechanism">
  ## Mechanismus
</div>

- Das Gateway bindet den WebSocket-Listener (Standard `ws://127.0.0.1:18789`) unmittelbar beim Start über einen exklusiven TCP-Listener.
- Wenn dieses Binden mit `EADDRINUSE` fehlschlägt, schlägt der Start mit `GatewayLockError("another gateway instance is already listening on ws://127.0.0.1:<port>")` fehl.
- Das Betriebssystem gibt den Listener bei jedem Prozessende automatisch frei, einschließlich Abstürzen und SIGKILL – eine separate Lock-Datei oder ein Bereinigungsschritt ist nicht erforderlich.
- Beim Herunterfahren schließt das Gateway den WebSocket-Server und den darunterliegenden HTTP-Server, um den Port zeitnah freizugeben.

<div id="error-surface">
  ## Fehlerbild
</div>

- Wenn ein anderer Prozess den Port belegt, schlägt der Start mit `GatewayLockError("another gateway instance is already listening on ws://127.0.0.1:<port>")` fehl.
- Andere Bind-Fehler äußern sich als `GatewayLockError("failed to bind gateway socket on ws://127.0.0.1:<port>: …")`.

<div id="operational-notes">
  ## Betriebshinweise
</div>

- Wenn der Port von *einem anderen* Prozess belegt ist, ist die Fehlermeldung dieselbe; gib den Port frei oder wähle einen anderen mit `openclaw gateway --port <port>`.
- Die macOS‑App verwendet weiterhin eine eigene leichte PID‑Sperre, bevor sie den Gateway startet; die Laufzeitsperre wird durch das WebSocket-Bind erzwingt.