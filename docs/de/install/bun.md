---
title: Bun
summary: "Bun-Workflow (experimentell): Installation und Fallstricke vs pnpm"
read_when:
  - Du möchtest den schnellsten lokalen Dev-Loop (bun + watch)
  - Du stößt auf Probleme mit Bun-Install-/Patch-/Lifecycle-Skripten
---

<div id="bun-experimental">
  # Bun (experimentell)
</div>

Ziel: Dieses Repository mit **Bun** ausführen (optional, nicht empfohlen für WhatsApp/Telegram),
ohne von pnpm-Workflows abzuweichen.

⚠️ **Für den Einsatz in der Gateway-Laufzeitumgebung nicht empfohlen** (WhatsApp/Telegram-Bugs). Verwende Node für den Produktivbetrieb.

<div id="status">
  ## Status
</div>

- Bun ist eine optionale lokale Laufzeitumgebung, um TypeScript direkt auszuführen (`bun run …`, `bun --watch …`).
- `pnpm` ist das Standardtool für Builds und wird weiterhin vollständig unterstützt (und von einigen Dokumentations-Tools verwendet).
- Bun kann `pnpm-lock.yaml` nicht verwenden und wird diese Datei ignorieren.

<div id="install">
  ## Installation
</div>

Standardmäßig:

```sh
bun install
```

Hinweis: `bun.lock`/`bun.lockb` stehen in `.gitignore`, daher entstehen so oder so keine Änderungen im Repo. Wenn du *keine Lockfile-Schreibvorgänge* willst:

```sh
bun install --no-save
```


<div id="build-test-bun">
  ## Build / Test (Bun)
</div>

```sh
bun run build
bun run vitest run
```


<div id="bun-lifecycle-scripts-blocked-by-default">
  ## Bun-Lebenszyklus-Skripte (standardmäßig blockiert)
</div>

Bun kann Lebenszyklus-Skripte von Abhängigkeiten blockieren, es sei denn, du vertraust ihnen explizit (`bun pm untrusted` / `bun pm trust`).
Für dieses Repository werden die üblicherweise blockierten Skripte nicht benötigt:

* `@whiskeysockets/baileys` `preinstall`: prüft, ob die Node-Major-Version &gt;= 20 ist (wir setzen Node 22+ ein).
* `protobufjs` `postinstall`: gibt Warnungen zu inkompatiblen Versionsschemata aus (keine Build-Artefakte).

Wenn du auf ein tatsächliches Laufzeitproblem stößt, das diese Skripte erfordert, vertraue ihnen explizit:

```sh
bun pm trust @whiskeysockets/baileys protobufjs
```


<div id="caveats">
  ## Einschränkungen
</div>

- Einige Skripte haben pnpm derzeit noch hartcodiert (z. B. `docs:build`, `ui:*`, `protocol:check`). Führe sie vorerst mit pnpm aus.