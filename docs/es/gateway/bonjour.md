---
title: Bonjour
summary: "Descubrimiento y depuración de Bonjour/mDNS (balizas del Gateway, clientes y fallos comunes)"
read_when:
  - Depurar problemas de descubrimiento de Bonjour en macOS/iOS
  - Cambiar tipos de servicio mDNS, registros TXT o la experiencia de descubrimiento (UX)
---

<div id="bonjour-mdns-discovery">
  # Bonjour / descubrimiento mDNS
</div>

OpenClaw usa Bonjour (mDNS / DNS‑SD) como una **facilidad limitada a la LAN** para descubrir
un Gateway activo (endpoint WebSocket). Es de tipo **best‑effort** y **no** sustituye la conectividad basada en SSH o Tailnet.

<div id="widearea-bonjour-unicast-dnssd-over-tailscale">
  ## Bonjour de área amplia (DNS‑SD unicast) sobre Tailscale
</div>

Si el nodo y el Gateway están en redes diferentes, el mDNS multicast no cruzará ese
límite. Puedes mantener la misma experiencia de descubrimiento cambiando a **DNS‑SD unicast**
(&quot;Bonjour de área amplia&quot;) sobre Tailscale.

Pasos generales:

1. Ejecuta un servidor DNS en el host del Gateway (accesible a través de Tailnet).
2. Publica registros DNS‑SD para `_openclaw-gw._tcp` bajo una zona dedicada
   (por ejemplo: `openclaw.internal.`).
3. Configura el **split DNS** de Tailscale para que el dominio que elijas se resuelva mediante ese
   servidor DNS para los clientes (incluido iOS).

OpenClaw admite cualquier dominio de descubrimiento; `openclaw.internal.` es solo un ejemplo.
Los nodos iOS/Android exploran tanto `local.` como el dominio de área amplia que hayas configurado.

<div id="gateway-config-recommended">
  ### Configuración del Gateway (recomendada)
</div>

```json5
{
  gateway: { bind: "tailnet" }, // tailnet-only (recommended)
  discovery: { wideArea: { enabled: true } } // habilita publicación DNS-SD de área amplia
}
```

<div id="onetime-dns-server-setup-gateway-host">
  ### Configuración inicial del servidor DNS (host del Gateway)
</div>

```bash
openclaw dns setup --apply
```

Esto instala CoreDNS y lo configura para:

* escuchar en el puerto 53 únicamente en las interfaces Tailscale del Gateway
* servir el dominio que elijas (ejemplo: `openclaw.internal.`) desde `~/.openclaw/dns/<domain>.db`

Verifica desde una máquina conectada a la Tailnet:

```bash
dns-sd -B _openclaw-gw._tcp openclaw.internal.
dig @<TAILNET_IPV4> -p 53 _openclaw-gw._tcp.openclaw.internal PTR +short
```

<div id="tailscale-dns-settings">
  ### Configuración de DNS en Tailscale
</div>

En la consola de administración de Tailscale:

* Añade un servidor de nombres que apunte a la IP de tailnet del Gateway (UDP/TCP 53).
* Añade DNS dividido (split DNS) para que tu dominio de descubrimiento use ese servidor de nombres.

Una vez que los clientes acepten el DNS de tailnet, los nodos iOS pueden explorar
`_openclaw-gw._tcp` en tu dominio de descubrimiento sin usar multidifusión.

<div id="gateway-listener-security-recommended">
  ### Seguridad del listener del Gateway (recomendado)
</div>

El puerto WS del Gateway (por defecto `18789`) se vincula a loopback de forma predeterminada. Para acceso desde la LAN o tailnet, vincúlalo explícitamente y mantén la autenticación habilitada.

Para configuraciones solo tailnet:

* Establece `gateway.bind: "tailnet"` en `~/.openclaw/openclaw.json`.
* Reinicia el Gateway (o reinicia la aplicación de la barra de menús de macOS).

<div id="what-advertises">
  ## Qué se anuncia
</div>

Solo el Gateway anuncia `_openclaw-gw._tcp`.

<div id="service-types">
  ## Tipos de servicio
</div>

* `_openclaw-gw._tcp` — anuncio de transporte del Gateway (utilizado por nodos macOS/iOS/Android).

<div id="txt-keys-nonsecret-hints">
  ## Claves TXT (pistas no secretas)
</div>

El Gateway anuncia pequeñas pistas no secretas para hacer que los flujos de la UI sean más cómodos:

* `role=gateway`
* `displayName=<friendly name>`
* `lanHost=<hostname>.local`
* `gatewayPort=<port>` (Gateway WS + HTTP)
* `gatewayTls=1` (solo cuando TLS está habilitado)
* `gatewayTlsSha256=<sha256>` (solo cuando TLS está habilitado y la huella digital está disponible)
* `canvasPort=<port>` (solo cuando el host de canvas está habilitado; valor predeterminado `18793`)
* `sshPort=<port>` (el valor predeterminado es 22 cuando no se ha anulado)
* `transport=gateway`
* `cliPath=<path>` (opcional; ruta absoluta a un punto de entrada ejecutable `openclaw`)
* `tailnetDns=<magicdns>` (pista opcional cuando Tailnet está disponible)

<div id="debugging-on-macos">
  ## Depuración en macOS
</div>

Herramientas integradas útiles:

* Examinar instancias:
  ```bash
  dns-sd -B _openclaw-gw._tcp local.
  ```
* Resolver una instancia (sustituye `<instance>`):
  ```bash
  dns-sd -L "<instance>" _openclaw-gw._tcp local.
  ```

Si el examen de instancias funciona pero la resolución falla, normalmente te estás topando con una política de la LAN o un problema del resolvedor mDNS.

<div id="debugging-in-gateway-logs">
  ## Depuración en los registros del Gateway
</div>

El Gateway escribe un archivo de registro rotativo (que se muestra al iniciarse como
`gateway log file: ...`). Busca las líneas con `bonjour:`, en especial:

* `bonjour: advertise failed ...`
* `bonjour: ... name conflict resolved` / `hostname conflict resolved`
* `bonjour: watchdog detected non-announced service ...`

<div id="debugging-on-ios-node">
  ## Depuración en el nodo iOS
</div>

El nodo iOS usa `NWBrowser` para descubrir `_openclaw-gw._tcp`.

Para capturar registros:

* Ajustes → Gateway → Avanzado → **Registros de depuración de descubrimiento**
* Ajustes → Gateway → Avanzado → **Registros de descubrimiento** → reproducir el problema → **Copiar**

El registro incluye transiciones de estado del *browser* y cambios en el conjunto de resultados.

<div id="common-failure-modes">
  ## Modos de fallo habituales
</div>

* **Bonjour no atraviesa redes**: usa Tailnet o SSH.
* **Multidifusión bloqueada**: algunas redes Wi‑Fi deshabilitan mDNS.
* **Suspensión / cambios de interfaz**: macOS puede descartar temporalmente resultados de mDNS; vuelve a intentarlo.
* **La exploración funciona, pero la resolución falla**: mantén simples los nombres de las máquinas (evita emojis o
  signos de puntuación) y luego reinicia el Gateway. El nombre de la instancia del servicio se deriva del nombre de host, por lo que los nombres demasiado complejos pueden confundir a algunos resolvedores de nombres.

<div id="escaped-instance-names-032">
  ## Nombres de instancia con escape (`\032`)
</div>

Bonjour/DNS‑SD a menudo escapa bytes en los nombres de instancia de servicio como secuencias decimales `\DDD`
(p. ej., los espacios se convierten en `\032`).

* Esto es normal a nivel de protocolo.
* Las UIs deberían decodificarlos al mostrarlos (iOS usa `BonjourEscapes.decode`).

<div id="disabling-configuration">
  ## Desactivación / configuración
</div>

* `OPENCLAW_DISABLE_BONJOUR=1` desactiva los anuncios (heredado: `OPENCLAW_DISABLE_BONJOUR`).
* `gateway.bind` en `~/.openclaw/openclaw.json` controla el modo de bind del Gateway.
* `OPENCLAW_SSH_PORT` sobrescribe el puerto SSH anunciado en TXT (heredado: `OPENCLAW_SSH_PORT`).
* `OPENCLAW_TAILNET_DNS` publica una sugerencia de MagicDNS en TXT (heredado: `OPENCLAW_TAILNET_DNS`).
* `OPENCLAW_CLI_PATH` sobrescribe la ruta de la CLI anunciada (heredado: `OPENCLAW_CLI_PATH`).

<div id="related-docs">
  ## Documentación relacionada
</div>

* Política de descubrimiento y selección de transporte: [Discovery](/es/gateway/discovery)
* Emparejamiento y aprobaciones de nodos: [Gateway pairing](/es/gateway/pairing)