---
title: Signieren
summary: "Schritte zum Signieren von macOS-Debug-Builds, die durch Packaging-Skripte erzeugt werden"
read_when:
  - Erstellen oder Signieren von macOS-Debug-Builds
---

<div id="mac-signing-debug-builds">
  # mac signing (Debug-Builds)
</div>

Diese App wird normalerweise über [`scripts/package-mac-app.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/package-mac-app.sh) gebaut, das jetzt:

* eine stabile Debug-Bundle-ID setzt: `ai.openclaw.mac.debug`
* die Info.plist mit dieser Bundle-ID schreibt (Überschreiben via `BUNDLE_ID=...`)
* [`scripts/codesign-mac-app.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/codesign-mac-app.sh) aufruft, um das Haupt-Binary und das App-Bundle zu signieren, sodass macOS jeden Rebuild als dasselbe signierte Bundle behandelt und TCC-Berechtigungen (Benachrichtigungen, Bedienungshilfen, Bildschirmaufnahme, Mikrofon, Sprachfunktionen) beibehält. Für stabile Berechtigungen solltest du eine echte Signing-Identität verwenden; Ad-hoc ist opt-in und fragil (siehe [macOS permissions](/de/platforms/mac/permissions)).
* standardmäßig `CODESIGN_TIMESTAMP=auto` verwendet; das aktiviert vertrauenswürdige Timestamps für Developer-ID-Signaturen. Setze `CODESIGN_TIMESTAMP=off`, um Timestamping zu überspringen (Offline-Debug-Builds).
* Build-Metadaten in die Info.plist injiziert: `OpenClawBuildTimestamp` (UTC) und `OpenClawGitCommit` (kurzer Hash), sodass der About-Dialog Build, Git und Debug/Release-Channel anzeigen kann.
* **Packaging erfordert Node 22+**: Das Skript führt TS-Builds und den Control-UI-Build aus.
* `SIGN_IDENTITY` aus der Umgebung liest. Füge `export SIGN_IDENTITY="Apple Development: Your Name (TEAMID)"` (oder dein Developer-ID-Application-Zertifikat) zu deiner Shell-RC-Datei hinzu, um immer mit deinem Zertifikat zu signieren. Ad-hoc-Signing erfordert explizites Opt-in via `ALLOW_ADHOC_SIGNING=1` oder `SIGN_IDENTITY="-"` (nicht empfohlen für Berechtigungstests).
* nach dem Signieren ein Team-ID-Audit ausführt und fehlschlägt, wenn irgendeine Mach-O-Datei innerhalb des App-Bundles mit einer anderen Team-ID signiert ist. Setze `SKIP_TEAM_ID_CHECK=1`, um dies zu umgehen.

<div id="usage">
  ## Verwendung
</div>

```bash
# vom Repository-Root
scripts/package-mac-app.sh               # wählt automatisch Identität; Fehler, falls keine gefunden
SIGN_IDENTITY="Developer ID Application: Your Name" scripts/package-mac-app.sh   # echtes Zertifikat
ALLOW_ADHOC_SIGNING=1 scripts/package-mac-app.sh    # ad-hoc (Berechtigungen bleiben nicht bestehen)
SIGN_IDENTITY="-" scripts/package-mac-app.sh        # explizit ad-hoc (gleicher Hinweis)
DISABLE_LIBRARY_VALIDATION=1 scripts/package-mac-app.sh   # nur für Entwicklung: Workaround für Sparkle Team-ID-Konflikt
```

<div id="ad-hoc-signing-note">
  ### Hinweis zur Ad-hoc-Signierung
</div>

Bei der Signierung mit `SIGN_IDENTITY="-"` (Ad-hoc) deaktiviert das Skript automatisch die **Hardened Runtime** (`--options runtime`). Dies ist erforderlich, um Abstürze zu verhindern, wenn die App eingebettete Frameworks (wie Sparkle) lädt, die nicht dieselbe Team-ID verwenden. Ad-hoc-Signaturen unterbrechen außerdem die dauerhafte Speicherung von TCC-Berechtigungen; siehe [macOS permissions](/de/platforms/mac/permissions) für Schritte zur Wiederherstellung.

<div id="build-metadata-for-about">
  ## Build-Metadaten für den About-Tab
</div>

`package-mac-app.sh` versieht das Bundle mit:

* `OpenClawBuildTimestamp`: ISO8601-UTC zum Zeitpunkt der Paketierung
* `OpenClawGitCommit`: kurzer Git-Hash (oder `unknown`, falls nicht verfügbar)

Der About-Tab liest diese Schlüssel aus, um Version, Build-Datum, Git-Commit und den Status als Debug-Build (über `#if DEBUG`) anzuzeigen. Führe das Packager-Skript nach Codeänderungen erneut aus, um diese Werte zu aktualisieren.

<div id="why">
  ## Warum
</div>

TCC-Berechtigungen sind sowohl an die Bundle-ID *als auch* an die Code-Signatur gebunden. Nicht signierte Debug-Builds mit sich ändernden UUIDs führten dazu, dass macOS die Berechtigungen nach jedem Rebuild vergaß. Das Signieren der Binärdateien (standardmäßig ad-hoc) und das Beibehalten einer festen Bundle-ID bzw. eines festen Pfads (`dist/OpenClaw.app`) sorgt dafür, dass die Berechtigungen zwischen Builds erhalten bleiben, analog zum VibeTunnel-Ansatz.