---
title: Descubrimiento
summary: "Descubrimiento de nodos y transportes (Bonjour, Tailscale, SSH) para encontrar el Gateway"
read_when:
  - Implementar o cambiar el descubrimiento/anuncio con Bonjour
  - Ajustar los modos de conexión remota (modo directo vs SSH)
  - Diseñar el descubrimiento y el emparejamiento de nodos remotos
---

<div id="discovery-transports">
  # Descubrimiento y transportes
</div>

OpenClaw tiene dos problemas distintos que a primera vista parecen similares:

1. **Control remoto del operador**: la app de la barra de menús de macOS que controla un Gateway que se ejecuta en otra máquina.
2. **Emparejamiento de nodos**: iOS/Android (y futuros nodos) que encuentran un Gateway y se emparejan de forma segura.

El objetivo de diseño es mantener todo el descubrimiento/anuncio de red en el **Node Gateway** (`openclaw gateway`) y mantener los clientes (app de macOS, iOS) como consumidores.

<div id="terms">
  ## Términos
</div>

* **Gateway**: un único proceso de Gateway de larga duración que posee el estado (sesiones, emparejamiento, registro de nodos) y gestiona los canales. La mayoría de las configuraciones usan uno por host; también son posibles configuraciones aisladas con múltiples Gateways.
* **Gateway WS (plano de control)**: el endpoint de WebSocket en `127.0.0.1:18789` por defecto; se puede vincular a la LAN/tailnet mediante `gateway.bind`.
* **Transporte WS directo**: un endpoint Gateway WS expuesto hacia la LAN/tailnet (sin SSH).
* **Transporte SSH (fallback)**: control remoto reenviando `127.0.0.1:18789` a través de SSH.
* **Bridge TCP (obsoleto/eliminado)**: transporte de nodo antiguo (ver [Bridge protocol](/es/gateway/bridge-protocol)); ya no se anuncia para descubrimiento automático.

Detalles del protocolo:

* [Gateway protocol](/es/gateway/protocol)
* [Bridge protocol (heredado)](/es/gateway/bridge-protocol)

<div id="why-we-keep-both-direct-and-ssh">
  ## Por qué mantenemos tanto “directo” como SSH
</div>

* **WS directo** ofrece la mejor experiencia de usuario en la misma red y dentro de un tailnet:
  * descubrimiento automático en la LAN mediante Bonjour
  * tokens de emparejamiento + ACL propiedad del Gateway
  * no se requiere acceso a shell; la superficie del protocolo puede mantenerse reducida y auditable
* **SSH** sigue siendo el mecanismo de respaldo universal:
  * funciona en cualquier lugar donde tengas acceso SSH (incluso entre redes no relacionadas)
  * sobrevive a problemas de multidifusión/mDNS
  * no requiere nuevos puertos de entrada aparte de SSH

<div id="discovery-inputs-how-clients-learn-where-the-gateway-is">
  ## Entradas de descubrimiento (cómo los clientes descubren dónde se encuentra el Gateway)
</div>

<div id="1-bonjour-mdns-lan-only">
  ### 1) Bonjour / mDNS (solo LAN)
</div>

Bonjour es de tipo “best-effort” y no atraviesa redes. Solo se utiliza como una comodidad para la “misma LAN”.

Dirección prevista:

* El **Gateway** anuncia su endpoint WS mediante Bonjour.
* Los clientes exploran y muestran una lista para “elegir un gateway”, y luego almacenan el endpoint elegido.

Resolución de problemas y detalles del beacon: [Bonjour](/es/gateway/bonjour).

<div id="service-beacon-details">
  #### Detalles de la baliza de servicio
</div>

* Tipos de servicio:
  * `_openclaw-gw._tcp` (baliza de transporte del Gateway)
* Claves TXT (no confidenciales):
  * `role=gateway`
  * `lanHost=<hostname>.local`
  * `sshPort=22` (o el puerto que se anuncie)
  * `gatewayPort=18789` (WS + HTTP del Gateway)
  * `gatewayTls=1` (solo cuando TLS está habilitado)
  * `gatewayTlsSha256=<sha256>` (solo cuando TLS está habilitado y la huella digital está disponible)
  * `canvasPort=18793` (puerto predeterminado del host de canvas; sirve `/__openclaw__/canvas/`)
  * `cliPath=<path>` (opcional; ruta absoluta a un punto de entrada o binario `openclaw` ejecutable)
  * `tailnetDns=<magicdns>` (sugerencia opcional; se detecta automáticamente cuando Tailscale está disponible)

Desactivar/anular:

* `OPENCLAW_DISABLE_BONJOUR=1` desactiva el anuncio.
* `gateway.bind` en `~/.openclaw/openclaw.json` controla el modo de enlace (bind) del Gateway.
* `OPENCLAW_SSH_PORT` anula el puerto SSH anunciado en TXT (por defecto, 22).
* `OPENCLAW_TAILNET_DNS` publica una sugerencia `tailnetDns` (MagicDNS).
* `OPENCLAW_CLI_PATH` anula la ruta de la CLI anunciada.

<div id="2-tailnet-cross-network">
  ### 2) Tailnet (entre redes)
</div>

Para configuraciones al estilo Londres/Viena, Bonjour no será de ayuda. El destino “directo” recomendado es:

* Nombre MagicDNS de Tailscale (preferido) o una IP estable de tailnet.

Si el Gateway puede detectar que se está ejecutando en Tailscale, publica `tailnetDns` como una indicación opcional para los clientes (incluidas las balizas de área amplia).

<div id="3-manual-ssh-target">
  ### 3) Destino manual / SSH
</div>

Cuando no hay una ruta directa (o la directa está deshabilitada), los clientes siempre pueden conectarse por SSH reenviando el puerto de loopback del Gateway.

Consulta [Acceso remoto](/es/gateway/remote).

<div id="transport-selection-client-policy">
  ## Selección de transporte (política del cliente)
</div>

Comportamiento recomendado del cliente:

1. Si hay un endpoint directo emparejado configurado y accesible, úsalo.
2. En caso contrario, si Bonjour encuentra un Gateway en la LAN, muestra una opción de un solo toque «Usar este Gateway» y guárdalo como endpoint directo.
3. En caso contrario, si hay un DNS/IP de tailnet configurado, intenta el acceso directo.
4. En caso contrario, recurre a SSH.

<div id="pairing-auth-direct-transport">
  ## Emparejamiento + autenticación (transporte directo)
</div>

El Gateway es la fuente de verdad para la admisión de nodos/clientes.

* Las solicitudes de emparejamiento se crean/aprueban/rechazan en el Gateway (consulta [Emparejamiento del Gateway](/es/gateway/pairing)).
* El Gateway impone:
  * autenticación (token / par de claves)
  * ámbitos/ACLs (el Gateway no es un proxy directo a cada método)
  * límites de frecuencia

<div id="responsibilities-by-component">
  ## Responsabilidades por componente
</div>

* **Gateway**: anuncia beacons de descubrimiento, se encarga de las decisiones de emparejamiento y aloja el endpoint de WS.
* **app de macOS**: te ayuda a elegir un Gateway, muestra avisos de emparejamiento y usa SSH solo como último recurso.
* **nodos iOS/Android**: usan Bonjour como comodidad y se conectan al WS del Gateway con el que están emparejados.