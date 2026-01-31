---
title: Erkennung
summary: "Knotenerkennung und Transportmechanismen (Bonjour, Tailscale, SSH) zum Auffinden des Gateways"
read_when:
  - Bonjour-Erkennung/-Ankündigung implementieren oder ändern
  - Remote-Verbindungsmodi (direkt vs SSH) anpassen
  - Knotenerkennung + Kopplung für entfernte Knoten entwerfen
---

<div id="discovery-transports">
  # Discovery &amp; transports
</div>

OpenClaw hat zwei verschiedene Probleme, die auf den ersten Blick ähnlich erscheinen:

1. **Remote-Steuerung durch den Operator**: die macOS-Menüleisten-App, die ein anderswo laufendes Gateway steuert.
2. **Node-Kopplung**: iOS/Android (und zukünftige Knoten), die ein Gateway finden und sich sicher koppeln.

Das Designziel ist, die gesamte Netzwerk-Discovery/-Ankündigung im **Node Gateway** (`openclaw gateway`) zu belassen und Clients (Mac-App, iOS) nur als Konsumenten zu behandeln.

<div id="terms">
  ## Begriffe
</div>

* **Gateway**: ein einzelner, langlaufender Gateway-Prozess, der Zustand (Sitzungen, Kopplung, Knoten-Registry) hält und Kanäle ausführt. Die meisten Setups verwenden einen pro Host; isolierte Multi-Gateway-Setups sind möglich.
* **Gateway WS (Control Plane)**: standardmäßig der WebSocket-Endpunkt auf `127.0.0.1:18789`; kann über `gateway.bind` an LAN/Tailnet gebunden werden.
* **Direkter WS-Transport**: ein auf LAN/Tailnet exponierter Gateway-WS-Endpunkt (kein SSH).
* **SSH-Transport (Fallback)**: Remote-Steuerung durch Weiterleitung von `127.0.0.1:18789` über SSH.
* **Legacy-TCP-Bridge (veraltet/entfernt)**: älterer Transport für Knoten (siehe [Bridge-Protokoll](/de/gateway/bridge-protocol)); wird für Discovery/Erkennung nicht mehr angekündigt.

Protokolldetails:

* [Gateway-Protokoll](/de/gateway/protocol)
* [Bridge-Protokoll (Legacy)](/de/gateway/bridge-protocol)

<div id="why-we-keep-both-direct-and-ssh">
  ## Warum wir sowohl „Direct“ als auch SSH beibehalten
</div>

* **Direct WS** bietet die beste UX im selben Netzwerk bzw. innerhalb eines Tailnets:
  * automatische Erkennung im LAN über Bonjour
  * Kopplungs-Tokens und ACLs werden vom Gateway verwaltet
  * kein Shell-Zugriff erforderlich; die Protokolloberfläche kann eng begrenzt und gut prüfbar bleiben
* **SSH** bleibt die universelle Fallback-Option:
  * funktioniert überall dort, wo du SSH-Zugriff hast (auch über voneinander getrennte Netzwerke hinweg)
  * übersteht Multicast-/mDNS-Probleme
  * erfordert keine neuen eingehenden Ports außer SSH

<div id="discovery-inputs-how-clients-learn-where-the-gateway-is">
  ## Discovery-Informationen (wie Clients den Standort des Gateways ermitteln)
</div>

<div id="1-bonjour-mdns-lan-only">
  ### 1) Bonjour / mDNS (nur LAN)
</div>

Bonjour arbeitet nach dem Best-Effort-Prinzip und funktioniert nicht über Netzwerkgrenzen hinweg. Es wird nur als Komfortfunktion im „gleichen LAN“ verwendet.

Verbindungsrichtung:

* Das **Gateway** kündigt seinen WS-Endpunkt über Bonjour an.
* Clients durchsuchen das Netzwerk, zeigen eine „Gateway auswählen“-Liste an und speichern anschließend den gewählten Endpunkt.

Fehlerbehebung und Beacon-Details: [Bonjour](/de/gateway/bonjour).

<div id="service-beacon-details">
  #### Details zum Service-Beacon
</div>

* Service-Typen:
  * `_openclaw-gw._tcp` (Gateway-Transport-Beacon)
* TXT-Keys (nicht geheim):
  * `role=gateway`
  * `lanHost=<hostname>.local`
  * `sshPort=22` (oder der angegebene Port)
  * `gatewayPort=18789` (Gateway WS + HTTP)
  * `gatewayTls=1` (nur, wenn TLS aktiviert ist)
  * `gatewayTlsSha256=<sha256>` (nur, wenn TLS aktiviert ist und ein Fingerabdruck verfügbar ist)
  * `canvasPort=18793` (Standard-Canvas-Host-Port; stellt `/__openclaw__/canvas/` bereit)
  * `cliPath=<path>` (optional; absoluter Pfad zu einem ausführbaren `openclaw`-Entrypoint oder Binary)
  * `tailnetDns=<magicdns>` (optionaler Hinweis; wird automatisch erkannt, wenn Tailscale verfügbar ist)

Deaktivieren/überschreiben:

* `OPENCLAW_DISABLE_BONJOUR=1` deaktiviert die Ankündigung.
* `gateway.bind` in `~/.openclaw/openclaw.json` steuert den Gateway-Bind-Modus.
* `OPENCLAW_SSH_PORT` überschreibt den im TXT-Eintrag angekündigten SSH-Port (Standard ist 22).
* `OPENCLAW_TAILNET_DNS` veröffentlicht einen `tailnetDns`-Hinweis (MagicDNS).
* `OPENCLAW_CLI_PATH` überschreibt den angekündigten CLI-Pfad.

<div id="2-tailnet-cross-network">
  ### 2) Tailnet (netzwerkübergreifend)
</div>

Für Setups im Stil London/Wien hilft Bonjour nicht weiter. Das empfohlene „direkte“ Ziel ist:

* Tailscale-MagicDNS-Name (bevorzugt) oder eine stabile Tailnet-IP-Adresse.

Wenn das Gateway erkennen kann, dass es unter Tailscale läuft, gibt es `tailnetDns` als optionalen Hinweis für Clients bekannt (einschließlich Wide-Area-Beacons).

<div id="3-manual-ssh-target">
  ### 3) Manuelles / SSH-Ziel
</div>

Wenn es keine direkte Route gibt (oder die Direktverbindung deaktiviert ist), können sich Clients immer per SSH verbinden, indem sie den Gateway-Port auf der Loopback-Schnittstelle weiterleiten.

Siehe [Remote-Zugriff](/de/gateway/remote).

<div id="transport-selection-client-policy">
  ## Transportauswahl (Client-Richtlinie)
</div>

Empfohlenes Verhalten des Clients:

1. Wenn ein gekoppelte*r* direkter*?* Endpunkt konfiguriert und erreichbar ist, verwende diesen.
2. Andernfalls, wenn Bonjour ein Gateway im LAN findet, biete eine „Mit einem Tipp verwenden“-Option „Dieses Gateway verwenden“ an und speichere es als direkten Endpunkt.
3. Andernfalls, wenn eine tailnet-DNS/IP konfiguriert ist, versuche eine Direktverbindung.
4. Andernfalls auf SSH zurückgreifen.

<div id="pairing-auth-direct-transport">
  ## Kopplung + Authentifizierung (direkter Transport)
</div>

Das Gateway ist die maßgebliche Instanz für die Zulassung von Knoten/Clients.

* Kopplungsanfragen werden im Gateway erstellt, genehmigt oder abgelehnt (siehe [Gateway-Kopplung](/de/gateway/pairing)).
* Das Gateway erzwingt:
  * Authentifizierung (Token / Schlüsselpaar)
  * Scopes/ACLs (das Gateway ist kein ungefilterter Proxy für beliebige Methoden)
  * Rate Limits (Anfragebegrenzungen)

<div id="responsibilities-by-component">
  ## Verantwortlichkeiten pro Komponente
</div>

* **Gateway**: sendet Discovery-Beacons, trifft Kopplungsentscheidungen und stellt den WS-Endpunkt bereit.
* **macOS-App**: hilft dir, ein Gateway auszuwählen, zeigt Kopplungsaufforderungen an und verwendet SSH nur als Fallback.
* **iOS/Android-Knoten**: durchsuchen Bonjour als Komfortfunktion und verbinden sich mit dem WS-Endpunkt des gekoppelten Gateways.