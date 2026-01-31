---
title: Dns
summary: "Référence CLI pour `openclaw dns` (outils de découverte sur réseau étendu)"
read_when:
  - Vous souhaitez une découverte sur réseau étendu (DNS-SD) via Tailscale + CoreDNS
  - Vous configurez un split DNS pour un domaine de découverte personnalisé (exemple : openclaw.internal)
---

<div id="openclaw-dns">
  # `openclaw dns`
</div>

Utilitaires DNS pour la découverte à large échelle (Tailscale + CoreDNS). Actuellement principalement orienté vers macOS + CoreDNS installé via Homebrew.

Liens associés :

* Découverte du Gateway OpenClaw : [Discovery](/fr/gateway/discovery)
* Configuration de la découverte à large échelle : [Configuration](/fr/gateway/configuration)

<div id="setup">
  ## Configuration
</div>

```bash
openclaw dns setup
openclaw dns setup --apply
```
