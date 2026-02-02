---
title: Rete
summary: "Hub di rete: interfacce del Gateway, abbinamento, discovery e sicurezza"
read_when:
  - Ti serve una panoramica dell'architettura e della sicurezza di rete
  - Stai eseguendo il debug dell'accesso locale rispetto all'accesso tramite tailnet o dell'abbinamento
  - Vuoi l'elenco canonico della documentazione di rete
---

<div id="network-hub">
  # Hub di rete
</div>

Questo hub raccoglie la documentazione principale su come OpenClaw si connette, effettua il pairing e protegge
i dispositivi su localhost, LAN e tailnet.

<div id="core-model">
  ## Modello principale
</div>

* [Architettura del Gateway](/it/concepts/architecture)
* [Protocollo del Gateway](/it/gateway/protocol)
* [Runbook del Gateway](/it/gateway)
* [Interfacce web + modalità di binding](/it/web)

<div id="pairing-identity">
  ## Abbinamento + identità
</div>

* [Panoramica dell’abbinamento (DM + nodi)](/it/start/pairing)
* [Abbinamento dei nodi di proprietà del Gateway](/it/gateway/pairing)
* [CLI dei dispositivi (abbinamento + rotazione dei token)](/it/cli/devices)
* [CLI di abbinamento (approvazioni DM)](/it/cli/pairing)

Trust locale:

* Le connessioni locali (loopback o l’indirizzo tailnet dell’host del Gateway) possono essere
  approvate automaticamente per l’abbinamento per mantenere fluida la UX sullo stesso host.
* I client tailnet/LAN non locali richiedono comunque un’esplicita approvazione di abbinamento.

<div id="discovery-transports">
  ## Scoperta + trasporti
</div>

* [Scoperta e trasporti](/it/gateway/discovery)
* [Bonjour / mDNS](/it/gateway/bonjour)
* [Accesso remoto (SSH)](/it/gateway/remote)
* [Tailscale](/it/gateway/tailscale)

<div id="nodes-transports">
  ## Nodi + trasporti
</div>

* [Panoramica sui nodi](/it/nodes)
* [Protocollo Bridge (nodi legacy)](/it/gateway/bridge-protocol)
* [Runbook del nodo: iOS](/it/platforms/ios)
* [Runbook del nodo: Android](/it/platforms/android)

<div id="security">
  ## Sicurezza
</div>

* [Panoramica sulla sicurezza](/it/gateway/security)
* [Riferimento alla configurazione del Gateway](/it/gateway/configuration)
* [Risoluzione dei problemi](/it/gateway/troubleshooting)
* [Doctor](/it/gateway/doctor)