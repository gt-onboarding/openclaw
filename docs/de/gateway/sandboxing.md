---
summary: "Funktionsweise des OpenClaw-Sandboxing: Modi, Scopes, Arbeitsbereichszugriff und Container-Images"
title: Sandboxing
read_when: "Wenn du eine ausführliche Erklärung zum Sandboxing brauchst oder agents.defaults.sandbox anpassen musst."
status: active
---

<div id="sandboxing">
  # Sandboxing
</div>

OpenClaw kann **Tools in Docker-Containern ausführen**, um den Schadensumfang zu verringern.
Dies ist **optional** und wird über die Konfiguration (`agents.defaults.sandbox` oder
`agents.list[].sandbox`) gesteuert. Wenn Sandboxing deaktiviert ist, laufen Tools auf dem Host.
Das Gateway bleibt auf dem Host; die Toolausführung läuft in einer isolierten Sandbox,
wenn Sandboxing aktiviert ist.

Dies ist keine perfekte Sicherheitsgrenze, beschränkt aber den Zugriff auf Dateisystem und Prozesse
spürbar, wenn das Modell etwas Dummes anstellt.

<div id="what-gets-sandboxed">
  ## Was in der sandbox ausgeführt wird
</div>

* Tool-Ausführung (`exec`, `read`, `write`, `edit`, `apply_patch`, `process` etc.).
* Optionaler Sandbox-Browser (`agents.defaults.sandbox.browser`).
  * Standardmäßig startet der Sandbox-Browser automatisch (stellt sicher, dass CDP erreichbar ist), wenn das Browser-Tool ihn benötigt.
    Dies wird über `agents.defaults.sandbox.browser.autoStart` und `agents.defaults.sandbox.browser.autoStartTimeoutMs` konfiguriert.
  * `agents.defaults.sandbox.browser.allowHostControl` ermöglicht es Sandbox-Sitzungen, den Host-Browser explizit anzusteuern.
  * Optionale Allowlists begrenzen `target: "custom"`: `allowedControlUrls`, `allowedControlHosts`, `allowedControlPorts`.

Nicht in der sandbox:

* Der Gateway-Prozess selbst.
* Jedes Tool, das explizit für die Ausführung auf dem Host zugelassen ist (z. B. `tools.elevated`).
  * **Elevated exec läuft auf dem Host und umgeht die sandbox.**
  * Wenn sandboxing ausgeschaltet ist, ändert `tools.elevated` nichts an der Ausführung (läuft bereits auf dem Host). Siehe [Elevated Mode](/de/tools/elevated).

<div id="modes">
  ## Modi
</div>

`agents.defaults.sandbox.mode` steuert, **wann** die sandbox verwendet wird:

* `"off"`: keine sandbox.
* `"non-main"`: nur **non-main**-Sitzungen laufen in einer sandbox (Standard, wenn du normale Chats direkt auf dem Host haben möchtest).
* `"all"`: jede Sitzung läuft in einer sandbox.
  Hinweis: `"non-main"` basiert auf `session.mainKey` (Standard `"main"`), nicht auf der Agent-ID.
  Gruppen-/Kanal-Sitzungen verwenden ihre eigenen Schlüssel, daher gelten sie als non-main und werden in einer sandbox ausgeführt.

<div id="scope">
  ## Scope
</div>

`agents.defaults.sandbox.scope` steuert, **wie viele Container** erstellt werden:

* `"session"` (Standard): ein Container pro Sitzung.
* `"agent"`: ein Container pro Agent.
* `"shared"`: ein Container, der von allen in der sandbox ausgeführten Sitzungen gemeinsam genutzt wird.

<div id="workspace-access">
  ## Zugriff auf den Arbeitsbereich
</div>

`agents.defaults.sandbox.workspaceAccess` legt fest, **was die sandbox sehen kann**:

* `"none"` (Standard): Tools sehen einen sandbox-Arbeitsbereich unter `~/.openclaw/sandboxes`.
* `"ro"`: bindet den agent-Arbeitsbereich schreibgeschützt bei `/agent` ein (deaktiviert `write`/`edit`/`apply_patch`).
* `"rw"`: bindet den agent-Arbeitsbereich mit Lese-/Schreibzugriff bei `/workspace` ein.

Eingehende Medien werden in den aktiven sandbox-Arbeitsbereich kopiert (`media/inbound/*`).
Hinweis zu Fähigkeiten: Das `read`-Tool ist im sandbox-Root verankert. Mit `workspaceAccess: "none"`
spiegelt OpenClaw berechtigte Fähigkeiten in den sandbox-Arbeitsbereich (`.../skills`), sodass
sie gelesen werden können. Mit `"rw"` sind Fähigkeiten aus dem Arbeitsbereich unter
`/workspace/skills` lesbar.

<div id="custom-bind-mounts">
  ## Benutzerdefinierte Bind-Mounts
</div>

`agents.defaults.sandbox.docker.binds` bindet zusätzliche Host-Verzeichnisse in den Container ein.
Format: `host:container:mode` (z. B. `"/home/user/source:/source:rw"`).

Globale und agent-spezifische Binds werden **zusammengeführt** (nicht ersetzt). Unter `scope: "shared"` werden agent-spezifische Binds ignoriert.

Beispiel (schreibgeschütztes Quellverzeichnis + Docker-Socket):

```json5
{
  agents: {
    defaults: {
      sandbox: {
        docker: {
          binds: [
            "/home/user/source:/source:ro",
            "/var/run/docker.sock:/var/run/docker.sock"
          ]
        }
      }
    },
    list: [
      {
        id: "build",
        sandbox: {
          docker: {
            binds: ["/mnt/cache:/cache:rw"]
          }
        }
      }
    ]
  }
}
```

Sicherheitshinweise:

* Binds umgehen das Dateisystem der Sandbox: Sie binden Host-Pfade im von dir gesetzten Modus (`:ro` oder `:rw`) ein.
* Empfindliche Mounts (z. B. `docker.sock`, Secrets, SSH-Schlüssel) sollten `:ro` sein, es sei denn, Schreibzugriff ist absolut erforderlich.
* Kombiniere dies mit `workspaceAccess: "ro"`, wenn du nur Lesezugriff auf den Arbeitsbereich brauchst; Bind-Modi bleiben davon unabhängig.
* Siehe [Sandbox vs Tool Policy vs Elevated](/de/gateway/sandbox-vs-tool-policy-vs-elevated) für Details dazu, wie Binds mit Tool-Policy und Elevated-Exec interagieren.

<div id="images-setup">
  ## Images + Setup
</div>

Standard-Image: `openclaw-sandbox:bookworm-slim`

Einmalig bauen:

```bash
scripts/sandbox-setup.sh
```

Hinweis: Das Standard-Image enthält **kein** Node.js. Wenn ein Skill Node.js (oder
andere Runtimes) benötigt, musst du entweder ein benutzerdefiniertes Image erstellen oder es über
`sandbox.docker.setupCommand` installieren (erfordert ausgehenden Netzwerkzugriff + beschreibbares Root-Dateisystem +
Root-User).

Sandboxed-Browser-Image:

```bash
scripts/sandbox-browser-setup.sh
```

Standardmäßig werden sandbox-Container **ohne Netzwerk** ausgeführt.
Du kannst dies mit `agents.defaults.sandbox.docker.network` überschreiben.

Docker-Installationen und das containerisierte Gateway findest du hier:
[Docker](/de/install/docker)

<div id="setupcommand-one-time-container-setup">
  ## setupCommand (einmalige Container-Einrichtung)
</div>

`setupCommand` wird **einmal** ausgeführt, nachdem der Sandbox-Container erstellt wurde (nicht bei jedem Lauf).
Es wird im Container über `sh -lc` ausgeführt.

Pfade:

* Global: `agents.defaults.sandbox.docker.setupCommand`
* Pro Agent: `agents.list[].sandbox.docker.setupCommand`

Häufige Fallstricke:

* Standardmäßig ist `docker.network` auf `"none"` gesetzt (kein ausgehender Netzwerkverkehr), daher schlagen Paketinstallationen fehl.
* `readOnlyRoot: true` verhindert Schreibzugriffe; setze `readOnlyRoot: false` oder erstelle ein eigenes Image.
* `user` muss für Paketinstallationen root sein (`user` weglassen oder `user: "0:0"` setzen).
* Sandbox-Ausführungen erben **nicht** die Host-`process.env`. Verwende
  `agents.defaults.sandbox.docker.env` (oder ein eigenes Image) für Skill-API-Schlüssel.

<div id="tool-policy-escape-hatches">
  ## Tool-Policy + Escape-Hatches
</div>

Tool-Allow/Deny-Policies gelten weiterhin vor Sandbox-Regeln. Wenn ein Tool
global oder pro agent verweigert wird, ändert auch die Sandbox daran nichts.

`tools.elevated` ist ein expliziter Escape-Hatch, der `exec` auf dem Host ausführt.
`/exec`-Direktiven gelten nur für autorisierte Absender und bleiben pro Sitzung bestehen; um
`exec` hart zu deaktivieren, verwende deny in der Tool-Policy (siehe [Sandbox vs Tool Policy vs Elevated](/de/gateway/sandbox-vs-tool-policy-vs-elevated)).

Debugging:

* Verwende `openclaw sandbox explain`, um den effektiven Sandbox-Modus, die Tool-Policy und Fix-it-Config-Keys zu inspizieren.
* Siehe [Sandbox vs Tool Policy vs Elevated](/de/gateway/sandbox-vs-tool-policy-vs-elevated) für das mentale Modell „Warum ist das blockiert?“.
  Halte die Umgebung streng abgeschottet.

<div id="multi-agent-overrides">
  ## Multi-Agent-Überschreibungen
</div>

Jeder Agent kann Sandbox und Tools überschreiben:
`agents.list[].sandbox` und `agents.list[].tools` (plus `agents.list[].tools.sandbox.tools` für die Sandbox-Tool-Richtlinie).
Siehe [Multi-Agent Sandbox &amp; Tools](/de/multi-agent-sandbox-tools) für die Priorität.

<div id="minimal-enable-example">
  ## Minimalbeispiel zur Aktivierung
</div>

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main",
        scope: "session",
        workspaceAccess: "none"
      }
    }
  }
}
```

<div id="related-docs">
  ## Verwandte Dokumentation
</div>

* [Sandbox-Konfiguration](/de/gateway/configuration#agentsdefaults-sandbox)
* [Multi-Agent-Sandbox &amp; Tools](/de/multi-agent-sandbox-tools)
* [Sicherheit](/de/gateway/security)