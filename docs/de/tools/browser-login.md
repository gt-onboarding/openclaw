---
title: Browser-Anmeldung
summary: "Manuelle Anmeldungen für Browserautomatisierung + X/Twitter-Posting"
read_when:
  - Du musst dich für die Browserautomatisierung auf Websites anmelden
  - Du möchtest Updates auf X/Twitter veröffentlichen
---

<div id="browser-login-xtwitter-posting">
  # Browser-Anmeldung und X/Twitter-Posting
</div>

<div id="manual-login-recommended">
  ## Manuelle Anmeldung (empfohlen)
</div>

Wenn eine Website eine Anmeldung erfordert, **melde dich manuell im** **Host-**Browserprofil (dem openclaw-Browser) **an**.

Gib dem Modell **nicht** deine Zugangsdaten. Automatisierte Logins lösen häufig Anti‑Bot‑Mechanismen aus und können dein Konto sperren.

Zurück zur Hauptdokumentation zum Browser: [Browser](/de/tools/browser).

<div id="which-chrome-profile-is-used">
  ## Welches Chrome-Profil wird verwendet?
</div>

OpenClaw steuert ein **dediziertes Chrome-Profil** (mit dem Namen `openclaw`, orange eingefärbte UI). Dieses ist getrennt von deinem täglichen Browserprofil.

Zwei einfache Möglichkeiten, darauf zuzugreifen:

1. **Bitte den agent, den Browser zu öffnen** und melde dich dann selbst an.
2. **Öffne es über die CLI**:

```bash
openclaw browser start
openclaw browser open https://x.com
```

Wenn Sie mehrere Profile haben, übergeben Sie `--browser-profile <name>` (Standardwert ist `openclaw`).

<div id="xtwitter-recommended-flow">
  ## X/Twitter: empfohlener Ablauf
</div>

* **Lesen/Suchen/Threads:** Verwende das **bird**-CLI-Skill (kein Browser erforderlich, stabil).
  * Repo: https://github.com/steipete/bird
* **Updates posten:** Verwende den **host**-Browser (manuelle Anmeldung).

<div id="sandboxing-host-browser-access">
  ## Sandboxing + Zugriff auf den Host-Browser
</div>

Browser-Sitzungen in einer sandbox lösen **mit höherer Wahrscheinlichkeit** Bot-Erkennung aus. Für X/Twitter (und andere sehr restriktive Websites) verwende bevorzugt den **Host**-Browser.

Wenn der agent in einer sandbox läuft, verwendet das Browser-Tool standardmäßig die sandbox. Um die Steuerung über den Host-Browser zu ermöglichen:

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main",
        browser: {
          allowHostControl: true
        }
      }
    }
  }
}
```

Richte dann den Host-Browser als Ziel ein:

```bash
openclaw browser open https://x.com --browser-profile openclaw --target host
```

Oder deaktiviere das Sandboxing für den agent, der Updates veröffentlicht.
