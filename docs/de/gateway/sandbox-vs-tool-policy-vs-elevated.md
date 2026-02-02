---
title: Sandbox vs Tool-Policy vs Elevated
summary: "Warum ein Tool blockiert wird: sandbox-Laufzeitumgebung, Tool-Allow/Deny-Policy und Elevated-Exec-Gates"
read_when: "Du landest im „sandbox jail“ oder erhältst eine Tool-/Elevated-Verweigerung und willst genau wissen, welchen Konfigurationsschlüssel du ändern musst."
status: active
---

<div id="sandbox-vs-tool-policy-vs-elevated">
  # Sandbox vs Tool Policy vs Elevated
</div>

OpenClaw hat drei verwandte (aber unterschiedliche) Kontrollmechanismen:

1. **Sandbox** (`agents.defaults.sandbox.*` / `agents.list[].sandbox.*`) entscheidet, **wo Tools ausgeführt werden** (Docker vs Host).
2. **Tool policy** (`tools.*`, `tools.sandbox.tools.*`, `agents.list[].tools.*`) entscheidet, **welche Tools verfügbar/erlaubt sind**.
3. **Elevated** (`tools.elevated.*`, `agents.list[].tools.elevated.*`) ist ein **reiner Exec-Escape-Hatch**, um auf dem Host auszuführen, wenn du in einer sandbox läufst.

<div id="quick-debug">
  ## Schnelles Debugging
</div>

Verwende den Inspector, um zu sehen, was OpenClaw *wirklich* tut:

```bash
openclaw sandbox explain
openclaw sandbox explain --session agent:main:main
openclaw sandbox explain --agent work
openclaw sandbox explain --json
```

Es gibt Folgendes aus:

* effektiven Sandbox-Modus/Scope/Arbeitsbereichszugriff
* ob die Sitzung derzeit in einer Sandbox läuft (main vs non-main)
* effektive Allow/Deny-Regeln für Sandbox-Tools (und ob sie von agent/global/default stammen)
* Elevated-Gates und Fix-it-Key-Pfade

<div id="sandbox-where-tools-run">
  ## Sandbox: wo Tools laufen
</div>

Sandboxing wird über `agents.defaults.sandbox.mode` gesteuert:

* `"off"`: alles läuft auf dem Hostsystem.
* `"non-main"`: nur Sitzungen außer der Hauptsitzung werden in der Sandbox ausgeführt (eine häufige „Überraschung“ bei Gruppen/Kanälen).
* `"all"`: alles wird in der Sandbox ausgeführt.

Siehe [Sandboxing](/de/gateway/sandboxing) für die vollständige Matrix (Scope, Arbeitsbereich-Mounts, Images).

<div id="bind-mounts-security-quick-check">
  ### Bind-Mounts (Sicherheits-Kurzcheck)
</div>

* `docker.binds` *durchdringt* das Sandbox-Dateisystem: Alles, was du einbindest, ist im Container mit dem von dir gesetzten Modus (`:ro` oder `:rw`) sichtbar.
* Standard ist Lese-/Schreibzugriff, wenn du den Modus weglässt; verwende `:ro` für Quellcode/Secrets.
* `scope: "shared"` ignoriert agent-spezifische Binds (es gelten nur globale Binds).
* Das Einbinden von `/var/run/docker.sock` übergibt der Sandbox faktisch die Kontrolle über den Host; tu das nur ganz bewusst.
* Der Arbeitsbereichszugriff (`workspaceAccess: "ro"`/`"rw"`) ist unabhängig von den Bind-Modi.

<div id="tool-policy-which-tools-existare-callable">
  ## Toolrichtlinie: welche Tools existieren / aufrufbar sind
</div>

Zwei Ebenen sind relevant:

* **Tool-Profil**: `tools.profile` und `agents.list[].tools.profile` (Basis-Allowlist)
* **Provider-Tool-Profil**: `tools.byProvider[provider].profile` und `agents.list[].tools.byProvider[provider].profile`
* **Globale/per-Agent-Toolrichtlinie**: `tools.allow`/`tools.deny` und `agents.list[].tools.allow`/`agents.list[].tools.deny`
* **Provider-Toolrichtlinie**: `tools.byProvider[provider].allow/deny` und `agents.list[].tools.byProvider[provider].allow/deny`
* **Sandbox-Toolrichtlinie** (gilt nur bei Ausführung in der sandbox): `tools.sandbox.tools.allow`/`tools.sandbox.tools.deny` und `agents.list[].tools.sandbox.tools.*`

Faustregeln:

* `deny` gewinnt immer.
* Wenn `allow` nicht leer ist, wird alles andere als blockiert behandelt.
* Die Toolrichtlinie ist die harte Grenze: `/exec` kann ein geblocktes `exec`-Tool nicht übersteuern.
* `/exec` ändert nur die Sitzungs-Defaults für autorisierte Absender; es gewährt keinen Tool-Zugriff.
  Provider-Tool-Schlüssel akzeptieren entweder `provider` (z. B. `google-antigravity`) oder `provider/model` (z. B. `openai/gpt-5.2`).

<div id="tool-groups-shorthands">
  ### Tool-Gruppen (Kurzformen)
</div>

Tool-Richtlinien (global, agent, sandbox) unterstützen `group:*`-Einträge, die auf mehrere Tools erweitert werden:

```json5
{
  tools: {
    sandbox: {
      tools: {
        allow: ["group:runtime", "group:fs", "group:sessions", "group:memory"]
      }
    }
  }
}
```

Verfügbare Gruppen:

* `group:runtime`: `exec`, `bash`, `process`
* `group:fs`: `read`, `write`, `edit`, `apply_patch`
* `group:sessions`: `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status`
* `group:memory`: `memory_search`, `memory_get`
* `group:ui`: `browser`, `canvas`
* `group:automation`: `cron`, `gateway`
* `group:messaging`: `message`
* `group:nodes`: `nodes`
* `group:openclaw`: alle eingebauten OpenClaw-Tools (ohne Anbieter-Plugins)

<div id="elevated-exec-only-run-on-host">
  ## Elevated: nur exec „auf dem Host ausführen“
</div>

Elevated gewährt **keine** zusätzlichen Tools; es wirkt sich nur auf `exec` aus.

* Wenn du in einer sandbox ausgeführt wirst, wird `/elevated on` (oder `exec` mit `elevated: true`) auf dem Host ausgeführt (Genehmigungen können weiterhin erforderlich sein).
* Verwende `/elevated full`, um exec-Genehmigungen für die Sitzung zu überspringen.
* Wenn du bereits direkt (ohne sandbox) ausführst, hat Elevated de facto keinen Effekt (ist weiterhin durch Gates abgesichert).
* Elevated ist **nicht** auf einen Skill-Umfang beschränkt (nicht skill-scoped) und setzt Tool-Allow/Deny-Regeln nicht außer Kraft.
* `/exec` ist getrennt von Elevated. Es passt nur die per-Sitzung-Standardwerte für exec für autorisierte Absender an.

Gates:

* Aktivierung: `tools.elevated.enabled` (und optional `agents.list[].tools.elevated.enabled`)
* Absender-Allowlists: `tools.elevated.allowFrom.<provider>` (und optional `agents.list[].tools.elevated.allowFrom.<provider>`)

Siehe [Elevated-Modus](/de/tools/elevated).

<div id="common-sandbox-jail-fixes">
  ## Häufige „sandbox-jail“-Probleme und Lösungen
</div>

<div id="tool-x-blocked-by-sandbox-tool-policy">
  ### „Tool X durch sandbox-Tool-Richtlinie blockiert“
</div>

Fix-it-Schlüssel (eine Option auswählen):

* sandbox deaktivieren: `agents.defaults.sandbox.mode=off` (oder pro Agent `agents.list[].sandbox.mode=off`)
* Das Tool innerhalb der sandbox zulassen:
  * es aus `tools.sandbox.tools.deny` entfernen (oder pro Agent `agents.list[].tools.sandbox.tools.deny`)
  * oder zu `tools.sandbox.tools.allow` hinzufügen (oder pro Agent in allow aufnehmen)

<div id="i-thought-this-was-main-why-is-it-sandboxed">
  ### „Ich dachte, das wäre main – warum läuft es in der Sandbox?“
</div>

Im Modus `"non-main"` sind Gruppen-/Kanal-Schlüssel *nicht* main. Verwende den main-Sitzungsschlüssel (angezeigt von `sandbox explain`) oder stelle den Modus auf `"off"` um.