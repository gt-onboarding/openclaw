---
title: Assistent
summary: "CLI-Onboarding-Assistent: geführte Einrichtung für Gateway, Arbeitsbereich, Kanäle und Fähigkeiten"
read_when:
  - Ausführen oder Konfigurieren des Onboarding-Assistenten
  - Einrichten eines neuen Rechners
---

<div id="onboarding-wizard-cli">
  # Onboarding-Assistent (CLI)
</div>

Der Onboarding-Assistent ist der **empfohlene** Weg, um OpenClaw auf macOS,
Linux oder Windows (über WSL2; dringend empfohlen) einzurichten.
Er konfiguriert ein lokales Gateway oder eine Remote-Gateway-Verbindung sowie Kanäle, Fähigkeiten
und Standardwerte für den Arbeitsbereich in einem geführten Ablauf.

Primärer Einstiegspunkt:

```bash
openclaw onboard
```

Am schnellsten zum ersten Chat: Öffne die Control UI (keine Channel-Einrichtung nötig). Führe
`openclaw dashboard` aus und chatte im Browser. Dokumentation: [Dashboard](/de/web/dashboard).

Nachträgliche Konfiguration:

```bash
openclaw configure
```

Empfohlen: Richte einen Brave-Search-API-Schlüssel ein, damit der agent `web_search` nutzen kann
(`web_fetch` funktioniert ohne Schlüssel). Einfachster Weg: `openclaw configure --section web`,
wodurch `tools.web.search.apiKey` gespeichert wird. Doku: [Web-Tools](/de/tools/web).

<div id="quickstart-vs-advanced">
  ## QuickStart vs Advanced
</div>

Der Wizard startet mit **QuickStart** (Standardwerte) vs **Advanced** (volle Kontrolle).

**QuickStart** behält die Standardwerte bei:

* Lokales Gateway (Loopback)
* Standard-Arbeitsbereich (oder bestehender Arbeitsbereich)
* Gateway-Port **18789**
* Gateway-Auth **Token** (automatisch generiert, auch bei Loopback)
* Tailscale-Freigabe **Aus**
* Telegram + WhatsApp-Direktnachrichten verwenden standardmäßig die **Allowlist** (du wirst nach deiner Telefonnummer gefragt)

**Advanced** führt dich durch jeden einzelnen Schritt (Modus, Arbeitsbereich, Gateway, Kanäle, Daemon, Fähigkeiten).

<div id="what-the-wizard-does">
  ## Was der Wizard macht
</div>

**Lokaler Modus (Standard)** führt dich durch:

* Modelle/Authentifizierung (OpenAI Code-(Codex)-Subscription-OAuth, Anthropic-API-Schlüssel (empfohlen) oder Setup-Token (einfügen), plus MiniMax-/GLM-/Moonshot-/AI-Gateway-Optionen)
* Speicherort des Arbeitsbereichs + Bootstrap-Dateien
* Gateway-Einstellungen (Port/Bind/Auth/Tailscale)
* Anbieter (Telegram, WhatsApp, Discord, Google Chat, Mattermost (Plugin), Signal)
* Daemon-Installation (LaunchAgent / systemd-User-Unit)
* Health-Check
* Fähigkeiten (empfohlen)

**Remote-Modus** konfiguriert nur den lokalen Client, damit er sich mit einem anderen Gateway verbindet.
Er **installiert oder ändert nichts** auf dem Remote-Host.

Um weitere isolierte Agenten hinzuzufügen (getrennter Arbeitsbereich + Sitzungen + Auth), verwende:

```bash
openclaw agents add <name>
```

Tipp: `--json` setzt **nicht** automatisch den nichtinteraktiven Modus. Verwenden Sie für Skripte `--non-interactive` (und `--workspace`).

<div id="flow-details-local">
  ## Ablaufdetails (lokal)
</div>

1. **Erkennung vorhandener Config**
   * Wenn `~/.openclaw/openclaw.json` existiert, wähle **Beibehalten / Anpassen / Zurücksetzen**.
   * Mehrfaches Ausführen des Wizards löscht **nichts**, außer du wählst explizit **Zurücksetzen**
     (oder übergibst `--reset`).
   * Wenn die Config ungültig ist oder Legacy-Keys enthält, stoppt der Wizard und fordert dich auf,
     `openclaw doctor` auszuführen, bevor du fortfährst.
   * Zurücksetzen verwendet `trash` (nie `rm`) und bietet folgende Scopes:
     * Nur Config
     * Config + Zugangsdaten + Sitzungen
     * Vollständiges Zurücksetzen (entfernt auch den Arbeitsbereich)

2. **Modell/Auth**
   * **Anthropic API-Schlüssel (empfohlen)**: verwendet `ANTHROPIC_API_KEY`, falls vorhanden, oder fragt nach einem Schlüssel und speichert ihn dann für die Daemon-Nutzung.
   * **Anthropic OAuth (Claude Code CLI)**: unter macOS prüft der Wizard den Keychain-Eintrag &quot;Claude Code-credentials&quot; (wähle &quot;Always Allow&quot;, damit launchd-Starts nicht blockieren); unter Linux/Windows wird `~/.claude/.credentials.json` wiederverwendet, falls vorhanden.
   * **Anthropic-Token (setup-token einfügen)**: führe `claude setup-token` auf einem beliebigen Rechner aus und füge dann das Token ein (du kannst es benennen; leer = Standard).
   * **OpenAI Code (Codex)-Abo (Codex CLI)**: wenn `~/.codex/auth.json` existiert, kann der Wizard es wiederverwenden.
   * **OpenAI Code (Codex)-Abo (OAuth)**: Browser-Flow; füge den `code#state` ein.
     * Setzt `agents.defaults.model` auf `openai-codex/gpt-5.2`, wenn das Modell nicht gesetzt ist oder `openai/*` lautet.
   * **OpenAI API-Schlüssel**: verwendet `OPENAI_API_KEY`, falls vorhanden, oder fragt nach einem Schlüssel und speichert ihn dann in `~/.openclaw/.env`, damit launchd ihn lesen kann.
   * **OpenCode Zen (Multi-Modell-Proxy)**: fragt nach `OPENCODE_API_KEY` (oder `OPENCODE_ZEN_API_KEY`, erhältlich unter https://opencode.ai/auth).
   * **API-Schlüssel**: speichert den Schlüssel für dich.
   * **Vercel AI Gateway (Multi-Modell-Proxy)**: fragt nach `AI_GATEWAY_API_KEY`.
   * Weitere Details: [Vercel AI Gateway](/de/providers/vercel-ai-gateway)
   * **MiniMax M2.1**: Die Config wird automatisch geschrieben.
   * Weitere Details: [MiniMax](/de/providers/minimax)
   * **Synthetic (Anthropic-kompatibel)**: fragt nach `SYNTHETIC_API_KEY`.
   * Weitere Details: [Synthetic](/de/providers/synthetic)
   * **Moonshot (Kimi K2)**: Die Config wird automatisch geschrieben.
   * **Kimi Code**: Die Config wird automatisch geschrieben.
   * Weitere Details: [Moonshot AI (Kimi + Kimi Code)](/de/providers/moonshot)
   * **Überspringen**: noch keine Auth konfiguriert.
   * Wähle ein Standardmodell aus den erkannten Optionen (oder gib Anbieter/Modell manuell ein).
   * Der Wizard führt eine Modellprüfung durch und warnt, wenn das konfigurierte Modell unbekannt ist oder Auth fehlt.

* OAuth-Zugangsdaten liegen in `~/.openclaw/credentials/oauth.json`; Auth-Profile liegen in `~/.openclaw/agents/<agentId>/agent/auth-profiles.json` (API-Schlüssel + OAuth).
  * Weitere Details: [/concepts/oauth](/de/concepts/oauth)

3. **Arbeitsbereich**
   * Standard `~/.openclaw/workspace` (konfigurierbar).
   * Initialisiert die Dateien im Arbeitsbereich, die für das Agent-Bootstrap-Ritual benötigt werden.
   * Vollständiges Layout des Arbeitsbereichs + Backup-Anleitung: [Agent workspace](/de/concepts/agent-workspace)

4. **Gateway**
   * Port, Bind-Adresse, Auth-Modus, Tailscale-Freigabe.
   * Auth-Empfehlung: **Token** beibehalten, selbst für Loopback, damit sich lokale WS-Clients authentifizieren müssen.
   * Deaktiviere Auth nur, wenn du jedem lokalen Prozess vollständig vertraust.
   * Non-Loopback-Binds erfordern weiterhin Authentifizierung.

5. **Kanäle**
   * [WhatsApp](/de/channels/whatsapp): optionaler QR-Login.
   * [Telegram](/de/channels/telegram): Bot-Token.
   * [Discord](/de/channels/discord): Bot-Token.
   * [Google Chat](/de/channels/googlechat): Service-Account-JSON + Webhook-Audience.
   * [Mattermost](/de/channels/mattermost) (Plugin): Bot-Token + Basis-URL.
   * [Signal](/de/channels/signal): optionale `signal-cli`-Installation + Konto-Config.
   * [iMessage](/de/channels/imessage): lokaler `imsg`-CLI-Pfad + DB-Zugriff.
   * DM-Sicherheit: Standard ist Kopplung. Die erste DM sendet einen Code; bestätige über `openclaw pairing approve <channel> <code>` oder verwende Allowlists.

6. **Daemon-Installation**
   * macOS: LaunchAgent
     * Erfordert eine eingeloggte Benutzersitzung; für Headless-Betrieb verwende einen eigenen LaunchDaemon (nicht mitgeliefert).
   * Linux (und Windows via WSL2): systemd-User-Unit
     * Der Assistent versucht, Lingering via `loginctl enable-linger <user>` zu aktivieren, damit das Gateway nach Logout weiterläuft.
     * Möglicherweise ist sudo erforderlich (schreibt nach `/var/lib/systemd/linger`); er versucht es zuerst ohne sudo.
   * **Laufzeitauswahl:** Node (empfohlen; erforderlich für WhatsApp/Telegram). Bun wird **nicht empfohlen**.

7. **Health-Check**
   * Startet das Gateway (falls nötig) und führt `openclaw health` aus.
   * Tipp: `openclaw status --deep` ergänzt die Statusausgabe um Gateway-Health-Checks (erfordert ein erreichbares Gateway).

8. **Fähigkeiten (empfohlen)**
   * Liest die verfügbaren Fähigkeiten und prüft Anforderungen.
   * Lässt dich einen Node-Manager wählen: **npm / pnpm** (bun nicht empfohlen).
   * Installiert optionale Abhängigkeiten (einige verwenden Homebrew auf macOS).

9. **Fertig**
   * Zusammenfassung + nächste Schritte, inklusive iOS/Android/macOS-Apps für zusätzliche Features.

* Wenn keine GUI erkannt wird, gibt der Assistent SSH-Port-Forward-Anweisungen für die Control UI aus, anstatt einen Browser zu öffnen.
  * Wenn die Control-UI-Assets fehlen, versucht der Assistent, diese zu bauen; Fallback ist `pnpm ui:build` (installiert UI-Abhängigkeiten automatisch).

<div id="remote-mode">
  ## Remote-Modus
</div>

Der Remote-Modus konfiguriert einen lokalen Client, um eine Verbindung zu einem entfernten Gateway herzustellen.

Was du einstellst:

* Remote-Gateway-URL (`ws://...`)
* Token, falls das entfernte Gateway Authentifizierung erfordert (empfohlen)

Hinweise:

* Es werden keine Remote-Installationen oder Änderungen an Daemons vorgenommen.
* Wenn das Gateway nur über Loopback erreichbar ist, verwende SSH-Tunneling oder ein Tailnet.
* Hinweise zur Discovery:
  * macOS: Bonjour (`dns-sd`)
  * Linux: Avahi (`avahi-browse`)

<div id="add-another-agent">
  ## Weiteren agent hinzufügen
</div>

Verwende `openclaw agents add <name>`, um einen separaten agent mit eigenem arbeitsbereich,
Sitzungen und Auth‑Profilen zu erstellen. Wenn du ohne `--workspace` ausführst, startet der Assistent.

Was damit konfiguriert wird:

* `agents.list[].name`
* `agents.list[].workspace`
* `agents.list[].agentDir`

Hinweise:

* Standard‑arbeitsbereiche haben das Schema `~/.openclaw/workspace-<agentId>`.
* Füge `bindings` hinzu, um eingehende Nachrichten weiterzuleiten (der Assistent kann das übernehmen).
* Nicht‑interaktive Flags: `--model`, `--agent-dir`, `--bind`, `--non-interactive`.

<div id="noninteractive-mode">
  ## Nicht-interaktiver Modus
</div>

Verwende `--non-interactive`, um das Onboarding zu automatisieren oder zu skripten:

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice apiKey \
  --anthropic-api-key "$ANTHROPIC_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback \
  --install-daemon \
  --daemon-runtime node \
  --skip-skills
```

Fügen Sie `--json` für eine maschinenlesbare Zusammenfassung hinzu.

Gemini‑Beispiel:

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice gemini-api-key \
  --gemini-api-key "$GEMINI_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

Beispiel für Z.AI:

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice zai-api-key \
  --zai-api-key "$ZAI_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

Beispiel für das Vercel AI Gateway:

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice ai-gateway-api-key \
  --ai-gateway-api-key "$AI_GATEWAY_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

Moonshot-Beispiel:

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice moonshot-api-key \
  --moonshot-api-key "$MOONSHOT_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

Synthetisches Beispiel:

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice synthetic-api-key \
  --synthetic-api-key "$SYNTHETIC_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

Beispiel für OpenCode Zen:

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice opencode-zen \
  --opencode-zen-api-key "$OPENCODE_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

Beispiel für das Hinzufügen eines agents (nicht interaktiv):

```bash
openclaw agents add work \
  --workspace ~/.openclaw/workspace-work \
  --model openai/gpt-5.2 \
  --bind whatsapp:biz \
  --non-interactive \
  --json
```

<div id="gateway-wizard-rpc">
  ## Gateway-Wizard-RPC
</div>

Das Gateway stellt den Wizard-Ablauf per RPC bereit (`wizard.start`, `wizard.next`, `wizard.cancel`, `wizard.status`).
Clients (macOS-App, Control UI) können die Schritte rendern, ohne die Onboarding-Logik selbst erneut implementieren zu müssen.

<div id="signal-setup-signal-cli">
  ## Signal-Einrichtung (signal-cli)
</div>

Der Assistent kann `signal-cli` aus den GitHub-Releases installieren:

* Lädt das passende Release-Asset herunter.
* Legt es unter `~/.openclaw/tools/signal-cli/<version>/` ab.
* Schreibt `channels.signal.cliPath` in deine Konfiguration.

Hinweise:

* JVM-Builds erfordern **Java 21**.
* Native-Builds werden verwendet, wenn sie verfügbar sind.
* Unter Windows wird WSL2 verwendet; die signal-cli-Installation folgt dem Linux-Flow innerhalb von WSL2.

<div id="what-the-wizard-writes">
  ## Was der Wizard schreibt
</div>

Typische Felder in `~/.openclaw/openclaw.json`:

* `agents.defaults.workspace`
* `agents.defaults.model` / `models.providers` (wenn Minimax gewählt wurde)
* `gateway.*` (Modus, bind, auth, tailscale)
* `channels.telegram.botToken`, `channels.discord.token`, `channels.signal.*`, `channels.imessage.*`
* Channel-Allowlists (Slack/Discord/Matrix/Microsoft Teams), wenn du dich während der Prompts dafür entscheidest (Namen werden, wenn möglich, in IDs aufgelöst).
* `skills.install.nodeManager`
* `wizard.lastRunAt`
* `wizard.lastRunVersion`
* `wizard.lastRunCommit`
* `wizard.lastRunCommand`
* `wizard.lastRunMode`

`openclaw agents add` schreibt `agents.list[]` und optionale `bindings`.

WhatsApp-Zugangsdaten liegen unter `~/.openclaw/credentials/whatsapp/<accountId>/`.
Sitzungen werden unter `~/.openclaw/agents/<agentId>/sessions/` gespeichert.

Einige Kanäle werden als Plugins ausgeliefert. Wenn du während des Onboardings einen auswählst, fordert dich der Wizard auf, ihn (per npm oder über einen lokalen Pfad) zu installieren, bevor er konfiguriert werden kann.

<div id="related-docs">
  ## Verwandte Dokumentation
</div>

* Onboarding der macOS‑App: [Onboarding](/de/start/onboarding)
* Konfigurationsreferenz: [Gateway-Konfiguration](/de/gateway/configuration)
* Anbieter: [WhatsApp](/de/channels/whatsapp), [Telegram](/de/channels/telegram), [Discord](/de/channels/discord), [Google Chat](/de/channels/googlechat), [Signal](/de/channels/signal), [iMessage](/de/channels/imessage)
* Fähigkeiten: [Fähigkeiten](/de/tools/skills), [Fähigkeiten-Konfiguration](/de/tools/skills-config)