---
title: Gmail Pub/Sub
summary: "Gmail Pub/Sub-Push über gogcli an OpenClaw-Webhooks anbinden"
read_when:
  - Trigger aus dem Gmail-Posteingang mit OpenClaw verbinden
  - Pub/Sub-Push für das Aufwecken von Agents einrichten
---

<div id="gmail-pubsub-openclaw">
  # Gmail Pub/Sub -&gt; OpenClaw
</div>

Ziel: Gmail Watch -&gt; Pub/Sub-Push -&gt; `gog gmail watch serve` -&gt; OpenClaw Webhook.

<div id="prereqs">
  ## Voraussetzungen
</div>

* `gcloud` installiert und angemeldet ([Installationsanleitung](https://docs.cloud.google.com/sdk/docs/install-sdk)).
* `gog` (gogcli) installiert und für das Gmail-Konto autorisiert ([gogcli.sh](https://gogcli.sh/)).
* OpenClaw Hooks aktiviert (siehe [Webhooks](/de/automation/webhook)).
* `tailscale` angemeldet ([tailscale.com](https://tailscale.com/)). Das unterstützte Setup verwendet Tailscale Funnel für den öffentlichen HTTPS-Endpunkt.
  Andere Tunnel-Dienste können funktionieren, sind aber DIY/nicht unterstützt und erfordern manuelle Anbindung.
  Aktuell unterstützen wir Tailscale.

Beispiel-Hook-Konfiguration (Gmail-Preset-Mapping aktivieren):

```json5
{
  hooks: {
    enabled: true,
    token: "OPENCLAW_HOOK_TOKEN",
    path: "/hooks",
    presets: ["gmail"]
  }
}
```

Um die Gmail-Zusammenfassung an eine Chat-Oberfläche zu übermitteln, überschreibst du das Preset mit einem Mapping,
das `deliver` und optional `channel`/`to` setzt:

```json5
{
  hooks: {
    enabled: true,
    token: "OPENCLAW_HOOK_TOKEN",
    presets: ["gmail"],
    mappings: [
      {
        match: { path: "gmail" },
        action: "agent",
        wakeMode: "now",
        name: "Gmail",
        sessionKey: "hook:gmail:{{messages[0].id}}",
        messageTemplate:
          "New email from {{messages[0].from}}\nSubject: {{messages[0].subject}}\n{{messages[0].snippet}}\n{{messages[0].body}}",
        model: "openai/gpt-5.2-mini",
        deliver: true,
        channel: "last"
        // to: "+15551234567"
      }
    ]
  }
}
```

Wenn du einen festen Channel möchtest, setze `channel` + `to`. Andernfalls verwendet `channel: "last"`
die letzte Zustellroute (fällt auf WhatsApp zurück).

Um ein günstigeres Modell für Gmail-Ausführungen zu erzwingen, setze `model` im Mapping
(`provider/model` oder Alias). Wenn du `agents.defaults.models` erzwingst, füge es dort ein.

Um ein Standardmodell und eine spezifische Denkstufe für Gmail-Hooks festzulegen, füge
`hooks.gmail.model` / `hooks.gmail.thinking` in deiner Konfiguration hinzu:

```json5
{
  hooks: {
    gmail: {
      model: "openrouter/meta-llama/llama-3.3-70b-instruct:free",
      thinking: "off"
    }
  }
}
```

Hinweise:

* Ein `model`/`thinking` pro Hook in der Mapping-Konfiguration überschreibt weiterhin diese Standardwerte.
* Fallback-Reihenfolge: `hooks.gmail.model` → `agents.defaults.model.fallbacks` → primär (Auth/Rate-Limits/Timeouts).
* Wenn `agents.defaults.models` gesetzt ist, muss das Gmail-Modell in der Allowlist stehen.
* Gmail-Hook-Inhalte werden standardmäßig mit Sicherheitsgrenzen für externe Inhalte umhüllt.
  Um dies zu deaktivieren (gefährlich), setze `hooks.gmail.allowUnsafeExternalContent` auf `true`.

Um die Payload-Verarbeitung weiter anzupassen, füge `hooks.mappings` oder ein JS/TS-Transformationsmodul
unter `hooks.transformsDir` hinzu (siehe [Webhooks](/de/automation/webhook)).

<div id="wizard-recommended">
  ## Assistent (empfohlen)
</div>

Verwende den OpenClaw-Helfer, um alles zu verbinden (installiert Abhängigkeiten unter macOS via brew):

```bash
openclaw webhooks gmail setup \
  --account openclaw@gmail.com
```

Standardmäßig:

* Verwendet Tailscale Funnel für den öffentlichen Push-Endpunkt.
* Schreibt die `hooks.gmail`-Konfiguration für `openclaw webhooks gmail run`.
* Aktiviert das Gmail-Hook-Preset (`hooks.presets: ["gmail"]`).

Hinweis zum Pfad: Wenn `tailscale.mode` aktiviert ist, setzt OpenClaw
`hooks.gmail.serve.path` automatisch auf `/` und belässt den öffentlichen Pfad bei
`hooks.gmail.tailscale.path` (Standard `/gmail-pubsub`), weil Tailscale
das gesetzte Pfadpräfix entfernt, bevor es die Anfrage proxyt.
Wenn das Backend den Pfad mit Präfix erhalten soll, setze
`hooks.gmail.tailscale.target` (oder `--tailscale-target`) auf eine vollständige URL wie
`http://127.0.0.1:8788/gmail-pubsub` und passe `hooks.gmail.serve.path` entsprechend an.

Du möchtest einen benutzerdefinierten Endpunkt? Verwende `--push-endpoint <url>` oder `--tailscale off`.

Hinweis zur Plattform: Unter macOS installiert der Assistent `gcloud`, `gogcli` und `tailscale`
über Homebrew; unter Linux musst du sie vorher manuell installieren.

Automatischer Gateway-Start (empfohlen):

* Wenn `hooks.enabled=true` und `hooks.gmail.account` gesetzt ist, startet das Gateway
  `gog gmail watch serve` beim Systemstart und erneuert die Watch automatisch.
* Setze `OPENCLAW_SKIP_GMAIL_WATCHER=1`, um das zu deaktivieren (nützlich, wenn du den Daemon selbst betreibst).
* Führe den manuellen Daemon nicht gleichzeitig aus, sonst erhältst du
  `listen tcp 127.0.0.1:8788: bind: address already in use`.

Manueller Daemon (startet `gog gmail watch serve` mit automatischer Erneuerung):

```bash
openclaw webhooks gmail run
```

<div id="one-time-setup">
  ## Einmalige Einrichtung
</div>

1. Wählen Sie das GCP-Projekt aus, zu dem der OAuth-Client gehört, der von `gog` verwendet wird.

```bash
gcloud auth login
gcloud config set project <project-id>
```

Hinweis: Der Gmail-Watch-Vorgang erfordert, dass das Pub/Sub-Topic im selben Projekt liegt wie der OAuth-Client.

2. APIs aktivieren:

```bash
gcloud services enable gmail.googleapis.com pubsub.googleapis.com
```

3. Erstelle ein Thema:

```bash
gcloud pubsub topics create gog-gmail-watch
```

4. Gmail-Push-Veröffentlichung zulassen:

```bash
gcloud pubsub topics add-iam-policy-binding gog-gmail-watch \
  --member=serviceAccount:gmail-api-push@system.gserviceaccount.com \
  --role=roles/pubsub.publisher
```

<div id="start-the-watch">
  ## Watch starten
</div>

```bash
gog gmail watch start \
  --account openclaw@gmail.com \
  --label INBOX \
  --topic projects/<project-id>/topics/gog-gmail-watch
```

Speichere die `history_id` aus der Ausgabe für Debugging-Zwecke.

<div id="run-the-push-handler">
  ## Push-Handler ausführen
</div>

Lokales Beispiel (Shared-Token-Auth):

```bash
gog gmail watch serve \
  --account openclaw@gmail.com \
  --bind 127.0.0.1 \
  --port 8788 \
  --path /gmail-pubsub \
  --token <shared> \
  --hook-url http://127.0.0.1:18789/hooks/gmail \
  --hook-token OPENCLAW_HOOK_TOKEN \
  --include-body \
  --max-bytes 20000
```

Hinweise:

* `--token` schützt den Push-Endpunkt (`x-gog-token` oder `?token=`).
* `--hook-url` verweist auf OpenClaw `/hooks/gmail` (gemappt; isolierter Lauf + Zusammenfassung an den Hauptlauf).
* `--include-body` und `--max-bytes` steuern den Nachrichtentext-Ausschnitt (Body-Snippet), der an OpenClaw gesendet wird.

Empfehlung: `openclaw webhooks gmail run` führt denselben Ablauf aus und erneuert den Watch automatisch.

<div id="expose-the-handler-advanced-unsupported">
  ## Handler exponieren (fortgeschritten, nicht unterstützt)
</div>

Wenn du einen Tunnel ohne Tailscale benötigst, richte ihn manuell ein und verwende die öffentliche URL im Push-
Abonnement (nicht unterstützt, ohne Schutzmechanismen):

```bash
cloudflared tunnel --url http://127.0.0.1:8788 --no-autoupdate
```

Verwende die generierte URL als Push-Endpunkt:

```bash
gcloud pubsub subscriptions create gog-gmail-watch-push \
  --topic gog-gmail-watch \
  --push-endpoint "https://<public-url>/gmail-pubsub?token=<shared>"
```

Im Produktivbetrieb: Verwende einen stabilen HTTPS-Endpunkt, konfiguriere Pub/Sub OIDC JWT und führe dann folgenden Befehl aus:

```bash
gog gmail watch serve --verify-oidc --oidc-email <svc@...>
```

<div id="test">
  ## Test
</div>

Sende eine Nachricht an den überwachten Posteingang:

```bash
gog gmail send \
  --account openclaw@gmail.com \
  --to openclaw@gmail.com \
  --subject "watch test" \
  --body "ping"
```

Watch-Status und -verlauf prüfen:

```bash
gog gmail watch status --account openclaw@gmail.com
gog gmail history --account openclaw@gmail.com --since <historyId>
```

<div id="troubleshooting">
  ## Fehlerbehebung
</div>

* `Invalid topicName`: Projekt stimmt nicht überein (Topic gehört nicht zum OAuth-Client-Projekt).
* `User not authorized`: fehlende Rolle `roles/pubsub.publisher` auf dem Topic.
* Leere Nachrichten: Gmail-Push stellt nur `historyId` bereit; über `gog gmail history` abrufen.

<div id="cleanup">
  ## Aufräumen
</div>

```bash
gog gmail watch stop --account openclaw@gmail.com
gcloud pubsub subscriptions delete gog-gmail-watch-push
gcloud pubsub topics delete gog-gmail-watch
```
