---
title: Approvazioni exec
summary: "Approvazioni exec, liste di autorizzati e prompt di evasione dalla sandbox"
read_when:
  - Configurazione delle approvazioni exec o delle liste di autorizzati
  - Implementazione della UX per le approvazioni exec nell'app macOS
  - Revisione dei prompt di evasione dalla sandbox e delle relative implicazioni
---

<div id="exec-approvals">
  # Approvazioni di exec
</div>

Le approvazioni di exec sono la **barriera di sicurezza dell&#39;host del nodo / companion app** che consente a un agente in sandbox di eseguire
comandi su un host reale (`gateway` o `node`). Puoi pensarle come un interblocco di sicurezza:
i comandi sono consentiti solo quando policy + lista di autorizzati + (eventuale) approvazione dell’utente sono tutti allineati.
Le approvazioni di exec sono **in aggiunta** alla tool policy e al gating elevato (a meno che `elevated` non sia impostato su `full`, che salta le approvazioni).
La policy effettiva è la **più restrittiva** tra `tools.exec.*` e i valori predefiniti delle approvazioni; se un campo delle approvazioni viene omesso, viene usato il valore di `tools.exec`.

Se la UI della companion app **non è disponibile**, qualunque richiesta che richieda un prompt viene
risolta tramite l’**ask fallback** (impostazione predefinita: deny).

<div id="where-it-applies">
  ## Ambito di applicazione
</div>

Le approvazioni di esecuzione vengono applicate localmente sull&#39;host di esecuzione:

* **gateway host** → processo `openclaw` sulla macchina del Gateway
* **node host** → runner del nodo (app companion macOS o host nodo headless)

Dettaglio macOS:

* **node host service** inoltra `system.run` all&#39;**app macOS** tramite IPC locale.
* **app macOS** applica le approvazioni + esegue il comando nel contesto UI.

<div id="settings-and-storage">
  ## Impostazioni e archiviazione
</div>

Le approvazioni sono salvate in un file JSON locale sull&#39;host di esecuzione:

`~/.openclaw/exec-approvals.json`

Schema di esempio:

```json
{
  "version": 1,
  "socket": {
    "path": "~/.openclaw/exec-approvals.sock",
    "token": "base64url-token"
  },
  "defaults": {
    "security": "deny",
    "ask": "on-miss",
    "askFallback": "deny",
    "autoAllowSkills": false
  },
  "agents": {
    "main": {
      "security": "allowlist",
      "ask": "on-miss",
      "askFallback": "deny",
      "autoAllowSkills": true,
      "allowlist": [
        {
          "id": "B0C8C0B3-2C2D-4F8A-9A3C-5A4B3C2D1E0F",
          "pattern": "~/Projects/**/bin/rg",
          "lastUsedAt": 1737150000000,
          "lastUsedCommand": "rg -n TODO",
          "lastResolvedPath": "/Users/user/Projects/.../bin/rg"
        }
      ]
    }
  }
}
```

<div id="policy-knobs">
  ## Opzioni di policy
</div>

<div id="security-execsecurity">
  ### Sicurezza (`exec.security`)
</div>

* **deny**: blocca tutte le richieste di exec sull&#39;host.
* **allowlist**: consente solo i comandi presenti nella lista di autorizzati.
* **full**: consente tutto (equivalente a `elevated`).

<div id="ask-execask">
  ### Ask (`exec.ask`)
</div>

* **off**: non richiedere mai conferma.
* **on-miss**: richiedere conferma solo quando la lista di autorizzati non ha una corrispondenza.
* **always**: richiedere conferma per ogni comando.

<div id="ask-fallback-askfallback">
  ### Ask fallback (`askFallback`)
</div>

Se è richiesto un prompt ma nessuna UI è raggiungibile, il fallback decide:

* **deny**: blocca.
* **allowlist**: consente solo se la lista di autorizzati corrisponde.
* **full**: consente.

<div id="allowlist-per-agent">
  ## Lista di autorizzati (per agente)
</div>

Le liste di autorizzati sono **per agente**. Se esistono più agenti, cambia l’agente che
stai modificando nell’app macOS. I pattern sono **corrispondenze glob senza distinzione tra maiuscole e minuscole**.
I pattern devono risolversi in **percorsi di eseguibili** (le voci con solo basename vengono ignorate).
Le voci legacy in `agents.default` vengono migrate a `agents.main` in fase di caricamento.

Esempi:

* `~/Projects/**/bin/bird`
* `~/.local/bin/*`
* `/opt/homebrew/bin/rg`

Ogni voce della lista di autorizzati registra:

* **id** UUID stabile usato per l’identità nella UI (opzionale)
* **ultimo utilizzo** timestamp
* **ultimo comando utilizzato**
* **ultimo percorso risolto**

<div id="auto-allow-skill-clis">
  ## CLI delle abilità con autorizzazione automatica
</div>

Quando **CLI delle abilità con autorizzazione automatica** è abilitato, gli eseguibili indicati da abilità note
sono considerati inclusi nella lista di autorizzati sui nodi (nodo macOS o host di un nodo headless). Questo usa
`skills.bins` tramite il Gateway RPC per recuperare l&#39;elenco dei binari delle abilità. Disabilita questa opzione se vuoi una lista di autorizzati gestita rigorosamente in modo manuale.

<div id="safe-bins-stdin-only">
  ## Safe bins (solo stdin)
</div>

`tools.exec.safeBins` definisce un piccolo elenco di binari **solo stdin** (per esempio `jq`)
che possono essere eseguiti in modalità lista di autorizzati **senza** voci esplicite nella lista di autorizzati. I safe bins rifiutano
argomenti posizionali di file e token simili a percorsi, quindi possono operare solo sullo stream in ingresso.
Il chaining della shell e le redirezioni non sono consentiti automaticamente in modalità lista di autorizzati.

Il chaining della shell (`&&`, `||`, `;`) è consentito quando ogni segmento di primo livello soddisfa i criteri della lista di autorizzati
(inclusi safe bins o skill autorizzate automaticamente). Le redirezioni restano non supportate in modalità lista di autorizzati.

Safe bins predefiniti: `jq`, `grep`, `cut`, `sort`, `uniq`, `head`, `tail`, `tr`, `wc`.

<div id="control-ui-editing">
  ## Modifica tramite Control UI
</div>

Usa la scheda **Control UI → Nodes → Exec approvals** per modificare i valori
predefiniti, gli override per singolo agente e la lista di autorizzati. Scegli
uno scope (Defaults oppure un agente), adatta la policy, aggiungi/rimuovi
pattern della lista di autorizzati, quindi fai clic su **Save**. La UI mostra i
metadati **last used** per ciascun pattern in modo che tu possa mantenere
l&#39;elenco ordinato.

Il selettore di destinazione consente di scegliere **Gateway** (approvazioni locali) oppure un
**nodo**. I nodi devono esporre `system.execApprovals.get/set` (app macOS o host
del nodo headless). Se un nodo non espone ancora le exec approvals, modifica
direttamente il suo file locale `~/.openclaw/exec-approvals.json`.

CLI: `openclaw approvals` supporta la modifica sul Gateway o sul nodo (vedi
[Approvals CLI](/it/cli/approvals)).

<div id="approval-flow">
  ## Flusso di approvazione
</div>

Quando è richiesta un&#39;approvazione, il Gateway trasmette `exec.approval.requested` ai client degli operatori.
La Control UI e l&#39;app macOS gestiscono la richiesta tramite `exec.approval.resolve`, quindi il Gateway inoltra la
richiesta approvata all&#39;host del nodo.

Quando è richiesta l&#39;approvazione, lo strumento exec restituisce immediatamente un ID di approvazione. Usa quell&#39;ID per
correlare i successivi eventi di sistema (`Exec finished` / `Exec denied`). Se non arriva alcuna decisione prima del
timeout, la richiesta viene trattata come un timeout di approvazione ed esposta come motivo di diniego.

La finestra di conferma include:

* comando + argomenti
* cwd
* agent id
* percorso dell&#39;eseguibile risolto
* host + metadati di policy

Azioni:

* **Allow once** → esegui ora
* **Always allow** → aggiungi alla lista di autorizzati + esegui
* **Deny** → blocca

<div id="approval-forwarding-to-chat-channels">
  ## Inoltro delle approvazioni ai canali di chat
</div>

Puoi inoltrare i prompt di approvazione `exec` a qualsiasi canale di chat (inclusi i canali dei plugin) e approvarli
con `/approve`. Questo usa la normale pipeline di consegna in uscita.

Configurazione:

```json5
{
  approvals: {
    exec: {
      enabled: true,
      mode: "session", // "sessione" | "destinazioni" | "entrambi"
      agentFilter: ["main"],
      sessionFilter: ["discord"], // substring or regex
      targets: [
        { channel: "slack", to: "U12345678" },
        { channel: "telegram", to: "123456789" }
      ]
    }
  }
}
```

Rispondi nella chat:

```
/approve <id> allow-once
/approve <id> allow-always
/approve <id> deny
```

<div id="macos-ipc-flow">
  ### Flusso IPC in macOS
</div>

```
Gateway -> Node Service (WS)
                 |  IPC (UDS + token + HMAC + TTL)
                 v
             Mac App (UI + approvals + system.run)
```

Note sulla sicurezza:

* Socket Unix con modalità `0600`, token memorizzato in `exec-approvals.json`.
* Verifica del peer con lo stesso UID.
* Challenge/response (nonce + token HMAC + hash della richiesta) + TTL breve.

<div id="system-events">
  ## Eventi di sistema
</div>

Il ciclo di vita di Exec viene esposto tramite messaggi di sistema:

* `Exec running` (solo se il comando supera la soglia di notifica per l’esecuzione)
* `Exec finished`
* `Exec denied`

Questi vengono pubblicati nella sessione dell’agente dopo che il nodo ha segnalato l’evento.
Le approvazioni di exec eseguite sul Gateway emettono gli stessi eventi di ciclo di vita quando il comando viene completato (e facoltativamente quando rimane in esecuzione oltre la soglia).
Gli exec soggetti ad approvazione riutilizzano l’ID di approvazione come `runId` in questi messaggi, per una correlazione più semplice.

<div id="implications">
  ## Implicazioni
</div>

* **full** è potente; preferisci le liste di autorizzati quando possibile.
* **ask** ti mantiene al corrente pur consentendo approvazioni rapide.
* Le liste di autorizzati per agente impediscono che le approvazioni di un agente si propaghino agli altri.
* Le approvazioni si applicano solo alle richieste di host exec provenienti da **mittenti autorizzati**. I mittenti non autorizzati non possono eseguire `/exec`.
* `/exec security=full` è una scorciatoia a livello di sessione per operatori autorizzati e, per progettazione, salta la fase di approvazione.
  Per bloccare completamente l&#39;host exec, imposta la sicurezza delle approvazioni su `deny` o nega lo strumento `exec` tramite la policy degli strumenti.

Correlato:

* [Strumento Exec](/it/tools/exec)
* [Modalità elevata](/it/tools/elevated)
* [Abilità](/it/tools/skills)