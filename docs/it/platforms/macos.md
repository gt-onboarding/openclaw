---
title: Macos
summary: "App companion di OpenClaw per macOS (barra dei menu + broker del Gateway)"
read_when:
  - Implementazione di funzionalità dell'app per macOS
  - Modifica del ciclo di vita del Gateway o del bridging del nodo su macOS
---

<div id="openclaw-macos-companion-menu-bar-gateway-broker">
  # OpenClaw macOS Companion (barra dei menu + broker del Gateway)
</div>

L&#39;app macOS è il **companion nella barra dei menu** per OpenClaw. Gestisce i permessi,
si collega/localmente al Gateway (tramite launchd o manualmente) ed espone le
funzionalità di macOS all&#39;agente come nodo.

<div id="what-it-does">
  ## Che cosa fa
</div>

* Mostra notifiche native e lo stato nella barra dei menu.
* Gestisce i prompt TCC (Notifiche, Accessibilità, Registrazione schermo, Microfono,
  Riconoscimento vocale, Automazione/AppleScript).
* Esegue o si connette al Gateway (locale o remoto).
* Espone strumenti esclusivi per macOS (Canvas, Fotocamera, Registrazione schermo, `system.run`).
* Avvia il servizio host del nodo locale in modalità **remota** (launchd) e lo arresta in modalità **locale**.
* Facoltativamente ospita **PeekabooBridge** per l&#39;automazione della UI.
* Su richiesta installa la CLI globale (`openclaw`) tramite npm/pnpm (bun è sconsigliato per il runtime del Gateway).

<div id="local-vs-remote-mode">
  ## Modalità locale vs remota
</div>

* **Locale** (impostazione predefinita): l&#39;app si connette a un Gateway locale già in esecuzione, se presente;
  in caso contrario attiva il servizio launchd tramite `openclaw gateway install`.
* **Remota**: l&#39;app si connette a un Gateway tramite SSH/Tailscale e non avvia mai
  un processo locale del Gateway.
  L&#39;app avvia il **servizio host del nodo** locale in modo che il Gateway remoto possa raggiungere questo Mac.
  L&#39;app non esegue il Gateway come processo figlio.

<div id="launchd-control">
  ## Controllo tramite launchd
</div>

L&#39;app gestisce un LaunchAgent per utente etichettato `bot.molt.gateway`
(oppure `bot.molt.<profile>` quando utilizzi `--profile`/`OPENCLAW_PROFILE`; i vecchi `com.openclaw.*` vengono comunque disattivati).

```bash
launchctl kickstart -k gui/$UID/bot.molt.gateway
launchctl bootout gui/$UID/bot.molt.gateway
```

Sostituisci l&#39;etichetta con `bot.molt.<profile>` quando esegui un profilo denominato.

Se il LaunchAgent non è installato, puoi abilitarlo dall&#39;app oppure eseguendo
`openclaw gateway install`.

<div id="node-capabilities-mac">
  ## Capacità del nodo (Mac)
</div>

L’app macOS si presenta come un nodo. Comandi comuni:

* Canvas: `canvas.present`, `canvas.navigate`, `canvas.eval`, `canvas.snapshot`, `canvas.a2ui.*`
* Camera: `camera.snap`, `camera.clip`
* Schermo: `screen.record`
* Sistema: `system.run`, `system.notify`

Il nodo espone una mappa `permissions` così che gli agenti possano decidere cosa è consentito.

Servizio del nodo + IPC dell’app:

* Quando il servizio host headless del nodo è in esecuzione (modalità remota), si connette al WS del Gateway come nodo.
* `system.run` viene eseguito nell’app macOS (contesto UI/TCC) tramite una socket Unix locale; prompt e output rimangono all’interno dell’app.

Diagramma (SCI):

```
Gateway -> Node Service (WS)
                 |  IPC (UDS + token + HMAC + TTL)
                 v
             Mac App (UI + TCC + system.run)
```

<div id="exec-approvals-systemrun">
  ## Approvazioni Exec (system.run)
</div>

`system.run` è controllato tramite **Exec approvals** nell&#39;app macOS (Settings → Exec approvals).
Le impostazioni di sicurezza, le richieste e la lista di autorizzati vengono memorizzate localmente sul Mac in:

```
~/.openclaw/exec-approvals.json
```

Esempio:

```json
{
  "version": 1,
  "defaults": {
    "security": "deny",
    "ask": "on-miss"
  },
  "agents": {
    "main": {
      "security": "allowlist",
      "ask": "on-miss",
      "allowlist": [
        { "pattern": "/opt/homebrew/bin/rg" }
      ]
    }
  }
}
```

Note:

* Le voci di `allowlist` sono pattern glob per i percorsi binari risolti.
* La scelta di &quot;Always Allow&quot; nel prompt aggiunge quel comando alla lista di autorizzati.
* Gli override dell&#39;ambiente di `system.run` vengono filtrati (vengono rimossi `PATH`, `DYLD_*`, `LD_*`, `NODE_OPTIONS`, `PYTHON*`, `PERL*`, `RUBYOPT`) e quindi uniti all&#39;ambiente dell&#39;applicazione.

<div id="deep-links">
  ## Deep links
</div>

L&#39;app registra lo schema di URL `openclaw://` per le azioni locali.

<div id="openclawagent">
  ### `openclaw://agent`
</div>

Attiva una richiesta di tipo `agent` al Gateway.

```bash
open 'openclaw://agent?message=Hello%20from%20deep%20link'
```

Parametri di query:

* `message` (obbligatorio)
* `sessionKey` (opzionale)
* `thinking` (opzionale)
* `deliver` / `to` / `channel` (opzionale)
* `timeoutSeconds` (opzionale)
* `key` (chiave opzionale per la modalità non presidiata)

Sicurezza:

* Senza `key`, l&#39;app richiede conferma.
* Con una `key` valida, l&#39;esecuzione avviene in modalità non presidiata (pensata per automazioni personali).

<div id="onboarding-flow-typical">
  ## Flusso di onboarding (tipico)
</div>

1. Installa e avvia **OpenClaw.app**.
2. Completa la checklist delle autorizzazioni (prompt TCC).
3. Assicurati che la modalità **Local** sia attiva e che il Gateway sia in esecuzione.
4. Installa la CLI se vuoi l’accesso dal terminale.

<div id="build-dev-workflow-native">
  ## Workflow di build e sviluppo (nativo)
</div>

* `cd apps/macos && swift build`
* `swift run OpenClaw` (oppure Xcode)
* Crea il pacchetto dell&#39;app: `scripts/package-mac-app.sh`

<div id="debug-gateway-connectivity-macos-cli">
  ## Esegui il debug della connettività del Gateway (CLI macOS)
</div>

Usa la CLI di debug per riprodurre lo stesso handshake e la stessa logica di discovery WebSocket del Gateway
utilizzati dall&#39;app macOS, senza avviare l&#39;app.

```bash
cd apps/macos
swift run openclaw-mac connect --json
swift run openclaw-mac discover --timeout 3000 --json
```

Opzioni di connessione:

* `--url <ws://host:port>`: sovrascrive la configurazione
* `--mode <local|remote>`: ricava il valore dalla configurazione (predefinito: config o local)
* `--probe`: forza un nuovo controllo di stato (health probe)
* `--timeout <ms>`: timeout della richiesta (predefinito: `15000`)
* `--json`: output strutturato per il confronto (diff)

Opzioni di discovery:

* `--include-local`: includi i gateway che verrebbero filtrati come “local”
* `--timeout <ms>`: finestra di discovery complessiva (predefinito: `2000`)
* `--json`: output strutturato per il confronto (diff)

Suggerimento: confronta con `openclaw gateway discover --json` per verificare se la
pipeline di discovery dell’app macOS (NWBrowser + fallback DNS‑SD su tailnet) differisce
dal meccanismo di discovery basato su `dns-sd` della CLI del nodo.

<div id="remote-connection-plumbing-ssh-tunnels">
  ## Infrastruttura delle connessioni remote (tunnel SSH)
</div>

Quando l&#39;app macOS è in modalità **Remote**, apre un tunnel SSH in modo che i componenti della UI locale possano comunicare con un Gateway remoto come se fosse in localhost.

<div id="control-tunnel-gateway-websocket-port">
  ### Tunnel di controllo (porta WebSocket del Gateway)
</div>

* **Scopo:** health check, stato, Web Chat, configurazione e altre chiamate del piano di controllo.
* **Porta locale:** la porta del Gateway (predefinita `18789`), sempre stabile.
* **Porta remota:** la stessa porta del Gateway sull&#39;host remoto.
* **Comportamento:** nessuna porta locale casuale; l&#39;app riutilizza un tunnel
  esistente e funzionante oppure lo riavvia se necessario.
* **Comando SSH:** `ssh -N -L <local>:127.0.0.1:<remote>` con le opzioni BatchMode +
  ExitOnForwardFailure + keepalive.
* **Segnalazione IP:** il tunnel SSH usa il loopback, quindi il Gateway vedrà l&#39;IP del nodo
  come `127.0.0.1`. Usa il trasporto **Direct (ws/wss)** se vuoi che compaia il vero IP
  del client (vedi [accesso remoto macOS](/it/platforms/mac/remote)).

Per i passaggi di configurazione, vedi [accesso remoto macOS](/it/platforms/mac/remote). Per i dettagli
sul protocollo, vedi [protocollo del Gateway](/it/gateway/protocol).

<div id="related-docs">
  ## Documentazione correlata
</div>

* [Runbook del Gateway](/it/gateway)
* [Gateway (macOS)](/it/platforms/mac/bundled-gateway)
* [Autorizzazioni macOS](/it/platforms/mac/permissions)
* [Canvas](/it/platforms/mac/canvas)