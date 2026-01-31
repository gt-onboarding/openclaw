---
title: Tests
summary: "Test-Kit: Unit-/E2E-/Live-Suites, Docker-Runner und was jeder Test abdeckt"
read_when:
  - Wenn du Tests lokal oder in CI ausführst
  - Wenn du Regressionen für Modell-/anbieter-Bugs hinzufügst
  - Wenn du das Verhalten von Gateway und Agent debuggen möchtest
---

<div id="testing">
  # Tests
</div>

OpenClaw hat drei Vitest-Suites (Unit/Integration, E2E, Live) und eine kleine Auswahl an Docker-Runnern.

Dieses Dokument ist ein „How we test“-Leitfaden:

* Was jede Suite abdeckt (und was sie ganz bewusst *nicht* abdeckt)
* Welche Befehle du für gängige Workflows ausführen kannst (lokal, vor dem Push, Debugging)
* Wie Live-Tests Zugangsdaten ermitteln und Modelle/Anbieter auswählen
* Wie du Regressionstests für reale Probleme mit Modellen/Anbietern hinzufügst

<div id="quick-start">
  ## Schnellstart
</div>

Üblicherweise:

* Vollständiger Gate-Run (wird vor dem Push erwartet): `pnpm lint && pnpm build && pnpm test`

Wenn du Tests änderst oder zusätzliche Sicherheit willst:

* Coverage-Gate: `pnpm test:coverage`
* E2E-Suite: `pnpm test:e2e`

Beim Debuggen von echten Anbietern/Modellen (benötigt echte Zugangsdaten):

* Live-Suite (Modelle + Gateway-Tool-/Image-Tests): `pnpm test:live`

Tipp: Wenn du nur einen einzelnen Fehlerfall brauchst, solltest du die Live-Tests lieber über die unten beschriebenen Allowlist-Umgebungsvariablen eingrenzen.

<div id="test-suites-what-runs-where">
  ## Test-Suites (wo was läuft)
</div>

Stell dir die Test-Suites als Abstufungen mit „zunehmendem Realismus“ (und zunehmender Flakiness/Kosten) vor:

<div id="unit-integration-default">
  ### Unit-/Integrationstests (Standard)
</div>

* Command: `pnpm test`
* Config: `vitest.config.ts`
* Dateien: `src/**/*.test.ts`
* Scope:
  * Reine Unit-Tests
  * In-Process-Integrationstests (Gateway-Auth, Routing, Tooling, Parsing, Config)
  * Deterministische Regressionstests für bekannte Bugs
* Erwartungen:
  * Läuft in CI
  * Keine echten Keys erforderlich
  * Sollte schnell und stabil laufen

<div id="e2e-gateway-smoke">
  ### E2E (Gateway-Schnelltest)
</div>

* Befehl: `pnpm test:e2e`
* Konfiguration: `vitest.e2e.config.ts`
* Dateien: `src/**/*.e2e.test.ts`
* Scope:
  * End-to-End-Verhalten mehrerer Gateway-Instanzen
  * WebSocket-/HTTP-Schnittstellen, Kopplung von Knoten und aufwändigere Netzwerkkommunikation
* Erwartungen:
  * Läuft in CI (sofern in der Pipeline aktiviert)
  * Keine echten API-Schlüssel erforderlich
  * Mehr bewegliche Teile als Unit-Tests (kann langsamer sein)

<div id="live-real-providers-real-models">
  ### Live (reale Anbieter + reale Modelle)
</div>

* Befehl: `pnpm test:live`
* Konfiguration: `vitest.live.config.ts`
* Dateien: `src/**/*.live.test.ts`
* Standard: Standardmäßig **aktiviert** durch `pnpm test:live` (setzt `OPENCLAW_LIVE_TEST=1`)
* Zweck:
  * „Funktioniert dieser Anbieter/dieses Modell *heute* tatsächlich mit echten Zugangsdaten?“
  * Deckt Formatänderungen beim Anbieter, Besonderheiten beim Tool-Calling, Authentifizierungsprobleme und Rate-Limit-Verhalten ab
* Erwartungen:
  * Absichtlich nicht CI-stabil (reale Netzwerke, reale Anbieter-Policies, Quoten, Ausfälle)
  * Kostet Geld / verbraucht Rate Limits
  * Führe nach Möglichkeit nur eingeschränkte Teilmengen statt „alles“ aus
  * Live-Läufe lesen `~/.profile` ein, um fehlende API-Keys zu übernehmen
  * Anthropic-Key-Rotation: Setze `OPENCLAW_LIVE_ANTHROPIC_KEYS="sk-...,sk-..."` (oder `OPENCLAW_LIVE_ANTHROPIC_KEY=sk-...`) oder mehrere `ANTHROPIC_API_KEY*`-Variablen; Tests versuchen es bei Rate Limits erneut

<div id="which-suite-should-i-run">
  ## Welche Suite sollte ich ausführen?
</div>

Verwende diese Entscheidungshilfe:

* Beim Bearbeiten von Logik/Tests: führe `pnpm test` aus (und `pnpm test:coverage`, wenn du viel geändert hast)
* Bei Änderungen an Gateway-Networking / WS-Protokoll / Kopplung: zusätzlich `pnpm test:e2e` ausführen
* Zum Debuggen von „Mein Bot ist down“ / anbieterspezifischen Fehlern / Tool-Aufrufen: ein gezieltes `pnpm test:live` ausführen

<div id="live-model-smoke-profile-keys">
  ## Live: Model-Smoke-Test (Profilschlüssel)
</div>

Live-Tests sind in zwei Stufen aufgeteilt, damit wir Fehler isolieren können:

* „Direct model“ zeigt uns, ob der Anbieter/das Modell mit dem angegebenen Schlüssel überhaupt antworten kann.
* „Gateway smoke“ zeigt uns, ob die vollständige Gateway+agent-Pipeline für dieses Modell funktioniert (Sitzungen, Verlauf, Tools, Sandbox-Richtlinie usw.).

<div id="layer-1-direct-model-completion-no-gateway">
  ### Layer 1: Direkte Modellvervollständigung (kein Gateway)
</div>

* Test: `src/agents/models.profiles.live.test.ts`
* Ziel:
  * Auflisten der gefundenen Modelle
  * `getApiKeyForModel` verwenden, um Modelle auszuwählen, für die du Credentials hast
  * Eine kleine Completion pro Modell ausführen (und gezielte Regressionstests, wo nötig)
* So aktivierst du sie:
  * `pnpm test:live` (oder `OPENCLAW_LIVE_TEST=1`, wenn du Vitest direkt aufrufst)
* Setze `OPENCLAW_LIVE_MODELS=modern` (oder `all`, Alias für modern), um diese Suite tatsächlich auszuführen; andernfalls wird sie übersprungen, damit `pnpm test:live` auf Gateway-Smoke-Tests fokussiert bleibt
* So wählst du Modelle aus:
  * `OPENCLAW_LIVE_MODELS=modern`, um die moderne Allowlist auszuführen (Opus/Sonnet/Haiku 4.5, GPT-5.x + Codex, Gemini 3, GLM 4.7, MiniMax M2.1, Grok 4)
  * `OPENCLAW_LIVE_MODELS=all` ist ein Alias für die moderne Allowlist
  * oder `OPENCLAW_LIVE_MODELS="openai/gpt-5.2,anthropic/claude-opus-4-5,..."` (kommagetrennte Allowlist)
* So wählst du Anbieter aus:
  * `OPENCLAW_LIVE_PROVIDERS="google,google-antigravity,google-gemini-cli"` (kommagetrennte Allowlist)
* Woher die Schlüssel kommen:
  * Standardmäßig: Profil-Store und Umgebungsvariablen-Fallbacks
  * Setze `OPENCLAW_LIVE_REQUIRE_PROFILE_KEYS=1`, um ausschließlich den **Profil-Store** zu erzwingen
* Warum es das gibt:
  * Trennt „Provider-API ist kaputt / Schlüssel ist ungültig“ von „Gateway-Agent-Pipeline ist kaputt“
  * Enthält kleine, isolierte Regressionstests (Beispiel: OpenAI Responses/Codex Responses Reasoning-Replay + Tool-Call-Flows)

<div id="layer-2-gateway-dev-agent-smoke-what-openclaw-actually-does">
  ### Ebene 2: Gateway + Dev-Agent-Smoketest (was „@openclaw“ tatsächlich macht)
</div>

* Test: `src/gateway/gateway-models.profiles.live.test.ts`
* Ziel:
  * Einen In-Process-Gateway hochfahren
  * Eine `agent:dev:*`-Sitzung erstellen/patchen (Model-Override pro Lauf)
  * Über Modelle mit Keys iterieren und prüfen:
    * „Sinnvolle“ Antwort (keine Tools)
    * Ein echter Tool-Aufruf funktioniert (read-Probe)
    * Optionale zusätzliche Tool-Probes (exec+read-Probe)
    * OpenAI-Regression-Pfade (nur Tool-Call → Follow-up) funktionieren weiterhin
* Probe-Details (damit du Fehler schnell erklären kannst):
  * `read`-Probe: Der Test schreibt eine Nonce-Datei in den arbeitsbereich und bittet den agent, sie mit `read` zu lesen und die Nonce zurückzugeben.
  * `exec+read`-Probe: Der Test bittet den agent, per `exec` eine Nonce in eine temporäre Datei zu schreiben und sie dann mit `read` zurückzulesen.
  * Image-Probe: Der Test hängt ein generiertes PNG (Katze + randomisierter Code) an und erwartet, dass das Modell `cat <CODE>` zurückgibt.
  * Implementierungsreferenz: `src/gateway/gateway-models.profiles.live.test.ts` und `src/gateway/live-image-probe.ts`.
* So aktivierst du das:
  * `pnpm test:live` (oder `OPENCLAW_LIVE_TEST=1`, wenn du Vitest direkt aufrufst)
* So wählst du Modelle aus:
  * Standard: moderne Allowlist (Opus/Sonnet/Haiku 4.5, GPT-5.x + Codex, Gemini 3, GLM 4.7, MiniMax M2.1, Grok 4)
  * `OPENCLAW_LIVE_GATEWAY_MODELS=all` ist ein Alias für die moderne Allowlist
  * Oder setze `OPENCLAW_LIVE_GATEWAY_MODELS="provider/model"` (oder Kommaliste), um einzugrenzen
* So wählst du anbieter aus (vermeide „OpenRouter für alles“):
  * `OPENCLAW_LIVE_GATEWAY_PROVIDERS="google,google-antigravity,google-gemini-cli,openai,anthropic,zai,minimax"` (Komma-Allowlist)
* Tool- und Image-Probes sind in diesem Live-Test immer aktiv:
  * `read`-Probe + `exec+read`-Probe (Tool-Stresstest)
  * Image-Probe läuft, wenn das Modell Bild-Input-Support anbietet
  * Ablauf (auf hoher Ebene):
    * Test erzeugt ein winziges PNG mit „CAT“ + Random-Code (`src/gateway/live-image-probe.ts`)
    * Sendet es über `agent` `attachments: [{ mimeType: "image/png", content: "<base64>" }]`
    * Gateway parst Attachments in `images[]` (`src/gateway/server-methods/agent.ts` + `src/gateway/chat-attachments.ts`)
    * Eingebetteter agent leitet eine multimodale Benutzernachricht an das Modell weiter
    * Assertion: Antwort enthält `cat` + den Code (OCR-Toleranz: kleinere Fehler sind erlaubt)

Tipp: Um zu sehen, was du auf deiner Maschine testen kannst (und die exakten `provider/model`-IDs), führe aus:

```bash
openclaw models list
openclaw models list --json
```

<div id="live-anthropic-setup-token-smoke">
  ## Live: Anthropic setup-token Smoke-Test
</div>

* Test: `src/agents/anthropic.setup-token.live.test.ts`
* Ziel: Verifizieren, dass Claude Code CLI setup-token (oder ein eingefügtes setup-token-Profil) eine Anthropic-Prompt-Anfrage erfolgreich ausführen kann.
* Aktivieren:
  * `pnpm test:live` (oder `OPENCLAW_LIVE_TEST=1`, wenn Vitest direkt aufgerufen wird)
  * `OPENCLAW_LIVE_SETUP_TOKEN=1`
* Token-Quellen (eine auswählen):
  * Profil: `OPENCLAW_LIVE_SETUP_TOKEN_PROFILE=anthropic:setup-token-test`
  * Roh-Token: `OPENCLAW_LIVE_SETUP_TOKEN_VALUE=sk-ant-oat01-...`
* Modell-Override (optional):
  * `OPENCLAW_LIVE_SETUP_TOKEN_MODEL=anthropic/claude-opus-4-5`

Setup-Beispiel:

```bash
openclaw models auth paste-token --provider anthropic --profile-id anthropic:setup-token-test
OPENCLAW_LIVE_SETUP_TOKEN=1 OPENCLAW_LIVE_SETUP_TOKEN_PROFILE=anthropic:setup-token-test pnpm test:live src/agents/anthropic.setup-token.live.test.ts
```

<div id="live-cli-backend-smoke-claude-code-cli-or-other-local-clis">
  ## Live: CLI-Backend-Schnelltest (Claude Code CLI oder andere lokale CLIs)
</div>

* Test: `src/gateway/gateway-cli-backend.live.test.ts`
* Ziel: die Gateway- und Agent-Pipeline mit einem lokalen CLI-Backend validieren, ohne deine Standardkonfiguration anzutasten.
* Aktivieren:
  * `pnpm test:live` (oder `OPENCLAW_LIVE_TEST=1`, wenn du Vitest direkt aufrufst)
  * `OPENCLAW_LIVE_CLI_BACKEND=1`
* Standardwerte:
  * Modell: `claude-cli/claude-sonnet-4-5`
  * Befehl: `claude`
  * Argumente: `["-p","--output-format","json","--dangerously-skip-permissions"]`
* Überschreibungen (optional):
  * `OPENCLAW_LIVE_CLI_BACKEND_MODEL="claude-cli/claude-opus-4-5"`
  * `OPENCLAW_LIVE_CLI_BACKEND_MODEL="codex-cli/gpt-5.2-codex"`
  * `OPENCLAW_LIVE_CLI_BACKEND_COMMAND="/full/path/to/claude"`
  * `OPENCLAW_LIVE_CLI_BACKEND_ARGS='["-p","--output-format","json","--permission-mode","bypassPermissions"]'`
  * `OPENCLAW_LIVE_CLI_BACKEND_CLEAR_ENV='["ANTHROPIC_API_KEY","ANTHROPIC_API_KEY_OLD"]'`
  * `OPENCLAW_LIVE_CLI_BACKEND_IMAGE_PROBE=1`, um einen echten Bildanhang zu senden (Pfade werden in den Prompt injiziert).
  * `OPENCLAW_LIVE_CLI_BACKEND_IMAGE_ARG="--image"`, um Bilddateipfade als CLI-Argumente statt per Prompt-Injektion zu übergeben.
  * `OPENCLAW_LIVE_CLI_BACKEND_IMAGE_MODE="repeat"` (oder `"list"`), um zu steuern, wie Bildargumente übergeben werden, wenn `IMAGE_ARG` gesetzt ist.
  * `OPENCLAW_LIVE_CLI_BACKEND_RESUME_PROBE=1`, um eine zweite Dialogrunde zu senden und den Resume-Flow zu validieren.
* `OPENCLAW_LIVE_CLI_BACKEND_DISABLE_MCP_CONFIG=0`, um die Claude Code CLI MCP-Konfiguration aktiviert zu lassen (standardmäßig wird die MCP-Konfiguration mit einer temporären leeren Datei deaktiviert).

Beispiel:

```bash
OPENCLAW_LIVE_CLI_BACKEND=1 \
  OPENCLAW_LIVE_CLI_BACKEND_MODEL="claude-cli/claude-sonnet-4-5" \
  pnpm test:live src/gateway/gateway-cli-backend.live.test.ts
```

<div id="recommended-live-recipes">
  ### Empfohlene Live-Rezepte
</div>

Schmale, explizite Allowlists sind am schnellsten und am wenigsten fehleranfällig:

* Einzelnes Modell, direkt (kein Gateway):
  * `OPENCLAW_LIVE_MODELS="openai/gpt-5.2" pnpm test:live src/agents/models.profiles.live.test.ts`

* Einzelnes Modell, Gateway-Smoketest:
  * `OPENCLAW_LIVE_GATEWAY_MODELS="openai/gpt-5.2" pnpm test:live src/gateway/gateway-models.profiles.live.test.ts`

* Tool-Aufrufe über mehrere Anbieter hinweg:
  * `OPENCLAW_LIVE_GATEWAY_MODELS="openai/gpt-5.2,anthropic/claude-opus-4-5,google/gemini-3-flash-preview,zai/glm-4.7,minimax/minimax-m2.1" pnpm test:live src/gateway/gateway-models.profiles.live.test.ts`

* Fokus auf Google (Gemini-API-Schlüssel + Antigravity):
  * Gemini (API-Schlüssel): `OPENCLAW_LIVE_GATEWAY_MODELS="google/gemini-3-flash-preview" pnpm test:live src/gateway/gateway-models.profiles.live.test.ts`
  * Antigravity (OAuth): `OPENCLAW_LIVE_GATEWAY_MODELS="google-antigravity/claude-opus-4-5-thinking,google-antigravity/gemini-3-pro-high" pnpm test:live src/gateway/gateway-models.profiles.live.test.ts`

Hinweise:

* `google/...` verwendet die Gemini API (API-Schlüssel).
* `google-antigravity/...` verwendet die Antigravity-OAuth-Bridge (Cloud-Code-Assist-artiger Agent-Endpunkt).
* `google-gemini-cli/...` verwendet die lokale Gemini-CLI auf deinem Rechner (separate Authentifizierung + Eigenheiten im Tooling).
* Gemini API vs Gemini CLI:
  * API: OpenClaw ruft die gehostete Gemini API von Google über HTTP auf (API-Schlüssel-/Profil-Authentifizierung); das ist das, was die meisten Nutzer mit „Gemini“ meinen.
  * CLI: OpenClaw startet ein lokales `gemini`-Binary; es hat seine eigene Authentifizierung und kann sich anders verhalten (Streaming-/Tool-Unterstützung/Versionsabweichungen).

<div id="live-model-matrix-what-we-cover">
  ## Live: Modellmatrix (was wir abdecken)
</div>

Es gibt keine feste „CI-Modellliste“ (Live ist Opt-in), aber dies sind die **empfohlenen** Modelle, die du regelmäßig auf einem Dev-Rechner mit Keys abdecken solltest.

<div id="modern-smoke-set-tool-calling-image">
  ### Moderner Smoke-Test (Tool-Calling + Bild)
</div>

Dies ist der „Common-Models“-Lauf, den wir voraussichtlich stabil halten:

* OpenAI (non-Codex): `openai/gpt-5.2` (optional: `openai/gpt-5.1`)
* OpenAI Codex: `openai-codex/gpt-5.2` (optional: `openai-codex/gpt-5.2-codex`)
* Anthropic: `anthropic/claude-opus-4-5` (oder `anthropic/claude-sonnet-4-5`)
* Google (Gemini API): `google/gemini-3-pro-preview` und `google/gemini-3-flash-preview` (verwende keine älteren Gemini-2.x-Modelle)
* Google (Antigravity): `google-antigravity/claude-opus-4-5-thinking` und `google-antigravity/gemini-3-flash`
* Z.AI (GLM): `zai/glm-4.7`
* MiniMax: `minimax/minimax-m2.1`

Führe den Gateway-Smoke-Test mit Tools + Bild aus:
`OPENCLAW_LIVE_GATEWAY_MODELS="openai/gpt-5.2,openai-codex/gpt-5.2,anthropic/claude-opus-4-5,google/gemini-3-pro-preview,google/gemini-3-flash-preview,google-antigravity/claude-opus-4-5-thinking,google-antigravity/gemini-3-flash,zai/glm-4.7,minimax/minimax-m2.1" pnpm test:live src/gateway/gateway-models.profiles.live.test.ts`

<div id="baseline-tool-calling-read-optional-exec">
  ### Baseline: Tool-Aufrufe (Read + optional Exec)
</div>

Wähle mindestens ein Modell pro Anbieterfamilie:

* OpenAI: `openai/gpt-5.2` (oder `openai/gpt-5-mini`)
* Anthropic: `anthropic/claude-opus-4-5` (oder `anthropic/claude-sonnet-4-5`)
* Google: `google/gemini-3-flash-preview` (oder `google/gemini-3-pro-preview`)
* Z.AI (GLM): `zai/glm-4.7`
* MiniMax: `minimax/minimax-m2.1`

Optionale zusätzliche Abdeckung (nice to have):

* xAI: `xai/grok-4` (oder jeweils die aktuellste verfügbare Version)
* Mistral: `mistral/`… (wähle ein „tools“-fähiges Modell, das du aktiviert hast)
* Cerebras: `cerebras/`… (falls du Zugriff hast)
* LM Studio: `lmstudio/`… (lokal; Tool-Aufrufe hängen vom API-Modus ab)

<div id="vision-image-send-attachment-multimodal-message">
  ### Vision: Bildversand (Anhang → multimodale Nachricht)
</div>

Füge mindestens ein bildfähiges Modell in `OPENCLAW_LIVE_GATEWAY_MODELS` hinzu (Vision-fähige Varianten von Claude, Gemini, OpenAI usw.), um den Bildtest durchzuführen.

<div id="aggregators-alternate-gateways">
  ### Aggregatoren / alternative Gateways
</div>

Wenn du Schlüssel konfiguriert hast, unterstützen wir Tests außerdem über:

* OpenRouter: `openrouter/...` (Hunderte von Modellen; verwende `openclaw models scan`, um Kandidaten mit Tool+Image-Funktionen zu finden)
* OpenCode Zen: `opencode/...` (Authentifizierung über `OPENCODE_API_KEY` / `OPENCODE_ZEN_API_KEY`)

Weitere Anbieter, die du in die Live-Matrix aufnehmen kannst (wenn du Credentials/Konfiguration hast):

* Integriert: `openai`, `openai-codex`, `anthropic`, `google`, `google-vertex`, `google-antigravity`, `google-gemini-cli`, `zai`, `openrouter`, `opencode`, `xai`, `groq`, `cerebras`, `mistral`, `github-copilot`
* Über `models.providers` (benutzerdefinierte Endpunkte): `minimax` (Cloud/API) sowie jeder OpenAI/Anthropic-kompatible Proxy (LM Studio, vLLM, LiteLLM, etc.)

Tipp: Versuche nicht, „alle Modelle“ fest in der Dokumentation zu hinterlegen. Die maßgebliche Liste ist das, was `discoverModels(...)` auf deiner Maschine zurückgibt, plus alle verfügbaren Schlüssel.

<div id="credentials-never-commit">
  ## Zugangsdaten (niemals committen)
</div>

Live-Tests ermitteln Zugangsdaten auf dieselbe Weise wie die CLI. Konsequenzen in der Praxis:

* Wenn die CLI funktioniert, sollten Live-Tests dieselben Keys finden.

* Wenn ein Live-Test „no creds“ meldet, geh beim Debuggen genauso vor, wie du es bei `openclaw models list` bzw. der Modellauswahl tun würdest.

* Profil-Speicher: `~/.openclaw/credentials/` (bevorzugt; das ist in den Tests mit „profile keys“ gemeint)

* Konfiguration: `~/.openclaw/openclaw.json` (oder `OPENCLAW_CONFIG_PATH`)

Wenn du dich auf Env-Keys verlassen willst (z. B. in deiner `~/.profile` exportiert), führe lokale Tests nach `source ~/.profile` aus oder verwende die Docker-Runner unten (sie können `~/.profile` ins Container-Dateisystem einbinden).

<div id="deepgram-live-audio-transcription">
  ## Deepgram Live (Live-Audiotranskription)
</div>

* Test: `src/media-understanding/providers/deepgram/audio.live.test.ts`
* Aktivieren: `DEEPGRAM_API_KEY=... DEEPGRAM_LIVE_TEST=1 pnpm test:live src/media-understanding/providers/deepgram/audio.live.test.ts`

<div id="docker-runners-optional-works-in-linux-checks">
  ## Docker-Runner (optionale „Funktioniert-unter-Linux“-Checks)
</div>

Diese führen `pnpm test:live` innerhalb des Repo-Docker-Images aus, binden dein lokales Config-Verzeichnis und den Arbeitsbereich ein (und lesen `~/.profile` ein, falls eingebunden):

* Direkte Modelle: `pnpm test:docker:live-models` (Skript: `scripts/test-live-models-docker.sh`)
* Gateway + Dev-Agent: `pnpm test:docker:live-gateway` (Skript: `scripts/test-live-gateway-models-docker.sh`)
* Onboarding-Assistent (TTY, vollständiges Scaffolding): `pnpm test:docker:onboard` (Skript: `scripts/e2e/onboard-docker.sh`)
* Gateway-Netzwerk (zwei Container, WS-Auth + Health-Checks): `pnpm test:docker:gateway-network` (Skript: `scripts/e2e/gateway-network-docker.sh`)
* Plugins (Laden benutzerdefinierter Erweiterungen + Registry-Smoke-Test): `pnpm test:docker:plugins` (Skript: `scripts/e2e/plugins-docker.sh`)

Nützliche Umgebungsvariablen:

* `OPENCLAW_CONFIG_DIR=...` (Standard: `~/.openclaw`), eingebunden unter `/home/node/.openclaw`
* `OPENCLAW_WORKSPACE_DIR=...` (Standard: `~/.openclaw/workspace`), eingebunden unter `/home/node/.openclaw/workspace`
* `OPENCLAW_PROFILE_FILE=...` (Standard: `~/.profile`), eingebunden unter `/home/node/.profile` und vor dem Ausführen der Tests eingelesen
* `OPENCLAW_LIVE_GATEWAY_MODELS=...` / `OPENCLAW_LIVE_MODELS=...`, um den Lauf einzugrenzen
* `OPENCLAW_LIVE_REQUIRE_PROFILE_KEYS=1`, um sicherzustellen, dass Anmeldedaten aus dem Profil-Store kommen (nicht aus der Umgebung)

<div id="docs-sanity">
  ## Docs-Sanity-Check
</div>

Führe nach Änderungen an der Dokumentation den Docs-Sanity-Check aus: `pnpm docs:list`.

<div id="offline-regression-ci-safe">
  ## Offline-Regressionen (CI-tauglich)
</div>

Dies sind „echte Pipeline“-Regressionen ohne echte Anbieter:

* Gateway-Tool-Aufrufe (Mock-OpenAI, echter Gateway- + Agent-Loop): `src/gateway/gateway.tool-calling.mock-openai.test.ts`
* Gateway-Wizard (WS `wizard.start`/`wizard.next`, schreibt Konfiguration und erzwingt Authentifizierung): `src/gateway/gateway.wizard.e2e.test.ts`

<div id="agent-reliability-evals-skills">
  ## Agent-Zuverlässigkeits-Evals (Fähigkeiten)
</div>

Wir haben bereits einige CI-sichere Tests, die sich wie „Agent-Zuverlässigkeits-Evals“ verhalten:

* Mock-Tool-Aufrufe über den echten Gateway- + Agent-Loop (`src/gateway/gateway.tool-calling.mock-openai.test.ts`).
* End-to-End-Wizard-Flows, die die Sitzungsverdrahtung und die Auswirkungen der Konfiguration validieren (`src/gateway/gateway.wizard.e2e.test.ts`).

Was für Fähigkeiten noch fehlt (siehe [Skills](/de/tools/skills)):

* **Entscheidungsfindung:** Wenn Fähigkeiten im Prompt aufgelistet sind, wählt der agent die richtige Fähigkeit (oder vermeidet irrelevante)?
* **Compliance:** Liest der agent `SKILL.md` vor der Verwendung und befolgt er die erforderlichen Schritte/Argumente?
* **Workflow-Verträge:** Multi-Turn-Szenarien, die Tool-Reihenfolge, Übernahme der Sitzungshistorie und sandbox-Grenzen sicherstellen.

Zukünftige Evals sollten zunächst deterministisch bleiben:

* Ein Szenario-Runner mit Mock-anbietern, um Tool-Aufrufe + Reihenfolge, Skill-Dateizugriffe und Sitzungsverdrahtung zu prüfen.
* Eine kleine Suite von Fähigkeiten-fokussierten Szenarien (verwenden vs. vermeiden, Gating, Prompt-Injection).
* Optionale Live-Evals (opt-in, per Umgebungsvariablen gesteuert) erst, nachdem die CI-sichere Suite steht.

<div id="adding-regressions-guidance">
  ## Hinzufügen von Regressionen (Leitfaden)
</div>

Wenn du ein Anbieter-/Modellproblem behebst, das im Live-Betrieb entdeckt wurde:

* Füge nach Möglichkeit eine CI-sichere Regression hinzu (Mock/Stub-Anbieter oder Erfassung der exakten Transformation der Request-Struktur)
* Wenn das Problem von Natur aus nur im Live-Betrieb auftritt (Rate-Limits, Auth-Richtlinien), halte den Live-Test so klein und fokussiert wie möglich und mache ihn per Umgebungsvariablen explizit opt-in
* Ziele bevorzugt auf die kleinste Schicht, die den Bug erfasst:
  * Bug bei der Anbieter-Request-Konvertierung bzw. beim Replay → direkter Test auf Model-Ebene
  * Gateway-Sitzungs-/Verlaufs-/Tool-Pipeline-Bug → Gateway-Live-Smoke-Test oder CI-sicherer Gateway-Mock-Test