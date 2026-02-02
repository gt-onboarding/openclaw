---
title: Update
summary: "CLI-Referenz für `openclaw update` (relativ sichere Aktualisierung des Quellstands + automatischer Gateway-Neustart)"
read_when:
  - Du möchtest ein Quell-Checkout sicher aktualisieren
  - Du musst das Verhalten der Kurzform `--update` verstehen
---

<div id="openclaw-update">
  # `openclaw update`
</div>

OpenClaw sicher aktualisieren und zwischen Stable-/Beta-/Dev-Kanälen wechseln.

Wenn du OpenClaw über **npm/pnpm** installiert hast (globale Installation, keine Git-Metadaten), führst du Updates über den Paketmanager-Workflow wie unter [Aktualisieren](/de/install/updating) beschrieben durch.

<div id="usage">
  ## Verwendung
</div>

```bash
openclaw update
openclaw update status
openclaw update wizard
openclaw update --channel beta
openclaw update --channel dev
openclaw update --tag beta
openclaw update --no-restart
openclaw update --json
openclaw --update
```

<div id="options">
  ## Optionen
</div>

* `--no-restart`: Neustart des Gateway-Dienstes nach einem erfolgreichen Update überspringen.
* `--channel <stable|beta|dev>`: Update-Kanal festlegen (git + npm; wird in der Konfiguration gespeichert).
* `--tag <dist-tag|version>`: npm-`dist-tag` oder Version nur für dieses Update übersteuern.
* `--json`: maschinenlesbares `UpdateRunResult`-JSON ausgeben.
* `--timeout <seconds>`: Timeout pro Schritt (Standard: 1200 s).

Hinweis: Downgrades erfordern eine Bestätigung, da ältere Versionen die Konfiguration beschädigen können.

<div id="update-status">
  ## `update status`
</div>

Zeigt den aktiven Update-Kanal + Git-Tag/Branch/SHA (für Quellcode-Checkouts) sowie die Verfügbarkeit von Updates an.

```bash
openclaw update status
openclaw update status --json
openclaw update status --timeout 10
```

Optionen:

* `--json`: maschinenlesbares Status-JSON ausgeben.
* `--timeout <seconds>`: Timeout für Prüfungen (Standard ist 3 s).

<div id="update-wizard">
  ## `update wizard`
</div>

Interaktiver Ablauf, mit dem du einen Update-Kanal auswählst und bestätigst, ob das Gateway
nach dem Update neu gestartet werden soll (Standard ist, neu zu starten). Wenn du `dev` ohne Git-Checkout auswählst, wird angeboten, eines zu erstellen.

<div id="what-it-does">
  ## Was es macht
</div>

Wenn du den Kanal explizit wechselst (`--channel ...`), hält OpenClaw auch die
Installationsmethode synchron:

* `dev` → stellt sicher, dass ein Git-Checkout vorhanden ist (Standard: `~/openclaw`, überschreibbar mit `OPENCLAW_GIT_DIR`),
  aktualisiert ihn und installiert die globale CLI aus diesem Checkout.
* `stable`/`beta` → installiert aus npm mit dem passenden dist-tag.

<div id="git-checkout-flow">
  ## Git-Checkout-Flow
</div>

Channels:

* `stable`: den aktuellsten nicht als Beta gekennzeichneten Tag auschecken, dann Build + Doctor ausführen.
* `beta`: den aktuellsten `-beta`-Tag auschecken, dann Build + Doctor ausführen.
* `dev`: `main` auschecken, dann Fetch + Rebase ausführen.

Überblick:

1. Erfordert einen sauberen Worktree (keine nicht committeten Änderungen).
2. Wechselt zum ausgewählten Channel (Tag oder Branch).
3. Holt Änderungen vom Upstream (nur dev).
4. Nur dev: Preflight-Lint + TypeScript-Build in einem temporären Worktree; wenn der Tip-Commit fehlschlägt, wird bis zu 10 Commits zurückgegangen, um den neuesten sauberen Build zu finden.
5. Führt ein Rebase auf den ausgewählten Commit durch (nur dev).
6. Installiert Abhängigkeiten (pnpm bevorzugt; npm als Fallback).
7. Baut das Projekt und die Control UI.
8. Führt `openclaw doctor` als abschließenden „Safe-Update“-Check aus.
9. Synchronisiert Plugins mit dem aktiven Channel (dev verwendet gebündelte Erweiterungen; stable/beta verwendet npm) und aktualisiert per npm installierte Plugins.

<div id="update-shorthand">
  ## `--update` Kurzform
</div>

`openclaw --update` ist eine Kurzform für `openclaw update` (praktisch für Shells und Startskripte).

<div id="see-also">
  ## Siehe auch
</div>

* `openclaw doctor` (bietet für Git-Checkouts an, zuerst ein Update auszuführen)
* [Entwicklungskanäle](/de/install/development-channels)
* [Aktualisieren](/de/install/updating)
* [CLI-Referenz](/de/cli)