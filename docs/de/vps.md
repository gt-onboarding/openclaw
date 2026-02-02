---
title: VPS
summary: "VPS-Hosting-Hub für OpenClaw (Oracle/Fly/Hetzner/GCP/exe.dev)"
read_when:
  - Du möchtest das Gateway in der Cloud betreiben
  - Du brauchst eine schnelle Übersicht über VPS-/Hosting-Anleitungen
---

<div id="vps-hosting">
  # VPS-Hosting
</div>

Diese Übersichtsseite verlinkt auf die unterstützten VPS-/Hosting-Anleitungen und erklärt auf hoher Ebene, wie Cloud-Deployments funktionieren.

<div id="pick-a-provider">
  ## Wähle einen Anbieter aus
</div>

* **Railway** (One-Click-Deployment + Browser-Setup): [Railway](/de/railway)
* **Northflank** (One-Click-Deployment + Browser-Setup): [Northflank](/de/northflank)
* **Oracle Cloud (Always Free)**: [Oracle](/de/platforms/oracle) — 0 $/Monat (Always Free, ARM; Kapazität/Registrierung kann etwas hakelig sein)
* **Fly.io**: [Fly.io](/de/platforms/fly)
* **Hetzner (Docker)**: [Hetzner](/de/platforms/hetzner)
* **GCP (Compute Engine)**: [GCP](/de/platforms/gcp)
* **exe.dev** (VM + HTTPS-Proxy): [exe.dev](/de/platforms/exe-dev)
* **AWS (EC2/Lightsail/Free Tier)**: funktioniert ebenfalls gut. Video-Anleitung:
  https://x.com/techfrenAJ/status/2014934471095812547

<div id="how-cloud-setups-work">
  ## Funktionsweise von Cloud-Setups
</div>

* Das **Gateway läuft auf dem VPS** und verwaltet Zustand + Arbeitsbereich.
* Du verbindest dich von deinem Laptop/Smartphone über die **Control UI** oder **Tailscale/SSH**.
* Behandle den VPS als maßgebliche Instanz und **sichere** Zustand + Arbeitsbereich.
* Sichere Standardeinstellung: Lass das Gateway auf der Loopback-Schnittstelle laufen und greife über einen SSH-Tunnel oder Tailscale Serve darauf zu.
  Wenn du an `lan`/`tailnet` bindest, setze `gateway.auth.token` oder `gateway.auth.password` voraus.

Remotezugriff: [Gateway remote](/de/gateway/remote)\
Plattformen-Übersicht: [Platforms](/de/platforms)

<div id="using-nodes-with-a-vps">
  ## Verwendung von Knoten mit einem VPS
</div>

Du kannst das Gateway in der Cloud betreiben und **Knoten** mit deinen lokalen Geräten
(Mac/iOS/Android/headless) koppeln. Knoten stellen lokale Bildschirm-/Kamera-/Canvas- und `system.run`-
Funktionen bereit, während das Gateway in der Cloud läuft.

Dokumentation: [Nodes](/de/nodes), [Nodes CLI](/de/cli/nodes)