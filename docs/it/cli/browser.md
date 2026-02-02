---
title: Browser
summary: "Riferimento CLI per `openclaw browser` (profili, schede, azioni, relay dell'estensione)"
read_when:
  - Usi `openclaw browser` e vuoi esempi per le operazioni più comuni
  - Vuoi controllare un browser in esecuzione su un'altra macchina tramite un nodo host
  - Vuoi usare il relay dell'estensione Chrome (attach/detach tramite pulsante sulla barra degli strumenti)
---

<div id="openclaw-browser">
  # `openclaw browser`
</div>

Gestisci il server di controllo del browser di OpenClaw ed esegui azioni nel browser (schede, snapshot, screenshot, navigazione, clic, digitazione).

Correlato:

* Strumento browser + API: [Browser tool](/it/tools/browser)
* Relay tramite estensione Chrome: [Chrome extension](/it/tools/chrome-extension)

<div id="common-flags">
  ## Flag comuni
</div>

* `--url <gatewayWsUrl>`: URL WebSocket del Gateway (predefinito dalla configurazione).
* `--token <token>`: token del Gateway (se richiesto).
* `--timeout <ms>`: timeout della richiesta (in ms).
* `--browser-profile <name>`: scegli un profilo del browser (predefinito dalla configurazione).
* `--json`: output leggibile da macchina (dove supportato).

<div id="quick-start-local">
  ## Guida rapida (locale)
</div>

```bash
openclaw browser --browser-profile chrome tabs
openclaw browser --browser-profile openclaw start
openclaw browser --browser-profile openclaw open https://example.com
openclaw browser --browser-profile openclaw snapshot
```

<div id="profiles">
  ## Profili
</div>

I profili sono configurazioni di instradamento del browser identificate da un nome. In pratica:

* `openclaw`: avvia o si collega a un&#39;istanza dedicata di Chrome gestita da OpenClaw (directory dei dati utente isolata).
* `chrome`: controlla le tue schede Chrome esistenti tramite il relay dell&#39;estensione di Chrome.

```bash
openclaw browser profiles
openclaw browser create-profile --name work --color "#FF5A36"
openclaw browser delete-profile --name work
```

Utilizza un profilo specifico:

```bash
openclaw browser --browser-profile work tabs
```

<div id="tabs">
  ## Schede
</div>

```bash
openclaw browser tabs
openclaw browser open https://docs.openclaw.ai
openclaw browser focus <targetId>
openclaw browser close <targetId>
```

<div id="snapshot-screenshot-actions">
  ## Istantanea / screenshot / azioni
</div>

Istantanea:

```bash
openclaw browser snapshot
```

Screenshot:

```bash
openclaw browser screenshot
```

Naviga/fai clic/digita (automazione UI basata su ref):

```bash
openclaw browser navigate https://example.com
openclaw browser click <ref>
openclaw browser type <ref> "hello"
```

<div id="chrome-extension-relay-attach-via-toolbar-button">
  ## Relay dell&#39;estensione Chrome (collega tramite pulsante della barra degli strumenti)
</div>

Questa modalità consente all&#39;agente di controllare una scheda di Chrome esistente che colleghi manualmente (non si collega automaticamente).

Installa l&#39;estensione non pacchettizzata in un percorso stabile:

```bash
openclaw browser extension install
openclaw browser extension path
```

Poi apri Chrome → `chrome://extensions` → abilita “Developer mode” → “Load unpacked” → seleziona la cartella indicata.

Guida completa: [Estensione Chrome](/it/tools/chrome-extension)

<div id="remote-browser-control-node-host-proxy">
  ## Controllo remoto del browser (proxy host del nodo)
</div>

Se il Gateway viene eseguito su una macchina diversa da quella del browser, esegui un **nodo host** sulla macchina in cui è in esecuzione Chrome/Brave/Edge/Chromium. Il Gateway inoltrerà le azioni del browser a quel nodo (non è richiesto un server separato per il controllo del browser).

Usa `gateway.nodes.browser.mode` per controllare l&#39;instradamento automatico e `gateway.nodes.browser.node` per bloccare un nodo specifico se ne sono connessi più di uno.

Sicurezza + configurazione remota: [Browser tool](/it/tools/browser), [Accesso remoto](/it/gateway/remote), [Tailscale](/it/gateway/tailscale), [Sicurezza](/it/gateway/security)