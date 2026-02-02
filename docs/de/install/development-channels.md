---
title: Entwicklungskanäle
summary: "Stable-, Beta- und Dev-Kanäle: Semantik, Wechsel und Tagging"
read_when:
  - Du möchtest zwischen Stable/Beta/Dev wechseln
  - Du versiehst Vorabversionen mit Tags oder veröffentlichst sie
---

<div id="development-channels">
  # Entwicklungskanäle
</div>

Zuletzt aktualisiert: 2026-01-21

OpenClaw bietet drei Update-Kanäle:

- **stable**: npm dist-tag `latest`.
- **beta**: npm dist-tag `beta` (Builds im Test).
- **dev**: aktueller Head von `main` (git). npm dist-tag: `dev` (falls veröffentlicht).

Wir liefern Builds an **beta** aus, testen sie und **stufen dann einen geprüften Build zu `latest` hoch**, ohne die Versionsnummer zu ändern — dist-tags sind die maßgebliche Referenz für npm-Installationen.

<div id="switching-channels">
  ## Kanäle wechseln
</div>

Git-Checkout:

```bash
openclaw update --channel stable
openclaw update --channel beta
openclaw update --channel dev
```

* `stable`/`beta` checkt den neuesten passenden Tag aus (oft derselbe Tag).
* `dev` wechselt auf `main` und führt ein Rebase auf den Upstream durch.

globale Installation mit npm/pnpm:

```bash
openclaw update --channel stable
openclaw update --channel beta
openclaw update --channel dev
```

Dies wird über das entsprechende npm-Dist-Tag (`latest`, `beta`, `dev`) aktualisiert.

Wenn du **explizit** mit `--channel` den Kanal wechselst, passt OpenClaw auch
die Installationsmethode entsprechend an:

* `dev` sorgt für einen Git-Checkout (Standard `~/openclaw`, überschreibbar mit `OPENCLAW_GIT_DIR`),
  aktualisiert ihn und installiert die globale CLI aus diesem Checkout.
* `stable`/`beta` werden aus npm mit dem passenden Dist-Tag installiert.

Tipp: Wenn du stable und dev parallel nutzen willst, verwende zwei separate Klone und konfiguriere dein Gateway so, dass es auf den stabilen Klon zeigt.


<div id="plugins-and-channels">
  ## Plugins und Kanäle
</div>

Wenn du mit `openclaw update` den Kanal wechselst, synchronisiert OpenClaw auch die Plugin-Quellen:

- `dev` bevorzugt gebündelte Plugins aus dem Git-Checkout.
- `stable` und `beta` stellen über npm installierte Plugin-Pakete wieder her.

<div id="tagging-best-practices">
  ## Best Practices für Tagging
</div>

- Vergib Tags für Releases, auf die `git checkout`-Aufrufe zeigen sollen (`vYYYY.M.D` oder `vYYYY.M.D-<patch>`).
- Halte Tags unveränderlich: verschiebe oder verwende einen Tag niemals erneut.
- npm dist-tags bleiben die maßgebliche Referenz für npm-Installationen:
  - `latest` → stabil
  - `beta` → Kandidaten-Build
  - `dev` → `main`-Snapshot (optional)

<div id="macos-app-availability">
  ## Verfügbarkeit der macOS-App
</div>

Beta- und Dev-Builds müssen **nicht** zwingend ein Release der macOS-App enthalten. Das ist in Ordnung:

- Der Git-Tag und der npm dist-tag können trotzdem veröffentlicht werden.
- Weisen Sie in den Release Notes oder im Changelog ausdrücklich auf „kein macOS-Build für diese Beta-Version“ hin.