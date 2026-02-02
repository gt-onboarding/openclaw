---
title: Bonjour
summary: "Bonjour/mDNS-Erkennung und Debugging (Gateway-Beacons, Clients und häufige Fehlerbilder)"
read_when:
  - Debugging von Bonjour-Erkennungsproblemen unter macOS/iOS
  - Ändern von mDNS-Diensttypen, TXT-Records oder der Discovery-UX
---

<div id="bonjour-mdns-discovery">
  # Bonjour- / mDNS-Erkennung
</div>

OpenClaw verwendet Bonjour (mDNS / DNS‑SD) als **reine LAN‑Komfortfunktion**, um
eine aktive Gateway-Instanz (WebSocket-Endpunkt) zu erkennen. Es handelt sich um ein Best‑Effort-Verfahren und **ersetzt nicht** SSH- oder Tailnet-basierte Konnektivität.

<div id="widearea-bonjour-unicast-dnssd-over-tailscale">
  ## Wide‑Area‑Bonjour (Unicast DNS‑SD) über Tailscale
</div>

Wenn sich der Knoten und das Gateway in unterschiedlichen Netzwerken befinden, kann
Multicast‑mDNS die Grenze nicht überschreiten. Du kannst dieselbe Discovery‑UX beibehalten, indem du auf
**Unicast‑DNS‑SD** („Wide‑Area Bonjour“) über Tailscale umstellst.

Auf hoher Ebene gehst du so vor:

1. Führe einen DNS‑Server auf dem Gateway‑Host aus (über Tailnet erreichbar).
2. Veröffentliche DNS‑SD‑Records für `_openclaw-gw._tcp` unter einer dedizierten Zone
   (zum Beispiel `openclaw.internal.`).
3. Konfiguriere Tailscale‑**Split‑DNS** so, dass deine gewählte Domain für Clients
   (einschließlich iOS) über diesen DNS‑Server aufgelöst wird.

OpenClaw unterstützt jede Discovery‑Domain; `openclaw.internal.` ist nur ein Beispiel.
iOS-/Android‑Knoten durchsuchen sowohl `local.` als auch deine konfigurierte Wide‑Area‑Domain.

<div id="gateway-config-recommended">
  ### Gateway-Konfiguration (empfohlen)
</div>

```json5
{
  gateway: { bind: "tailnet" }, // tailnet-only (recommended)
  discovery: { wideArea: { enabled: true } } // aktiviert Wide-Area-DNS-SD-Veröffentlichung
}
```

<div id="onetime-dns-server-setup-gateway-host">
  ### Einmalige Einrichtung des DNS-Servers (Gateway-Host)
</div>

```bash
openclaw dns setup --apply
```

Dies installiert CoreDNS und konfiguriert es so, dass es:

* nur auf Port 53 auf den Tailscale‑Schnittstellen des Gateways lauscht
* deine gewählte Domain (Beispiel: `openclaw.internal.`) aus `~/.openclaw/dns/<domain>.db` ausliefert

Überprüfe dies von einem mit dem Tailnet verbundenen Rechner aus:

```bash
dns-sd -B _openclaw-gw._tcp openclaw.internal.
dig @<TAILNET_IPV4> -p 53 _openclaw-gw._tcp.openclaw.internal PTR +short
```

<div id="tailscale-dns-settings">
  ### Tailscale-DNS-Einstellungen
</div>

In der Tailscale-Admin-Konsole:

* Füge einen Nameserver hinzu, der auf die Tailnet-IP-Adresse des Gateways verweist (UDP/TCP 53).
* Füge Split DNS hinzu, damit deine Discovery-Domain diesen Nameserver verwendet.

Sobald Clients Tailnet-DNS akzeptieren, können iOS-Knoten in deiner Discovery-Domain den Dienst
`_openclaw-gw._tcp` auch ohne Multicast finden.

<div id="gateway-listener-security-recommended">
  ### Gateway-Listener-Sicherheit (empfohlen)
</div>

Der Gateway-WS-Port (Standard `18789`) ist standardmäßig an Loopback gebunden. Für LAN-/Tailnet-
Zugriff binde ihn explizit und lasse die Authentifizierung aktiviert.

Für reine Tailnet-Setups:

* Setze `gateway.bind: "tailnet"` in `~/.openclaw/openclaw.json`.
* Starte das Gateway neu (oder starte die macOS-Menüleisten-App neu).

<div id="what-advertises">
  ## Wer kündigt an
</div>

Nur das Gateway kündigt `_openclaw-gw._tcp` an.

<div id="service-types">
  ## Diensttypen
</div>

* `_openclaw-gw._tcp` — Gateway-Transport-Beacon (wird von macOS-/iOS-/Android-Knoten verwendet).

<div id="txt-keys-nonsecret-hints">
  ## TXT-Keys (nicht-geheime Hinweise)
</div>

Das Gateway veröffentlicht kleine, nicht-geheime Hinweise, um UI-Flows komfortabler zu machen:

* `role=gateway`
* `displayName=<friendly name>`
* `lanHost=<hostname>.local`
* `gatewayPort=<port>` (Gateway WS + HTTP)
* `gatewayTls=1` (nur, wenn TLS aktiviert ist)
* `gatewayTlsSha256=<sha256>` (nur, wenn TLS aktiviert ist und ein Fingerabdruck verfügbar ist)
* `canvasPort=<port>` (nur, wenn der Canvas-Host aktiviert ist; Standardwert `18793`)
* `sshPort=<port>` (Standard ist `22`, wenn nicht überschrieben)
* `transport=gateway`
* `cliPath=<path>` (optional; absoluter Pfad zu einem ausführbaren `openclaw`-Entrypoint)
* `tailnetDns=<magicdns>` (optionaler Hinweis, wenn Tailnet verfügbar ist)

<div id="debugging-on-macos">
  ## Debugging unter macOS
</div>

Nützliche integrierte Tools:

* Instanzen auflisten:
  ```bash
  dns-sd -B _openclaw-gw._tcp local.
  ```
* Eine Instanz auflösen (ersetze `<instance>`):
  ```bash
  dns-sd -L "<instance>" _openclaw-gw._tcp local.
  ```

Wenn das Auflisten funktioniert, aber das Auflösen fehlschlägt, liegt in der Regel ein Problem mit der LAN-Richtlinie oder dem mDNS-Resolver vor.

<div id="debugging-in-gateway-logs">
  ## Debugging in den Gateway-Logs
</div>

Das Gateway schreibt eine rotierende Logdatei (beim Start ausgegeben als
`gateway log file: ...`). Achte auf Zeilen mit `bonjour:`, insbesondere:

* `bonjour: advertise failed ...`
* `bonjour: ... name conflict resolved` / `hostname conflict resolved`
* `bonjour: watchdog detected non-announced service ...`

<div id="debugging-on-ios-node">
  ## Debugging auf dem iOS-Knoten
</div>

Der iOS-Knoten verwendet `NWBrowser`, um `_openclaw-gw._tcp` zu entdecken.

So erfasst du Logs:

* Einstellungen → Gateway → Erweitert → **Discovery Debug Logs**
* Einstellungen → Gateway → Erweitert → **Discovery Logs** → Problem reproduzieren → **Copy**

Das Log umfasst Zustandsübergänge des Browsers und Änderungen der Ergebnismenge.

<div id="common-failure-modes">
  ## Häufige Fehlerbilder
</div>

* **Bonjour funktioniert nicht über Netzwerkgrenzen hinweg**: verwende Tailnet oder SSH.
* **Multicast blockiert**: einige WLANs deaktivieren mDNS.
* **Sleep / Interface‑Wechsel**: macOS kann mDNS-Ergebnisse vorübergehend verwerfen; versuche es erneut.
* **Browsing funktioniert, aber Auflösung schlägt fehl**: halte Rechnernamen einfach (vermeide Emojis oder
  Satzzeichen) und starte dann den Gateway neu. Der Serviceinstanzname wird aus dem Hostnamen abgeleitet, daher können zu komplexe Namen manche Resolver verwirren.

<div id="escaped-instance-names-032">
  ## Maskierte Instanznamen (`\032`)
</div>

Bonjour/DNS‑SD maskiert häufig Bytes in Dienstinstanznamen als dezimale `\DDD`‑
Sequenzen (z. B. wird ein Leerzeichen zu `\032`).

* Das ist auf Protokollebene normal.
* UIs sollten diese Escapes für die Anzeige dekodieren (iOS verwendet `BonjourEscapes.decode`).

<div id="disabling-configuration">
  ## Deaktivierung / Konfiguration
</div>

* `OPENCLAW_DISABLE_BONJOUR=1` deaktiviert die Dienstankündigung (veraltet: `OPENCLAW_DISABLE_BONJOUR`).
* `gateway.bind` in `~/.openclaw/openclaw.json` steuert den Bind-Modus des Gateways.
* `OPENCLAW_SSH_PORT` überschreibt den im TXT-Eintrag angekündigten SSH-Port (veraltet: `OPENCLAW_SSH_PORT`).
* `OPENCLAW_TAILNET_DNS` veröffentlicht einen MagicDNS-Hinweis im TXT-Eintrag (veraltet: `OPENCLAW_TAILNET_DNS`).
* `OPENCLAW_CLI_PATH` überschreibt den angekündigten CLI-Pfad (veraltet: `OPENCLAW_CLI_PATH`).

<div id="related-docs">
  ## Verwandte Dokumentation
</div>

* Discovery-Richtlinie und Transportauswahl: [Discovery](/de/gateway/discovery)
* Knoten-Kopplung + Genehmigungen: [Gateway pairing](/de/gateway/pairing)