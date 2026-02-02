---
title: Umgebung
summary: "Woher OpenClaw Umgebungsvariablen lädt und in welcher Reihenfolge"
read_when:
  - Du musst wissen, welche Umgebungsvariablen geladen werden und in welcher Reihenfolge
  - Du behebst Fehler bei fehlenden API-Schlüsseln im Gateway
  - Du dokumentierst Anbieter-Authentifizierung oder Bereitstellungsumgebungen
---

<div id="environment-variables">
  # Umgebungsvariablen
</div>

OpenClaw bezieht Umgebungsvariablen aus mehreren Quellen. Die Regel lautet: **vorhandene Werte niemals überschreiben**.

<div id="precedence-highest-lowest">
  ## Rangfolge (höchste → niedrigste)
</div>

1. **Prozessumgebung** (was der Gateway‑Prozess bereits von der übergeordneten Shell bzw. dem Dienst geerbt hat).
2. **`.env` im aktuellen Arbeitsverzeichnis** (dotenv-Standardverhalten; überschreibt nichts).
3. **Globale `.env`** unter `~/.openclaw/.env` (auch `$OPENCLAW_STATE_DIR/.env`; überschreibt nichts).
4. **`env`-Block in der Config** in `~/.openclaw/openclaw.json` (wird nur angewendet, wenn Werte fehlen).
5. **Optionaler Login-Shell-Import** (`env.shellEnv.enabled` oder `OPENCLAW_LOAD_SHELL_ENV=1`), wird nur für fehlende erwartete Keys angewendet.

Wenn die Config-Datei gar nicht vorhanden ist, wird Schritt 4 übersprungen; der Shell-Import läuft dennoch, falls aktiviert.

<div id="config-env-block">
  ## Config-Block `env`
</div>

Zwei gleichwertige Möglichkeiten, Inline-Env-Variablen zu setzen (beide überschreiben bestehende Werte nicht):

```json5
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: {
      GROQ_API_KEY: "gsk-..."
    }
  }
}
```

<div id="shell-env-import">
  ## Import der Shell-Umgebung
</div>

`env.shellEnv` führt deine Login-Shell aus und importiert nur die **fehlenden** erwarteten Schlüssel:

```json5
{
  env: {
    shellEnv: {
      enabled: true,
      timeoutMs: 15000
    }
  }
}
```

Entsprechende Umgebungsvariablen:

* `OPENCLAW_LOAD_SHELL_ENV=1`
* `OPENCLAW_SHELL_ENV_TIMEOUT_MS=15000`

<div id="env-var-substitution-in-config">
  ## Substitution von Umgebungsvariablen in der Konfiguration
</div>

Sie können Umgebungsvariablen direkt in String-Werten der Konfiguration mithilfe der Syntax `${VAR_NAME}` referenzieren:

```json5
{
  models: {
    providers: {
      "vercel-gateway": {
        apiKey: "${VERCEL_GATEWAY_API_KEY}"
      }
    }
  }
}
```

Ausführliche Informationen findest du unter [Konfiguration: Substitution von Umgebungsvariablen](/de/gateway/configuration#env-var-substitution-in-config).

<div id="related">
  ## Verwandte Themen
</div>

* [Gateway-Konfiguration](/de/gateway/configuration)
* [FAQ: Umgebungsvariablen und Laden von .env-Dateien](/de/help/faq#env-vars-and-env-loading)
* [Modellübersicht](/de/concepts/models)