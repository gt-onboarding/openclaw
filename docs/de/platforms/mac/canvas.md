---
title: Canvas
summary: "Agent-gesteuertes Canvas-Panel, eingebettet über WKWebView + benutzerdefiniertes URL-Schema"
read_when:
  - Implementieren des Canvas-Panels unter macOS
  - Hinzufügen von Agent-Steuerungen für den visuellen Arbeitsbereich
  - Debuggen von Canvas-Ladevorgängen in WKWebView
---

<div id="canvas-macos-app">
  # Canvas (macOS‑App)
</div>

Die macOS‑App bettet ein agent‑gesteuertes **Canvas‑Panel** mittels `WKWebView` ein. Es ist ein leichtgewichtiger visueller Arbeitsbereich für HTML/CSS/JS, A2UI und kleine interaktive UI‑Oberflächen.

<div id="where-canvas-lives">
  ## Wo Canvas lebt
</div>

Der Canvas-Zustand wird unter Application Support gespeichert:

- `~/Library/Application Support/OpenClaw/canvas/<session>/...`

Das Canvas-Panel stellt diese Dateien über ein **benutzerdefiniertes URL-Schema** bereit:

- `openclaw-canvas://<session>/<path>`

Beispiele:

- `openclaw-canvas://main/` → `<canvasRoot>/main/index.html`
- `openclaw-canvas://main/assets/app.css` → `<canvasRoot>/main/assets/app.css`
- `openclaw-canvas://main/widgets/todo/` → `<canvasRoot>/main/widgets/todo/index.html`

Wenn im Wurzelverzeichnis keine `index.html` vorhanden ist, zeigt die App eine **eingebaute Gerüstseite** an.

<div id="panel-behavior">
  ## Panelverhalten
</div>

- Rahmenloses, in der Größe veränderbares Panel, das in der Nähe der Menüleiste (oder des Mauszeigers) verankert ist.
- Merkt sich Größe/Position pro Sitzung.
- Wird automatisch neu geladen, wenn lokale Canvas‑Dateien geändert werden.
- Es kann immer nur ein Canvas‑Panel gleichzeitig sichtbar sein (die Sitzung wird bei Bedarf gewechselt).

Canvas kann in den Einstellungen unter **Canvas zulassen** deaktiviert werden. Wenn deaktiviert, geben Canvas‑Node‑Befehle `CANVAS_DISABLED` zurück.

<div id="agent-api-surface">
  ## Agent-API-Oberfläche
</div>

Canvas wird über den **Gateway-WebSocket** bereitgestellt, sodass der Agent Folgendes tun kann:

* das Panel ein- oder ausblenden
* zu einem Pfad oder einer URL navigieren
* JavaScript ausführen
* ein Snapshot-Bild erfassen

CLI-Beispiele:

```bash
openclaw nodes canvas present --node <id>
openclaw nodes canvas navigate --node <id> --url "/"
openclaw nodes canvas eval --node <id> --js "document.title"
openclaw nodes canvas snapshot --node <id>
```

Hinweise:

* `canvas.navigate` akzeptiert **lokale Canvas-Pfade**, `http(s)`-URLs und `file://`-URLs.
* Wenn du `"/"` übergibst, zeigt Canvas das lokale Scaffold oder `index.html` an.


<div id="a2ui-in-canvas">
  ## A2UI in Canvas
</div>

A2UI wird vom Gateway-Canvas-Host bereitgestellt und im Canvas-Panel gerendert.
Wenn das Gateway einen Canvas-Host ankündigt, navigiert die macOS‑App beim
ersten Öffnen automatisch zur A2UI-Hostseite.

Standard-A2UI-Host-URL:

```
http://<gateway-host>:18793/__openclaw__/a2ui/
```


<div id="a2ui-commands-v08">
  ### A2UI-Befehle (v0.8)
</div>

Canvas unterstützt derzeit **A2UI v0.8**-Server-zu-Client-Nachrichten:

* `beginRendering`
* `surfaceUpdate`
* `dataModelUpdate`
* `deleteSurface`

`createSurface` (v0.9) wird nicht unterstützt.

CLI-Beispiel:

```bash
cat > /tmp/a2ui-v0.8.jsonl <<'EOFA2'
{"surfaceUpdate":{"surfaceId":"main","components":[{"id":"root","component":{"Column":{"children":{"explicitList":["title","content"]}}}},{"id":"title","component":{"Text":{"text":{"literalString":"Canvas (A2UI v0.8)"},"usageHint":"h1"}}},{"id":"content","component":{"Text":{"text":{"literalString":"If you can read this, A2UI push works."},"usageHint":"body"}}}]}}
{"beginRendering":{"surfaceId":"main","root":"root"}}
EOFA2

openclaw nodes canvas a2ui push --jsonl /tmp/a2ui-v0.8.jsonl --node <id>
```

Kurzer Smoke-Test:

```bash
openclaw nodes canvas a2ui push --node <id> --text "Hello from A2UI"
```


<div id="triggering-agent-runs-from-canvas">
  ## Auslösen von agent-Ausführungen aus Canvas heraus
</div>

Canvas kann neue agent-Ausführungen über Deep Links auslösen:

* `openclaw://agent?...`

Beispiel (in JS):

```js
window.location.href = "openclaw://agent?message=Review%20this%20design";
```

Die App bittet um Bestätigung, solange kein gültiger Schlüssel angegeben ist.


<div id="security-notes">
  ## Sicherheitshinweise
</div>

- Das Canvas-Schema blockiert Directory-Traversal; Dateien müssen sich im Sitzungs-Root-Verzeichnis befinden.
- Lokale Canvas-Inhalte verwenden ein benutzerdefiniertes Schema (kein Loopback-Server erforderlich).
- Externe `http(s)`-URLs sind nur zulässig, wenn sie explizit aufgerufen werden.