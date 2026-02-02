---
title: Aktualisieren
summary: "Sicheres Aktualisieren von OpenClaw (globale Installation oder aus dem Quellcode), plus Rollback-Strategie"
read_when:
  - OpenClaw aktualisieren
  - Wenn nach einem Update etwas nicht mehr funktioniert
---

<div id="updating">
  # Aktualisieren
</div>

OpenClaw entwickelt sich schnell weiter (vor „1.0“). Behandle Updates wie Infrastruktur-Deployments: aktualisieren → Checks ausführen → neu starten (oder `openclaw update` verwenden, das neu startet) → überprüfen.

<div id="recommended-re-run-the-website-installer-upgrade-in-place">
  ## Empfohlen: Website-Installer erneut ausführen (In-place-Upgrade)
</div>

Der **bevorzugte** Aktualisierungspfad besteht darin, den Installer von der Website erneut auszuführen. Er erkennt vorhandene Installationen, führt ein In-place-Upgrade durch und startet bei Bedarf `openclaw doctor`.

```bash
curl -fsSL https://openclaw.bot/install.sh | bash
```

Hinweise:

* Füge die Option `--no-onboard` hinzu, wenn du nicht möchtest, dass der Onboarding-Assistent erneut ausgeführt wird.
* Für **Installationen aus dem Quellcode** verwende:
  ```bash
  curl -fsSL https://openclaw.bot/install.sh | bash -s -- --install-method git --no-onboard
  ```
  Der Installer führt `git pull --rebase` **nur** aus, wenn das Repository sauber ist.
* Für **globale Installationen** verwendet das Skript intern `npm install -g openclaw@latest`.
* Hinweis zur Abwärtskompatibilität: `openclaw` bleibt als Kompatibilitäts-Shim verfügbar.

<div id="before-you-update">
  ## Bevor du aktualisierst
</div>

* Stelle fest, wie du installiert hast: **global** (npm/pnpm) vs **aus dem Quellcode** (git clone).
* Stelle fest, wie dein Gateway läuft: **im Vordergrund im Terminal** vs **über einen überwachten Dienst** (launchd/systemd).
* Erstelle eine Sicherung deiner Anpassungen:
  * Konfiguration: `~/.openclaw/openclaw.json`
  * Zugangsdaten: `~/.openclaw/credentials/`
  * Arbeitsbereich: `~/.openclaw/workspace`

<div id="update-global-install">
  ## Update (globale Installation)
</div>

Globale Installation (eine der folgenden Optionen wählen):

```bash
npm i -g openclaw@latest
```

```bash
pnpm add -g openclaw@latest
```

Wir **empfehlen Bun nicht** als Runtime des Gateway (WhatsApp-/Telegram-Bugs).

So wechselst du Update-Kanäle (git + npm-Installationen):

```bash
openclaw update --channel beta
openclaw update --channel dev
openclaw update --channel stable
```

Verwende `--tag <dist-tag|version>` für eine einmalige Installation mit einem bestimmten Tag bzw. einer bestimmten Version.

Siehe [Entwicklungskanäle](/de/install/development-channels) für die Semantik der Kanäle und die Release Notes.

Hinweis: Bei npm-Installationen schreibt das Gateway beim Start einen Update-Hinweis ins Log (es prüft das aktuelle Channel-Tag). Deaktiviere dies über `update.checkOnStart: false`.

Dann:

```bash
openclaw doctor
openclaw gateway restart
openclaw health
```

Hinweise:

* Wenn dein Gateway als Dienst läuft, ist `openclaw gateway restart` der bevorzugte Weg gegenüber dem Beenden von PIDs.
* Wenn du an eine bestimmte Version gebunden bist, siehe „Rollback / Pinning“ unten.

<div id="update-openclaw-update">
  ## Update (`openclaw update`)
</div>

Für **Source-Installationen** (Git-Checkout) verwende bevorzugt:

```bash
openclaw update
```

Es führt einen relativ sicheren Update-Workflow aus:

* Erfordert einen sauberen Worktree.
* Wechselt in den ausgewählten Channel (Tag oder Branch).
* Holt Änderungen und führt ein Rebase auf den konfigurierten Upstream (Dev-Channel) aus.
* Installiert Abhängigkeiten, baut das Projekt, baut die Control UI und führt `openclaw doctor` aus.
* Startet das Gateway standardmäßig neu (verwende `--no-restart`, um das zu überspringen).

Wenn du über **npm/pnpm** installiert hast (keine Git-Metadaten), versucht `openclaw update`, das Update über deinen Paketmanager durchzuführen. Wenn die Installation nicht erkannt werden kann, verwende stattdessen „Update (global install)“.

<div id="update-control-ui-rpc">
  ## Update (Control UI / RPC)
</div>

Die Control UI verfügt über **Update &amp; Restart** (RPC: `update.run`). Diese Funktion:

1. Führt denselben Update-Vorgang aus dem Quellcode aus wie `openclaw update` (nur `git checkout`).
2. Schreibt ein Neustart-Sentinel mit einem strukturierten Bericht (Tail von stdout/stderr).
3. Startet das Gateway neu und pingt die zuletzt aktive Sitzung mit dem Bericht an.

Wenn das Rebase fehlschlägt, bricht das Gateway ab und startet ohne Anwendung des Updates neu.

<div id="update-from-source">
  ## Update (aus dem Quellcode)
</div>

Im Repository-Checkout:

Bevorzugt:

```bash
openclaw update
```

Manuell (in etwa entsprechend):

```bash
git pull
pnpm install
pnpm build
pnpm ui:build # installiert UI-Abhängigkeiten beim ersten Durchlauf automatisch
openclaw doctor
openclaw health
```

Hinweise:

* `pnpm build` ist wichtig, wenn du das gepackte `openclaw`-Binary ([`openclaw.mjs`](https://github.com/openclaw/openclaw/blob/main/openclaw.mjs)) ausführst oder Node.js verwendest, um `dist/` auszuführen.
* Wenn du aus einem Repo-Checkout ohne globale Installation ausführst, verwende für CLI-Befehle `pnpm openclaw ...`.
* Wenn du direkt aus TypeScript ausführst (`pnpm openclaw ...`), ist ein erneuter Build normalerweise nicht nötig, aber **Konfigurationsmigrationen gelten trotzdem** → führe `openclaw doctor` aus.
* Das Umschalten zwischen globalen und Git-Installationen ist einfach: Installiere die andere Variante und führe anschließend `openclaw doctor` aus, damit der Gateway-Service-Entrypoint auf die aktuelle Installation umgeschrieben wird.

<div id="always-run-openclaw-doctor">
  ## Immer ausführen: `openclaw doctor`
</div>

Doctor ist der „Safe-Update“-Befehl. Er ist absichtlich langweilig: reparieren + migrieren + warnen.

Hinweis: Wenn du eine **Source-Installation** (Git-Checkout) verwendest, wird `openclaw doctor` anbieten, zuerst `openclaw update` auszuführen.

Typische Aufgaben:

* Veraltete Konfigurationsschlüssel und Legacy-Konfigurationsdateistandorte migrieren.
* DM-Richtlinien prüfen und vor riskanten `open`-Einstellungen warnen (Policy-Token `open`, erlaubt unbegrenzte Nachrichtenannahme von beliebigen Nutzern).
* Zustand des Gateways prüfen und bei Bedarf einen Neustart anbieten.
* Ältere Gateway-Dienste (launchd/systemd; Legacy-schtasks) erkennen und auf aktuelle OpenClaw-Dienste migrieren.
* Unter Linux sicherstellen, dass `systemd user lingering` aktiviert ist (damit das Gateway einen Logout überlebt).

Details: [Doctor](/de/gateway/doctor)

<div id="start-stop-restart-the-gateway">
  ## Gateway starten / stoppen / neu starten
</div>

CLI (plattformunabhängig):

```bash
openclaw gateway status
openclaw gateway stop
openclaw gateway restart
openclaw gateway --port 18789
openclaw logs --follow
```

Wenn du einen Supervisor nutzt:

* macOS launchd (App-gebundener LaunchAgent): `launchctl kickstart -k gui/$UID/bot.molt.gateway` (verwende `bot.molt.<profile>`; das frühere `com.openclaw.*` funktioniert weiterhin)
* Linux systemd User-Service: `systemctl --user restart openclaw-gateway[-<profile>].service`
* Windows (WSL2): `systemctl --user restart openclaw-gateway[-<profile>].service`
  * `launchctl`/`systemctl` funktionieren nur, wenn der Service installiert ist; andernfalls führe `openclaw gateway install` aus.

Runbook + exakte Service-Labels: [Gateway-Runbook](/de/gateway)

<div id="rollback-pinning-when-something-breaks">
  ## Rollback / Version-Pinning (bei Problemen)
</div>

<div id="pin-global-install">
  ### Pin (globale Installation)
</div>

Installiere eine als funktionierend bekannte Version (ersetze `<version>` durch die zuletzt funktionierende):

```bash
npm i -g openclaw@<version>
```

```bash
pnpm add -g openclaw@<version>
```

Tipp: Um die aktuell veröffentlichte Version zu sehen, führe `npm view openclaw version` aus.

Starte anschließend neu und führe `doctor` erneut aus:

```bash
openclaw doctor
openclaw gateway restart
```

<div id="pin-source-by-date">
  ### Pin (Quellstand) nach Datum
</div>

Wähle einen Commit anhand eines Datums aus (Beispiel: „Stand von main am 2026-01-01“):

```bash
git fetch origin
git checkout "$(git rev-list -n 1 --before=\"2026-01-01\" origin/main)"
```

Dann Abhängigkeiten neu installieren und neu starten:

```bash
pnpm install
pnpm build
openclaw gateway restart
```

Wenn du später wieder auf die neueste Version wechseln möchtest:

```bash
git checkout main
git pull
```

<div id="if-youre-stuck">
  ## Wenn du feststeckst
</div>

* Führe `openclaw doctor` erneut aus und lies die Ausgabe sorgfältig durch (sie verrät dir häufig die Lösung).
* Sieh dir die [Fehlerbehebung](/de/gateway/troubleshooting) an.
* Frag im Discord: https://channels.discord.gg/clawd