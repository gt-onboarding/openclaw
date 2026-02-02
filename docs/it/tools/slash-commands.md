---
title: Comandi slash
summary: "Comandi slash: di testo vs nativi, configurazione e comandi supportati"
read_when:
  - Uso o configurazione dei comandi di chat
  - Debug dell’instradamento dei comandi o dei permessi
---

<div id="slash-commands">
  # Comandi slash
</div>

I comandi sono gestiti dal Gateway. La maggior parte dei comandi deve essere inviata come messaggio **autonomo** che inizia con `/`.
Il comando di chat bash riservato all&#39;host usa `! <cmd>` (con `/bash <cmd>` come alias).

Esistono due sistemi correlati:

* **Comandi**: messaggi autonomi `/...`.
* **Direttive**: `/think`, `/verbose`, `/reasoning`, `/elevated`, `/exec`, `/model`, `/queue`.
  * Le direttive vengono rimosse dal messaggio prima che il modello lo veda.
  * Nei normali messaggi di chat (non solo direttive), sono trattate come “suggerimenti in linea” e **non** rendono persistenti le impostazioni di sessione.
  * Nei messaggi composti solo da direttive (il messaggio contiene solo direttive), queste persistono nella sessione e rispondono con una conferma di ricezione.
  * Le direttive vengono applicate solo per **mittenti autorizzati** (lista di autorizzati del canale/abbinamento più `commands.useAccessGroups`).
    I mittenti non autorizzati vedono le direttive trattate come semplice testo.

Esistono anche alcune **scorciatoie in linea** (solo mittenti presenti nella lista di autorizzati o comunque autorizzati): `/help`, `/commands`, `/status`, `/whoami` (`/id`).
Vengono eseguite immediatamente, rimosse prima che il modello veda il messaggio, e il testo rimanente prosegue nel flusso normale.

<div id="config">
  ## Config
</div>

```json5
{
  commands: {
    native: "auto",
    nativeSkills: "auto",
    text: true,
    bash: false,
    bashForegroundMs: 2000,
    config: false,
    debug: false,
    restart: false,
    useAccessGroups: true
  }
}
```

* `commands.text` (predefinito `true`) abilita l&#39;interpretazione di `/...` nei messaggi di chat.
  * Sulle piattaforme senza comandi nativi (WhatsApp/WebChat/Signal/iMessage/Google Chat/MS Teams), i comandi testuali continuano a funzionare anche se imposti questo valore a `false`.
* `commands.native` (predefinito `"auto"`) registra i comandi nativi.
  * Auto: attivo per Discord/Telegram; disattivo per Slack (finché non aggiungi gli slash command); ignorato per i provider senza supporto nativo.
  * Imposta `channels.discord.commands.native`, `channels.telegram.commands.native` o `channels.slack.commands.native` per eseguire l&#39;override per singolo provider (bool o `"auto"`).
  * `false` cancella all&#39;avvio i comandi precedentemente registrati su Discord/Telegram. I comandi Slack sono gestiti nell&#39;app Slack e non vengono rimossi automaticamente.
* `commands.nativeSkills` (predefinito `"auto"`) registra nativamente i comandi di **abilità** quando supportato.
  * Auto: attivo per Discord/Telegram; disattivo per Slack (Slack richiede la creazione di uno slash command per ogni abilità).
  * Imposta `channels.discord.commands.nativeSkills`, `channels.telegram.commands.nativeSkills` o `channels.slack.commands.nativeSkills` per eseguire l&#39;override per singolo provider (bool o `"auto"`).
* `commands.bash` (predefinito `false`) abilita `! <cmd>` per eseguire comandi della shell dell&#39;host (`/bash <cmd>` è un alias; richiede liste di autorizzati `tools.elevated`).
* `commands.bashForegroundMs` (predefinito `2000`) controlla per quanto tempo bash attende prima di passare alla modalità in background (`0` lo manda immediatamente in background).
* `commands.config` (predefinito `false`) abilita `/config` (legge/scrive `openclaw.json`).
* `commands.debug` (predefinito `false`) abilita `/debug` (override validi solo a runtime).
* `commands.useAccessGroups` (predefinito `true`) applica liste di autorizzati e policy per i comandi.

<div id="command-list">
  ## Elenco comandi
</div>

Testo + comandi nativi (quando abilitati):

* `/help`
* `/commands`
* `/skill <name> [input]` (esegue una skill per nome)
* `/status` (mostra lo stato corrente; include utilizzo/quota del provider del modello corrente quando disponibile)
* `/allowlist` (elenca/aggiunge/rimuove voci dalla lista di autorizzati)
* `/approve <id> allow-once|allow-always|deny` (risolve i prompt di approvazione di esecuzione)
* `/context [list|detail|json]` (spiega il “contesto”; `detail` mostra la dimensione per file + per tool + per skill + system prompt)
* `/whoami` (mostra il tuo ID mittente; alias: `/id`)
* `/subagents list|stop|log|info|send` (ispeziona, arresta, registra o invia messaggi alle esecuzioni dei sottoagenti per la sessione corrente)
* `/config show|get|set|unset` (rende persistente la configurazione su disco, solo proprietario; richiede `commands.config: true`)
* `/debug show|set|unset|reset` (override runtime, solo proprietario; richiede `commands.debug: true`)
* `/usage off|tokens|full|cost` (piè di pagina per risposta con i dati di utilizzo o riepilogo locale dei costi)
* `/tts off|always|inbound|tagged|status|provider|limit|summary|audio` (controlla il TTS; vedi [/tts](/it/tts))
  * Discord: il comando nativo è `/voice` (Discord riserva `/tts`); il comando testuale `/tts` funziona comunque.
* `/stop`
* `/restart`
* `/dock-telegram` (alias: `/dock_telegram`) (instrada le risposte su Telegram)
* `/dock-discord` (alias: `/dock_discord`) (instrada le risposte su Discord)
* `/dock-slack` (alias: `/dock_slack`) (instrada le risposte su Slack)
* `/activation mention|always` (solo gruppi)
* `/send on|off|inherit` (solo proprietario)
* `/reset` o `/new [model]` (hint di modello opzionale; il resto viene passato così com&#39;è)
* `/think <off|minimal|low|medium|high|xhigh>` (scelte dinamiche per modello/provider; alias: `/thinking`, `/t`)
* `/verbose on|full|off` (alias: `/v`)
* `/reasoning on|off|stream` (alias: `/reason`; quando attivo, invia un messaggio separato con prefisso `Reasoning:`; `stream` = solo bozza Telegram)
* `/elevated on|off|ask|full` (alias: `/elev`; `full` salta le approvazioni di esecuzione)
* `/exec host=<sandbox|gateway|node> security=<deny|allowlist|full> ask=<off|on-miss|always> node=<id>` (invia `/exec` per mostrare i valori correnti)
* `/model <name>` (alias: `/models`; oppure `/<alias>` da `agents.defaults.models.*.alias`)
* `/queue <mode>` (più opzioni come `debounce:2s cap:25 drop:summarize`; invia `/queue` per vedere le impostazioni correnti)
* `/bash <command>` (solo host; alias di `! <command>`; richiede `commands.bash: true` + liste di autorizzati `tools.elevated`)

Solo testo:

* `/compact [instructions]` (vedi [/concepts/compaction](/it/concepts/compaction))
* `! <command>` (solo host; uno alla volta; usa `!poll` + `!stop` per job di lunga durata)
* `!poll` (controlla output/stato; accetta `sessionId` opzionale; funziona anche `/bash poll`)
* `!stop` (ferma il job bash in esecuzione; accetta `sessionId` opzionale; funziona anche `/bash stop`)

Note:

* I comandi accettano un due punti `:` facoltativo tra il comando e gli argomenti (ad es. `/think: high`, `/send: on`, `/help:`).
* `/new <model>` accetta un alias di modello, `provider/model` o un nome di provider (corrispondenza approssimativa); se non c&#39;è corrispondenza, il testo viene trattato come corpo del messaggio.
* Per una panoramica completa dell&#39;utilizzo dei provider, usa `openclaw status --usage`.
* `/allowlist add|remove` richiede `commands.config=true` e rispetta `configWrites` del canale.
* `/usage` controlla il footer sull&#39;utilizzo per risposta; `/usage cost` stampa un riepilogo locale dei costi dai log di sessione di OpenClaw.
* `/restart` è disabilitato per impostazione predefinita; imposta `commands.restart: true` per abilitarlo.
* `/verbose` è pensato per il debug e per una visibilità aggiuntiva; tienilo **disattivato** nell&#39;uso normale.
* `/reasoning` (e `/verbose`) sono rischiosi nei contesti di gruppo: possono rivelare ragionamenti interni o output di strumenti che non intendevi esporre. È preferibile lasciarli disattivati, soprattutto nelle chat di gruppo.
* **Percorso rapido:** i messaggi composti solo da comandi provenienti da mittenti nella lista di autorizzati vengono gestiti immediatamente (aggirano coda + modello).
* **Controllo tramite menzioni nei gruppi:** i messaggi composti solo da comandi provenienti da mittenti nella lista di autorizzati ignorano i requisiti di menzione.
* **Scorciatoie in linea (solo mittenti nella lista di autorizzati):** alcuni comandi funzionano anche quando sono incorporati in un messaggio normale e vengono rimossi prima che il modello veda il testo rimanente.
  * Esempio: `hey /status` attiva una risposta di stato e il testo rimanente prosegue nel flusso normale.
* Attualmente: `/help`, `/commands`, `/status`, `/whoami` (`/id`).
* I messaggi non autorizzati composti solo da comandi vengono ignorati senza alcuna notifica e i token in linea `/...` sono trattati come testo normale.
* **Comandi delle abilità:** le abilità `user-invocable` sono esposte come comandi slash. I nomi vengono normalizzati a `a-z0-9_` (massimo 32 caratteri); in caso di collisione viene aggiunto un suffisso numerico (ad es. `_2`).
  * `/skill <name> [input]` esegue un&#39;abilità per nome (utile quando i limiti dei comandi nativi impediscono comandi per singola abilità).
  * Per impostazione predefinita, i comandi delle abilità sono inoltrati al modello come una richiesta normale.
  * Le abilità possono opzionalmente dichiarare `command-dispatch: tool` per instradare il comando direttamente a uno strumento (deterministico, senza modello).
  * Esempio: `/prose` (plugin OpenProse) — vedi [OpenProse](/it/prose).
* **Argomenti dei comandi nativi:** Discord usa l&#39;autocompletamento per le opzioni dinamiche (e i menu a pulsanti quando ometti argomenti obbligatori). Telegram e Slack mostrano un menu a pulsanti quando un comando supporta delle scelte e ometti l&#39;argomento.

<div id="usage-surfaces-what-shows-where">
  ## Superfici di utilizzo (cosa viene mostrato dove)
</div>

* **Utilizzo/quota del provider** (esempio: “Claude 80% residuo”) viene mostrato in `/status` per il provider di modello corrente quando il monitoraggio dell&#39;utilizzo è abilitato.
* **Token/costo per risposta** è controllato da `/usage off|tokens|full` (aggiunto alle risposte normali).
* `/model status` riguarda **modelli/auth/endpoint**, non l&#39;utilizzo.

<div id="model-selection-model">
  ## Selezione del modello (`/model`)
</div>

`/model` è implementato come direttiva.

Esempi:

```
/model
/model list
/model 3
/model openai/gpt-5.2
/model opus@anthropic:default
/model status
```

Note:

* `/model` e `/model list` mostrano un selettore compatto e numerato (famiglia di modelli + provider disponibili).
* `/model &lt;#&gt;` seleziona un elemento da quel selettore (e preferisce il provider attuale quando possibile).
* `/model status` mostra una vista dettagliata, includendo l&#39;endpoint del provider configurato (`baseUrl`) e la modalità `api` (`api`) quando disponibile.

<div id="debug-overrides">
  ## Override di debug
</div>

`/debug` ti consente di impostare override di configurazione **solo a runtime** (in memoria, non su disco). Solo per il proprietario. Disabilitato per impostazione predefinita; attivalo con `commands.debug: true`.

Esempi:

```
/debug show
/debug set messages.responsePrefix="[openclaw]"
/debug set channels.whatsapp.allowFrom=["+1555","+4477"]
/debug unset messages.responsePrefix
/debug reset
```

Note:

* Gli override si applicano immediatamente alle nuove letture della configurazione, ma **non** vengono scritti in `openclaw.json`.
* Usa `/debug reset` per cancellare tutti gli override e tornare alla configurazione salvata su disco.

<div id="config-updates">
  ## Aggiornamenti di configurazione
</div>

`/config` scrive nella configurazione salvata su disco (`openclaw.json`). Utilizzabile solo dal proprietario. Disabilitato per impostazione predefinita; abilitalo con `commands.config: true`.

Esempi:

```
/config show
/config show messages.responsePrefix
/config get messages.responsePrefix
/config set messages.responsePrefix="[openclaw]"
/config unset messages.responsePrefix
```

Note:

* La configurazione viene validata prima del salvataggio; le modifiche non valide vengono scartate.
* Gli aggiornamenti di `/config` vengono mantenuti anche dopo i riavvii.

<div id="surface-notes">
  ## Note generali
</div>

* **Comandi testuali** vengono eseguiti nella normale sessione di chat (i DM condividono `main`, i gruppi hanno la propria sessione).
* **Comandi nativi** usano sessioni isolate:
  * Discord: `agent:<agentId>:discord:slash:<userId>`
  * Slack: `agent:<agentId>:slack:slash:<userId>` (prefisso configurabile tramite `channels.slack.slashCommand.sessionPrefix`)
  * Telegram: `telegram:slash:<userId>` (indirizza la sessione di chat tramite `CommandTargetSessionKey`)
* **`/stop`** si applica alla sessione di chat attiva, in modo da poter interrompere l&#39;esecuzione corrente.
* **Slack:** `channels.slack.slashCommand` è ancora supportato per un singolo comando in stile `/openclaw`. Se abiliti `commands.native`, devi creare un comando slash Slack per ciascun comando integrato (con gli stessi nomi di `/help`). I menu degli argomenti dei comandi per Slack vengono mostrati come pulsanti Block Kit effimeri.