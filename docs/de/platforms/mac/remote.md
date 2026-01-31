---
title: Remote
summary: "Ablauf in der macOS-App zur Steuerung eines entfernten OpenClaw-Gateways über SSH"
read_when:
  - Einrichten oder Debuggen der Remote-Steuerung vom Mac aus
---

<div id="remote-openclaw-macos-remote-host">
  # Remote OpenClaw (macOS ⇄ Remote-Host)
</div>

Mit diesem Ablauf kann die macOS-App als vollständige Fernsteuerung für ein OpenClaw-Gateway fungieren, das auf einem anderen Host (Desktop/Server) läuft. Dies ist die **Remote über SSH**-Funktion (Remote-Ausführung) der App. Alle Funktionen – Health-Checks, Voice-Wake-Weiterleitung und Web-Chat – verwenden dieselbe Remote-SSH-Konfiguration aus *Settings → General*.

<div id="modes">
  ## Modi
</div>

* **Lokal (dieser Mac)**: Alles läuft auf dem Laptop. Kein SSH beteiligt.
* **Remote über SSH (Standard)**: OpenClaw-Befehle werden auf dem Remote-Host ausgeführt. Die Mac-App öffnet eine SSH-Verbindung mit `-o BatchMode` plus deiner gewählten Identität/deinem Schlüssel und einer lokalen Portweiterleitung.
* **Remote direkt (ws/wss)**: Kein SSH-Tunnel. Die Mac-App verbindet sich direkt mit der Gateway-URL (zum Beispiel über Tailscale Serve oder einen öffentlichen HTTPS-Reverse-Proxy).

<div id="remote-transports">
  ## Remote-Transporte
</div>

Der Remote-Modus unterstützt zwei Übertragungsarten:

* **SSH-Tunnel** (Standard): Verwendet `ssh -N -L ...`, um den Gateway-Port auf `localhost` weiterzuleiten. Das Gateway sieht die IP-Adresse des Knotens als `127.0.0.1`, da der Tunnel über die Loopback-Schnittstelle läuft.
* **Direkt (ws/wss)**: Stellt eine direkte Verbindung zur Gateway-URL her. Das Gateway sieht die tatsächliche Client-IP-Adresse.

<div id="prereqs-on-the-remote-host">
  ## Voraussetzungen auf dem Remote-Host
</div>

1. Installiere Node.js + pnpm und baue/installiere die OpenClaw CLI (`pnpm install && pnpm build && pnpm link --global`).
2. Stelle sicher, dass `openclaw` im PATH für nicht-interaktive Shells verfügbar ist (falls nötig, einen Symlink nach `/usr/local/bin` oder `/opt/homebrew/bin` anlegen).
3. Richte SSH-Zugriff mit Schlüssel-Authentifizierung ein. Wir empfehlen **Tailscale**-IP-Adressen für stabile Erreichbarkeit außerhalb des LAN.

<div id="macos-app-setup">
  ## macOS-App-Einrichtung
</div>

1. Öffne *Settings → General*.
2. Wähle unter **OpenClaw runs** die Option **Remote over SSH** und konfiguriere:
   * **Transport**: **SSH tunnel** oder **Direct (ws/wss)**.
   * **SSH target**: `user@host` (optional `:port`).
     * Wenn das Gateway im selben LAN läuft und sich per Bonjour ankündigt, wähle es aus der gefundenen Liste aus, um dieses Feld automatisch auszufüllen.
   * **Gateway URL** (nur Direct): `wss://gateway.example.ts.net` (oder `ws://...` für lokal/LAN).
   * **Identity file** (erweitert): Pfad zu deinem Schlüssel.
   * **Project root** (erweitert): Remote-Checkout-Pfad, der für Befehle verwendet wird.
   * **CLI path** (erweitert): optionaler Pfad zu einem ausführbaren `openclaw`-Entrypoint/Binary (wird automatisch ausgefüllt, wenn per Bonjour angekündigt).
3. Klicke auf **Test remote**. Ein erfolgreicher Test zeigt an, dass der entfernte Befehl `openclaw status --json` korrekt ausgeführt wird. Fehler deuten in der Regel auf PATH-/CLI-Probleme hin; Exit-Code 127 bedeutet, dass die CLI auf dem entfernten System nicht gefunden wurde.
4. Health checks und Web-Chat werden nun automatisch über diesen SSH-Tunnel ausgeführt.

<div id="web-chat">
  ## Web Chat
</div>

* **SSH-Tunnel**: Web Chat verbindet sich über den weitergeleiteten WS-Control-Port (Standardport 18789) mit dem Gateway.
* **Direkt (ws/wss)**: Web Chat verbindet sich direkt mit der konfigurierten Gateway-URL.
* Es gibt keinen separaten WebChat-HTTP-Server mehr.

<div id="permissions">
  ## Berechtigungen
</div>

* Der Remote-Host benötigt dieselben TCC-Berechtigungen wie lokal (Automation, Bedienungshilfen, Bildschirmaufnahme, Mikrofon, Spracherkennung, Mitteilungen). Führe das Onboarding auf diesem Rechner aus, um sie einmalig zu gewähren.
* Knoten geben ihren Berechtigungsstatus über `node.list` / `node.describe` bekannt, damit Agenten wissen, was verfügbar ist.

<div id="security-notes">
  ## Sicherheitshinweise
</div>

* Bevorzuge Loopback-Binds auf dem Remote-Host und verbinde dich per SSH oder Tailscale.
* Wenn du das Gateway an ein Nicht-Loopback-Interface bindest, verlange Token-/Passwort-Authentifizierung.
* Siehe [Sicherheit](/de/gateway/security) und [Tailscale](/de/gateway/tailscale).

<div id="whatsapp-login-flow-remote">
  ## WhatsApp-Login-Flow (remote)
</div>

* Führe `openclaw channels login --verbose` **auf dem Remote-Host** aus. Scanne den QR-Code mit WhatsApp auf deinem Telefon.
* Führe den Login auf diesem Host erneut aus, wenn die Authentifizierung abläuft. Der Health-Check zeigt Verbindungsprobleme an.

<div id="troubleshooting">
  ## Fehlerbehebung
</div>

* **exit 127 / not found**: `openclaw` ist für Non-Login-Shells nicht im PATH. Füge es zu `/etc/paths` oder deiner Shell-RC-Datei hinzu, oder setze einen Symlink nach `/usr/local/bin`/`/opt/homebrew/bin`.
* **Health probe failed**: Prüfe die SSH-Erreichbarkeit, den PATH und ob Baileys angemeldet ist (`openclaw status --json`).
* **Web Chat hängt**: Stelle sicher, dass das Gateway auf dem Remote-Host läuft und der weitergeleitete Port dem Gateway-WS-Port entspricht; die UI benötigt eine stabile WS-Verbindung.
* **Node-IP zeigt 127.0.0.1**: ist beim SSH-Tunnel erwartetes Verhalten. Stelle **Transport** auf **Direct (ws/wss)**, wenn das Gateway die echte Client-IP sehen soll.
* **Voice Wake**: Trigger-Phrasen werden im Remote-Modus automatisch weitergeleitet; es ist kein separater Forwarder erforderlich.

<div id="notification-sounds">
  ## Benachrichtigungstöne
</div>

Wähle für jede Benachrichtigung einen Ton in Skripten mit `openclaw` und `node.invoke` aus, z. B.:

```bash
openclaw nodes notify --node <id> --title "Ping" --body "Remote gateway ready" --sound Glass
```

Es gibt in der App keinen globalen Schalter „Standardton“ mehr; der Ton wird jetzt für jede Anfrage einzeln ausgewählt (oder es wird keiner verwendet).
