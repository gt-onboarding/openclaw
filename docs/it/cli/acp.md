---
title: Acp
summary: "Esegui il bridge ACP per le integrazioni IDE"
read_when:
  - Configurazione delle integrazioni IDE basate su ACP
  - Debug del routing delle sessioni ACP verso il Gateway
---

<div id="acp">
  # acp
</div>

Avvia il bridge ACP (Agent Client Protocol) che comunica con un Gateway OpenClaw.

Questo comando utilizza ACP tramite stdio per gli IDE e inoltra i prompt al Gateway
tramite WebSocket. Mantiene le sessioni ACP mappate alle chiavi di sessione del Gateway.

<div id="usage">
  ## Utilizzo
</div>

```bash
openclaw acp

# Remote Gateway
openclaw acp --url wss://gateway-host:18789 --token <token>

# Attach to an existing session key
openclaw acp --session agent:main:main

# Attach by label (must already exist)
openclaw acp --session-label "support inbox"

# Reimposta la chiave di sessione prima del primo prompt
openclaw acp --session agent:main:main --reset-session
```

<div id="acp-client-debug">
  ## Client ACP (debug)
</div>

Usa il client ACP integrato per eseguire un rapido sanity check del bridge senza un IDE.
Avvia il bridge ACP e ti consente di inserire prompt in modo interattivo.

```bash
openclaw acp client

# Point the spawned bridge at a remote Gateway
openclaw acp client --server-args --url wss://gateway-host:18789 --token <token>

# Sovrascrivi il comando del server (predefinito: openclaw)
openclaw acp client --server "node" --server-args openclaw.mjs acp --url ws://127.0.0.1:19001
```

<div id="how-to-use-this">
  ## Come usarlo
</div>

Usa ACP quando un IDE (o un altro client) supporta Agent Client Protocol e vuoi
che gestisca una sessione del Gateway OpenClaw.

1. Assicurati che il Gateway sia in esecuzione (locale o remoto).
2. Configura la destinazione del Gateway (config o flag).
3. Imposta il tuo IDE per eseguire `openclaw acp` tramite stdio.

Configurazione di esempio (persistente):

```bash
openclaw config set gateway.remote.url wss://gateway-host:18789
openclaw config set gateway.remote.token <token>
```

Esempio di esecuzione diretta (senza salvataggio della configurazione):

```bash
openclaw acp --url wss://gateway-host:18789 --token <token>
```

<div id="selecting-agents">
  ## Selezione degli agenti
</div>

ACP non seleziona direttamente gli agenti. Instrada in base alla chiave di sessione del Gateway.

Usa chiavi di sessione con scope agente per indirizzare uno specifico agente:

```bash
openclaw acp --session agent:main:main
openclaw acp --session agent:design:main
openclaw acp --session agent:qa:bug-123
```

Ogni sessione ACP è associata a una singola chiave di sessione del Gateway. Un agente può avere molte
sessioni; ACP usa per impostazione predefinita una sessione isolata `acp:<uuid>` a meno che tu non sostituisca
la chiave o l&#39;etichetta.

<div id="zed-editor-setup">
  ## Configurazione dell&#39;editor Zed
</div>

Aggiungi un agente ACP personalizzato nel file `~/.config/zed/settings.json` (oppure usa la UI Impostazioni di Zed):

```json
{
  "agent_servers": {
    "OpenClaw ACP": {
      "type": "custom",
      "command": "openclaw",
      "args": ["acp"],
      "env": {}
    }
  }
}
```

Per puntare a uno specifico Gateway o agente:

```json
{
  "agent_servers": {
    "OpenClaw ACP": {
      "type": "custom",
      "command": "openclaw",
      "args": [
        "acp",
        "--url", "wss://gateway-host:18789",
        "--token", "<token>",
        "--session", "agent:design:main"
      ],
      "env": {}
    }
  }
}
```

In Zed, apri il pannello Agente e seleziona &quot;OpenClaw ACP&quot; per creare un thread.

<div id="session-mapping">
  ## Mappatura delle sessioni
</div>

Per impostazione predefinita, le sessioni ACP ricevono una chiave di sessione del Gateway isolata con prefisso `acp:`.
Per riutilizzare una sessione nota, passa una chiave o un&#39;etichetta di sessione:

* `--session <key>`: usa una specifica chiave di sessione del Gateway.
* `--session-label <label>`: risolvi una sessione esistente tramite l&#39;etichetta.
* `--reset-session`: genera un nuovo ID di sessione per quella chiave (stessa chiave, nuova cronologia).

Se il tuo client ACP supporta i metadati, puoi eseguire l&#39;override a livello di singola sessione:

```json
{
  "_meta": {
    "sessionKey": "agent:main:main",
    "sessionLabel": "inbox di supporto",
    "resetSession": true
  }
}
```

Per saperne di più sulle chiavi di sessione, consulta [/concepts/session](/it/concepts/session).

<div id="options">
  ## Opzioni
</div>

* `--url <url>`: URL WebSocket del Gateway (valore predefinito: gateway.remote.url quando configurato).
* `--token <token>`: token di autenticazione del Gateway.
* `--password <password>`: password di autenticazione del Gateway.
* `--session <key>`: chiave di sessione predefinita.
* `--session-label <label>`: etichetta di sessione predefinita da risolvere.
* `--require-existing`: genera un errore se la chiave/etichetta di sessione non esiste.
* `--reset-session`: reimposta la chiave di sessione prima del primo uso.
* `--no-prefix-cwd`: non anteporre ai prompt la directory di lavoro.
* `--verbose, -v`: logging dettagliato su stderr.

<div id="acp-client-options">
  ### Opzioni di `acp client`
</div>

* `--cwd &lt;dir&gt;`: directory di lavoro per la sessione ACP.
* `--server &lt;command&gt;`: comando del server ACP (predefinito: `openclaw`).
* `--server-args &lt;args...&gt;`: argomenti aggiuntivi passati al server ACP.
* `--server-verbose`: abilita il logging dettagliato sul server ACP.
* `--verbose, -v`: logging dettagliato del client.