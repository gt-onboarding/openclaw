---
title: Zalouser
summary: "Zalo Personal Plugin: QR-Login + Messaging über zca-cli (Plugin-Installation + Channel-Konfiguration + CLI + Tool)"
read_when:
  - Du möchtest inoffiziellen Support für Zalo Personal in OpenClaw
  - Du konfigurierst oder entwickelst das zalouser-Plugin
---

<div id="zalo-personal-plugin">
  # Zalo Personal (Plugin)
</div>

Unterstützung für Zalo Personal in OpenClaw über ein Plugin, das `zca-cli` verwendet, um ein normales Zalo-Nutzerkonto zu automatisieren.

> **Warnung:** Inoffizielle Automatisierung kann zur (vorübergehenden oder dauerhaften) Sperrung Ihres Kontos führen. Nutzung auf eigenes Risiko.

<div id="naming">
  ## Benennung
</div>

Die Channel-ID ist `zalouser`, um klarzustellen, dass damit ein **persönliches Zalo-Benutzerkonto** (inoffiziell) automatisiert wird. Wir reservieren `zalo` für eine mögliche zukünftige offizielle Zalo-API-Integration.

<div id="where-it-runs">
  ## Wo es läuft
</div>

Dieses Plugin läuft **innerhalb des Gateway-Prozesses**.

Wenn du ein Remote-Gateway verwendest, installiere/konfiguriere es auf der **Maschine, auf der das Gateway läuft**, und starte anschließend das Gateway neu.

<div id="install">
  ## Installation
</div>

<div id="option-a-install-from-npm">
  ### Option A: Installation mit npm
</div>

```bash
openclaw plugins install @openclaw/zalouser
```

Starte das Gateway anschließend neu.


<div id="option-b-install-from-a-local-folder-dev">
  ### Option B: Installation aus einem lokalen Ordner (Dev)
</div>

```bash
openclaw plugins install ./extensions/zalouser
cd ./extensions/zalouser && pnpm install
```

Starte anschließend das Gateway neu.


<div id="prerequisite-zca-cli">
  ## Voraussetzung: zca-cli
</div>

Auf dem Gateway-Host muss `zca` im `PATH` verfügbar sein:

```bash
zca --version
```


<div id="config">
  ## Konfiguration
</div>

Die Channel-Konfiguration befindet sich unter `channels.zalouser` (nicht unter `plugins.entries.*`):

```json5
{
  channels: {
    zalouser: {
      enabled: true,
      dmPolicy: "pairing"
    }
  }
}
```


<div id="cli">
  ## CLI
</div>

```bash
openclaw channels login --channel zalouser
openclaw channels logout --channel zalouser
openclaw channels status --probe
openclaw message send --channel zalouser --target <threadId> --message "Hello from OpenClaw"
openclaw directory peers list --channel zalouser --query "name"
```


<div id="agent-tool">
  ## Agent-Tool
</div>

Toolname: `zalouser`

Aktionen: `send`, `image`, `link`, `friends`, `groups`, `me`, `status`