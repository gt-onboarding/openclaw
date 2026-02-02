---
title: Fly.io
description: OpenClaw auf Fly.io bereitstellen
---

<div id="flyio-deployment">
  # Fly.io-Deployment
</div>

**Ziel:** Das OpenClaw Gateway auf einer [Fly.io](https://fly.io)-Instanz mit persistentem Speicher, automatischem HTTPS und Discord-/Channel-Zugriff betreiben.

<div id="what-you-need">
  ## Was du benötigst
</div>

- [flyctl CLI](https://fly.io/docs/hands-on/install-flyctl/) installiert
- Fly.io-Konto (kostenloser Tarif reicht aus)
- Modell-Authentifizierung: Anthropic API-Schlüssel (oder andere Anbieter-Schlüssel)
- Kanal-Zugangsdaten: Discord-Bot-Token, Telegram-Token, usw.

<div id="beginner-quick-path">
  ## Schnellstart für Einsteiger
</div>

1. Repository klonen → `fly.toml` anpassen
2. App und Volume erstellen → Secrets setzen
3. Mit `fly deploy` bereitstellen
4. Per SSH einloggen, um die Konfiguration zu erstellen, oder die Control UI verwenden

<div id="1-create-the-fly-app">
  ## 1) Fly-App erstellen
</div>

```bash
# Repository klonen
git clone https://github.com/openclaw/openclaw.git
cd openclaw

# Neue Fly-App erstellen (eigenen Namen wählen)
fly apps create my-openclaw

# Persistentes Volume erstellen (1 GB ist normalerweise ausreichend)
fly volumes create openclaw_data --size 1 --region iad
```

**Tipp:** Wähle eine Region in deiner Nähe. Typische Optionen sind: `lhr` (London), `iad` (Virginia), `sjc` (San Jose).


<div id="2-configure-flytoml">
  ## 2) `fly.toml` konfigurieren
</div>

Passe `fly.toml` an deinen App-Namen und deine Anforderungen an.

**Sicherheitshinweis:** Die Standardkonfiguration macht eine öffentliche URL verfügbar. Für ein gehärtetes Deployment ohne öffentliche IP-Adresse siehe [Private Deployment](#private-deployment-hardened) oder verwende `fly.private.toml`.

```toml
app = "my-openclaw"  # Ihr App-Name
primary_region = "iad"

[build]
  dockerfile = "Dockerfile"

[env]
  NODE_ENV = "production"
  OPENCLAW_PREFER_PNPM = "1"
  OPENCLAW_STATE_DIR = "/data"
  NODE_OPTIONS = "--max-old-space-size=1536"

[processes]
  app = "node dist/index.js gateway --allow-unconfigured --port 3000 --bind lan"

[http_service]
  internal_port = 3000
  force_https = true
  auto_stop_machines = false
  auto_start_machines = true
  min_machines_running = 1
  processes = ["app"]

[[vm]]
  size = "shared-cpu-2x"
  memory = "2048mb"

[mounts]
  source = "openclaw_data"
  destination = "/data"
```

**Wichtige Einstellungen:**

| Einstellung                    | Begründung                                                                                 |
| ------------------------------ | ------------------------------------------------------------------------------------------ |
| `--bind lan`                   | Bindet an `0.0.0.0`, damit der Fly-Proxy das Gateway erreichen kann                        |
| `--allow-unconfigured`         | Startet ohne Konfigurationsdatei (du legst sie anschließend an)                            |
| `internal_port = 3000`         | Muss mit `--port 3000` (oder `OPENCLAW_GATEWAY_PORT`) für Fly-Health-Checks übereinstimmen |
| `memory = "2048mb"`            | 512MB sind zu wenig; 2GB werden empfohlen                                                  |
| `OPENCLAW_STATE_DIR = "/data"` | Speichert den Zustand dauerhaft auf dem Volume                                             |


<div id="3-set-secrets">
  ## 3) Secrets einrichten
</div>

```bash
# Erforderlich: Gateway-Token (für Nicht-Loopback-Bindung)
fly secrets set OPENCLAW_GATEWAY_TOKEN=$(openssl rand -hex 32)

# API-Schlüssel für Modellanbieter
fly secrets set ANTHROPIC_API_KEY=sk-ant-...

# Optional: Weitere Anbieter
fly secrets set OPENAI_API_KEY=sk-...
fly secrets set GOOGLE_API_KEY=...

# Channel-Tokens
fly secrets set DISCORD_BOT_TOKEN=MTQ...
```

**Hinweise:**

* Non-loopback-Binds (`--bind lan`) erfordern aus Sicherheitsgründen `OPENCLAW_GATEWAY_TOKEN`.
* Behandle diese Token wie Passwörter.
* **Verwende Umgebungsvariablen statt der Konfigurationsdatei** für alle API-Schlüssel und Token. So bleiben geheime Werte (Secrets) aus `openclaw.json` heraus, wo sie versehentlich offengelegt oder protokolliert werden könnten.


<div id="4-deploy">
  ## 4) Bereitstellen
</div>

```bash
fly deploy
```

Beim ersten Deployment wird das Docker-Image gebaut (~2–3 Minuten). Nachfolgende Deployments sind schneller.

Überprüfe nach dem Deployment:

```bash
fly status
fly logs
```

Du solltest Folgendes sehen:

```
[gateway] listening on ws://0.0.0.0:3000 (PID xxx)
[discord] logged in to discord as xxx
```


<div id="5-create-config-file">
  ## 5) Konfigurationsdatei erstellen
</div>

Stelle per SSH eine Verbindung zur Maschine her, um die Konfigurationsdatei anzulegen:

```bash
fly ssh console
```

Lege das Konfigurationsverzeichnis und die Konfigurationsdatei an:

```bash
mkdir -p /data
cat > /data/openclaw.json << 'EOF'
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "anthropic/claude-opus-4-5",
        "fallbacks": ["anthropic/claude-sonnet-4-5", "openai/gpt-4o"]
      },
      "maxConcurrent": 4
    },
    "list": [
      {
        "id": "main",
        "default": true
      }
    ]
  },
  "auth": {
    "profiles": {
      "anthropic:default": { "mode": "token", "provider": "anthropic" },
      "openai:default": { "mode": "token", "provider": "openai" }
    }
  },
  "bindings": [
    {
      "agentId": "main",
      "match": { "channel": "discord" }
    }
  ],
  "channels": {
    "discord": {
      "enabled": true,
      "groupPolicy": "allowlist",
      "guilds": {
        "YOUR_GUILD_ID": {
          "channels": { "general": { "allow": true } },
          "requireMention": false
        }
      }
    }
  },
  "gateway": {
    "mode": "local",
    "bind": "auto"
  },
  "meta": {
    "lastTouchedVersion": "2026.1.29"
  }
}
EOF
```

**Hinweis:** Mit `OPENCLAW_STATE_DIR=/data` lautet der Konfigurationspfad `/data/openclaw.json`.

**Hinweis:** Das Discord-Token kann entweder stammen aus:

* Umgebungsvariable: `DISCORD_BOT_TOKEN` (für Secrets empfohlen)
* Konfigurationsdatei: `channels.discord.token`

Wenn du die Umgebungsvariable verwendest, musst du das Token nicht in die Konfiguration eintragen. Das Gateway liest `DISCORD_BOT_TOKEN` automatisch.

Starte das Gateway neu, um die Änderungen zu übernehmen:

```bash
exit
fly machine restart <machine-id>
```


<div id="6-access-the-gateway">
  ## 6) Auf das Gateway zugreifen
</div>

<div id="control-ui">
  ### Control UI
</div>

Im Browser öffnen:

```bash
fly open
```

Oder rufe `https://my-openclaw.fly.dev/` auf.

Gib deinen Gateway-Token (den aus `OPENCLAW_GATEWAY_TOKEN`) ein, um dich zu authentifizieren.


<div id="logs">
  ### Logs
</div>

```bash
fly logs              # Live-Logs
fly logs --no-tail    # Letzte Logs
```


<div id="ssh-console">
  ### SSH-Konsole
</div>

```bash
fly ssh console
```


<div id="troubleshooting">
  ## Fehlerbehebung
</div>

<div id="app-is-not-listening-on-expected-address">
  ### „App hört nicht auf der erwarteten Adresse“
</div>

Das Gateway bindet an `127.0.0.1` statt auf `0.0.0.0`.

**Lösung:** Füge `--bind lan` zu deinem Prozessbefehl in `fly.toml` hinzu.

<div id="health-checks-failing-connection-refused">
  ### Health-Checks schlagen fehl / Verbindung verweigert
</div>

Fly kann das Gateway auf dem konfigurierten Port nicht erreichen.

**Lösung:** Stelle sicher, dass `internal_port` mit dem Gateway-Port übereinstimmt (setze `--port 3000` oder `OPENCLAW_GATEWAY_PORT=3000`).

<div id="oom-memory-issues">
  ### OOM- / Speicherprobleme
</div>

Container startet ständig neu oder wird beendet. Anzeichen: `SIGABRT`, `v8::internal::Runtime_AllocateInYoungGeneration` oder stille Neustarts.

**Lösung:** Speicher in `fly.toml` erhöhen:

```toml
[[vm]]
  memory = "2048mb"
```

Oder eine bestehende Maschine aktualisieren:

```bash
fly machine update <machine-id> --vm-memory 2048 -y
```

**Hinweis:** 512 MB sind zu wenig. 1 GB kann funktionieren, kann aber unter Last oder bei ausführlichem Logging zu OOM führen. **2 GB werden empfohlen.**


<div id="gateway-lock-issues">
  ### Gateway-Lock-Probleme
</div>

Gateway startet nicht und meldet „already running“-Fehler.

Das passiert, wenn der Container neu startet, aber die PID-Lock-Datei auf dem Volume bestehen bleibt.

**Lösung:** Löschen Sie die Lock-Datei:

```bash
fly ssh console --command "rm -f /data/gateway.*.lock"
fly machine restart <machine-id>
```

Die Lock-Datei befindet sich unter dem Pfad `/data/gateway.*.lock` (nicht in einem Unterverzeichnis).


<div id="config-not-being-read">
  ### Config wird nicht gelesen
</div>

Wenn du `--allow-unconfigured` verwendest, erstellt der Gateway eine minimale Config. Deine benutzerdefinierte Config unter `/data/openclaw.json` sollte beim Neustart eingelesen werden.

Stelle sicher, dass die Config existiert:

```bash
fly ssh console --command "cat /data/openclaw.json"
```


<div id="writing-config-via-ssh">
  ### Konfiguration über SSH schreiben
</div>

Der Befehl `fly ssh console -C` unterstützt keine Shell-Umleitungen. Um eine Konfigurationsdatei zu schreiben:

```bash
# Use echo + tee (pipe from local to remote)
echo '{"your":"config"}' | fly ssh console -C "tee /data/openclaw.json"

# Or use sftp
fly sftp shell
> put /local/path/config.json /data/openclaw.json
```

**Hinweis:** `fly sftp` kann fehlschlagen, wenn die Datei bereits vorhanden ist. Lösche die Datei zuerst:

```bash
fly ssh console --command "rm /data/openclaw.json"
```


<div id="state-not-persisting">
  ### Zustand wird nicht beibehalten
</div>

Wenn du Zugangsdaten oder Sitzungen nach einem Neustart verlierst, wird das State-Verzeichnis in das Container-Dateisystem geschrieben.

**Behebung:** Stelle sicher, dass `OPENCLAW_STATE_DIR=/data` in `fly.toml` gesetzt ist und führe ein erneutes Deployment durch.

<div id="updates">
  ## Updates
</div>

```bash
# Neueste Änderungen pullen
git pull

# Neu deployen
fly deploy

# Health-Status prüfen
fly status
fly logs
```


<div id="updating-machine-command">
  ### Aktualisieren des Machine-Startbefehls
</div>

Wenn Sie den Startbefehl ändern müssen, ohne eine vollständige Neubereitstellung durchzuführen:

```bash
# Maschinen-ID abrufen
fly machines list

# Befehl aktualisieren
fly machine update <machine-id> --command "node dist/index.js gateway --port 3000 --bind lan" -y

# Oder mit Speichererhöhung
fly machine update <machine-id> --vm-memory 2048 --command "node dist/index.js gateway --port 3000 --bind lan" -y
```

**Hinweis:** Nach `fly deploy` kann der Machine-Befehl auf den in `fly.toml` definierten Wert zurückgesetzt werden. Wenn du manuelle Änderungen vorgenommen hast, wende sie nach dem Deployment erneut an.


<div id="private-deployment-hardened">
  ## Private Deployment (gehärtet)
</div>

Standardmäßig weist Fly öffentliche IP-Adressen zu, sodass dein Gateway unter `https://your-app.fly.dev` erreichbar ist. Das ist bequem, bedeutet aber auch, dass deine Deployment-Instanz von Internet-Scannern (Shodan, Censys usw.) gefunden werden kann.

Für eine gehärtete Bereitstellung ohne **jegliche öffentliche Erreichbarkeit** verwendest du das private Template.

<div id="when-to-use-private-deployment">
  ### Wann du Private Deployment verwenden solltest
</div>

- Du führst nur **ausgehende** Aufrufe/Nachrichten aus (keine eingehenden Webhooks)
- Du verwendest **ngrok- oder Tailscale**-Tunnel für alle Webhook-Callbacks
- Du greifst über **SSH, Proxy oder WireGuard** statt über einen Browser auf das Gateway zu
- Du möchtest das Deployment **vor Internet-Scannern verborgen halten**

<div id="setup">
  ### Einrichtung
</div>

Verwende `fly.private.toml` anstelle der Standardkonfiguration:

```bash
# Deployment mit privater Konfiguration
fly deploy -c fly.private.toml
```

Oder ein bestehendes Deployment konvertieren:

```bash
# List current IPs
fly ips list -a my-openclaw

# Release public IPs
fly ips release <public-ipv4> -a my-openclaw
fly ips release <public-ipv6> -a my-openclaw

# Zur privaten Konfiguration wechseln, damit zukünftige Deployments keine öffentlichen IPs neu zuweisen
# ([http_service] entfernen oder mit dem privaten Template deployen)
fly deploy -c fly.private.toml

# Allocate private-only IPv6
fly ips allocate-v6 --private -a my-openclaw
```

Danach sollte `fly ips list` nur noch eine IP-Adresse des Typs `private` anzeigen:

```
VERSION  IP                   TYPE             REGION
v6       fdaa:x:x:x:x::x      private          global
```


<div id="accessing-a-private-deployment">
  ### Zugriff auf ein privates Deployment
</div>

Da es keine öffentliche URL gibt, nutze eine der folgenden Methoden:

**Option 1: Lokaler Proxy (einfachste Variante)**

```bash
# Lokalen Port 3000 zur App weiterleiten
fly proxy 3000:3000 -a my-openclaw

# Dann http://localhost:3000 im Browser öffnen
```

**Option 2: WireGuard VPN**

```bash
# WireGuard-Konfiguration erstellen (einmalig)
fly wireguard create

# In WireGuard-Client importieren, dann über interne IPv6-Adresse zugreifen
# Beispiel: http://[fdaa:x:x:x:x::x]:3000
```

**Option 3: Nur SSH-Zugriff**

```bash
fly ssh console -a my-openclaw
```


<div id="webhooks-with-private-deployment">
  ### Webhooks mit privatem Deployment
</div>

Wenn du Webhook-Callbacks (Twilio, Telnyx, etc.) ohne öffentliche Erreichbarkeit benötigst:

1. **ngrok-Tunnel** – Führe ngrok im Container oder als Sidecar-Container aus
2. **Tailscale Funnel** – Stelle bestimmte Pfade über Tailscale bereit
3. **Nur ausgehend** – Manche Anbieter (Twilio) funktionieren für ausgehende Anrufe problemlos ohne Webhooks

Beispielkonfiguration für Sprachanrufe mit ngrok:

```json
{
  "plugins": {
    "entries": {
      "voice-call": {
        "enabled": true,
        "config": {
          "provider": "twilio",
          "tunnel": { "provider": "ngrok" }
        }
      }
    }
  }
}
```

Der ngrok-Tunnel läuft innerhalb des Containers und stellt eine öffentliche Webhook-URL bereit, ohne dabei die eigentliche Fly-App nach außen freizugeben.


<div id="security-benefits">
  ### Sicherheitsvorteile
</div>

| Aspekt | Öffentlich | Privat |
|--------|-----------|--------|
| Internet-Scanner | Erkennbar | Verborgen |
| Direkte Angriffe | Möglich | Unterbunden |
| Control UI-Zugriff | Browser | Proxy/VPN |
| Webhook-Zustellung | Direkt | Über Tunnelverbindung |

<div id="notes">
  ## Hinweise
</div>

- Fly.io verwendet **x86-Architektur** (nicht ARM)
- Das Dockerfile ist mit beiden Architekturen kompatibel
- Für das WhatsApp/Telegram-Onboarding verwende `fly ssh console`
- Persistente Daten liegen auf dem Volume unter `/data`
- Signal benötigt Java + signal-cli; verwende ein benutzerdefiniertes Image und stelle mindestens 2&nbsp;GB Speicher ein.

<div id="cost">
  ## Kosten
</div>

Mit der empfohlenen Konfiguration (`shared-cpu-2x`, 2GB RAM):

- Ca. 10–15 $/Monat, abhängig von der Nutzung
- Der kostenlose Tarif enthält ein gewisses Kontingent

Details findest du in der [Fly.io-Preisübersicht](https://fly.io/docs/about/pricing/).