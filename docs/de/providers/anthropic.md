---
title: Anthropic
summary: "Anthropic Claude mit API-Schlüsseln oder setup-token in OpenClaw verwenden"
read_when:
  - Du möchtest Anthropic-Modelle in OpenClaw verwenden
  - Du möchtest setup-token statt API-Schlüsseln verwenden
---

<div id="anthropic-claude">
  # Anthropic (Claude)
</div>

Anthropic entwickelt die **Claude**-Modellfamilie und bietet Zugriff über eine API.
In OpenClaw kannst du dich mit einem API-Schlüssel oder einem **setup-token** authentifizieren.

<div id="option-a-anthropic-api-key">
  ## Option A: Anthropic API-Schlüssel
</div>

**Am besten geeignet für:** Standard-API-Zugriff und nutzungsabhängige Abrechnung.
Erstellen Sie Ihren API-Schlüssel in der Anthropic Console.

### CLI-Setup

```bash
openclaw onboard
# wähle: Anthropic API key

# or non-interactive
openclaw onboard --anthropic-api-key "$ANTHROPIC_API_KEY"
```

<div id="config-snippet">
  ### Konfigurationsbeispiel
</div>

```json5
{
  env: { ANTHROPIC_API_KEY: "sk-ant-..." },
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-5" } } }
}
```

<div id="prompt-caching-anthropic-api">
  ## Prompt-Caching (Anthropic API)
</div>

OpenClaw überschreibt **nicht** die standardmäßige Cache-TTL von Anthropic, es sei denn, du legst sie explizit fest.
Dies ist **nur für die API**; Abo-Authentifizierung berücksichtigt keine TTL-Einstellungen.

Um die TTL pro Modell festzulegen, verwende `cacheControlTtl` in den Modell-`params`:

```json5
{
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-5": {
          params: { cacheControlTtl: "5m" } // oder „1h"
        }
      }
    }
  }
}
```

OpenClaw enthält das Beta-Flag `extended-cache-ttl-2025-04-11` für Anthropic-API-Anfragen; belasse es darin, falls du Anbieter-Header überschreibst (siehe [/gateway/configuration](/de/gateway/configuration)).

<div id="option-b-claude-setup-token">
  ## Option B: Claude setup-token
</div>

**Am besten geeignet für:** die Nutzung deines Claude-Abonnements.

<div id="where-to-get-a-setup-token">
  ### Wo du ein Setup-Token bekommst
</div>

Setup-Tokens werden von der **Claude Code CLI** erstellt, nicht von der Anthropic Console. Du kannst das auf **jedem beliebigen Rechner** ausführen:

```bash
claude setup-token
```

Füge das Token in OpenClaw ein (Assistent: **Anthropic-Token (setup-token einfügen)**) oder führe den Befehl auf dem Gateway-Host aus:

```bash
openclaw models auth setup-token --provider anthropic
```

Wenn du das Token auf einem anderen Rechner erstellt hast, füge es ein:

```bash
openclaw models auth paste-token --provider anthropic
```

<div id="cli-setup">
  ### CLI-Einrichtung
</div>

```bash
# Setup-Token während des Onboardings einfügen
openclaw onboard --auth-choice setup-token
```

<div id="config-snippet">
  ### Konfigurationsbeispiel
</div>

```json5
{
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-5" } } }
}
```

<div id="notes">
  ## Hinweise
</div>

* Generiere das Setup-Token mit `claude setup-token` und füge es ein oder führe `openclaw models auth setup-token` auf dem Gateway-Host aus.
* Wenn du bei einem Claude‑Abonnement die Meldung „OAuth token refresh failed …“ siehst, führe die Authentifizierung mit einem Setup-Token erneut durch. Siehe [/gateway/troubleshooting#oauth-token-refresh-failed-anthropic-claude-subscription](/de/gateway/troubleshooting#oauth-token-refresh-failed-anthropic-claude-subscription).
* Details zur Authentifizierung und zu Wiederverwendungsregeln findest du unter [/concepts/oauth](/de/concepts/oauth).

<div id="troubleshooting">
  ## Fehlerbehebung
</div>

**401-Fehler / Token plötzlich ungültig**

* Die Authentifizierung deines Claude-Abonnements kann ablaufen oder widerrufen werden. Führe `claude setup-token` erneut aus
  und füge das Token im **Gateway-Host** ein.
* Wenn die Claude-CLI-Anmeldung auf einer anderen Maschine erfolgt ist, verwende
  `openclaw models auth paste-token --provider anthropic` auf dem Gateway-Host.

**No API key found for provider &quot;anthropic&quot;**

* Auth erfolgt **pro Agent**. Neue Agenten erben nicht die Schlüssel des Haupt-Agents.
* Führe das Onboarding für diesen Agent erneut aus oder füge ein `setup-token` bzw. einen API-Schlüssel auf dem
  Gateway-Host ein und überprüfe anschließend mit `openclaw models status`.

**No credentials found for profile `anthropic:default`**

* Führe `openclaw models status` aus, um zu sehen, welches Auth-Profil aktiv ist.
* Führe das Onboarding erneut aus oder füge ein `setup-token` bzw. einen API-Schlüssel für dieses Profil ein.

**No available auth profile (all in cooldown/unavailable)**

* Prüfe `openclaw models status --json` auf `auth.unusableProfiles`.
* Füge ein weiteres Anthropic-Profil hinzu oder warte auf das Ende der Cooldown-Phase.

Mehr: [/gateway/troubleshooting](/de/gateway/troubleshooting) und [/help/faq](/de/help/faq).