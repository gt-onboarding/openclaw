---
title: Doctor
summary: "Doctor-Befehl: Health Checks, Konfigurationsmigrationen und Reparaturschritte"
read_when:
  - Hinzufügen oder Ändern von Doctor-Migrationen
  - Einführen inkompatibler Konfigurationsänderungen
---

<div id="doctor">
  # Doctor
</div>

`openclaw doctor` ist das Reparatur- und Migrationstool für OpenClaw. Es bereinigt veraltete
Konfigurationen/Zustände, führt Health-Checks durch und gibt konkrete Schritte zur Problembehebung aus.

<div id="quick-start">
  ## Schnellstart
</div>

```bash
openclaw doctor
```

<div id="headless-automation">
  ### Headless-/Automatisierungsbetrieb
</div>

```bash
openclaw doctor --yes
```

Standardwerte ohne Rückfrage akzeptieren (einschließlich Neustart-, Dienst- und sandbox-Reparaturschritten, sofern zutreffend).

```bash
openclaw doctor --repair
```

Empfohlene Reparaturen ohne Rückfrage anwenden (Reparaturen + Neustarts, sofern sicher).

```bash
openclaw doctor --repair --force
```

Auch aggressive Reparaturmaßnahmen anwenden (überschreibt benutzerdefinierte Supervisor-Konfigurationen).

```bash
openclaw doctor --non-interactive
```

Ohne Rückfragen ausführen und nur sichere Migrationen anwenden (Konfigurationsnormalisierung + Verschiebungen des Zustands auf dem Datenträger). Überspringt Neustart-, Dienst- und sandbox-Aktionen, die eine manuelle Bestätigung erfordern.
Legacy-Zustandsmigrationen werden automatisch ausgeführt, sobald sie erkannt werden.

```bash
openclaw doctor --deep
```

Systemdienste nach zusätzlichen Gateway-Installationen durchsuchen (launchd/systemd/schtasks).

Wenn du dir die Änderungen vor dem Anwenden ansehen möchtest, öffne zuerst die Konfigurationsdatei:

```bash
cat ~/.openclaw/openclaw.json
```

<div id="what-it-does-summary">
  ## Was es tut (Zusammenfassung)
</div>

* Optionales Preflight-Update für git-Installationen (nur interaktiv).
* UI-Protokoll-Aktualitätsprüfung (baut die Control UI neu, wenn das Protokollschema neuer ist).
* Health-Check + Neustart-Aufforderung.
* Fähigkeiten-Statusübersicht (berechtigt/fehlend/blockiert).
* Konfig-Normalisierung für Legacy-Werte.
* OpenCode-Zen-Anbieter-Override-Warnungen (`models.providers.opencode`).
* Migration von Legacy-Zustand auf Datenträger (Sitzungen/agent-Verzeichnis/WhatsApp-Auth).
* Prüfungen von Zustandsintegrität und Berechtigungen (Sitzungen, Transkripte, Zustandsverzeichnis).
* Prüfungen der Konfigurationsdatei-Berechtigungen (chmod 600) bei lokaler Ausführung.
* Zustand der Modell-Authentifizierung: prüft OAuth-Ablauf, kann ablaufende Tokens erneuern und meldet Cooldown-/Deaktiviert-Zustände von Auth-Profilen.
* Erkennung zusätzlicher Arbeitsbereichsverzeichnisse (`~/openclaw`).
* Reparatur des Sandbox-Images, wenn Sandboxing aktiviert ist.
* Migration von Legacy-Diensten und Erkennung zusätzlicher Gateways.
* Gateway-Laufzeitprüfungen (Dienst installiert, aber nicht laufend; zwischengespeichertes launchd-Label).
* Kanal-Statuswarnungen (vom laufenden Gateway ermittelt).
* Überprüfung der Supervisor-Konfiguration (launchd/systemd/schtasks) mit optionaler Reparatur.
* Best-Practice-Prüfungen für Gateway-Laufzeit (Node vs Bun, Version-Manager-Pfade).
* Gateway-Portkollisionsdiagnose (Standard `18789`).
* Sicherheitswarnungen für offene DM-Richtlinien.
* Gateway-Auth-Warnungen, wenn kein `gateway.auth.token` gesetzt ist (lokaler Modus; bietet Token-Generierung an).
* systemd-Linger-Prüfung unter Linux.
* Prüfungen für Source-Installationen (pnpm-Arbeitsbereichs-Abweichung, fehlende UI-Assets, fehlendes tsx-Binary).
* Schreibt aktualisierte Konfiguration + Wizard-Metadaten.

<div id="detailed-behavior-and-rationale">
  ## Detailliertes Verhalten und Hintergründe
</div>

<div id="0-optional-update-git-installs">
  ### 0) Optionales Update (Git-Installationen)
</div>

Falls es sich um ein Git-Checkout handelt und `doctor` interaktiv ausgeführt wird, wird angeboten, vor dem Ausführen von `doctor` ein Update (fetch/rebase/build) durchzuführen.

<div id="1-config-normalization">
  ### 1) Konfigurationsnormalisierung
</div>

Wenn die Konfiguration veraltete Wertformate enthält (zum Beispiel `messages.ackReaction`
ohne kanalspezifische Überschreibung), normalisiert `doctor` sie auf das aktuelle
Schema.

<div id="2-legacy-config-key-migrations">
  ### 2) Migration veralteter Config-Keys
</div>

Wenn die Config veraltete Keys enthält, verweigern andere Befehle die Ausführung
und fordern dich auf, `openclaw doctor` auszuführen.

`openclaw doctor` wird:

* erklären, welche veralteten Keys gefunden wurden.
* die angewendete Migration anzeigen.
* `~/.openclaw/openclaw.json` mit dem aktualisierten Schema neu schreiben.

Das Gateway führt beim Start außerdem automatisch Doctor-Migrationen aus, wenn es
ein veraltetes Config-Format erkennt, sodass veraltete Configs ohne manuellen Eingriff
repariert werden.

Aktuelle Migrationen:

* `routing.allowFrom` → `channels.whatsapp.allowFrom`
* `routing.groupChat.requireMention` → `channels.whatsapp/telegram/imessage.groups."*".requireMention`
* `routing.groupChat.historyLimit` → `messages.groupChat.historyLimit`
* `routing.groupChat.mentionPatterns` → `messages.groupChat.mentionPatterns`
* `routing.queue` → `messages.queue`
* `routing.bindings` → Top-Level-`bindings`
* `routing.agents`/`routing.defaultAgentId` → `agents.list` + `agents.list[].default`
* `routing.agentToAgent` → `tools.agentToAgent`
* `routing.transcribeAudio` → `tools.media.audio.models`
* `bindings[].match.accountID` → `bindings[].match.accountId`
* `identity` → `agents.list[].identity`
* `agent.*` → `agents.defaults` + `tools.*` (tools/elevated/exec/sandbox/subagents)
* `agent.model`/`allowedModels`/`modelAliases`/`modelFallbacks`/`imageModelFallbacks`
  → `agents.defaults.models` + `agents.defaults.model.primary/fallbacks` + `agents.defaults.imageModel.primary/fallbacks`

<div id="2b-opencode-zen-provider-overrides">
  ### 2b) OpenCode Zen-Anbieter-Overrides
</div>

Wenn du `models.providers.opencode` (oder `opencode-zen`) manuell hinzugefügt
hast, überschreibt das den integrierten OpenCode Zen-Katalog aus
`@mariozechner/pi-ai`. Das kann dazu führen, dass jedes Modell über eine einzige
API läuft oder die Kosten auf null gesetzt werden. Doctor warnt dich, damit du
das Override entfernen und das API-Routing und die Kosten pro Modell
wiederherstellen kannst.

<div id="3-legacy-state-migrations-disk-layout">
  ### 3) Legacy-Zustandsmigrationen (Speicherlayout)
</div>

Doctor kann ältere On-Disk-Layouts in die aktuelle Struktur migrieren:

* Sitzungs-Store + Transkripte:
  * von `~/.openclaw/sessions/` nach `~/.openclaw/agents/<agentId>/sessions/`
* Agent-Verzeichnis:
  * von `~/.openclaw/agent/` nach `~/.openclaw/agents/<agentId>/agent/`
* WhatsApp-Authentifizierungsstatus (Baileys):
  * von Legacy `~/.openclaw/credentials/*.json` (außer `oauth.json`)
  * nach `~/.openclaw/credentials/whatsapp/<accountId>/...` (Standard-Account-ID: `default`)

Diese Migrationen erfolgen nach Best-Effort-Prinzip und sind idempotent; doctor gibt Warnungen aus,
wenn alte Verzeichnisse als Backups zurückgelassen werden. Das Gateway und die CLI migrieren beim Start
ebenfalls automatisch die alten Sitzungen und das Agent-Verzeichnis, sodass Verlauf/Auth/Modelle
im Pfad pro Agent landen, ohne dass ein manueller doctor-Lauf nötig ist. Die WhatsApp-Authentifizierung
wird bewusst nur über `openclaw doctor` migriert.

<div id="4-state-integrity-checks-session-persistence-routing-and-safety">
  ### 4) Integritätsprüfungen des Zustands (Sitzungspersistenz, Routing und Sicherheit)
</div>

Das Zustandsverzeichnis ist der operative Hirnstamm. Wenn es verschwindet, verlierst du
Sitzungen, Zugangsdaten, Logs und Konfiguration (sofern du keine Backups an anderer Stelle hast).

Doctor prüft:

* **Zustandsverzeichnis fehlt**: warnt vor katastrophalem Zustandsverlust, fordert dazu auf,
  das Verzeichnis neu zu erstellen, und erinnert dich daran, dass fehlende Daten nicht
  wiederhergestellt werden können.
* **Zustandsverzeichnis-Berechtigungen**: prüft Schreibbarkeit; bietet an, Berechtigungen zu
  reparieren (und gibt einen `chown`‑Hinweis aus, wenn eine Abweichung bei Besitzer/Gruppe
  erkannt wird).
* **Sitzungsverzeichnisse fehlen**: `sessions/` und das Sitzungs-Store-Verzeichnis sind
  erforderlich, um Verlauf zu persistieren und `ENOENT`‑Abstürze zu vermeiden.
* **Transkript‑Inkonsistenz**: warnt, wenn für aktuelle Sitzungseinträge Transkriptdateien fehlen.
* **Hauptsitzung „1‑Zeilen‑JSONL“**: markiert, wenn das Haupttranskript nur eine Zeile hat
  (Verlauf wird nicht fortgeschrieben).
* **Mehrere Zustandsverzeichnisse**: warnt, wenn mehrere `~/.openclaw`‑Ordner in
  Home‑Verzeichnissen existieren oder wenn `OPENCLAW_STATE_DIR` auf ein anderes Verzeichnis zeigt
  (Verlauf kann sich zwischen Installationen aufspalten).
* **Remote‑Modus‑Erinnerung**: wenn `gateway.mode=remote`, erinnert dich Doctor daran, Doctor
  auf dem Remote‑Host auszuführen (dort liegt der Zustand).
* **Config‑Datei‑Berechtigungen**: warnt, wenn `~/.openclaw/openclaw.json`
  für Gruppe/Welt lesbar ist, und bietet an, auf `600` zu verschärfen.

<div id="5-model-auth-health-oauth-expiry">
  ### 5) Status der Modell-Authentifizierung (OAuth-Ablauf)
</div>

Doctor untersucht OAuth-Profile im Auth-Store, warnt, wenn Token bald ablaufen
oder bereits abgelaufen sind, und kann sie bei Bedarf sicher aktualisieren. Wenn das
Anthropic Claude Code-Profil veraltet ist, schlägt er vor, `claude setup-token`
auszuführen (oder ein Setup-Token einzufügen). Aufforderungen zur Aktualisierung
erscheinen nur bei interaktiver Ausführung (TTY); `--non-interactive` überspringt
Aktualisierungsversuche.

Doctor meldet außerdem Auth-Profile, die vorübergehend nicht nutzbar sind, aufgrund von:

* kurzen Cooldowns (Rate-Limits/Timeouts/Auth-Fehlern)
* längeren Deaktivierungen (Abrechnungs-/Guthabenfehlern)

<div id="6-hooks-model-validation">
  ### 6) Validierung des Hook-Modells
</div>

Wenn `hooks.gmail.model` gesetzt ist, validiert doctor die Modellreferenz anhand
des Katalogs und der Allowlist und warnt, wenn sie nicht aufgelöst werden kann oder nicht zugelassen ist.

<div id="7-sandbox-image-repair">
  ### 7) Reparatur von Sandbox-Images
</div>

Wenn Sandboxing aktiviert ist, überprüft `doctor` die Docker-Images und bietet an, fehlende Images zu erstellen oder auf Legacy-Namen zu wechseln, wenn das aktuelle Image nicht vorhanden ist.

<div id="8-gateway-service-migrations-and-cleanup-hints">
  ### 8) Gateway-Dienstmigrationen und Bereinigungshinweise
</div>

Doctor erkennt veraltete Gateway-Dienste (launchd/systemd/schtasks) und
bietet an, sie zu entfernen und den OpenClaw-Dienst mit dem aktuellen Gateway-Port
zu installieren. Es kann auch nach zusätzlichen gateway-ähnlichen Diensten scannen und Bereinigungshinweise ausgeben.
OpenClaw-Gateway-Dienste mit Profilnamen gelten als reguläre (First-Class-)Dienste und werden nicht
als „zusätzlich“ markiert.

<div id="9-security-warnings">
  ### 9) Sicherheitswarnungen
</div>

Doctor gibt Warnungen aus, wenn ein Anbieter für Direktnachrichten (DMs) ohne Allowlist freigegeben ist oder wenn eine Richtlinie gefährlich konfiguriert ist.

<div id="10-systemd-linger-linux">
  ### 10) systemd linger (Linux)
</div>

Wenn das Gateway als systemd-User-Service läuft, stellt `doctor` sicher, dass Lingering aktiviert ist, damit das Gateway nach dem Abmelden weiterläuft.

<div id="11-skills-status">
  ### 11) Status der Fähigkeiten
</div>

Doctor gibt eine kurze Übersicht über die verfügbaren/fehlenden/blockierten Fähigkeiten für den aktuellen Arbeitsbereich aus.

<div id="12-gateway-auth-checks-local-token">
  ### 12) Gateway-Auth-Prüfungen (lokales Token)
</div>

Doctor gibt eine Warnung aus, wenn `gateway.auth` auf einem lokalen Gateway fehlt, und bietet an,
ein Token zu erzeugen. Verwende `openclaw doctor --generate-gateway-token`, um die
Token-Erzeugung in automatisierten Abläufen zu erzwingen.

<div id="13-gateway-health-check-restart">
  ### 13) Gateway-Healthcheck + Neustart
</div>

Doctor führt einen Healthcheck aus und bietet an, das Gateway neu zu starten, wenn es nicht einwandfrei läuft.

<div id="14-channel-status-warnings">
  ### 14) Kanalstatus-Warnungen
</div>

Wenn das Gateway fehlerfrei läuft, führt `doctor` eine Kanalstatusprüfung durch und meldet
Warnungen mit Vorschlägen zur Behebung.

<div id="15-supervisor-config-audit-repair">
  ### 15) Supervisor-Konfigurationsprüfung + Reparatur
</div>

Doctor prüft die installierte Supervisor-Konfiguration (launchd/systemd/schtasks) auf
fehlende oder veraltete Standardwerte (z. B. systemd-Abhängigkeiten von network-online
und Neustartverzögerung). Wenn eine Abweichung gefunden wird, empfiehlt das Tool ein Update
und kann die Service-Datei bzw. den Task auf die aktuellen Standardwerte neu schreiben.

Hinweise:

* `openclaw doctor` fragt nach einer Bestätigung, bevor die Supervisor-Konfiguration neu geschrieben wird.
* `openclaw doctor --yes` akzeptiert die Standard-Reparaturvorschläge.
* `openclaw doctor --repair` wendet empfohlene Korrekturen ohne Rückfragen an.
* `openclaw doctor --repair --force` überschreibt benutzerdefinierte Supervisor-Konfigurationen.
* Du kannst jederzeit eine vollständige Neuschreibung mit `openclaw gateway install --force` erzwingen.

<div id="16-gateway-runtime-port-diagnostics">
  ### 16) Gateway-Laufzeit und Portdiagnose
</div>

Doctor überprüft die Dienstlaufzeit (PID, letzter Exit-Status) und warnt, wenn
der Dienst installiert, aber nicht läuft. Außerdem prüft er auf Portkollisionen
am Gateway-Port (Standard `18789`) und meldet wahrscheinliche Ursachen (Gateway läuft bereits, SSH-Tunnel).

<div id="17-gateway-runtime-best-practices">
  ### 17) Best Practices für den Gateway-Runtime-Betrieb
</div>

Doctor warnt, wenn der Gateway-Dienst auf Bun oder einem versionsverwalteten Node-Pfad
(`nvm`, `fnm`, `volta`, `asdf` usw.) läuft. WhatsApp- und Telegram-Kanäle benötigen Node,
und Versionsmanager-Pfade können nach Upgrades kaputtgehen, weil der Dienst deine
Shell-Initialisierungsskripte nicht lädt. Doctor bietet an, auf eine systemweite
Node-Installation zu migrieren, wenn verfügbar (Homebrew/apt/choco).

<div id="18-config-write-wizard-metadata">
  ### 18) Config-Write + Wizard-Metadaten
</div>

Doctor speichert alle Konfigurationsänderungen dauerhaft und versieht die Config mit Wizard-Metadaten, um den Doctor-Durchlauf zu protokollieren.

<div id="19-workspace-tips-backup-memory-system">
  ### 19) Tipps zum Arbeitsbereich (Backup + Speichersystem)
</div>

Doctor schlägt ein Speichersystem für den Arbeitsbereich vor, wenn keins vorhanden ist, und gibt einen Backup‑Hinweis aus,
falls der Arbeitsbereich noch nicht unter Git versioniert ist.

Siehe [/concepts/agent-workspace](/de/concepts/agent-workspace) für eine vollständige Anleitung zur
Struktur des Arbeitsbereichs und zu Git‑Backups (empfohlen: privates GitHub oder GitLab).