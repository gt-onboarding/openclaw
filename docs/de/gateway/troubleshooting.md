---
title: Fehlerbehebung
summary: "Kurzanleitung zur Fehlerbehebung bei häufigen OpenClaw-Problemen"
read_when:
  - Untersuchung von Laufzeitproblemen oder -fehlern
---

<div id="troubleshooting">
  # Fehlerbehebung 🔧
</div>

Wenn OpenClaw nicht wie erwartet funktioniert, gehst du so bei der Fehlerbehebung vor.

Beginne mit den FAQ: [First 60 seconds](/de/help/faq#first-60-seconds-if-somethings-broken), wenn du nur eine schnelle Erstdiagnose brauchst. Diese Seite geht tiefer auf Laufzeitfehler und Diagnostik ein.

Anbieterspezifische Schnellzugriffe: [/channels/troubleshooting](/de/channels/troubleshooting)

<div id="status-diagnostics">
  ## Status &amp; Diagnostik
</div>

Schnelle Erstdiagnose-Befehle (in dieser Reihenfolge):

| Befehl | Welche Infos du erhältst | Wann du ihn verwendest |
|---|---|---|
| `openclaw status` | Lokale Zusammenfassung: OS + Update, Gateway-Erreichbarkeit/Modus, Dienst, Agenten/Sitzungen, Anbieter-Konfigurationszustand | Erste Prüfung, schneller Überblick |
| `openclaw status --all` | Vollständige lokale Diagnose (read-only, gut kopierbar/einfügbar, weitgehend sicher) inkl. letztem Log-Ausschnitt | Wenn du einen Debug-Report weitergeben musst |
| `openclaw status --deep` | Führt Gateway-Health-Checks aus (inkl. Anbieter-Probes; erfordert ein erreichbares Gateway) | Wenn „konfiguriert“ nicht automatisch „funktioniert“ bedeutet |
| `openclaw gateway probe` | Gateway-Discovery + Erreichbarkeit (lokale + entfernte Ziele) | Wenn du vermutest, dass du das falsche Gateway prüfst |
| `openclaw channels status --probe` | Fragt das laufende Gateway nach Kanalstatus (und führt optional Probes aus) | Wenn das Gateway erreichbar ist, Kanäle sich aber fehlerhaft verhalten |
| `openclaw gateway status` | Supervisor-Status (launchd/systemd/schtasks), Laufzeit-PID/Exit, letzter Gateway-Fehler | Wenn der Dienst „geladen aussieht“, aber nichts läuft |
| `openclaw logs --follow` | Live-Logs (bestes Signal für Laufzeitprobleme) | Wenn du den tatsächlichen Fehlergrund brauchst |

**Ausgaben teilen:** bevorzuge `openclaw status --all` (Token werden geschwärzt). Wenn du `openclaw status` einfügst, setze vorher am besten `OPENCLAW_SHOW_SECRETS=0` (Token-Vorschauen).

Siehe auch: [Health-Checks](/de/gateway/health) und [Logging](/de/logging).

<div id="common-issues">
  ## Häufige Probleme
</div>

<div id="no-api-key-found-for-provider-anthropic">
  ### Kein API-Schlüssel für Anbieter &quot;anthropic&quot; gefunden
</div>

Das bedeutet, dass **der Auth-Store des Agents leer ist** oder Anthropic-Zugangsdaten fehlen.
Auth ist **pro Agent**, daher übernimmt ein neuer Agent nicht die Schlüssel des Haupt-Agents.

Mögliche Lösungen:

* Onboarding erneut ausführen und **Anthropic** für diesen Agent auswählen.
* Oder ein Setup-Token auf dem **Gateway-Host** einfügen:
  ```bash
  openclaw models auth setup-token --provider anthropic
  ```
* Oder `auth-profiles.json` aus dem Verzeichnis des Haupt-Agents in das Verzeichnis des neuen Agents kopieren.

Überprüfen:

```bash
openclaw models status
```

<div id="oauth-token-refresh-failed-anthropic-claude-subscription">
  ### OAuth-Token-Aktualisierung fehlgeschlagen (Anthropic-Claude-Abonnement)
</div>

Das bedeutet, dass der gespeicherte Anthropic-OAuth-Token abgelaufen ist und die Aktualisierung fehlgeschlagen hat.
Wenn du ein Claude-Abonnement verwendest (kein API-Schlüssel), ist die zuverlässigste Lösung,
auf ein **Claude Code setup-token** zu wechseln und dieses auf dem **Gateway-Host** einzutragen.

**Empfohlen (setup-token):**

```bash
# Auf dem Gateway-Host ausführen (setup-token einfügen)
openclaw models auth setup-token --provider anthropic
openclaw models status
```

Wenn du das Token anderweitig generiert hast:

```bash
openclaw models auth paste-token --provider anthropic
openclaw models status
```

Weitere Details: [Anthropic](/de/providers/anthropic) und [OAuth](/de/concepts/oauth).

<div id="control-ui-fails-on-http-device-identity-required-connect-failed">
  ### Control UI funktioniert über HTTP nicht (&quot;device identity required&quot; / &quot;connect failed&quot;)
</div>

Wenn du das Dashboard über reines HTTP öffnest (z.B. `http://<lan-ip>:18789/` oder
`http://<tailscale-ip>:18789/`), läuft der Browser in einem **unsicheren Kontext**
und blockiert WebCrypto, sodass keine Geräteidentität generiert werden kann.

**Lösung:**

* Verwende nach Möglichkeit HTTPS über [Tailscale Serve](/de/gateway/tailscale).
* Oder öffne das Dashboard lokal auf dem Gateway-Host: `http://127.0.0.1:18789/`.
* Wenn du weiterhin HTTP verwenden musst, aktiviere `gateway.controlUi.allowInsecureAuth: true` und
  verwende ein Gateway-Token (nur Token; keine Geräteidentität/kopplung). Siehe
  [Control UI](/de/web/control-ui#insecure-http).

<div id="ci-secrets-scan-failed">
  ### CI-Secrets-Scan fehlgeschlagen
</div>

Das bedeutet, `detect-secrets` hat neue Kandidaten gefunden, die noch nicht in der Baseline erfasst sind.
Befolge die Anleitung unter [Secret scanning](/de/gateway/security#secret-scanning-detect-secrets).

<div id="service-installed-but-nothing-is-running">
  ### Dienst installiert, aber es läuft nichts
</div>

Wenn der Gateway-Dienst installiert ist, der Prozess aber sofort beendet wird, kann
der Dienst als „geladen“ angezeigt werden, obwohl nichts läuft.

**Prüfen:**

```bash
openclaw gateway status
openclaw doctor
```

Doctor/Service zeigt den Laufzeitstatus (PID/letzter Exit-Code) und Log-Hinweise an.

**Logs:**

* Bevorzugt: `openclaw logs --follow`
* Datei-Logs (immer): `/tmp/openclaw/openclaw-YYYY-MM-DD.log` (oder dein konfiguriertes `logging.file`)
* macOS LaunchAgent (falls installiert): `$OPENCLAW_STATE_DIR/logs/gateway.log` und `gateway.err.log`
* Linux systemd (falls installiert): `journalctl --user -u openclaw-gateway[-<profile>].service -n 200 --no-pager`
* Windows: `schtasks /Query /TN "OpenClaw Gateway (<profile>)" /V /FO LIST`

**Mehr Logging aktivieren:**

* Detailgrad der Datei-Logs erhöhen (persistiertes JSONL):
  ```json
  { "logging": { "level": "debug" } }
  ```
* Ausführlichkeit der Konsolenausgabe erhöhen (nur TTY-Ausgabe):
  ```json
  { "logging": { "consoleLevel": "debug", "consoleStyle": "pretty" } }
  ```
* Kurzer Tipp: `--verbose` beeinflusst nur die **Konsolen**ausgabe. Datei-Logs werden weiterhin durch `logging.level` gesteuert.

Siehe [/logging](/de/logging) für einen vollständigen Überblick über Formate, Konfiguration und Zugriff.

<div id="gateway-start-blocked-set-gatewaymodelocal">
  ### &quot;Gateway-Start blockiert: gateway.mode=local setzen&quot;
</div>

Das bedeutet, dass die Konfigurationsdatei existiert, aber `gateway.mode` nicht gesetzt ist (oder nicht `local`), sodass das Gateway den Start verweigert.

**Behebung (empfohlen):**

* Führe den Assistenten aus und setze den Gateway-Ausführungsmodus auf **Local**:
  ```bash
  openclaw configure
  ```
* Oder setze ihn direkt:
  ```bash
  openclaw config set gateway.mode local
  ```

**Wenn du stattdessen ein Remote-Gateway ausführen wolltest:**

* Setze eine Remote-URL und belasse `gateway.mode=remote`:
  ```bash
  openclaw config set gateway.mode remote
  openclaw config set gateway.remote.url "wss://gateway.example.com"
  ```

**Nur ad hoc/Dev:** Übergib `--allow-unconfigured`, um das Gateway ohne
`gateway.mode=local` zu starten.

**Noch keine Konfigurationsdatei?** Führe `openclaw setup` aus, um eine Starter-Konfiguration zu erstellen, und starte
das Gateway dann erneut.

<div id="service-environment-path-runtime">
  ### Service Environment (PATH + runtime)
</div>

Der Gateway-Dienst läuft mit einem **minimalen PATH**, um Shell-/Manager-Ballast zu vermeiden:

* macOS: `/opt/homebrew/bin`, `/usr/local/bin`, `/usr/bin`, `/bin`
* Linux: `/usr/local/bin`, `/usr/bin`, `/bin`

Dies schließt bewusst Versionsmanager (nvm/fnm/volta/asdf) und Paketmanager
(pnpm/npm) aus, da der Dienst deine Shell-Initialisierung nicht lädt. Laufzeitvariablen
wie `DISPLAY` sollten in `~/.openclaw/.env` liegen (wird frühzeitig vom Gateway geladen).
Exec-Läufe auf `host=gateway` übernehmen den `PATH` deiner Login-Shell in die Exec-Umgebung,
sodass fehlende Tools in der Regel bedeuten, dass deine Shell-Initialisierung sie nicht exportiert (oder setze
`tools.exec.pathPrepend`). Siehe [/tools/exec](/de/tools/exec).

WhatsApp- und Telegram-Kanäle erfordern **Node**; Bun wird nicht unterstützt. Wenn dein
Dienst mit Bun oder einem versionsverwalteten Node-Pfad installiert wurde, führe `openclaw doctor`
aus, um auf eine System-Node-Installation zu migrieren.

<div id="skill-missing-api-key-in-sandbox">
  ### Skill fehlt API-Schlüssel in sandbox
</div>

**Symptom:** Skill funktioniert auf dem Host, schlägt aber in der sandbox mit fehlendem API-Schlüssel fehl.

**Warum:** sandboxed exec läuft innerhalb von Docker und übernimmt **nicht** die `process.env` des Hosts.

**Lösung:**

* setze `agents.defaults.sandbox.docker.env` (oder pro Agent `agents.list[].sandbox.docker.env`)
* oder hinterlege den Schlüssel direkt in deinem benutzerdefinierten sandbox-Image
* führe anschließend `openclaw sandbox recreate --agent <id>` (oder `--all`) aus

<div id="service-running-but-port-not-listening">
  ### Dienst läuft, aber Port lauscht nicht
</div>

Wenn der Dienst den Status **running** meldet, aber auf dem Gateway-Port nichts lauscht,
hat das Gateway vermutlich das Binden verweigert.

**Was „running“ hier bedeutet**

* `Runtime: running` bedeutet, dass dein Supervisor (launchd/systemd/schtasks) davon ausgeht, dass der Prozess läuft.
* `RPC probe` bedeutet, dass die CLI sich tatsächlich mit dem Gateway-WebSocket verbinden und `status` aufrufen konnte.
* Verlass dich immer auf `Probe target:` + `Config (service):` als die Zeilen „was haben wir tatsächlich versucht?“.

**Prüfen:**

* `gateway.mode` muss `local` sein – sowohl für `openclaw gateway` als auch für den Dienst.
* Wenn du `gateway.mode=remote` gesetzt hast, verwendet die **CLI-Standardkonfiguration** eine Remote-URL. Der Dienst kann trotzdem lokal laufen, aber deine CLI könnte an der falschen Stelle prüfen. Verwende `openclaw gateway status`, um den vom Dienst aufgelösten Port + Probe-Target zu sehen (oder übergib `--url`).
* `openclaw gateway status` und `openclaw doctor` zeigen den **letzten Gateway-Fehler** aus den Logs an, wenn der Dienst „running“ aussieht, aber der Port geschlossen ist.
* Nicht-Loopback-Binds (`lan`/`tailnet`/`custom` oder `auto`, wenn Loopback nicht verfügbar ist) erfordern Authentifizierung:
  `gateway.auth.token` (oder `OPENCLAW_GATEWAY_TOKEN`).
* `gateway.remote.token` ist nur für Remote-CLI-Aufrufe; es aktiviert **keine** lokale Authentifizierung.
* `gateway.token` wird ignoriert; verwende `gateway.auth.token`.

**Wenn `openclaw gateway status` eine Konfigurationsabweichung anzeigt**

* `Config (cli): ...` und `Config (service): ...` sollten normalerweise übereinstimmen.
* Wenn nicht, bearbeitest du mit sehr hoher Wahrscheinlichkeit eine Konfiguration, während der Dienst eine andere verwendet.
* Lösung: Führe `openclaw gateway install --force` erneut aus, und zwar aus demselben `--profile` / `OPENCLAW_STATE_DIR`, das der Dienst verwenden soll.

**Wenn `openclaw gateway status` Probleme mit der Dienstkonfiguration meldet**

* Die Supervisor-Konfiguration (launchd/systemd/schtasks) enthält nicht die aktuellen Standardwerte.
* Lösung: Führe `openclaw doctor` aus, um sie zu aktualisieren (oder `openclaw gateway install --force` für ein vollständiges Überschreiben).

**Wenn `Last gateway error:` „refusing to bind … without auth“ erwähnt**

* Du hast `gateway.bind` auf einen Nicht-Loopback-Modus gesetzt (`lan`/`tailnet`/`custom` oder `auto`, wenn Loopback nicht verfügbar ist), aber keine Authentifizierung konfiguriert.
* Lösung: Setze `gateway.auth.mode` + `gateway.auth.token` (oder exportiere `OPENCLAW_GATEWAY_TOKEN`) und starte den Dienst neu.

**Wenn `openclaw gateway status` `bind=tailnet` anzeigt, aber keine Tailnet-Schnittstelle gefunden wurde**

* Das Gateway hat versucht, an eine Tailscale-IP (100.64.0.0/10) zu binden, aber auf dem Host wurde keine erkannt.
* Lösung: Starte Tailscale auf dieser Maschine (oder ändere `gateway.bind` auf `loopback`/`lan`).

**Wenn `Probe note:` sagt, dass die Probe Loopback verwendet**

* Das ist bei `bind=lan` zu erwarten: Das Gateway lauscht auf `0.0.0.0` (alle Schnittstellen), und Loopback sollte lokal weiterhin verbinden.
* Für Remote-Clients verwende eine echte LAN-IP (nicht `0.0.0.0`) plus den Port und stelle sicher, dass Authentifizierung konfiguriert ist.

<div id="address-already-in-use-port-18789">
  ### Adresse wird bereits verwendet (Port 18789)
</div>

Das bedeutet, dass der Gateway-Port bereits von einem anderen Prozess verwendet wird.

**Überprüfen:**

```bash
openclaw gateway status
```

Es zeigt dir den/die Listener und wahrscheinliche Ursachen dafür an (Gateway läuft bereits, SSH-Tunnel).
Falls nötig, stoppe den Dienst oder wähle einen anderen Port.

<div id="extra-workspace-folders-detected">
  ### Zusätzliche Arbeitsbereichsordner erkannt
</div>

Wenn du von einer älteren Installation aktualisiert hast, hast du möglicherweise noch `~/openclaw` auf der Festplatte.
Mehrere Arbeitsbereichsverzeichnisse können zu verwirrenden Authentifizierungs- oder Zustandsabweichungen führen, da
immer nur ein einzelner Arbeitsbereich aktiv ist.

**Lösung:** Halte nur einen Arbeitsbereich aktiv und archiviere oder entferne die übrigen. Siehe
[Agent-Arbeitsbereich](/de/concepts/agent-workspace#extra-workspace-folders).

<div id="main-chat-running-in-a-sandbox-workspace">
  ### Hauptchat läuft in einem sandbox-Arbeitsbereich
</div>

Symptome: `pwd` oder Datei‑Tools zeigen `~/.openclaw/sandboxes/...`, obwohl du
den Host‑Arbeitsbereich erwartet hast.

**Warum:** `agents.defaults.sandbox.mode: "non-main"` orientiert sich an `session.mainKey` (Standardwert `"main"`).
Gruppen-/Kanal‑Sitzungen verwenden ihre eigenen Schlüssel, daher werden sie als „non-main“ behandelt und
erhalten sandbox-Arbeitsbereiche.

**Mögliche Lösungen:**

* Wenn du Host‑Arbeitsbereiche für einen agent möchtest: setze `agents.list[].sandbox.mode: "off"`.
* Wenn du Host‑Arbeitsbereichszugriff innerhalb der sandbox möchtest: setze `workspaceAccess: "rw"` für diesen agent.

<div id="agent-was-aborted">
  ### &quot;Agent wurde abgebrochen&quot;
</div>

Der Agent wurde während der Antwortausgabe unterbrochen.

**Ursachen:**

* Benutzer hat `stop`, `abort`, `esc`, `wait` oder `exit` eingegeben
* Zeitüberschreitung
* Prozess abgestürzt

**Lösung:** Sende einfach eine weitere Nachricht. Die Sitzung läuft weiter.

<div id="agent-failed-before-reply-unknown-model-anthropicclaude-haiku-3-5">
  ### &quot;Agent failed before reply: Unknown model: anthropic/claude-haiku-3-5&quot;
</div>

OpenClaw lehnt **ältere/unsichere Modelle** bewusst ab (insbesondere solche,
die besonders anfällig für Prompt-Injection sind). Wenn dieser Fehler auftritt,
wird der Modellname nicht mehr unterstützt.

**Behebung:**

* Wähle ein **aktuelles** Modell für den Anbieter und aktualisiere deine Config
  oder deinen Modellalias.
* Wenn du dir nicht sicher bist, welche Modelle verfügbar sind, führe
  `openclaw models list` oder `openclaw models scan` aus und wähle ein
  unterstütztes Modell.
* Überprüfe die Gateway-Logs für den detaillierten Fehlergrund.

Siehe auch: [Models CLI](/de/cli/models) und [Model providers](/de/concepts/model-providers).

<div id="messages-not-triggering">
  ### Nachrichten lösen nichts aus
</div>

**Prüfung 1:** Ist der Absender in der Allowlist eingetragen?

```bash
openclaw status
```

Suche in der Ausgabe nach `AllowFrom: ...`.

**Check 2:** Ist in Gruppenchats eine Erwähnung erforderlich?

```bash
# Die Nachricht muss mit mentionPatterns oder expliziten Erwähnungen übereinstimmen; Standardwerte liegen in Kanal-Gruppen/Gilden.
# Multi-Agent: `agents.list[].groupChat.mentionPatterns` überschreibt globale Muster.
grep -n "agents\\|groupChat\\|mentionPatterns\\|channels\\.whatsapp\\.groups\\|channels\\.telegram\\.groups\\|channels\\.imessage\\.groups\\|channels\\.discord\\.guilds" \
  "${OPENCLAW_CONFIG_PATH:-$HOME/.openclaw/openclaw.json}"
```

**Check 3:** Logs prüfen

```bash
openclaw logs --follow
# oder für schnelle Filter:
tail -f "$(ls -t /tmp/openclaw/openclaw-*.log | head -1)" | grep "blocked\\|skip\\|unauthorized"
```

<div id="pairing-code-not-arriving">
  ### Kopplungscode kommt nicht an
</div>

Wenn `dmPolicy` auf `pairing` gesetzt ist, sollten unbekannte Absender einen Code erhalten und ihre Nachricht wird ignoriert, bis sie genehmigt wird.

**Check 1:** Wartet bereits eine ausstehende Anfrage?

```bash
openclaw pairing list <channel>
```

Ausstehende DM-Kopplungsanfragen sind standardmäßig auf **3 pro Kanal** begrenzt. Wenn die Liste voll ist, wird für neue Anfragen kein Code generiert, bis eine genehmigt wird oder abläuft.

**Check 2:** Wurde die Anfrage erstellt, aber es wurde keine Antwort gesendet?

```bash
openclaw logs --follow | grep "pairing request"
```

**Prüfung 3:** Stelle sicher, dass `dmPolicy` für diesen Kanal nicht auf `open` (Einstellung, die die uneingeschränkte Annahme von Nachrichten von beliebigen Nutzern erlaubt) oder `allowlist` gesetzt ist.

<div id="image-mention-not-working">
  ### Bild + Erwähnung funktioniert nicht
</div>

Bekannter Fehler: Wenn du ein Bild nur mit einer Erwähnung (ohne weiteren Text) sendest, enthält WhatsApp die Erwähnungs-Metadaten manchmal nicht.

**Workaround:** Füge etwas Text zusammen mit der Erwähnung hinzu:

* ❌ `@openclaw` + Bild
* ✅ `@openclaw check this` + Bild

<div id="session-not-resuming">
  ### Sitzung wird nicht wieder aufgenommen
</div>

**Prüfung 1:** Ist die Sitzungsdatei vorhanden?

```bash
ls -la ~/.openclaw/agents/<agentId>/sessions/
```

**Check 2:** Ist das Reset-Zeitfenster zu kurz?

```json
{
  "session": {
    "reset": {
      "mode": "daily",
      "atHour": 4,
      "idleMinutes": 10080  // 7 Tage
    }
  }
}
```

**Check 3:** Hat jemand `/new`, `/reset` oder einen Reset-Trigger ausgelöst?

<div id="agent-timing-out">
  ### Agent-Timeout
</div>

Das Standard-Timeout liegt bei 30 Minuten. Bei langen Tasks:

```json
{
  "reply": {
    "timeoutSeconds": 3600  // 1 Stunde
  }
}
```

Oder verwende das Tool `process`, um lange Befehle im Hintergrund auszuführen.

<div id="whatsapp-disconnected">
  ### WhatsApp-Verbindung getrennt
</div>

```bash
# Check local status (creds, sessions, queued events)
openclaw status
# Laufendes Gateway + Kanäle testen (WA-Verbindung + Telegram + Discord-APIs)
openclaw status --deep

# View recent connection events
openclaw logs --limit 200 | grep "connection\\|disconnect\\|logout"
```

**Lösung:** Normalerweise wird die Verbindung automatisch wiederhergestellt, sobald das Gateway läuft. Wenn du trotzdem feststeckst, starte den Gateway-Prozess neu (je nachdem, wie du ihn überwachst), oder führe ihn manuell mit ausführlicher Ausgabe aus:

```bash
openclaw gateway --verbose
```

Wenn du abgemeldet bist oder die Verknüpfung aufgehoben ist:

```bash
openclaw channels logout
trash "${OPENCLAW_STATE_DIR:-$HOME/.openclaw}/credentials" # falls logout nicht alles sauber entfernen kann
openclaw channels login --verbose       # re-scan QR
```

<div id="media-send-failing">
  ### Medienversand schlägt fehl
</div>

**Überprüfung 1:** Ist der Dateipfad gültig?

```bash
ls -la /path/to/your/image.jpg
```

**Prüfung 2:** Ist es zu groß?

* Bilder: max. 6 MB
* Audio/Video: max. 16 MB
* Dokumente: max. 100 MB

**Prüfung 3:** Medienprotokolle prüfen

```bash
grep "media\\|fetch\\|download" "$(ls -t /tmp/openclaw/openclaw-*.log | head -1)" | tail -20
```

<div id="high-memory-usage">
  ### Hohe Speicherauslastung
</div>

OpenClaw behält den Gesprächsverlauf im Speicher.

**Lösung:** Starten Sie OpenClaw regelmäßig neu oder legen Sie Grenzen für Sitzungen fest:

```json
{
  "session": {
    "historyLimit": 100  // Maximale Anzahl an Nachrichten, die behalten werden
  }
}
```

<div id="common-troubleshooting">
  ## Häufige Probleme und Lösungen
</div>

<div id="gateway-wont-start-configuration-invalid">
  ### „Gateway startet nicht – Konfiguration ist ungültig“
</div>

OpenClaw verweigert jetzt den Start, wenn die Konfiguration unbekannte Schlüssel, fehlerhafte Werte oder ungültige Datentypen enthält.
Dies ist aus Sicherheitsgründen beabsichtigt.

Behebe das Problem mit Doctor:

```bash
openclaw doctor
openclaw doctor --fix
```

Hinweise:

* `openclaw doctor` gibt jeden ungültigen Eintrag aus.
* `openclaw doctor --fix` führt Migrationen/Reparaturen durch und schreibt die Konfiguration neu.
* Diagnosebefehle wie `openclaw logs`, `openclaw health`, `openclaw status`, `openclaw gateway status` und `openclaw gateway probe` laufen auch dann noch, wenn die Konfiguration ungültig ist.

<div id="all-models-failed-what-should-i-check-first">
  ### „Alle Modelle sind fehlgeschlagen“ — was solltest du zuerst prüfen?
</div>

* **Credentials** (Zugangsdaten) sind für die verwendeten Anbieter vorhanden (Auth-Profile + Umgebungsvariablen).
* **Modellrouting**: Überprüfe, ob `agents.defaults.model.primary` und Fallbacks Modelle sind, auf die du Zugriff hast.
* **Gateway-Logs** in `/tmp/openclaw/…` auf den genauen Anbieterfehler.
* **Modellstatus**: Verwende `/model status` (Chat) oder `openclaw models status` (CLI).

<div id="im-running-on-my-personal-whatsapp-number-why-is-self-chat-weird">
  ### Ich nutze meine persönliche WhatsApp-Nummer – warum verhält sich der Self-Chat seltsam?
</div>

Aktiviere den Self-Chat-Modus und füge deine eigene Nummer zur Allowlist hinzu:

```json5
{
  channels: {
    whatsapp: {
      selfChatMode: true,
      dmPolicy: "allowlist",
      allowFrom: ["+15555550123"]
    }
  }
}
```

Siehe [WhatsApp-Einrichtung](/de/channels/whatsapp).

<div id="whatsapp-logged-me-out-how-do-i-reauth">
  ### WhatsApp hat mich ausgeloggt. Wie melde ich mich erneut an?
</div>

Führe den Login-Befehl erneut aus und scanne den QR-Code:

```bash
openclaw channels login
```

<div id="build-errors-on-main-whats-the-standard-fix-path">
  ### Build-Fehler im Branch `main` — was ist das Standard-Vorgehen zur Behebung?
</div>

1. `git pull origin main && pnpm install`
2. `openclaw doctor`
3. GitHub-Issues oder Discord überprüfen
4. Vorübergehender Workaround: einen älteren Commit auschecken

<div id="npm-install-fails-allow-build-scripts-missing-tar-or-yargs-what-now">
  ### npm install schlägt fehl (allow-build-scripts / fehlendes tar oder yargs). Was jetzt?
</div>

Wenn du aus dem Quellcode arbeitest, verwende den Paketmanager des Repos: **pnpm** (bevorzugt).
Das Repo deklariert `packageManager: "pnpm@…"`.

Typisches Vorgehen zur Wiederherstellung:

```bash
git status   # sicherstellen, dass Sie sich im Repository-Stammverzeichnis befinden
pnpm install
pnpm build
openclaw doctor
openclaw gateway restart
```

Warum: pnpm ist der für dieses Repo konfigurierte Paketmanager.

<div id="how-do-i-switch-between-git-installs-and-npm-installs">
  ### Wie wechsle ich zwischen Git-Installationen und npm-Installationen?
</div>

Verwende den **Website-Installer** und wähle die Installationsmethode per Flag aus. Er aktualisiert die bestehende Installation und passt den Gateway-Dienst so an, dass er auf die neue Installation verweist.

Wechsle **auf eine Git-Installation**:

```bash
curl -fsSL https://openclaw.bot/install.sh | bash -s -- --install-method git --no-onboard
```

Wechsle auf **npm global**:

```bash
curl -fsSL https://openclaw.bot/install.sh | bash
```

Hinweise:

* Der Git-Flow rebaset nur, wenn das Repository sauber ist. Committe deine Änderungen oder lege sie zunächst per Stash zurück.
* Nach dem Wechsel führe Folgendes aus:
  ```bash
  openclaw doctor
  openclaw gateway restart
  ```

<div id="telegram-block-streaming-isnt-splitting-text-between-tool-calls-why">
  ### Block-Streaming in Telegram teilt den Text nicht zwischen Tool-Aufrufen auf. Warum?
</div>

Block-Streaming sendet nur **abgeschlossene Textblöcke**. Häufige Gründe, warum du nur eine einzelne Nachricht siehst:

* `agents.defaults.blockStreamingDefault` ist noch `"off"`.
* `channels.telegram.blockStreaming` ist auf `false` gesetzt.
* `channels.telegram.streamMode` ist `partial` oder `block` **und Draft-Streaming ist aktiv**
  (Privatchat + Themen). Draft-Streaming deaktiviert in diesem Fall Block-Streaming.
* Deine `minChars`-/Coalesce-Einstellungen sind zu hoch, sodass Chunks zusammengeführt werden.
* Das Modell gibt einen einzigen großen Textblock aus (keine Flush-Punkte innerhalb der Antwort).

Checkliste zur Fehlerbehebung:

1. Platziere die Block-Streaming-Einstellungen unter `agents.defaults`, nicht in der Root-Ebene.
2. Setze `channels.telegram.streamMode: "off"`, wenn du echte, mehrteilige Antworten mit Blöcken möchtest.
3. Verwende kleinere Chunk-/Coalesce-Schwellenwerte beim Debuggen.

Siehe [Streaming](/de/concepts/streaming).

<div id="discord-doesnt-reply-in-my-server-even-with-requiremention-false-why">
  ### Discord antwortet auf meinem Server nicht, obwohl `requireMention: false` gesetzt ist. Warum?
</div>

`requireMention` steuert die Erwähnungspflicht nur **nachdem** der Channel die Allowlist-Prüfung durchlaufen hat.
Standardmäßig ist `channels.discord.groupPolicy` auf **allowlist** gesetzt, daher müssen Server (Guilds) explizit aktiviert werden.
Wenn du `channels.discord.guilds.<guildId>.channels` setzt, sind nur die aufgelisteten Channels erlaubt; lasse den Eintrag weg, um alle Channels in der Guild zuzulassen.

Checkliste zur Fehlerbehebung:

1. Setze `channels.discord.groupPolicy: "open"` **oder** füge einen Allowlist-Eintrag für die Guild hinzu (und optional eine Channel-Allowlist).
2. Verwende **numerische Channel-IDs** in `channels.discord.guilds.<guildId>.channels`.
3. Setze `requireMention: false` **unter** `channels.discord.guilds` (global oder pro Channel).
   Das Top-Level-Feld `channels.discord.requireMention` wird nicht unterstützt.
4. Stelle sicher, dass der Bot **Message Content Intent** und die erforderlichen Channel-Berechtigungen hat.
5. Führe `openclaw channels status --probe` aus, um Hinweise zur Analyse zu erhalten.

Dokumentation: [Discord](/de/channels/discord), [Channels – Fehlerbehebung](/de/channels/troubleshooting).

<div id="cloud-code-assist-api-error-invalid-tool-schema-400-what-now">
  ### Cloud Code Assist API-Fehler: invalid tool schema (400). Was jetzt?
</div>

Das ist fast immer ein Problem mit der **Tool-Schema-Kompatibilität**. Der Cloud Code Assist
Endpoint akzeptiert nur eine strikte Teilmenge von JSON Schema. OpenClaw bereinigt/normalisiert Tool‑Schemas im aktuellen `main`, aber der Fix ist (Stand
13. Januar 2026) noch nicht im letzten Release enthalten.

Checkliste zur Behebung:

1. **OpenClaw aktualisieren**:
   * Wenn du aus dem Quellcode laufen lassen kannst, `main` pullen und den Gateway neu starten.
   * Andernfalls auf das nächste Release warten, das den Schema‑Scrubber enthält.
2. Nicht unterstützte Schlüsselwörter wie `anyOf/oneOf/allOf`, `patternProperties`,
   `additionalProperties`, `minLength`, `maxLength`, `format` usw. vermeiden.
3. Wenn du eigene Tools definierst, lass das Top‑Level‑Schema als `type: "object"` mit
   `properties` und einfachen Enums.

Siehe [Tools](/de/tools) und [TypeBox-Schemas](/de/concepts/typebox).

<div id="macos-specific-issues">
  ## Spezifische Probleme unter macOS
</div>

<div id="app-crashes-when-granting-permissions-speechmic">
  ### App stürzt ab beim Erteilen von Berechtigungen (Sprache/Mikrofon)
</div>

Wenn die App verschwindet oder &quot;Abort trap 6&quot; anzeigt, sobald du bei einer Datenschutzabfrage auf &quot;Zulassen&quot; klickst:

**Lösung 1: TCC-Cache zurücksetzen**

```bash
tccutil reset All bot.molt.mac.debug
```

**Fix 2: Neue Bundle-ID erzwingen**
Wenn das Zurücksetzen nicht funktioniert, ändere die `BUNDLE_ID` in [`scripts/package-mac-app.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/package-mac-app.sh) (hänge z. B. ein `.test`-Suffix an) und baue die App neu. Dadurch behandelt macOS sie als neue App.

<div id="gateway-stuck-on-starting">
  ### Gateway bleibt bei „Starting...“ hängen
</div>

Die App stellt eine Verbindung zu einem lokalen Gateway auf Port `18789` her. Wenn sie dort hängen bleibt:

**Lösung 1: Supervisor stoppen (bevorzugt)**
Wenn das Gateway von launchd überwacht wird, führt das Beenden des Prozesses per PID nur dazu, dass es neu gestartet wird. Beende zuerst den Supervisor:

```bash
openclaw gateway status
openclaw gateway stop
# Oder: launchctl bootout gui/$UID/bot.molt.gateway (ersetze mit bot.molt.<profile>; legacy com.openclaw.* funktioniert weiterhin)
```

**Fix 2: Port ist belegt (ermittle den lauschenden Prozess)**

```bash
lsof -nP -iTCP:18789 -sTCP:LISTEN
```

Wenn es sich um einen unbeaufsichtigten Prozess handelt, versuche zuerst, ihn geordnet zu stoppen, und eskaliere dann:

```bash
kill -TERM <PID>
sleep 1
kill -9 <PID> # letztes Mittel
```

**Lösung 3: CLI-Installation überprüfen**
Stelle sicher, dass die systemweite `openclaw` CLI installiert ist und mit der App-Version übereinstimmt:

```bash
openclaw --version
npm install -g openclaw@<version>
```

<div id="debug-mode">
  ## Debug-Modus
</div>

Ausführliche Protokollausgabe:

```bash
# Trace-Logging in der Konfiguration aktivieren:
#   ${OPENCLAW_CONFIG_PATH:-$HOME/.openclaw/openclaw.json} -> { logging: { level: "trace" } }
#
# Anschließend Befehle mit --verbose ausführen, um Debug-Ausgabe nach stdout zu spiegeln:
openclaw gateway --verbose
openclaw channels login --verbose
```

<div id="log-locations">
  ## Protokollspeicherorte
</div>

| Protokoll | Speicherort |
|-----|----------|
| Gateway-Datei-Logs (strukturiert) | `/tmp/openclaw/openclaw-YYYY-MM-DD.log` (oder `logging.file`) |
| Gateway-Dienst-Logs (Supervisor) | macOS: `$OPENCLAW_STATE_DIR/logs/gateway.log` + `gateway.err.log` (Standard: `~/.openclaw/logs/...`; Profile verwenden `~/.openclaw-<profile>/logs/...`)<br />Linux: `journalctl --user -u openclaw-gateway[-<profile>].service -n 200 --no-pager`<br />Windows: `schtasks /Query /TN "OpenClaw Gateway (<profile>)" /V /FO LIST` |
| Sitzungsdateien | `$OPENCLAW_STATE_DIR/agents/<agentId>/sessions/` |
| Medien-Cache | `$OPENCLAW_STATE_DIR/media/` |
| Zugangsdaten | `$OPENCLAW_STATE_DIR/credentials/` |

<div id="health-check">
  ## Health-Check
</div>

```bash
# Supervisor + probe target + config paths
openclaw gateway status
# Systemweite Scans einbeziehen (Legacy-/zusätzliche Dienste, Port-Listener)
openclaw gateway status --deep

# Is the gateway reachable?
openclaw health --json
# If it fails, rerun with connection details:
openclaw health --verbose

# Is something listening on the default port?
lsof -nP -iTCP:18789 -sTCP:LISTEN

# Recent activity (RPC log tail)
openclaw logs --follow
# Fallback if RPC is down
tail -20 /tmp/openclaw/openclaw-*.log
```

<div id="reset-everything">
  ## Alles zurücksetzen
</div>

Der letzte Ausweg:

```bash
openclaw gateway stop
# Falls Sie einen Dienst installiert haben und eine Neuinstallation durchführen möchten:
# openclaw gateway uninstall

trash "${OPENCLAW_STATE_DIR:-$HOME/.openclaw}"
openclaw channels login         # WhatsApp erneut koppeln
openclaw gateway restart           # oder: openclaw gateway
```

⚠️ Dadurch gehen alle Sitzungen verloren und es ist eine erneute Kopplung von WhatsApp erforderlich.

<div id="getting-help">
  ## Hilfe erhalten
</div>

1. Prüfe zuerst die Logs unter: `/tmp/openclaw/` (Standard: `openclaw-YYYY-MM-DD.log` oder dein konfiguriertes `logging.file`)
2. Durchsuche vorhandene Issues auf GitHub
3. Eröffne ein neues Issue mit:
   * OpenClaw-Version
   * Relevanten Log-Auszügen
   * Schritten zur Reproduktion des Problems
   * Deiner Konfiguration (Secrets/sensible Daten unkenntlich machen!)

***

*&quot;Hast du schon versucht, es aus- und wieder einzuschalten?&quot;* — Jede IT-Person, jemals

🦞🔧

<div id="browser-not-starting-linux">
  ### Browser startet nicht (Linux)
</div>

Wenn die Meldung `"Failed to start Chrome CDP on port 18800"` erscheint:

**Wahrscheinlichste Ursache:** Als Snap verpacktes Chromium auf Ubuntu.

**Schnelle Lösung:** Installiere stattdessen Google Chrome:

```bash
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
sudo dpkg -i google-chrome-stable_current_amd64.deb
```

Dann in der Konfiguration setzen:

```json
{
  "browser": {
    "executablePath": "/usr/bin/google-chrome-stable"
  }
}
```

**Ausführliche Anleitung:** Siehe [browser-linux-troubleshooting](/de/tools/browser-linux-troubleshooting)
