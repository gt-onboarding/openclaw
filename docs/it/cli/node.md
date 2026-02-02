---
title: Node
summary: "Riferimento CLI per `openclaw node` (host nodo headless)"
read_when:
  - Esecuzione dell'host nodo headless
  - Abbinamento di un nodo non macOS per system.run
---

<div id="openclaw-node">
  # `openclaw node`
</div>

Esegui un **nodo host headless** che si connette al WebSocket del Gateway ed espone
`system.run` e `system.which` su questa macchina.

<div id="why-use-a-node-host">
  ## Perché usare un nodo host?
</div>

Usa un nodo host quando vuoi che gli agenti **eseguano comandi su altre macchine**
nella tua rete senza installare lì una companion app completa per macOS.

Casi d&#39;uso comuni:

* Eseguire comandi su server Linux/Windows remoti (server di build, macchine di laboratorio, NAS).
* Mantenere l&#39;esecuzione **in sandbox** sul Gateway, ma delegare esecuzioni approvate ad altri host.
* Fornire un target di esecuzione leggero e headless per l&#39;automazione o per nodi CI.

L&#39;esecuzione è comunque protetta da **approvazioni exec** e da liste di autorizzati per agente sul
nodo host, così puoi mantenere l&#39;accesso ai comandi con ambito limitato ed esplicito.

<div id="browser-proxy-zero-config">
  ## Proxy del browser (zero-config)
</div>

Gli host nodo espongono automaticamente un proxy del browser se `browser.enabled` non è
disabilitato sul nodo. Questo permette all&#39;agente di usare l&#39;automazione del browser su quel nodo
senza configurazione aggiuntiva.

Disabilitalo sul nodo se necessario:

```json5
{
  nodeHost: {
    browserProxy: {
      enabled: false
    }
  }
}
```

<div id="run-foreground">
  ## Esecuzione (in foreground)
</div>

```bash
openclaw node run --host <gateway-host> --port 18789
```

Options:

* `--host <host>`: host WebSocket del Gateway (predefinito: `127.0.0.1`)
* `--port <port>`: porta WebSocket del Gateway (predefinita: `18789`)
* `--tls`: usa TLS per la connessione al Gateway
* `--tls-fingerprint <sha256>`: impronta digitale prevista del certificato TLS (sha256)
* `--node-id <id>`: sovrascrive l’ID del nodo (azzera il token di abbinamento)
* `--display-name <name>`: sovrascrive il nome visualizzato del nodo

<div id="service-background">
  ## Servizio (in background)
</div>

Installa un host headless del nodo come servizio utente.

```bash
openclaw node install --host <gateway-host> --port 18789
```

Opzioni:

* `--host <host>`: host WebSocket del Gateway (predefinito: `127.0.0.1`)
* `--port <port>`: porta WebSocket del Gateway (predefinita: `18789`)
* `--tls`: Usa TLS per la connessione al Gateway
* `--tls-fingerprint <sha256>`: impronta digitale attesa del certificato TLS (sha256)
* `--node-id <id>`: Sovrascrivi l&#39;id del nodo (azzera il token di abbinamento)
* `--display-name <name>`: Sovrascrivi il nome visualizzato del nodo
* `--runtime <runtime>`: runtime del servizio (`node` o `bun`)
* `--force`: Reinstalla/sovrascrive se già installato

Gestisci il servizio:

```bash
openclaw node status
openclaw node stop
openclaw node restart
openclaw node uninstall
```

Usa `openclaw node run` per eseguire un nodo host in foreground (non come servizio).

I comandi di servizio accettano `--json` per produrre output leggibile dalle macchine.

<div id="pairing">
  ## Abbinamento
</div>

La prima connessione crea sul Gateway una richiesta di abbinamento del nodo in sospeso.
Approvala tramite:

```bash
openclaw nodes pending
openclaw nodes approve <requestId>
```

L&#39;host del nodo memorizza il relativo ID di nodo, il token, il nome visualizzato e le informazioni di connessione al Gateway in
`~/.openclaw/node.json`.

<div id="exec-approvals">
  ## Approvazioni di esecuzione
</div>

`system.run` è vincolato alle approvazioni di esecuzione locali:

* `~/.openclaw/exec-approvals.json`
* [Approvazioni di esecuzione](/it/tools/exec-approvals)
* `openclaw approvals --node <id|name|ip>` (modifica dal Gateway)