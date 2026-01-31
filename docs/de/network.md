---
title: Netzwerk
summary: "Netzwerk-Hub: Gateway-Oberflächen, Kopplung, Erkennung und Sicherheit"
read_when:
  - Du brauchst einen Überblick über Netzwerkarchitektur und Sicherheit
  - Du debuggst Probleme beim lokalen vs. Tailnet-Zugriff oder der Kopplung
  - Du möchtest die kanonische Liste der Netzwerkdokumentation
---

<div id="network-hub">
  # Netzwerk-Hub
</div>

Dieser Hub führt die zentralen Dokumente dazu zusammen, wie OpenClaw Geräte über Localhost, LAN und Tailnet verbindet, koppelt und sichert.

<div id="core-model">
  ## Kernmodell
</div>

* [Gateway-Architektur](/de/concepts/architecture)
* [Gateway-Protokoll](/de/gateway/protocol)
* [Gateway-Runbook](/de/gateway)
* [Web-Oberflächen + Bindemodi](/de/web)

<div id="pairing-identity">
  ## Kopplung + Identität
</div>

* [Kopplungsübersicht (DM + Knoten)](/de/start/pairing)
* [Kopplung von Gateway-eigenen Knoten](/de/gateway/pairing)
* [Devices-CLI (Kopplung + Token-Rotation)](/de/cli/devices)
* [Kopplungs-CLI (DM-Freigaben)](/de/cli/pairing)

Lokales Vertrauen:

* Lokale Verbindungen (Loopback oder die eigene Tailnet-Adresse des Gateway-Hosts) können für die Kopplung automatisch freigegeben werden, um die UX auf demselben Host reibungslos zu gestalten.
* Nicht-lokale Tailnet-/LAN-Clients erfordern weiterhin eine explizite Kopplungsfreigabe.

<div id="discovery-transports">
  ## Discovery + Transporte
</div>

* [Discovery &amp; Transporte](/de/gateway/discovery)
* [Bonjour / mDNS](/de/gateway/bonjour)
* [Remotezugriff (SSH)](/de/gateway/remote)
* [Tailscale](/de/gateway/tailscale)

<div id="nodes-transports">
  ## Nodes + Transports
</div>

* [Nodes-Übersicht](/de/nodes)
* [Bridge-Protokoll (Legacy-Nodes)](/de/gateway/bridge-protocol)
* [Node-Runbook: iOS](/de/platforms/ios)
* [Node-Runbook: Android](/de/platforms/android)

<div id="security">
  ## Sicherheit
</div>

* [Sicherheitsübersicht](/de/gateway/security)
* [Referenz zur Gateway-Konfiguration](/de/gateway/configuration)
* [Fehlerbehebung](/de/gateway/troubleshooting)
* [Doctor](/de/gateway/doctor)