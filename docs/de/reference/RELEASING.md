---
title: RELEASE
summary: "Schritt-für-Schritt-Checkliste für Releases von npm und der macOS-App"
read_when:
  - Beim Erstellen eines neuen npm-Releases
  - Beim Erstellen eines neuen macOS-App-Releases
  - Beim Überprüfen der Metadaten vor der Veröffentlichung
---

<div id="release-checklist-npm-macos">
  # Release-Checkliste (npm + macOS)
</div>

Verwende `pnpm` (Node 22+) vom Repository-Root aus. Halte den Working Tree sauber, bevor du taggst bzw. veröffentlichst.

<div id="operator-trigger">
  ## Operator-Trigger
</div>

Wenn der Operator „release“ sagt, führe sofort diesen Preflight-Check aus (keine zusätzlichen Fragen, außer du bist blockiert):

* Lies dieses Dokument und `docs/platforms/mac/release.md`.
* Lade die Umgebungsvariablen aus `~/.profile` und bestätige, dass `SPARKLE_PRIVATE_KEY_FILE` + App-Store-Connect-Variablen gesetzt sind (`SPARKLE_PRIVATE_KEY_FILE` sollte in `~/.profile` liegen).
* Verwende bei Bedarf die Sparkle-Schlüssel aus `~/Library/CloudStorage/Dropbox/Backup/Sparkle`.

1. **Version &amp; Metadaten**

* [ ] Erhöhe die `package.json`-Version (z. B. `2026.1.29`).
* [ ] Führe `pnpm plugins:sync` aus, um Erweiterungs-Paketversionen + Changelogs zu synchronisieren.
* [ ] Aktualisiere CLI/Versions-Strings: [`src/cli/program.ts`](https://github.com/openclaw/openclaw/blob/main/src/cli/program.ts) und den Baileys-User-Agent in [`src/provider-web.ts`](https://github.com/openclaw/openclaw/blob/main/src/provider-web.ts).
* [ ] Bestätige die Paket-Metadaten (Name, Beschreibung, Repository, Keywords, Lizenz) und dass die `bin`-Map für `openclaw` auf [`openclaw.mjs`](https://github.com/openclaw/openclaw/blob/main/openclaw.mjs) zeigt.
* [ ] Wenn sich Abhängigkeiten geändert haben, führe `pnpm install` aus, damit `pnpm-lock.yaml` aktuell ist.

2. **Build &amp; Artefakte**

* [ ] Wenn sich A2UI-Inputs geändert haben, führe `pnpm canvas:a2ui:bundle` aus und committe alle aktualisierten [`src/canvas-host/a2ui/a2ui.bundle.js`](https://github.com/openclaw/openclaw/blob/main/src/canvas-host/a2ui/a2ui.bundle.js).
* [ ] `pnpm run build` (generiert `dist/` neu).
* [ ] Überprüfe, dass die `files` im npm-Paket alle benötigten `dist/*`-Ordner enthalten (insbesondere `dist/node-host/**` und `dist/acp/**` für den headless Knoten + die ACP-CLI).
* [ ] Bestätige, dass `dist/build-info.json` existiert und den erwarteten `commit`-Hash enthält (der CLI-Banner verwendet diesen für npm-Installationen).
* [ ] Optional: `npm pack --pack-destination /tmp` nach dem Build; inspiziere den Inhalt des Tarballs und halte ihn für das GitHub-Release bereit (auf keinen Fall einchecken).

3. **Changelog &amp; Doku**

* [ ] Aktualisiere `CHANGELOG.md` mit benutzerrelevanten Highlights (erstelle die Datei, falls sie fehlt); halte Einträge strikt absteigend nach Version sortiert.
* [ ] Stelle sicher, dass README-Beispiele/Flags dem aktuellen CLI-Verhalten entsprechen (insbesondere neue Befehle oder Optionen).

4. **Validierung**

* [ ] `pnpm lint`
* [ ] `pnpm test` (oder `pnpm test:coverage`, falls du Coverage-Ausgabe benötigst)
* [ ] `pnpm run build` (letzter Plausibilitätscheck nach den Tests)
* [ ] `pnpm release:check` (überprüft den Inhalt von `npm pack`)
* [ ] `OPENCLAW_INSTALL_SMOKE_SKIP_NONROOT=1 pnpm test:install:smoke` (Docker-Install-Smoke-Test, schneller Pfad; vor dem Release erforderlich)
  * Wenn das direkt vorherige npm-Release als fehlerhaft bekannt ist, setze `OPENCLAW_INSTALL_SMOKE_PREVIOUS=<last-good-version>` oder `OPENCLAW_INSTALL_SMOKE_SKIP_PREVIOUS=1` für den Preinstall-Schritt.
* [ ] (Optional) Vollständiger Installer-Smoke (fügt Non-Root- + CLI-Coverage hinzu): `pnpm test:install:smoke`
* [ ] (Optional) Installer-E2E (Docker, führt `curl -fsSL https://openclaw.bot/install.sh | bash` aus, führt das Onboarding durch und führt dann echte Tool-Aufrufe aus):
  * `pnpm test:install:e2e:openai` (benötigt `OPENAI_API_KEY`)
  * `pnpm test:install:e2e:anthropic` (benötigt `ANTHROPIC_API_KEY`)
  * `pnpm test:install:e2e` (benötigt beide Schlüssel; führt beide Anbieter aus)
* [ ] (Optional) Stichprobenprüfung des Web-Gateways, falls deine Änderungen Sende-/Empfangspfade betreffen.

5. **macOS-App (Sparkle)**

* [ ] Baue und signiere die macOS-App und packe sie anschließend als ZIP-Archiv für die Verteilung.
* [ ] Generiere den Sparkle-Appcast (HTML-Notizen über [`scripts/make_appcast.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/make_appcast.sh)) und aktualisiere `appcast.xml`.
* [ ] Halte das App-ZIP (und optional das dSYM-ZIP) bereit, um es an das GitHub-Release anzuhängen.
* [ ] Siehe [macOS release](/de/platforms/mac/release) für die exakten Befehle und erforderlichen Umgebungsvariablen.
  * `APP_BUILD` muss numerisch und monoton steigend sein (kein `-beta`), damit Sparkle Versionen korrekt vergleicht.
  * Beim Notarisieren verwende das `openclaw-notary`-Schlüsselbundprofil, das aus den App Store Connect API-Umgebungsvariablen erstellt wurde (siehe [macOS release](/de/platforms/mac/release)).

6. **Veröffentlichen (npm)**

* [ ] Stelle sicher, dass der Git-Status sauber ist; committe und pushe bei Bedarf.
* [ ] `npm login` (prüfe 2FA), falls nötig.
* [ ] `npm publish --access public` (verwende `--tag beta` für Vorabversionen).
* [ ] Überprüfe das Registry: `npm view openclaw version`, `npm view openclaw dist-tags` und `npx -y openclaw@X.Y.Z --version` (oder `--help`).

<div id="troubleshooting-notes-from-200-beta2-release">
  ### Troubleshooting (Notizen zum 2.0.0-beta2-Release)
</div>

* **`npm pack/publish` hängt oder erzeugt ein riesiges Tar-Archiv**: Das macOS-App-Bundle in `dist/OpenClaw.app` (und Release-ZIPs) wird in das Paket mit aufgenommen. Behebe das Problem, indem du die zu veröffentlichenden Inhalte über das Feld `files` in der `package.json` definierst (dist-Unterverzeichnisse, Doku, Fähigkeiten einbeziehen; App-Bundles ausschließen). Bestätige mit `npm pack --dry-run`, dass `dist/OpenClaw.app` nicht aufgeführt ist.
* **npm-Auth-Web-Loop für dist-tags**: Verwende Legacy-Authentifizierung, um eine OTP-Eingabeaufforderung zu erhalten:
  * `NPM_CONFIG_AUTH_TYPE=legacy npm dist-tag add openclaw@X.Y.Z latest`
* **`npx`-Verifizierung schlägt fehl mit `ECOMPROMISED: Lock compromised`**: Erneut versuchen mit frischem Cache:
  * `NPM_CONFIG_CACHE=/tmp/npm-cache-$(date +%s) npx -y openclaw@X.Y.Z --version`
* **Tag muss nach einem späten Fix neu ausgerichtet werden**: Tag per Force-Update neu setzen und pushen, dann sicherstellen, dass die GitHub-Release-Assets weiterhin übereinstimmen:
  * `git tag -f vX.Y.Z && git push -f origin vX.Y.Z`

7. **GitHub-Release + Appcast**

* [ ] Taggen und pushen: `git tag vX.Y.Z && git push origin vX.Y.Z` (oder `git push --tags`).
* [ ] Das GitHub-Release für `vX.Y.Z` erstellen/aktualisieren mit **Titel `openclaw X.Y.Z`** (nicht nur der Tag); der Inhalt sollte den **vollständigen** Changelog-Abschnitt für diese Version enthalten (Highlights + Changes + Fixes), inline (keine nackten Links) und **darf den Titel im Inhalt nicht wiederholen**.
* [ ] Artefakte anhängen: `npm pack`-Tarball (optional), `OpenClaw-X.Y.Z.zip` und `OpenClaw-X.Y.Z.dSYM.zip` (falls erzeugt).
* [ ] Die aktualisierte `appcast.xml` committen und pushen (Sparkle bezieht Feeds von `main`).
* [ ] In einem sauberen temporären Verzeichnis (kein `package.json`) `npx -y openclaw@X.Y.Z send --help` ausführen, um zu bestätigen, dass Installation/CLI-Einstiegspunkte funktionieren.
* [ ] Release-Notes ankündigen/teilen.

<div id="plugin-publish-scope-npm">
  ## Plugin-Veröffentlichungs-Scope (npm)
</div>

Wir veröffentlichen nur **bereits vorhandene npm-Plugins** unter dem `@openclaw/*` Scope. Gebündelte
Plugins, die nicht auf npm sind, bleiben **nur im Verzeichnisbaum auf der Disk** (werden weiterhin in
`extensions/**` ausgeliefert).

Vorgehen zur Ermittlung der Liste:

1. `npm search @openclaw --json` ausführen und die Paketnamen erfassen.
2. Mit den Namen aus `extensions/*/package.json` vergleichen.
3. Nur die **Schnittmenge** veröffentlichen (bereits auf npm vorhanden).

Aktuelle npm-Plugin-Liste (bei Bedarf aktualisieren):

* @openclaw/bluebubbles
* @openclaw/diagnostics-otel
* @openclaw/discord
* @openclaw/lobster
* @openclaw/matrix
* @openclaw/msteams
* @openclaw/nextcloud-talk
* @openclaw/nostr
* @openclaw/voice-call
* @openclaw/zalo
* @openclaw/zalouser

Release-Notes müssen außerdem **neue optionale gebündelte Plugins** hervorheben, die **standardmäßig
deaktiviert** sind (Beispiel: `tlon`).