---
title: DNS
summary: "CLI-Referenz für `openclaw dns` (Hilfsprogramme für Wide-Area-Discovery)"
read_when:
  - Du möchtest Wide-Area-Discovery (DNS-SD) über Tailscale + CoreDNS einrichten
  - Du richtest Split-DNS für eine eigene Discovery-Domain ein (z. B. openclaw.internal)
---

<div id="openclaw-dns">
  # `openclaw dns`
</div>

DNS-Hilfsfunktionen für Wide-Area-Discovery (Tailscale + CoreDNS). Derzeit ausgerichtet auf macOS + Homebrew CoreDNS.

Siehe auch:

* Gateway-Erkennung: [Discovery](/de/gateway/discovery)
* Wide-Area-Discovery-Konfiguration: [Konfiguration](/de/gateway/configuration)

<div id="setup">
  ## Einrichtung
</div>

```bash
openclaw dns setup
openclaw dns setup --apply
```
