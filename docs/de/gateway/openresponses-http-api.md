---
title: OpenResponses HTTP-API
summary: "Einen OpenResponses-kompatiblen /v1/responses HTTP-Endpunkt vom Gateway bereitstellen"
read_when:
  - Wenn du Clients integrierst, die die OpenResponses-API verwenden
  - Wenn du itembasierte Eingaben, Client-Tool-Aufrufe oder SSE-Ereignisse benötigst
---

<div id="openresponses-api-http">
  # OpenResponses API (HTTP)
</div>

Das Gateway von OpenClaw kann einen OpenResponses-kompatiblen `POST /v1/responses`-Endpunkt bereitstellen.

Dieser Endpunkt ist **standardmäßig deaktiviert**. Aktiviere ihn zuerst in der Konfiguration.

- `POST /v1/responses`
- Gleicher Port wie das Gateway (WS + HTTP-Multiplexing): `http://<gateway-host>:<port>/v1/responses`

Im Hintergrund werden Anfragen als normaler Lauf eines Gateway-agents ausgeführt (gleicher Codepfad wie
`openclaw agent`), sodass Routing/Berechtigungen/Konfiguration mit deinem Gateway übereinstimmen.

<div id="authentication">
  ## Authentifizierung
</div>

Verwendet die Authentifizierungskonfiguration des Gateway. Sende ein Bearer-Token:

- `Authorization: Bearer <token>`

Hinweise:

- Wenn `gateway.auth.mode="token"` ist, verwende `gateway.auth.token` (oder `OPENCLAW_GATEWAY_TOKEN`).
- Wenn `gateway.auth.mode="password"` ist, verwende `gateway.auth.password` (oder `OPENCLAW_GATEWAY_PASSWORD`).

<div id="choosing-an-agent">
  ## Auswahl eines agents
</div>

Keine benutzerdefinierten Header erforderlich: Geben Sie die agent-ID im OpenResponses-`model`-Feld an:

- `model: "openclaw:&lt;agentId&gt;"` (Beispiel: `"openclaw:main"`, `"openclaw:beta"`)
- `model: "agent:&lt;agentId&gt;"` (Alias)

Oder adressieren Sie einen bestimmten OpenClaw-agent über einen Header:

- `x-openclaw-agent-id: &lt;agentId&gt;` (Standardwert: `main`)

Erweitert:

- `x-openclaw-session-key: &lt;sessionKey&gt;` für die vollständige Kontrolle des Sitzungsroutings.

<div id="enabling-the-endpoint">
  ## Aktivieren des Endpunkts
</div>

Setzen Sie `gateway.http.endpoints.responses.enabled` auf `true`:

```json5
{
  gateway: {
    http: {
      endpoints: {
        responses: { enabled: true }
      }
    }
  }
}
```


<div id="disabling-the-endpoint">
  ## Deaktivieren des Endpunkts
</div>

Setzen Sie `gateway.http.endpoints.responses.enabled` auf `false`:

```json5
{
  gateway: {
    http: {
      endpoints: {
        responses: { enabled: false }
      }
    }
  }
}
```


<div id="session-behavior">
  ## Sitzungsverhalten
</div>

Standardmäßig ist der Endpunkt **zustandslos je Anfrage** (für jeden Aufruf wird ein neuer Sitzungsschlüssel erzeugt).

Wenn die Anfrage einen OpenResponses-`user`-String enthält, leitet das Gateway daraus einen stabilen Sitzungsschlüssel ab, sodass wiederholte Aufrufe eine gemeinsame Agent-Sitzung nutzen können.

<div id="request-shape-supported">
  ## Request-Struktur (unterstützt)
</div>

Die Anfrage folgt der OpenResponses API mit itembasiertem Input. Aktuell unterstützt werden:

- `input`: String oder Array von Item-Objekten.
- `instructions`: werden in den System-Prompt eingebettet.
- `tools`: Client-Tool-Definitionen (Funktions-Tools).
- `tool_choice`: filtert oder erzwingt Client-Tools.
- `stream`: aktiviert SSE-Streaming.
- `max_output_tokens`: Best-Effort-Ausgabegrenze (anbieterabhängig).
- `user`: stabile Sitzungszuordnung.

Akzeptiert, aber **derzeit ignoriert**:

- `max_tool_calls`
- `reasoning`
- `metadata`
- `store`
- `previous_response_id`
- `truncation`

<div id="items-input">
  ## Items (Eingabe)
</div>

<div id="message">
  ### `message`
</div>

Rollen: `system`, `developer`, `user`, `assistant`.

- `system` und `developer` werden an den Systemprompt angehängt.
- Das neueste `user`- oder `function_call_output`-Element wird zur „aktuellen Nachricht“.
- Frühere `user`-/`assistant`-Nachrichten werden als Verlaufskontext einbezogen.

<div id="function_call_output-turn-based-tools">
  ### `function_call_output` (turn-based tools)
</div>

Sende Tool-Ergebnisse zurück an das Modell:

```json
{
  "type": "function_call_output",
  "call_id": "call_123",
  "output": "{\"temperature\": \"72F\"}"
}
```


<div id="reasoning-and-item_reference">
  ### `reasoning` und `item_reference`
</div>

Zur Wahrung der Schema-Kompatibilität akzeptiert, aber beim Erstellen des Prompts ignoriert.

<div id="tools-client-side-function-tools">
  ## Tools (clientseitige Funktions-Tools)
</div>

Stelle Tools über `tools: [{ type: "function", function: { name, description?, parameters? } }]` bereit.

Wenn dein agent entscheidet, ein Tool aufzurufen, enthält die Antwort ein `function_call`-Ausgabe-Element.
Du sendest dann eine Folgeanfrage mit `function_call_output`, um den Dialogschritt fortzusetzen.

<div id="images-input_image">
  ## Bilder (`input_image`)
</div>

Unterstützt Base64- oder URL-Quellen:

```json
{
  "type": "input_image",
  "source": { "type": "url", "url": "https://example.com/image.png" }
}
```

Zulässige MIME-Typen (derzeit): `image/jpeg`, `image/png`, `image/gif`, `image/webp`.
Maximale Größe (derzeit): 10 MB.


<div id="files-input_file">
  ## Dateien (`input_file`)
</div>

Unterstützt Base64- oder URL-Quellen:

```json
{
  "type": "input_file",
  "source": {
    "type": "base64",
    "media_type": "text/plain",
    "data": "SGVsbG8gV29ybGQh",
    "filename": "hello.txt"
  }
}
```

Zulässige MIME-Typen (derzeit): `text/plain`, `text/markdown`, `text/html`, `text/csv`,
`application/json`, `application/pdf`.

Maximale Größe (derzeit): 5MB.

Derzeitiges Verhalten:

* Dateiinhalte werden dekodiert und zur **System-Prompt** hinzugefügt, nicht zur Benutzernachricht,
  sodass sie flüchtig bleiben (nicht in der Sitzungshistorie persistiert werden).
* PDFs werden auf Text analysiert. Wenn nur wenig Text gefunden wird, werden die ersten Seiten
  in Bilder gerastert und an das Modell übergeben.

Das PDF-Parsing verwendet den Node.js-kompatiblen `pdfjs-dist` Legacy-Build (ohne Worker). Der moderne
PDF.js-Build erwartet Browser-Worker/DOM-Globals und wird daher im Gateway nicht verwendet.

Standardwerte für URL-Fetch:

* `files.allowUrl`: `true`
* `images.allowUrl`: `true`
* Anfragen werden abgesichert (DNS-Auflösung, Blockierung privater IPs, Begrenzung von Weiterleitungen, Timeouts).


<div id="file-image-limits-config">
  ## Grenzwerte für Dateien und Bilder (Konfiguration)
</div>

Standardwerte können unter `gateway.http.endpoints.responses` angepasst werden:

```json5
{
  gateway: {
    http: {
      endpoints: {
        responses: {
          enabled: true,
          maxBodyBytes: 20000000,
          files: {
            allowUrl: true,
            allowedMimes: ["text/plain", "text/markdown", "text/html", "text/csv", "application/json", "application/pdf"],
            maxBytes: 5242880,
            maxChars: 200000,
            maxRedirects: 3,
            timeoutMs: 10000,
            pdf: {
              maxPages: 4,
              maxPixels: 4000000,
              minTextChars: 200
            }
          },
          images: {
            allowUrl: true,
            allowedMimes: ["image/jpeg", "image/png", "image/gif", "image/webp"],
            maxBytes: 10485760,
            maxRedirects: 3,
            timeoutMs: 10000
          }
        }
      }
    }
  }
}
```

Standardwerte, wenn nicht angegeben:

* `maxBodyBytes`: 20MB
* `files.maxBytes`: 5MB
* `files.maxChars`: 200k
* `files.maxRedirects`: 3
* `files.timeoutMs`: 10s
* `files.pdf.maxPages`: 4
* `files.pdf.maxPixels`: 4,000,000
* `files.pdf.minTextChars`: 200
* `images.maxBytes`: 10MB
* `images.maxRedirects`: 3
* `images.timeoutMs`: 10s


<div id="streaming-sse">
  ## Streaming (SSE)
</div>

Setze `stream: true`, um Server-Sent Events (SSE) zu empfangen:

- `Content-Type: text/event-stream`
- Jede Ereigniszeile hat die Form `event: <type>` und `data: <json>`
- Der Stream endet mit `data: [DONE]`

Derzeit ausgegebene Ereignistypen:

- `response.created`
- `response.in_progress`
- `response.output_item.added`
- `response.content_part.added`
- `response.output_text.delta`
- `response.output_text.done`
- `response.content_part.done`
- `response.output_item.done`
- `response.completed`
- `response.failed` (bei Fehler)

<div id="usage">
  ## Verwendung
</div>

`usage` wird gefüllt, wenn der zugrunde liegende anbieter Token-Zählungen meldet.

<div id="errors">
  ## Fehler
</div>

Fehler werden in einem JSON-Objekt dargestellt, z. B.:

```json
{ "error": { "message": "...", "type": "invalid_request_error" } }
```

Typische Fälle:

* `401` fehlende oder ungültige Authentifizierung
* `400` ungültiger Request-Body
* `405` falsche Methode


<div id="examples">
  ## Beispiele
</div>

Ohne Streaming:

```bash
curl -sS http://127.0.0.1:18789/v1/responses \
  -H 'Authorization: Bearer YOUR_TOKEN' \
  -H 'Content-Type: application/json' \
  -H 'x-openclaw-agent-id: main' \
  -d '{
    "model": "openclaw",
    "input": "hi"
  }'
```

Streaming:

```bash
curl -N http://127.0.0.1:18789/v1/responses \
  -H 'Authorization: Bearer YOUR_TOKEN' \
  -H 'Content-Type: application/json' \
  -H 'x-openclaw-agent-id: main' \
  -d '{
    "model": "openclaw",
    "stream": true,
    "input": "hi"
  }'
```
