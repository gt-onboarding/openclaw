---
title: iOS
summary: "iOS-Knoten-App: Verbindung zum Gateway, Kopplung, Canvas und Fehlerbehebung"
read_when:
  - Kopplung oder erneute Verbindung des iOS-Knotens
  - Ausführen der iOS-App aus dem Quellcode
  - Debugging der Gateway-Erkennung oder von Canvas-Befehlen
---

<div id="ios-app-node">
  # iOS App (Node)
</div>

Verfügbarkeit: interne Vorschauversion. Die iOS-App ist noch nicht öffentlich verfügbar.

<div id="what-it-does">
  ## Was es macht
</div>

* Verbindet sich per WebSocket mit dem Gateway (LAN oder Tailnet).
* Stellt Node-Funktionen bereit: Canvas, Bildschirm-Snapshot, Kameraaufnahme, Standort, Sprechmodus, Sprachaktivierung.
* Empfängt `node.invoke`-Befehle und sendet Node-Statusereignisse.

<div id="requirements">
  ## Anforderungen
</div>

* Gateway, das auf einem anderen Gerät ausgeführt wird (macOS, Linux oder Windows über WSL2).
* Netzwerkpfad:
  * Selbes LAN über Bonjour, **oder**
  * Tailnet über Unicast-DNS-SD (Beispieldomain: `openclaw.internal.`), **oder**
  * Manueller Host/Port (als Fallback).

<div id="quick-start-pair-connect">
  ## Schnellstart (Koppeln + Verbinden)
</div>

1. Starten Sie das Gateway:

```bash
openclaw gateway --port 18789
```

2. Öffne in der iOS-App die Einstellungen und wähle ein gefundenes Gateway aus (oder aktiviere „Manual Host“ und gib Host/Port ein).

3. Bestätige die Kopplungsanforderung auf dem Gateway-Host:

```bash
openclaw nodes pending
openclaw nodes approve <requestId>
```

4. Verbindung prüfen:

```bash
openclaw nodes status
openclaw gateway call node.list --params "{}"
```

<div id="discovery-paths">
  ## Discovery-Methoden
</div>

<div id="bonjour-lan">
  ### Bonjour (LAN)
</div>

Das Gateway kündigt `_openclaw-gw._tcp` auf `local.` an. Die iOS-App listet sie automatisch auf.

<div id="tailnet-cross-network">
  ### Tailnet (netzwerkübergreifend)
</div>

Wenn mDNS blockiert ist, verwende eine Unicast-DNS-SD-Zone (wähle eine Domain; Beispiel: `openclaw.internal.`) und Tailscale Split DNS.
Siehe [Bonjour](/de/gateway/bonjour) für das CoreDNS-Beispiel.

<div id="manual-hostport">
  ### Manuelle Host/Port-Konfiguration
</div>

Aktiviere in den Einstellungen **Manual Host** und gib den Gateway-Host und -Port ein (Standardport ist `18789`).

<div id="canvas-a2ui">
  ## Canvas + A2UI
</div>

Der iOS-Knoten rendert ein WKWebView-Canvas. Verwende `node.invoke`, um es zu steuern:

```bash
openclaw nodes invoke --node "iOS Node" --command canvas.navigate --params '{"url":"http://<gateway-host>:18793/__openclaw__/canvas/"}'
```

Hinweise:

* Der Gateway-Canvas-Host stellt `/__openclaw__/canvas/` und `/__openclaw__/a2ui/` bereit.
* Der iOS Node navigiert bei der Verbindung automatisch zu A2UI, wenn eine Canvas-Host-URL bereitgestellt wird.
* Kehren Sie mit `canvas.navigate` und `{"url":""}` zum integrierten Scaffold zurück.

<div id="canvas-eval-snapshot">
  ### Canvas-Evaluierung / Snapshot
</div>

```bash
openclaw nodes invoke --node "iOS Node" --command canvas.eval --params '{"javaScript":"(() => { const {ctx} = window.__openclaw; ctx.clearRect(0,0,innerWidth,innerHeight); ctx.lineWidth=6; ctx.strokeStyle=\"#ff2d55\"; ctx.beginPath(); ctx.moveTo(40,40); ctx.lineTo(innerWidth-40, innerHeight-40); ctx.stroke(); return \"ok\"; })()"}'
```

```bash
openclaw nodes invoke --node "iOS Node" --command canvas.snapshot --params '{"maxWidth":900,"format":"jpeg"}'
```

<div id="voice-wake-talk-mode">
  ## Sprachaktivierung + Sprechmodus
</div>

* Sprachaktivierung und Sprechmodus sind in den Einstellungen verfügbar.
* iOS kann Audiowiedergabe im Hintergrund aussetzen; behandle Sprachfunktionen nach dem Best-Effort-Prinzip, wenn die App nicht aktiv ist.

<div id="common-errors">
  ## Häufige Fehler
</div>

* `NODE_BACKGROUND_UNAVAILABLE`: bringe die iOS-App in den Vordergrund (canvas-/camera-/screen-Befehle erfordern dies).
* `A2UI_HOST_NOT_CONFIGURED`: das Gateway hat keine Canvas-Host-URL bereitgestellt; überprüfe `canvasHost` in der [Gateway-Konfiguration](/de/gateway/configuration).
* Kopplungsaufforderung erscheint nie: führe `openclaw nodes pending` aus und genehmige manuell.
* Wiederverbindung nach Neuinstallation schlägt fehl: das Keychain-Kopplungstoken wurde gelöscht; kopple den Knoten erneut.

<div id="related-docs">
  ## Verwandte Dokumentation
</div>

* [Kopplung](/de/gateway/pairing)
* [Erkennung](/de/gateway/discovery)
* [Bonjour](/de/gateway/bonjour)