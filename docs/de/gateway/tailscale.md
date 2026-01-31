---
title: Tailscale
summary: "Integriertes Tailscale Serve/Funnel für das Gateway-Dashboard"
read_when:
  - Bereitstellung der Gateway Control UI außerhalb von localhost
  - Automatisierung des Zugriffs auf das Tailnet- oder öffentliche Dashboard
---

<div id="tailscale-gateway-dashboard">
  # Tailscale (Gateway-Dashboard)
</div>

OpenClaw kann Tailscale **Serve** (Tailnet) oder **Funnel** (öffentlich) für das
Gateway-Dashboard und den WebSocket-Port automatisch konfigurieren. Dadurch bleibt das Gateway an die Loopback-Schnittstelle gebunden, während
Tailscale HTTPS, Routing und (für Serve) Identitäts-Header bereitstellt.

<div id="modes">
  ## Modi
</div>

- `serve`: Serve nur innerhalb des Tailnets über `tailscale serve`. Das Gateway bleibt auf `127.0.0.1`.
- `funnel`: Öffentliches HTTPS über `tailscale funnel`. OpenClaw erfordert ein gemeinsames Passwort.
- `off`: Standardwert (keine Tailscale-Automatisierung).

<div id="auth">
  ## Auth
</div>

Setze `gateway.auth.mode` fest, um den Handshake zu steuern:

- `token` (Standard, wenn `OPENCLAW_GATEWAY_TOKEN` gesetzt ist)
- `password` (gemeinsamer geheimer Schlüssel über `OPENCLAW_GATEWAY_PASSWORD` oder Konfiguration)

Wenn `tailscale.mode = "serve"` und `gateway.auth.allowTailscale` auf `true` steht,
können gültige Serve-Proxy-Anfragen sich über Tailscale-Identity-Header
(`tailscale-user-login`) authentifizieren, ohne ein Token/Passwort anzugeben. OpenClaw
verifiziert die Identität, indem es die `x-forwarded-for`-Adresse über den lokalen
Tailscale-Daemon (`tailscale whois`) auflöst und sie mit dem Header abgleicht,
bevor die Anfrage akzeptiert wird. OpenClaw behandelt eine Anfrage nur dann als Serve,
wenn sie vom Loopback-Interface mit den Tailscale-Headern `x-forwarded-for`,
`x-forwarded-proto` und `x-forwarded-host` eintrifft.
Um explizite Zugangsdaten zu erzwingen, setze `gateway.auth.allowTailscale: false`
oder erzwinge `gateway.auth.mode: "password"`.

<div id="config-examples">
  ## Konfigurationsbeispiele
</div>

<div id="tailnet-only-serve">
  ### Nur Tailnet (Serve)
</div>

```json5
{
  gateway: {
    bind: "loopback",
    tailscale: { mode: "serve" }
  }
}
```

Öffne: `https://<magicdns>/` (oder den konfigurierten `gateway.controlUi.basePath`)


<div id="tailnet-only-bind-to-tailnet-ip">
  ### Nur Tailnet (an Tailnet-IP binden)
</div>

Verwende diese Option, wenn der Gateway direkt an der Tailnet-IP lauschen soll (ohne Serve/Funnel).

```json5
{
  gateway: {
    bind: "tailnet",
    auth: { mode: "token", token: "your-token" }
  }
}
```

Von einem anderen Tailnet-Gerät aus verbinden:

* Control UI: `http://<tailscale-ip>:18789/`
* WebSocket: `ws://<tailscale-ip>:18789`

Hinweis: Loopback (`http://127.0.0.1:18789`) funktioniert in diesem Modus **nicht**.


<div id="public-internet-funnel-shared-password">
  ### Öffentliches Internet (Funnel + gemeinsames Passwort)
</div>

```json5
{
  gateway: {
    bind: "loopback",
    tailscale: { mode: "funnel" },
    auth: { mode: "password", password: "replace-me" }
  }
}
```

Verwende nach Möglichkeit `OPENCLAW_GATEWAY_PASSWORD`, statt ein Passwort im Dateisystem zu hinterlegen.


<div id="cli-examples">
  ## CLI-Beispiele
</div>

```bash
openclaw gateway --tailscale serve
openclaw gateway --tailscale funnel --auth password
```


<div id="notes">
  ## Hinweise
</div>

- Tailscale Serve/Funnel erfordert, dass die `tailscale` CLI installiert ist und du angemeldet bist.
- `tailscale.mode: "funnel"` startet nicht, sofern der Authentifizierungsmodus nicht `password` ist, um eine öffentliche Freigabe zu vermeiden.
- Setze `gateway.tailscale.resetOnExit`, wenn OpenClaw beim Herunterfahren die Konfiguration von
  `tailscale serve` oder `tailscale funnel` zurücksetzen soll.
- `gateway.bind: "tailnet"` ist eine direkte Tailnet-Bindung (kein HTTPS, kein Serve/Funnel).
- `gateway.bind: "auto"` bevorzugt Loopback; verwende `tailnet`, wenn du ausschließlich Tailnet-Zugriff möchtest.
- Serve/Funnel machen nur die **Gateway Control UI + WS** erreichbar. Nodes verbinden sich über
  denselben Gateway-WS-Endpunkt, daher kann Serve auch für den Zugriff auf Nodes verwendet werden.

<div id="browser-control-remote-gateway-local-browser">
  ## Browser-Steuerung (entferntes Gateway + lokaler Browser)
</div>

Wenn du das Gateway auf einer Maschine betreibst, aber einen Browser auf einer anderen Maschine steuern willst,
führe auf der Browser-Maschine einen **Knoten-Host** aus und sorge dafür, dass beide im selben Tailnet sind.
Das Gateway leitet Browser-Aktionen an den Knoten weiter; kein separater Steuerungsserver oder Serve-URL erforderlich.

Vermeide Funnel für die Browser-Steuerung; behandle die Knoten-Kopplung wie Operatorzugriff.

<div id="tailscale-prerequisites-limits">
  ## Tailscale-Voraussetzungen + Einschränkungen
</div>

- Serve erfordert aktiviertes HTTPS für dein Tailnet; die CLI weist dich darauf hin, falls es fehlt.
- Serve fügt Tailscale-Identitäts-Header ein; Funnel nicht.
- Funnel erfordert Tailscale v1.38.3+, MagicDNS, aktiviertes HTTPS und ein Funnel-Knotenattribut.
- Funnel unterstützt nur die Ports `443`, `8443` und `10000` über TLS.
- Funnel unter macOS erfordert die Open-Source-Variante der Tailscale-App.

<div id="learn-more">
  ## Weiterführende Informationen
</div>

- Übersicht über Tailscale Serve: https://tailscale.com/kb/1312/serve
- Befehl `tailscale serve`: https://tailscale.com/kb/1242/tailscale-serve
- Übersicht über Tailscale Funnel: https://tailscale.com/kb/1223/tailscale-funnel
- Befehl `tailscale funnel`: https://tailscale.com/kb/1311/tailscale-funnel