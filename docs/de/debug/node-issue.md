---
title: Node-Problem
summary: Hinweise und Workarounds zu Node-+‑tsx-„__name is not a function“-Abstürzen
read_when:
  - Debugging von reinen Node-Entwicklerskripten oder Fehlern im Watch-Modus
  - Untersuchung von tsx/esbuild-Loader-Abstürzen in OpenClaw
---

<div id="node-tsx-__name-is-not-a-function-crash">
  # Node + tsx „__name is not a function“-Absturz
</div>

<div id="summary">
  ## Zusammenfassung
</div>

Das Ausführen von OpenClaw mit Node.js und `tsx` schlägt bereits beim Start mit folgender Fehlermeldung fehl:

```
[openclaw] CLI-Start fehlgeschlagen: TypeError: __name is not a function
    at createSubsystemLogger (.../src/logging/subsystem.ts:203:25)
    at .../src/agents/auth-profiles/constants.ts:25:20
```

Das trat nach dem Umstellen der Dev-Skripte von Bun auf `tsx` auf (Commit `2871657e`, 2026-01-06). Derselbe Codepfad funktionierte mit Bun.


<div id="environment">
  ## Umgebung
</div>

- Node: v25.x (beobachtet mit v25.3.0)
- tsx: 4.21.0
- OS: macOS (wahrscheinlich auch auf anderen Plattformen reproduzierbar, die Node 25 ausführen)

<div id="repro-node-only">
  ## Repro (nur Node.js)
</div>

```bash
# im Repository-Stammverzeichnis
node --version
pnpm install
node --import tsx src/entry.ts status
```


<div id="minimal-repro-in-repo">
  ## Minimales Reproduktionsbeispiel im Repository
</div>

```bash
node --import tsx scripts/repro/tsx-name-repro.ts
```


<div id="node-version-check">
  ## Überprüfung der Node-Version
</div>

- Node 25.3.0: schlägt fehl
- Node 22.22.0 (Homebrew `node@22`): schlägt fehl
- Node 24: hier noch nicht installiert; muss noch geprüft werden

<div id="notes-hypothesis">
  ## Notizen / Hypothesen
</div>

- `tsx` verwendet esbuild, um TS/ESM zu transformieren. Die esbuild-Option `keepNames` emittiert einen `__name`-Helper und umschließt Funktionsdefinitionen mit `__name(...)`.
- Der Absturz zeigt an, dass `__name` existiert, aber zur Laufzeit keine Funktion ist, was impliziert, dass der Helper im Node-25-Loader-Pfad für dieses Modul fehlt oder überschrieben wurde.
- Ähnliche Probleme mit dem `__name`-Helper wurden bei anderen esbuild-Nutzern berichtet, wenn der Helper fehlt oder umgeschrieben wird.

<div id="regression-history">
  ## Regressionsverlauf
</div>

- `2871657e` (2026-01-06): Skripte wurden von Bun auf tsx umgestellt, um Bun optional zu machen.
- Davor (Bun-Codepfad) funktionierten `openclaw status` und `gateway:watch`.

<div id="workarounds">
  ## Workarounds
</div>

- Verwende Bun für Dev-Skripte (derzeit als temporärer Rückgriff).
- Verwende Node + tsc watch und führe dann die kompilierte Ausgabe aus:
  ```bash
  pnpm exec tsc --watch --preserveWatchOutput
  node --watch openclaw.mjs status
  ```
- Lokal bestätigt: `pnpm exec tsc -p tsconfig.json` + `node openclaw.mjs status` funktioniert mit Node 25.
- Deaktiviere `esbuild keepNames` im TS-Loader, falls möglich (verhindert das Einfügen des `__name`-Helpers); `tsx` stellt dies aktuell nicht zur Verfügung.
- Teste Node LTS (22/24) mit `tsx`, um zu prüfen, ob das Problem spezifisch für Node 25 ist.

<div id="references">
  ## Referenzen
</div>

- https://opennext.js.org/cloudflare/howtos/keep_names
- https://esbuild.github.io/api/#keep-names
- https://github.com/evanw/esbuild/issues/1031

<div id="next-steps">
  ## Nächste Schritte
</div>

- Unter Node 22/24 reproduzieren, um die Regression unter Node 25 zu bestätigen.
- Nightly-Build von `tsx` testen oder auf eine frühere Version pinnen, falls eine bekannte Regression vorliegt.
- Wenn sich das Problem unter Node LTS reproduzieren lässt, ein minimales Reproduktionsbeispiel mit dem `__name`-Stacktrace upstream einreichen.