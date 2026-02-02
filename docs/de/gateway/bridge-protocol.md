---
title: Bridge-Protokoll
summary: "Bridge-Protokoll (Legacy-Knoten): TCP JSONL, Kopplung, Scope-begrenztes RPC"
read_when:
  - Erstellen oder Debuggen von Knoten-Clients (iOS/Android/macOS-Knotenmodus)
  - Untersuchung von Kopplungs- oder Bridge-Authentifizierungsfehlern
  - Überprüfen der vom Gateway bereitgestellten Knotenoberfläche
---

<div id="bridge-protocol-legacy-node-transport">
  # Bridge-Protokoll (Legacy-Knotentransport)
</div>

Das Bridge-Protokoll ist ein **Legacy**-Knotentransport (TCP JSONL). Neue Knoten-Clients
sollten stattdessen das vereinheitlichte Gateway-WebSocket-Protokoll verwenden.

Wenn du einen Operator- oder Knoten-Client entwickelst, verwende das
[Gateway-Protokoll](/de/gateway/protocol).

**Hinweis:** Aktuelle OpenClaw-Builds enthalten den TCP-Bridge-Listener nicht mehr; dieses Dokument wird nur noch aus historischen Gründen bereitgehalten.
Legacy-`bridge.*`-Konfigurationsschlüssel sind nicht länger Teil des Konfigurationsschemas.

<div id="why-we-have-both">
  ## Warum wir beides haben
</div>

* **Sicherheitsgrenze**: Die Bridge stellt nur eine kleine Allowlist statt der
  vollständigen Gateway-API-Oberfläche bereit.
* **Kopplung + Knoten-Identität**: Die Aufnahme von Knoten wird vom Gateway gesteuert und ist an ein Token pro Knoten gebunden.
* **Discovery-UX**: Knoten können Gateways über Bonjour im LAN entdecken oder
  sich direkt über ein Tailnet verbinden.
* **Loopback-WS**: Die vollständige WS-Control-Plane bleibt lokal, es sei denn, sie wird über SSH getunnelt.

<div id="transport">
  ## Transport
</div>

* TCP, ein JSON-Objekt pro Zeile (JSONL).
* Optionales TLS (wenn `bridge.tls.enabled` auf `true` steht).
* Der Legacy-Standardport des Listeners war `18790` (aktuelle Builds starten keine TCP-Bridge).

Wenn TLS aktiviert ist, enthalten die Discovery-TXT-Records `bridgeTls=1` plus
`bridgeTlsSha256`, damit Knoten das Zertifikat pinnen können.

<div id="handshake-pairing">
  ## Handshake + Kopplung
</div>

1. Client sendet `hello` mit Node-Metadaten + Token (falls bereits gekoppelt).
2. Falls nicht gekoppelt, antwortet das Gateway mit `error` (`NOT_PAIRED`/`UNAUTHORIZED`).
3. Client sendet `pair-request`.
4. Gateway wartet auf Genehmigung und sendet dann `pair-ok` und `hello-ok`.

`hello-ok` gibt `serverName` zurück und kann `canvasHostUrl` enthalten.

<div id="frames">
  ## Frames
</div>

Client → Gateway:

* `req` / `res`: bereichsbezogene Gateway-RPC-Aufrufe (`chat`, `sessions`, `config`, `health`, `voicewake`, `skills.bins`)
* `event`: Knoten-Signale (Sprachtranskript, Agent-Anfrage, Chat-Subscribe, Exec-Lebenszyklus)

Gateway → Client:

* `invoke` / `invoke-res`: Knoten-Befehle (`canvas.*`, `camera.*`, `screen.record`,
  `location.get`, `sms.send`)
* `event`: Chat-Updates für abonnierte Sitzungen
* `ping` / `pong`: Keepalive

Die frühere Durchsetzung der Allowlist befand sich in `src/gateway/server-bridge.ts` (entfernt).

<div id="exec-lifecycle-events">
  ## Exec-Lebenszyklusereignisse
</div>

Knoten können `exec.finished`- oder `exec.denied`-Ereignisse senden, um `system.run`-Aktivität sichtbar zu machen.
Diese werden im Gateway auf Systemereignisse abgebildet. (Legacy-Knoten können weiterhin `exec.started` senden.)

Payload-Felder (alle optional, sofern nicht anders angegeben):

* `sessionKey` (erforderlich): agent-Sitzung, die das Systemereignis empfangen soll.
* `runId`: eindeutige Exec-ID zur Gruppierung.
* `command`: rohe oder formatierte Befehlszeichenkette.
* `exitCode`, `timedOut`, `success`, `output`: Abschlussdetails (nur bei `exec.finished`).
* `reason`: Ablehnungsgrund (nur bei `exec.denied`).

<div id="tailnet-usage">
  ## Tailnet-Nutzung
</div>

* Binde die Bridge an eine Tailnet-IP: `bridge.bind: "tailnet"` in
  `~/.openclaw/openclaw.json`.
* Clients stellen über den MagicDNS-Namen oder die Tailnet-IP eine Verbindung her.
* Bonjour funktioniert **nicht** netzwerkübergreifend; verwende bei Bedarf manuell Host/Port oder Wide-Area-DNS‑SD.

<div id="versioning">
  ## Versionierung
</div>

Bridge ist derzeit **implizit v1** (keine Aushandlung von Min/Max‑Versionen). Abwärtskompatibilität wird erwartet; füge vor allen Breaking Changes ein Bridge‑Protokoll‑Versionsfeld hinzu.