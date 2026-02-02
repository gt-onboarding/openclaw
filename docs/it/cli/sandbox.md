---
title: Sandbox CLI
summary: "Gestisci i container della sandbox e ispeziona la policy sandbox effettiva"
read_when: "Stai gestendo container sandbox o eseguendo il debug del comportamento della sandbox e delle tool policy."
status: active
---

<div id="sandbox-cli">
  # Sandbox CLI
</div>

Gestisci i container sandbox basati su Docker per l&#39;esecuzione isolata degli agenti.

<div id="overview">
  ## Panoramica
</div>

OpenClaw può eseguire agenti in container Docker isolati per motivi di sicurezza. I comandi `sandbox` ti aiutano a gestire questi container, in particolare dopo aggiornamenti o modifiche alla configurazione.

<div id="commands">
  ## Comandi
</div>

<div id="openclaw-sandbox-explain">
  ### `openclaw sandbox explain`
</div>

Ispeziona la modalità/scope/accesso allo spazio di lavoro **effettivi** della sandbox, i criteri degli strumenti della sandbox e i gate elevati (con percorsi delle chiavi di configurazione da correggere).

```bash
openclaw sandbox explain
openclaw sandbox explain --session agent:main:main
openclaw sandbox explain --agent work
openclaw sandbox explain --json
```

<div id="openclaw-sandbox-list">
  ### `openclaw sandbox list`
</div>

Elenca tutti i container sandbox con il relativo stato e le rispettive configurazioni.

```bash
openclaw sandbox list
openclaw sandbox list --browser  # Elenca solo i container del browser
openclaw sandbox list --json     # Output JSON
```

**L&#39;output contiene:**

* Nome e stato del container (in esecuzione/arrestato)
* Immagine Docker e corrispondenza con la configurazione
* Tempo trascorso dalla creazione
* Tempo di inattività (tempo dall&#39;ultimo utilizzo)
* Sessione/agente associato

<div id="openclaw-sandbox-recreate">
  ### `openclaw sandbox recreate`
</div>

Rimuove i contenitori sandbox per forzarne la ricreazione con immagini e configurazioni aggiornate.

```bash
openclaw sandbox recreate --all                # Ricrea tutti i container
openclaw sandbox recreate --session main       # Sessione specifica
openclaw sandbox recreate --agent mybot        # Agente specifico
openclaw sandbox recreate --browser            # Solo i container del browser
openclaw sandbox recreate --all --force        # Salta la conferma
```

**Opzioni:**

* `--all`: Ricrea tutti i container sandbox
* `--session <key>`: Ricrea il container per una specifica sessione
* `--agent <id>`: Ricrea i container per uno specifico agente
* `--browser`: Ricrea solo i container del browser
* `--force`: Salta la richiesta di conferma

**Importante:** I container vengono ricreati automaticamente la volta successiva che l&#39;agente viene utilizzato.

<div id="use-cases">
  ## Casi d&#39;uso
</div>

<div id="after-updating-docker-images">
  ### Dopo aver aggiornato le immagini Docker
</div>

```bash
# Scarica nuova immagine
docker pull openclaw-sandbox:latest
docker tag openclaw-sandbox:latest openclaw-sandbox:bookworm-slim

# Aggiorna config per usare nuova immagine
# Modifica config: agents.defaults.sandbox.docker.image (o agents.list[].sandbox.docker.image)

# Ricrea container
openclaw sandbox recreate --all
```

<div id="after-changing-sandbox-configuration">
  ### Dopo aver modificato la configurazione della sandbox
</div>

```bash
# Modifica la configurazione: agents.defaults.sandbox.* (oppure agents.list[].sandbox.*)

# Ricrea per applicare la nuova configurazione
openclaw sandbox recreate --all
```

<div id="after-changing-setupcommand">
  ### Dopo la modifica di setupCommand
</div>

```bash
openclaw sandbox recreate --all
# oppure solo un agente:
openclaw sandbox recreate --agent family
```

<div id="for-a-specific-agent-only">
  ### Solo per un agente specifico
</div>

```bash
# Aggiorna solo i container di un agente
openclaw sandbox recreate --agent alfred
```

<div id="why-is-this-needed">
  ## Perché è necessario?
</div>

**Problema:** Quando aggiorni le immagini Docker della sandbox o la configurazione:

* I container esistenti continuano a essere in esecuzione con le vecchie impostazioni
* I container vengono eliminati solo dopo 24 ore di inattività
* Gli agenti utilizzati regolarmente mantengono in esecuzione i vecchi container indefinitamente

**Soluzione:** Usa `openclaw sandbox recreate` per forzare la rimozione dei vecchi container. Verranno ricreati automaticamente con le impostazioni correnti quando saranno nuovamente necessari.

Suggerimento: preferisci `openclaw sandbox recreate` all’uso manuale di `docker rm`. Utilizza il sistema di naming dei container del Gateway ed evita incongruenze quando le chiavi di scope/session cambiano.

<div id="configuration">
  ## Configurazione
</div>

Le impostazioni della sandbox si trovano in `~/.openclaw/openclaw.json` nella sezione `agents.defaults.sandbox` (le impostazioni specifiche per singolo agente vanno in `agents.list[].sandbox`):

```jsonc
{
  "agents": {
    "defaults": {
      "sandbox": {
        "mode": "all",                    // off, non-main, all
        "scope": "agent",                 // session, agent, shared
        "docker": {
          "image": "openclaw-sandbox:bookworm-slim",
          "containerPrefix": "openclaw-sbx-"
          // ... more Docker options
        },
        "prune": {
          "idleHours": 24,               // Pulizia automatica dopo 24 ore di inattività
          "maxAgeDays": 7                // Auto-prune after 7 days
        }
      }
    }
  }
}
```

<div id="see-also">
  ## Vedi anche
</div>

* [Documentazione sulla sandbox](/it/gateway/sandboxing)
* [Configurazione dell&#39;Agente](/it/concepts/agent-workspace)
* [Comando doctor](/it/gateway/doctor) - Verifica la configurazione della sandbox