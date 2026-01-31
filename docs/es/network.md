---
title: Red
summary: "Nodo central de red: superficies del Gateway, emparejamiento, descubrimiento y seguridad"
read_when:
  - Necesitas una visión general de la arquitectura de red y la seguridad
  - Estás solucionando problemas de acceso local frente a acceso mediante tailnet o de emparejamiento
  - Quieres la lista canónica de la documentación de red
---

<div id="network-hub">
  # Concentrador de red
</div>

Este concentrador enlaza la documentación principal sobre cómo OpenClaw conecta, empareja y protege
dispositivos en localhost, la LAN y la tailnet.

<div id="core-model">
  ## Modelo principal
</div>

* [Arquitectura del Gateway](/es/concepts/architecture)
* [Protocolo del Gateway](/es/gateway/protocol)
* [Runbook del Gateway](/es/gateway)
* [Interfaces web + modos de vinculación](/es/web)

<div id="pairing-identity">
  ## Emparejamiento + identidad
</div>

* [Descripción general del emparejamiento (DM + nodos)](/es/start/pairing)
* [Emparejamiento de nodos propiedad del Gateway](/es/gateway/pairing)
* [CLI de dispositivos (emparejamiento + rotación de tokens)](/es/cli/devices)
* [CLI de emparejamiento (aprobaciones de DM)](/es/cli/pairing)

Confianza local:

* Las conexiones locales (loopback o la propia dirección tailnet del host del Gateway) pueden aprobarse automáticamente para el emparejamiento a fin de mantener una experiencia de usuario fluida en el mismo host.
* Los clientes tailnet/LAN no locales siguen requiriendo una aprobación explícita de emparejamiento.

<div id="discovery-transports">
  ## Descubrimiento + transportes
</div>

* [Descubrimiento &amp; transportes](/es/gateway/discovery)
* [Bonjour / mDNS](/es/gateway/bonjour)
* [Acceso remoto (SSH)](/es/gateway/remote)
* [Tailscale](/es/gateway/tailscale)

<div id="nodes-transports">
  ## Nodos + transportes
</div>

* [Descripción general de los nodos](/es/nodes)
* [Protocolo Bridge (nodos heredados)](/es/gateway/bridge-protocol)
* [Runbook del nodo: iOS](/es/platforms/ios)
* [Runbook del nodo: Android](/es/platforms/android)

<div id="security">
  ## Seguridad
</div>

* [Descripción general de la seguridad](/es/gateway/security)
* [Referencia de la configuración de Gateway](/es/gateway/configuration)
* [Solución de problemas](/es/gateway/troubleshooting)
* [Doctor](/es/gateway/doctor)