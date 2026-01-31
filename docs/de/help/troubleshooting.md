---
title: Fehlerbehebung
summary: "Zentrale zur Fehlerbehebung: Symptome → Checks → Lösungen"
read_when:
  - Du siehst einen Fehler und suchst den Lösungsweg
  - Der Installer meldet „Erfolg“, aber die CLI funktioniert nicht
---

<div id="troubleshooting">
  # Fehlerbehebung
</div>

<div id="first-60-seconds">
  ## Erste 60 Sekunden
</div>

Führe die folgenden Schritte der Reihe nach aus:

```bash
openclaw status
openclaw status --all
openclaw gateway probe
openclaw logs --follow
openclaw doctor
```

Wenn das Gateway erreichbar ist, führe tiefgehende Prüfungen durch:

```bash
openclaw status --deep
```

<div id="common-it-broke-cases">
  ## Häufige „Jetzt ist es kaputt“-Fälle
</div>

<div id="openclaw-command-not-found">
  ### `openclaw: command not found`
</div>

Fast immer ein Node.js-/npm-PATH-Problem. Fang hier an:

* [Installation (Node.js-/npm-PATH-Check)](/de/install#nodejs--npm-path-sanity)

<div id="installer-fails-or-you-need-full-logs">
  ### Installer schlägt fehl (oder du benötigst vollständige Logs)
</div>

Führe den Installer im ausführlichen (Verbose-)Modus erneut aus, um den vollständigen Trace und die npm-Ausgabe anzuzeigen:

```bash
curl -fsSL https://openclaw.bot/install.sh | bash -s -- --verbose
```

Bei Beta-Installationen:

```bash
curl -fsSL https://openclaw.bot/install.sh | bash -s -- --beta --verbose
```

Du kannst statt des Flags auch `OPENCLAW_VERBOSE=1` setzen.

<div id="gateway-unauthorized-cant-connect-or-keeps-reconnecting">
  ### Gateway meldet „unauthorized“, keine Verbindung möglich oder verbindet sich ständig neu
</div>

* [Gateway-Fehlerbehebung](/de/gateway/troubleshooting)
* [Gateway-Authentifizierung](/de/gateway/authentication)

<div id="control-ui-fails-on-http-device-identity-required">
  ### Control UI funktioniert über HTTP nicht (Geräteidentität erforderlich)
</div>

* [Gateway-Fehlerbehebung](/de/gateway/troubleshooting)
* [Control UI](/de/web/control-ui#insecure-http)

<div id="docsopenclawai-shows-an-ssl-error-comcastxfinity">
  ### `docs.openclaw.ai` zeigt einen SSL-Fehler (Comcast/Xfinity)
</div>

Einige Comcast/Xfinity‑Verbindungen blockieren `docs.openclaw.ai` über Xfinity Advanced Security.
Deaktiviere Advanced Security oder füge `docs.openclaw.ai` zur Allowlist hinzu und versuche es dann erneut.

* Hilfe zu Xfinity Advanced Security: https://www.xfinity.com/support/articles/using-xfinity-xfi-advanced-security
* Schnelle Checks: Verwende einen mobilen Hotspot oder ein VPN, um zu bestätigen, dass es sich um eine Filterung auf ISP‑Ebene handelt

<div id="service-says-running-but-rpc-probe-fails">
  ### Dienst läuft, aber RPC-Probe schlägt fehl
</div>

* [Gateway-Fehlerbehebung](/de/gateway/troubleshooting)
* [Hintergrundprozess / Dienst](/de/gateway/background-process)

<div id="modelauth-failures-rate-limit-billing-all-models-failed">
  ### Modell-/Auth-Fehler (Rate-Limit, Abrechnung, „alle Modelle sind fehlgeschlagen“)
</div>

* [Modelle](/de/cli/models)
* [OAuth-/Auth-Konzepte](/de/concepts/oauth)

<div id="model-says-model-not-allowed">
  ### `/model` meldet `model not allowed`
</div>

Das bedeutet in der Regel, dass `agents.defaults.models` als Allowlist konfiguriert ist. Wenn sie nicht leer ist,
können nur diese anbieter-/Modell-Schlüssel ausgewählt werden.

* Überprüfe die Allowlist: `openclaw config get agents.defaults.models`
* Füge das gewünschte Modell hinzu (oder leere die Allowlist) und führe `/model` erneut aus
* Verwende `/models`, um die erlaubten anbieter/Modelle zu durchsuchen

<div id="when-filing-an-issue">
  ### Beim Melden eines Issues
</div>

Füge einen sicheren Bericht ein:

```bash
openclaw status --all
```

Wenn möglich, füge einen relevanten Log-Ausschnitt aus `openclaw logs --follow` bei.
