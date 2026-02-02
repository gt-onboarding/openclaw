---
title: Googlechat
summary: "Status der Unterstützung für Google-Chat-Apps, Funktionsumfang und Konfiguration"
read_when:
  - Arbeit an Funktionen des Google-Chat-Kanals
---

<div id="google-chat-chat-api">
  # Google Chat (Chat API)
</div>

Status: bereit für DMs + Spaces über Google Chat API-Webhooks (nur HTTP).

<div id="quick-setup-beginner">
  ## Schnelleinrichtung (Einsteiger)
</div>

1. Erstelle ein Google-Cloud-Projekt und aktiviere die **Google Chat API**.
   * Gehe zu: [Google Chat API Credentials](https://console.cloud.google.com/apis/api/chat.googleapis.com/credentials)
   * Aktiviere die API, falls sie noch nicht aktiviert ist.
2. Erstelle ein **Servicekonto**:
   * Klicke auf **Create Credentials** &gt; **Service Account**.
   * Vergib einen beliebigen Namen (z. B. `openclaw-chat`).
   * Lass die Berechtigungen leer (klicke auf **Continue**).
   * Lass die Principals mit Zugriff leer (klicke auf **Done**).
3. Erstelle und lade den **JSON-Schlüssel** herunter:
   * Klicke in der Liste der Servicekonten auf das soeben erstellte Konto.
   * Gehe zum Tab **Keys**.
   * Klicke auf **Add Key** &gt; **Create new key**.
   * Wähle **JSON** und klicke auf **Create**.
4. Speichere die heruntergeladene JSON-Datei auf deinem Gateway-Host (z. B. `~/.openclaw/googlechat-service-account.json`).
5. Erstelle eine Google-Chat-App in der [Google Cloud Console Chat Configuration](https://console.cloud.google.com/apis/api/chat.googleapis.com/hangouts-chat):
   * Fülle die **Application info** aus:
     * **App name**: (z. B. `OpenClaw`)
     * **Avatar URL**: (z. B. `https://openclaw.ai/logo.png`)
     * **Description**: (z. B. `Personal AI Assistant`)
   * Aktiviere **Interactive features**.
   * Unter **Functionality** aktiviere **Join spaces and group conversations**.
   * Unter **Connection settings** wähle **HTTP endpoint URL**.
   * Unter **Triggers** wähle **Use a common HTTP endpoint URL for all triggers** und setze sie auf die öffentliche URL deines Gateways, gefolgt von `/googlechat`.
     * *Tipp: Führe `openclaw status` aus, um die öffentliche URL deines Gateways zu finden.*
   * Unter **Visibility** aktiviere **Make this Chat app available to specific people and groups in &lt;Your Domain&gt;**.
   * Gib deine E-Mail-Adresse (z. B. `user@example.com`) in das Textfeld ein.
   * Klicke unten auf **Save**.
6. **Aktiviere den App-Status**:
   * **Aktualisiere die Seite**, nachdem du gespeichert hast.
   * Suche nach dem Abschnitt **App status** (normalerweise oben oder unten nach dem Speichern).
   * Ändere den Status auf **Live - available to users**.
   * Klicke erneut auf **Save**.
7. Konfiguriere OpenClaw mit dem Servicekonto-Pfad und der Webhook-Audience:
   * Env: `GOOGLE_CHAT_SERVICE_ACCOUNT_FILE=/path/to/service-account.json`
   * Oder Config: `channels.googlechat.serviceAccountFile: "/path/to/service-account.json"`.
8. Setze den Webhook-Audience-Typ und -Wert (muss zu deiner Chat-App-Konfiguration passen).
9. Starte den Gateway. Google Chat sendet dann POST-Anfragen an deinen Webhook-Pfad.

<div id="add-to-google-chat">
  ## Zu Google Chat hinzufügen
</div>

Sobald das Gateway läuft und deine E-Mail-Adresse zur Sichtbarkeitsliste hinzugefügt wurde:

1. Gehe zu [Google Chat](https://chat.google.com/).
2. Klicke auf das **+** (Plus)-Symbol neben **Direct Messages**.
3. Gib im Suchfeld (wo du normalerweise Personen hinzufügst) den **App-Namen** ein, den du in der Google Cloud Console konfiguriert hast.
   * **Hinweis**: Der Bot erscheint *nicht* in der „Marketplace“-Browse-Liste, weil es sich um eine private App handelt. Du musst gezielt nach seinem Namen suchen.
4. Wähle deinen Bot aus den Ergebnissen aus.
5. Klicke auf **Add** oder **Chat**, um eine 1:1-Konversation zu starten.
6. Sende „Hello“, um den Assistenten auszulösen!

<div id="public-url-webhook-only">
  ## Öffentliche URL (nur Webhook)
</div>

Google-Chat-Webhooks erfordern einen öffentlichen HTTPS-Endpunkt. Gib aus Sicherheitsgründen **nur den Pfad `/googlechat`** ins Internet frei. Halte das OpenClaw-Dashboard und andere sensible Endpunkte in deinem privaten Netzwerk.

<div id="option-a-tailscale-funnel-recommended">
  ### Option A: Tailscale Funnel (empfohlen)
</div>

Verwende Tailscale Serve für das private Dashboard und Funnel für den öffentlichen Webhook-Pfad. Dadurch bleibt `/` privat, während nur `/googlechat` öffentlich erreichbar ist.

1. **Prüfe, an welche Adresse dein Gateway gebunden ist:**
   ```bash
   ss -tlnp | grep 18789
   ```
   Notiere dir die IP-Adresse (z. B. `127.0.0.1`, `0.0.0.0` oder deine Tailscale-IP wie `100.x.x.x`).

2. **Stelle das Dashboard nur im Tailnet bereit (Port 8443):**
   ```bash
   # Falls an localhost gebunden (127.0.0.1 oder 0.0.0.0):
   tailscale serve --bg --https 8443 http://127.0.0.1:18789

   # Falls nur an die Tailscale-IP gebunden (z. B. 100.106.161.80):
   tailscale serve --bg --https 8443 http://100.106.161.80:18789
   ```

3. **Mache nur den Webhook-Pfad öffentlich verfügbar:**
   ```bash
   # Falls an localhost gebunden (127.0.0.1 oder 0.0.0.0):
   tailscale funnel --bg --set-path /googlechat http://127.0.0.1:18789/googlechat

   # Falls nur an die Tailscale-IP gebunden (z. B. 100.106.161.80):
   tailscale funnel --bg --set-path /googlechat http://100.106.161.80:18789/googlechat
   ```

4. **Autorisiere den Knoten für Funnel-Zugriff:**
   Falls du dazu aufgefordert wirst, öffne die in der Ausgabe angezeigte Autorisierungs-URL, um Funnel für diesen Knoten in deiner Tailnet-Richtlinie zu aktivieren.

5. **Überprüfe die Konfiguration:**
   ```bash
   tailscale serve status
   tailscale funnel status
   ```

Deine öffentliche Webhook-URL lautet:
`https://<node-name>.<tailnet>.ts.net/googlechat`

Dein privates Dashboard bleibt nur im Tailnet erreichbar:
`https://<node-name>.<tailnet>.ts.net:8443/`

Verwende die öffentliche URL (ohne `:8443`) in der Google-Chat-App-Konfiguration.

> Hinweis: Diese Konfiguration bleibt über Neustarts hinweg bestehen. Um sie später zu entfernen, führe `tailscale funnel reset` und `tailscale serve reset` aus.

<div id="option-b-reverse-proxy-caddy">
  ### Option B: Reverse Proxy (Caddy)
</div>

Wenn Sie einen Reverse Proxy wie Caddy verwenden, leiten Sie nur den spezifischen Pfad weiter:

```caddy
your-domain.com {
    reverse_proxy /googlechat* localhost:18789
}
```

Mit dieser Konfiguration werden alle Anfragen an `your-domain.com/` ignoriert oder mit einem 404-Fehler beantwortet, während `your-domain.com/googlechat` sicher an OpenClaw weitergeleitet wird.

<div id="option-c-cloudflare-tunnel">
  ### Option C: Cloudflare Tunnel
</div>

Konfiguriere die Ingress-Regeln deines Tunnels so, dass nur der Webhook-Pfad weitergeleitet wird:

* **Path**: `/googlechat` -&gt; `http://localhost:18789/googlechat`
* **Default Rule**: HTTP 404 (Not Found)

<div id="how-it-works">
  ## Funktionsweise
</div>

1. Google Chat sendet Webhook-POSTs an das Gateway. Jede Anfrage enthält einen `Authorization: Bearer <token>`-Header.
2. OpenClaw überprüft das Token anhand der konfigurierten Werte für `audienceType` und `audience`:
   * `audienceType: "app-url"` → `audience` ist deine HTTPS-Webhook-URL.
   * `audienceType: "project-number"` → `audience` ist die Cloud-Projektnummer.
3. Nachrichten werden pro Space geroutet:
   * DMs verwenden den Sitzungsschlüssel `agent:<agentId>:googlechat:dm:<spaceId>`.
   * Spaces verwenden den Sitzungsschlüssel `agent:<agentId>:googlechat:group:<spaceId>`.
4. DM-Zugriff läuft standardmäßig über Kopplung. Unbekannte Absender erhalten einen Kopplungscode, den du mit folgendem Befehl genehmigst:
   * `openclaw pairing approve googlechat <code>`
5. Gruppen-Spaces erfordern standardmäßig eine @-Erwähnung. Verwende `botUser`, wenn die Erwähnungserkennung den Benutzernamen der App benötigt.

<div id="targets">
  ## Ziele
</div>

Verwende diese Kennungen für Zustellung und Allowlists:

* Direktnachrichten: `users/<userId>` oder `users/<email>` (E-Mail-Adressen werden akzeptiert).
* Spaces: `spaces/<spaceId>`.

<div id="config-highlights">
  ## Wichtige Konfigurationspunkte
</div>

```json5
{
  channels: {
    "googlechat": {
      enabled: true,
      serviceAccountFile: "/path/to/service-account.json",
      audienceType: "app-url",
      audience: "https://gateway.example.com/googlechat",
      webhookPath: "/googlechat",
      botUser: "users/1234567890", // optional; hilft bei der Erwähnungs-Erkennung
      dm: {
        policy: "pairing",
        allowFrom: ["users/1234567890", "name@example.com"]
      },
      groupPolicy: "allowlist",
      groups: {
        "spaces/AAAA": {
          allow: true,
          requireMention: true,
          users: ["users/1234567890"],
          systemPrompt: "Short answers only."
        }
      },
      actions: { reactions: true },
      typingIndicator: "message",
      mediaMaxMb: 20
    }
  }
}
```

Hinweise:

* Service-Account-Anmeldedaten können auch direkt als JSON-String über `serviceAccount` übergeben werden.
* Der Standard-Webhook-Pfad lautet `/googlechat`, wenn `webhookPath` nicht gesetzt ist.
* Reaktionen sind über das `reactions`-Tool und `channels action` verfügbar, wenn `actions.reactions` aktiviert ist.
* `typingIndicator` unterstützt `none`, `message` (Standard) und `reaction` (Reaktionen erfordern Benutzer-OAuth).
* Anhänge werden über die Chat-API heruntergeladen und in der Medien-Pipeline gespeichert (Größe begrenzt durch `mediaMaxMb`).

<div id="troubleshooting">
  ## Fehlerbehebung
</div>

<div id="405-method-not-allowed">
  ### 405 Method Not Allowed
</div>

Wenn der Google Cloud Logs Explorer Fehler wie etwa die folgenden anzeigt:

```
status code: 405, reason phrase: HTTP error response: HTTP/1.1 405 Method Not Allowed
```

Das bedeutet, dass der Webhook-Handler nicht registriert ist. Häufige Ursachen:

1. **Channel nicht konfiguriert**: Der Abschnitt `channels.googlechat` fehlt in deiner Konfiguration. Prüfe das mit:
   ```bash
   openclaw config get channels.googlechat
   ```
   Wenn &quot;Config path not found&quot; zurückgegeben wird, füge die Konfiguration hinzu (siehe [Config highlights](#config-highlights)).

2. **Plugin nicht aktiviert**: Prüfe den Plugin-Status:
   ```bash
   openclaw plugins list | grep googlechat
   ```
   Wenn dort &quot;disabled&quot; angezeigt wird, füge `plugins.entries.googlechat.enabled: true` zu deiner Konfiguration hinzu.

3. **Gateway nicht neu gestartet**: Starte das Gateway nach dem Hinzufügen der Konfiguration neu:
   ```bash
   openclaw gateway restart
   ```

Überprüfe, ob der Channel läuft:

```bash
openclaw channels status
# Sollte anzeigen: Google Chat default: enabled, configured, …
```

<div id="other-issues">
  ### Sonstige Probleme
</div>

* Prüfe `openclaw channels status --probe` auf Authentifizierungsfehler oder eine fehlende Audience-Konfiguration.
* Wenn keine Nachrichten ankommen, überprüfe die Webhook-URL der Chat-App und die Ereignisabonnements.
* Wenn Mention-Gating Antworten blockiert, setze `botUser` auf den Benutzerressourcennamen der App und überprüfe `requireMention`.
* Nutze `openclaw logs --follow`, während du eine Testnachricht sendest, um zu sehen, ob Anfragen das Gateway erreichen.

Verwandte Dokumentation:

* [Gateway-Konfiguration](/de/gateway/configuration)
* [Sicherheit](/de/gateway/security)
* [Reaktionen](/de/tools/reactions)