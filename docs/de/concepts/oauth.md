---
title: OAuth
summary: "OAuth in OpenClaw: Tokenaustausch, Speicherung und Multi-Account-Muster"
read_when:
  - Du das OAuth-Verhalten von OpenClaw End-to-End verstehen möchtest
  - Du auf Probleme mit Token-Invalidierung oder Logout stößt
  - Du setup-token- oder OAuth-Authentifizierungsflows einrichten möchtest
  - Du mehrere Accounts oder Profil-Routing verwenden möchtest
---

<div id="oauth">
  # OAuth
</div>

OpenClaw unterstützt „Subscription-Authentifizierung“ über OAuth für anbieter, die dies anbieten (insbesondere **OpenAI Codex (ChatGPT OAuth)**). Für Anthropic‑Abonnements verwendest du den **setup-token**‑Flow. Diese Seite erklärt:

* wie der OAuth‑**Token-Austausch** funktioniert (PKCE)
* wo Tokens **gespeichert** werden (und warum)
* wie du mit **mehreren Accounts** umgehst (Profile + Sitzungsbezogene Overrides)

OpenClaw unterstützt außerdem **anbieter-Plugins**, die ihre eigenen OAuth‑ oder API‑Key‑Flows mitbringen. Du führst sie aus über:

```bash
openclaw models auth login --provider <id>
```

<div id="the-token-sink-why-it-exists">
  ## Die Token-Senke (warum es sie gibt)
</div>

OAuth-Anbieter stellen in Login-/Refresh-Flows häufig ein **neues Refresh-Token** aus. Manche Anbieter (oder OAuth-Clients) können ältere Refresh-Tokens ungültig machen, wenn ein neues für denselben Benutzer/dieselbe App ausgegeben wird.

Praktisches Symptom:

* du meldest dich sowohl über OpenClaw *als auch* über Claude Code / Codex CLI an → eine der Sitzungen wird später scheinbar zufällig „abgemeldet“

Um das zu reduzieren, behandelt OpenClaw die `auth-profiles.json` als eine **Token-Senke**:

* die Runtime liest Zugangsdaten aus **einer Quelle**
* wir können mehrere Profile vorhalten und sie deterministisch routen

<div id="storage-where-tokens-live">
  ## Speicherung (wo Tokens gespeichert werden)
</div>

Secrets werden **pro Agent** gespeichert:

* Auth-Profile (OAuth + API-Schlüssel): `~/.openclaw/agents/<agentId>/agent/auth-profiles.json`
* Laufzeit-Cache (wird automatisch verwaltet; nicht bearbeiten): `~/.openclaw/agents/<agentId>/agent/auth.json`

Veraltete, nur für den Import genutzte Datei (weiterhin unterstützt, aber nicht der Hauptspeicher):

* `~/.openclaw/credentials/oauth.json` (wird bei der ersten Verwendung in `auth-profiles.json` importiert)

Alle oben genannten Pfade respektieren außerdem `$OPENCLAW_STATE_DIR` (zum Überschreiben des Zustandsverzeichnisses). Vollständige Referenz: [/gateway/configuration](/de/gateway/configuration#auth-storage-oauth--api-keys)

<div id="anthropic-setup-token-subscription-auth">
  ## Anthropic setup-token (Abonnementauthentifizierung)
</div>

Führe `claude setup-token` auf einer beliebigen Maschine aus und füge den ausgegebenen Token anschließend in OpenClaw ein:

```bash
openclaw models auth setup-token --provider anthropic
```

Wenn Sie das Token an anderer Stelle generiert haben, fügen Sie es manuell ein:

```bash
openclaw models auth paste-token --provider anthropic
```

Prüfen:

```bash
openclaw models status
```

<div id="oauth-exchange-how-login-works">
  ## OAuth-Austausch (Funktionsweise der Anmeldung)
</div>

Die interaktiven Anmelde-Workflows von OpenClaw sind in `@mariozechner/pi-ai` implementiert und in die Assistenten/Wizards und Befehle integriert.

<div id="anthropic-claude-promax-setup-token">
  ### Anthropic (Claude Pro/Max) setup-token
</div>

Ablauf:

1. Führe `claude setup-token` aus
2. Füge das Token in OpenClaw ein
3. Speichere es als Token-Auth-Profil (ohne Refresh)

Der Wizard-Pfad lautet `openclaw onboard` → Auth-Auswahl `setup-token` (Anthropic).

<div id="openai-codex-chatgpt-oauth">
  ### OpenAI Codex (ChatGPT OAuth)
</div>

Ablaufmuster (PKCE):

1. PKCE-Verifier/-Challenge und zufälligen `state` erzeugen
2. `https://auth.openai.com/oauth/authorize?...` im Browser öffnen
3. versuchen, den Callback auf `http://127.0.0.1:1455/auth/callback` abzufangen
4. wenn der Callback-Endpunkt nicht gebunden werden kann (oder du remote/headless unterwegs bist), die Redirect-URL/den Code einfügen
5. Token-Austausch über `https://auth.openai.com/oauth/token`
6. `accountId` aus dem Access-Token extrahieren und `{ access, refresh, expires, accountId }` speichern

Wizard-Pfad ist `openclaw onboard` → Auth-Auswahl `openai-codex`.

<div id="refresh-expiry">
  ## Refresh + Ablauf
</div>

Profile speichern einen `expires`-Zeitstempel.

Zur Laufzeit:

* wenn `expires` in der Zukunft liegt → das gespeicherte Access-Token verwenden
* wenn abgelaufen → Refresh durchführen (unter einem File-Lock) und die gespeicherten Anmeldedaten überschreiben

Der Refresh-Vorgang ist automatisch; du musst Token in der Regel nicht manuell verwalten.

<div id="multiple-accounts-profiles-routing">
  ## Mehrere Konten (Profile) + Routing
</div>

Zwei Ansätze:

<div id="1-preferred-separate-agents">
  ### 1) Bevorzugt: getrennte Agenten
</div>

Wenn du möchtest, dass „privat“ und „beruflich“ niemals miteinander interagieren, verwende isolierte Agenten (separate Sitzungen + Zugangsdaten + arbeitsbereich):

```bash
openclaw agents add work
openclaw agents add personal
```

Konfiguriere anschließend die Authentifizierung pro Agent (Wizard) und leite Chats an den richtigen agent weiter.

<div id="2-advanced-multiple-profiles-in-one-agent">
  ### 2) Fortgeschritten: mehrere Profile in einem Agent
</div>

`auth-profiles.json` unterstützt mehrere Profil-IDs für denselben Anbieter.

Lege fest, welches Profil verwendet wird:

* global über die Reihenfolge in der Konfiguration (`auth.order`)
* pro Sitzung über `/model ...@<profileId>`

Beispiel (sitzungsspezifische Überschreibung):

* `/model Opus@anthropic:work`

So findest du heraus, welche Profil-IDs existieren:

* `openclaw channels list --json` (zeigt `auth[]`)

Zugehörige Dokumentation:

* [/concepts/model-failover](/de/concepts/model-failover) (Rotation + Cooldown-Regeln)
* [/tools/slash-commands](/de/tools/slash-commands) (Befehlsoberfläche)