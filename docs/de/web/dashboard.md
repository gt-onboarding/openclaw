---
title: Dashboard
summary: "Zugriff auf und Authentifizierung für das Gateway-Dashboard (Control UI)"
read_when:
  - Beim Ändern der Authentifizierungs- oder Freigabemodi des Dashboards
---

<div id="dashboard-control-ui">
  # Dashboard (Control UI)
</div>

Das Gateway-Dashboard ist die browserbasierte Control UI, die standardmäßig unter `/` bereitgestellt wird
(kann mit `gateway.controlUi.basePath` überschrieben werden).

Schnellzugriff (lokales Gateway):

* http://127.0.0.1:18789/ (oder http://localhost:18789/)

Wichtige Referenzen:

* [Control UI](/de/web/control-ui) für Nutzung und UI-Funktionen.
* [Tailscale](/de/gateway/tailscale) für Serve-/Funnel-Automatisierung.
* [Web surfaces](/de/web) für Bind-Modi und Sicherheitshinweise.

Die Authentifizierung wird beim WebSocket-Handshake über `connect.params.auth`
(Token oder Passwort) erzwungen. Siehe `gateway.auth` in der [Gateway-Konfiguration](/de/gateway/configuration).

Sicherheitshinweis: Die Control UI ist eine **Admin-Oberfläche** (Chat, Konfiguration, Exec-Freigaben).
Mache sie nicht öffentlich zugänglich. Die UI speichert das Token nach dem ersten Laden in `localStorage`.
Bevorzuge localhost, Tailscale Serve oder einen SSH-Tunnel.

<div id="fast-path-recommended">
  ## Schnellstart (empfohlen)
</div>

* Nach dem Onboarding öffnet die CLI jetzt automatisch das Dashboard mit deinem Token und gibt denselben tokenisierten Link im Terminal aus.
* Jederzeit erneut öffnen: `openclaw dashboard` (kopiert den Link, öffnet den Browser, wenn möglich, und zeigt einen SSH-Hinweis im Headless-Betrieb).
* Das Token bleibt lokal (nur als Query-Parameter); die UI entfernt es nach dem ersten Laden und speichert es in localStorage.

<div id="token-basics-local-vs-remote">
  ## Token-Grundlagen (lokal vs. remote)
</div>

* **Localhost**: Rufe `http://127.0.0.1:18789/` im Browser auf. Wenn „unauthorized“ angezeigt wird, führe `openclaw dashboard` aus und verwende den tokenisierten Link (`?token=...`).
* **Token-Quelle**: `gateway.auth.token` (oder `OPENCLAW_GATEWAY_TOKEN`); die UI speichert ihn nach dem ersten Laden.
* **Nicht Localhost**: Verwende Tailscale Serve (ohne Token, wenn `gateway.auth.allowTailscale: true`), eine Tailnet-Bindung mit Token oder einen SSH-Tunnel. Siehe [Web surfaces](/de/web).

<div id="if-you-see-unauthorized-1008">
  ## Wenn „unauthorized“ / 1008 angezeigt wird
</div>

* Führe `openclaw dashboard` aus, um einen neuen Link mit Token zu erhalten.
* Stelle sicher, dass das Gateway erreichbar ist (lokal: `openclaw status`; remote: per SSH-Tunnel `ssh -N -L 18789:127.0.0.1:18789 user@host` und dann `http://127.0.0.1:18789/?token=...` im Browser öffnen).
* Füge in den Dashboard-Einstellungen denselben Token ein, den du in `gateway.auth.token` (oder `OPENCLAW_GATEWAY_TOKEN`) konfiguriert hast.