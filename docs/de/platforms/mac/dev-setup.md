---
title: Dev-Setup
summary: "Einrichtungsanleitung für Entwickler der OpenClaw macOS-App"
read_when:
  - Einrichten der Entwicklungsumgebung für macOS
---

<div id="macos-developer-setup">
  # macOS-Entwickler-Setup
</div>

In diesem Leitfaden werden die erforderlichen Schritte beschrieben, um die OpenClaw macOS-Anwendung aus dem Quellcode zu erstellen und auszuführen.

<div id="prerequisites">
  ## Voraussetzungen
</div>

Bevor du die App erstellst, stelle sicher, dass Folgendes installiert ist:

1.  **Xcode 26.2+**: Erforderlich für die Swift-Entwicklung.
2.  **Node.js 22+ & pnpm**: Erforderlich für das Gateway, die CLI und die Packaging-Skripte.

<div id="1-install-dependencies">
  ## 1. Abhängigkeiten installieren
</div>

Installiere die Projektabhängigkeiten:

```bash
pnpm install
```


<div id="2-build-and-package-the-app">
  ## 2. App erstellen und paketieren
</div>

Um die macOS-App zu erstellen und in `dist/OpenClaw.app` zu paketieren, führe den folgenden Befehl aus:

```bash
./scripts/package-mac-app.sh
```

Wenn du kein Apple Developer ID-Zertifikat hast, verwendet das Skript automatisch **ad-hoc-Signing** (`-`).

Informationen zu Dev-Run-Modi, Signing-Flags und Team-ID-Troubleshooting findest du im macOS-App-README:
https://github.com/openclaw/openclaw/blob/main/apps/macos/README.md

> **Hinweis**: Ad-hoc-signierte Apps können Sicherheitsabfragen auslösen. Wenn die App sofort mit „Abort trap 6“ abstürzt, siehe den Abschnitt [Troubleshooting](#troubleshooting).


<div id="3-install-the-cli">
  ## 3. CLI installieren
</div>

Die macOS-App erwartet eine globale Installation der `openclaw`-CLI, um Hintergrundaufgaben zu verwalten.

**So installierst du sie (empfohlen):**

1. Öffne die OpenClaw-App.
2. Wechsle in den Einstellungen zum Tab **General**.
3. Klicke auf **&quot;Install CLI&quot;**.

Alternativ kannst du sie manuell installieren:

```bash
npm install -g openclaw@<version>
```


<div id="troubleshooting">
  ## Fehlerbehebung
</div>

<div id="build-fails-toolchain-or-sdk-mismatch">
  ### Build schlägt fehl: Toolchain- oder SDK-Inkompatibilität
</div>

Der Build der macOS-App erwartet das neueste macOS-SDK und die Swift-6.2-Toolchain.

**Systemvoraussetzungen (erforderlich):**

* **Neueste in der Softwareupdate-Funktion verfügbare macOS-Version** (erforderlich für die SDKs von Xcode 26.2)
* **Xcode 26.2** (Swift-6.2-Toolchain)

**Prüfungen:**

```bash
xcodebuild -version
xcrun swift --version
```

Wenn die Versionen nicht übereinstimmen, aktualisiere macOS/Xcode und führe den Build erneut durch.


<div id="app-crashes-on-permission-grant">
  ### App stürzt beim Erteilen von Berechtigungen ab
</div>

Wenn die App abstürzt, sobald du versuchst, **Spracherkennungs**- oder **Mikrofon**zugriff zu erlauben, kann das an einem beschädigten TCC-Cache oder einer fehlerhaften Signatur liegen.

**Lösung:**

1. Setze die TCC-Berechtigungen zurück:
   ```bash
   tccutil reset All bot.molt.mac.debug
   ```
2. Wenn das nicht hilft, ändere `BUNDLE_ID` vorübergehend in [`scripts/package-mac-app.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/package-mac-app.sh), um macOS zu einem „sauberen Ausgangszustand“ zu zwingen.

<div id="gateway-starting-indefinitely">
  ### Gateway bleibt dauerhaft bei „Starting…“
</div>

Wenn der Gateway-Status dauerhaft auf „Starting…“ steht, prüfe, ob ein Zombie-Prozess den Port belegt:

```bash
openclaw gateway status
openclaw gateway stop

# Falls Sie keinen LaunchAgent verwenden (Dev-Modus / manuelle Ausführungen), finden Sie den Listener:
lsof -nP -iTCP:18789 -sTCP:LISTEN
```

Wenn eine manuelle Ausführung den Port belegt, beende diesen Prozess mit Strg+C. Als letzte Maßnahme kannst du den Prozess mit der oben gefundenen PID beenden.
