---
title: Sandbox vs Criteri degli Strumenti vs Elevated
summary: "Perché uno strumento viene bloccato: runtime della sandbox, criteri allow/deny degli strumenti e gate di esecuzione Elevated"
read_when: "Finisci nella 'sandbox jail' o vedi un rifiuto relativo a uno strumento/Elevated e vuoi sapere quale chiave di configurazione modificare."
status: active
---

<div id="sandbox-vs-tool-policy-vs-elevated">
  # Sandbox vs Politica degli Strumenti vs Elevated
</div>

OpenClaw ha tre controlli correlati (ma diversi):

1. **Sandbox** (`agents.defaults.sandbox.*` / `agents.list[].sandbox.*`) decide **dove vengono eseguiti gli strumenti** (Docker vs host).
2. **Politica degli strumenti** (`tools.*`, `tools.sandbox.tools.*`, `agents.list[].tools.*`) decide **quali strumenti sono disponibili/consentiti**.
3. **Elevated** (`tools.elevated.*`, `agents.list[].tools.elevated.*`) è una **via di fuga solo per exec** che consente di eseguire direttamente sull&#39;host quando sei in sandbox.

<div id="quick-debug">
  ## Debug rapido
</div>

Usa l&#39;Inspector per vedere cosa sta *davvero* facendo OpenClaw:

```bash
openclaw sandbox explain
openclaw sandbox explain --session agent:main:main
openclaw sandbox explain --agent work
openclaw sandbox explain --json
```

Stampa:

* modalità di sandbox/scope/accesso allo spazio di lavoro effettivi
* se la sessione è attualmente in sandbox (main vs non-main)
* policy effettive di allow/deny per gli strumenti in sandbox (e se provengono da agente/global/default)
* gate con privilegi elevati e percorsi delle chiavi fix-it

<div id="sandbox-where-tools-run">
  ## Sandbox: dove vengono eseguiti gli strumenti
</div>

La sandbox è gestita da `agents.defaults.sandbox.mode`:

* `"off"`: tutto viene eseguito sull&#39;host.
* `"non-main"`: solo le sessioni non-main sono in sandbox (causa comune di “sorprese” per gruppi/canali).
* `"all"`: tutto è in sandbox.

Consulta [Sandboxing](/it/gateway/sandboxing) per la matrice completa (scope, mount dello spazio di lavoro, immagini).

<div id="bind-mounts-security-quick-check">
  ### Bind mount (controllo rapido di sicurezza)
</div>

* `docker.binds` *perfora* il filesystem della sandbox: qualsiasi cosa monti è visibile all&#39;interno del container con la modalità che imposti (`:ro` o `:rw`).
* Se ometti la modalità, il valore predefinito è lettura-scrittura; preferisci `:ro` per sorgenti/secrets.
* `scope: "shared"` ignora i bind per agente (si applicano solo i bind globali).
* Montare `/var/run/docker.sock` di fatto consegna il controllo dell&#39;host alla sandbox; fallo solo intenzionalmente.
* L&#39;accesso allo spazio di lavoro (`workspaceAccess: "ro"`/`"rw"`) è indipendente dalle modalità dei bind.

<div id="tool-policy-which-tools-existare-callable">
  ## Politica degli strumenti: quali strumenti esistono/sono richiamabili
</div>

Contano due livelli:

* **Profilo degli strumenti**: `tools.profile` e `agents.list[].tools.profile` (lista di autorizzati di base)
* **Profilo degli strumenti del provider**: `tools.byProvider[provider].profile` e `agents.list[].tools.byProvider[provider].profile`
* **Politica globale/per-agente degli strumenti**: `tools.allow`/`tools.deny` e `agents.list[].tools.allow`/`agents.list[].tools.deny`
* **Politica degli strumenti del provider**: `tools.byProvider[provider].allow/deny` e `agents.list[].tools.byProvider[provider].allow/deny`
* **Politica degli strumenti della sandbox** (si applica solo quando la sandbox è attiva): `tools.sandbox.tools.allow`/`tools.sandbox.tools.deny` e `agents.list[].tools.sandbox.tools.*`

Regole pratiche:

* `deny` vince sempre.
* Se `allow` non è vuoto, tutto il resto è considerato bloccato.
* La politica degli strumenti è il limite invalicabile: `/exec` non può scavalcare uno strumento `exec` negato.
* `/exec` modifica solo i valori predefiniti della sessione per i mittenti autorizzati; non concede l’accesso agli strumenti.

Le chiavi degli strumenti del provider accettano sia `provider` (ad es. `google-antigravity`) sia `provider/model` (ad es. `openai/gpt-5.2`).

<div id="tool-groups-shorthands">
  ### Gruppi di tool (abbreviazioni)
</div>

Le policy dei tool (globali, per agente, per sandbox) supportano voci `group:*` che si espandono in più tool:

```json5
{
  tools: {
    sandbox: {
      tools: {
        allow: ["group:runtime", "group:fs", "group:sessions", "group:memory"]
      }
    }
  }
}
```

Gruppi disponibili:

* `group:runtime`: `exec`, `bash`, `process`
* `group:fs`: `read`, `write`, `edit`, `apply_patch`
* `group:sessions`: `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status`
* `group:memory`: `memory_search`, `memory_get`
* `group:ui`: `browser`, `canvas`
* `group:automation`: `cron`, `gateway`
* `group:messaging`: `message`
* `group:nodes`: `nodes`
* `group:openclaw`: tutti gli strumenti OpenClaw integrati (esclude i plugin dei provider)

<div id="elevated-exec-only-run-on-host">
  ## Elevated: solo exec, “esegui sull’host”
</div>

Elevated **non** concede strumenti aggiuntivi; influisce solo su `exec`.

* Se sei in sandbox, `/elevated on` (o `exec` con `elevated: true`) viene eseguito sull’host (le approvazioni possono comunque essere richieste).
* Usa `/elevated full` per saltare le approvazioni di exec per la sessione.
* Se stai già eseguendo direttamente sull’host, elevated è di fatto un no-op (resta comunque soggetto ai gate).
* Elevated **non** è limitato allo scope di una skill e **non** scavalca la allow/deny list degli strumenti.
* `/exec` è separato da elevated. Regola solo i valori predefiniti di exec a livello di sessione per i mittenti autorizzati.

Gate:

* Abilitazione: `tools.elevated.enabled` (e opzionalmente `agents.list[].tools.elevated.enabled`)
* Liste di autorizzati per i mittenti: `tools.elevated.allowFrom.<provider>` (e opzionalmente `agents.list[].tools.elevated.allowFrom.<provider>`)

Vedi [Elevated Mode](/it/tools/elevated).

<div id="common-sandbox-jail-fixes">
  ## Correzioni comuni per la “sandbox jail”
</div>

<div id="tool-x-blocked-by-sandbox-tool-policy">
  ### “Strumento X bloccato dalla policy degli strumenti della sandbox”
</div>

Opzioni di correzione (scegline una):

* Disabilita la sandbox: `agents.defaults.sandbox.mode=off` (o per singolo agente `agents.list[].sandbox.mode=off`)
* Consenti lo strumento all&#39;interno della sandbox:
  * rimuovilo da `tools.sandbox.tools.deny` (o per singolo agente `agents.list[].tools.sandbox.tools.deny`)
  * oppure aggiungilo a `tools.sandbox.tools.allow` (o per singolo agente nella lista allow)

<div id="i-thought-this-was-main-why-is-it-sandboxed">
  ### “Pensavo fosse main, perché è in sandbox?”
</div>

In modalità `"non-main"`, le chiavi di gruppo/canale *non* sono considerate main. Usa la chiave della sessione principale (mostrata da `sandbox explain`) oppure imposta la modalità su `"off"`.