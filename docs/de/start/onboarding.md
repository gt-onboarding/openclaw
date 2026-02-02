---
title: Onboarding
summary: "Onboarding-Ablauf beim ersten Start für OpenClaw (macOS-App)"
read_when:
  - Entwerfen des macOS-Onboarding-Assistenten
  - Implementieren der Authentifizierungs- oder Identitätseinrichtung
---

<div id="onboarding-macos-app">
  # Onboarding (macOS‑App)
</div>

Dieses Dokument beschreibt den **aktuellen** Onboarding‑Ablauf beim ersten Start. Ziel ist ein reibungsloses „Day‑0“-Erlebnis: Du wählst, wo der Gateway laufen soll, richtest die Authentifizierung ein, führst den Assistenten aus und lässt den agent sich selbst bootstrappen.

<div id="page-order-current">
  ## Seitenreihenfolge (derzeit)
</div>

1. Willkommen + Sicherheitshinweis
2. **Gateway-Auswahl** (Lokal / Remote / Später einrichten)
3. **Auth (Anthropic OAuth)** — nur lokal
4. **Setup-Assistent** (Gateway-gesteuert)
5. **Berechtigungen** (TCC-Eingabeaufforderungen)
6. **CLI** (optional)
7. **Onboarding-Chat** (eigene Sitzung)
8. Bereit

<div id="1-local-vs-remote">
  ## 1) Lokal vs Remote
</div>

Wo läuft das **Gateway**?

* **Lokal (dieser Mac):** Onboarding kann OAuth-Flows durchlaufen und Anmeldedaten lokal speichern.
* **Remote (über SSH/Tailnet):** Onboarding führt OAuth-Flows **nicht** lokal aus; Anmeldedaten müssen auf dem Gateway-Host vorhanden sein.
* **Später konfigurieren:** Setup überspringen und die App unkonfiguriert lassen.

Gateway-Authentifizierungs-Tipp:

* Der Assistent erzeugt jetzt auch für Loopback ein **Token**, daher müssen sich lokale WS-Clients authentifizieren.
* Wenn du die Authentifizierung deaktivierst, kann sich jeder lokale Prozess verbinden; verwende das nur auf vollständig vertrauenswürdigen Maschinen.
* Verwende ein **Token** für den Zugriff von mehreren Maschinen oder für Nicht-Loopback-Bindings.

<div id="2-local-only-auth-anthropic-oauth">
  ## 2) Nur lokale Authentifizierung (Anthropic OAuth)
</div>

Die macOS-App unterstützt Anthropic OAuth (Claude Pro/Max). Der Ablauf:

* Öffnet den Browser für OAuth (PKCE)
* Fordert den Benutzer auf, den `code#state`-Wert einzugeben
* Schreibt Anmeldedaten in `~/.openclaw/credentials/oauth.json`

Andere Anbieter (OpenAI, benutzerdefinierte APIs) werden vorerst über Umgebungsvariablen
oder Konfigurationsdateien konfiguriert.

<div id="3-setup-wizard-gatewaydriven">
  ## 3) Setup-Assistent (Gateway‑gesteuert)
</div>

Die App kann denselben Setup-Assistenten ausführen wie die CLI. Das hält das Onboarding mit dem Gateway-seitigen Verhalten synchron und vermeidet duplizierte Logik in SwiftUI.

<div id="4-permissions">
  ## 4) Berechtigungen
</div>

Während des Onboardings werden die folgenden TCC-Berechtigungen angefordert:

* Mitteilungen
* Bedienungshilfen
* Bildschirmaufnahme
* Mikrofon / Spracherkennung
* Automatisierung (AppleScript)

<div id="5-cli-optional">
  ## 5) CLI (optional)
</div>

Die App kann die globale `openclaw`-CLI über npm/pnpm installieren, damit Terminal-Workflows und launchd-Aufgaben out of the box funktionieren.

<div id="6-onboarding-chat-dedicated-session">
  ## 6) Onboarding-Chat (dedizierte Sitzung)
</div>

Nach der Einrichtung öffnet die App eine dedizierte Onboarding-Chat-Sitzung, damit der Agent
sich vorstellen und dich durch die nächsten Schritte führen kann. So bleibt die Ersteinrichtungsanleitung getrennt
von deinen normalen Unterhaltungen.

<div id="agent-bootstrap-ritual">
  ## Agent-Bootstrap-Ritual
</div>

Beim ersten Start eines agent richtet OpenClaw einen Arbeitsbereich ein (Standard `~/.openclaw/workspace`):

* Legt `AGENTS.md`, `BOOTSTRAP.md`, `IDENTITY.md`, `USER.md` an
* Führt ein kurzes Q&amp;A-Ritual durch (Frage für Frage)
* Schreibt Identität + Präferenzen in `IDENTITY.md`, `USER.md`, `SOUL.md`
* Entfernt `BOOTSTRAP.md` nach Abschluss, sodass es nur einmal ausgeführt wird

<div id="optional-gmail-hooks-manual">
  ## Optional: Gmail-Hooks (manuell)
</div>

Das Einrichten von Gmail Pub/Sub ist derzeit ein manueller Schritt. Verwende:

```bash
openclaw webhooks gmail setup --account you@gmail.com
```

Siehe [/automation/gmail-pubsub](/de/automation/gmail-pubsub) für weitere Informationen.

<div id="remote-mode-notes">
  ## Hinweise zum Remote-Modus
</div>

Wenn das Gateway auf einem anderen Rechner läuft, liegen Anmeldedaten und Arbeitsbereichsdateien
**auf diesem Host**. Wenn du OAuth im Remote-Modus benötigst, erstelle:

* `~/.openclaw/credentials/oauth.json`
* `~/.openclaw/agents/<agentId>/agent/auth-profiles.json`

auf dem Gateway-Host.