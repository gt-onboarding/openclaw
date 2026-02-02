---
title: Deinstallieren
summary: "OpenClaw vollständig deinstallieren (CLI, Dienst, Statusdaten, arbeitsbereich)"
read_when:
  - Sie möchten OpenClaw von einem Rechner entfernen
  - Der Gateway-Dienst läuft nach der Deinstallation weiterhin
---

<div id="uninstall">
  # Deinstallation
</div>

Zwei Optionen:

- **Einfacher Weg**, falls `openclaw` noch installiert ist.
- **Manuelles Entfernen des Dienstes**, falls die CLI zwar entfernt wurde, der Dienst aber noch läuft.

<div id="easy-path-cli-still-installed">
  ## Einfacher Weg (CLI noch installiert)
</div>

Empfohlen: Verwenden Sie das integrierte Deinstallationsprogramm:

```bash
openclaw uninstall
```

Nicht interaktiv (Automatisierung / npx):

```bash
openclaw uninstall --all --yes --non-interactive
npx -y openclaw uninstall --all --yes --non-interactive
```

Manuelle Schritte (gleiches Ergebnis):

1. Stoppen Sie den Gateway-Dienst:

```bash
openclaw gateway stop
```

2. Deinstalliere den Gateway-Dienst (launchd/systemd/schtasks):

```bash
openclaw gateway uninstall
```

3. Status + Konfiguration löschen:

```bash
rm -rf "${OPENCLAW_STATE_DIR:-$HOME/.openclaw}"
```

Wenn du `OPENCLAW_CONFIG_PATH` auf einen benutzerdefinierten Speicherort außerhalb des State-Verzeichnisses festgelegt hast, lösche diese Datei ebenfalls.

4. Lösche deinen Arbeitsbereich (optional, entfernt Agent-Dateien):

```bash
rm -rf ~/.openclaw/workspace
```

5. Entferne die CLI-Installation (wähle die von dir verwendete Variante):

```bash
npm rm -g openclaw
pnpm remove -g openclaw
bun remove -g openclaw
```

6. Falls Sie die macOS-App installiert haben:

```bash
rm -rf /Applications/OpenClaw.app
```

Hinweise:

* Wenn du Profile (`--profile` / `OPENCLAW_PROFILE`) verwendet hast, wiederhole Schritt 3 für jedes Statusverzeichnis (Standard ist `~/.openclaw-<profile>`).
* Im Remote-Modus liegt das Statusverzeichnis auf dem **Gateway-Host**, daher musst du die Schritte 1–4 auch dort ausführen.


<div id="manual-service-removal-cli-not-installed">
  ## Manuelles Entfernen des Dienstes (CLI nicht installiert)
</div>

Verwende dies, wenn der Gateway-Dienst noch läuft, aber `openclaw` nicht installiert ist.

<div id="macos-launchd">
  ### macOS (launchd)
</div>

Das Standard-Label ist `bot.molt.gateway` (oder `bot.molt.<profile>`; veraltete `com.openclaw.*`-Einträge können noch vorhanden sein):

```bash
launchctl bootout gui/$UID/bot.molt.gateway
rm -f ~/Library/LaunchAgents/bot.molt.gateway.plist
```

Wenn Sie ein Profil verwendet haben, ersetzen Sie das Label und den plist-Namen durch `bot.molt.&lt;profile&gt;`. Entfernen Sie alle veralteten `com.openclaw.*`-plists, falls vorhanden.


<div id="linux-systemd-user-unit">
  ### Linux (systemd-User-Unit)
</div>

Der Standardname der Unit ist `openclaw-gateway.service` (oder `openclaw-gateway-<profile>.service`):

```bash
systemctl --user disable --now openclaw-gateway.service
rm -f ~/.config/systemd/user/openclaw-gateway.service
systemctl --user daemon-reload
```


<div id="windows-scheduled-task">
  ### Windows (Geplante Aufgabe)
</div>

Der Standardname der Aufgabe ist `OpenClaw Gateway` (oder `OpenClaw Gateway (<profile>)`).
Das Aufgabenskript befindet sich in deinem State-Verzeichnis.

```powershell
schtasks /Delete /F /TN "OpenClaw Gateway"
Remove-Item -Force "$env:USERPROFILE\.openclaw\gateway.cmd"
```

Wenn du ein Profil verwendet hast, lösche den entsprechenden Tasknamen sowie `~\.openclaw-<profile>\gateway.cmd`.


<div id="normal-install-vs-source-checkout">
  ## Standardinstallation vs. Quellcode-Checkout
</div>

<div id="normal-install-installsh-npm-pnpm-bun">
  ### Standardinstallation (install.sh / npm / pnpm / bun)
</div>

Wenn du `https://openclaw.bot/install.sh` oder `install.ps1` verwendet hast, wurde die CLI mit `npm install -g openclaw@latest` installiert.
Entferne sie mit `npm rm -g openclaw` (oder `pnpm remove -g` / `bun remove -g`, falls du es auf diesem Weg installiert hast).

<div id="source-checkout-git-clone">
  ### Source checkout (git clone)
</div>

Wenn du OpenClaw direkt aus einem Repository-Checkout (`git clone` + `openclaw ...` / `bun run openclaw ...`) ausführst:

1) Deinstalliere den Gateway-Dienst **bevor** du das Repository löschst (verwende den einfachen Weg oben oder entferne den Dienst manuell).
2) Lösche das Repository-Verzeichnis.
3) Entferne State und Arbeitsbereich wie oben beschrieben.