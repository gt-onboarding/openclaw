---
title: Canvas
summary: "Pannello Canvas controllato dall'Agente, incorporato tramite WKWebView e schema URL personalizzato"
read_when:
  - Implementare il pannello Canvas nell'app macOS
  - Aggiungere controlli dell'Agente per lo spazio di lavoro visivo
  - Eseguire il debug dei caricamenti di Canvas in WKWebView
---

<div id="canvas-macos-app">
  # Canvas (app macOS)
</div>

L'app per macOS incorpora un **pannello Canvas** controllato da un agente tramite `WKWebView`. Si tratta di uno spazio di lavoro visivo leggero per HTML/CSS/JS, A2UI e piccole superfici UI interattive.

<div id="where-canvas-lives">
  ## Dove si trova Canvas
</div>

Lo stato di Canvas è memorizzato sotto Application Support:

- `~/Library/Application Support/OpenClaw/canvas/<session>/...`

Il pannello Canvas espone quei file tramite uno **schema URL personalizzato**:

- `openclaw-canvas://<session>/<path>`

Esempi:

- `openclaw-canvas://main/` → `<canvasRoot>/main/index.html`
- `openclaw-canvas://main/assets/app.css` → `<canvasRoot>/main/assets/app.css`
- `openclaw-canvas://main/widgets/todo/` → `<canvasRoot>/main/widgets/todo/index.html`

Se non esiste alcun `index.html` nella directory radice, l'app mostra una **pagina di scaffolding integrata**.

<div id="panel-behavior">
  ## Comportamento del pannello
</div>

- Pannello senza bordi e ridimensionabile, ancorato vicino alla barra dei menu (o al cursore del mouse).
- Ricorda dimensioni/posizione per ogni sessione.
- Si ricarica automaticamente quando cambiano i file Canvas locali.
- È visibile un solo pannello Canvas alla volta (la sessione viene cambiata se necessario).

Canvas può essere disabilitato da Impostazioni → **Allow Canvas**. Quando è disabilitato, i comandi del node Canvas restituiscono `CANVAS_DISABLED`.

<div id="agent-api-surface">
  ## Superficie API dell&#39;Agente
</div>

Canvas è esposto tramite il **Gateway WebSocket**, quindi l&#39;agente può:

* mostrare/nascondere il pannello
* navigare a un percorso o a un URL
* eseguire JavaScript
* catturare un&#39;istantanea (snapshot)

Esempi CLI:

```bash
openclaw nodes canvas present --node <id>
openclaw nodes canvas navigate --node <id> --url "/"
openclaw nodes canvas eval --node <id> --js "document.title"
openclaw nodes canvas snapshot --node <id>
```

Note:

* `canvas.navigate` accetta **percorsi locali Canvas**, URL `http(s)` e URL `file://`.
* Se specifichi `"/"`, Canvas mostra lo scaffold locale o `index.html`.


<div id="a2ui-in-canvas">
  ## A2UI in Canvas
</div>

A2UI è ospitata dal Canvas host del Gateway e viene renderizzata all&#39;interno del pannello Canvas.
Quando il Gateway espone un Canvas host, l&#39;app macOS passa automaticamente
alla pagina host di A2UI al primo avvio.

URL host predefinito di A2UI:

```
http://<gateway-host>:18793/__openclaw__/a2ui/
```


<div id="a2ui-commands-v08">
  ### Comandi A2UI (v0.8)
</div>

Canvas attualmente accetta messaggi server→client **A2UI v0.8**:

* `beginRendering`
* `surfaceUpdate`
* `dataModelUpdate`
* `deleteSurface`

`createSurface` (v0.9) non è supportato.

Esempio CLI:

```bash
cat > /tmp/a2ui-v0.8.jsonl <<'EOFA2'
{"surfaceUpdate":{"surfaceId":"main","components":[{"id":"root","component":{"Column":{"children":{"explicitList":["title","content"]}}}},{"id":"title","component":{"Text":{"text":{"literalString":"Canvas (A2UI v0.8)"},"usageHint":"h1"}}},{"id":"content","component":{"Text":{"text":{"literalString":"If you can read this, A2UI push works."},"usageHint":"body"}}}]}}
{"beginRendering":{"surfaceId":"main","root":"root"}}
EOFA2

openclaw nodes canvas a2ui push --jsonl /tmp/a2ui-v0.8.jsonl --node <id>
```

Smoke test rapido:

```bash
openclaw nodes canvas a2ui push --node <id> --text "Hello from A2UI"
```


<div id="triggering-agent-runs-from-canvas">
  ## Avviare esecuzioni di agenti da Canvas
</div>

Canvas può avviare nuove esecuzioni di agenti tramite deep link:

* `openclaw://agent?...`

Esempio (in JS):

```js
window.location.href = "openclaw://agent?message=Review%20this%20design";
```

L&#39;app richiede conferma, a meno che non venga fornita una chiave valida.


<div id="security-notes">
  ## Note sulla sicurezza
</div>

- Lo schema di Canvas blocca l'attraversamento di directory; i file devono risiedere all'interno della root della sessione.
- I contenuti locali di Canvas utilizzano uno schema personalizzato (non è richiesto alcun server di loopback).
- Gli URL `http(s)` esterni sono consentiti solo quando vengono aperti esplicitamente.