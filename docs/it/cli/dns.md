---
title: Dns
summary: "Riferimento CLI per `openclaw dns` (strumenti di discovery su ampia scala)"
read_when:
  - Vuoi abilitare il discovery su ampia scala (DNS-SD) tramite Tailscale + CoreDNS
  - Stai configurando lo split DNS per un dominio di discovery personalizzato (esempio: openclaw.internal)
---

<div id="openclaw-dns">
  # `openclaw dns`
</div>

Utility DNS per la discovery su vasta area (Tailscale + CoreDNS). Attualmente incentrato su macOS + CoreDNS tramite Homebrew.

Correlato:

* Discovery del Gateway: [Discovery](/it/gateway/discovery)
* Configurazione della discovery su vasta area: [Configuration](/it/gateway/configuration)

<div id="setup">
  ## Configurazione
</div>

```bash
openclaw dns setup
openclaw dns setup --apply
```
