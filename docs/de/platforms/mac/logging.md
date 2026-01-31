---
title: Protokollierung
summary: "OpenClaw-Protokollierung: rollierende Diagnose-Protokolldatei + Unified-Log-Datenschutz-Flags"
read_when:
  - Erfassen von macOS-Protokollen oder Untersuchen der Protokollierung personenbezogener Daten
  - Beheben von Problemen mit Voice-Wake und dem Sitzungslebenszyklus
---

<div id="logging-macos">
  # Logging (macOS)
</div>

<div id="rolling-diagnostics-file-log-debug-pane">
  ## Rollierendes Diagnoseprotokoll als Datei (Debug-Bereich)
</div>

OpenClaw leitet macOS-App-Logs über swift-log (standardmäßig Unified Logging) und kann bei Bedarf ein lokales, rotierendes Protokoll als Datei dauerhaft auf die Festplatte schreiben.

- Ausführlichkeit: **Debug-Bereich → Logs → App logging → Verbosity**
- Aktivieren: **Debug-Bereich → Logs → App logging → „Write rolling diagnostics log (JSONL)”**
- Speicherort: `~/Library/Logs/OpenClaw/diagnostics.jsonl` (rotiert automatisch; alte Dateien erhalten die Suffixe `.1`, `.2`, …)
- Löschen: **Debug-Bereich → Logs → App logging → „Clear”**

Hinweise:

- Dies ist **standardmäßig deaktiviert**. Aktiviere es nur, während du aktiv debuggst.
- Behandle die Datei als sensibel; teile sie nicht ohne vorherige Prüfung.

<div id="unified-logging-private-data-on-macos">
  ## Private Daten im Unified Logging unter macOS
</div>

Unified Logging blendet die meisten Payload-Daten aus, sofern ein Subsystem nicht explizit `privacy -off` aktiviert. Gemäß Peters Artikel zu macOS-[Logging-Privacy-Spielereien](https://steipete.me/posts/2025/logging-privacy-shenanigans) (2025) wird dies über eine plist in `/Library/Preferences/Logging/Subsystems/` gesteuert, die den Subsystem-Namen als Schlüssel verwendet. Nur neue Log-Einträge übernehmen dieses Flag, aktiviere es also, bevor du ein Problem reproduzierst.

<div id="enable-for-openclaw-botmolt">
  ## Für OpenClaw (`bot.molt`) aktivieren
</div>

* Schreibe die plist-Datei zunächst in eine temporäre Datei und installiere sie anschließend als Root-Benutzer atomar:

```bash
cat <<'EOF' >/tmp/bot.molt.plist
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>DEFAULT-OPTIONS</key>
    <dict>
        <key>Enable-Private-Data</key>
        <true/>
    </dict>
</dict>
</plist>
EOF
sudo install -m 644 -o root -g wheel /tmp/bot.molt.plist /Library/Preferences/Logging/Subsystems/bot.molt.plist
```

* Kein Neustart ist erforderlich; `logd` erkennt die Datei schnell, aber nur neue Protokollzeilen enthalten private Payloads.
* Sehen Sie sich die ausführlichere Ausgabe mit dem vorhandenen Hilfsskript an, z. B. `./scripts/clawlog.sh --category WebChat --last 5m`.


<div id="disable-after-debugging">
  ## Nach dem Debugging deaktivieren
</div>

- Entferne das Override: `sudo rm /Library/Preferences/Logging/Subsystems/bot.molt.plist`.
- Führe optional `sudo log config --reload` aus, um logd dazu zu zwingen, das Override sofort zu verwerfen.
- Denk daran, dass diese Log-Ausgabe Telefonnummern und Nachrichteninhalte enthalten kann; behalte die plist-Datei nur so lange bei, wie du die zusätzlichen Details aktiv benötigst.