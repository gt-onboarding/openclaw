---
title: Modelle
summary: "Models-CLI: Auflisten, Setzen, Aliase, Fallbacks, Scans, Status"
read_when:
  - Hinzufügen oder Ändern von Models-CLI-Funktionen (models list/set/scan/aliases/fallbacks)
  - Ändern des Fallback-Verhaltens von Modellen oder der Auswahl-UX
  - Aktualisieren der Scan-Probes für Modelle (tools/images)
---

<div id="models-cli">
  # Models-CLI
</div>

Siehe [/concepts/model-failover](/de/concepts/model-failover) für Informationen zur Rotation von Auth-Profilen, zu Cooldown-Phasen und dazu, wie diese mit Fallbacks zusammenhängen.
Kurzer Überblick über Anbieter + Beispiele: [/concepts/model-providers](/de/concepts/model-providers).

<div id="how-model-selection-works">
  ## Wie die Modellauswahl funktioniert
</div>

OpenClaw wählt Modelle in dieser Reihenfolge:

1. **Primäres** Modell (`agents.defaults.model.primary` oder `agents.defaults.model`).
2. **Fallbacks** in `agents.defaults.model.fallbacks` (in dieser Reihenfolge).
3. **Provider-Auth-Failover** findet innerhalb eines Anbieters statt, bevor zum
   nächsten Modell gewechselt wird.

Verwandt:

* `agents.defaults.models` ist die Allowlist bzw. der Katalog der Modelle, die OpenClaw verwenden kann (plus Aliasse).
* `agents.defaults.imageModel` wird **nur dann** verwendet, wenn das primäre Modell keine Bilder akzeptieren kann.
* Agent-spezifische Standardwerte können `agents.defaults.model` über `agents.list[].model` und Bindings überschreiben (siehe [/concepts/multi-agent](/de/concepts/multi-agent)).

<div id="quick-model-picks-anecdotal">
  ## Schnelle Modelltipps (anekdotisch)
</div>

* **GLM**: etwas besser für Code-/Tool-Aufrufe.
* **MiniMax**: besser fürs Schreiben und die allgemeine Stimmung.

<div id="setup-wizard-recommended">
  ## Einrichtungsassistent (empfohlen)
</div>

Wenn du die Konfiguration nicht manuell anpassen möchtest, führe den Onboarding-Assistenten aus:

```bash
openclaw onboard
```

Es kann Modell- und Authentifizierungs-Setup für gängige Anbieter einrichten, einschließlich **OpenAI Code (Codex)
Abonnement** (OAuth) und **Anthropic** (API-Schlüssel empfohlen; `claude
setup-token` wird ebenfalls unterstützt).

<div id="config-keys-overview">
  ## Config-Keys (Übersicht)
</div>

* `agents.defaults.model.primary` und `agents.defaults.model.fallbacks`
* `agents.defaults.imageModel.primary` und `agents.defaults.imageModel.fallbacks`
* `agents.defaults.models` (Allowlist + Aliase + anbieter-Parameter)
* `models.providers` (benutzerdefinierte anbieter, die in `models.json` eingetragen werden)

Modellreferenzen werden in Kleinbuchstaben normalisiert. Anbieter-Aliase wie `z.ai/*` werden zu `zai/*` normalisiert.

Beispiele für anbieter-Konfigurationen (einschließlich OpenCode Zen) findest du unter
[/gateway/configuration](/de/gateway/configuration#opencode-zen-multi-model-proxy).

<div id="model-is-not-allowed-and-why-replies-stop">
  ## „Modell ist nicht erlaubt“ (und warum Antworten ausbleiben)
</div>

Wenn `agents.defaults.models` gesetzt ist, dient es als **Allowlist** für `/model` und für
Sitzungs-Overrides. Wenn ein Nutzer ein Modell auswählt, das nicht in dieser Allowlist enthalten ist,
gibt OpenClaw Folgendes zurück:

```
Modell „anbieter/modell" ist nicht zulässig. Verwenden Sie /model, um verfügbare Modelle aufzulisten.
```

Dies geschieht **bevor** eine normale Antwort generiert wird, sodass es sich so
anfühlen kann, als wäre nicht geantwortet worden. Die Lösung ist entweder:

* Füge das Modell zu `agents.defaults.models` hinzu, oder
* Leere die Allowlist (entferne `agents.defaults.models`), oder
* Wähle ein Modell aus `/model list`.

Beispielkonfiguration für die Allowlist:

```json5
{
  agent: {
    model: { primary: "anthropic/claude-sonnet-4-5" },
    models: {
      "anthropic/claude-sonnet-4-5": { alias: "Sonnet" },
      "anthropic/claude-opus-4-5": { alias: "Opus" }
    }
  }
}
```

<div id="switching-models-in-chat-model">
  ## Modelle im Chat wechseln (`/model`)
</div>

Du kannst das Modell für die aktuelle Sitzung ändern, ohne einen Neustart durchzuführen:

```
/model
/model list
/model 3
/model openai/gpt-5.2
/model status
```

Hinweise:

* `/model` (und `/model list`) ist eine kompakte, nummerierte Auswahl (Modellfamilie + verfügbare Anbieter).
* `/model <#>` wählt aus dieser Auswahl.
* `/model status` ist die Detailansicht (Auth-Kandidaten und – falls konfiguriert – Anbieter-Endpunkt `baseUrl` + `api`-Modus).
* Modell-Refs werden geparst, indem an der **ersten** `/` getrennt wird. Verwende `provider/model`, wenn du `/model <ref>` eingibst.
* Wenn die Modell-ID selbst `/` enthält (OpenRouter-Stil), musst du das Anbieter-Präfix angeben (Beispiel: `/model openrouter/moonshotai/kimi-k2`).
* Wenn du den Anbieter weglässt, behandelt OpenClaw die Eingabe als Alias oder Modell für den **Standardanbieter** (funktioniert nur, wenn kein `/` in der Modell-ID vorkommt).

Vollständiges Befehlsverhalten und Konfiguration: [Slash commands](/de/tools/slash-commands).

<div id="cli-commands">
  ## CLI-Befehle
</div>

```bash
openclaw models list
openclaw models status
openclaw models set <anbieter/model>
openclaw models set-image <anbieter/model>

openclaw models aliases list
openclaw models aliases add <alias> <anbieter/model>
openclaw models aliases remove <alias>

openclaw models fallbacks list
openclaw models fallbacks add <anbieter/model>
openclaw models fallbacks remove <anbieter/model>
openclaw models fallbacks clear

openclaw models image-fallbacks list
openclaw models image-fallbacks add <anbieter/model>
openclaw models image-fallbacks remove <anbieter/model>
openclaw models image-fallbacks clear
```

`openclaw models` (ohne Unterbefehl) ist eine Kurzform von `models status`.

<div id="models-list">
  ### `models list`
</div>

Zeigt standardmäßig die konfigurierten Modelle an. Nützliche Flags:

* `--all`: vollständiger Katalog
* `--local`: nur lokale Anbieter
* `--provider <name>`: nach Anbieter filtern
* `--plain`: ein Modell pro Zeile
* `--json`: maschinenlesbare Ausgabe

<div id="models-status">
  ### `models status`
</div>

Zeigt das aufgelöste primäre Modell, Fallbacks, das Bildmodell sowie eine Übersicht
über die Authentifizierung der konfigurierten anbieter. Außerdem wird der OAuth-Ablaufstatus
für Profile im Auth-Store angezeigt (warnt standardmäßig innerhalb von 24 Stunden).
`--plain` gibt nur das aufgelöste primäre Modell aus.
Der OAuth‑Status wird immer angezeigt (und ist in der `--json`‑Ausgabe enthalten). Wenn ein
konfigurierter anbieter keine Zugangsdaten hat, gibt `models status` einen Abschnitt
**Missing auth** aus.
JSON umfasst `auth.oauth` (Warnfenster + Profile) und `auth.providers`
(effektive Auth pro anbieter).
Verwende `--check` für Automatisierung (Exit-Code `1` bei fehlender/abgelaufener Auth, `2` bei bald ablaufender Auth).

Die bevorzugte Anthropic-Authentifizierung erfolgt über das Claude Code CLI setup-token (überall ausführbar; bei Bedarf auf dem Gateway-Host einfügen):

```bash
claude setup-token
openclaw models status
```

<div id="scanning-openrouter-free-models">
  ## Scannen (kostenlose OpenRouter-Modelle)
</div>

`openclaw models scan` untersucht OpenRouters **Katalog kostenloser Modelle**
und kann optional Modelle auf Tool- und Bildunterstützung prüfen.

Wichtige Flags:

* `--no-probe`: Live-Tests überspringen (nur Metadaten)
* `--min-params <b>`: minimale Parametergröße (in Milliarden)
* `--max-age-days <days>`: ältere Modelle überspringen
* `--provider <name>`: Anbieterpräfix-Filter
* `--max-candidates <n>`: Größe der Fallback-Liste
* `--set-default`: setzt `agents.defaults.model.primary` auf die erste Auswahl
* `--set-image`: setzt `agents.defaults.imageModel.primary` auf die erste Bildauswahl

Probing erfordert einen OpenRouter-API-Schlüssel (aus Auth-Profilen oder
`OPENROUTER_API_KEY`). Ohne Schlüssel verwende `--no-probe`, um nur Kandidaten
aufzulisten.

Scan-Ergebnisse werden nach folgenden Kriterien gerankt:

1. Bildunterstützung
2. Tool-Latenz
3. Kontextgröße
4. Parameteranzahl

Eingaben

* OpenRouter-`/models`-Liste (Filter `:free`)
* Erfordert OpenRouter-API-Schlüssel aus Auth-Profilen oder `OPENROUTER_API_KEY` (siehe [/environment](/de/environment))
* Optionale Filter: `--max-age-days`, `--min-params`, `--provider`, `--max-candidates`
* Steuerung des Probing: `--timeout`, `--concurrency`

Wenn du es in einem TTY ausführst, kannst du Fallbacks interaktiv auswählen. Im
nichtinteraktiven Modus übergib `--yes`, um Standardwerte zu akzeptieren.

<div id="models-registry-modelsjson">
  ## Models-Registry (`models.json`)
</div>

Benutzerdefinierte Anbieter in `models.providers` werden in `models.json` im
Agent-Verzeichnis (Standard: `~/.openclaw/agents/<agentId>/models.json`) gespeichert. Diese Datei
wird standardmäßig zusammengeführt, es sei denn, `models.mode` ist auf `replace` gesetzt.