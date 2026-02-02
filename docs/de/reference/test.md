---
title: Test
summary: "Wie du Tests lokal mit vitest ausführst und wann du die Modi force/coverage verwenden solltest"
read_when:
  - Tests ausführen oder Fehler in Tests beheben
---

<div id="tests">
  # Tests
</div>

* Vollständiges Testkit (Suites, Live, Docker): [Testing](/de/testing)

* `pnpm test:force`: Beendet jeden verbleibenden Gateway-Prozess, der den Standard-Steuerport belegt, und führt dann die vollständige Vitest-Suite mit einem isolierten Gateway-Port aus, damit Servertests nicht mit einer laufenden Instanz kollidieren. Verwende dies, wenn ein vorheriger Gateway-Lauf Port 18789 belegt zurückgelassen hat.

* `pnpm test:coverage`: Führt Vitest mit V8-Coverage aus. Globale Grenzwerte sind 70 % für Zeilen/Branches/Funktionen/Statements. Coverage schließt integrationslastige Einstiegspunkte (CLI-Verkabelung, Gateway/Telegram-Bridges, statischer Webchat-Server) aus, damit der Fokus auf unit-testbarer Logik bleibt.

* `pnpm test:e2e`: Führt Gateway-End-to-End-Smoke-Tests aus (Multi-Instanz-WS/HTTP/Knoten-Kopplung).

* `pnpm test:live`: Führt Live-Tests für Anbieter (minimax/zai) aus. Erfordert API-Schlüssel und `LIVE=1` (oder anbieter­spezifisches `*_LIVE_TEST=1`), damit Tests nicht übersprungen werden.

<div id="model-latency-bench-local-keys">
  ## Modelllatenz-Benchmark (lokale Keys)
</div>

Skript: [`scripts/bench-model.ts`](https://github.com/openclaw/openclaw/blob/main/scripts/bench-model.ts)

Verwendung:

* `source ~/.profile && pnpm tsx scripts/bench-model.ts --runs 10`
* Optionale Umgebungsvariablen: `MINIMAX_API_KEY`, `MINIMAX_BASE_URL`, `MINIMAX_MODEL`, `ANTHROPIC_API_KEY`
* Standard-Prompt: „Reply with a single word: ok. No punctuation or extra text.“

Letzter Lauf (2025-12-31, 20 Durchläufe):

* minimax, Median 1279 ms (min 1114, max 2431)
* opus, Median 2454 ms (min 1224, max 3170)

<div id="onboarding-e2e-docker">
  ## Onboarding E2E (Docker)
</div>

Docker ist optional; dies wird nur für containerisierte Onboarding-Smoke-Tests benötigt.

Vollständiger Cold-Start-Ablauf in einem frischen Linux-Container:

```bash
scripts/e2e/onboard-docker.sh
```

Dieses Skript steuert den interaktiven Assistenten über ein Pseudo-TTY, überprüft die Konfigurations-, Arbeitsbereichs- und Sitzungsdateien, startet anschließend das Gateway und führt `openclaw health` aus.

<div id="qr-import-smoke-docker">
  ## QR-Import Smoke-Test (Docker)
</div>

Stellt sicher, dass `qrcode-terminal` in Docker unter Node 22+ geladen werden kann:

```bash
pnpm test:docker:qr
```
