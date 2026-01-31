---
title: Remotezugriff
summary: "Remotezugriff über SSH-Tunnel (Gateway WS) und Tailnets"
read_when:
  - Betrieb oder Fehlerbehebung von Remote-Gateway-Setups
---

<div id="remote-access-ssh-tunnels-and-tailnets">
  # Fernzugriff (SSH, Tunnels und Tailnets)
</div>

Dieses Repository unterstützt „Remote-Zugriff über SSH“, indem ein einzelnes Gateway (das primäre) dauerhaft auf einem dedizierten Host (Desktop/Server) läuft und Clients sich damit verbinden.

* Für **Operatoren (dich / die macOS-App)**: SSH-Tunneling ist der universelle Fallback.
* Für **Knoten (iOS/Android und zukünftige Geräte)**: über den **WebSocket** des Gateways verbinden (LAN/Tailnet oder SSH-Tunnel nach Bedarf).

<div id="the-core-idea">
  ## Die Kernidee
</div>

* Der Gateway-WebSocket lauscht auf **Loopback** auf deinem konfigurierten Port (standardmäßig 18789).
* Für die Remote-Nutzung leitest du diesen Loopback-Port per SSH weiter (oder nutzt ein Tailnet/VPN und brauchst weniger Tunneling).

<div id="common-vpntailnet-setups-where-the-agent-lives">
  ## Übliche VPN-/Tailnet-Setups (wo der agent lebt)
</div>

Stell dir den **Gateway-Host** als den Ort vor, an dem der agent „lebt“. Er besitzt Sitzungen, Auth-Profile, Kanäle und Status.
Dein Laptop oder Desktop (und Knoten) verbinden sich mit diesem Host.

<div id="1-always-on-gateway-in-your-tailnet-vps-or-home-server">
  ### 1) Dauerhaft laufender Gateway in deinem Tailnet (VPS oder Heimserver)
</div>

Betreibe den Gateway auf einem persistenten Host und erreiche ihn über **Tailscale** oder SSH.

* **Beste UX:** `gateway.bind: "loopback"` beibehalten und **Tailscale Serve** für die Control UI verwenden.
* **Fallback:** Loopback beibehalten + SSH-Tunnel von jedem Rechner, der Zugriff benötigt.
* **Beispiele:** [exe.dev](/de/platforms/exe-dev) (einfache VM) oder [Hetzner](/de/platforms/hetzner) (Produktions-VPS).

Das ist ideal, wenn dein Laptop häufig in den Ruhezustand geht, du den agent aber dauerhaft verfügbar haben möchtest.

<div id="2-home-desktop-runs-the-gateway-laptop-is-remote-control">
  ### 2) Desktop-Rechner zu Hause führt das Gateway aus, Laptop dient als Fernsteuerung
</div>

Der Laptop führt den agent **nicht** aus. Er verbindet sich per Fernzugriff:

* Verwende in der macOS-App den **Remote over SSH**-Modus (Settings → General → „OpenClaw runs“).
* Die App öffnet und verwaltet den Tunnel, sodass WebChat + Health-Checks „einfach funktionieren“.

Runbook: [macOS-Remotezugriff](/de/platforms/mac/remote).

<div id="3-laptop-runs-the-gateway-remote-access-from-other-machines">
  ### 3) Laptop betreibt das Gateway, Fernzugriff von anderen Rechnern
</div>

Lass das Gateway lokal, aber mache es sicher erreichbar:

* SSH-Tunnel zum Laptop von anderen Rechnern, oder
* Tailscale Serve für die Control UI verwenden und das Gateway auf Loopback beschränkt lassen.

Anleitung: [Tailscale](/de/gateway/tailscale) und [Web-Übersicht](/de/web).

<div id="command-flow-what-runs-where">
  ## Befehlsfluss (was läuft wo)
</div>

Ein Gateway-Dienst hält Zustand + Kanäle. Knoten fungieren als Peripheriegeräte.

Ablaufbeispiel (Telegram → Knoten):

* Eine Telegram-Nachricht trifft im **Gateway** ein.
* Das Gateway führt den **agent** aus und entscheidet, ob ein Knoten-Tool aufgerufen werden soll.
* Das Gateway ruft den **Knoten** über den Gateway-WebSocket (`node.*` RPC) auf.
* Der Knoten gibt das Ergebnis zurück; das Gateway antwortet wieder an Telegram.

Hinweise:

* **Knoten führen den Gateway-Dienst nicht aus.** Pro Host sollte nur ein Gateway laufen, es sei denn, du betreibst bewusst isolierte Profile (siehe [Multiple gateways](/de/gateway/multiple-gateways)).
* Der macOS-App-„Node-Modus“ ist lediglich ein Node-Client über den Gateway-WebSocket.

<div id="ssh-tunnel-cli-tools">
  ## SSH-Tunnel (CLI + Tools)
</div>

Erstelle einen lokalen Tunnel zum WS des Remote-Gateways:

```bash
ssh -N -L 18789:127.0.0.1:18789 user@host
```

Mit aktivem Tunnel:

* `openclaw health` und `openclaw status --deep` erreichen nun das Remote-Gateway über `ws://127.0.0.1:18789`.
* `openclaw gateway {status,health,send,agent,call}` kann bei Bedarf ebenfalls mit `--url` die weitergeleitete URL ansteuern.

Hinweis: Ersetze `18789` durch deinen konfigurierten `gateway.port` (oder `--port`/`OPENCLAW_GATEWAY_PORT`).

<div id="cli-remote-defaults">
  ## CLI-Remote-Standardwerte
</div>

Du kannst ein Remote-Ziel dauerhaft speichern, damit CLI-Befehle es standardmäßig verwenden:

```json5
{
  gateway: {
    mode: "remote",
    remote: {
      url: "ws://127.0.0.1:18789",
      token: "your-token"
    }
  }
}
```

Wenn das Gateway nur über Loopback erreichbar ist, belasse die URL auf `ws://127.0.0.1:18789` und öffne zuerst den SSH-Tunnel.

<div id="chat-ui-over-ssh">
  ## Chat-UI über SSH
</div>

WebChat verwendet keinen separaten HTTP-Port mehr. Die SwiftUI-Chat-UI verbindet sich direkt mit dem WebSocket des Gateways.

* Leite den Port `18789` über SSH weiter (siehe oben) und verbinde lokale Clients anschließend mit `ws://127.0.0.1:18789`.
* Unter macOS solltest du bevorzugt den Modus „Remote over SSH“ der App verwenden, der den Tunnel automatisch verwaltet.

<div id="macos-app-remote-over-ssh">
  ## macOS-App „Remote over SSH“
</div>

Die macOS-Menüleisten-App kann dasselbe Setup End-to-End steuern (Remote-Statuschecks, WebChat und Voice-Wake-Weiterleitung).

Runbook: [Remotezugriff unter macOS](/de/platforms/mac/remote).

<div id="security-rules-remotevpn">
  ## Sicherheitsregeln (Remote/VPN)
</div>

Kurzfassung: **betreibe das Gateway ausschließlich auf Loopback**, es sei denn, du bist sicher, dass du ein externes Bind brauchst.

* **Loopback + SSH/Tailscale Serve** ist die sicherste Voreinstellung (keine öffentliche Freigabe).
* **Nicht-Loopback-Binds** (`lan`/`tailnet`/`custom` oder `auto`, wenn Loopback nicht verfügbar ist) müssen Auth-Tokens/Passwörter verwenden.
* `gateway.remote.token` ist **nur** für Remote-CLI-Aufrufe — es **aktiviert keine** lokale Authentifizierung.
* `gateway.remote.tlsFingerprint` pinnt das TLS-Zertifikat der Gegenstelle bei Verwendung von `wss://`.
* **Tailscale Serve** kann über Identity-Header authentifizieren, wenn `gateway.auth.allowTailscale: true` ist.
  Setze es auf `false`, wenn du stattdessen Tokens/Passwörter verwenden möchtest.
* Behandle Browser-Steuerung wie Operator-Zugriff: nur im Tailnet + explizite Knoten-Kopplung.

Ausführliche Infos: [Sicherheit](/de/gateway/security).