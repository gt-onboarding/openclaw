---
title: Nodi
summary: "Nodi: abbinamento, capacità, autorizzazioni e helper della CLI per canvas/fotocamera/schermo/sistema"
read_when:
  - Abbinamento di nodi iOS/Android con un Gateway
  - Utilizzo del nodo canvas/fotocamera per il contesto dell'agente
  - Aggiunta di nuovi comandi del nodo o helper della CLI
---

<div id="nodes">
  # Nodi
</div>

Un **nodo** è un dispositivo associato (macOS/iOS/Android/headless) che si connette al **WebSocket** del Gateway (stessa porta degli operatori) con `role: "node"` ed espone un’interfaccia di comando (ad es. `canvas.*`, `camera.*`, `system.*`) tramite `node.invoke`. Dettagli del protocollo: [Gateway protocol](/it/gateway/protocol).

Trasporto legacy: [Bridge protocol](/it/gateway/bridge-protocol) (TCP JSONL; deprecato/rimosso per i nodi attuali).

macOS può anche essere eseguito in **modalità nodo**: l&#39;app della barra dei menu si connette al server WS del Gateway ed espone i propri comandi locali di canvas/camera come nodo (così `openclaw nodes …` funziona con questo Mac).

Note:

* I nodi sono **periferiche**, non Gateway. Non eseguono il servizio Gateway.
* I messaggi Telegram/WhatsApp/etc. arrivano sul **Gateway**, non sui nodi.

<div id="pairing-status">
  ## Abbinamento + stato
</div>

**I nodi WS usano l&#39;abbinamento del dispositivo.** I nodi presentano un&#39;identità di dispositivo durante `connect`; il Gateway
crea una richiesta di abbinamento del dispositivo per `role: node`. Approva la richiesta tramite la CLI dispositivi (o UI).

CLI rapida:

```bash
openclaw devices list
openclaw devices approve <requestId>
openclaw devices reject <requestId>
openclaw nodes status
openclaw nodes describe --node <idOrNameOrIp>
```

Note:

* `nodes status` contrassegna un nodo come **paired** quando il suo ruolo di abbinamento del dispositivo include `node`.
* `node.pair.*` (CLI: `openclaw nodes pending/approve/reject`) è un archivio di abbinamento dei nodi separato, di proprietà del Gateway; **non** controlla l’handshake `connect` via WS.

<div id="remote-node-host-systemrun">
  ## Host remoto del nodo (system.run)
</div>

Usa un **host del nodo** quando il tuo Gateway è in esecuzione su una macchina e vuoi che i comandi
vengano eseguiti su un&#39;altra. Il modello continua a comunicare con il **Gateway**; il Gateway
inoltra le chiamate `exec` all&#39;**host del nodo** quando selezioni `host=node`.

<div id="what-runs-where">
  ### Cosa viene eseguito dove
</div>

* **Gateway host**: riceve i messaggi, esegue il modello, instrada le chiamate agli strumenti.
* **Host del nodo**: esegue `system.run`/`system.which` sulla macchina del nodo.
* **Approvazioni**: applicate sull&#39;host del nodo tramite `~/.openclaw/exec-approvals.json`.

<div id="start-a-node-host-foreground">
  ### Avvia un nodo host (in primo piano)
</div>

Sulla macchina che esegue il nodo:

```bash
openclaw node run --host <gateway-host> --port 18789 --display-name "Build Node"
```

<div id="start-a-node-host-service">
  ### Avviare l&#39;host del nodo (servizio)
</div>

```bash
openclaw node install --host <gateway-host> --port 18789 --display-name "Build Node"
openclaw node restart
```

<div id="pair-name">
  ### Effettua il pairing e assegna un nome
</div>

Sull&#39;host del Gateway:

```bash
openclaw nodes pending
openclaw nodes approve <requestId>
openclaw nodes list
```

Opzioni di denominazione:

* `--display-name` in `openclaw node run` / `openclaw node install` (viene salvato in modo persistente in `~/.openclaw/node.json` sul nodo).
* `openclaw nodes rename --node <id|name|ip> --name "Build Node"` (override lato Gateway).

<div id="allowlist-the-commands">
  ### Aggiungi i comandi alla lista di autorizzati
</div>

Le approvazioni di exec sono **specifiche per ogni nodo host**. Aggiungi le voci alla lista di autorizzati dal Gateway:

```bash
openclaw approvals allowlist add --node <id|name|ip> "/usr/bin/uname"
openclaw approvals allowlist add --node <id|name|ip> "/usr/bin/sw_vers"
```

Le approvazioni si trovano sull&#39;host del nodo in `~/.openclaw/exec-approvals.json`.

<div id="point-exec-at-the-node">
  ### Imposta exec per puntare al nodo
</div>

Configura i valori predefiniti (configurazione del Gateway):

```bash
openclaw config set tools.exec.host node
openclaw config set tools.exec.security allowlist
openclaw config set tools.exec.node "<id-or-name>"
```

Oppure per singola sessione:

```
/exec host=node security=allowlist node=<id-or-name>
```

Una volta impostato, qualsiasi chiamata `exec` con `host=node` viene eseguita sull&#39;host del nodo (nel rispetto della lista di autorizzati e delle approvazioni del nodo).

Correlato:

* [CLI dell&#39;host del nodo](/it/cli/node)
* [Strumento Exec](/it/tools/exec)
* [Approvazioni Exec](/it/tools/exec-approvals)

<div id="invoking-commands">
  ## Invocare i comandi
</div>

A basso livello (RPC raw):

```bash
openclaw nodes invoke --node <idOrNameOrIp> --command canvas.eval --params '{"javaScript":"location.href"}'
```

Sono disponibili helper di livello superiore per i flussi di lavoro più comuni che prevedono di “fornire all&#39;agente un allegato MEDIA”.

<div id="screenshots-canvas-snapshots">
  ## Screenshot (snapshot del Canvas)
</div>

Se il nodo sta mostrando il Canvas (WebView), `canvas.snapshot` restituisce `{ format, base64 }`.

Utility CLI (scrive su un file temporaneo e stampa `MEDIA:<path>`):

```bash
openclaw nodes canvas snapshot --node <idOrNameOrIp> --format png
openclaw nodes canvas snapshot --node <idOrNameOrIp> --format jpg --max-width 1200 --quality 0.9
```

<div id="canvas-controls">
  ### Controlli del canvas
</div>

```bash
openclaw nodes canvas present --node <idOrNameOrIp> --target https://example.com
openclaw nodes canvas hide --node <idOrNameOrIp>
openclaw nodes canvas navigate https://example.com --node <idOrNameOrIp>
openclaw nodes canvas eval --node <idOrNameOrIp> --js "document.title"
```

Note:

* `canvas present` accetta URL o percorsi di file locali (`--target`), oltre ai parametri opzionali `--x/--y/--width/--height` per il posizionamento.
* `canvas eval` accetta JS inline (`--js`) o un argomento posizionale.

<div id="a2ui-canvas">
  ### A2UI (Canvas)
</div>

```bash
openclaw nodes canvas a2ui push --node <idOrNameOrIp> --text "Ciao"
openclaw nodes canvas a2ui push --node <idOrNameOrIp> --jsonl ./payload.jsonl
openclaw nodes canvas a2ui reset --node <idOrNameOrIp>
```

Note:

* Solo A2UI v0.8 JSONL è supportato (v0.9/createSurface viene rifiutato).

<div id="photos-videos-node-camera">
  ## Foto + video (fotocamera del nodo)
</div>

Foto (`jpg`):

```bash
openclaw nodes camera list --node <idOrNameOrIp>
openclaw nodes camera snap --node <idOrNameOrIp>            # predefinito: entrambe le fotocamere (2 righe MEDIA)
openclaw nodes camera snap --node <idOrNameOrIp> --facing front
```

Clip video in formato `mp4`:

```bash
openclaw nodes camera clip --node <idOrNameOrIp> --duration 10s
openclaw nodes camera clip --node <idOrNameOrIp> --duration 3000 --no-audio
```

Note:

* Il nodo deve essere **in primo piano** per `canvas.*` e `camera.*` (le chiamate in background restituiscono `NODE_BACKGROUND_UNAVAILABLE`).
* La durata della clip è limitata (attualmente `<= 60s`) per evitare payload base64 troppo grandi.
* Quando possibile, Android richiederà le autorizzazioni `CAMERA`/`RECORD_AUDIO`; se negate, le richieste falliscono con `*_PERMISSION_REQUIRED`.

<div id="screen-recordings-nodes">
  ## Registrazioni dello schermo (nodi)
</div>

I nodi espongono `screen.record` (mp4). Esempio:

```bash
openclaw nodes screen record --node <idOrNameOrIp> --duration 10s --fps 10
openclaw nodes screen record --node <idOrNameOrIp> --duration 10s --fps 10 --no-audio
```

Note:

* `screen.record` richiede che l&#39;app del nodo sia in primo piano.
* Android visualizzerà il prompt di sistema per la cattura dello schermo prima di avviare la registrazione.
* Le registrazioni dello schermo sono limitate a `<= 60s`.
* `--no-audio` disattiva l&#39;acquisizione del microfono (supportato su iOS/Android; macOS utilizza l&#39;audio della cattura di sistema).
* Usa `--screen <index>` per selezionare un display quando sono disponibili più schermi.

<div id="location-nodes">
  ## Posizione (nodi)
</div>

I nodi espongono `location.get` quando Location è attivata nelle impostazioni.

Utility CLI:

```bash
openclaw nodes location get --node <idOrNameOrIp>
openclaw nodes location get --node <idOrNameOrIp> --accuracy precise --max-age 15000 --location-timeout 10000
```

Note:

* La posizione è **disattivata per impostazione predefinita**.
* L&#39;opzione &quot;Sempre&quot; richiede l&#39;autorizzazione di sistema; il recupero in background è su base best-effort.
* La risposta include lat/lon, precisione (metri) e timestamp.

<div id="sms-android-nodes">
  ## SMS (nodi Android)
</div>

I nodi Android possono esporre `sms.send` quando l&#39;utente concede l&#39;autorizzazione agli **SMS** e il dispositivo supporta funzionalità telefoniche.

Invocazione a basso livello:

```bash
openclaw nodes invoke --node <idOrNameOrIp> --command sms.send --params '{"to":"+15555550123","message":"Hello from OpenClaw"}'
```

Note:

* La richiesta di autorizzazione deve essere accettata sul dispositivo Android prima che la funzionalità venga annunciata.
* I dispositivi solo Wi‑Fi senza funzionalità di telefonia non annunceranno `sms.send`.

<div id="system-commands-node-host-mac-node">
  ## Comandi di sistema (host del nodo / nodo mac)
</div>

Il nodo macOS espone `system.run`, `system.notify` e `system.execApprovals.get/set`.
L&#39;host del nodo headless espone `system.run`, `system.which` e `system.execApprovals.get/set`.

Esempi:

```bash
openclaw nodes run --node <idOrNameOrIp> -- echo "Hello from mac node"
openclaw nodes notify --node <idOrNameOrIp> --title "Ping" --body "Gateway ready"
```

Note:

* `system.run` restituisce stdout/stderr/codice di uscita nel payload.
* `system.notify` rispetta lo stato dei permessi di notifica nell&#39;app macOS.
* `system.run` supporta `--cwd`, `--env KEY=VAL`, `--command-timeout` e `--needs-screen-recording`.
* `system.notify` supporta `--priority <passive|active|timeSensitive>` e `--delivery <system|overlay|auto>`.
* I nodi macOS scartano le sovrascritture di `PATH`; gli host nodo headless accettano `PATH` solo quando questo premette il PATH dell&#39;host nodo.
* In modalità nodo macOS, `system.run` è subordinato alle approvazioni di esecuzione nell&#39;app macOS (Impostazioni → Approvazioni di esecuzione).
  Ask/allowlist/full si comportano allo stesso modo dell&#39;host nodo headless; i prompt negati restituiscono `SYSTEM_RUN_DENIED`.
* Sull&#39;host nodo headless, `system.run` è subordinato alle approvazioni di esecuzione (`~/.openclaw/exec-approvals.json`).

<div id="exec-node-binding">
  ## Associazione di exec al nodo
</div>

Quando sono disponibili più nodi, puoi associare `exec` a un nodo specifico.
Questo imposta il nodo predefinito per `exec host=node` (e può essere sovrascritto per singolo agente).

Valore predefinito globale:

```bash
openclaw config set tools.exec.node "node-id-or-name"
```

Sovrascrittura per agente:

```bash
openclaw config get agents.list
openclaw config set agents.list[0].tools.exec.node "node-id-or-name"
```

Lascia vuoto per consentire qualsiasi nodo:

```bash
openclaw config unset tools.exec.node
openclaw config unset agents.list[0].tools.exec.node
```

<div id="permissions-map">
  ## Mappa dei permessi
</div>

I nodi possono includere una mappa `permissions` in `node.list` / `node.describe`, indicizzata per nome del permesso (ad es. `screenRecording`, `accessibility`) con valori booleani (`true` = concesso).

<div id="headless-node-host-cross-platform">
  ## Host headless del nodo (multipiattaforma)
</div>

OpenClaw può eseguire un **host headless del nodo** (senza UI) che si connette al WebSocket
del Gateway ed espone `system.run` / `system.which`. È utile su Linux/Windows
o per eseguire un nodo minimale affiancato a un server.

Avvialo:

```bash
openclaw node run --host <gateway-host> --port 18789
```

Note:

* L&#39;abbinamento è ancora necessario (il Gateway mostrerà un prompt di approvazione per il nodo).
* L&#39;host del nodo memorizza il suo ID del nodo, il token, il nome visualizzato e le informazioni di connessione al Gateway in `~/.openclaw/node.json`.
* Le approvazioni di esecuzione vengono applicate localmente tramite `~/.openclaw/exec-approvals.json`
  (consulta [Exec approvals](/it/tools/exec-approvals)).
* Su macOS, l&#39;host del nodo headless preferisce l&#39;host di esecuzione dell&#39;app companion quando è raggiungibile e, in caso contrario, passa all&#39;esecuzione locale se l&#39;app non è disponibile. Imposta `OPENCLAW_NODE_EXEC_HOST=app` per richiedere l&#39;app oppure `OPENCLAW_NODE_EXEC_FALLBACK=0` per disabilitare il fallback.
* Aggiungi `--tls` / `--tls-fingerprint` quando il WS del Gateway usa TLS.

<div id="mac-node-mode">
  ## Modalità nodo Mac
</div>

* L&#39;app della barra dei menu di macOS si connette al server WS del Gateway come nodo (in modo che `openclaw nodes …` funzioni con questo Mac).
* In modalità remota, l&#39;app apre un tunnel SSH verso la porta del Gateway e si connette a `localhost`.