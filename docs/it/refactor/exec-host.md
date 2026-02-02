---
title: Host di esecuzione
summary: "Piano di refactoring: instradamento dell'host di esecuzione, approvazioni dei nodi e runner headless"
read_when:
  - Quando progetti l'instradamento dell'host di esecuzione o le approvazioni dell'esecuzione
  - Quando implementi il runner del nodo + IPC della UI
  - Quando aggiungi modalità di sicurezza dell'host di esecuzione e comandi slash
---

<div id="exec-host-refactor-plan">
  # Piano di refactoring per l&#39;host Exec
</div>

<div id="goals">
  ## Obiettivi
</div>

* Aggiungere `exec.host` + `exec.security` per instradare l&#39;esecuzione tra **sandbox**, **Gateway** e **nodo**.
* Mantenere i valori predefiniti **sicuri**: nessuna esecuzione cross-host a meno che non sia esplicitamente abilitata.
* Suddividere l&#39;esecuzione in un **servizio di esecuzione headless** con UI opzionale (app macOS) tramite IPC locale.
* Fornire criteri **per singolo agente**, lista di autorizzati, modalità di richiesta e associazione al nodo.
* Supportare **modalità di richiesta** che funzionino *con* o *senza* lista di autorizzati.
* Multipiattaforma: socket Unix + autenticazione tramite token (stessa funzionalità su macOS/Linux/Windows).

<div id="non-goals">
  ## Ambiti esclusi
</div>

* Nessuna migrazione della lista di autorizzati legacy né supporto per lo schema legacy.
* Nessun PTY/streaming per l&#39;esecuzione sul nodo (solo output aggregato).
* Nessun nuovo livello di rete oltre all&#39;esistente Bridge + Gateway.

<div id="decisions-locked">
  ## Decisioni (definitive)
</div>

* **Config keys:** `exec.host` + `exec.security` (override per-agente consentito).
* **Elevation:** mantenere `/elevated` come alias per l&#39;accesso completo al Gateway.
* **Ask default:** valore predefinito di ask: `on-miss`.
* **Approvals store:** `~/.openclaw/exec-approvals.json` (JSON, nessuna migrazione legacy).
* **Runner:** servizio di sistema headless; l&#39;app UI espone un socket Unix per le approvazioni.
* **Node identity:** usare il `nodeId` esistente.
* **Socket auth:** socket Unix + token (multipiattaforma); da separare in seguito se necessario.
* **Node host state:** `~/.openclaw/node.json` (id del nodo + token di abbinamento).
* **macOS exec host:** eseguire `system.run` all&#39;interno dell&#39;app macOS; il servizio host del nodo inoltra le richieste tramite IPC locale.
* **No XPC helper:** mantenere socket Unix + token + controlli sul peer.

<div id="key-concepts">
  ## Concetti chiave
</div>

<div id="host">
  ### Host
</div>

* `sandbox`: exec Docker (comportamento attuale).
* `gateway`: exec sull&#39;host del Gateway.
* `node`: exec sul runner del nodo tramite Bridge (`system.run`).

<div id="security-mode">
  ### Modalità di sicurezza
</div>

* `deny`: blocca sempre.
* `allowlist`: consenti solo ciò che è presente nella lista di autorizzati.
* `full`: consenti tutto (equivalente a `elevated`).

<div id="ask-mode">
  ### Modalità di richiesta
</div>

* `off`: non chiedere mai.
* `on-miss`: chiedi solo quando non c’è corrispondenza con la lista di autorizzati.
* `always`: chiedi ogni volta.

La richiesta è **indipendente** dalla lista di autorizzati; la lista di autorizzati può essere utilizzata con `always` o `on-miss`.

<div id="policy-resolution-per-exec">
  ### Risoluzione delle policy (per exec)
</div>

1. Determina `exec.host` (parametro dello strumento → override dell&#39;agente → valore predefinito globale).
2. Determina `exec.security` e `exec.ask` (con la stessa precedenza).
3. Se l&#39;host è `sandbox`, procedi con l&#39;esecuzione in sandbox locale.
4. Se l&#39;host è `gateway` o `node`, applica le policy di sicurezza e di ask su quell&#39;host.

<div id="default-safety">
  ## Sicurezza predefinita
</div>

* Impostazione predefinita: `exec.host = sandbox`.
* Impostazione predefinita: `exec.security = deny` per `Gateway` e `nodo`.
* Impostazione predefinita: `exec.ask = on-miss` (rilevante solo se le impostazioni di sicurezza lo consentono).
* Se non è configurato alcun binding al nodo, **l&#39;agente può puntare a qualsiasi nodo**, ma solo se la policy lo consente.

<div id="config-surface">
  ## Ambito di configurazione
</div>

<div id="tool-parameters">
  ### Parametri degli strumenti
</div>

* `exec.host` (facoltativo): `sandbox | gateway | node`.
* `exec.security` (facoltativo): `deny | allowlist | full`.
* `exec.ask` (facoltativo): `off | on-miss | always`.
* `exec.node` (facoltativo): ID/nome del nodo da utilizzare quando `host=node`.

<div id="config-keys-global">
  ### Chiavi di configurazione (globali)
</div>

* `tools.exec.host`
* `tools.exec.security`
* `tools.exec.ask`
* `tools.exec.node` (associazione al nodo predefinito)

<div id="config-keys-per-agent">
  ### Chiavi di configurazione (per agente)
</div>

* `agents.list[].tools.exec.host`
* `agents.list[].tools.exec.security`
* `agents.list[].tools.exec.ask`
* `agents.list[].tools.exec.node`

<div id="alias">
  ### Alias
</div>

* `/elevated on` = imposta `tools.exec.host=gateway`, `tools.exec.security=full` per la sessione dell&#39;agente.
* `/elevated off` = ripristina le precedenti impostazioni di esecuzione per la sessione dell&#39;agente.

<div id="approvals-store-json">
  ## Archivio approvazioni (JSON)
</div>

Percorso: `~/.openclaw/exec-approvals.json`

Scopo:

* Criteri locali + lista di autorizzati per l&#39;**host di esecuzione** (Gateway o nodo runner).
* Richiedere conferma come fallback quando non è disponibile alcuna UI.
* Credenziali IPC per i client della UI.

Schema proposto (v1):

```json
{
  "version": 1,
  "socket": {
    "path": "~/.openclaw/exec-approvals.sock",
    "token": "base64-opaque-token"
  },
  "defaults": {
    "security": "deny",
    "ask": "on-miss",
    "askFallback": "deny"
  },
  "agents": {
    "agent-id-1": {
      "security": "allowlist",
      "ask": "on-miss",
      "allowlist": [
        {
          "pattern": "~/Projects/**/bin/rg",
          "lastUsedAt": 0,
          "lastUsedCommand": "rg -n TODO",
          "lastResolvedPath": "/Users/user/Projects/.../bin/rg"
        }
      ]
    }
  }
}
```

Note:

* Nessun formato legacy per la lista di autorizzati.
* `askFallback` si applica solo quando `ask` è richiesto e non è raggiungibile alcuna UI.
* Permessi del file: `0600`.

<div id="runner-service-headless">
  ## Servizio Runner (headless)
</div>

<div id="role">
  ### Ruolo
</div>

* Applicare localmente `exec.security` + `exec.ask`.
* Eseguire comandi di sistema e restituire l&#39;output.
* Emettere eventi Bridge relativi al ciclo di vita di exec (facoltativo ma consigliato).

<div id="service-lifecycle">
  ### Ciclo di vita del servizio
</div>

* launchd/daemon su macOS; servizio di sistema su Linux/Windows.
* Il JSON delle approvazioni è locale sull&#39;host di esecuzione.
* La UI espone un socket Unix locale; i runner si connettono su richiesta.

<div id="ui-integration-macos-app">
  ## Integrazione con la UI (app macOS)
</div>

<div id="ipc">
  ### IPC
</div>

* Socket Unix in `~/.openclaw/exec-approvals.sock` (0600).
* Token memorizzato in `exec-approvals.json` (0600).
* Controlli peer: stesso UID soltanto.
* Challenge/response: nonce + HMAC(token, request-hash) per prevenire attacchi di replay.
* TTL breve (ad es. 10s) + dimensione massima del payload + limitazione della frequenza (rate limit).

<div id="ask-flow-macos-app-exec-host">
  ### Flusso Ask (host di esecuzione app macOS)
</div>

1. Il servizio del nodo riceve `system.run` dal Gateway.
2. Il servizio del nodo si connette al socket locale e invia la richiesta di prompt/exec.
3. L&#39;app convalida peer + token + HMAC + TTL, quindi mostra la finestra di dialogo se necessario.
4. L&#39;app esegue il comando nel contesto UI e restituisce l&#39;output.
5. Il servizio del nodo restituisce l&#39;output al Gateway.

Se la UI non è disponibile:

* Applica `askFallback` (`deny|allowlist|full`).

<div id="diagram-sci">
  ### Schema (SCI)
</div>

```
Agent -> Gateway -> Bridge -> Node Service (TS)
                         |  IPC (UDS + token + HMAC + TTL)
                         v
                     Mac App (UI + TCC + system.run)
```

<div id="node-identity-binding">
  ## Identità del nodo + associazione
</div>

* Usa il `nodeId` esistente dall&#39;abbinamento Bridge.
* Modello di associazione:
  * `tools.exec.node` limita l&#39;agente a un nodo specifico.
  * Se non impostato, l&#39;agente può scegliere qualsiasi nodo (la policy continua a far rispettare i valori predefiniti).
* Logica di selezione del nodo:
  * corrispondenza esatta su `nodeId`
  * `displayName` (normalizzato)
  * `remoteIp`
  * prefisso di `nodeId` (&gt;= 6 caratteri)

<div id="eventing">
  ## Gestione eventi
</div>

<div id="who-sees-events">
  ### Chi vede gli eventi
</div>

* Gli eventi di sistema sono **specifici per sessione** e vengono mostrati all&#39;agente al prompt successivo.
* Sono memorizzati nella coda in-memory del Gateway (`enqueueSystemEvent`).

<div id="event-text">
  ### Testo degli eventi
</div>

* `Exec started (node=<id>, id=<runId>)`
* `Exec finished (node=<id>, id=<runId>, code=<code>)` + coda opzionale dell&#39;output
* `Exec denied (node=<id>, id=<runId>, <reason>)`

<div id="transport">
  ### Trasporto
</div>

Opzione A (consigliata):

* Runner invia al Bridge frame di tipo `event` `exec.started` / `exec.finished`.
* Gateway, tramite `handleBridgeEvent`, li mappa su `enqueueSystemEvent`.

Opzione B:

* Lo strumento `exec` del Gateway gestisce direttamente il ciclo di vita (solo sincrono).

<div id="exec-flows">
  ## Flussi di Exec
</div>

<div id="sandbox-host">
  ### Host sandbox
</div>

* Comportamento `exec` attuale (Docker o host quando non in sandbox).
* Il PTY è supportato solo in modalità non sandbox.

<div id="gateway-host">
  ### Host del Gateway
</div>

* Il processo Gateway viene eseguito su una macchina dedicata.
* Applica il file di configurazione locale `exec-approvals.json` (sicurezza/richiesta/lista di autorizzati).

<div id="node-host">
  ### Host del nodo
</div>

* Gateway chiama `node.invoke` con `system.run`.
* Runner applica le autorizzazioni locali.
* Runner restituisce stdout/stderr aggregati.
* Eventi Bridge opzionali per avvio/completamento/rifiuto.

<div id="output-caps">
  ## Limiti di output
</div>

* Limita stdout+stderr combinati a **200k**; conserva gli **ultimi 20k** per gli eventi.
* Tronca con un suffisso chiaro (ad esempio `"… (truncated)"`).

<div id="slash-commands">
  ## Comandi slash
</div>

* `/exec host=<sandbox|gateway|node> security=<deny|allowlist|full> ask=<off|on-miss|always> node=<id>`
* Override per agente e per sessione; non sono persistenti a meno che non vengano salvati nella config.
* `/elevated on|off|ask|full` continua a essere una scorciatoia per `host=gateway security=full` (con `full` che salta le approvazioni).

<div id="cross-platform-story">
  ## Modello cross-platform
</div>

* Il servizio runner è la destinazione di esecuzione portabile.
* La UI è opzionale; se assente, viene applicato `askFallback`.
* Windows e Linux supportano lo stesso protocollo di approvazione JSON + socket.

<div id="implementation-phases">
  ## Fasi di implementazione
</div>

<div id="phase-1-config-exec-routing">
  ### Fase 1: configurazione + routing di exec
</div>

* Aggiungi lo schema di configurazione per `exec.host`, `exec.security`, `exec.ask`, `exec.node`.
* Aggiorna il plumbing interno dei tool in modo che rispettino `exec.host`.
* Aggiungi il comando slash `/exec` e mantieni l&#39;alias `/elevated`.

<div id="phase-2-approvals-store-gateway-enforcement">
  ### Fase 2: archivio approvazioni + enforcement nel Gateway
</div>

* Implementare il lettore/scrittore per `exec-approvals.json`.
* Applicare la lista di autorizzati e le modalità ask per l&#39;host `Gateway`.
* Aggiungere limiti massimi di output.

<div id="phase-3-node-runner-enforcement">
  ### Fase 3: applicazione nel node runner
</div>

* Aggiorna il node runner per applicare la lista di autorizzati + ask.
* Aggiungi un bridge di prompt tramite socket Unix all&#39;UI dell&#39;app macOS.
* Collega `askFallback`.

<div id="phase-4-events">
  ### Fase 4: eventi
</div>

* Aggiungi eventi Bridge nodo → Gateway per il ciclo di vita dell&#39;esecuzione.
* Collega a `enqueueSystemEvent` per i prompt degli agenti.

<div id="phase-5-ui-polish">
  ### Fase 5: rifinitura della UI
</div>

* App Mac: editor della lista di autorizzati, selettore per agente, UI dei criteri di richiesta.
* Controlli di binding del nodo (opzionale).

<div id="testing-plan">
  ## Piano di test
</div>

* Test unitari: corrispondenza con la lista di autorizzati (glob + case-insensitive).
* Test unitari: risoluzione delle policy in ordine di precedenza (parametro dello strumento → override dell&#39;agente → globale).
* Test di integrazione: flussi deny/allow/ask del node runner.
* Test sugli eventi del bridge: instradamento evento del nodo → evento di sistema.

<div id="open-risks">
  ## Rischi aperti
</div>

* Indisponibilità della UI: verifica che `askFallback` venga rispettato.
* Comandi di lunga esecuzione: affidati a timeout + limiti di output.
* Ambiguità multi-nodo: genera un errore salvo binding del nodo o parametro di nodo esplicito.

<div id="related-docs">
  ## Documentazione correlata
</div>

* [Strumento Exec](/it/tools/exec)
* [Approvazioni Exec](/it/tools/exec-approvals)
* [Nodi](/it/nodes)
* [Modalità con privilegi elevati](/it/tools/elevated)