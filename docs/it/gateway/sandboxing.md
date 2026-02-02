---
summary: "Come funziona il sandboxing in OpenClaw: modalità, scope, accesso allo spazio di lavoro e immagini"
title: Sandboxing
read_when: "Quando ti serve una spiegazione approfondita del sandboxing o devi configurare agents.defaults.sandbox."
status: active
---

<div id="sandboxing">
  # Sandboxing
</div>

OpenClaw può eseguire **tool all&#39;interno di container Docker** per ridurre l&#39;impatto potenziale dei danni.
Questa è una funzionalità **opzionale** ed è controllata dalla configurazione (`agents.defaults.sandbox` o
`agents.list[].sandbox`). Se il sandboxing è disattivato, i tool vengono eseguiti sull&#39;host.
Il Gateway rimane sull&#39;host; l&#39;esecuzione dei tool avviene in una sandbox isolata
quando è abilitata.

Questo non costituisce un confine di sicurezza perfetto, ma limita in modo sostanziale l&#39;accesso al filesystem
e ai processi quando il modello fa qualcosa di stupido.

<div id="what-gets-sandboxed">
  ## Cosa viene eseguito nella sandbox
</div>

* Esecuzione degli strumenti (`exec`, `read`, `write`, `edit`, `apply_patch`, `process`, ecc.).
* Browser opzionale in sandbox (`agents.defaults.sandbox.browser`).
  * Per impostazione predefinita, il browser in sandbox si avvia automaticamente (garantisce che CDP sia raggiungibile) quando lo strumento del browser ne ha bisogno.
    Configuralo tramite `agents.defaults.sandbox.browser.autoStart` e `agents.defaults.sandbox.browser.autoStartTimeoutMs`.
  * `agents.defaults.sandbox.browser.allowHostControl` permette alle sessioni in sandbox di indirizzare esplicitamente il browser dell&#39;host.
  * Liste di autorizzati opzionali limitano `target: "custom"`: `allowedControlUrls`, `allowedControlHosts`, `allowedControlPorts`.

Non in sandbox:

* Il processo del Gateway stesso.
* Qualsiasi strumento esplicitamente autorizzato a essere eseguito sull&#39;host (ad es. `tools.elevated`).
  * **L&#39;esecuzione elevata (elevated exec) viene eseguita sull&#39;host e bypassa la sandbox.**
  * Se la sandbox è disattivata, `tools.elevated` non modifica l&#39;esecuzione (è già sull&#39;host). Vedi [Elevated Mode](/it/tools/elevated).

<div id="modes">
  ## Modalità
</div>

`agents.defaults.sandbox.mode` controlla **quando** viene utilizzata la sandbox:

* `"off"`: nessuna sandbox.
* `"non-main"`: sandbox solo per le sessioni **non-main** (valore predefinito se vuoi che le chat normali girino sull&#39;host).
* `"all"`: ogni sessione viene eseguita in una sandbox.
  Nota: `"non-main"` si basa su `session.mainKey` (predefinito `"main"`), non sull&#39;ID dell&#39;agente.
  Le sessioni di gruppo/canale usano le proprie chiavi, quindi sono considerate non-main e verranno eseguite in sandbox.

<div id="scope">
  ## Ambito
</div>

`agents.defaults.sandbox.scope` controlla **quanti container** vengono creati:

* `"session"` (predefinito): un container per ogni sessione.
* `"agent"`: un container per ogni agente.
* `"shared"`: un container condiviso tra tutte le sessioni in sandbox.

<div id="workspace-access">
  ## Accesso allo spazio di lavoro
</div>

`agents.defaults.sandbox.workspaceAccess` controlla **cosa può vedere la sandbox**:

* `"none"` (predefinito): gli strumenti vedono uno spazio di lavoro della sandbox sotto `~/.openclaw/sandboxes`.
* `"ro"`: monta lo spazio di lavoro dell&#39;agente in sola lettura su `/agent` (disabilita `write`/`edit`/`apply_patch`).
* `"rw"`: monta lo spazio di lavoro dell&#39;agente in lettura/scrittura su `/workspace`.

I contenuti multimediali in ingresso vengono copiati nello spazio di lavoro della sandbox attiva (`media/inbound/*`).
Nota sulle abilità: lo strumento `read` ha la root nella sandbox. Con `workspaceAccess: "none"`,
OpenClaw replica le abilità idonee nello spazio di lavoro della sandbox (`.../skills`) in modo che
possano essere lette. Con `"rw"`, le abilità dello spazio di lavoro sono leggibili da
`/workspace/skills`.

<div id="custom-bind-mounts">
  ## Bind mount personalizzati
</div>

`agents.defaults.sandbox.docker.binds` monta directory aggiuntive dell&#39;host nel container.
Formato: `host:container:mode` (ad es. `"/home/user/source:/source:rw"`).

I bind globali e per agente vengono **combinati** (non sostituiti). Con `scope: "shared"`, i bind per agente vengono ignorati.

Esempio (sorgente in sola lettura + socket Docker):

```json5
{
  agents: {
    defaults: {
      sandbox: {
        docker: {
          binds: [
            "/home/user/source:/source:ro",
            "/var/run/docker.sock:/var/run/docker.sock"
          ]
        }
      }
    },
    list: [
      {
        id: "build",
        sandbox: {
          docker: {
            binds: ["/mnt/cache:/cache:rw"]
          }
        }
      }
    ]
  }
}
```

Note sulla sicurezza:

* I bind aggirano il filesystem della sandbox: espongono percorsi dell&#39;host con qualsiasi modalità tu imposti (`:ro` o `:rw`).
* I mount sensibili (ad esempio `docker.sock`, segreti, chiavi SSH) dovrebbero essere `:ro` a meno che l&#39;accesso in scrittura non sia assolutamente necessario.
* Combina con `workspaceAccess: "ro"` se ti serve solo l&#39;accesso in sola lettura allo spazio di lavoro; le modalità dei bind restano indipendenti.
* Vedi [Sandbox vs Tool Policy vs Elevated](/it/gateway/sandbox-vs-tool-policy-vs-elevated) per capire come i bind interagiscono con la policy degli strumenti e l&#39;esecuzione con privilegi elevati.

<div id="images-setup">
  ## Immagini + configurazione
</div>

Immagine predefinita: `openclaw-sandbox:bookworm-slim`

Compilala una sola volta:

```bash
scripts/sandbox-setup.sh
```

Nota: l&#39;immagine predefinita **non** include Node. Se una skill richiede Node (o
altri runtime), crea un&#39;immagine personalizzata oppure installalo tramite
`sandbox.docker.setupCommand` (richiede traffico uscente verso la rete +
filesystem root scrivibile + utente root).

Immagine del browser in sandbox:

```bash
scripts/sandbox-browser-setup.sh
```

Per impostazione predefinita, i container della sandbox vengono eseguiti **senza accesso alla rete**.
Puoi sovrascrivere questa impostazione con `agents.defaults.sandbox.docker.network`.

Le installazioni di Docker e il Gateway containerizzato si trovano qui:
[Docker](/it/install/docker)

<div id="setupcommand-one-time-container-setup">
  ## setupCommand (configurazione iniziale del container)
</div>

`setupCommand` viene eseguito **una sola volta** dopo che il container della sandbox è stato creato (non a ogni esecuzione).
Viene eseguito all&#39;interno del container tramite `sh -lc`.

Percorsi:

* Globale: `agents.defaults.sandbox.docker.setupCommand`
* Per agente: `agents.list[].sandbox.docker.setupCommand`

Errori comuni:

* Il valore predefinito di `docker.network` è `"none"` (nessun egress), quindi l&#39;installazione dei pacchetti fallirà.
* `readOnlyRoot: true` impedisce le scritture; imposta `readOnlyRoot: false` oppure incorpora un&#39;immagine personalizzata.
* `user` deve essere root per installare pacchetti (ometti `user` oppure imposta `user: "0:0"`).
* L&#39;esecuzione nella sandbox **non** eredita il `process.env` dell&#39;host. Usa
  `agents.defaults.sandbox.docker.env` (o un&#39;immagine personalizzata) per le chiavi API delle skill.

<div id="tool-policy-escape-hatches">
  ## Criteri degli strumenti + vie di fuga
</div>

I criteri di allow/deny degli strumenti si applicano comunque prima delle regole della sandbox. Se uno strumento è bloccato
globalmente o per agente, il sandboxing non lo riabilita.

`tools.elevated` è una via di fuga esplicita che esegue `exec` sull&#39;host.
Le direttive `/exec` si applicano solo ai mittenti autorizzati e persistono per sessione; per disabilitare definitivamente
`exec`, usa un criterio degli strumenti con deny (vedi [Sandbox vs Tool Policy vs Elevated](/it/gateway/sandbox-vs-tool-policy-vs-elevated)).

Debugging:

* Usa `openclaw sandbox explain` per ispezionare la modalità sandbox effettiva, i criteri degli strumenti e le chiavi di configurazione di “fix-it”.
* Vedi [Sandbox vs Tool Policy vs Elevated](/it/gateway/sandbox-vs-tool-policy-vs-elevated) per il modello mentale «perché questo è bloccato?».
  Mantienilo ben protetto.

<div id="multi-agent-overrides">
  ## Override multi-agente
</div>

Ogni agente può definire override per sandbox e strumenti:
`agents.list[].sandbox` e `agents.list[].tools` (più `agents.list[].tools.sandbox.tools` per la policy degli strumenti nella sandbox).
Consulta [Sandbox e strumenti multi-agente](/it/multi-agent-sandbox-tools) per l&#39;ordine di precedenza.

<div id="minimal-enable-example">
  ## Esempio minimo di attivazione
</div>

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main",
        scope: "session",
        workspaceAccess: "none"
      }
    }
  }
}
```

<div id="related-docs">
  ## Documenti correlati
</div>

* [Configurazione della sandbox](/it/gateway/configuration#agentsdefaults-sandbox)
* [Sandbox multi-agente e strumenti](/it/multi-agent-sandbox-tools)
* [Sicurezza](/it/gateway/security)