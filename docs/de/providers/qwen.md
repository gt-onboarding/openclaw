---
title: Qwen
summary: "Qwen-OAuth (Free-Tier) in OpenClaw nutzen"
read_when:
  - Du möchtest Qwen mit OpenClaw verwenden
  - Du möchtest kostenlosen OAuth-Zugriff im Free-Tier auf Qwen Coder
---

<div id="qwen">
  # Qwen
</div>

Qwen stellt einen kostenlosen OAuth-Flow (Free Tier) für Qwen Coder- und Qwen Vision-Modelle bereit
(2.000 Requests pro Tag, vorbehaltlich der Qwen-Rate-Limits).

<div id="enable-the-plugin">
  ## Plugin aktivieren
</div>

```bash
openclaw plugins enable qwen-portal-auth
```

Starte das Gateway nach der Aktivierung neu.

<div id="authenticate">
  ## Authentifizierung
</div>

```bash
openclaw models auth login --provider qwen-portal --set-default
```

Dies führt den OAuth-Gerätecode-Flow für Qwen aus und schreibt einen Anbietereintrag in deine
`models.json` (plus ein `qwen`-Alias für schnelles Umschalten).

<div id="model-ids">
  ## Modell-IDs
</div>

* `qwen-portal/coder-model`
* `qwen-portal/vision-model`

Wechsle das Modell mit:

```bash
openclaw models set qwen-portal/coder-model
```

<div id="reuse-qwen-code-cli-login">
  ## Qwen Code CLI-Login wiederverwenden
</div>

Wenn du dich bereits mit der Qwen Code CLI angemeldet hast, synchronisiert OpenClaw die Anmeldedaten
aus `~/.qwen/oauth_creds.json`, wenn der Auth-Store geladen wird. Du benötigst trotzdem einen
`models.providers.qwen-portal`-Eintrag (verwende den obigen Login-Befehl, um einen zu erstellen).

<div id="notes">
  ## Hinweise
</div>

* Token werden automatisch aktualisiert; führe den Login-Befehl erneut aus, wenn die Aktualisierung fehlschlägt oder der Zugriff widerrufen wurde.
* Standardbasis-URL: `https://portal.qwen.ai/v1` (überschreibe sie mit
  `models.providers.qwen-portal.baseUrl`, wenn Qwen einen anderen Endpoint bereitstellt).
* Siehe [Modellanbieter](/de/concepts/model-providers) für anbieterweite Regeln.