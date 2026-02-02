---
title: Node
summary: "CLI-Referenz zu `openclaw node` (Headless-Knoten-Host)"
read_when:
  - Ausführen des Headless-Knoten-Hosts
  - Kopplung eines nicht-macOS-Knotens für system.run
---

<div id="openclaw-node">
  # `openclaw node`
</div>

Starte einen headless Knoten-Host, der sich mit dem WebSocket des Gateways verbindet und
`system.run` / `system.which` auf dieser Maschine bereitstellt.

<div id="why-use-a-node-host">
  ## Warum einen Knoten-Host verwenden?
</div>

Verwende einen Knoten-Host, wenn du möchtest, dass Agenten **Befehle auf anderen Maschinen** in deinem
Netzwerk ausführen, ohne dort eine vollständige macOS-Companion-App installieren zu müssen.

Typische Anwendungsfälle:

* Befehle auf Remote-Linux-/Windows-Rechnern ausführen (Build-Server, Laborrechner, NAS).
* `exec` weiterhin im Gateway **sandboxen**, aber freigegebene Ausführungen an andere Hosts delegieren.
* Ein leichtgewichtiges, headless Ausführungsziel für Automatisierung oder CI-Knoten bereitstellen.

Die Ausführung wird weiterhin durch **exec approvals** und Allowlists pro Agent auf dem
Knoten-Host abgesichert, sodass du den Befehlszugriff klar begrenzt und explizit halten kannst.

<div id="browser-proxy-zero-config">
  ## Browser-Proxy (Zero-Config)
</div>

Knoten-Hosts stellen automatisch einen Browser-Proxy bereit, sofern `browser.enabled` auf dem Knoten nicht deaktiviert ist. Dadurch kann der Agent Browser-Automatisierung auf diesem Knoten ohne zusätzliche Konfiguration verwenden.

Deaktiviere ihn bei Bedarf auf dem Knoten:

```json5
{
  nodeHost: {
    browserProxy: {
      enabled: false
    }
  }
}
```

<div id="run-foreground">
  ## Ausführen (im Vordergrund)
</div>

```bash
openclaw node run --host <gateway-host> --port 18789
```

Optionen:

* `--host <host>`: Gateway-WebSocket-Host (Standardwert: `127.0.0.1`)
* `--port <port>`: Gateway-WebSocket-Port (Standardwert: `18789`)
* `--tls`: TLS für die Gateway-Verbindung verwenden
* `--tls-fingerprint <sha256>`: Erwarteter Fingerabdruck des TLS-Zertifikats (sha256)
* `--node-id <id>`: Knoten-ID überschreiben (löscht Kopplungs-Token)
* `--display-name <name>`: Anzeigenamen des Knotens überschreiben

<div id="service-background">
  ## Dienst (im Hintergrund)
</div>

Installiere einen Headless-Knotenhost als Benutzerdienst.

```bash
openclaw node install --host <gateway-host> --port 18789
```

Optionen:

* `--host <host>`: Gateway-WebSocket-Host (Standardwert: `127.0.0.1`)
* `--port <port>`: Gateway-WebSocket-Port (Standardwert: `18789`)
* `--tls`: TLS für die Gateway-Verbindung verwenden
* `--tls-fingerprint <sha256>`: Erwarteter TLS-Zertifikat-Fingerabdruck (SHA-256)
* `--node-id <id>`: Knoten-ID überschreiben (löscht Kopplungs-Token)
* `--display-name <name>`: Anzeigenamen des Knotens überschreiben
* `--runtime <runtime>`: Service-Runtime (`node` oder `bun`)
* `--force`: Neu installieren/überschreiben, falls bereits installiert

Service verwalten:

```bash
openclaw node status
openclaw node stop
openclaw node restart
openclaw node uninstall
```

Verwende `openclaw node run` für einen Node-Host im Vordergrund (ohne Dienst).

Dienstbefehle akzeptieren die Option `--json` für maschinenlesbare Ausgaben.

<div id="pairing">
  ## Kopplung
</div>

Die erste Verbindung erzeugt im Gateway eine ausstehende Kopplungsanfrage für den Knoten.
Genehmige sie über:

```bash
openclaw nodes pending
openclaw nodes approve <requestId>
```

Der Node-Host speichert seine Node-ID, sein Token, seinen Anzeigenamen und die Gateway-Verbindungsdaten in
`~/.openclaw/node.json`.

<div id="exec-approvals">
  ## Ausführungsfreigaben
</div>

`system.run` erfordert lokale Ausführungsfreigaben:

* `~/.openclaw/exec-approvals.json`
* [Ausführungsfreigaben](/de/tools/exec-approvals)
* `openclaw approvals --node <id|name|ip>` (vom Gateway aus bearbeiten)