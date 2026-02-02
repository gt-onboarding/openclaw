---
title: Exec
summary: "Uso dello strumento Exec, modalità stdin e supporto TTY"
read_when:
  - Uso o modifica dello strumento Exec
  - Debug del comportamento di stdin o TTY
---

<div id="exec-tool">
  # Strumento exec
</div>

Esegue comandi della shell nello spazio di lavoro. Supporta l&#39;esecuzione in primo piano e in background tramite `process`.
Se `process` non è consentito, `exec` viene eseguito in modo sincrono e ignora `yieldMs`/`background`.
Le sessioni in background hanno scope per agente: `process` vede solo le sessioni dello stesso agente.

<div id="parameters">
  ## Parametri
</div>

* `command` (obbligatorio)
* `workdir` (predefinito: cwd)
* `env` (override di coppie chiave/valore)
* `yieldMs` (predefinito 10000): passa in background automaticamente dopo il ritardo
* `background` (bool): passa immediatamente in background
* `timeout` (secondi, predefinito 1800): termina allo scadere del timeout
* `pty` (bool): esegue in un pseudo-terminale quando disponibile (CLI che richiedono TTY, agenti di coding, UI di terminale)
* `host` (`sandbox | gateway | node`): dove eseguire
* `security` (`deny | allowlist | full`): modalità di enforcement per `gateway`/`node`
* `ask` (`off | on-miss | always`): richieste di approvazione per `gateway`/`node`
* `node` (string): id/nome del nodo per `host=node`
* `elevated` (bool): richiede modalità elevata (host Gateway); `security=full` viene forzato solo quando elevated risolve a `full`

Note:

* `host` ha come valore predefinito `sandbox`.
* `elevated` viene ignorato quando la sandbox è disattivata (exec è già in esecuzione sull&#39;host).
* Le approvazioni `gateway`/`node` sono controllate da `~/.openclaw/exec-approvals.json`.
* `node` richiede un nodo associato (app companion o nodo headless come host).
* Se sono disponibili più nodi, imposta `exec.node` o `tools.exec.node` per selezionarne uno.
* Sugli host non-Windows, exec usa `SHELL` quando impostato; se `SHELL` è `fish`, preferisce `bash` (o `sh`)
  da `PATH` per evitare script non compatibili con fish, quindi torna a `SHELL` se nessuno dei due esiste.
* Importante: la sandbox è **disattivata per impostazione predefinita**. Se la sandbox è disattivata, `host=sandbox` viene eseguito direttamente
  sull&#39;host Gateway (nessun container) e **non richiede approvazioni**. Per richiedere approvazioni, esegui con
  `host=gateway` e configura le approvazioni exec (o abilita la sandbox).

<div id="config">
  ## Config
</div>

* `tools.exec.notifyOnExit` (predefinito: true): quando è impostato su true, le sessioni exec in background accodano un evento di sistema e richiedono un heartbeat alla terminazione.
* `tools.exec.approvalRunningNoticeMs` (predefinito: 10000): emette una singola notifica di “in esecuzione” quando un exec soggetto ad approvazione dura più a lungo di questo valore (0 lo disabilita).
* `tools.exec.host` (predefinito: `sandbox`)
* `tools.exec.security` (predefinito: `deny` per sandbox, `allowlist` per Gateway + nodo quando non impostato)
* `tools.exec.ask` (predefinito: `on-miss`)
* `tools.exec.node` (predefinito: non impostato)
* `tools.exec.pathPrepend`: elenco di directory da anteporre a `PATH` per le esecuzioni exec.
* `tools.exec.safeBins`: binari sicuri che accettano solo input da stdin e che possono essere eseguiti senza voci esplicite nella lista di autorizzati.

Esempio:

```json5
{
  tools: {
    exec: {
      pathPrepend: ["~/bin", "/opt/oss/bin"]
    }
  }
}
```

<div id="path-handling">
  ### Gestione di `PATH`
</div>

* `host=gateway`: unisce il `PATH` della tua login shell all&#39;ambiente di esecuzione (a meno che la chiamata exec
  imposti già `env.PATH`). Il demone stesso continua a essere eseguito con un `PATH` minimale:
  * macOS: `/opt/homebrew/bin`, `/usr/local/bin`, `/usr/bin`, `/bin`
  * Linux: `/usr/local/bin`, `/usr/bin`, `/bin`
* `host=sandbox`: esegue `sh -lc` (login shell) all&#39;interno del container, quindi `/etc/profile` può reimpostare `PATH`.
  OpenClaw antepone `env.PATH` dopo il caricamento del profilo tramite una variabile d’ambiente interna (nessuna interpolazione
  della shell); `tools.exec.pathPrepend` si applica anche qui.
* `host=node`: solo le sovrascritture di ambiente che passi vengono inviate al nodo. `tools.exec.pathPrepend` si applica
  solo se la chiamata exec imposta già `env.PATH`. I nodi headless accettano `PATH` solo quando questo
  viene aggiunto in testa al PATH dell’host del nodo (nessuna sostituzione). I nodi macOS scartano del tutto le sovrascritture di `PATH`.

Binding del nodo per agente (usa l’indice dell’elenco agenti nella config):

```bash
openclaw config get agents.list
openclaw config set agents.list[0].tools.exec.node "node-id-or-name"
```

Control UI: la scheda Nodes include un piccolo pannello denominato &quot;Exec node binding&quot; con le stesse impostazioni.

<div id="session-overrides-exec">
  ## Override di sessione (`/exec`)
</div>

Usa `/exec` per impostare i valori predefiniti **per sessione** di `host`, `security`, `ask` e `node`.
Invia `/exec` senza argomenti per mostrare i valori correnti.

Esempio:

```
/exec host=gateway security=allowlist ask=on-miss node=mac-1
```

<div id="authorization-model">
  ## Modello di autorizzazione
</div>

`/exec` viene applicato solo per **mittenti autorizzati** (lista di autorizzati del canale/abbinamento e `commands.useAccessGroups`).
Aggiorna **solo lo stato della sessione** e non modifica la configurazione. Per disabilitare completamente `exec`, negalo tramite la
policy degli strumenti (`tools.deny: ["exec"]` o per agente). Le approvazioni dell&#39;host si applicano comunque, a meno che tu non imposti esplicitamente
`security=full` e `ask=off`.

<div id="exec-approvals-companion-app-node-host">
  ## Approvazioni Exec (app companion / host del nodo)
</div>

Gli agenti in sandbox possono richiedere un&#39;approvazione per ogni richiesta prima che `exec` venga eseguito sul Gateway o sull&#39;host del nodo.
Consulta [Exec approvals](/it/tools/exec-approvals) per la policy, la lista di autorizzati e il flusso della UI.

Quando sono richieste le approvazioni, il tool exec restituisce immediatamente
`status: "approval-pending"` e un ID di approvazione. Una volta approvato (o rifiutato / andato in timeout),
il Gateway emette eventi di sistema (`Exec finished` / `Exec denied`). Se il comando è ancora
in esecuzione dopo `tools.exec.approvalRunningNoticeMs`, viene emessa una singola notifica `Exec running`.

<div id="allowlist-safe-bins">
  ## Lista di autorizzati + binari sicuri
</div>

L&#39;applicazione della lista di autorizzati controlla **solo i percorsi binari risolti** (nessuna corrispondenza sul basename). Quando
`security=allowlist`, i comandi shell sono autorizzati automaticamente solo se ogni segmento della pipeline è
presente nella lista di autorizzati o è un binario sicuro. Il chaining (`;`, `&&`, `||`) e le redirezioni vengono bloccati in
modalità lista di autorizzati.

<div id="examples">
  ## Esempi
</div>

In primo piano:

```json
{"tool":"exec","command":"ls -la"}
```

Contesto e sondaggio:

```json
{"tool":"exec","command":"npm run build","yieldMs":1000}
{"tool":"process","action":"poll","sessionId":"<id>"}
```

Invio di tasti (stile tmux):

```json
{"tool":"process","action":"send-keys","sessionId":"<id>","keys":["Enter"]}
{"tool":"process","action":"send-keys","sessionId":"<id>","keys":["C-c"]}
{"tool":"process","action":"send-keys","sessionId":"<id>","keys":["Up","Up","Enter"]}
```

Invia (solo CR):

```json
{"tool":"process","action":"submit","sessionId":"<id>"}
```

Incolla (racchiuso tra parentesi per impostazione predefinita):

```json
{"tool":"process","action":"paste","sessionId":"<id>","text":"line1\nline2\n"}
```

<div id="apply_patch-experimental">
  ## apply_patch (sperimentale)
</div>

`apply_patch` è un sotto-strumento di `exec` per modifiche strutturate su più file.
Devi abilitarlo esplicitamente:

```json5
{
  tools: {
    exec: {
      applyPatch: { enabled: true, allowModels: ["gpt-5.2"] }
    }
  }
}
```

Note:

* Disponibile solo per i modelli OpenAI/OpenAI Codex.
* La policy degli strumenti si applica comunque; `allow: ["exec"]` abilita implicitamente `apply_patch`.
* La configurazione è definita in `tools.exec.applyPatch`.
