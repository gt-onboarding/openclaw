---
title: Konfiguration
summary: "CLI-Referenz für `openclaw config` (Konfigurationswerte abrufen/setzen/zurücksetzen)"
read_when:
  - Du Konfigurationseinstellungen nicht interaktiv lesen oder bearbeiten möchtest
---

<div id="openclaw-config">
  # `openclaw config`
</div>

Konfigurations-Helfer: Werte anhand eines Pfads abrufen/setzen/zurücksetzen. Ohne Unterbefehl ausführen, um den Konfigurationsassistenten zu öffnen (entspricht `openclaw configure`).

<div id="examples">
  ## Beispiele
</div>

```bash
openclaw config get browser.executablePath
openclaw config set browser.executablePath "/usr/bin/google-chrome"
openclaw config set agents.defaults.heartbeat.every "2h"
openclaw config set agents.list[0].tools.exec.node "node-id-or-name"
openclaw config unset tools.web.search.apiKey
```


<div id="paths">
  ## Pfade
</div>

Pfade verwenden Punkt- oder Klammernotation:

```bash
openclaw config get agents.defaults.workspace
openclaw config get agents.list[0].id
```

Verwende den Index der Agentenliste, um einen bestimmten agent zu adressieren:

```bash
openclaw config get agents.list
openclaw config set agents.list[1].tools.exec.node "node-id-or-name"
```


<div id="values">
  ## Werte
</div>

Werte werden nach Möglichkeit als JSON5 geparst; andernfalls werden sie als Strings behandelt.
Verwende `--json`, um das JSON5-Parsing zu erzwingen.

```bash
openclaw config set agents.defaults.heartbeat.every "0m"
openclaw config set gateway.port 19001 --json
openclaw config set channels.whatsapp.groups '["*"]' --json
```

Starte das Gateway nach Änderungen neu.
