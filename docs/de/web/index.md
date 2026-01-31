---
title: Web
summary: "Gateway-Weboberflächen: Control UI, Bind-Modi und Sicherheit"
read_when:
  - Du möchtest über Tailscale auf das Gateway zugreifen
  - Du möchtest im Browser auf die Control UI zugreifen und Konfigurationen bearbeiten
---

<div id="web-gateway">
  # Web (Gateway)
</div>

Das Gateway stellt eine kleine **browserbasierte Control UI** (Vite + Lit) über denselben Port wie den Gateway-WS-Endpunkt bereit:

* Standard: `http://<host>:18789/`
* optionales Präfix: setze `gateway.controlUi.basePath` (z. B. `/openclaw`)

Die Funktionen befinden sich in der [Control UI](/de/web/control-ui).
Diese Seite konzentriert sich auf Bind-Modi, Sicherheit und öffentlich erreichbare Web-Oberflächen.

<div id="webhooks">
  ## Webhooks
</div>

Wenn `hooks.enabled=true` ist, stellt das Gateway ebenfalls einen kleinen Webhook-Endpunkt auf demselben HTTP-Server bereit.
Siehe [Gateway-Konfiguration](/de/gateway/configuration) → `hooks` für Authentifizierung und Payloads.

<div id="config-default-on">
  ## Konfiguration (standardmäßig aktiviert)
</div>

Die Control UI ist **standardmäßig aktiviert**, sobald Assets vorhanden sind (`dist/control-ui`).
Du kannst sie per Konfiguration steuern:

```json5
{
  gateway: {
    controlUi: { enabled: true, basePath: "/openclaw" } // basePath optional
  }
}
```

<div id="tailscale-access">
  ## Zugriff über Tailscale
</div>

<div id="integrated-serve-recommended">
  ### Integriertes Serve (empfohlen)
</div>

Belasse das Gateway auf Loopback und lasse Tailscale Serve als Proxy davor schalten:

```json5
{
  gateway: {
    bind: "loopback",
    tailscale: { mode: "serve" }
  }
}
```

Starte anschließend das Gateway:

```bash
openclaw gateway
```

Öffnen:

* `https://<magicdns>/` (oder Ihr konfigurierter `gateway.controlUi.basePath`)

<div id="tailnet-bind-token">
  ### Tailnet Bind + Token
</div>

```json5
{
  gateway: {
    bind: "tailnet",
    controlUi: { enabled: true },
    auth: { mode: "token", token: "your-token" }
  }
}
```

Starte anschließend das Gateway (Token erforderlich für Non-Loopback-Binds):

```bash
openclaw gateway
```

Öffne in deinem Browser:

* `http://<tailscale-ip>:18789/` (oder deinen konfigurierten `gateway.controlUi.basePath`)

<div id="public-internet-funnel">
  ### Öffentliches Internet (Funnel)
</div>

```json5
{
  gateway: {
    bind: "loopback",
    tailscale: { mode: "funnel" },
    auth: { mode: "password" } // oder OPENCLAW_GATEWAY_PASSWORD
  }
}
```

<div id="security-notes">
  ## Sicherheitshinweise
</div>

* Gateway-Authentifizierung ist standardmäßig erforderlich (Token/Passwort oder Tailscale-Identitäts-Header).
* Nicht-Loopback-Bindings **erfordern** weiterhin ein gemeinsames Token/Passwort (`gateway.auth` oder Umgebungsvariablen).
* Der Wizard generiert standardmäßig ein Gateway-Token (auch bei Loopback).
* Die UI sendet `connect.params.auth.token` oder `connect.params.auth.password`.
* Mit Serve können Tailscale-Identitäts-Header die Authentifizierung erfüllen, wenn
  `gateway.auth.allowTailscale` auf `true` gesetzt ist (kein Token/Passwort erforderlich). Setzen Sie
  `gateway.auth.allowTailscale: false`, um explizite Zugangsdaten zu erzwingen. Siehe
  [Tailscale](/de/gateway/tailscale) und [Sicherheit](/de/gateway/security).
* `gateway.tailscale.mode: "funnel"` erfordert `gateway.auth.mode: "password"` (gemeinsam genutztes Passwort).

<div id="building-the-ui">
  ## UI erstellen
</div>

Das Gateway stellt statische Dateien aus `dist/control-ui` bereit. Erstelle sie mit:

```bash
pnpm ui:build # installiert UI-Abhängigkeiten beim ersten Ausführen automatisch
```
