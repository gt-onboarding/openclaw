---
title: Réseau
summary: "Hub réseau : interfaces du Gateway, appairage, découverte et sécurité"
read_when:
  - Vous avez besoin d'une vue d'ensemble de l'architecture réseau et de la sécurité
  - Vous êtes en train de diagnostiquer l'accès local vs tailnet ou l'appairage
  - Vous voulez la liste canonique des documentations réseau
---

<div id="network-hub">
  # Hub réseau
</div>

Ce hub regroupe la documentation principale sur la façon dont OpenClaw connecte, apparie et sécurise des appareils via localhost, le LAN et le tailnet.

<div id="core-model">
  ## Modèle principal
</div>

* [Architecture du Gateway](/fr/concepts/architecture)
* [Protocole du Gateway](/fr/gateway/protocol)
* [Runbook du Gateway](/fr/gateway)
* [Surfaces Web + modes de liaison](/fr/web)

<div id="pairing-identity">
  ## Appairage + identité
</div>

* [Vue d’ensemble de l’appairage (DM + nœuds)](/fr/start/pairing)
* [Appairage de nœuds appartenant au Gateway](/fr/gateway/pairing)
* [CLI Devices (appairage + rotation de jetons)](/fr/cli/devices)
* [CLI Pairing (approbations DM)](/fr/cli/pairing)

Confiance locale :

* Les connexions locales (boucle locale ou adresse tailnet de l’hôte du Gateway) peuvent être approuvées automatiquement pour l’appairage afin de préserver une UX fluide sur la même machine.
* Les clients tailnet/LAN non locaux nécessitent toujours une approbation explicite d’appairage.

<div id="discovery-transports">
  ## Découverte + transports
</div>

* [Découverte &amp; transports](/fr/gateway/discovery)
* [Bonjour / mDNS](/fr/gateway/bonjour)
* [Accès distant (SSH)](/fr/gateway/remote)
* [Tailscale](/fr/gateway/tailscale)

<div id="nodes-transports">
  ## Nœuds et transports
</div>

* [Vue d&#39;ensemble des nœuds](/fr/nodes)
* [Protocole de bridge (anciens nœuds)](/fr/gateway/bridge-protocol)
* [Runbook du nœud : iOS](/fr/platforms/ios)
* [Runbook du nœud : Android](/fr/platforms/android)

<div id="security">
  ## Sécurité
</div>

* [Vue d’ensemble de la sécurité](/fr/gateway/security)
* [Référence de configuration de Gateway](/fr/gateway/configuration)
* [Dépannage](/fr/gateway/troubleshooting)
* [Doctor](/fr/gateway/doctor)