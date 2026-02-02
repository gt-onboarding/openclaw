---
title: Minimax
summary: "MiniMax M2.1 in OpenClaw verwenden"
read_when:
  - Du möchtest MiniMax-Modelle in OpenClaw nutzen
  - Du benötigst eine Einrichtungsanleitung für MiniMax
---

<div id="minimax">
  # MiniMax
</div>

MiniMax ist ein KI-Unternehmen, das die Modellfamilie **M2/M2.1** entwickelt. Die aktuelle,
auf Programmierung ausgerichtete Version ist **MiniMax M2.1** (23. Dezember 2025), entwickelt für
komplexe Aufgaben in realen Anwendungsszenarien.

Quelle: [MiniMax M2.1 Release Note](https://www.minimax.io/news/minimax-m21)

<div id="model-overview-m21">
  ## Modellübersicht (M2.1)
</div>

MiniMax hebt diese Verbesserungen in M2.1 hervor:

* Stärkere **mehrsprachige Programmierung** (Rust, Java, Go, C++, Kotlin, Objective-C, TS/JS).
* Bessere **Web-/App-Entwicklung** und höhere ästhetische Ausgabequalität (einschließlich nativer Mobile-Apps).
* Verbesserte Verarbeitung von **zusammengesetzten Anweisungen** für Office-ähnliche Workflows, aufbauend auf
  verschachteltem Denken und integrierter Ausführung von Constraints.
* **Kürzere Antworten** mit geringerem Token-Verbrauch und schnelleren Iterationsschleifen.
* Höhere Kompatibilität mit **Tool-/Agent-Frameworks** und besseres Kontextmanagement (Claude Code,
  Droid/Factory AI, Cline, Kilo Code, Roo Code, BlackBox).
* Hochwertigere Ausgaben für **Dialoge und technisches Schreiben**.

<div id="minimax-m21-vs-minimax-m21-lightning">
  ## MiniMax M2.1 vs MiniMax M2.1 Lightning
</div>

* **Geschwindigkeit:** Lightning ist in der MiniMax-Preisdokumentation die „schnelle“ Variante.
* **Kosten:** Die Preisangaben zeigen dieselben Eingabekosten, aber Lightning hat höhere Ausgabekosten.
* **Routing im Coding-Tarif:** Das Lightning-Backend ist im MiniMax-Coding-Tarif nicht direkt verfügbar. MiniMax routet die meisten Requests automatisch zu Lightning, fällt bei Traffic-Spitzen jedoch auf das reguläre M2.1-Backend zurück.

<div id="choose-a-setup">
  ## Setup auswählen
</div>

<div id="minimax-m21-recommended">
  ### MiniMax M2.1 — empfohlen
</div>

**Am besten geeignet für:** gehostetes MiniMax mit Anthropic-kompatibler API.

Konfiguriere über die CLI:

* Führe `openclaw configure` aus
* Wähle **Model/auth**
* Wähle **MiniMax M2.1**

```json5
{
  env: { MINIMAX_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "minimax/MiniMax-M2.1" } } },
  models: {
    mode: "merge",
    providers: {
      minimax: {
        baseUrl: "https://api.minimax.io/anthropic",
        apiKey: "${MINIMAX_API_KEY}",
        api: "anthropic-messages",
        models: [
          {
            id: "MiniMax-M2.1",
            name: "MiniMax M2.1",
            reasoning: false,
            input: ["text"],
            cost: { input: 15, output: 60, cacheRead: 2, cacheWrite: 10 },
            contextWindow: 200000,
            maxTokens: 8192
          }
        ]
      }
    }
  }
}
```

<div id="minimax-m21-as-fallback-opus-primary">
  ### MiniMax M2.1 als Fallback (Opus primär)
</div>

**Am besten geeignet, wenn du:** Opus 4.5 als primäres Modell nutzen und bei Ausfall auf MiniMax M2.1 zurückfallen möchtest.

```json5
{
  env: { MINIMAX_API_KEY: "sk-..." },
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-5": { alias: "opus" },
        "minimax/MiniMax-M2.1": { alias: "minimax" }
      },
      model: {
        primary: "anthropic/claude-opus-4-5",
        fallbacks: ["minimax/MiniMax-M2.1"]
      }
    }
  }
}
```

<div id="optional-local-via-lm-studio-manual">
  ### Optional: Lokal mit LM Studio (manuell)
</div>

**Am besten geeignet für:** lokale Inferenz mit LM Studio.
Wir haben gute Ergebnisse mit MiniMax M2.1 auf leistungsfähiger Hardware beobachtet (z. B. einem
Desktop oder Server) unter Verwendung des lokalen Servers von LM Studio.

Manuell über `openclaw.json` konfigurieren:

```json5
{
  agents: {
    defaults: {
      model: { primary: "lmstudio/minimax-m2.1-gs32" },
      models: { "lmstudio/minimax-m2.1-gs32": { alias: "Minimax" } }
    }
  },
  models: {
    mode: "merge",
    providers: {
      lmstudio: {
        baseUrl: "http://127.0.0.1:1234/v1",
        apiKey: "lmstudio",
        api: "openai-responses",
        models: [
          {
            id: "minimax-m2.1-gs32",
            name: "MiniMax M2.1 GS32",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 196608,
            maxTokens: 8192
          }
        ]
      }
    }
  }
}
```

<div id="configure-via-openclaw-configure">
  ## Über `openclaw configure` konfigurieren
</div>

Verwende den interaktiven Konfigurationsassistenten, um MiniMax einzurichten, ohne JSON manuell zu bearbeiten:

1. Führe den Befehl `openclaw configure` aus.
2. Wähle **Model/auth**.
3. Wähle **MiniMax M2.1**.
4. Wähle dein Standardmodell, wenn du dazu aufgefordert wirst.

<div id="configuration-options">
  ## Konfigurationsoptionen
</div>

* `models.providers.minimax.baseUrl`: verwende bevorzugt `https://api.minimax.io/anthropic` (Anthropic-kompatibel); `https://api.minimax.io/v1` ist optional für OpenAI-kompatible Anfragen.
* `models.providers.minimax.api`: verwende bevorzugt `anthropic-messages`; `openai-completions` ist optional für OpenAI-kompatible Anfragen.
* `models.providers.minimax.apiKey`: MiniMax-API-Schlüssel (`MINIMAX_API_KEY`).
* `models.providers.minimax.models`: definiere `id`, `name`, `reasoning`, `contextWindow`, `maxTokens`, `cost`.
* `agents.defaults.models`: lege Aliase für Modelle fest, die du in der Allowlist haben möchtest.
* `models.mode`: belasse `merge` unverändert, wenn du MiniMax zusätzlich zu den eingebauten Modellen verwenden möchtest.

<div id="notes">
  ## Hinweise
</div>

* Modell-Referenzen haben das Format `minimax/<model>`.
* API zur Abfrage der Coding-Plan-Nutzung: `https://api.minimaxi.com/v1/api/openplatform/coding_plan/remains` (erfordert einen Coding-Plan-Schlüssel).
* Aktualisiere die Preisangaben in `models.json`, wenn du eine exakte Kostenverfolgung benötigst.
* Referral-Link für den MiniMax Coding Plan (10 % Rabatt): https://platform.minimax.io/subscribe/coding-plan?code=DbXJTRClnb&amp;source=link
* Siehe [/concepts/model-providers](/de/concepts/model-providers) für Anbieterregeln.
* Verwende `openclaw models list` und `openclaw models set minimax/MiniMax-M2.1`, um zu wechseln.

<div id="troubleshooting">
  ## Fehlerbehebung
</div>

<div id="unknown-model-minimaxminimax-m21">
  ### „Unknown model: minimax/MiniMax-M2.1“
</div>

Das bedeutet in der Regel, dass der **MiniMax-Anbieter nicht konfiguriert ist** (kein Anbieter-Eintrag
und kein MiniMax-Auth-Profil/Env-Key gefunden). Ein Fix für diese Erkennung ist in
**2026.1.12** enthalten (zum Zeitpunkt des Schreibens noch unveröffentlicht). Behebe das Problem, indem du:

* Auf **2026.1.12** aktualisierst (oder aus dem Quellcode-Branch `main` ausführst) und dann das Gateway neu startest.
* `openclaw configure` ausführst und **MiniMax M2.1** auswählst, oder
* Den Block `models.providers.minimax` manuell hinzufügst, oder
* `MINIMAX_API_KEY` (oder ein MiniMax-Auth-Profil) setzt, damit der Anbieter automatisch eingebunden werden kann.

Achte darauf, dass bei der Modell-ID die **Groß-/Kleinschreibung exakt übereinstimmt**:

* `minimax/MiniMax-M2.1`
* `minimax/MiniMax-M2.1-lightning`

Prüfe anschließend erneut mit:

```bash
openclaw models list
```
