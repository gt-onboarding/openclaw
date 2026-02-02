---
title: Ollama
summary: "OpenClaw mit Ollama ausführen (lokale LLM-Laufzeitumgebung)"
read_when:
  - Sie OpenClaw mit lokalen Modellen über Ollama ausführen möchten
  - Sie Anleitungen zur Einrichtung und Konfiguration von Ollama benötigen
---

<div id="ollama">
  # Ollama
</div>

Ollama ist eine lokale LLM-Laufzeitumgebung, mit der du Open-Source-Modelle einfach auf deinem Rechner ausführen kannst. OpenClaw integriert sich mit der OpenAI-kompatiblen API von Ollama und kann **toolfähige Modelle automatisch erkennen**, wenn du `OLLAMA_API_KEY` (oder ein Auth-Profil) konfigurierst und keinen expliziten Eintrag für `models.providers.ollama` definierst.

<div id="quick-start">
  ## Schnellstart
</div>

1. Installiere Ollama: https://ollama.ai

2. Lade ein Modell herunter:

```bash
ollama pull llama3.3
# oder
ollama pull qwen2.5-coder:32b
# oder
ollama pull deepseek-r1:32b
```

3. Aktiviere Ollama für OpenClaw (ein beliebiger Wert funktioniert; Ollama benötigt keinen echten API-Schlüssel):

```bash
# Umgebungsvariable setzen
export OLLAMA_API_KEY="ollama-local"

# Oder in der Konfigurationsdatei konfigurieren
openclaw config set models.providers.ollama.apiKey "ollama-local"
```

4. Verwende Ollama-Modelle:

```json5
{
  agents: {
    defaults: {
      model: { primary: "ollama/llama3.3" }
    }
  }
}
```

<div id="model-discovery-implicit-provider">
  ## Model-Erkennung (impliziter Anbieter)
</div>

Wenn du `OLLAMA_API_KEY` (oder ein Auth-Profil) setzt und **kein** `models.providers.ollama` definierst, erkennt OpenClaw Modelle aus der lokalen Ollama-Instanz unter `http://127.0.0.1:11434`:

* Ruft `/api/tags` und `/api/show` auf
* Behält nur Modelle, die die Fähigkeit `tools` melden
* Markiert `reasoning`, wenn das Modell `thinking` meldet
* Liest `contextWindow` aus `model_info["<arch>.context_length"]`, falls vorhanden
* Setzt `maxTokens` auf das 10-Fache des Kontextfensters
* Setzt alle Kosten auf `0`

Dadurch entfallen manuelle Modelleinträge, während der Katalog mit den Fähigkeiten von Ollama synchron bleibt.

Um zu sehen, welche Modelle verfügbar sind:

```bash
ollama list
openclaw models list
```

Um ein neues Modell hinzuzufügen, laden Sie es einfach mit Ollama herunter:

```bash
ollama pull mistral
```

Das neue Modell wird automatisch erkannt und steht zur Nutzung zur Verfügung.

Wenn du `models.providers.ollama` explizit setzt, wird die automatische Erkennung übersprungen und du musst Modelle manuell definieren (siehe unten).

<div id="configuration">
  ## Konfiguration
</div>

<div id="basic-setup-implicit-discovery">
  ### Grundlegende Einrichtung (automatische Erkennung)
</div>

Die einfachste Möglichkeit, Ollama zu aktivieren, ist über eine Umgebungsvariable:

```bash
export OLLAMA_API_KEY="ollama-local"
```

<div id="explicit-setup-manual-models">
  ### Explizites Setup (manuelle Modelle)
</div>

Verwende eine explizite Konfiguration, wenn:

* Ollama auf einem anderen Host/Port läuft.
* Du bestimmte Kontextfenster oder Modelllisten erzwingen möchtest.
* Du Modelle einbinden möchtest, die keine Unterstützung für Tools melden.

```json5
{
  models: {
    providers: {
      ollama: {
        // Verwenden Sie einen Host, der /v1 für OpenAI-kompatible APIs enthält
        baseUrl: "http://ollama-host:11434/v1",
        apiKey: "ollama-local",
        api: "openai-completions",
        models: [
          {
            id: "llama3.3",
            name: "Llama 3.3",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 8192,
            maxTokens: 8192 * 10
          }
        ]
      }
    }
  }
}
```

Wenn `OLLAMA_API_KEY` gesetzt ist, kannst du `apiKey` im Anbieter-Eintrag weglassen, und OpenClaw trägt ihn für Verfügbarkeitsprüfungen automatisch ein.

<div id="custom-base-url-explicit-config">
  ### Benutzerdefinierte Basis-URL (explizite Konfiguration)
</div>

Wenn Ollama auf einem anderen Host oder Port läuft (explizite Konfiguration deaktiviert die automatische Erkennung, daher musst du Modelle manuell definieren):

```json5
{
  models: {
    providers: {
      ollama: {
        apiKey: "ollama-local",
        baseUrl: "http://ollama-host:11434/v1"
      }
    }
  }
}
```

<div id="model-selection">
  ### Modellauswahl
</div>

Sobald alles konfiguriert ist, stehen dir alle Ollama-Modelle zur Verfügung:

```json5
{
  agents: {
    defaults: {
      model: {
        primary: "ollama/llama3.3",
        fallback: ["ollama/qwen2.5-coder:32b"]
      }
    }
  }
}
```

<div id="advanced">
  ## Erweitert
</div>

<div id="reasoning-models">
  ### Reasoning-Modelle
</div>

OpenClaw markiert Modelle als reasoning-fähig, wenn Ollama in `/api/show` `thinking` meldet:

```bash
ollama pull deepseek-r1:32b
```

<div id="model-costs">
  ### Modellkosten
</div>

Ollama ist kostenlos und läuft lokal, daher betragen alle Modellkosten 0 $.

<div id="context-windows">
  ### Kontextfenster
</div>

Für automatisch erkannte Modelle verwendet OpenClaw das von Ollama gemeldete Kontextfenster, wenn verfügbar, andernfalls wird standardmäßig `8192` verwendet. Du kannst `contextWindow` und `maxTokens` in einer expliziten Anbieter-Konfiguration überschreiben.

<div id="troubleshooting">
  ## Fehlerbehebung
</div>

<div id="ollama-not-detected">
  ### Ollama nicht erkannt
</div>

Stelle sicher, dass Ollama läuft, dass du `OLLAMA_API_KEY` (oder ein Auth-Profil) gesetzt hast, und dass du **keinen** expliziten `models.providers.ollama`-Eintrag in deiner Konfiguration definiert hast:

```bash
ollama serve
```

Und dass die API erreichbar ist:

```bash
curl http://localhost:11434/api/tags
```

<div id="no-models-available">
  ### Keine Modelle verfügbar
</div>

OpenClaw erkennt nur automatisch Modelle, die Tool-Unterstützung bieten. Wenn dein Modell nicht aufgeführt ist, dann entweder:

* Lade ein Modell mit Tool-Unterstützung, oder
* definiere das Modell explizit in `models.providers.ollama`.

So fügst du Modelle hinzu:

```bash
ollama list  # Zeigt an, was installiert ist
ollama pull llama3.3  # Lädt ein Modell herunter
```

<div id="connection-refused">
  ### Verbindung abgelehnt
</div>

Überprüfen Sie, ob Ollama auf dem richtigen Port läuft:

```bash
# Prüfen, ob Ollama läuft
ps aux | grep ollama

# Oder Ollama neu starten
ollama serve
```

<div id="see-also">
  ## Siehe auch
</div>

* [Modell-Anbieter](/de/concepts/model-providers) - Übersicht über alle Anbieter
* [Modellauswahl](/de/concepts/models) - So wählst du Modelle aus
* [Konfiguration](/de/gateway/configuration) - Vollständige Referenz zur Konfiguration